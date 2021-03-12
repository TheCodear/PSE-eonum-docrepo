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
und als HTML Dokument im Code verfügbar gemacht werden. Des weiteren bietet "Nokogiri" auch entsprechende Methoden, um
das HTML Dokument zu durchsuchen, beispielsweise mit CSS Selektoren. So können auf der BAG und BFS Seite die richtigen
Links zu den richtigen Quellen gefunden werden und so die PDFs geholt werden.

tbd: ein Beispiel

Im Task sollte dann auch gleich die Verarbeitung der PDFs geschehen und das Einfügen in die Datenbank...

#### Datenablage und Datenverarbeitung

tbd

#### API

tbd

#### Frontendintegration

tbd