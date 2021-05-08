# PoC - Hauptdokumentation

## Ansätze

* Indexiere ganzes Pdf und schauen, was zurückkommt.
* Pdf in einzelne Abschnitte trennen und schaue, was zurück kommt.
* Benutze Highlight um Fragmentgrösse zu definieren und schaue, wie
  viel zurück kommt.
* Kompromiss wäre, Pdf als ganzes zu durchsuchen und beschränkte
  Seitenzahl mit übereinstimmenden Anfragen auf Tabelle durchführen. Wie verhälts sich diese?
  
TODO: weiter verfeinern!
  
## Entscheide / fixe Outcomes

...

## Activity Log

### 8.5. 17:40 Jan

Erster Versuch, das Medizinische Kodierungshandbuch zu indexieren ist auf
"pse-2021-poc" gepusht.
Der Ansatz:
- Ein neues Model speichert in der Datenbank pro Seite im PDF jeweils den Text
  (der dann auch indexiert wird) und die PDF Seite als base64 encoded (noch nicht verifiziert) ab.
  
- Danach kann man mit ES Curls nach Schlagwörtern suchen (funktioniert!)

Zu beachten:
- rails db:migrate ausführen

- im poc.rake file den task "Encode pdf" ausführen

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




