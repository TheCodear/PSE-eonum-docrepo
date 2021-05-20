# PoC-Report

## Data flow

1. For crawling the new sources, some effort needs to be put into building a
crawler. E.g. for the 'Medizinisches Kodierhandbuch' source the website to be
   crawled is quite different from the KLV1 page (BAG vs. BFS), this means that
   the KLV1-Crawler cannot be reused so easily. The best way to go would probably be
   to introduce an abstract crawler, which unifies the common behaviour of each 
   crawler and then for each source generate a subclass. But for MKB nothing has been
   done during PoC. (Alternatively the crawling could also be put into the rake
   task...).
   

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

every new source has an own model, containing data of multiple languages if available. 

#### pro:
- object oriented -> easier to debug and understand
- more options in FE: select sources
- queries can be optimized


#### con:

- more effort to maintain
- db might be inconsistent eg. page 112 of the german version might show different information than page 112 of the french version.
If this is a deal breaker a different approach could be to haave 3 models per document each only containing data of 1 langauge. 

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

### arguments:

- Everytime a search is made the whole index has to be searched. 
- If something should corrupt the dataset the loss is bigger and takes more time to recover than having multiple tables containing only the data of one source.

Having multiple tables to maintain is more work. The minimum steps for each new pdf source are outlined here:



### model

create model for new source.

```
bin/rails generate model newSource version:string text_de:text, text_fr:text, text_it:text, page_nr:integer page_base64_de:text page_base64_fr:text, page_base64_it:text
```

delete the following files if not needed:

- test/fixtures/newSource.yml

remove timestamp from migration if not needed
- t.timestamps

edit the model file:

- indexed fields
- short entry


```
class NewSource < ApplicationRecord

  include Elasticsearch::Model
  include Rails.application.routes.url_helpers

  # avoid conflicts with other application that index the same model
  index_name    "medcodesearch_newSource"

  extend Searchable

  settings analysis: Searchable::CUSTOM_ANALYZERS do
    mappings dynamic: 'false' do
      indexes :version, type: :string, index: :not_analyzed
      Searchable.indexes_internationalized(self, %w(text), %w(de fr it))
    end
  end

 
end

```

### poc.rake

edit the rake task to add the new source to the database and start indexing. 

MISSING AUTO LOCALE -> text_{locale} in db

start the rake task with:

```
rails poc:parse_mkb21['./beispiel/pfad/newSource.pdf']
```

### pocs_controller.rb

here custom searching can be implemented. 

MISSING: REGEX ADD MODELS TO QUERY

To add more than one index to the search, the following line has to be adjusted.

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
    render json: { :page => entry[page_base64].gsub("\n", '') }
  end

  def page_base64
    return ("page_base64_#{I18n.locale}").to_sym
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

MISSING: AUTO LANGUAGE FOR TABLE ROW AND ANALYZER: pass locale as argument in search_full_text, so table row and analyzer get adjusted accordingly

Resoning: MISSING

```
  def search_full_text(search_term, fragment_size)
    query = {
               "query": {
                 "match": {
                   "text_de": {
                     "query": search_term,
                     "analyzer": "german",
                     "fuzziness": "AUTO"
                   }
                 }
               },
               "highlight": {
                 "fields": {
                   "text_de": {
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
      res['highlight']['text_de'].each do |hit|
        entry = {'id' => id, 'hit' => hit, 'page_nr' => page_nr, 'version' => version}
        json << entry
      end
    end
    json
  end
```

## Demo-Frontend

For exploring if the designed solution also works in the frontend, we created a small
demo project as throw-away prototype, which simply sends search request to the Backend
and presents the results to the user. The FE also renders the PDF page when clicking on 
a search result.

This demo project showed, that our designed solution works full stack, including the base64
encoding, which can easily be rendered directly as PDF by the used pdf-viewer library.

The Angular demo project can be checked out <a href=https://github.com/TheCodear/ng-demo-pdfviewer>here</a>.
More details are also provided directly in the project.
