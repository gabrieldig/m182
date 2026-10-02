# Auftrag: KI-Antworten in der Systemsicherheit validieren

[TOC]

> **Kompetenz:** überfachlich (KI-Kompetenz) · **Bloom:** Beurteilen · **Dauer:** ~90 Min. · **Sozialform:** Einzel- oder Gruppenauftrag (2-3 Personen)

> **Pflichtauftrag.** Dieser Auftrag ist Voraussetzung für die Annahme Ihres Portfolios, belegt aber **keinen** der fünf Portfolio-Plätze. Er kostet Sie also keinen Punkt und bringt Ihnen auch keinen - er ist die Eintrittskarte.

---

## Ausgangslage

Sie werden in diesem Modul KI-Werkzeuge einsetzen - das ist ausdrücklich erwünscht und im [KI-Leitfaden](../ki-leitfaden.md) geregelt. In der Systemsicherheit ist eine falsche KI-Antwort aber teurer als anderswo: Wer eine erfundene CVE-Nummer in einen Bericht schreibt, verliert Glaubwürdigkeit. Wer eine generierte Firewall-Regel ungeprüft übernimmt, sperrt im schlimmsten Fall den produktiven Zugriff aus oder lässt ein Loch offen.

Das Problem ist dabei nicht, dass KI-Modelle sich irren. Das Problem ist, dass sie sich **überzeugend** irren: Eine erfundene CVE-Nummer sieht aus wie eine echte, und eine falsche Konfigurationsempfehlung ist syntaktisch fehlerfrei.

In diesem Auftrag üben Sie deshalb genau das, was Sie im ganzen restlichen Modul brauchen werden: **eine KI-Antwort so zu prüfen, dass Sie sie verantworten können.**

---

## Ziel

Sie können Ihre Fragen an eine KI so formulieren, dass die Antworten überprüfbar werden, und Sie können eine sicherheitsrelevante KI-Aussage systematisch gegen Primärquellen validieren. Sie wissen aus eigener Erfahrung, bei welcher Art von Frage Sie besonders misstrauisch sein müssen.

---

## Aufgabe 1: Dieselbe Frage, drei Prompts

Stellen Sie einer KI Ihrer Wahl dreimal dieselbe fachliche Frage - aber unterschiedlich formuliert. Nehmen Sie als Thema die Absicherung eines SSH-Dienstes.

**Variante A - naiv:**
```
Wie sichere ich SSH ab?
```

**Variante B - mit Rolle und Systemkontext:**
```
Du bist Systemadministrator. Ich betreibe einen Ubuntu-24.04-Server,
der aus dem Internet per SSH erreichbar ist und von drei Personen
administriert wird. Welche Massnahmen empfiehlst du, und in welcher
Reihenfolge? Begründe die Reihenfolge.
```

