## Bool Query

A query that matches documents matching boolean combinations of other queries. The bool query maps to Lucene `BooleanQuery`. It is built using one or more boolean clauses, each clause with a typed occurrence. The occurrence types are:

Occur | Description  
---|---  
`must`| The clause (query) must appear in matching documents and will contribute to the score.    
`filter`| The clause (query) must appear in matching documents. However unlike `must` the score of the query will be ignored.Filter clauses are executed in [filter context](query-filter-context.html), meaning that scoring is ignored andclauses are considered for caching.    
`should`| The clause (query) should appear in the matching document. If the `bool` query is in a [query context]query-filter-context.html) and has a `must` or `filter` clause then a document will match the `bool` query even ifnone of the `should` queries match. In this case these clauses are only used to influence the score. If the `bool`query is a [filter context](query-filter-context.html) or has neither `must` or `filter` then at least one of the should` queries must match a document for it to match the `bool` query. This behavior may be explicitly controlled bysettings the [`minimum_should_match`](query-dsl-minimum-should-match.html) parameter.    
`must_not`| The clause (query) must not appear in the matching documents. Clauses are executed in [filter context]query-filter-context.html) meaning that scoring is ignored and clauses are considered for caching. Because scoring is ignored, a score of `0` for all documents is returned.  
  
![Important](/images/icons/important.png)

### Bool query in filter context

If this query is used in a filter context and it has `should` clauses then at least one `should` clause is required to match.

The bool query also supports `disable_coord` parameter (defaults to `false`) which changes how the `classic` similarity calculates the `bool` query’s score. Basically the coord similarity computes a score factor based on the fraction of all query terms that a document contains. See Lucene `BooleanQuery` for more details.

The `bool` query takes a _more-matches-is-better_ approach, so the score from each matching `must` or `should` clause will be added together to provide the final `_score` for each document.
    
    
    POST _search
    {
      "query": {
        "bool" : {
          "must" : {
            "term" : { "user" : "kimchy" }
          },
          "filter": {
            "term" : { "tag" : "tech" }
          },
          "must_not" : {
            "range" : {
              "age" : { "gte" : 10, "lte" : 20 }
            }
          },
          "should" : [
            { "term" : { "tag" : "wow" } },
            { "term" : { "tag" : "elasticsearch" } }
          ],
          "minimum_should_match" : 1,
          "boost" : 1.0
        }
      }
    }

### Scoring with `bool.filter`

Queries specified under the `filter` element have no effect on scoring — scores are returned as `0`. Scores are only affected by the query that has been specified. For instance, all three of the following queries return all documents where the `status` field contains the term `active`.

This first query assigns a score of `0` to all documents, as no scoring query has been specified:
    
    
    GET _search
    {
      "query": {
        "bool": {
          "filter": {
            "term": {
              "status": "active"
            }
          }
        }
      }
    }

This `bool` query has a `match_all` query, which assigns a score of `1.0` to all documents.
    
    
    GET _search
    {
      "query": {
        "bool": {
          "must": {
            "match_all": {}
          },
          "filter": {
            "term": {
              "status": "active"
            }
          }
        }
      }
    }

This `constant_score` query behaves in exactly the same way as the second example above. The `constant_score` query assigns a score of `1.0` to all documents matched by the filter.
    
    
    GET _search
    {
      "query": {
        "constant_score": {
          "filter": {
            "term": {
              "status": "active"
            }
          }
        }
      }
    }

### Using named queries to see which clauses matched

If you need to know which of the clauses in the bool query matched the documents returned from the query, you can use [named queries](search-request-named-queries-and-filters.html) to assign a name to each clause.
