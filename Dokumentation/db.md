# Anhang 1a der Krankenpflege-Leistungsverordnung (KLV)

Anhang 1a der KLV enthält die Liste der nach Artikel 3c KLV grundsätzlich ambulant durchzuführenden Eingriffe und die Kriterien zugunsten einer stationären Durchführung.


I. Liste der grundsätzlich ambulant durchzuführenden elektiven Eingriffe

II. Kriterien zugunsten einer stationären Durchführung

# klv1as

columnname | type | comment
--- | --- | ---
id | unique int| 
chapter|int|
chapter_title |text|necessary?
chop_2021|character varying|
text_de|text|
text_fr|text|
text_it|text|
additional_info|text|
version|character varying|

eg.
columnname | type | comment
--- | --- | ---
id | xy| 
chapter|3|
chapter_title |Einseitige Hernienoperationen|
chop_2021|53.00 |
text_de|Operation einer Inguinalhernie, nicht näher bezeichnet| 
text_fr|text|
text_it|text|
additional_info|Folgende elektiven Eingriffe sind nur dann grundsätzlich ambulant durchzuführen, wenn:a. sie eine einzige Körperseite betreffen;b. es sich nicht um eine Rezidivoperation handelt.|
version|character varying|

# klv1as_ambulant_chapters
columnname | type | comment
--- | --- | ---
id | unique int| 
chapter|character varying|
text_de|string|
text_fr|string|
text_it|string|
version|character varying|

eg.
columnname | type | comment
--- | --- | ---
id | xy| 
chapter|2|
text_de|Eingriffe an Hämorrhoiden|
text_fr|-|
text_it|-|
version|Ausgabe vom 1. Januar 2021|


# klv1as_stationary

columnname | type | comment
--- | --- | ---
id | unique int| 
chapter|character varying|
category|text| can be null
criteria|text|
additional_info|text|
ICD-10|string|
version|character varying|

eg. 

columnname | type | comment
--- | --- | ---
id | xy| 
chapter|2.1|
category|Fehlbildungen|
criteria|Angeborene Fehlbildungen am Herz-Kreislauf- und/oder Atmungssystem|
additional_info|text|Ein * am Ende eines ICD-10-Kodes in der letzten Spalte der Tabelle bedeutet, dass alle Kodes des bezeichneten Stamms (= Buchstabe und Zahl vor *) mit den allfälligen weiteren Stellen eingeschlossen sind.
ICD-10|Q20*–Q34*|
version|Ausgabe vom 1. Januar 2021|

# Krankenpflege-Leistungsverordnung KLV, Anhang 1

Der Anhang 1 der KLV enthält die hinsichtlich Wirksamkeit, Zweckmässigkeit und Wirtschaftlichkeit geprüften Leistungen, die von Ärzten und Ärztinnen oder Chiropraktoren und Chiorpraktorinnen erbrachten werden und bezeichnet die Kostenübernahme der obligatorischen Krankenpflegeversicherung. 

## title: klv1s

columnname | type | comment
--- | --- | ---
id | unique int| 
chapter | int| (1- 11) ?? fk?
subchapter | int| ?? fk?
subsubchapterid?|?|
chaptertitle_de | string| 
chaptertitle_fr | string| 
chaptertitle_it | string| 
version | string?| 
measure_de | string| can be null
measure_fr | string| can be null
measure_it | string| can be null
Obligation to perform |  boolean| can be null
requirements_de | string| 
requirements_fr | string| 
requirements_it | string| 
valid from | string?| 
					
eg.
columnname | type | comment
--- | --- | ---
id |xy|
chapter |1|
subchapter|4|
subsubchapterid?|2|
chaptertitle_de|Urologie und Proktologie|
chaptertitle_fr|-|
chaptertitle_it|-|
version |01.01.2021|
measure_de|Extrakorporale Stosswellenlithotripsie (ESWL), Nierensteinzertrümmerung|
measure_fr |-|
measure_it |-|
Obligation to perform|true|
requirements_de| Indikationen: ESWL eignet sich:a. bei Harnsteinen des Nierenbeckens, b. bei Harnsteinen des Nierenkelches, c. bei Harnsteinen des Ureters,falls die konservative Behandlung jeweils erfolglos geblieben ist und wegen der Lage, der Form und der Grösse des Steines ein Spontanabgang als unwahrscheinlich beurteilt wird. Die mit der speziellen Lagerung des Patienten oder der Patientin verbundenen erhöhten Risiken bei der Narkose erfordern eine besonders kompetente fachliche und apparative Betreuung während der Narkose (spezielle Ausbildung der Ärzte und Ärztinnen sowie der Narkosegehilfen und -gehilfinnen und adäquate Überwachungsgeräte).|
requirements_fr| -|
requirements_it	|-|	
valid from|22.8.1985/ 1.8.2006 	|


## title: klv1_chapters
columnname | type | comment
--- | --- | ---
id | unique int| 
chapter|int|
subchapter|int|
text_de|string|
text_fr|string|
text_it|string|
version|string|

eg.
columnname | type | comment
--- | --- | ---
id | xy| 
chapter|1|
subchapter|4|
text_de|Transplantationschirurgie|
text_fr|-|
text_it|-|
version|01.01.2021|


# Medizinisches Kodierhandbuch

## title: medcode_chapters

columnname | type | comment
--- | --- | ---
id | unique int| 
chapter|character varying|

eg.

columnname | type | comment
--- | --- | ---
id | xy| 
chapter|Allgemeine Kodierrichtlinien für Prozeduren P00 – P11 |

## title: medcode_subchapters

columnname | type | comment
--- | --- | ---
id | unique int| 
chapter|character varying| necessary????
subchapter|character varying|
title|text|

eg.

columnname | type | comment
--- | --- | ---
id | xy| 
chapter|Allgemeine Kodierrichtlinien für Prozeduren P00 – P11|
subchapter|P07a|
title|Bilaterale Operationen|
