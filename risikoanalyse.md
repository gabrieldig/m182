| System                          | Bedrohung                                                                                             | Schwachstelle                                                                              | Eintrittswahrscheinlichkeit | Schweregrad | Schutzziele (C/I/A) | Risikobewertung |
| :------------------------------ | :---------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------- | :-------------------------- | :---------- | :------------------ | :-------------- |
| Webserver| Unbefugter Zugriff durch Ausnutzung einer Web-Schwachstelle| Veraltete CMS-/Shop-Software, fehlendes Patch-Management, unzureichende Input-Validierung  | häufig | hoch | C, I, A| kritisch|
| Datenbankserver| Datendiebstahl / unbefugter Zugriff auf Kundendaten| Fehlende Verschlüsselung der Daten (at rest), zu weit gefasste Zugriffsrechte| gelegentlich| kritisch    | C| kritisch|
| Notebook Support-Mitarbeitender | Diebstahl/Verlust des Geräts mit gespeicherten Zugangsdaten| Keine oder schwache Festplattenverschlüsselung, gespeicherte VPN-Credentials| gelegentlich| hoch| C, I| hoch|


--- 
| System                              | Identifiziertes Risiko                     | Schutzmassnahme                                      | Erwartete Wirkung (z. B. Reduktion Eintrittswahrscheinlichkeit oder Schweregrad) |
| ----------------------------------- | ------------------------------------------ | ---------------------------------------------------- | -------------------------------------------------------------------------------- |
| Datenbank|Datendiebstahl durch unbefugten Zugriff|Verschlüsselung der Daten, umstztung von Least-Privillege-Zugriffskontrolle| Reduktion der Eintrittswahrscheinlichkeit und Schweregrad durch Verschlüsselung und eingeschränkte Zugriffsrechte|
| Notebook eines Support-Mitarbeiters |Malwareinfektion bereitet sich durch VPN|Netzwerksegmentierung und Endpoint-Detection & Response Lösung und Zero Trust-Ansatz | Reduktion der Eintrittswarscheinlichkeit und Schweregrad durch frühzeitige Erkennung und Behebung von Schwachstellen |

## Aufgabe 3
- **Handlungsbedarf**: Der Notebook sollte die erste Priorität haben, da der Verlust oder Diebstahl eines solchen Gerätes zu einem **direkten** **Zugriff** auf die anderen **Systeme** führen kann, da jemand von diesem Laptop **ohne** die Notwendigkeit von weiteren **Authentifizierungen** auf die Systeme zugreifen kann.
- Die Risikobewertung sinkt, da der **Angrifspfad** durch die Umsetzung der Schutzmassnahme *abgesperrt* wird. Jedoch bleibt die **Bedrohung** immernoch **vorhanden**. Da die Bedrohung etwas **Externes** ist, was nicht "**gepatcht**" werden kann, und weiterhin besteht. Es wird immer **jemand** geben, der *versucht*, auf die Systeme **zuzugreifen**. 
- Da etwas nur so stark ist, wie die schwächste Stelle. Auch wenn die Systeme gut Geschützt sind, kann ein ungeschulter Mitarbeiter, der ein Gerät verliert oder gestohlen wird, die gesamte Sicherheitsarchitektur gefährden. Daher sind SChulungen und Sensibilisierung der Mitarbeiter ein wichtiger Bestandteil der Sicherheitsstrategie.
