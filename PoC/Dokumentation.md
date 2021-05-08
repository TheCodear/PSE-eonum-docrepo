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

### 7.5 16:50 Tobi

Wäre es möglich den Mapper (Vorgänger vom Ingest Attachment Plugin) zu verwenden? <br>
https://www.elastic.co/guide/en/elasticsearch/plugins/5.6/mapper-attachments.html

https://github.com/elastic/elasticsearch/tree/2.4/plugins/mapper-attachments

### 7.5 17:30 Sascha

Pdf in Base64 umwandeln in Ruby: 
  ```bundle exec rails c
  => file = open("tmp/file.pdf")
  #> #File>File:tmp/receipts.pdf>
  => base_64 = Base64.encode64(file.read)
  #> "HVBGA6GF7sK....HAHUASH/askfohahau=\n"
```

### 8.5 12:30 Tobi

Using Tika in Ruby

https://github.com/yomurb/yomu

https://stories.algolia.com/indexing-pdf-or-other-file-contents-for-searching-b2499c23568f

- funktioniert recht gut für Text. Tabellen in querformat failen (S. 51 - 54)
- Nicht querformat kommen gar nicht so schlecht (S. 90)
