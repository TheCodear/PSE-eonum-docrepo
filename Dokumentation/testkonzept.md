# Testkonzept für Projekt eonum

## Testkonzept

Hier wird kurz das Testkonzept für das Projekt eonum beschrieben. Allgemein zu den Anforderungen kann gesagt werden, dass die Testanforderungen nur relativ gering 
sind, da es sich bei MedCodeSearch auch um eine Suchmaschine handelt, also eine nicht kritische Anwendung, welche Daten nur lesend verarbeitet und anzeigt.
Das bestehende Testniveau ist hauptsächlich auf die funktionale Ebene beschränkt (im FE & auch im BE) sowie einige API-Integrationstests im BE. Das Konzept zielt
also eher darauf ab, das vorhandene Niveau mindestens zu halten.

### Unit Tests

Unit Tests werden gemacht, um die Funktionalität der einzelnen Teile zu testen. Im Angular Frontend beschränkt sich der Scope der Unit Tests deshalb auch auf die 
funktionalen Units. Es werden keine Tests für die Darstellung gemacht.

Im Rails Backend sind im Scope der Unit Tests einerseits die Models, die die Datenbank lesend testen und andererseits sind für den Webcrawler und den Parser Unit
Tests vorhanden, um die gesamte Funktionalität zu testen.
Andere Komponenten des Backends, wie z.B. Controllers, werden nicht direkt über Unit Tests abgedeckt, aus dem Grund, da einerseits die API über Integrationstests
getestet wird (damit implizit die Controller) und andererseits, weil auch diese auch im bestehenden Teil des Projektes nicht mit Tests abgedeckt worden sind.

### Datenbank Tests

Die Datenbank wird in den Unit Tests getestet und zwar grösstenteils nur mit lesenden Zugriffen. Das, weil die Suchmaschine auch nie Daten anders verarbeitet als 
lesend. Einzig der Parser schreibt Daten in die Datenbank (bzw. die Rake Tasks für die andere Kode-Daten). In den Parser-Unit Tests wird die Datenbank deshalb
auch geringfügig schreibend & lesend getestet.
Allgemein überwiegt die Anwendung der Datenbank als Speicher und insofern ist es nicht nötig, weiterführende Datenbanktests zu machen. Als Test-Daten werden deshalb
auch echte Daten verwendet, die auch in der Produktion zur Verfügung stehen. Es werden also nicht zusätzliche Test-Daten generiert für die Datenbanktests.

### Integrationstest

Integrationstest beschränken sich auf die Backend-API. Da wird auch hauptsächlich überprüft, dass alle Endpunkte ordnungsgemäss funktionieren. Insofern sind durch 
diese Integrationstests auch die hauptsächlichen Use-Cases abgedeckt, dass ein GET Request an die API mit einem Suchbegriff geschickt wird und eine positive Antwort
zurückerwartet wird.

(Wie werden die Storys getestet? Welches sind die Use Cases? Wie werden spezielle Fälle (z.B. sog. ”Race conditions“) 
getestet?)

### Installationstest

Da das Projekt nicht von Grund auf neu gebaut worden ist, ist die Installation auch bereits vorgegeben. Einzig die zusätzlichen Anforderungen durch den Parser müssen
berücksichtig werden (eine lib, die auf dem System, wo das Backend läuft, enthalten sein muss). Dies wurde in der (bereits vorhandenen) Installationsanweisung 
vermerkt und konnte auch gleich durch eonum getestet werden, als sie den Code gereviewt haben.

### GUI Test

Das GUI wird nicht automatisiert getestet. Von "Hand" wird nur der neu hinzugefügte Teil getestet. Viele Funktionalitäten und sind bereits gegeben (siehe auch 
Usability-Test).

### Stress-Test

Stress Tests sind keine vorgesehen.

### Usability-Test

Usability-Testing wird direkt durch eonum vorgenommen. Da auch das Frontend bereits existiert hat, sind viele Elemente gleich und vorgegeben und müssen nicht mehr
getestet werden. Die Erweiterung wird und wurde aber getestet.

(Wer sind die künftigen Anwender der Software? Über welche Vorkenntnisse verfügen sie? Wer testet die Software auf ihre 
Bedienbarkeit? Wie läuft dieser Usability-Test ab? )

## Testresultate

Nachdem alle im Testkonzept vorgesehenen Tests durchgeführt wurden, sind die Testresultate zu dokumentieren
und bei Projektende als Teil der Dokumentation abzugeben.
