## Fuzzy Query

The fuzzy query uses similarity based on Levenshtein edit distance.

### String fields

The `fuzzy` query generates all possible matching terms that are within the maximum edit distance specified in `fuzziness` and then checks the term dictionary to find out which of those generated terms actually exist in the index.

Here is a simple example:
    
    
    GET /_search
    {
        "query": {
           "fuzzy" : { "user" : "ki" }
        }
    }

Or with more advanced settings:
    
    
    GET /_search
    {
        "query": {
            "fuzzy" : {
                "user" : {
                    "value" :         "ki",
                        "boost" :         1.0,
                        "fuzziness" :     2,
                        "prefix_length" : 0,
                        "max_expansions": 100
                }
            }
        }
    }

##### Parameters

`fuzziness`

| 

The maximum edit distance. Defaults to `AUTO`. See [Fuzziness.   
  
---|---  
  
`prefix_length`

| 

The number of initial characters which will not be “fuzzified”. This helps to reduce the number of terms which must be examined. Defaults to `0`.   
  
`max_expansions`

| 

The maximum number of terms that the `fuzzy` query will expand to. Defaults to `50`.   
  
![Warning](images/icons/warning.png)

This query can be very heavy if `prefix_length` is set to `0` and if `max_expansions` is set to a high number. It could result in every term in the index being examined!