# Monitoring der Endpunkte mit wazuh

[TOC]

> **Kompetenz:** D1B–D1A · **Bloom:** Anwenden bis Beurteilen · **Dauer:** ~120–180 Min. (je nach Stufe) · **Sozialform:** Einzel- oder Gruppenauftrag (2-3 Personen)

---

## Ausgangslage
Voraussetzung für die effiziente Erledigung dieses Auftrags ist, dass Sie sich mit der Theorie zum [Monitoring](../inputs/monitoring.md) auseinandergesetzt haben. Dieser Auftrag soll das Wissen aus der Theorie über die praktische Anwendung vertiefen.

---

## Ziel
Der Auftrag soll dazu führen, dass Sie erste praktische Erfahrungen mit der Überwachung von Endpunkten sammeln. Die Idee ist, dass Sie Ihr eigenes Arbeitsgerät überwachen und quasi als Bonus Ihr eigenes System mit der Hilfe von wazuh härten können (für Details zur Systemhärtung siehe Theorie [hardening](../inputs/hardening.md)).

> Hinweis: Wenn Sie den wazuh-Agent aus irgendwelchen Gründen nicht auf Ihrem Gerät installieren möchten bzw. nicht Ihr Gerät härten möchten, können Sie alternativ auch eine VM mit einem Betriebssystem Ihrer Wahl (beispielsweise Ubuntu) mit wazuh verbinden und härten.

---

## Stufen dieses Auftrags

Dieser Auftrag lässt sich auf drei Stufen des Kompetenzbands **D – HIDS in Betrieb nehmen** lösen. Die Stufen bauen **aufeinander auf**: Wer auf Stufe I arbeitet, hat Stufe B bereits erledigt; wer auf Stufe A arbeitet, hat B und I erledigt.

Entscheiden Sie vor dem Start, welche Stufe Sie anstreben - und halten Sie das in Ihrer Dokumentation fest.

### Stufe B (D1B) - «ein HIDS in Betrieb nehmen»
*Aufwand: ca. 120 Minuten*

