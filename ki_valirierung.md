# KI Validierung (Gemini)

## AUfgabe 2
| Promt Variant | Brauchbarkeit für mein System        | Was Fehlte | Was falsch war oder unbelegt |
| -------------- | ------------------------------------ | ---------- | ---------------------------- |
| Naiv           | SSh-Key, Root deavtication, fail2ban | Absicherung bei firewallchanges(z.b. Hauptsitzung nie schliessen sondern zweites Fenster öffnen) | Analogieen Gemacht,n|
|Rolle & Systemkontext|nie zu zweit ein Konto verwenden, ssh Daemon härten, port ändern|-|-|
| Quellenzwang | Sinzungsbegrenzung, inaktive Siztzungen beenden, verzeichnsssberechtigungen | einige Tips von vorheriger Antwort|

# Aufgabe 2
-> NGINX 1.18.0 
| Behauptung der KI (CVE-Nr. und Kern) | In NVD gefunden? | Betrifft wirklich diese Version? | Bewertung: korrekt / falsch / nicht belegbar |
| ------------------------------------ | ---------------- | -------------------------------- | -------------------------------------------- |
| CVE-2021-23017                       | CVE.org gefunden | Yes                              | korrekt                                      |
| CVE-2019-20372                       | CVE.org gefunden | Nein, sondern < 17.7.0           | Jedoch richtig beschrieben                   |
| CVE-2022-41741                       | CVE.org gefunden | Yes                              | nicht korrekt beschrieben                    |
| CVE-2022-41742                       | CVE.org gefunden | Yes                              | nicht korrekt beschrieben                    |
| CVE-2018-16843                       | CVE.org gefunden | Nein, sondern < 1.15.6           | korrekt                                      |