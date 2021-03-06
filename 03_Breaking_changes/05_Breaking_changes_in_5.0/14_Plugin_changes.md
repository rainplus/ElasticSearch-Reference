## 插件变更 Plugin changes

The command `bin/plugin` has been renamed to `bin/elasticsearch-plugin`. The structure of the plugin ZIP archive has changed. All the plugin files must be contained in a top-level directory called `elasticsearch`. If you use the gradle build, this structure is automatically generated.

### Plugins isolation

`isolated` option has been removed. Each plugin will have its own classloader.

### Site plugins removed

Site plugins have been removed. Site plugins should be reimplemented as Kibana plugins.

### Multicast plugin removed

Multicast has been removed. Use unicast discovery, or one of the cloud discovery plugins.

### Plugins with custom query implementations

Plugins implementing custom queries need to implement the `fromXContent(QueryParseContext)` method in their `QueryParser` subclass rather than `parse`. This method will take care of parsing the query from `XContent` format into an intermediate query representation that can be streamed between the nodes in binary format, effectively the query object used in the java api. Also, the query builder needs to be registered as a `NamedWriteable`. This is all done by implementing the `SearchPlugin` interface and overriding the `getQueries` method. The query object can then transform itself into a lucene query through the new `toQuery(QueryShardContext)` method, which returns a lucene query to be executed on the data node.

Similarly, plugins implementing custom score functions need to implement the `fromXContent(QueryParseContext)` method in their `ScoreFunctionParser` subclass rather than `parse`. This method will take care of parsing the function from `XContent` format into an intermediate function representation that can be streamed between the nodes in binary format, effectively the function object used in the java api. The function object can then transform itself into a lucene function through the new `toFunction(QueryShardContext)` method, which returns a lucene function to be executed on the data node.

### Cloud AWS plugin changes

Cloud AWS plugin has been split in two plugins:

  * [Discovery EC2 plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/discovery-ec2.html)
  * [Repository S3 plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/repository-s3.html)



Proxy settings for both plugins have been renamed:

  * from `cloud.aws.proxy_host` to `cloud.aws.proxy.host`
  * from `cloud.aws.ec2.proxy_host` to `cloud.aws.ec2.proxy.host`
  * from `cloud.aws.s3.proxy_host` to `cloud.aws.s3.proxy.host`
  * from `cloud.aws.proxy_port` to `cloud.aws.proxy.port`
  * from `cloud.aws.ec2.proxy_port` to `cloud.aws.ec2.proxy.port`
  * from `cloud.aws.s3.proxy_port` to `cloud.aws.s3.proxy.port`



### Cloud Azure plugin changes

Cloud Azure plugin has been split in three plugins:

  * [Discovery Azure plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/discovery-azure-classic.html)
  * [Repository Azure plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/repository-azure.html)
  * [Store SMB plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/store-smb.html)



If you were using the `cloud-azure` plugin for snapshot and restore, you had in `elasticsearch.yml`:
    
    
    cloud:
        azure:
            storage:
                account: your_azure_storage_account
                key: your_azure_storage_key

You need to give a unique id to the storage details now as you can define multiple storage accounts:
    
    
    cloud:
        azure:
            storage:
                my_account:
                    account: your_azure_storage_account
                    key: your_azure_storage_key

### Cloud GCE plugin changes

Cloud GCE plugin has been renamed to [Discovery GCE plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/discovery-gce.html).

### Delete-By-Query plugin removed

The Delete-By-Query plugin has been removed in favor of a new [Delete By Query API](docs-delete-by-query.html) implementation in core. It now supports throttling, retries and cancellation but no longer supports timeouts. Instead use the [cancel API](docs-delete-by-query.html#docs-delete-by-query-cancel-task-api) to cancel deletes that run too long.

### Mapper Attachments plugin deprecated

Mapper attachments has been deprecated. Users should use now the [`ingest-attachment`](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/ingest-attachment.html) plugin.

### Passing of Java System Properties

Previously, Java system properties could be passed to the plugin command by passing `-D` style arguments directly to the plugin script. This is no longer permitted and such system properties must be passed via ES_JAVA_OPTS.

### Custom plugins path

The ability to specify a custom plugins path via `path.plugins` has been removed.

### ScriptPlugin

Plugins that register custom scripts should implement `ScriptPlugin` and remove their `onModule(ScriptModule)` implementation.

### AnalysisPlugin

Plugins that register custom analysis components should implement `AnalysisPlugin` and remove their `onModule(AnalysisModule)` implementation.

### MapperPlugin

Plugins that register custom mappers should implement `MapperPlugin` and remove their `onModule(IndicesModule)` implementation.

### ActionPlugin

Plugins that register custom actions should implement `ActionPlugin` and remove their `onModule(ActionModule)` implementation.

Plugins that register custom `RestHandler`s should implement `ActionPlugin` and remove their `onModule(NetworkModule)` implemnetation.

### SearchPlugin

Plugins that register custom search time behavior (`Query`, `Suggester`, `ScoreFunction`, `FetchSubPhase`, `Highlighter`, etc) should implement `SearchPlugin` and remove their `onModule(SearchModule)` implementation.

### SearchParseElement

The `SearchParseElement` interface has been removed. Custom search request divs can only be provided under the `ext` element. Plugins can plug in custom parsers for those additional divs by providing a `SearchPlugin.SearchExtSpec`, which consists of a `SearchExtParser` implementation that can parse`XContent` into a `SearchExtBuilder` implementation. The parsing happens now in the coordinating node. The result of parsing is serialized to the data nodes through transport layer together with the rest of the search request and stored in the search context for later retrieval.

### Testing Custom Plugins

`ESIntegTestCase#pluginList` has been removed. Use `Arrays.asList` instead. It isn’t needed now that all plugins require Java 1.8.

### Mapper-Size plugin

The metadata field `_size` is not accessible in aggregations, scripts and when sorting for indices created in 2.x. If these features are needed in your application it is required to reindex the data with Elasticsearch 5.x.
