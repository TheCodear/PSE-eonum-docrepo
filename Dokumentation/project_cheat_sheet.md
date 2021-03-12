# Projekt Cheat Sheet

Nützliches Know-How, Links, Pitfalls beim Projekt, etc. alles hier drin gesammelt.

## Links

### Ruby Cheat Sheets

http://www.testingeducation.org/conference/wtst3_pettichord9.pdf

[Ruby Quickref for Syntax etc.](https://www.zenspider.com/ruby/quickref.html)

[Cheat sheet for terminal](http://cheat.errtheblog.com/)

## Setup

### rails db:reseed['data']

Wenn bei diesem Befehl (dauert sehr lange!) folgender Fehler auftritt:
![img.png](img.png)
(bzw. statt medcodesearch_chops medcodesearch_icds)

Dann kann das folgendermassen gelöst werden:

`rails console`, dann kommt man in eine andere Konsolenumgebung und dann muss man `Chop.import(force: true)` bzw. 
`Icd.import(force: true)` ausführen. Danach kann man den reseed erneut machen und sollte nun durchlaufen.