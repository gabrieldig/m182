# Lernjournal

# Aufträge
| Auftrag                   | Link                                                    | Datum      | Abgabeform | KI genutzt (ja/nein) | Testat LP  |
| ------------------------- | ------------------------------------------------------- | ---------- | ---------- | -------------------- | ---------- |
| Risikoanalyse             | [Risikoanalyse](risikoanalyse.md)                       | 28.09.2026 | Vorzeigen  | nein                 | 04.09.2026 |
| Enumeration & Fingerprint | [Enumeration & Fingerprint](enumeration_fingerprint.md) | 28.09.2026 | Vorzeigen  | nein                 | 04.09.2026 |
| Hardening                 | [Hardening](hardening.md)                               | 28.09.2026 | Vorzeigen  | nein                 |            |
| Honeypot                  | [Honeypot](honeypot.md)                                 | 02.10.2026 | Vorzeigen  | ja                   |            |
| KI Validierung            | [KI Validierung](ki_validierung.md)                     | 02.10.2026 | Vorzeigen  | ja                   |            |
| Log-Pipeline              | [Log-Pipeline](log_pipeline.md)                         | 02.10.2026 | Vorzeigen  | ja                   |            |

# KI Nutzung

| Promt                                                                                        | Usefullness | Used Part                                                                   | Unused Part                                                                    |
| -------------------------------------------------------------------------------------------- | ----------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Transform CMD output into CSV Table                                                          | 5/5         | Aus unformatiertem Text wurde eine lesbare Tabelle erstellt worden          | -                                                                              |
| root:x:0:0:root:/root:/bin/bash, How do I format thsis                                       | 5/5         | Die Usertabelle wurde in eine Sinnvolle Datebelle verwandelt.               | -                                                                              |
| Nenne mir 5 Vulnerabilities von NGINX 1.18.0, mit jeweils der CVE und einem kurzen Beschreib | 2/5         | CVE-2021-23017 war korrekt; die restlichen waren veraltet/ungenau           | Falsche Versionen (< 1.17.7, < 1.15.6) verworfen                               |
| SSH Root-Login deaktivieren und Prüfbefehl für Ubuntu generieren                             | 4/5         | Parameter `PermitRootLogin no` und Prüfbefehl `sshd -T` übernommen          | Befehl `systemctl restart ssh` verworfen (durch reload ersetzt)                |
| Überprüfe meine Log-Pipeline auf Lücken und konzeptionelle Risiken bei mobilen Endgeräten    | 4/5         | Lokales Puffer/Disk-Spooling für mobile Clients und SPoF-Analyse übernommen | Vorschlag zur sofortigen globalen AD-Kontosperrung verworfen (Self-DoS-Gefahr) |

