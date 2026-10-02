# KI Validierung (Gemini)

## Aufgabe 1

### Prompts
- **Variante A (Naiv):** `Wie sichere ich SSH ab?`
- **Variante B (Rolle & Systemkontext):** `Du bist Systemadministrator. Ich betreibe einen Ubuntu-24.04-Server, der aus dem Internet per SSH erreichbar ist und von drei Personen administriert wird. Welche Massnahmen empfiehlst du, und in welcher Reihenfolge? Begründe die Reihenfolge.`
- **Variante C (Quellenzwang):** `Du bist Linux-Sicherheitsadministrator. Nenne mir konkrete Härtungsmassnahmen für den OpenSSH-Server unter Ubuntu 24.04. Nenne zu jeder Massnahme die genaue Konfigurationsdirektive für die sshd_config, den empfohlenen Wert sowie die offizielle Primärquelle (z. B. CIS Benchmark oder man sshd_config). Wenn du eine Behauptung nicht belegen kannst, erwähne sie nicht.`

### Vergleich
| Promt Variant | Brauchbarkeit für mein System | Was Fehlte | Was falsch war oder unbelegt |
| :--- | :--- | :--- | :--- |
| **Naiv** | SSH-Key, Root Deaktivierung, Fail2ban | Absicherung bei Firewall-Änderungen (z. B. Hauptsitzung nie schliessen sondern zweites Fenster öffnen), konkrete Pfade | Analogien gemacht, Port-Änderung als echter Schutz dargestellt ohne Belege |
| **Rolle & Systemkontext** | Nie zu zweit ein Konto verwenden, SSH Daemon härten, Port ändern, Reihenfolge mit Begründung | Spezifische Richtlinien zu Host-Keys / Ciphers | - |
| **Quellenzwang** | Sitzungsbegrenzung, inaktive Sitzungen beenden, Verzeichnisberechtigungen, exakte Direktiven | Einige praktische Tipps von vorheriger Antwort (nur noch harte Fakten) | Keine Falschaussagen, da unbelegte Punkte weggelassen wurden |

**Fazit:** Der Zusatz von **Rolle und Systemkontext (Variante B)** hat die Antwort am stärksten verbessert, weil die Massnahmen direkt auf die Situation (mehrere Admins, Ubuntu) angepasst und in eine sinnvolle Reihenfolge gebracht wurden (zuerst neuen User mit Sudo anlegen, bevor Root deaktiviert wird). Der **Quellenzwang (Variante C)** hat zusätzlich geholfen, genau die richtigen Direktiven für die Konfigurationsdatei zu erhalten.

---

## Aufgabe 2
-> NGINX 1.18.0

**Prompt:** `Nenne mir mindestens 5 bekannte Schwachstellen von NGINX Version 1.18.0, jeweils mit CVE-Nummer und einer kurzen Beschreibung.` (ohne Quellenzwang)

| Behauptung der KI (CVE-Nr. und Kern) | In NVD gefunden? | Betrifft wirklich diese Version? | Bewertung: korrekt / falsch / nicht belegbar |
| :--- | :--- | :--- | :--- |
| **CVE-2021-23017** (1-Byte Heap Overflow im DNS-Resolver) | CVE.org / NVD gefunden | Ja (betrifft 0.6.18 bis 1.20.0) | **korrekt** (setzt aber die Direktive `resolver` voraus) |
| **CVE-2019-20372** (HTTP Request Smuggling via error_page) | CVE.org / NVD gefunden | Nein, sondern < 1.17.7 (in 1.17.7 gepatcht) | **falsch** (jedoch richtig beschrieben) |
| **CVE-2022-41741** (Memory Corruption im MP4-Modul) | CVE.org / NVD gefunden | Ja (betrifft Versionen vor 1.22.1 / 1.23.2) | **nicht korrekt beschrieben** (trifft nur zu, wenn ngx_http_mp4_module genutzt wird) |
| **CVE-2022-41742** (Memory Disclosure im MP4-Modul) | CVE.org / NVD gefunden | Ja (betrifft Versionen vor 1.22.1 / 1.23.2) | **nicht korrekt beschrieben** (trifft ebenfalls nur mit MP4-Modul zu) |
| **CVE-2018-16843** (DoS durch Speicherverbrauch bei HTTP/2) | CVE.org / NVD gefunden | Nein, sondern < 1.15.6 (in 1.15.6 behoben) | **falsch** (betrifft 1.18.0 nicht) |

**Fazit:**
- **Bestanden:** Nur 2 von 5 Aussagen trafen wirklich auf Version 1.18.0 zu, und auch dort fehlte die Einschränkung, dass das MP4-Modul überhaupt aktiv sein muss. 2 CVEs waren für diese Version schlichtweg falsch, da sie schon in älteren Versionen gepatcht wurden.
- **Erkennbarkeit:** Ohne die Gegenprüfung in NVD hätte man den Fehlern **nicht angesehen**, dass sie falsch sind. Die CVE-Nummern existieren alle und die Beschreibungen klangen technisch überzeugend. Die KI kann Versionsgrenzen nicht zuverlässig unterscheiden.

