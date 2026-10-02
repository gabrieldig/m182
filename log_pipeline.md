# Log-Pipeline

## Ausgangslage & Szenario
- **Unternehmen:** Mittelständischer Betrieb mit rund 50 Mitarbeitenden, Onlineshop (Webserver in DMZ), interner Datenbankserver mit Kundendaten, 50 Clients (teilweise mobile Notebooks via VPN).
- **Vorfall:** Auf dem Notebook eines Support-Mitarbeitenden versucht jemand um **02:14 Uhr**, sich mehrfach mit falschen Anmeldedaten anzumelden (potenzieller Brute-Force- oder Credential-Stuffing-Angriff).
- **Ziel:** Aufzeigen des vollständigen Wegs der Meldung von der Entstehung bis zur Reaktion sowie Analyse der Schwachstellen und konzeptionellen Risiken der Log-Pipeline.

---

## Aufgabe 1: Die Pipeline aufzeigen

### 1.1 Skizze der Pipeline

```text
[ Endpunkt: Support-Notebook ]
  │
  ├─► (1) Ereignis: Fehlgeschlagene Logins um 02:14 Uhr
  │        │
  │        ▼
  ├─► (2) Lokaler Logeintrag: Windows Event Log (Event ID 4625) / /var/log/auth.log
  │        │
  │        ▼
  ├─► (3) Agent / Collector: Wazuh-Agent / Eventlog-Collector liest Event
  │        │  [KI-Ergänzung: Lokales Disk-Spooling falls VPN offline]
  │        ▼
  └────── (4) Übermittlung: Verschlüsselter TLS-Tunnel (Port 1514) über VPN
               │
               ▼
[ Zentraler Monitoring-Server / SIEM ]
  │
  ├─► (5) Normalisierung & Parsing: Wazuh-Server dekodiert Rohtext -> JSON-Schema
  │        │
  │        ▼
  ├─► (6) Regelauswertung: Wazuh Analysis Engine (Regel 5712: "Multiple authentication failures")
  │        │
  │        ▼
  ├─► (7) Alarmierung: Erzeugung Alert (Level 10) -> Dashboard-Eintrag, Mail/Slack an On-Call
  │        │
  ├─► (8) Reaktion (HIDS vs. HIPS):
  │        ├─ HIDS: Nur Benachrichtigung & Dashboard-Markierung (passiv)
  │        └─ HIPS: Wazuh Active Response sendet Befehl an Agent -> IP/User temporär sperren (aktiv)
  │        │
  │        ▼
  └─► (9) Aufbewahrung & Archivierung: Wazuh-Indexer (Elasticsearch/OpenSearch) & WORM/Cold-Storage
```

> **Hinweis zur Skizze (KI-Review):**
> Die Station *(3a) Lokales Disk-Spooling/Buffer bei getrennter Verbindung* wurde nach der Review-Schleife ergänzt, da mobile Notebooks um 02:14 Uhr nicht zwingend im VPN eingeloggt sind.

---

### 1.2 Pipeline-Stationen im Detail

