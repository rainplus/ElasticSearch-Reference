## 数值类型 Numeric datatypes

The following numeric types are supported:

`long`| A signed 64-bit integer with a minimum value of `-263` and a maximum value of `263-1`.     
---|---    
`integer`| A signed 32-bit integer with a minimum value of `-231` and a maximum value of `231-1`.     
`short`| A signed 16-bit integer with a minimum value of `-32,768` and a maximum value of `32,767`.     
`byte`| A signed 8-bit integer with a minimum value of `-128` and a maximum value of `127`.     
`double`| A double-precision 64-bit IEEE 754 floating point.     
`float`| A single-precision 32-bit IEEE 754 floating point.     
`half_float`| A half-precision 16-bit IEEE 754 floating point.     
`scaled_float`| A floating point that is backed by a `long` and a fixed scaling factor.   
  
Below is an example of configuring a mapping with numeric fields:
    
    
    PUT my_index
    {
      "mappings": {
        "my_type": {
          "properties": {
            "number_of_bytes": {
              "type": "integer"
            },
            "time_in_seconds": {
              "type": "float"
            },
            "price": {
              "type": "scaled_float",
              "scaling_factor": 100
            }
          }
        }
      }
    }

![Note](/images/icons/note.png)

The `double`, `float` and `half_float` types consider that `-0.0` and `+0.0` are different values. As a consequence, doing a `term` query on `-0.0` will not match `+0.0` and vice-versa. Same is true for range queries: if the upper bound is `-0.0` then `+0.0` will not match, and if the lower bound is `+0.0` then `-0.0` will not match.

### Which type should I use?

As far as integer types (`byte`, `short`, `integer` and `long`) are concerned, you should pick the smallest type which is enough for your use-case. This will help indexing and searching be more efficient. Note however that given that storage is optimized based on the actual values that are stored, picking one type over another one will have no impact on storage requirements.

For floating-point types, it is often more efficient to store floating-point data into an integer using a scaling factor, which is what the `scaled_float` type does under the hood. For instance, a `price` field could be stored in a `scaled_float` with a `scaling_factor` of `100`. All APIs would work as if the field was stored as a double, but under the hood elasticsearch would be working with the number of cents, `price*100`, which is an integer. This is mostly helpful to save disk space since integers are way easier to compress than floating points. `scaled_float` is also fine to use in order to trade accuracy for disk space. For instance imagine that you are tracking cpu utilization as a number between `0` and `1`. It usually does not matter much whether cpu utilization is `12.7%` or `13%`, so you could use a `scaled_float` with a `scaling_factor` of `100` in order to round cpu utilization to the closest percent in order to save space.

If `scaled_float` is not a good fit, then you should pick the smallest type that is enough for the use-case among the floating-point types: `double`, `float` and `half_float`. Here is a table that compares these types in order to help make a decision.

Type | Minimum value | Maximum value | Significant bits / digits  
---|---|---|---  
`double`| `2-1074`| `(2-2-52)·21023`| `53` / `15.95`    
`float`| `2-149`| `(2-2-23)·2127`| `24` / `7.22`    
`half_float`| `2-24`| `65504`| `11` / `3.31`  
  
### Parameters for numeric fields

The following parameters are accepted by numeric types:

[`coerce`](coerce.html)| Try to convert strings to numbers and truncate fractions for integers. Accepts `true` (default) and `false`.     
---|---    
[`boost`](mapping-boost.html)| Mapping field-level query time boosting. Accepts a floating point number, defaults to `1.0`.     
[`doc_values`](doc-values.html)| Should the field be stored on disk in a column-stride fashion, so that it can later be used for sorting, aggregations,or scripting? Accepts `true` (default) or `false`.     
[`ignore_malformed`](ignore-malformed.html)| If `true`, malformed numbers are ignored. If `false` (default), malformed numbers throw an exception and reject thewhole document.     
[`include_in_all`](include-in-all.html)| Whether or not the field value should be included in the 
[`_all`](mapping-all-field.html) field? Accepts `true` or false`. Defaults to `false` if 
[`index`](mapping-index.html) is set to `false`, or if a parent 
[`object`](object.html)field sets `include_in_all` to `false`. Otherwise defaults to `true`.     
[`index`](mapping-index.html)| Should the field be searchable? Accepts `true` (default) and `false`.     
[`null_value`](null-value.html)| Accepts a numeric value of the same `type` as the field which is substituted for any explicit `null` values. Defaultsto `null`, which means the field is treated as missing.     
[`store`](mapping-store.html)| Whether the field value should be stored and retrievable separately from the 
[`_source`](mapping-source-field.html)field. Accepts `true` or `false` (default).   
  
### Parameters for `scaled_float`

`scaled_float` accepts an additional parameter:

`scaling_factor`| The scaling factor to use when encoding values. Values will be multiplied by this factor at index time and rounded to the closest long value. For instance, a `scaled_float` with a `scaling_factor` of `10` would internally store `2.34` as `23` and all search-time operations (queries, aggregations, sorting) will behave as if the document had a value of `2.3`. High values of `scaling_factor` improve accuracy but also increase space requirements. This parameter is required.   
---|---
