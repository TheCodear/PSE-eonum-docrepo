# on mac use:
rails db:reseed\['data']

# **Datenbank Seeding anhand Beispiel icd_chapters**

**Schema für Darstellung der icd_chapters in der Datenbank.**

<img src="images\ICD.png" alt="ICD.png" style="zoom:67%;" />

​																		`\db\migrate\20160919085103_create_icd_chapters`

Dieses Schema wird beim Ausführen von `rails db:schema:load` automatisch in das Datenbankschema `schema.rb` übertragen.



**Modell für ICD**

<img src="images\ICD_Model.png" alt="ICD_Model.png" style="zoom:50%;" />

​																			`\models\icd.rb`

**ICDs in die Datenbank seeden**

<img src="images\seed_icd.png" alt="seed_icd.png" style="zoom:100%;" />

​																		`\db\seed_icd.rb`

1.  Die Versionen werden einzeln und manuell im File `icd_versions.rb`  (1) abgespeichert.

    `seed_icd.rb` läuft dann jedes File durch und seedet (2) diese in die Datenbank ein.

<img src="images\icd_versions.PNG" alt="icd_versions.png" style="zoom:75%;" />

​														`\db\seed_helpers\icd_versions.rb`

2. Mit Hilfe des Seedhelpers `claml_helpers.rb` werden die Dateien in die Datenbank geseedet. Insbesondere wird hier die Methode `seed_claml(model, model_group, model_chapter, version, main_language)` definiert, welche im Bild oben aufgerufen wird (2).

   Hier ein Beispiel, wie die Daten der Tabelle  `icd_chapters` in die Datenbank geseedet werden. 

   

   <img src="images\claml_helper.png" alt="claml_helper.png" style="zoom:75%;" />

   ​															`\db\seed_helpers\claml_helpers.rb`

   Zunächst wird ein Datenbankeintrag generiert, welche in der Datenbanktabelle bei `version`, `code` und `code_short` jeweils die Version eingetragen hat. 
   In einem Zweiten Teil, wird die jeweilige ICD vom Programm eingelesen und mit Hilfsmethoden aus `icd_helpers.rb` entsprechend geparsed. Anschliessend werden die entsprechenden Einträge ausgelesen und für jedes Kapitel ein `icd_chapter` Eintrag generiert. 



<img src="images\DB_Auszug.png" alt="DB_Auszug.png" style="zoom:100%;" />

​																		`Auszug aus der Datenbank Tabelle icd_chapters`

Eintrag 1 ist wie oben beschrieben der Eintrag, welcher am Anfang generiert wird. Die restlichen Einträge sind die jeweiligen Kapitel die folgen.