**Variante C - mit Quellenzwang:** den Quellenzwang-Prompt aus dem [KI-Leitfaden](../ki-leitfaden.md#quellenzwang-behauptungen-überprüfbar-machen) verwenden.

Vergleichen Sie die drei Antworten:

| Prompt-Variante | Brauchbarkeit für mein System | Was fehlte | Was war falsch oder unbelegt |
| --------------- | ----------------------------- | ---------- | ---------------------------- |
| A - naiv | | | |
| B - Rolle und Kontext | | | |
| C - Quellenzwang | | | |

**Halten Sie fest:** Welcher Zusatz im Prompt hat die Antwort am stärksten verbessert - und warum?

---

## Aufgabe 2: Halluzinationen jagen

Jetzt provozieren Sie gezielt Fehler.

1. Wählen Sie eine konkrete Software **mit Versionsnummer** - zum Beispiel einen Dienst, den Sie im Auftrag [Enumeration + Fingerprinting](./enumeration_fingerprinting.md) auf der Metasploitable-VM gefunden haben.
2. Fragen Sie die KI nach **mindestens fünf bekannten Schwachstellen** dieser Version, jeweils mit CVE-Nummer und einer kurzen Beschreibung. Verwenden Sie dabei bewusst **keinen** Quellenzwang.
3. Prüfen Sie jede einzelne Angabe gegen die **National Vulnerability Database** (https://nvd.nist.gov/vuln/search) oder https://www.cve.org.

| Behauptung der KI (CVE-Nr. und Kern) | In NVD gefunden? | Betrifft wirklich diese Version? | Bewertung: korrekt / falsch / nicht belegbar |
| ------------------------------------ | ---------------- | -------------------------------- | -------------------------------------------- |
| | | | |

**Achten Sie auf drei Fehlertypen:**
- Die CVE-Nummer existiert gar nicht.
- Die CVE-Nummer existiert, beschreibt aber etwas anderes.
- Die CVE existiert und passt inhaltlich, betrifft aber eine andere Version oder ein anderes Produkt.

**Halten Sie fest:** Wie viele der fünf Aussagen haben die Prüfung bestanden? Und: Hätten Sie den Fehlern ohne Prüfung angesehen, dass sie falsch sind?

---

## Aufgabe 3: Eine generierte Massnahme validieren

Lassen Sie die KI etwas erzeugen, das Sie anschliessend **wirklich ausprobieren** - wählen Sie eine der beiden Varianten:

- **Variante Härtung:** eine konkrete Härtungsmassnahme für Ihr System aus dem [Hardening-Auftrag](./auftrag_hardening_ubuntu_vm.md), inklusive Befehl und Prüfbefehl.
- **Variante Monitoring:** eine wazuh-Regel für ein selbst gewähltes Ereignis, passend zum [wazuh-Auftrag](./monitoring_mit_wazuh.md).

Prüfen Sie das Ergebnis in drei Schritten und dokumentieren Sie jeden:

1. **Gegen die Primärquelle:** Steht das so im CIS Benchmark beziehungsweise in der offiziellen [wazuh-Dokumentation](https://documentation.wazuh.com/)? Oder klingt es nur so?
2. **Im Labor:** Führen Sie die Massnahme in Ihrer VM oder Ihrem Container aus. Funktioniert sie überhaupt? Tut sie das, was behauptet wurde? Weisen Sie das mit dem Prüfbefehl nach.
3. **Im Kopf:** Wo könnte diese Massnahme in einem Produktivsystem Schaden anrichten - Betriebsunterbruch, ausgesperrte Benutzer, Fehlalarme?

| Schritt | Ergebnis | Beleg (Quelle, Befehlsausgabe, Screenshot) |
| ------- | -------- | ------------------------------------------ |
| 1. Primärquelle | | |
| 2. Labortest | | |
| 3. Risiko im Produktivbetrieb | | |

> **Vorsicht:** Testen Sie ausschliesslich in Ihrer Laborumgebung und nie über die Verbindung, mit der Sie selbst auf dem System arbeiten.

---

## Reflexionsfragen

1. Woran haben Sie in Aufgabe 2 eine falsche Antwort erkannt, **ohne die richtige zu kennen**? Beschreiben Sie das Warnsignal.
2. Bei welcher Art von Frage würden Sie einer KI künftig nicht mehr ohne Prüfung glauben - und bei welcher schon? Begründen Sie die Grenze, die Sie ziehen.
3. Was kostet Sie die Validierung an Zeit, und in welchen Fällen lohnt sich der Aufwand trotzdem? Denken Sie dabei an Ihre spätere Rolle im Betrieb.
4. Die KI hat in Aufgabe 3 etwas vorgeschlagen, das Sie übernommen oder verworfen haben. Welche Entscheidung haben Sie getroffen, und was war Ihr Ausschlagargument?

---

## Abgabe
Sofern die Lehrperson nichts anderes festlegt, geben Sie die drei ausgefüllten Tabellen, die verwendeten Prompts im Wortlaut und die Antworten auf die Reflexionsfragen in geeigneter Form ab. Die Prompts gehören zusätzlich in Ihr [KI-Log](../umsetzungsplan.md#lernjournal-vorlage).

---

## Quellenangaben
Dieser Auftrag wurde durch KI generiert und von der Lehrperson inhaltlich überprüft und optimiert. Sollten Teile des Auftrages unklar sein oder Sie Verbesserungsvorschläge haben, melden Sie sich bitte bei der Lehrperson.
