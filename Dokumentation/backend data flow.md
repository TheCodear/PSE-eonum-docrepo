# Backend Data Flow

In this file the data flow of a API request from the frontend to the backend is documented.

## Frontend to API

The service `../medcodesearch-frontend/src/app/service/catalog.service.ts` handles the API requests.
The methode search is responsible for the API call.

```
  /**
   * Searches in a specific version of the catalog for the specified
   * query.
   *
   * Returns an array of search results or an empty array if no results
   * matching the query are found.
   *
   * @param version the version of the catalog to use
   * @param query the query to search for
   */
  public async search(version: string, query: string): Promise<CatalogElement[]> {
    const types: string[] = this.config.searchableTypes;
    let results: CatalogElement[] = [];

    for (let i = 0; i < types.length; i++) {
      const type: string = types[i];
      try {
        const webResults = await this.getSearchForType(type, version, query);
        results = results.concat(webResults);
      } catch (e) {
        this.logger.error(e);
      }
    }
    return Promise.resolve(results);
  }
```

The helper method getSearchForType() is called by search() and handles the HTTP request with the given query:

```
private async getSearchForType(elementType: string, version: string, query: string): Promise<CatalogElement[]> {
    const url = `${this.baseUrl}${this.getLocale()}/${elementType}/${version}/search?highlight=1&search=${query}`;
    this.logger.http(url);

    return this.http.get(url).toPromise()
      .then(result => {
        const data: CatalogElement[] = result as CatalogElement[];
        data.forEach(element => {
          element.type = elementType;
        });
        return data;
      })
      .catch(reason => {
        throw new Error(reason);
      });
  }
```

## API

The backend handles all the endpoints for requests in the `../medcodesearch/config/routes.rb`. In there are all the routes listed, which can be called with an HTTP request. Here's a snippet:

```
Rails.application.routes.draw do
  root to: 'home#home'
  get 'documentation', to: 'home#documentation'

  scope '/:locale', :locale => /de|fr|it|en/, :format => /json|html/ do
    get 'chops/versions' => 'chops#versions'
    get 'chops/:version/search' => 'chops#search', constraints: { :version => /CHOP_[0-9]+/ }, as: :chop_search
    get 'chops/:version/:code' => 'chops#show', :as => 'chop', constraints: { :code => /[0-z.\-]+/ } # dash is also included for changing from ops -> chop
    get 'chop_chapters/:version/:code' => 'chops#show_chapter', :as => 'chop_chapter', constraints: { :code => /[0-z.]+/ }
    ...
```

After the `get` comes the route e.g. `chops/:version/search`, whereby the words with a colon in front are variable. After the route, the `=> chops#search` comes. This is the way you define a responsible methode which gets called in a specific file. The word infront of the # is a controller, so here `../controller/chops_controller.rb` has a method called `search`. You can also define constraints for the variables i nthe routes.

The `chops_controller.rb` uses from the helper class `../helper/search_helper.rb` the method `simple_search`. It also uses the `code_search_in` method from `../helper/searchable.rb`.
Both methods return an response from the database. The `search` method from the controller then uses these arrays to generate a formatted JSON-response, which acts as a reply to the frontend.

> This document will be extended upon a better understanding of the whole data flow.