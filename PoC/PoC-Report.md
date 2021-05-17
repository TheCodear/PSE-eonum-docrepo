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





## Philosophy

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

start the rake task with:

```
rails poc:parse_mkb21['./beispiel/pfad']
```

### controller 

here custom searching can be implemented. 
MISSING: REGEX ADD MODELS TO QUERY
To add more than one index to the search, the following line has to be ajusted.

```
@results = Mkb.search_multi_pdf("2021", search_term, [Mkb], fragment_size ,
```

```
class PocsController < ApplicationController
  include SearchHelper

  def search
    # Set some defaults
    search_term = params[:search] || 'Streptokokken'
      fragment_size = params[:fragment_size] || 100
    @results = Mkb.search_multi_pdf("2021", search_term, [Mkb], fragment_size ,
                                    ngramed: params[:ngramed] == '1', locale: params[:locale])
    render json: assemble_pdf_search_results(@results, params)
  end

  def get_single_page
    id = params[:id] || 1
    entry = Mkb.find(id)
    render json: { :page => entry[:page_base64].gsub("\n", '') }
  end


end


```



### searchable.rb

The following additions were made to searchable.rb 

Reasoning: 
- Current code is not affected in search quality.
- Fragment size can be modified

```
  def search_multi_pdf(version, search_term, models, fragment_size, options={})
    options.merge! fields: %w(text)
    h = self.search_full_text(search_term, fragment_size)
    Elasticsearch::Model.search(h, models)
  end
```

MISSING: AUTO ANALAYZER 

```
  def search_full_text(search_term, fragment_size)
    query = {
               "query": {
                 "match": {
                   "content_text_de": {
                     "query": search_term,
                     "analyzer": "german",
                     "fuzziness": "AUTO"
                   }
                 }
               },
               "highlight": {
                 "fields": {
                   "content_text_de": {
                     "fragment_size": fragment_size
                   }
                 }
               }
             }
    return query
  end
```

### search_helper.rb
The following additions were made to search_helper.rb 
Reasoning: 
- data is strucutred very differently compared to code based sources

```
  # assembles the search results of pdf-document search
  def assemble_pdf_search_results(results, params)
    max_results = params[:max_results].to_i || 5
    json = []
    results.results.to_a[0..max_results-1].each do |res|
      id = res['_source']['id']
      page_nr = res['_source']['page_nr']
      version = res['_source']['version']
      res['highlight']['content_text_de'].each do |hit|
        entry = {'id' => id, 'hit' => hit, 'page_nr' => page_nr, 'version' => version}
        json << entry
      end
    end
    json
  end
```