---

## Aufgabe 3

Gewählte Methode: **Härtung** (basierend auf [Hardening](hardening.md) – Massnahme: SSH-Root-Login deaktivieren).

### KI-Generierung
**Prompt:** `Gib mir den genauen Linux-Befehl, um auf einem Ubuntu-Server den SSH-Root-Login zu deaktivieren, und den Prüfbefehl, um das zu überprüfen.`

**KI-Befehle:**
```bash
sudo sed -i 's/^#\?PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart ssh
```
**KI-Prüfbefehl:**
```bash
sshd -T | grep -i permitrootlogin
```

### Validierung
| Schritt | Ergebnis | Beleg (Quelle, Befehlsausgabe, Screenshot) |
| :--- | :--- | :--- |
| **1. Primärquelle** | Steht genau so im **CIS Ubuntu Benchmark (5.2.10)** und in `man sshd_config`. Allerdings empfiehlt CIS, vor dem Neustart die Syntax mit `sshd -t` zu prüfen und Drop-in-Dateien in `/etc/ssh/sshd_config.d/` zu beachten. | CIS Benchmark 5.2.10: `Ensure SSH root login is disabled`<br>Sollwert: `PermitRootLogin no` |
| **2. Labortest** | Massnahme im Container/VM getestet. Der Prüfbefehl bestätigt die Einstellung. Der Login als Root wird abgelehnt, der normale User kommt weiterhin rein. | `$ sshd -T \| grep -i permitrootlogin`<br>`permitrootlogin no`<br><br>`$ ssh root@localhost`<br>`root@localhost: Permission denied (publickey,password).` |
| **3. Risiko im Produktivbetrieb** | Sehr hohes Risiko ausgesperrt zu werden, wenn noch kein anderer Benutzer mit `sudo`-Rechten existiert. Zudem kann `systemctl restart ssh` bei einem Tippfehler den ganzen SSH-Dienst abschiessen. | **Gegenmassnahmen:**<br>- Vorher einen Sudo-User erstellen und testen<br>- Konfiguration mit `sshd -t` prüfen<br>- `reload` statt `restart` nutzen<br>- Zweites Terminal offen lassen |

---

## Reflektion

- **Warnsignal bei falschen Antworten:**
  Ein wichtiges Warnsignal war die Jahreszahl der CVEs im Vergleich zum Release der Software. NGINX 1.18.0 ist 2020 herausgekommen, die KI hat aber Lücken aus 2018 und 2019 vorgeschlagen, ohne genaue Versionsbereiche zu nennen. Ausserdem formuliert die KI oft sehr pauschal ("NGINX ist verwundbar"), obwohl die Schwachstelle nur bei ganz bestimmten Modulen oder Direktiven auftritt.

- **Grenzen beim Vertrauen in die KI:**
  Ich würde einer KI bei konkreten CVE-Nummern, Versionsständen und sicherheitsrelevanten Konfigurationen (wie Firewall-Regeln, `/etc/sudoers` oder SSH-Einstellungen) **nie** ungeprüft glauben. Ein kleiner Fehler kann dazu führen, dass man sich aussperrt oder ein Loch im System offen bleibt. 
  Vertrauen würde ich der KI eher bei einfachen Aufgaben wie dem Formatieren von Befehlsausgaben in Tabellen, Formulierungshilfen oder als Inspiration für Befehle, die ich danach selber prüfe.

- **Aufwand und Nutzen im Betrieb:**
  Das Nachschlagen in der NVD und in den CIS Benchmarks hat mich ca. 20 Minuten gekostet. Im Betrieb lohnt sich dieser Aufwand aber auf jeden Fall: Wenn ein Server durch eine ungeprüfte Firewall-Regel ausfällt oder ein Admin ausgesperrt wird, steht der Betrieb still und die Behebung dauert viel länger und kostet mehr Geld.

- **Entscheidung zu den Vorschlägen der KI:**
  Die Massnahme `PermitRootLogin no` und den Prüfbefehl `sshd -T` habe ich übernommen, weil beides exakt mit dem CIS Benchmark übereinstimmt. 
  Verworfen habe ich aber den Befehl `systemctl restart ssh` und das blinde Überschreiben mit `sed`. Wenn man einen Syntaxfehler in der `sshd_config` hat, stoppt der SSH-Dienst beim `restart` und startet nicht mehr neu. Stattdessen habe ich zuerst mit `sshd -t` geprüft, mit `systemctl reload ssh` neu geladen und die bestehende SSH-Sitzung offen gelassen, bis der Login mit dem neuen Benutzer erfolgreich getestet war.
