# Lösungsdokumentation eonum

## Beschrieb

Diese Dokumentation beschreibt die erarbeiteten Lösungen für die Erweiterungen von MCS (MedCodeSearch). MCS ist ein Such-
portal für die medizinische Kodierung, sowohl als Website zugänglich als auch als öffentliche API. Enthalten in diesem
Dokument sind neben Text auch Diagramme, Bilder etc.
Die Erweiterung von MCS beinhaltet folgende Punkte:

- Ergänzung der bestehenden Inhalte mit Gesetzestexten und Handbüchern
- Integration der neuen Inhalte ins FE und BE.

## Ziel 1

Als erstes Ziel sollen folgende Punkte erfüllt werden:

- Wenn ich mich im Reiter Gesetze befinde, dann kann ich nach Protonen-Strahlentherapie suchen. Alle Suchergebnisse im
 Reiter Gesetze sollen nun angezeigt werden.
- Überall wo das Wort ’Protonen-Strahlentherapie’ vorkommt, soll es markiert werden.
- Ich kann nun auf ein Resultat klicken und mehr darüber lesen.

Dieses Ziel wurde in den Iterationen 2 & 3 für die Quelle KLV Anhang 1 erarbeitet, für die Quelle Medizinisches Kodierhandbuch soll in einem PoC ein Lösungsansatz in den Iterationen 3 & 4 erarbeitet werden.

### Beschreibung System und Lösungsansätze

#### Datenabholung

Die Quelle KLV Anhang 1 wird automatisiert von der BAG Seite abgeholt. Dafür gibt es eine Webcrawler-Klasse, die diese Abholung übernimmt. Die Vorgehensweise ist relativ simpel: 
Zuerst wird überprüft, welche Versionen des KLV Anhang 1 bereits vorhanden sind, dann werden die fehlenden Versionen abgeholt.

Um herauszufinden, welche Versionen verfügbar sind, wird die ältest verfügbare Version hard-coded gesetzt (Version 2 vom Jahr 2020). Dann wird geschaut, was die momentane Zeit ist. Davon wird die letzte verfügbare Version abgeleitet (Jan-Jun -> Version 1 des aktuellen Jahres, Jul-Dez -> Version 2 des aktuellen Jahres). Mit diesen beiden Rand-Versionen werden alle Versionen dazwischen berechnet. Dann wird im Data-Folder (dort wo die PDFs dann auch abgelegt werden) geschaut, welche dieser Versionen bereits vorhanden sind. Hierbei ist noch wichtig zu erwähnen, dass nur jeweils nach der Deutschen Version gesucht wird, da die Versionen dann auch immer in allen drei Sprachen geholt werden (d.h. wenn eine deutsche Version gefunden wird, wird davon ausgegangen, dass auch die französische und italienische Version vorhanden sind).

Schliesslich werden die fehlenden PDFs von der BAG Seite abgeholt und im Data-Folder für die weitere Prozessierung abgelegt.

Technisch basiert der Webcrawler auf den beiden Ruby-Gems 'nokogiri' und 'open-uri'. Zu beachten ist beim Technischen auch noch, dass davon ausgegangen wird, dass die PDFs in einem <div> HTML Tag mit der Klasse "mod mod-download" sind und jeweils "Gesamtliste" / "Édition complète" / "Edizione completa" im Titel beinhalten. Auch die Seite auf das BAG ist hard-coded. Sollte sich in diesen Dingen jemals etwas ändern, dann wird der Webcrawler nicht mehr funktionieren!
 
 -- Archiv