| Station | Was passiert hier | Datenform / Zustand | Beispiel-Tool |
| :--- | :--- | :--- | :--- |
| **1. Ereignis auf dem Endpunkt** | Ein Benutzer oder Skript gibt um 02:14 Uhr am Support-Notebook mehrfach falsche Anmeldedaten ein. Das Betriebssystem verweigert den Zugriff. | Zustand im RAM / Kernel-Subsystem (Auth-Prozess) | Betriebssystem (Windows LSA / Linux PAM) |
| **2. Logeintrag wird geschrieben** | Der Authentifizierungsdienst schreibt das Scheitern in das lokale Logbuch des Systems. | Strukturierter OS-Event (Windows Event-ID 4625) oder Rohtext (`/var/log/auth.log`) | OS-Logging (Windows Event Viewer, `systemd-journald`, `rsyslog`) |
| **3. Agent / Collector liest den Eintrag** | Ein lokaler Hintergrunddienst überwacht die Logdateien fortlaufend, greift neue Einträge sofort ab und puffert sie lokal bei fehlender Verbindung. | Gelesener Log-Datensatz im Arbeitsspeicher/Puffer des Agenten | **Wazuh-Agent**, **Velociraptor**-Client |
| **4. Übermittlung an den Monitoring-Server** | Der Agent sendet den Logeintrag über ein verschlüsseltes Netzwerkprotokoll (TLS) via VPN an die zentrale Monitoring-Instanz. | Verschlüsseltes Datenpaket im Netzwerk (TCP/UDP, z. B. Port 1514) | **Wazuh** (integriertes Agent-Server-Protokoll), Syslog-ng / Filebeat |
| **5. Normalisierung / Parsing** | Der Monitoring-Server empfängt die Rohdaten verschiedener Quellen, zerlegt sie mittels Decodern und bringt sie in ein einheitliches Schema (Felder: `src_ip`, `user`, `status`, `timestamp`). | Strukturierter Datensatz (JSON / ECS - Elastic Common Schema) | **Wazuh-Server** (Decoder-Engine), Logstash |
| **6. Regelauswertung** | Die Regel-Engine gleicht die normalisierten Daten mit hinterlegten Regeln ab. Erkennt sie Muster (z. B. "mehr als 5 Fehlversuche innerhalb von 60s"), schlägt die Korrelationsregel an. | Log-Event angereichert mit Metadaten (Rule-ID, Alert-Level, MITRE ATT&CK Taktik) | **Wazuh-Server** (Ruleset), **Suricata** (für Netzwerk-Traffic) |
| **7. Alarmierung** | Beim Überschreiten eines Schwellenwerts (z. B. Alert Level ≥ 10) wird ein Sicherheitsalarm generiert und Benachrichtigungen werden versendet. | Alarmobjekt (Alert Notification via E-Mail, Teams, Webhook, Dashboard-Event) | **Wazuh-Dashboard**, **Zabbix** (Trigger & Alerting) |
| **8. Reaktion (manuell / automatisiert)** | Es erfolgen Gegenmassnahmen: entweder manuell durch einen Sicherheitsverantwortlichen oder automatisiert durch Skripte. | Befehl / Skriptaufruf (Active Response) oder Incident-Ticket | **Wazuh Active Response** (HIPS), **Fail2Ban** (HIPS) |
| **9. Aufbewahrung / Archivierung** | Die Rohdaten und Alarme werden indexiert, durchsuchbar gespeichert und für spätere Forensik revisionssicher archiviert. | Indexierte Dokumente in NoSQL-Datenbank sowie komprimierte, signierte Archivdateien | **Wazuh-Indexer** (OpenSearch), Elasticsearch, Cold Storage (S3/WORM) |

---

### 1.3 Tool-Abdeckung und HIDS vs. HIPS

#### Tool-Abdeckung
Nicht jedes Tool deckt nur eine Station ab:
- **Wazuh:** Ist eine vollwertige Suite und deckt die **Stationen 3 bis 9** vollständig ab (Agent liest, sendet verschlüsselt, Server parst und korreliert, löst Alarme aus, führt Active Responses aus und der Indexer archiviert).
- **Fail2Ban:** Kombiniert die **Stationen 3, 5, 6 und 8** auf einem einzelnen System (liest lokale Logs, parst Regex, bewertet Fehlversuche und blockiert die Quell-IP via Firewall direkt vor Ort).
- **Suricata:** Überwacht als NIDS/NIPS das Netzwerk und deckt die **Stationen 1–3** (auf Netzwerkebene) sowie **5–8** ab, sendet jedoch typischerweise Logs an ein SIEM wie Wazuh weiter.
- **Zabbix:** Konzentriert sich primär auf Metriken und Schwellenwerte (**Stationen 3, 4, 6, 7**), ist aber kein klassisches Security-Log-SIEM.

#### HIDS vs. HIPS an Station 8
- **HIDS (Host-based Intrusion Detection System):**  
  Arbeitet rein **passiv / beobachtend**. Es generiert bei Station 7 einen Alarm im Dashboard und informiert den Administrator, greift aber **nicht** in das laufende System ein. Würde der Angriff um 02:14 Uhr stattfinden, läuft der Brute-Force-Angriff weiter, bis am Morgen jemand den Alarm sichtet.
- **HIPS (Host-based Intrusion Prevention System):**  
  Arbeitet **aktiv / abwehrend**. Sobald die Regel bei Station 6 anschlägt, schickt die Pipeline bei Station 8 einen Steuerbefehl zurück an den Host (z. B. via *Wazuh Active Response* oder lokal via *Fail2Ban*), der die verdächtige IP in der lokalen Windows-Firewall blockiert oder den Account vorübergehend sperrt. Der Angriff wird um 02:14 Uhr in Echtzeit gestoppt.

---

## Aufgabe 2: Konzeptionelle Risiken bewerten

