## Nested Aggregation

A special single bucket aggregation that enables aggregating nested documents.

For example, lets say we have a index of products, and each product holds the list of resellers - each having its own price for the product. The mapping could look like:
    
    
    {
        ...
    
        "product" : {
            "properties" : {
                "resellers" : { <1>
                    "type" : "nested",
                    "properties" : {
                        "name" : { "type" : "text" },
                        "price" : { "type" : "double" }
                    }
                }
            }
        }
    }

<1>| The `resellers` is an array that holds nested documents under the `product` object.     
---|---  
  
The following aggregations will return the minimum price products can be purchased in:
    
    
    {
        "query" : {
            "match" : { "name" : "led tv" }
        },
        "aggs" : {
            "resellers" : {
                "nested" : {
                    "path" : "resellers"
                },
                "aggs" : {
                    "min_price" : { "min" : { "field" : "resellers.price" } }
                }
            }
        }
    }

As you can see above, the nested aggregation requires the `path` of the nested documents within the top level documents. Then one can define any type of aggregation over these nested documents.

响应如下：
    
    
    {
        "aggregations": {
            "resellers": {
                "min_price": {
                    "value" : 350
                }
            }
        }
    }
