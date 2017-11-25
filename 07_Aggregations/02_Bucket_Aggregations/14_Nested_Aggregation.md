## Nested Aggregation

A special single bucket aggregation that enables aggregating nested documents.

For example, lets say we have a index of products, and each product holds the list of resellers - each having its own price for the product. The mapping could look like:
    
    
    {
        ...
    
        "product" : {
            "properties" : {
                "resellers" : { ![](images/icons/callouts/1.png)
                    "type" : "nested",
                    "properties" : {
                        "name" : { "type" : "text" },
                        "price" : { "type" : "double" }
                    }
                }
            }
        }
    }

![](images/icons/callouts/1.png)

| 

The `resellers` is an array that holds nested documents under the `product` object.   
  
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

Response:
    
    
    {
        "aggregations": {
            "resellers": {
                "min_price": {
                    "value" : 350
                }
            }
        }
    }