| Station in der Pipeline | Konzeptionelles Problem | Auswirkung | Bewertung | Gegenmassnahme |
| :--- | :--- | :--- | :--- | :--- |
| **6. Regelauswertung** | **False Positives:** Die Regel schlägt bei harmlosen Ereignissen an (z. B. Mitarbeiter vertippt sich nach Passwortänderung mehrfach). | **Alarmmüdigkeit (Alert Fatigue):** Das Team wird mit Meldungen überflutet, stuft Alarme als bedeutungslos ein und übersieht echte Angriffe. | **Mittel** | Schwellenwerte dynamisch anpassen (z. B. erst ab 10 Fehlversuchen in 2 Min.), Ausnahmen für bekannte interne Netze definieren, Regel-Tuning nach Baseline-Messungen. |
| **6. Regelauswertung** | **False Negatives:** Es existiert keine Regel für den Angriff (z. B. Slow-and-Low-Angriff mit nur 1 Loginversuch alle 30 Minuten). | **Unerkannte Kompromittierung:** Der Angreifer dringt unbemerkt ein, da feste Zeitfenster-Schwellenwerte nicht greifen. | **Kritisch** | Ergänzung statischer Regeln durch Verhaltensanalyse (Anomaly Detection / UEBA), Erkennung ungewöhnlicher Login-Zeiten (02:14 Uhr), Multi-Faktor-Authentifizierung (MFA). |
| **4. Übermittlung / 5–7. Server** | **Ausfall des Monitoring-Servers:** Der zentrale Wazuh-Server stürzt ab oder der Dienst bleibt hängen. | **Blinder Fleck für das gesamte Unternehmen:** Keine Events werden mehr ausgewertet. Ein Ausfall bleibt oft unbemerkt, weil keine Alarme mehr ankommen. | **Hoch** | Externe "Watchdog"-Überwachung (Wer überwacht die Überwachung? z. B. Ping/Heartbeat via Zabbix oder Uptime-Kuma), Agent-seitige Verbindungsalarme bei Ausfall. |
| **2. & 3. Endpunkt** | **Angreifer löscht oder manipuliert Logs auf dem Endpunkt:** Nach erfolgreichem Login löscht der Angreifer Eventlogs (`wevtutil cl` oder Löschen von `/var/log`). | **Verhinderung der Forensik:** Lokale Spuren werden vernichtet; Vorfall kann nicht mehr lückenlos nachvollzogen werden. | **Hoch** | **Near-Realtime-Streaming:** Events werden innerhalb von Millisekunden an den Server übertragen, bevor der Angreifer Root/Admin erlangen kann. Lokale Log-Forwarder als geschützter Dienst ausführen; Alarm auslösen, sobald der Logging-Dienst gestoppt wird. |
| **4. & 5. Übermittlung & Normalisierung** | **Uhren der Systeme laufen nicht synchron:** Support-Notebook geht 5 Minuten vor, Datenbankserver 2 Minuten nach. | **Fehlschlagende Korrelation:** Events über mehrere Systeme lassen sich forensisch nicht chronologisch ordnen; Korrelationsregeln mit Zeitfenstern versagen. | **Hoch** | Zwingende NTP-Zeitsynchronisation (Network Time Protocol) im Active Directory / via DHCP für alle Geräte; Reject/Warning bei stark abweichendem Timestamp im SIEM. |
| **9. Aufbewahrung** | **Datenmenge und Aufbewahrungsdauer:** Logs wachsen unkontrolliert an, Festplatten des SIEM laufen voll. | **Systemstillstand oder Datenverlust:** SIEM stellt Dienst ein; ältere, für Forensik notwendige Logs werden unkoordiniert überschrieben; hohe Speicherkosten. | **Mittel** | Log-Rotation, Tiered Storage (Hot Storage auf schnellen SSDs für 30 Tage, Cold Storage komprimiert/revisionssicher für 6–12 Monate), striktes Filtern irrelevanter Debug-Logs am Agent. |
| **2., 5., 9. Datenhaltung** | **Personenbezogene Daten in den Logs:** Logs enthalten Klarnamen, private IP-Adressen und Bewegungsprofile der Support-Mitarbeitenden. | **Verstoss gegen Datenschutzgesetze (DSG / DSGVO):** Ungerechtfertigte Überwachung der Angestellten; Gefahr bei Datenabfluss des Monitoring-Servers. | **Mittel** | Pseudonymisierung/Hashing von Userdaten im Monitoring, striktes Rollen- und Berechtigungskonzept (RBAC) für den Zugriff auf Log-Server, klare Betriebsvereinbarung. |
| **3. & 4. Endpunkt / Transport (Eigener Punkt 1)** | **Mobiler Client offline / VPN getrennt:** Support-Notebook ist nachts offline oder ohne VPN im Hotel-WLAN; Angreifer probiert Logins lokal aus. | **Verzögerte oder verlorene Alarmierung:** Der Alarm trifft erst Stunden später ein (z. B. 08:30 Uhr bei Arbeitsbeginn), wenn das Gerät sich wieder verbindet. | **Hoch** | Lokales Disk-Spooling (Agent speichert Events persistent auf der Festplatte zwischen und überträgt sie priorisiert beim nächsten Verbindungsaufbau); lokale Schutzmassnahme (HIPS) direkt auf dem Client aktiv halten. |
| **8. Reaktion (Eigener Punkt 2)** | **Fehlkonfigurierte Active Response (Self-Denial-of-Service):** Automatisierte Sperrung blockiert legitime Konten oder Gateways bei Fehlalarmen. | **Betriebsunterbrechung:** Wichtige Support-Mitarbeiter oder interne Kommunikationswege (z. B. VPN-Gateway-IP) werden ausgesperrt. | **Hoch** | Whitelisting für kritische Infrastruktur-IPs und Admin-Accounts; automatische Reaktionen stufenweise gestalten (z. B. temporäre Sperre für 15 Min. statt dauerhafte Kontosperre im AD); Human-in-the-Loop bei kritischen Aktionen. |