[Installation von wazuh](#installation-von-wazuh) und [Testen von wazuh](#testen-von-wazuh): Die zentralen Komponenten laufen, Ihr Agent ist verbunden und im Dashboard sichtbar. Sie haben sich durch die Kacheln geklickt und dokumentiert, welche Informationen über Ihr Gerät gesammelt werden.

### Stufe I (D1I) - «ein geeignetes HIDS auswählen und auf eine bestimmte Systemsituation konfigurieren»
*Aufwand: zusätzlich ca. 45 Minuten*

Zusätzlich zu Stufe B: [Härten von eigenem System](#härten-von-eigenem-system) und [wazuh auf Ihre Systemsituation konfigurieren](#wazuh-auf-ihre-systemsituation-konfigurieren).

### Stufe A (D1A) - «Grenzen beurteilen und mit zusätzlichen Massnahmen ergänzen»
*Aufwand: zusätzlich ca. 30 Minuten*

Zusätzlich zu Stufe I: [Grenzen von wazuh beurteilen](#grenzen-von-wazuh-beurteilen) und die [Reflexionsfragen](#reflexionsfragen) am Schluss. Wenn Sie den Auftrag auf dieser Stufe abschliessen wollen, überspringen Sie diese beiden Teile also nicht.

---

## Vorgehen

### Installation von wazuh
#### Installation von Requirements
Die zentralen Teile von Wazuh lässt sich entweder auf einer VM oder via Docker betreiben. Wird der Weg über eine VM gewählt, brauchen Sie eine Virtualisierungsumgebung wie VirtualBox, Hyper-V oder VMWare. Wird Docker eingesetzt, wird docker desktop benötigt (und gegebenenfalls auch docker compose - sofern dies noch nicht installiert ist bzw. nicht als Abhängigkeit bei Docker Desktop mit installiert würde).

#### Installation von wazuh Core
Das folgende Bild zeigt die Architektur von wazuh:
![architektur wazuh cluster](../_resources/images/deployment-architecture_wazuh.png)
_Quelle Bild & Details: https://documentation.wazuh.com/current/getting-started/architecture.html_

Beim Singlenode-Cluster wie auch beim Multinode-Cluster besteht die Zentrale Einheit aus dem wazuh-Server, -Indexer und -Dashboard. Das wazuh-Dashboard ist dabei das Userinterface, was die Daten für die Systemadministratoren aufbereitet und anzeigt. Der wazuh-Indexer enthält die Daten, die vom wazuh-Server gesammelt und aufbereitet werden. Die wazuh-Agents auf den Endpunkten sammeln die Daten und übermitteln diese verschlüsselt an den wazuh-Server.

Wie wazuh installiert werden muss, ist unter https://documentation.wazuh.com/current/installation-guide/index.html beschrieben. Für diesen Auftrag reicht es jedoch eine der Installationsalternativen zu verwenden:
- **Als virtuelle Maschine:** wazuh bietet vorinstallierte VMs im OVA-Format zur Verfügung, die einfach in die gängigen Virtualisierungsumgebungen importiert werden können. Die Anleitung finden Sie unter https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html.
- **Als Container unter docker oder Kubernetes:** Für ein einfaches deployment unter docker verwendet wazuh docker compose, um die drei container (Server, Indexer und Dashboard) einfach zu installieren (Siehe Anleitung zu Docker unter https://documentation.wazuh.com/current/deployment-options/docker/index.html). Die Anleitung für ein deployment auf Kubernetes ist unter https://documentation.wazuh.com/current/deployment-options/deploying-with-kubernetes/index.html zu finden.

Die Installationsanleitungen unterscheiden in der Regel zwischen Singlenode- und Multinode-Cluster. Ein Singlenode-Cluster reicht für unsere Zwecke vollkommen.

#### Installation von wazuh Agent
Der Agent ist das Stück Software, welches den Endpunkt überwacht und im Extremfall - beim Feststellen von verdächtigen Aktivitäten - Scripts aufrufen kann, die dann den Endpunkt isolieren, damit der Angreifer von einem infizierten Endpunkt aus nicht weitere Endpunkte angreifen kann.

> **Wichtig:** 
> 
> der Agent muss die gleiche Version oder eine ältere Version wie die zentralen Komponenten haben, um sich korrekt mit dem Server zu verbinden. Wenn also beispielsweise der Server noch alt ist und neue Clients hinzugefügt werden mit der aktuellsten Agent-Version, dann wird das so nicht gehen. Erst müsste dann ein Update auf dem Server installiert werden und anschliessend die Clients aktualisiert werden. 
> 
> Das Update der Clients ist dabei aber nicht ganz so wichtig, weil ein neuerer Core mit älteren Agents umgehen kann (ist rückwärtskompatibel). Im Sinne von einer guten Security sollten die Agents aber so oder so auch aktualisiert werden - dies kann aber zeitversetzt geschehen und kann vom wazuh-Server aus angestossen werden.

Der Agent benötigt bei der Installation (oder via Config-File) lediglich die IP oder den Domainnamen des Servers. Er verbindet sich anschliessend automatisch mit dem Server und fordert einen neuen Schlüssel an für die Verschlüsselung der Datenverbindung zum Server. Der Agent wird als Dienst installiert und läuft in der Regel im Hintergrund und loggt Informationen über verdächtige Aktivitäten aber auch über die installierte Software und deren Versionen. Wenn über das Dashboard ein neuer Agent hinzugefügt wird, dann wird im Dashboard die Anleitung gezeigt, wie auf dem entsprechenden System der Agent installiert werden muss (inkl. CLI-Befehle).

### Testen von wazuh
Loggen Sie sich ins wazuh-Dashboard ein (Standardzugangsdaten sind: Benutzername = admin, Passwort = SecretPassword). Falls Sie das System produktiv nutzen möchten, sollten Sie spätestens jetzt die Default-Zugänge ändern. Für unsere Labor-Umgebung ist das aber nicht unbedingt erforderlich.

Im wazuh-Dashboard sollten Sie nun Ihren Agenten sehen, den Sie auf Ihrem Lokalen Rechner installiert haben. Falls nicht, stellen Sie sicher, dass Ihr Rechner auf die IP zugreifen kann, die Sie für die zentralen Komponenten bei der Installation vergeben haben bzw. die zugewiesen wurde. Wenn Sie Zugriff auf die IP haben, aber die Verbindung dennoch nicht funktioniert, kann es sein, dass Firewalls zwischen Ihrem Rechner und den Zentralen Komponenten die TCP-Ports 1514 und 1515 blockieren, die für die Kommunikation zwischen Server und wazuh-Agent verwendet werden. Eine komplette Liste der Ports (inkl. weiterer Details) finden Sie unter https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/requirements.html.

Ist Ihr Agent korrekt verbunden, dann klicken Sie sich durch das wazuh-Dashboard und analysieren Sie, welche Informationen über Ihr Gerät gesammelt wurden und welche Empfehlungen abgegeben werden seitens wazuh. Interessant für diesen Übungsauftrag sind vor allem die Kacheln in **ENDPOINT SECURITY** und **THREAT INTELLIGENCE**.

![Wazuh Dashboard](../_resources/images/wazuh_dashboard.png)

### Härten von eigenem System
Um Ihr eigenes System zu härten, verwenden wir die beiden Kacheln **Configuration Assessment** und **Vulnerability Detection**. Bei **Configuration Assessment** wird die Konfiguration der angeschlossenen Agenten aufgrund von CIS Benchmarks untersucht. Weitere Informationen zu CIS Benchmarks haben Sie bereits in der Theorie zum Thema [hardening](../inputs/hardening.md) kennengelernt. Die Empfehlungen aus den CIS Benchmarks wurden durch wazuh in Prüfroutinen umgesetzt, die durch den wazuh-Agent auf dem zu schützenden System ausgeführt werden. Die Resultate der Tests zeigen Abweichungen zu den Empfehlungen aus den CIS Benchmarks (inkl. Verweis auf die Benchmarks). 

Beschäftigen Sie sich mit den Empfehlungen und überprüfen Sie diese kritisch. Ist es sinnvoll alles was empfohlen wird umzusetzen? Passen die Empfehlungen auf Ihre Situation?

Wenn Sie beispielsweise einen Windows-Client untersuchen, dann finden die Tests des wazuh-Agenten teilweise nicht relevante Probleme. So kann es passieren, dass er in der Registry settings findet, die unsicher sind aber zu nicht aktivierten Windows-Features gehören. D. h. die Empfehlung ist so lange irrelevant, bis das Windows-Feature tatsächlich aktiviert würde. Finden Sie auch bei Ihrem Gerät ähnliche Fälle?

Wichtig für die Praxis ist, dass Sie die Empfehlungen einmal alle durchgehen. Das, was sich für Ihre Situation als irrelevant herausstellt, kann entsprechend ignoriert werden. Alles andere sollten Sie aber angehen und lösen, wenn Ihnen bzw. Ihrem Unternehmen eine möglichst hohe Systemsicherheit wichtig sind.

Die zweite Kachel **Vulnerability Detection** zeigt auf, welche Software mit bekannten Schwachstellen auf Ihrem Endpunkt aktuell ausgeführt werden. Die Schwachstellen sind nach Kritikalität sortiert, was eine einfache Priorisierung für die Behebung der Probleme gestattet.

Analysieren Sie die Informationen für Ihre Situation. Finden Sie Software, die schon seit längerem Security-Updates zur Verfügung stellt und auf Ihrem Rechner noch nicht installiert wurden?

Beheben Sie die gefundenen Fehlkonfigurationen und installieren Sie Updates soweit möglich / sinnvoll / gewünscht, um Ihr eigenes System zu härten. Achten Sie aber dabei darauf vorsichtig vorzugehen und nichts zu unternehmen, was die Stabilität Ihres Rechners bzw. die Funktionalitäten, die Sie benötigen beeinträchtigen könnten. Machen Sie gegebenenfalls Backups bevor Sie Ihr System anpassen / optimieren.

### wazuh auf Ihre Systemsituation konfigurieren
Bis hierhin haben Sie wazuh so betrieben, wie es nach der Installation daherkommt. In der Praxis passt diese Standardkonfiguration aber nie exakt auf das System, das Sie überwachen sollen. In diesem Schritt begründen Sie zuerst die Werkzeugwahl und passen wazuh anschliessend auf Ihre konkrete Situation an.

#### Die Werkzeugwahl begründen
Stellen Sie sich vor, Sie sollen den öffentlich erreichbaren Webserver eines Unternehmens überwachen (dasselbe Unternehmen wie im [Risikoanalyse-Auftrag](./auftrag_risikoanalyse.md)). Zur Auswahl stehen die Tools aus der Theorie [Monitoring](../inputs/monitoring.md): Fail2Ban, Suricata, Velociraptor, Zabbix und wazuh.

Begründen Sie schriftlich, warum Sie für diese Situation wazuh wählen würden - und nennen Sie eine Situation, in der eines der anderen Tools die bessere Wahl wäre. Ein Tool auszuwählen heisst immer auch, die Alternativen zu kennen.

#### wazuh anpassen
Setzen Sie die folgenden Anpassungen um und dokumentieren Sie jeweils, **was** Sie geändert haben und **warum** diese Änderung für Ihr System sinnvoll ist:

1. **Passende CSA-Policy zuweisen:** wazuh liefert für jedes Betriebssystem eigene Policies für das Configuration Assessment mit. Prüfen Sie, ob die aktive Policy tatsächlich zu Ihrem überwachten System passt, und korrigieren Sie das gegebenenfalls.
2. **Alarmschwelle festlegen:** wazuh bewertet jede Regel mit einem Level. Ab welchem Level soll bei Ihnen ein Alarm ausgelöst werden? Setzen Sie die Schwelle bewusst und begründen Sie Ihre Wahl - was passiert bei einer zu tiefen, was bei einer zu hohen Schwelle?
3. **Eigene Regel erstellen:** Erstellen Sie mindestens eine eigene oder angepasste Regel, die für Ihr System relevant ist (z. B. auf wiederholte fehlgeschlagene Anmeldungen). Lösen Sie die Regel anschliessend mit einem selbst provozierten Ereignis aus und weisen Sie mit einem Screenshot aus dem Dashboard nach, dass sie greift.

   > **KI-Review-Schleife:**
   > 1. Schreiben Sie die Regel **zuerst selbst**, auch wenn sie noch nicht perfekt ist.
   > 2. Lassen Sie sie anschliessend mit dem [Review-Prompt für wazuh-Regeln](../ki-leitfaden.md#reviewer-generierte-wazuh-regel-hinterfragen) auf Fehlalarme und blinde Flecken prüfen.
   > 3. Halten Sie im KI-Log fest: **was Sie übernommen und was Sie verworfen haben - und warum**. Ein begründet verworfener Vorschlag zählt mehr als ein unbegründet übernommener.
   >
   > Eine von der KI vorgeschlagene Regel gilt erst als Ihre, wenn Sie sie getestet haben und erklären können, warum sie so aussieht.
4. **Active Response konfigurieren:** Hinterlegen Sie für diese Regel eine automatische Reaktion (z. B. das Sperren der auslösenden IP-Adresse) und testen Sie diese. Halten Sie fest, an welcher Stelle wazuh damit vom **HIDS** zum **HIPS** wird - die Begriffe kennen Sie aus der Theorie [Monitoring](../inputs/monitoring.md).

> **Vorsicht bei der Active Response:** Testen Sie diese nicht über die Verbindung, mit der Sie selbst auf dem System arbeiten - sonst sperren Sie sich unter Umständen selbst aus.

### Grenzen von wazuh beurteilen
Bis hierhin haben Sie gesehen, was wazuh alles kann. Genauso wichtig für die Praxis ist aber die andere Frage: **Was sieht wazuh in Ihrem Setup nicht?** Wer die Grenzen eines Tools nicht kennt, wiegt sich in falscher Sicherheit - und genau das ist gefährlicher als gar kein Monitoring, weil dann niemand mehr nach weiteren Massnahmen fragt.

Gehen Sie Ihr eigenes Setup nochmals durch und halten Sie schriftlich fest, wo die Grenzen liegen. Die folgenden Denkanstösse helfen Ihnen dabei - sie sind aber weder vollständig noch müssen alle auf Ihre Situation zutreffen:

- **Erkennen ist nicht verhindern:** wazuh ist primär ein SIEM, es alarmiert also in erster Linie. Ein Alarm blockiert noch gar nichts. Haben Sie in Ihrem Setup eine Active Response konfiguriert? Und wenn ja: Wie schnell reagiert diese im Vergleich zu einem Angreifer, der bereits auf dem System ist?
- **Hostbasiert statt netzwerkbasiert:** Der Agent sieht, was auf dem Endpunkt passiert. Was passiert zwischen Ihren Endpunkten im Netzwerk, bleibt ihm verborgen. Welche Angriffe würde ein NIDS / NIPS (siehe Theorie [Monitoring](../inputs/monitoring.md), Abschnitt Suricata) sehen, die Ihr wazuh-Setup nicht sieht?
- **Nur überwachte Geräte sind überwachte Geräte:** wazuh weiss nur von Endpunkten, auf denen ein Agent installiert ist. Wie sieht es mit privaten Geräten (BYOD), Druckern, IoT-Geräten oder mit [Schatten-IT](../inputs/Schatten-IT.md) aus, von der die IT-Abteilung gar nichts weiss?
- **Der Mensch als Angriffsziel:** Wenn eine Mitarbeiterin ihre Zugangsdaten auf einer Phishing-Seite eingibt und sich der Angreifer anschliessend regulär anmeldet - was genau sollte wazuh daran als verdächtig erkennen?
- **Schwache und wiederverwendete Passwörter:** wazuh prüft Konfigurationen und bekannte Schwachstellen. Sagt es Ihnen auch, ob Ihr Passwort schon in einem Datenleck aufgetaucht ist oder ob dasselbe Passwort auf fünf Systemen verwendet wird?

Notieren Sie mindestens **zwei Grenzen**, die für Ihr eigenes Setup tatsächlich relevant sind. Diese brauchen Sie für die Reflexionsfragen.

---

## Reflexionsfragen
1. Beschreiben Sie die **zwei Grenzen** Ihres wazuh-Setups, die Sie im vorherigen Abschnitt festgehalten haben. Skizzieren Sie zu jeder Grenze ein konkretes Angriffsszenario, das dadurch unentdeckt bliebe.
2. Wählen Sie **zwei Zusatzmassnahmen**, mit denen Sie Ihr wazuh-Setup ergänzen würden - davon **eine technische und eine organisatorische**. Mögliche Stossrichtungen (Sie dürfen auch andere wählen):
   * **Intrusion Prevention:** beispielsweise [Fail2Ban](../inputs/monitoring.md#fail2ban-hids-hips) als HIPS, das auffällige IP-Adressen automatisch sperrt.
   * **Mitarbeitersensibilisierung / Schulung:** siehe [Überblick über häufige Probleme in der Systemsicherheit](../inputs/README.md), Abschnitt "Organisatorisch".
   * **Passwort-Tooling:** Passwortmanager, Multi-Faktor-Authentifizierung, Prüfung gegen bekannte Datenlecks.

   Begründen Sie jede der beiden Massnahmen schriftlich und gehen Sie dabei auf drei Punkte ein:
   * Welche der unter 1. genannten Lücken schliesst die Massnahme?
   * Welche Wirkung erwarten Sie konkret (was würde neu erkannt oder verhindert)?
   * Welche neuen Nachteile, Kosten oder Risiken handeln Sie sich damit ein (z. B. Aufwand, Fehlalarme, ausgesperrte Benutzer)?
3. Warum lösen zusätzliche Tools das Problem nie allein? Nehmen Sie in Ihrer Antwort Bezug auf das Prinzip [Defense-in-Depth](./ikt_minimalstandard.md#defense-in-depth).
4. Falls Sie bei der eigenen wazuh-Regel KI eingesetzt haben: Welchen Fehlalarm oder welchen blinden Fleck hat die KI aufgezeigt, den Sie selbst übersehen hatten - und welchen ihrer Vorschläge haben Sie nach dem Test verworfen?

---

## Abgabe
Sofern die Lehrperson nichts anderes festlegt, sollten Sie Ihre Dokumentation (Erkenntnisse aus dem Dashboard, durchgeführte Härtungsmassnahmen, beurteilte Grenzen sowie die Antworten auf die Reflexionsfragen) in geeigneter Form der Lehrperson abgeben.

