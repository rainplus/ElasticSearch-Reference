## HTTP

The http module allows to expose **elasticsearch** APIs over HTTP.

The http mechanism is completely asynchronous in nature, meaning that there is no blocking thread waiting for a response. The benefit of using asynchronous communication for HTTP is solving the [C10k problem](http://en.wikipedia.org/wiki/C10k_problem).

When possible, consider using [HTTP keep alive](http://en.wikipedia.org/wiki/Keepalive#HTTP_Keepalive) when connecting for better performance and try to get your favorite client not to do [HTTP chunking](http://en.wikipedia.org/wiki/Chunked_transfer_encoding).

### Settings

The settings in the table below can be configured for HTTP. Note that none of them are dynamically updatable so for them to take effect they should be set in `elasticsearch.yml`.

Setting | Description  
---|---      
`http.port`| A bind port range. Defaults to `9200-9300`.    
`http.publish_port`| The port that HTTP clients should use when communicating with this node. Useful when a cluster node is behind a proxy or firewall and the `http.port` is not directly addressable from the outside. Defaults to the actual port assigned via `http.port`.    
`http.bind_host`| The host address to bind the HTTP service to. Defaults to `http.host` (if set) or `network.bind_host`.    
`http.publish_host`| The host address to publish for HTTP clients to connect to. Defaults to `http.host` (if set) or `network.publish_host`.    
`http.host`| Used to set the `http.bind_host` and the `http.publish_host` Defaults to `http.host` or `network.host`.    
`http.max_content_length`| The max content of an HTTP request. Defaults to `100mb`. If set to greater than `Integer.MAX_VALUE`, it will be reset to 100mb.    
`http.max_initial_line_length`| The max length of an HTTP URL. Defaults to `4kb`    
`http.max_header_size`| The max size of allowed headers. Defaults to `8kB`    
`http.compression`| Support for compression when possible (with Accept-Encoding). Defaults to `true`.    
`http.compression_level`| Defines the compression level to use for HTTP responses. Valid values are in the range of 1 (minimum compression) and 9 (maximum compression). Defaults to `3`.    
`http.cors.enabled`| Enable or disable cross-origin resource sharing, i.e. whether a browser on another origin can execute requests against Elasticsearch. Set to `true` to enable Elasticsearch to process pre-flight [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) requests. Elasticsearch will respond to those requests with the `Access-Control-Allow-Origin` header if the `Origin` sent in the request is permitted by the `http.cors.allow-origin` list. Set to `false` (the default) to make Elasticsearch ignore the `Origin` request header, effectively disabling CORS requests because Elasticsearch will never respond with the `Access-Control-Allow-Origin` response header. Note that if the client does not send a pre-flight request with an `Origin` header or it does not check the response headers from the server to validate the `Access-Control-Allow-Origin` response header, then cross-origin security is compromised. If CORS is not enabled on Elasticsearch, the only way for the client to know is to send a pre-flight request and realize the required response headers are missing.    
`http.cors.allow-origin`| Which origins to allow. Defaults to no origins allowed. If you prepend and append a `/` to the value, this will be treated as a regular expression, allowing you to support HTTP and HTTPs. for example using `/https?:\/\/localhost(:[0-9]+)?/` would return the request header appropriately in both cases. `*` is a valid value but is considered a **security risk** as your elasticsearch instance is open to cross origin requests from **anywhere**.    
`http.cors.max-age`| Browsers send a "preflight" OPTIONS-request to determine CORS settings. `max-age` defines how long the result should be cached for. Defaults to `1728000` (20 days)    
`http.cors.allow-methods`| Which methods to allow. Defaults to `OPTIONS, HEAD, GET, POST, PUT, DELETE`.    
`http.cors.allow-headers`| Which headers to allow. Defaults to `X-Requested-With, Content-Type, Content-Length`.    
`http.cors.allow-credentials`| Whether the `Access-Control-Allow-Credentials` header should be returned. Note: This header is only returned, when the setting is set to `true`. Defaults to `false`    
`http.detailed_errors.enabled`| Enables or disables the output of detailed error messages and stack traces in response output. Note: When set to `false` and the `error_trace` request parameter is specified, an error will be returned; when `error_trace` is not specified, a simple message will be returned. Defaults to `true`    
`http.pipelining`| Enable or disable HTTP pipelining, defaults to `true`.    
`http.pipelining.max_events`| The maximum number of events to be queued up in memory before a HTTP connection is closed, defaults to `10000`.    
`http.content_type.required`| Enables or disables strict checking and usage of the `Content-Type` header for all requests with content, defaults to `false`.  
  
It also uses the common [network settings](modules-network.html).

### Disable HTTP

The http module can be completely disabled and not started by setting `http.enabled` to `false`. Elasticsearch nodes (and Java clients) communicate internally using the [transport interface](modules-transport.html), not HTTP. It might make sense to disable the `http` layer entirely on nodes which are not meant to serve REST requests directly. For instance, you could disable HTTP on [data-only nodes](modules-node.html) if you also have [client nodes](modules-node.html) which are intended to serve all REST requests. Be aware, however, that you will not be able to send any REST requests (eg to retrieve node stats) directly to nodes which have HTTP disabled.