---

## Aufgabe 3: Diskussion und Bewertung

### 1. Warum führt "mehr loggen" nicht automatisch zu "mehr Sicherheit"?
1. **Verstärkung der Alarmmüdigkeit (Signal-to-Noise Ratio):**  
   Wenn jedes minimale Ereignis protokolliert und alarmiert wird, ertrinkt das Sicherheitsteam in tausenden irrelevanten Meldungen. Echte, kritische Angriffe gehen im Grundrauschen unter und werden ignoriert oder übersehen.
2. **Erhöhte Angriffsfläche und Ressourcenprobleme:**  
   Umfangreiches Logging führt zu vollen Festplatten und hoher CPU-Last auf Endpunkten und Servern. Zudem werden Logdateien selbst zum lohnenden Angriffsziel: Sie enthalten oft sensible Informationen (Authentifizierungs-Tokens, interne Pfade, Benutzerdaten), die bei unzureichendem Schutz einem Angreifer wertvolle Spionage-Informationen liefern.

### 2. Welche zwei Stationen der Pipeline sind Single Points of Failure (SPoF)? Wie lassen sie sich entschärfen?
1. **Station 3 (Der lokale Agent auf dem Endpunkt):**  
   - *Problem:* Wird der Agent beendet, stürzt er ab oder blockiert eine lokale Firewall den Agenten, ist das Gerät für das Monitoring komplett unsichtbar.  
   - *Entschärfung:* Einsatz eines Watchdog-Dienstes, der den Agenten überwacht und bei Absturz neu startet; Berechtigungen so absichern, dass selbst lokale Benutzer den Dienst nicht beenden können; Server-seitiges "Heartbeat-Monitoring", das sofort Alarm schlägt, wenn sich ein Agent länger als z. B. 10 Minuten nicht gemeldet hat.
2. **Station 5 & 6 (Der zentrale Monitoring-Server / Manager):**  
   - *Problem:* Fällt der zentrale Wazuh-Server aus, können keine Logs mehr empfangen, normalisiert oder auf Regeln geprüft werden. Die gesamte Erkennungsfähigkeit des Betriebs bricht zusammen.  
   - *Entschärfung:* Aufbau eines hochverfügbaren Clusters (Multi-Node Wazuh Manager mit Load Balancer davor); Pufferung auf den Agents bei Serverausfall; externes Monitoring (z. B. Zabbix oder ein Pingdom-Heartbeat), das die Verfügbarkeit des SIEM überwacht.

### 3. Wie beeinflusst die Empfindlichkeit einer Regel das Verhältnis von False Positives zu False Negatives? Warum lassen sich nicht beide auf null bringen?
- **Einfluss der Empfindlichkeit:**  
  - Stellt man eine Regel **sehr empfindlich** ein (z. B. Alarm bereits nach 2 Fehlversuchen innerhalb von 5 Minuten), sinkt die Zahl der übersehenen Angriffe (**niedrige False Negatives**), aber die Zahl der Fehlalarme steigt drastisch an (**hohe False Positives**), weil auch legitime Tippfehler von Mitarbeitern Alarme auslösen.
  - Stellt man die Regel **sehr unempfindlich** ein (z. B. erst ab 20 Fehlversuchen in 1 Minute), gibt es kaum Fehlalarme (**niedrige False Positives**), aber langsame Angreifer (Slow Brute Force / Password Spraying) schlüpfen unbemerkt durch (**hohe False Negatives**).
