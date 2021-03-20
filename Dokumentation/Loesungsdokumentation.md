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

### Beschreibung System und Lösungsansätze

#### Datenabholung

Die zu integrierenden Datensätze sind die Anhänge 1 und 1a des KLV sowie das Kodierhandbuch und das dazugehörige
Rundschreiben. Erstere finden sich [hier](https://www.bag.admin.ch/bag/de/home/versicherungen/krankenversicherung/krankenversicherung-leistungen-tarife/Aerztliche-Leistungen-in-der-Krankenversicherung/anhang1klv.html)
 und das Kodierhandbuch mit dem Rundschreiben [hier](https://www.bfs.admin.ch/bfs/de/home/statistiken/gesundheit/nomenklaturen/medkk/instrumente-medizinische-kodierung.html).

Im Moment werden die anderen Daten über ein weiteres Data Repository der Applikation zur Verfügung gestellt und von dort
mit Rake Tasks in die Datenbank geladen. Für die neuen Datenquellen bietet sich natürlich die gleiche Möglichkeit, jedoch
müssten dann die jeweiligen PDFs manuell immer hinzugefügt werden...

Eine andere Variante ist folgende:
Ein Rake Task (mit mehreren Subtasks) besucht die Quellseiten, holt die PDFs dort ab und lädt sie dann in die Datenbank.

Bei dieser Variante stellen sich noch folgende Fragen:

- Soll der Task gescheduled werden (jährlich für KLV Anhänge und Kodierhandbuch, halbjährlich für Rundschreiben)?
- Oder soll der Task jeweils manuell angestossen werden?
- Ist die produktive Umgebung vom Medcodesearch BE durch irgendwelche Firewalls / Proxies geschützt, sodass man nicht
aufs Web direkt zugreifen kann?
- Wie generisch sollen die Tasks gehalten werden? Gibt es eine Anforderung, dass auch dynamisch weitere Quellen
integriert werden sollen?
  
Diese Variante könnte mit folgenden Technologien umgesetzt werden:

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

Im Task sollte dann auch gleich die Verarbeitung der PDFs geschehen und das Einfügen in die Datenbank. Hier stellt sich noch 
die Frage nach dem Parsen, wie das genau gehen soll. Es gibt keine "Standardlösung", PDFs einfach so zu parsen und die Quellen
enthalten auch entsprechend komplexere Teile (z.B. Tabellen).

D.h. hier braucht es die Lösung eines dedizierten Parsers.

##### Parser

tbd

#### Datenbank

Folgendes Diagramm zeigt das Schema Version 1.0:

<img src="images/db_scheme.png" alt="db_scheme.png">

#### API

tbd

#### Frontend

[Link](https://github.com/TheCodear/PSE-eonum-docrepo/blob/master/Dokumentation/FE_Mockup.pdf) zu den Design Mockups im Frontend.
