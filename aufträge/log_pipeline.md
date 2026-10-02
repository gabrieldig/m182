# Analyseauftrag: Die Log-Pipeline

[TOC]

> **Kompetenz:** E1A · **Bloom:** Analysieren · **Dauer:** ~90 Min. · **Sozialform:** Einzel- oder Gruppenauftrag (2-3 Personen)

---

## Ausgangslage
Sie arbeiten als IT-Sicherheitsbeauftragte bzw. -beauftragter in einem mittelständischen Unternehmen mit rund 50 Mitarbeitenden. Das Unternehmen betreibt einen öffentlich erreichbaren Webserver mit einem Onlineshop, einen internen Datenbankserver mit sensiblen Kundendaten sowie rund 50 Clients - darunter auch Notebooks, die mobil und via VPN im internen Netz eingesetzt werden.

Die Geschäftsleitung hat beschlossen, dass alle diese Systeme ihre Meldungen künftig an einen zentralen Monitoring-Server schicken sollen. Bevor investiert wird, möchte sie von Ihnen wissen: **Wie funktioniert so eine Log-Pipeline überhaupt - und wo kann sie versagen?**

Voraussetzung für diesen Auftrag ist, dass Sie sich mit der Theorie zum [Monitoring](../inputs/monitoring.md) auseinandergesetzt haben. Der Auftrag ist bewusst konzeptionell gehalten: Sie müssen dafür **nichts installieren**. Wenn Sie den Auftrag [Monitoring mit wazuh](./monitoring_mit_wazuh.md) bereits bearbeitet haben, können Sie Ihr eigenes Setup als Anschauungsbeispiel verwenden - zwingend ist das aber nicht.

---

## Ziel
Sie sollen den Weg einer Meldung von ihrer Entstehung bis zur Reaktion vollständig aufzeigen können und verstehen, dass eine Log-Pipeline nicht erst dann versagt, wenn ein Gerät ausfällt - sondern bereits durch konzeptionelle Schwächen im Aufbau. Genau diese Schwächen entscheiden im Ernstfall darüber, ob ein Angriff bemerkt wird oder nicht.

---

## Aufgabe 1: Die Pipeline aufzeigen

Auf dem Notebook eines Support-Mitarbeitenden versucht jemand um 02:14 Uhr, sich mehrfach mit falschen Zugangsdaten anzumelden.

Zeigen Sie auf, welchen Weg diese Information nimmt, bis jemand darauf reagiert. Erstellen Sie dazu **eine Skizze** der Pipeline (Zeichnung oder Textdarstellung mit Pfeilen genügt) und füllen Sie anschliessend die folgende Tabelle aus:

| Station | Was passiert hier | Beispiel-Tool |
| ------- | ----------------- | ------------- |
| 1. Ereignis auf dem Endpunkt | | |
| 2. Logeintrag wird geschrieben | | |
| 3. Agent / Collector liest den Eintrag | | |
| 4. Übermittlung an den Monitoring-Server | | |
| 5. Normalisierung / Parsing | | |
| 6. Regelauswertung | | |
| 7. Alarmierung | | |
| 8. Reaktion (manuell oder automatisiert) | | |
| 9. Aufbewahrung / Archivierung | | |

**Hinweise:**

* **Was passiert hier:** Beschreiben Sie in ein bis zwei Sätzen, was mit der Meldung an dieser Station geschieht und in welcher Form sie vorliegt (Rohtext? strukturierter Datensatz? Alarm?).
* **Beispiel-Tool:** Verwenden Sie die Tools aus der Theorie [Monitoring](../inputs/monitoring.md) (Wazuh, Fail2Ban, Suricata, Velociraptor, Zabbix). Nicht jede Station braucht ein eigenes Tool - manche Tools decken mehrere Stationen ab. Halten Sie fest, welche.
* Überlegen Sie bei Station 8, wo der Unterschied zwischen einem **HIDS** und einem **HIPS** in dieser Pipeline sichtbar wird.

