## Lowercase Token Filter

A token filter of type `lowercase` that normalizes token text to lower case.

Lowercase token filter supports Greek, Irish, and Turkish lowercase token filters through the `language` parameter. Below is a usage example in a custom analyzer
    
    
    PUT /lowercase_example
    {
      "settings": {
        "analysis": {
          "analyzer": {
            "standard_lowercase_example": {
              "type": "custom",
              "tokenizer": "standard",
              "filter": ["lowercase"]
            },
            "greek_lowercase_example": {
              "type": "custom",
              "tokenizer": "standard",
              "filter": ["greek_lowercase"]
            }
          },
          "filter": {
            "greek_lowercase": {
              "type": "lowercase",
              "language": "greek"
            }
          }
        }
      }
    }
