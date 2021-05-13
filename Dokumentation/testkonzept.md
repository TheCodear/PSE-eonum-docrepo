# Testkonzept für Projekt eonum

## Testkonzept

Im Folgenden wird das Testkonzept für das Projekt eonum beschrieben. 

Allgemein zu den Anforderungen kann gesagt werden, dass die Testanforderungen nur relativ gering 
sind, da es sich bei MedCodeSearch auch "nur" um eine Suchmaschine handelt, also eine nicht kritische Anwendung, welche Daten nur lesend verarbeitet und anzeigt.
Das bestehende Testniveau ist hauptsächlich auf die funktionale Ebene beschränkt (im FE & auch im BE) sowie einige API-Integrationstests im BE. Das Konzept zielt
also darauf ab, das vorhandene Niveau zu halten.

### Unit Tests

Unit Tests werden gemacht, um die Funktionalität der einzelnen Teile zu testen. Im Angular Frontend beschränkt sich der Scope der Unit Tests deshalb auch auf die 
funktionalen Units (Funktionen in den Components). Es werden keine Tests für die HTML-Templates gemacht.

Im Rails Backend sollen mit Unit Tests in erster Linie die Models getestet werden. Und damit implizit
die Datenbank mit lesenden Zugriffen. Dazu werden auch produktive Daten in die Testdatenbank geladen.

Zudem werden die Komponenten Parser und Webcrawler erweitert mit Unit Tests abgedeckt. Der
Scope dieser Unit Tests ist einerseits die Logik auf die Korrektheit zu überprüfen und andereseits (v.a. im Parser)
den Algorithmus, der das KLV1 Dokument verarbeitet, zu testen und so sicherzustellen, dass bei der Abbildung
vom PDF ins MCS-System keine Informationen verloren gehen.

Die Unit Tests im Webcrawler zielen ausserdem darauf aus, dass wenn das BAG bei ihrer Website die Struktur
ändert, die Tests fehlschlagen, d.h. hier geben die Tests auch eine Indikation, ob der Webcrawler noch funktionstüchtig
ist und abholen kann, was er soll, oder ob es Handlungsbedarf gibt.

Alle anderen Komponenten (Controller, Tasks, Views, Search Helper, etc.) sind nicht mit Unit Tests abgedeckt.
Das ist vom Projekt her nicht gegeben und für uns deshalb auch nicht im Scope.

### Datenbank Tests

Die Datenbank wird einerseits implizit in den Unit Tests für die Models getestet, andererseits auch in den BE-Integrationstests.
Anzumerken hierbei ist, dass auf die Datenbank (ausser wenn Daten eingespiesen werden), nur lesend zugegriffen wird. D.h.
es gibt insbesondere keine kritischen Operationen, die durch Tests abgedeckt werden müssen.

Die Daten für die Tests werden direkt aus den produktiven Quellen bezogen, da nur lesend darauf zugegriffen wird und MCS sowieso eine
Suchmaschine ist, lohnt es sich nicht, extra Testdaten zu generieren, mit denen die Test-Datenbank gespiesen
werden kann.

### Integrationstest

Integrationstests werden nur im BE für die API gemacht. Dabei werden hauptsächlich die Endpunkte
auf ihre grundsätzliche Funktion geprüft (d.h. dass Status 200 zurückkommt bei einer normalen Anfrage). Einige
Use Cases werden für die KLV1 Endpunkte getestet, aber nicht extensiv.

Es gibt insbesondere auch keine kritischen Anfragen, die speziell getestet werden müssen.

### Installationstest

Die Installation des Projektes wird nicht explizit getestet, da es sich um ein 
bestehendes Projekt handelt, das bereits läuft (also auch installiert ist).

Die für die Erweiterung benötigten Schritte wurden im Setup-Prozess vermerkt.

Implizit wird die Installation durch eonum getestet, da sie den Code Reviewen und so das Projekt mit
unserer Erweiterung zum Laufen bringen müssen.

### GUI Test

Das GUI wird nur manuell getestet.

### Stress-Test

Stress Tests sind keine vorgesehen.

### Usability-Test

Usability-Testing ist nicht vorgesehen. Eonum "testet" unsere Erweiterung im Rahmen der
Code-Review, dies kann jedoch nicht als echtes Usability-Testing angesehen werden.

## Testresultate

Da das Testkonzept nicht viele Tests voraussieht, gibt es hier auch nicht allzu viel zu reporten.
Trotzdem sind die wichtigsten Ergebnisse zu den einzelnen Punkten hier aufgeführt.

### Unit Tests

Unit Tests im FE laufen alle erfolgreich durch.

Im Backend laufen ebenfalls alle Tests durch, ausser ein Test für die KLV1 Models
musste als `ignored` markiert werden, da er aufgrund eines Fehlers im Parsing-Algorithmus
fehlschlägt:

Der Grund für den Fehler ist ein durchgestrichenes Wort im Quell-PDF, das
durchgestrichen ist. Dies führt zu einer Missinterpretation durch das Gem, welches
diese PDF Seite Scannt. Es erkennt dort fälschlicherweise eine Tabellenlinie und splittet
so den Eintrag in zwei Einträge. 

Wir haben diesen Fehler nicht behoben, da der Aufwand dafür zu gross gewesen wäre.
Trotzdem haben wir den Test drin belasse und skippen ihn. So ist dieser Fehler dokumentiert.

Die Testabdeckung wurde nicht gemessen, da auch es keine Anforderung diesbezüglich
gibt.

### Datenbank Tests

Die impliziten Datenbanktests (Unit Tests & Integrations Tests) laufen alle durch.

### Integrationstests

Die Integrationstests für die API laufen alle durch.

### Installationstests

Eonum konnte das Projekt mit unserer Erweiterung ohne Probleme zum Laufen bringen,
d.h. der implizite Installationstest war erfolgreich.

### GUI Tests

Da manuell durchgeführt, gibt es auch keine wirklichen Testergebnisse. Mängel, die durch
das manuelle testen gefunden wurden, sind alle behoben worden.

### Stress-Tests

Da keine Stress-Tests vorgenommen worden sind, gibt es auch keine Resultate.

### Usability-Tests

Da keine wirklichen Usability-Tests gemacht worden sind, gibt es auch keine
Ergebnisse. Die Mängel, die durch die Review von Eonum angemerkt worden sind, 
wurden alle behoben.