> **KI-Review-Schleife:**
> 1. Erstellen Sie Skizze und Tabelle **zuerst selbst** - vollständig, nicht als Fragment.
> 2. Lassen Sie beides anschliessend mit dem [Review-Prompt für die Log-Pipeline](../ki-leitfaden.md#reviewer-log-pipeline-auf-lücken-prüfen-lassen) prüfen.
> 3. Halten Sie im KI-Log fest: **was Sie übernommen und was Sie verworfen haben - und warum**. Ein begründet verworfener Vorschlag zählt mehr als ein unbegründet übernommener.
>
> Übernommene Stationen markieren Sie in Ihrer Skizze, damit im Testatgespräch klar ist, was von Ihnen stammt.

---

## Aufgabe 2: Konzeptionelle Risiken bewerten

Eine Log-Pipeline kann technisch einwandfrei laufen und trotzdem ihren Zweck verfehlen. Bewerten Sie die folgenden konzeptionellen Probleme für das oben beschriebene Unternehmen und ergänzen Sie mindestens **zwei weitere**, die Ihnen bei Ihrer Skizze aus Aufgabe 1 aufgefallen sind:

| Station in der Pipeline | Konzeptionelles Problem | Auswirkung | Bewertung (niedrig / mittel / hoch / kritisch) | Gegenmassnahme |
| ----------------------- | ----------------------- | ---------- | ---------------------------------------------- | -------------- |
| | False Positives: die Regel schlägt bei harmlosen Ereignissen an | | | |
| | False Negatives: es existiert keine Regel für den Angriff | | | |
| | Ausfall des Monitoring-Servers | | | |
| | Angreifer löscht oder manipuliert die Logs auf dem Endpunkt | | | |
| | Uhren der Systeme laufen nicht synchron | | | |
| | Datenmenge und Aufbewahrungsdauer | | | |
| | Personenbezogene Daten in den Logs | | | |
| | | | | |
| | | | | |

**Hinweise:**

* **False Positives:** Was passiert mit einem Team, das täglich 200 Alarme erhält, von denen 198 harmlos sind? Der Fachbegriff dafür ist "Alarmmüdigkeit" (alert fatigue).
* **Ausfall des Monitoring-Servers:** Die eigentlich interessante Frage ist hier nicht, dass Daten fehlen - sondern: **Würde der Ausfall überhaupt jemandem auffallen?** Wer überwacht die Überwachung?
* **Manipulation der Logs:** Ein Angreifer mit Administratorrechten auf dem Endpunkt kann dort alles verändern. An welcher Station der Pipeline sind die Daten seinem Zugriff entzogen - und wie schnell gelangen sie dorthin?
* **Zeitsynchronisation:** Ein Angriff läuft über Webserver, Datenbankserver und Notebook. Was passiert bei der Rekonstruktion des Ablaufs, wenn eine Uhr drei Minuten falsch geht?
* **Datenmenge:** Je länger Sie Daten aufbewahren, desto besser die Forensik - und desto höher die Kosten. In der Theorie zu [Fail2Ban](../inputs/monitoring.md#fail2ban-hids-hips) wird dieselbe Problematik anhand der wachsenden Datenbank beschrieben.
* **Personenbezogene Daten:** Logs enthalten Benutzernamen, IP-Adressen und Zugriffszeiten - also Daten, mit denen sich das Verhalten einzelner Mitarbeitender nachvollziehen lässt. Die Grundlagen dazu kennen Sie aus [Modul 231](https://www.modulbaukasten.ch/module/231).
* **Bewertung:** Leiten Sie die Einstufung wie bei der [Risikoanalyse](./auftrag_risikoanalyse.md) aus Eintrittswahrscheinlichkeit und Schweregrad ab.

---

## Aufgabe 3: Diskussion und Bewertung

Beantworten Sie die folgenden Fragen schriftlich:

1. Warum führt "mehr loggen" nicht automatisch zu "mehr Sicherheit"? Nennen Sie zwei Gründe.
2. Welche **zwei Stationen** Ihrer Pipeline aus Aufgabe 1 sind Single Points of Failure? Wie liesse sich das jeweils entschärfen?
3. Wie beeinflusst die Empfindlichkeit einer Regel das Verhältnis von False Positives zu False Negatives? Warum lassen sich nicht beide gleichzeitig auf null bringen?
4. Die Geschäftsleitung fragt Sie: "Wir haben jetzt ein Monitoring - sind wir damit sicher?" Wie antworten Sie in drei Sätzen?
5. Welche Station hat Ihnen die KI in der Review-Schleife ergänzt, und welchen ihrer Vorschläge haben Sie verworfen? Beschreiben Sie, woran Sie den unbrauchbaren Vorschlag erkannt haben.

---

## Abgabe
Sofern die Lehrperson nichts anderes festlegt, sollten Sie Ihre Skizze, die beiden ausgefüllten Tabellen sowie die Antworten auf die Diskussionsfragen in geeigneter Form der Lehrperson abgeben.

---

## Quellenangaben
Dieser Auftrag wurde durch KI generiert und von der Lehrperson inhaltlich überprüft und optimiert. Sollten Teile des Auftrages unklar sein oder Sie Verbesserungsvorschläge haben, melden Sie sich bitte bei der Lehrperson.
