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

### [!!!] Index does not exist (Elasticsearch::Transport::Transport::Errors::NotFound)

Normalerweise wird bei diesem Fehler zusätzlich noch angegeben, bei welchem Model der Fehler auftritt (Chop, Adrg, Drg,
Tarmed, Icd). Mit dieser zusätzlichen Information kann folgendes getan werden:

Gehe mit `rails console` in die Rails Konsolenumgebung, dort gebe folgende Befehl(e) ein (angenommen der Fehler bezieht
sich auf Drg):

```
Drg.__elasticsearch__.create_index!
Drg.__elasticsearch__.create_index!(force: true)
Drg.import(force: true)
```

Manchmal reicht es, nur den ersten Befehl auszuführen, manchmal braucht es auch noch die force Option, bzw. den import
Befehl mit der force Option.