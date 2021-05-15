# PoC-Report

## Data flow

1. Add new source to webcrawler?
Source is can be safed in directory of choice

2. Run Rake Task to 

- split pdf into single pages 

- extract text and convert single page to base64 

- safe to db

- index text



FE -> search request

BE -> translate search request -> query db -> sends back highlighted results + id of entry

FE -> clicking on result will send request with id of entry to backend 

BE -> sends back base64 of pdf -> can be displayed using pdf reader





## philosophy

### one table to rule them all:

  every indexed document lives in the same db table. elasticsearch searches through all text in 1 index.

  ### pro:
  - no need for more models or controllers
  - easier to maintain


  ### con:

  - huge index
  - huge table


### more controll:

### pro:
- object oriented
- more options in FE: select sources
- queries can be optimized


### con:

- more effort to maintain


## Technology

- pdf gem

## Scalability

For each new pdf source the following steps have to be made:

- model

create model for new source.

- indexed fields
- short entry

```
class Mkb < ApplicationRecord

  include Elasticsearch::Model
  include Rails.application.routes.url_helpers

  # avoid conflicts with other application that index the same model
  index_name    "medcodesearch_mkb"

  extend Searchable

  settings analysis: Searchable::CUSTOM_ANALYZERS do
    mappings dynamic: 'false' do
      indexes :version, type: :string, index: :not_analyzed
      Searchable.indexes_internationalized(self, %w(content_text), %w(de))
    end
  end

  def short_entry
    {page_nr: self.page_nr, content_text_de: self.content_text_de, page_base64: page_base64}
  end


end

```

- db

create a new db table for the new document containing at least the following rows:

- version
- page_nr
- page_base64
- content_text_de

if document is available in multiple languages add more fields in db plus add different indexes in model

```
class CreateMkbs < ActiveRecord::Migration[6.0]
  def change
    create_table :mkbs do |t|
      t.string :version
      t.integer :page_nr
      t.text :content_text_de
      t.text :page_base64
    end
  end
end
```

- controller 


```
class PocsController < ApplicationController
  include SearchHelper

  def search
    # Set some defaults
    search_term = params[:search] || 'Streptokokken'
    fragment_size = params[:fragment_size] || 100
    @results = Mkb.search_full_text(search_term, fragment_size)

    #render json: @results
    render json: assemble_fulltext_search_results(@results, params)
  end


end


```
