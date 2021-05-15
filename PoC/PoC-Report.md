# PoC-Report

## Data flow

1. The current webcrawler could be modified to crawl future sources


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

  #### pro:
  - no need for more models or controllers
  - easier to maintain


  #### con:

  - huge index
  - huge table


### no pain no gain:

#### pro:
- object oriented -> easier to debug and understand
- more options in FE: select sources
- queries can be optimized


#### con:

- more effort to maintain


## Technology

```
# used for parsing
gem 'pdf-reader'
gem 'iguvium'

# poc
gem 'yomu'
gem 'combine_pdf'
```

## Scalability: 

Our guess is that the approach of having only one table is flawed. 

#### arguments:

- Everytime a search is made the whole index has to be searched. 
- If something should corrupt the dataset the loss is bigger and takes more time to recover than having multiple tables containing only the data of one source.

Having multiple tables to maintain is more work. The minimum steps for each new pdf source are outlined here:

#### model

create model for new source.

```
bin/rails generate model newSource version:string text_de:text page_nr:integer page_base64:text
```

if document is available in multiple languages add more fields for text and the base64

```
bin/rails generate model newSource version:string text_de:text, text_fr:text page_nr:integer page_base64_de:text page_base64_fr:text
```

delete the following files if not needed:

- test/fixtures/newSource.yml

remove timestamp from migration if not needed
- t.timestamps

edit the model file:

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

#### raketask

edit the rake task to add the new source to the database and start indexing. 

??? WHICH LINES?

### controller 

here custom searching can be implemented. 
To add more than one index to the search, the following line has to be ajusted.

```
 def self.search_in(version, search, options={})
    options.merge! fields: %w(text)
    h = self.search_in_helper(version, search, options)
    self.search(h, [Klv1, Klv1Measure])
  end
```

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
