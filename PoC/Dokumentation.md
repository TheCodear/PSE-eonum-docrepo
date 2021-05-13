# PoC - Hauptdokumentation

## Ansätze

* Indexiere ganzes Pdf und schauen, was zurückkommt. -> nicht weiter verfolgt
* Pdf in einzelne Abschnitte trennen und schauen, was zurück kommt. -> verfolgt & verfeinert
* Benutze Highlight um Fragmentgrösse zu definieren und schaue, wie
  viel zurück kommt.
* Kompromiss wäre, Pdf als ganzes zu durchsuchen und beschränkte
  Seitenzahl mit übereinstimmenden Anfragen auf Tabelle durchführen. Wie verhälts sich diese?
  
TODO: weiter verfeinern!
  
## Entscheide / fixe Outcomes

...

## Activity Log

### 13.5.21 Janni & Sascha

Man kann nun via `localhost:3000/de/poc/search?search=Suchbegriff` eine Suche starten. 
Als Antwort erhält man ein Array mit Übereinstimmungen der Suche, welche nach `score` geordnet ist.
Darin enthalten ist die Seitenzahl, der ganze Text dieser Seite sowie die Seite encoded als Base64. 

Hierbei hat es im Moment noch `\n`im Base64, welches das Base64 unbrauchar macht. Muss man noch daraus entfernen oder 
herausfinden, warum es reingeschribene wird

Was ist wichtiger: Ordnung der Suchresultate nach Score oder nach aufsteigender Seitenzahl?





### 13.5. 16:24 Jan

Es ist auch möglich, den base64 String direkt in einem PDF Viewer anzuzeigen. Um das auszutesten,
habe ich eine kleine Angular App gebaut, die eigentlich nur die lib `ng2-pdf-viewer` verwendet.
Dann kann man im HTML dieser Komponente diesen PDF Viewer einbinden:
```html
<pdf-viewer [src]="obj" [render-text]="true" style="display: block;"></pdf-viewer>
```
Hierbei kann `obj` entweder ein String mit einer URL / Pfad zu einem PDF Dokument sein
oder es ist auch möglich ein komplexeres Objekt zu machen:

```typescript
obj = {
    data: atob(base64String)
}
```
Hier habe ich mit dem `data`-Property direkt den base64 encoded String reingehängt. Das funktioniert,
muss aber noch die `atob()` Methode drüber laufen lassen.

Unser Ansatz funktioniert also gut.

Für weitere Details siehe: <a href="https://openbase.com/js/ng2-pdf-viewer/documentation#src">Doku ng2-pdf-viewer</a>

(Mit dieser Lib kann man auch Suchen im PDF, d.h. man könnte evtl. bei der Antwort dann den
gesuchten Begriff nochmal im FE im PDF suchen, dann sollte das Highlighting im PDF auch funktionieren...)

### 13.5. 14:10 Jan

Die base64 Verschlüsselung der einzelnen PDF-Seiten ist nun sichergestellt. Es wird ein anderes Gem
(combine_pdf) verwendet, um das PDF in die einzelnen Seiten zu splitten. Diese werden in einem tmp Folder
zwischengespeichert, dann während dem Eintragen in die Datenbank werden die einzelnen Seiten mit der 
`File` Klasse geöffnet, ins base64 kodiert und in der Datenbank abgelegt. Der tmp Folder wird nach dem 
Vorgang wieder gelöscht.

Im Zuge dieser Arbeiten habe ich auch Änderungen in den Rake Tasks gemacht:

- Der Rake Task für das MKB ist nun `poc:parse_mkb21`, der als Argument den Pfad
  zum MKB Pdf hat.
- Zum Ausführen also `rake poc:parse_mkb21['./beispiel/pfad']` in der Konsole ausführen.
  Wobei `./beispiel/pfad` mit dem richtigen Pfad zum PDF ersetzt werden muss.

Ansonsten bleibt alles gleich, insbesondere das Model.

### 9.5. 16:00

Route `poc/search` wurde erstellt & Controller für poc wurde angefangen. War mir noch nicht sicher 
wie man die Query mit dem Request an Elasticsearch senden muss. Direct calls oder via Model, welches noch zu machen wäre?

Beispiel `localhost:3000/de/poc/search?search=Laser&fragment_size=40`

