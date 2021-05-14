# PoC-Report

## Data flow

-> Take pdf 

-> split pdf into single pages 

-> extract text and convert single page to base64 

-> safe to db

-> index text



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

- controller
