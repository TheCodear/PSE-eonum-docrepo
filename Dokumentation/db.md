Anhang 1ader Krankenpflege-Leistungsverordnung (KLV)

klv_attachment_1a

db schema:



klv_attachment_1a

db schema:

ambulant
	copy db schema of chop
	copy db schema of chop_chapters

stationär
	copy db schema of icd
	copy db schema of icd_chapters
	

#Krankenpflege-Leistungsverordnung KLV, Anhang 1

title: klv_attachment_1

db schema:
columnname | type | comment
--- | --- | ---
id | unique int| 
chapter | int| (1- 11)
subchapter | int| 
chaptertitle_de | string| 
chaptertitle_fr | string| 
chaptertitle_it | string| 
version | string?| 
measure_de | string| 
measure_fr | string| 
measure_it | string| 
Obligation to perform |  boolean| 
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



store chaptertitle info in different table? similar to icd_chapters?

---
