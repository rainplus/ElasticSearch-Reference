## Get API

The get API allows to get a typed JSON document from the index based on its id. The following example gets a JSON document from an index called twitter, under a type called tweet, with id valued 0:
    
    
    GET twitter/tweet/0

The result of the above get operation is:
    
    
    {
        "_index" : "twitter",
        "_type" : "tweet",
        "_id" : "0",
        "_version" : 1,
        "found": true,
        "_source" : {
            "user" : "kimchy",
            "date" : "2009-11-15T14:12:12",
            "likes": 0,
            "message" : "trying out Elasticsearch"
        }
    }

The above result includes the `_index`, `_type`, `_id` and `_version` of the document we wish to retrieve, including the actual `_source` of the document if it could be found (as indicated by the `found` field in the response).

The API also allows to check for the existence of a document using `HEAD`, for example:
    
    
    HEAD twitter/tweet/0

### Realtime

By default, the get API is realtime, and is not affected by the refresh rate of the index (when data will become visible for search). If a document has been updated but is not yet refreshed, the get API will issue a refresh call in-place to make the document visible. This will also make other documents changed since the last refresh visible. In order to disable realtime GET, one can set the `realtime` parameter to `false`.

### Optional Type

The get API allows for `_type` to be optional. Set it to `_all` in order to fetch the first document matching the id across all types.

### Source filtering

By default, the get operation returns the contents of the `_source` field unless you have used the `stored_fields` parameter or if the `_source` field is disabled. You can turn off `_source` retrieval by using the `_source` parameter:
    
    
    GET twitter/tweet/0?_source=false

If you only need one or two fields from the complete `_source`, you can use the `_source_include` & `_source_exclude` parameters to include or filter out that parts you need. This can be especially helpful with large documents where partial retrieval can save on network overhead. Both parameters take a comma separated list of fields or wildcard expressions. Example:
    
    
    GET twitter/tweet/0?_source_include=*.id&_source_exclude=entities

If you only want to specify includes, you can use a shorter notation:
    
    
    GET twitter/tweet/0?_source=*.id,retweeted

### Stored Fields

The get operation allows specifying a set of stored fields that will be returned by passing the `stored_fields` parameter. If the requested fields are not stored, they will be ignored. Consider for instance the following mapping:
    
    
    PUT twitter
    {
       "mappings": {
          "tweet": {
             "properties": {
                "counter": {
                   "type": "integer",
                   "store": false
                },
                "tags": {
                   "type": "keyword",
                   "store": true
                }
             }
          }
       }
    }

Now we can add a document:
    
    
    PUT twitter/tweet/1
    {
        "counter" : 1,
        "tags" : ["red"]
    }

  1. and try to retrieve it: 


    
    
    GET twitter/tweet/1?stored_fields=tags,counter

The result of the above get operation is:
    
    
    {
       "_index": "twitter",
       "_type": "tweet",
       "_id": "1",
       "_version": 1,
       "found": true,
       "fields": {
          "tags": [
             "red"
          ]
       }
    }

Field values fetched from the document it self are always returned as an array. Since the `counter` field is not stored the get request simply ignores it when trying to get the `stored_fields.`

It is also possible to retrieve metadata fields like `_routing` and `_parent` fields:
    
    
    PUT twitter/tweet/2?routing=user1
    {
        "counter" : 1,
        "tags" : ["white"]
    }
    
    
    GET twitter/tweet/2?routing=user1&stored_fields=tags,counter

The result of the above get operation is:
    
    
    {
       "_index": "twitter",
       "_type": "tweet",
       "_id": "2",
       "_version": 1,
       "_routing": "user1",
       "found": true,
       "fields": {
          "tags": [
             "white"
          ]
       }
    }

Also only leaf fields can be returned via the `stored_field` option. So object fields can’t be returned and such requests will fail.

### Generated fields

If no refresh occurred between indexing and refresh, GET will access the transaction log to fetch the document. However, some fields are generated only when indexing. If you try to access a field that is only generated when indexing, you will get an exception (default). You can choose to ignore field that are generated if the transaction log is accessed by setting `ignore_errors_on_generated_fields=true`.

### Getting the _source directly

Use the `/{index}/{type}/{id}/_source` endpoint to get just the `_source` field of the document, without any additional content around it. For example:
    
    
    GET twitter/tweet/1/_source

You can also use the same source filtering parameters to control which parts of the `_source` will be returned:
    
    
    GET twitter/tweet/1/_source?_source_include=*.id&_source_exclude=entities'

Note, there is also a HEAD variant for the _source endpoint to efficiently test for document _source existence. An existing document will not have a _source if it is disabled in the [mapping](mapping-source-field.html "_source field").
    
    
    HEAD twitter/tweet/1/_source

### Routing

When indexing using the ability to control the routing, in order to get a document, the routing value should also be provided. For example:
    
    
    GET twitter/tweet/2?routing=user1

The above will get a tweet with id 2, but will be routed based on the user. Note, issuing a get without the correct routing, will cause the document not to be fetched.

### Preference

Controls a `preference` of which shard replicas to execute the get request on. By default, the operation is randomized between the shard replicas.

The `preference` can be set to:

`_primary`
     The operation will go and be executed only on the primary shards. 
`_local`
     The operation will prefer to be executed on a local allocated shard if possible. 
Custom (string) value 
     A custom value will be used to guarantee that the same shards will be used for the same custom value. This can help with "jumping values" when hitting different shards in different refresh states. A sample value can be something like the web session id, or the user name. 

### Refresh

The `refresh` parameter can be set to `true` in order to refresh the relevant shard before the get operation and make it searchable. Setting it to `true` should be done after careful thought and verification that this does not cause a heavy load on the system (and slows down indexing).

### Distributed

The get operation gets hashed into a specific shard id. It then gets redirected to one of the replicas within that shard id and returns the result. The replicas are the primary shard and its replicas within that shard id group. This means that the more replicas we will have, the better GET scaling we will have.

### Versioning support

You can use the `version` parameter to retrieve the document only if its current version is equal to the specified one. This behavior is the same for all version types with the exception of version type `FORCE` which always retrieves the document. Note that `FORCE` version type is deprecated.

Internally, Elasticsearch has marked the old document as deleted and added an entirely new document. The old version of the document doesn’t disappear immediately, although you won’t be able to access it. Elasticsearch cleans up deleted documents in the background as you continue to index more data.