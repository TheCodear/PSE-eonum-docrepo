# Projekt Cheat Sheet

Nützliches Know-How, Links, Pitfalls beim Projekt, etc. alles hier drin gesammelt.

## Links

### Ruby Cheat Sheets

http://www.testingeducation.org/conference/wtst3_pettichord9.pdf

[Ruby Quickref for Syntax etc.](https://www.zenspider.com/ruby/quickref.html)

[Cheat sheet for terminal](http://cheat.errtheblog.com/)

[Basic tutorial for haml](http://haml.info/tutorial.html)



#### Ruby Controller

- Logical center of application, coordinates interaction between user, views and model. 
	- routs external requests to internal actions
	- manages helpers which extend the capabilities of the view templates
	- manages sessions 
```
rails generate controller Book
```


#### Helpers

search_helper.rb

is included in all controllers

```
include SearchHelper
```

highlights keywords

#### Views

json is handled by frontend


#### Models

Each entity (such as book) gets a table in the database named after it, but in the plural (books).

Notice that you are capitalizing Book and Subject and using the singular form. This is a Rails paradigm that you should follow each time you create a model.

```
ruby script/generate model Book
```

create connection between models:

has_one, has_many, belongs_to, and has_and_belongs_to_many.

[More Model](https://www.tutorialspoint.com/ruby-on-rails/rails-models.htm)


#### Migrations

```
rails generate migration table_name
 
export RAILS_ENV = development
//
export RAILS_ENV = test 

rake db:migrate
```

### Migration How-to's
[Source](guides.rubyonrails.org/active_record_migrations)

#### How to generate a migration file:
	rails g migration AppropriateName
This will generate you a file like this:
```
class AppropriateName < ActiveRewcord::Migration[6.0]
  def change
  end
end
```

#### Add new column to a XY-db in ruby:
	rails g migration Add"Column"To"Table" newcolumn1:text newcolumn2:string etc.
	rails db:migrate
it creates a migration file, which defines a new change method within it, so the existing db-table will be updated

Remove Column from table in ruby:
rails g migration Remove"Column"From"Table" column:string


#### Rename a table name in ruby:
create a migration file like this:
```
class RenameOldTableToNewTable < ActiveRecord::Migration
  def change
    rename_table :old_table_name, :new_table_name
  end
end
```

#### How to delete a db-table in ruby:
Create a migration file like this:
```
class DropTablename < ActiveRecord::Migration
  def change
    drop_table :products
  end
end
```
You can also go into the terminal and do the following:

	rails db //opens up the db console
	drop table Tablename;
	\q

#### Joint tables etc
tbe

### How to create a model & finally get a JSON response
Rails console:

	rails g controller Klv1s index --skip-routes

Create route:

	get 'klv1s', to: 'klv1s#index'

Create model:

	rails g model Klv1 code:string text_de:text text_fr:text text_it:text version:string
	rails db:migrate

Add content to model: Rails console:

	klv1 = Klv1.new(code: "1291.2021", text_de: "Die Schweiz ist jung", text_fr: "Bonjour monsieur Bonaparte", text_it: "Quanta costa, mozarella pizza", version: "CHE-21")
	klv1 = Klv1.new(code: "1291.2021.v2", text_de: "Corona ist ein Bier", text_fr: "Bonjour, du Baguette", text_it: "Leonardo DaVinci", version: "CHE-22")
	klv1.save

Handle request in controller and generate output:

	class Klv1sController < ApplicationController
	  def index
	    @klv1s = Klv1.all
	    render json: @klv1s
	  end
	end

## Git Cheatsheet

### Branches

#### Go into branch

	git ls-remote 			//shows you all branches
	git fetch --all 		//fetches all branches
	git checkout BranchName		//goes into BranchName

#### Create new branch

	git checkout -b YourNewBranch DevBranch //Creates a new subbranch of the DevBranch

#### Push changes to your subbranch

	git commit -am "Your commit message"	//Commits all added changes to your current branch
	git push origin YourSubBranch		//Pushes the commited changes to the defined branch

#### Delete local branch

	git branch -D BranchName	//Deletes defined branch

#### Create Push request / merge with branch above

tbe

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
