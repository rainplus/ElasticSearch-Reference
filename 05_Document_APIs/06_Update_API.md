## Update API

The update API allows to update a document based on a script provided. The operation gets the document (collocated with the shard) from the index, runs the script (with optional script language and parameters), and index back the result (also allows to delete, or ignore the operation). It uses versioning to make sure no updates have happened during the "get" and "reindex".

Note, this operation still means full reindex of the document, it just removes some network roundtrips and reduces chances of version conflicts between the get and the index. The `_source` field needs to be enabled for this feature to work.

For example, lets index a simple doc:
    
    
    PUT test/type1/1
    {
        "counter" : 1,
        "tags" : ["red"]
    }

### Scripted updates

Now, we can execute a script that would increment the counter:
    
    
    POST test/type1/1/_update
    {
        "script" : {
            "inline": "ctx._source.counter += params.count",
            "lang": "painless",
            "params" : {
                "count" : 4
            }
        }
    }

We can add a tag to the list of tags (note, if the tag exists, it will still add it, since its a list):
    
    
    POST test/type1/1/_update
    {
        "script" : {
            "inline": "ctx._source.tags.add(params.tag)",
            "lang": "painless",
            "params" : {
                "tag" : "blue"
            }
        }
    }

In addition to `_source`, the following variables are available through the `ctx` map: `_index`, `_type`, `_id`, `_version`, `_routing`, `_parent`, and `_now` (the current timestamp).

We can also add a new field to the document:
    
    
    POST test/type1/1/_update
    {
        "script" : "ctx._source.new_field = \"value_of_new_field\""
    }

Or remove a field from the document:
    
    
    POST test/type1/1/_update
    {
        "script" : "ctx._source.remove(\"new_field\")"
    }

And, we can even change the operation that is executed. This example deletes the doc if the `tags` field contain `green`, otherwise it does nothing (`noop`):
    
    
    POST test/type1/1/_update
    {
        "script" : {
            "inline": "if (ctx._source.tags.contains(params.tag)) { ctx.op = \"delete\" } else { ctx.op = \"none\" }",
            "lang": "painless",
            "params" : {
                "tag" : "green"
            }
        }
    }

### Updates with a partial document

The update API also support passing a partial document, which will be merged into the existing document (simple recursive merge, inner merging of objects, replacing core "keys/values" and arrays). For example:
    
    
    POST test/type1/1/_update
    {
        "doc" : {
            "name" : "new_name"
        }
    }

If both `doc` and `script` are specified, then `doc` is ignored. Best is to put your field pairs of the partial document in the script itself.

### Detecting noop updates

If `doc` is specified its value is merged with the existing `_source`. By default updates that don’t change anything detect that they don’t change anything and return "result": "noop" like this:
    
    
    POST test/type1/1/_update
    {
        "doc" : {
            "name" : "new_name"
        }
    }

If `name` was `new_name` before the request was sent then the entire update request is ignored. The `result` element in the response returns `noop` if the request was ignored.
    
    
    {
       "_shards": {
            "total": 0,
            "successful": 0,
            "failed": 0
       },
       "_index": "test",
       "_type": "type1",
       "_id": "1",
       "_version": 6,
       "result": noop
    }

You can disable this behavior by setting "detect_noop": false like this:
    
    
    POST test/type1/1/_update
    {
        "doc" : {
            "name" : "new_name"
        },
        "detect_noop": false
    }

### Upserts

If the document does not already exist, the contents of the `upsert` element will be inserted as a new document. If the document does exist, then the `script` will be executed instead:
    
    
    POST test/type1/1/_update
    {
        "script" : {
            "inline": "ctx._source.counter += params.count",
            "lang": "painless",
            "params" : {
                "count" : 4
            }
        },
        "upsert" : {
            "counter" : 1
        }
    }

#### `scripted_upsert`

If you would like your script to run regardless of whether the document exists or not — i.e. the script handles initializing the document instead of the `upsert` element — then set `scripted_upsert` to `true`:
    
    
    POST sessions/session/dh3sgudg8gsrgl/_update
    {
        "scripted_upsert":true,
        "script" : {
            "id": "my_web_session_summariser",
            "params" : {
                "pageViewEvent" : {
                    "url":"foo.com/bar",
                    "response":404,
                    "time":"2014-01-01 12:32"
                }
            }
        },
        "upsert" : {}
    }

#### `doc_as_upsert`

Instead of sending a partial `doc` plus an `upsert` doc, setting `doc_as_upsert` to `true` will use the contents of `doc` as the `upsert` value:
    
    
    POST test/type1/1/_update
    {
        "doc" : {
            "name" : "new_name"
        },
        "doc_as_upsert" : true
    }

### Parameters

The update operation supports the following query-string parameters:

`retry_on_conflict`

| 

In between the get and indexing phases of the update, it is possible that another process might have already updated the same document. By default, the update will fail with a version conflict exception. The `retry_on_conflict` parameter controls how many times to retry the update before finally throwing an exception.   
  
---|---  
  
`routing`

| 

Routing is used to route the update request to the right shard and sets the routing for the upsert request if the document being updated doesn’t exist. Can’t be used to update the routing of an existing document.   
  
`parent`

| 

Parent is used to route the update request to the right shard and sets the parent for the upsert request if the document being updated doesn’t exist. Can’t be used to update the `parent` of an existing document. If an alias index routing is specified then it overrides the parent routing and it is used to route the request.   
  
`timeout`

| 

Timeout waiting for a shard to become available.   
  
`wait_for_active_shards`

| 

The number of shard copies required to be active before proceeding with the update operation. See [here](docs-index_.html#index-wait-for-active-shards "Wait For Active Shardsedit") for details.   
  
`refresh`

| 

Control when the changes made by this request are visible to search. See [_`?refresh`_](docs-refresh.html "?refresh").   
  
`_source`

| 

Allows to control if and how the updated source should be returned in the response. By default the updated source is not returned. See [`source filtering`](search-request-source-filtering.html "Source filtering") for details.   
  
`version` & `version_type`

| 

The update API uses the Elasticsearch’s versioning support internally to make sure the document doesn’t change during the update. You can use the `version` parameter to specify that the document should only be updated if its version matches the one specified. By setting version type to `force` you can force the new version of the document after update (use with care! with `force` there is no guarantee the document didn’t change).   
  
![Note](images/icons/note.png)

### The update API does not support external versioning

External versioning (version types `external` & `external_gte`) is not supported by the update API as it would result in Elasticsearch version numbers being out of sync with the external system. Use the [`index` API](docs-index_.html "Index API") instead.