Die zu integrierenden Datensätze sind die Anhänge 1 und 1a des KLV sowie das Kodierhandbuch und das dazugehörige
Rundschreiben. Erstere finden sich [hier](https://www.bag.admin.ch/bag/de/home/versicherungen/krankenversicherung/krankenversicherung-leistungen-tarife/Aerztliche-Leistungen-in-der-Krankenversicherung/anhang1klv.html)
 und das Kodierhandbuch mit dem Rundschreiben [hier](https://www.bfs.admin.ch/bfs/de/home/statistiken/gesundheit/nomenklaturen/medkk/instrumente-medizinische-kodierung.html).

Es soll nun in einem ersten Schritt der KLV Anhang 1 mit dem Webcrawler abgegriffen werden. Das soll mit Rake Tasks geschehen, wie
folgendes Beispiel zeigt:

Die Packages "open-uri" und "Nokogiri" erlauben das Öffnen bzw. Parsen von Websiten. Damit kann eine Website abgerufen
und als HTML Dokument im Code verfügbar gemacht werden. Des Weiteren bietet "Nokogiri" auch entsprechende Methoden, um
das HTML Dokument zu durchsuchen, beispielsweise mit CSS Selektoren. So können auf der BAG und BFS Seite die richtigen
Links zu den richtigen Quellen gefunden werden und so die PDFs geholt werden.
Damit ein PDF gelesen werden kann, braucht es zusätzlich das gem "pdf-reader".

Folgender Task besucht die BFS Website und holt das Kodierungshandbuch 2021:

```
task :get_handbuch do
    base_url = 'https://www.bfs.admin.ch'
    links = []
    html_doc = Nokogiri::HTML(open(base_url + '/bfs/de/home/statistiken/gesundheit/nomenklaturen/medkk/instrumente-medizinische-kodierung.html'))
    html_doc.css('div[data-url*="kodierungshandbuch"]').each do |div|
      links << div['data-url']
    end
    link = links.first
    detail_page_link = Nokogiri::HTML(open(base_url + link)).css('a[class]').first['href']
    handbook_link = Nokogiri::HTML(open(base_url + detail_page_link)).css('a[class*="download-event"]').first['href']
    reader = PDF::Reader.new(open(base_url + handbook_link))
    puts reader.page_count
  end
```

Im Task sollte dann das geholte PDF abgelegt werden und dann der Task für das Parsen der Dokumente angestossen werden, sobald
alle Dokumente abgeholt sind, bzw. evtl. zuerst der Task für die Konvertierung der PDFs in ein maschinenlesbares Format.

Die Konverter / Parser - Thematik ist im Moment noch ein grosser weisser Fleck, im Folgenden werden Optionen für den Parser
diskutiert.

##### Parser

Nach dem Abholen der KLV Anhang 1 Versionen werden diese durch einen Parser verarbeitet. Da der KLV Anhang 1 hauptsächlich aus Tabellen besteht, ist auch hier die Idee relativ simpel: Parse die drei Sprachen einer Version nacheinander, füge die Resultate zusammen und schreibe sie in die Datenbank.

Für das Parsen eines einzelnen PDFs wird folgendermassen vorgegangen:
Zuerst wird geschaut, auf welchen Seiten des PDFs das Inhaltsverzeichnis zu finden ist. Dann werden die einzelnen Einträge des Verzeichnisses herausgelesen und jeweils registriert, welches Kapitel auf welcher Seite beginnt und auf welcher Seite endet.

Diese Informationen werden dann im Hauptschritt verwendet. Es wird nun Kapitel für Kapitel durchgegangen, die Seiten, die zu diesem Kapitel gehören, werden über einen OCR-Mechanismus gescannt und die erkannten Tabellen werden extrahiert und zu einer ganzen Tabelle zusammengefügt.

Beim Zusammenfügen der einzelnen Tabellen werden folgende Punkte beachtet:
 - Die Tabellen-Köpfe werden entfernt
 - Wenn in der ersten Zeile der Tabelle nur in der Spalte Voraussetzung etwas steht, dann wird der Inhalt zur letzten Zeile der vorherigen Tabelle hinzugefügt: der Inhalt ist über zwei Seiten verteilt
 - Wenn in der ersten Zeile der Tabelle nur in der Spalte gültig ab etwas steht, dann wird der Inhalt zur letzten Zeile der vorherigen Tabelle hinzugefügt: der Inhalt ist über zwei Seiten verteilt
 - Wenn nur in den Spalten Voraussetzung und gültig ab etwas steht, sowie die Spalte gültig ab nicht mit einem Grossbuchstaben beginnt und in der vorherigen Zeile (der lezten Zeile der vorherigen Tabelle) in der Spalte gültig ab '/' als letztes Zeichen vorkommt, dann wird der Inhalt zur letzten Zeile der vorherigen Tabelle hinzugefügt: der Inhalt ist über zwei Seiten verteilt
 - Wenn in der ersten Spalte (Massnahme) der Inhalt mit einem Kleinbuchstaben beginnt, dann wird der Inhalt zur letzten Zeile der vorherigen Tabelle hinzugefügt: der Inhalt ist über zwei Seiten verteilt
 
Wenn die Tabellen zusammengefügt worden sind, dann wird die ganze Tabelle Zeile für Zeile durchgegangen und es werden Massnahme-Objekte kreiert. Dabei wird folgendermassen vorgegangen:
- Wenn in der Massnahmen Spalte ein normaler Eintrag ist, wird ein neues Massnahme-Objekt kreiert, die Felder Leistungspflicht bis gültig ab werden in einem KLV-Objekt diesem Massnahme Objekt hinzugefügt.
- Wenn in der Massnahmen Spalte kein Eintrag ist, dann werden die andere Spalten als KLV-Objekt dem letzten Massnahme-Objekt angehängt.
- Wenn in der Massnahmen Spalte die Massnahme mit '-' beginnt, dann wird diese Massnahme prefixed, d.h. die letzte normale Massnahme wird vor den Bindestrich hinzugefügt und alles wird als neues Massnahmen-Objekt gespeichert.
- Leere Leistungspflicht Felder werden mit dem letzten gelesenen Eintrag gelesen.

Wenn dann die Tabelle in Objekte umgewandelt worden ist, dann werden die Objekte mit den (falls vorhanden) anderen Sprachen zusammengemerged. Dabei sind foldende Punkte wichtig:
- Beim mergen der Massnahme-Objekte wird zuerst überprüft, ob die dazugehörigen KLVs auch äquivalent sind, d.h. ob die Leistungspflicht dieselbe ist und ob die Gültigkeit auch dieselbe ist. Wenn das der Fall ist, werden die Sprachen gemerged, ansonsten wird das Massnahme-Objekt ohne zu mergen mit der nächsten verfügbaren Massnahmen-Nummer versehen und am darüberliegenden Kapitel angehängt. Dieser Fall sollte so selten als möglich auftreten, trotzdem ist er wegen Inkonsistenzen in den PDFs nicht auszuschliessen.

Sind alle drei Sprachen geparsed, werden die Objekte in die dazugehörigen Tabellen der Datenbank mit den dazugehörigen Verbindungen gespeichert. Dann ist das Parsen einer Version abgeschlossen.

-- Archiv

Folgende Gems / Libs sind für den Parser bis jetzt betrachtet worden:

- pdf-reader: ist ein Ruby-Gem, welches ermöglicht, PDF Files zu öffnen und auch gewisse Extraktionen zu machen (v.a. Text
  kann aus dem PDF herausextrahiert werden, dies ist jedoch nicht sehr akkurat...). Zudem kann das Dokument auf sehr tiefer
  Ebene durchgegangen werden, d.h. man kann low-level objects abgreifen. Um damit aber etwas anzufangen, bräuchte es rel. viel
  PDF Knowledge, deshalb nicht unbedingt geeignet.
  
- yomu: Ist ein weiteres Ruby-Gem, welches PDFs öffnen und auch umwandeln, bzw. Metadaten abgreifen kann. Die Transformation in
  ein HTML-File funktioniert auch relativ gut, aber für Tabellenstrukturen wird der ganze Inhalt durcheinander gewürfelt, d.h.
  auch nicht alleinig geeignet.
  
- Docsplit ist ein weiteres Ruby-Gem, mit dem PDFs gesplittet werden können oder auch Text extrahiert werden kann, aber auch
  da ist die Text-Extraktion nicht geeignet, da so überhaupt keine Struktur der PDFs übernommen wird. Einzig was von Nutzen sein
  könnte, ist die Fähigkeit, das PDFs in einzelne Seiten zu splitten.
  
- Neben den Ruby-Gems, gibt es auch noch die Möglichkeit, mit System-Calls die Befehle `pdftotext` bzw. `pdftohtml` auszuführen.
  Diese wandeln die PDFs in Text bzw. HTML files um, mit sogar relativ guter Accuracy, jedoch gehen auch da im HTML beispielsweise
  die Tabellenstrukturen verloren, also auch da nicht alleinig geeignet für das Konvertieren / Parsen der PDFs.
  
- Iguvium: ist ein Ruby-Gem mit dem sich Tabellen aus einem PDF auslesen lassen und verschiedenst konvertieren (in Arrays 
  direkt in Ruby, in CSV, etc.). Damit könnte also das Problem der Tabellen in den Files gelöst werden. Diese Gem braucht
  aber Ghostscript als Abhängigkeit installiert auf dem OS, auf dem es läuft.
  
- Nebst Ruby-Gems besteht auch die Möglichkeit, den Parser / Konverter in Python zu schreiben. Hier gibt es dedizierte Libs, 
  welche PDFs verarbeiten können (PDFMiner, PyPDF2, pdfrw, slate, camelot)
  
*Verfolgter Lösungsansatz*

Für den KLV Anhang 1 ist gegeben, dass alles in Tabellenform ist, ausser der Überschriften der Abschnitte. Ansonsten ist die
Tabelle aber immer gleich aufgebaut, nämlich mit "Massnahmen", "Leistungspflicht", "Voraussetzungen" und "gültig ab". Da es nicht
möglich ist, gleichzeitig normalen Text und Tabellen zu extrahieren, wird folgender Ansatz verfolgt:
  
Da alle Überschriften und Abschnitte im Inhaltsverzeichnis aufgelistet sind, auch mit den jeweiligen Seitenzahlen, kann man
zweistufig vorgehen: Extrahiere zuerst aus dem Inhaltsverzeichnis Informationen über das Dokument. Insbesondere die Überschriften,
bzw. Kapitel, auf welcher Seite diese zu finden sind, auf wie viele Seiten sich das erstreckt. Dann sammle diese Informationen und
gehe gemäss diesen auf die jeweiligen Seiten, extrahiere die Tabellen und extrahiere dann aus den Tabellen jeweils die notwendigen
Informationen, die dann in die Datenbank geschrieben werden. So kann man sicher sein, dass man zu jeder Tabelle, die man extrahiert
auch den Abschnitt weiss, bzw. dass aller Inhalt gemäss dem Inhaltsverzeichnis aufgenommen wird.

Dazu muss das Inhaltsverzeichnis zuerst identifiziert und geparsed werden. Die Informationen, die zu sammeln sind, sind der Abschnitt
und die zugehörigen Seiten.
Dann wird Abschnitt für Abschnitt abgearbeitet, zuerst jeweils auf den Seiten die Tabellen extrahiert und dann alle Tabellen zusammengefügt.
Dabei muss auf folgendes aufgepasst werden:
- Es kann sein, dass ein Eintrag über eine Seite hinausgeht, insb. wenn nur in der Spalte Voraussetzung etwas steht auf der neuen Seite,
dann gehört der Inhalt noch zur letzten Zeile der vorherigen Seite.
- Bei der Version 2020 ist auf einer neuen Seite nicht mehr eine Header Zeile drin (mit Massnahme, Leistungspflicht, etc.), bei
  der Version 2021 jedoch schon.
    
Nachdem die Tabellen zusammengefügt worden sind, müssen die Informationen daraus gewonnen werden, Objekte erstellt werden, die
dann in die DB geschrieben werden können. Dazu folgendes Vorgehen:
1. Erstelle zuerst für das Chapter einen Eintrag (Überschrift, Nr, etc.), damit dann die ID für die darunterliegenden Massnahmen
   vorhanden ist.
   
2. Dann lese eine Zeile ein. Mit der Massnahme soll ein Measure Objekt erstellt werden, ausser die Spalten Leistungspflicht, Voraussetzungen
   und gültig ab sind leer, dann erstelle kein Measure-Objekt. Falls diese Spalten nicht leer sind, erstelle einen DB Eintrag für die Massnahme,
   um dann eine ID für den (Leistungspfl, Vor, gültig)-Eintrag zu bekommen. Aus diesen Tabellen soll dann ein KLV-Eintrag erstellt werden.
   In jedem Fall speichere die gelesene Massnahme in einer Variablen zwischen.
   
3. Lese die nächste Zeile ein. Hier gibt es nun drei Möglichkeiten 
   
   a. Ist die Massnahme Spalte Leer, dann erstelle aus den anderen Spalteneinträge ein KLV mit derselben ID auf die Measure wie bei
   der Zeile zuvor.
   
   b. Beginnt die Massnahme mit einem Bindestrich ('-'), dann erstelle einen neuen Massnahme-Eintrag mit der eingelesenen Massnahme
   und der zwischengespeicherten Massnahme als Präfix und erstelle für die anderen Spalteneinträge einen KLV-Eintrag mit der Id 
   der gerade erstellten Massnahme.
   
   c. Ist die Massnahme ohne Bindestrich und nicht leer, verfahre wie unter 2.
   
4. Lies die nächste Zeile ein und verfahre wie unter 3.

Falls die Spalte Leistungspflicht leer ist und auch die Spalte Massnahme leer ist, dann muss der Leistungspflichteintrag der vorherigen
Zeile übernommen werden.
   
   


*Alternativer Lösungsansatz*

Man könnte das Parsen bzw. das Konvertieren des PDFs in ein maschinenlesbares Format auch auslagern in ein Python-Skript. Python
bietet viele mächtige libs und Werkzeuge, die das Parsen erleichtern könnten...
Es ist jedoch vorzuziehen, eine Lösung in Ruby zu bauen, wenn das möglich ist, da dann nicht noch Technologiewechsel etc. 
vollzogen werden müssen...

#### Datenbank

Folgendes Diagramm zeigt das Schema Version 1.0:

<img src="images/db_scheme.png" alt="db_scheme.png">

#### API

tbd

#### Frontend

[Link](https://github.com/TheCodear/PSE-eonum-docrepo/blob/master/Dokumentation/FE_Mockup.pdf) zu den Design Mockups im Frontend.
