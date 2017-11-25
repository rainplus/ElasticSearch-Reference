## Has Child Query

The `has_child` filter accepts a query and the child type to run against, and results in parent documents that have child docs matching the query. Here is an example:
    
    
    GET /_search
    {
        "query": {
            "has_child" : {
                "type" : "blog_tag",
                "query" : {
                    "term" : {
                        "tag" : "something"
                    }
                }
            }
        }
    }

#### Scoring capabilities

The `has_child` also has scoring support. The supported score modes are `min`, `max`, `sum`, `avg` or `none`. The default is `none` and yields the same behaviour as in previous versions. If the score mode is set to another value than `none`, the scores of all the matching child documents are aggregated into the associated parent documents. The score type can be specified with the `score_mode` field inside the `has_child` query:
    
    
    GET /_search
    {
        "query": {
            "has_child" : {
                "type" : "blog_tag",
                "score_mode" : "min",
                "query" : {
                    "term" : {
                        "tag" : "something"
                    }
                }
            }
        }
    }

#### Min/Max Children

The `has_child` query allows you to specify that a minimum and/or maximum number of children are required to match for the parent doc to be considered a match:
    
    
    GET /_search
    {
        "query": {
            "has_child" : {
                "type" : "blog_tag",
                "score_mode" : "min",
                "min_children": 2, ![](images/icons/callouts/1.png)
                "max_children": 10, ![](images/icons/callouts/2.png)
                "query" : {
                    "term" : {
                        "tag" : "something"
                    }
                }
            }
        }
    }

![](images/icons/callouts/1.png) ![](images/icons/callouts/2.png)

| 

Both `min_children` and `max_children` are optional.   
  
---|---  
  
The `min_children` and `max_children` parameters can be combined with the `score_mode` parameter.

#### Ignore Unmapped

When set to `true` the `ignore_unmapped` option will ignore an unmapped `type` and will not match any documents for this query. This can be useful when querying multiple indexes which might have different mappings. When set to `false` (the default value) the query will throw an exception if the `type` is not mapped.

#### Sorting

Parent documents can’t be sorted by fields in matching child documents via the regular sort options. If you need to sort parent document by field in the child documents then you can should use the `function_score` query and then just sort by `_score`.

Sorting blogs by child documents' `click_count` field:
    
    
    GET /_search
    {
        "query": {
            "has_child" : {
                "type" : "blog_tag",
                "score_mode" : "max",
                "query" : {
                    "function_score" : {
                        "script_score": {
                            "script": "_score * doc['click_count'].value"
                        }
                    }
                }
            }
        }
    }