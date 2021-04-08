The current db schema is found at

[db.schema](https://app.diagrams.net/#HTheCodear%2FPSE-eonum-docrepo%2Fmaster%2FDokumentation%2Fdiagrams%2FUntitled%20Diagram.drawio)

Draw.io is connected to this git repository


# Dummy Data


klv1 = Klv1.new(obligation: "Ja", req_de: "– bei einer klinisch manifesten Osteoporose und nach einem Knochenbruch bei inadäquatem Trauma – bei Langzeit-Cortisontherapie oder Hypogonadismus – Erkrankungen des Verdauungssystems mit Malabsorptionssyndrom (insbesondere Morbus Crohn, Colitis ulcerosa, Zöliakie) – primärer Hyperparathyreoïdismus (sofern keine klare Operationsindikation besteht) – Osteogenesis imperfecta – HIV – bei Therapie mit Aromatasehemmer (nach der Menopause) oder mit der Kombination GnRH- Analogon+Aromatasehemmer (vor der Menopause)", req_fr: "", req_it: "", version: "KLV1-V1-2021", valid_from:"1.3.1995/ 1.1.1999/ 1.7.2010 1.7.2012 1.1.1999/ 1.7.2010/ 1.1.2015 1.7.2019/ 1.4.2020", measure_id:"9.1.1")


klv1 = Klv1.new(obligation: "Ja", req_de: "Verlaufsuntersuchungen solange die prädisponierte Risikosituation besteht, in der Regel höchstens alle zwei Jahre.", req_fr: "", req_it: "", version: "KLV1-V1-2021", valid_from:"1.3.1995/ 1.1.1999/ 1.7.2010 1.7.2012 1.1.1999/ 1.7.2010/ 1.1.2015 1.7.2019/ 1.4.2020", measure_id:"9.1.2")

klv1.save



klv1chapter = Klv1Chapter.new(chapter_id:"9", subchapter_id:"9.1", chapter_de:"Radiologie", chapter_fr:"", chapter_it:"", subchapter_de:"Röntgendiagnostik", subchapter_fr:"", subchapter_it:"", version:"KLV1-V1-2021")

klv1chapter.save

klv1measure = Klv1Measure.new(position:"9.1.1", measure_de:"Knochendensitometrie mit Doppelenergie- Röntgen-Absorptiometrie (DEXA)", measure_fr:"", measure_it:"",version:"KLV1-V1-2021",chapter_id:"9.1")

klv1measure.save

klv1measure = Klv1Measure.new(position:"9.1.2", measure_de:"Knochendensitometrie mit Ganzkörper-Scanner", measure_fr:"", measure_it:"",version:"KLV1-V1-2021",chapter_id:"9.1")

klv1measure.save