- **Warum nicht beide null sein können:**  
  Bösartiges Angreiferverhalten und legitimes menschliches Fehlverhalten überschneiden sich technisch im Logfile vollständig. Ein Fehl-Login sieht im Event Log exakt gleich aus, egal ob ein Mitarbeiter sein neues Passwort vergessen hat oder ein Angreifer ein Passwort erraten will. Ohne allwissenden Kontext lässt sich dieser Grenzbereich mathematisch nicht fehlerfrei trennen.

### 4. Antwort an die Geschäftsleitung: "Wir haben jetzt ein Monitoring – sind wir damit sicher?"
> "Ein Monitoring ist wie eine Alarmanlage: Es verhindert den Einbruch nicht selbst, sondern schlägt lediglich Alarm, wenn verdächtige Bewegungen erkannt werden. Echte Sicherheit entsteht erst dadurch, dass unsere Systeme im Vorfeld gehärtet sind und unser Team im Ernstfall schnell und richtig auf die Alarme reagiert. Zudem bleiben Angriffe mit bereits kompromittierten, gültigen Zugangsdaten oder bisher unbekannte Sicherheitslücken für rein regelbasierte Monitore oft unsichtbar."

### 5. KI-Review-Schleife: Ergänzungen und verworfene Vorschläge
- **Was die KI sinnvoll ergänzt hat:**  
  Die KI wies darauf hin, dass bei mobilen Clients (Notebook des Support-Mitarbeiters) um 02:14 Uhr keine permanente VPN-Verbindung garantiert ist. Sie schlug vor, an Station 3 zwingend ein **lokales Disk-Spooling (persistente Zwischenspeicherung)** zu berücksichtigen, damit Events bei getrennter Verbindung nicht im RAM verloren gehen, sondern beim nächsten Verbindungsaufbau gebündelt übertragen werden. Dies habe ich in die Skizze und die Tabelle aufgenommen.
- **Welchen Vorschlag ich verworfen habe und warum:**  
  Die KI schlug vor, bei Station 8 eine Active Response zu hinterlegen, die bei 5 Fehlversuchen das Benutzerkonto des Support-Mitarbeiters sofort **unternehmensweit im Active Directory sperrt**.  
  *Begründung für die Ablehnung:* Dieser Vorschlag ist in der Praxis hochgradig gefährlich (Denial-of-Service gegen den eigenen Support). Wenn ein Support-Mitarbeiter im Notfalleinsatz ist oder ein Angreifer gezielt die bekannten Benutzernamen des Supports mit falschen Passwörtern bombardiert, würde das System den gesamten Support lahmlegen. Die richtige Reaktion ist das temporäre Blockieren der auslösenden Remote-IP oder das Anfordern eines zweiten Faktors (MFA), nicht die sofortige globale Sperrung des Benutzerkontos.

---

## Reflektion

- **Bedeutung der Log-Pipeline im Sicherheitskonzept:**  
  Die Übung verdeutlicht, dass Monitoring kein einzelnes Programm ist, sondern eine zusammenhängende Kette. Bricht ein einziges Glied (z. B. ungenaue Uhren, voller Speicher, Agent-Absturz oder fehlende Normalisierung), verpufft der gesamte Sicherheitsnutzen.
- **HIDS vs. HIPS in der Praxis:**  
  Während ein HIDS wertvolle Transparenz für die Forensik liefert, reicht es für Angriffe ausserhalb der Bürozeiten (wie im Szenario um 02:14 Uhr) oft nicht aus, da niemand vor dem Dashboard sitzt. HIPS-Mechanismen (z. B. Wazuh Active Response) sind notwendig, um die erste Angriffswelle automatisch zu bremsen, müssen aber mit Bedacht konfiguriert werden, um keine Selbstblockaden zu erzeugen.
- **Grenzen von Kennzahlen:**  
  Die Annahme "viel hilft viel" ist im Logging gefährlich. Ein fokussiertes, gehärtetes SIEM mit 20 gut abgestimmten Regeln ist in einem KMU mit 50 Mitarbeitenden deutlich wirksamer als Millionen unkuratierte Logzeilen, die niemand auswerten kann.