### 8.5. 17:40 Jan

Erster Versuch, das Medizinische Kodierungshandbuch zu indexieren ist auf
"pse-2021-poc" gepusht.
Der Ansatz:
- Ein neues Model speichert in der Datenbank pro Seite im PDF jeweils den Text
  (der dann auch indexiert wird) und die PDF Seite als base64 encoded (noch nicht verifiziert) ab.
  
- Danach kann man mit ES Curls nach Schlagwörtern suchen (funktioniert!)

Zu beachten:
- rails db:migrate ausführen

- Das [Medizinische Kodierhandbuch](https://www.bfs.admin.ch/bfs/de/home/statistiken/kataloge-datenbanken/publikationen.assetdetail.14628240.html) 
  ist unter `medcodesearch/lib/assets` abgespeichert
  
- Pfad zu PDF anpassen in poc.rake

- im poc.rake file den task "Encode pdf" ausführen via `rake poc:encodePdf`

- mit curl eine Suche absetzten z.B.
```
(echo -n '
{
    "query": {
        "match": {
            "content_text_de": {
                "query": "Streptokokken"
            }
        }
    },
    "highlight": {
        "fields": {
            "content_text_de": {
                "fragment_size": 100     
            }                       
        }    
    }    
}    
') | curl -XGET 'localhost:9200/medcodesearch_mkb/_search?pretty=yes' -d @- > result
```

Speichert das Resultat in einem file "result" ab.
-> man kann im "query" Feld den Suchbegriff ändern und im "fragment_size" Feld ändern, 
wie viel Text ES zurückgeben soll.

TODO:
- Überprüfen, ob das base64 Feld in der Datenbank zurückkodiert zu einem PDF werden kann (damit sollen
  dann beispielsweise Tabellen angezeigt werden können...)
  
- Controller bauen, damit man über die API Suchanfragen schicken kann
- Überlegen, wie die Tabelle mit Meta-Daten enriched werden kann, damit Suchergebnisse
verbessert werden können (evtl. info, ob auf dieser Seite eine Tabelle ist?)
  
- Den Ansatz PDF als ganzes Indexieren ist auch noch auszuprobieren...

### 8.5 12:30 Tobi

Using Tika in Ruby

https://github.com/yomurb/yomu

https://stories.algolia.com/indexing-pdf-or-other-file-contents-for-searching-b2499c23568f

- funktioniert recht gut für Text. Tabellen in querformat failen (S. 51 - 54)
- Nicht querformat kommen gar nicht so schlecht (S. 90)



### 7.5 17:30 Sascha

Pdf in Base64 umwandeln in Ruby:
  ```bundle exec rails c
  => file = open("tmp/file.pdf")
  #> #File>File:tmp/receipts.pdf>
  => base_64 = Base64.encode64(file.read)
  #> "HVBGA6GF7sK....HAHUASH/askfohahau=\n"
```

### 7.5 16:50 Tobi

Wäre es möglich den Mapper (Vorgänger vom Ingest Attachment Plugin) zu verwenden? <br>
https://www.elastic.co/guide/en/elasticsearch/plugins/5.6/mapper-attachments.html

https://github.com/elastic/elasticsearch/tree/2.4/plugins/mapper-attachments


### 7.5. 15:27 Jan

Alles ein bisschen angeschaut -> denke müssen Ansätze noch ein bisschen
konkretisieren, d.h. wie wollen wir es genau ausprobieren? <br> Denn identifizierte Probleme:
* Backend von eonum verwendet ES Version 2.4.6, in dieser Version gibt es das "Ingest Attachment Plugin"
noch nicht, d.h. entweder müssen sie ES updaten oder wir müssen etwas anderes probieren...
  
* Ruby bietet auch keine direkte Integration des Plugins an, d.h. müssen hier den Umweg direkt
über ES machen (direkte calls)...
  
Ansonsten bin ich gerade noch am Ausprobieren, wie es mit dem "Ingest Attachment Plugin"
und der neuesten ES Version geht, da bin ich aber noch nicht sehr weit...

-> Noch TODO:

- Ansätze konkretisieren (was genau machen, technisch v.a.)
- Struktur im BE machen? (models, controller, etc.) -> weiss nicht ob sinnvoll...
- Weiter ausprobieren mit "Ingest Attachment Plugin"




