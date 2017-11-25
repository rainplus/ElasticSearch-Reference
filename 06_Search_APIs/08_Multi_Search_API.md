## Multi Search API

The multi search API allows to execute several search requests within the same API. The endpoint for it is `_msearch`.

The format of the request is similar to the bulk API format and makes use of the newline delimited JSON (NDJSON) format. the structure is as follows (the structure is specifically optimized to reduce parsing if a specific search ends up redirected to another node):
    
    
    header\n
    body\n
    header\n
    body\n

 **NOTE** : the final line of data must end with a newline character `\n`. Each newline character may be preceded by a carriage return `\r`. When sending requests to this endpoint the `Content-Type` header should be set to `application/x-ndjson`.

The header part includes which index / indices to search on, optional (mapping) types to search on, the `search_type`, `preference`, and `routing`. The body includes the typical search body request (including the `query`, `aggregations`, `from`, `size`, and so on). Here is an example:
    
    
    $ cat requests
    {"index" : "test"}
    {"query" : {"match_all" : {} }, "from" : 0, "size" : 10}
    {"index" : "test", "search_type" : "dfs_query_then_fetch"}
    {"query" : {"match_all" : {} } }
    {}
    {"query" : {"match_all" : {} } }
    
    {"query" : {"match_all" : {} } }
    {"search_type" : "dfs_query_then_fetch"}
    {"query" : {"match_all" : {} } }
    
    
    $ curl -H "Content-Type: application/x-ndjson" -XGET localhost:9200/_msearch --data-binary "@requests"; echo

Note, the above includes an example of an empty header (can also be just without any content) which is supported as well.

The response returns a `responses` array, which includes the search response and status code for each search request matching its order in the original multi search request. If there was a complete failure for that specific search request, an object with `error` message and corresponding status code will be returned in place of the actual search response.

The endpoint allows to also search against an index/indices and type/types in the URI itself, in which case it will be used as the default unless explicitly defined otherwise in the header. For example:
    
    
    GET twitter/_msearch
    {}
    {"query" : {"match_all" : {} }, "from" : 0, "size" : 10}
    {}
    {"query" : {"match_all" : {} } }
    {"index" : "twitter2"}
    {"query" : {"match_all" : {} } }

The above will execute the search against the `twitter` index for all the requests that don’t define an index, and the last one will be executed against the `twitter2` index.

The `search_type` can be set in a similar manner to globally apply to all search requests.

The msearch’s `max_concurrent_searches` request parameter can be used to control the maximum number of concurrent searches the multi search api will execute. This default is based on the number of data nodes and the default search thread pool size.

### Security

See [_URL-based access control_](url-access-control.html "URL-based access control")

### Template support

Much like described in [_Search Template_](search-template.html "Search Template") for the _search resource, _msearch also provides support for templates. Submit them like follows:
    
    
    GET _msearch/template
    {"index" : "twitter"}
    { "inline" : "{ \"query\": { \"match\": { \"message\" : \"{ {keywords} }\" } } } }", "params": { "query_type": "match", "keywords": "some message" } }
    {"index" : "twitter"}
    { "inline" : "{ \"query\": { \"match_{ {template} }\": {} } }", "params": { "template": "all" } }

for inline templates.

You can also create search templates:
    
    
    POST /_search/template/my_template_1
    {
        "template": {
            "query": {
                "match": {
                    "message": "{ {query_string} }"
                }
            }
        }
    }
    
    
    POST /_search/template/my_template_2
    {
        "template": {
            "query": {
                "term": {
                    "{ {field} }": "{ {value} }"
                }
            }
        }
    }

and later use them in a _msearch:
    
    
    GET _msearch/template
    {"index" : "main"}
    { "id": "my_template_1", "params": { "query_string": "some message" } }
    {"index" : "main"}
    { "id": "my_template_2", "params": { "field": "user", "value": "test" } }