## `fielddata`

Most fields are [indexed](mapping-index.html) by default, which makes them searchable. Sorting, aggregations, and accessing field values in scripts, however, requires a different access pattern from search.

Search needs to answer the question _"Which documents contain this term?"_ , while sorting and aggregations need to answer a different question: _"What is the value of this field for **this** document?"_.

Most fields can use index-time, on-disk [`doc_values`](doc-values.html) for this data access pattern, but [`text`](text.html) fields do not support `doc_values`.

Instead, `text` fields use a query-time **in-memory** data structure called `fielddata`. This data structure is built on demand the first time that a field is used for aggregations, sorting, or in a script. It is built by reading the entire inverted index for each segment from disk, inverting the term ↔︎ document relationship, and storing the result in memory, in the JVM heap.

### Fielddata is disabled on `text` fields by default

Fielddata can consume a **lot** of heap space, especially when loading high cardinality `text` fields. Once fielddata has been loaded into the heap, it remains there for the lifetime of the segment. Also, loading fielddata is an expensive process which can cause users to experience latency hits. This is why fielddata is disabled by default.

If you try to sort, aggregate, or access values from a script on a `text` field, you will see this exception:

> Fielddata is disabled on text fields by default. Set `fielddata=true` on [`your_field_name`] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory.

### Before enabling fielddata

Before you enable fielddata, consider why you are using a `text` field for aggregations, sorting, or in a script. It usually doesn’t make sense to do so.

A text field is analyzed before indexing so that a value like `New York` can be found by searching for `new` or for `york`. A `terms` aggregation on this field will return a `new` bucket and a `york` bucket, when you probably want a single bucket called `New York`.

Instead, you should have a `text` field for full text searches, and an unanalyzed [`keyword`](keyword.html) field with [`doc_values`](doc-values.html) enabled for aggregations, as follows:
    
    
    PUT my_index
    {
      "mappings": {
        "my_type": {
          "properties": {
            "my_field": { <1>
              "type": "text",
              "fields": {
                "keyword": { <2>
                  "type": "keyword"
                }
              }
            }
          }
        }
      }
    }

<1>| Use the `my_field` field for searches.     
---|---    
<2>| Use the `my_field.keyword` field for aggregations, sorting, or in scripts.   
  
### Enabling fielddata on `text` fields

You can enable fielddata on an existing `text` field using the [PUT mapping API](indices-put-mapping.html) as follows:
    
    
    PUT my_index/_mapping/my_type
    {
      "properties": {
        "my_field": { <1>
          "type":     "text",
          "fielddata": true
        }
      }
    }

<1>| The mapping that you specify for `my_field` should consist of the existing mapping for that field, plus the `fielddata` parameter.     
---|---  
  
![Tip](/images/icons/tip.png)

The `fielddata.*` parameter must have the same settings for fields of the same name in the same index. Its value can be updated on existing fields using the [PUT mapping API](indices-put-mapping.html).

 **Global ordinals**

Global ordinals is a data-structure on top of fielddata and doc values, that maintains an incremental numbering for each unique term in a lexicographic order. Each term has a unique number and the number of term _A_ is lower than the number of term _B_. Global ordinals are only supported on [`text`](text.html) and [`keyword`](keyword.html) fields.

Fielddata and doc values also have ordinals, which is a unique numbering for all terms in a particular segment and field. Global ordinals just build on top of this, by providing a mapping between the segment ordinals and the global ordinals, the latter being unique across the entire shard.

Global ordinals are used for features that use segment ordinals, such as sorting and the terms aggregation, to improve the execution time. A terms aggregation relies purely on global ordinals to perform the aggregation at the shard level, then converts global ordinals to the real term only for the final reduce phase, which combines results from different shards.

Global ordinals for a specified field are tied to _all the segments of a shard_ , while fielddata and doc values ordinals are tied to a single segment. which is different than for field data for a specific field which is tied to a single segment. For this reason global ordinals need to be entirely rebuilt whenever a once new segment becomes visible.

The loading time of global ordinals depends on the number of terms in a field, but in general it is low, since it source field data has already been loaded. The memory overhead of global ordinals is a small because it is very efficiently compressed.

### `fielddata_frequency_filter`

Fielddata filtering can be used to reduce the number of terms loaded into memory, and thus reduce memory usage. Terms can be filtered by _frequency_ :

The frequency filter allows you to only load terms whose document frequency falls between a `min` and `max` value, which can be expressed an absolute number (when the number is bigger than 1.0) or as a percentage (eg `0.01` is `1%` and `1.0` is `100%`). Frequency is calculated **per segment**. Percentages are based on the number of docs which have a value for the field, as opposed to all docs in the segment.

Small segments can be excluded completely by specifying the minimum number of docs that the segment should contain with `min_segment_size`:
    
    
    PUT my_index
    {
      "mappings": {
        "my_type": {
          "properties": {
            "tag": {
              "type": "text",
              "fielddata": true,
              "fielddata_frequency_filter": {
                "min": 0.001,
                "max": 0.1,
                "min_segment_size": 500
              }
            }
          }
        }
      }
    }
