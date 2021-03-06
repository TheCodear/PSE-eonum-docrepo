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


### divide and conquer:

every new source has an own model, containing data of multiple languages if available. 

#### pro:
- object oriented -> easier to debug and understand
- more options in FE: select sources
- queries can be optimized


#### con:

- more effort to maintain
- db might be inconsistent eg. page 112 of the german version might show different information than page 112 of the french version.
If this is a deal breaker a different approach could be to have 3 models per document each only containing data of 1 langauge or adding checks to the parser. 

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
bin/rails generate model newSource version:string, page_nr:integer, text_de:text, text_fr:text, text_it:text, page_base64_de:text page_base64_fr:text, page_base64_it:text
```

delete the following files if not needed:

- test/fixtures/newSource.yml

remove timestamp from migration if not needed
- t.timestamps

edit the model file:

- indexed fields


```
 # avoid conflicts with other application that index the same model
  index_name    "medcodesearch_newsource"

  extend Searchable

  settings analysis: Searchable::CUSTOM_ANALYZERS do
    mappings dynamic: 'false' do
      indexes :version, type: :string, index: :not_analyzed
      Searchable.indexes_internationalized(self, %w(text), %w(de fr it))
    end
  end

```

### poc.rake


start the rake task with (Important: no spaces inbetween files):

```
rake poc:parse_mkb21["beispiel/pfad/deutsch","beispiel/pfad/französisch","beispiel/pfad/italienisch"]
```

For new sources add a new rake task or modify the exsting one (parse_mkb21). 

### pocs_controller.rb

here custom searching could be implemented. 

To add more than one index to the search, the following line has to be adjusted.

```
@results = Mkb.search_multi_pdf("2021", search_term, [Mkb], fragment_size ,
```

```
include SearchHelper

  def search
    # Set some defaults
    search_term = params[:search] || 'Streptokokken'
    fragment_size = params[:fragment_size] || 100
    @results = Mkb.search_multi_pdf("MEDKHB-2021", search_term, [Mkb], fragment_size, params[:locale],
                                    ngramed: params[:ngramed] == '1')
    render json: assemble_pdf_search_results(@results, params)
  end

  def get_single_page
    id = params[:id] || 1
    entry = Mkb.find(id)
    render json: { :page => entry[page_base64].gsub("\n", '') }
  end

  def page_base64
    ("page_base64_#{I18n.locale}").to_sym
  end

```


### searchable.rb

The following additions were made to searchable.rb 

Reasoning: 
- Current code is not affected in search quality.
- Fragment size can be modified

```
 def search_multi_pdf(version, search_term, models, fragment_size, locale, options={})
    options.merge! fields: %w(text)
    h = self.search_full_text(search_term, fragment_size, locale)
    Elasticsearch::Model.search(h, models)
  end
```

```
  def search_full_text(search_term, fragment_size, locale)
    {
      "query": {
        "match": {
          "text_#{locale}": {
            "query": search_term,
            "analyzer": LANGUAGE_ANALYZERS[locale.to_sym].to_s,
            "fuzziness": "AUTO"
          }
        }
      },
      "highlight": {
        "fields": {
          "text_#{locale}": {
            "fragment_size": fragment_size
          }
        }
      }
    }
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


### known issuse

- text in tabels (especially landscape format) can not be extracted properly -> therefore the text in the table can not be indexed properly or seached for
- elasticsearch query could be optimized (field boost etc.) -> if exsisting functions of BE could be used, then this issue will be resolved
- Db inconsistencies (eg.  page 152 of german version might not show the same thing as page 152 of french version). This could be resolved by using 3 Models for each new source. Each containing only data of one language
- Highlight in pdf matches the color of changed text in Medizinisches Kodierhandbuch. Look into wether the color of the highlight is changable 


