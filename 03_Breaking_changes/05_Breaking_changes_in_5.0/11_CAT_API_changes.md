## CAT API变更  CAT API changes

### Use Accept header for specifying response media type

Previous versions of Elasticsearch accepted the Content-type header field for controlling the media type of the response in the cat API. This is in opposition to the HTTP spec which specifies the Accept header field for this purpose. Elasticsearch now uses the Accept header field and support for using the Content-Type header field for this purpose has been removed.

### Host field removed from the cat nodes API

The `host` field has been removed from the cat nodes API as its value is always equal to the `ip` field. The `name` field is available in the cat nodes API and should be used instead of the `host` field.

### Changes to cat recovery API

The fields `bytes_recovered` and `files_recovered` have been added to the cat recovery API. These fields, respectively, indicate the total number of bytes and files that have been recovered.

The fields `total_files` and `total_bytes` have been renamed to `files_total` and `bytes_total`, respectively.

Additionally, the field `translog` has been renamed to `translog_ops_recovered`, the field `translog_total` to `translog_ops` and the field `translog_percent` to `translog_ops_percent`. The short aliases for these fields are `tor`, `to`, and `top`, respectively.

### Changes to cat nodes API

The cat nodes endpoint returns `m` for master eligible, `d` for data, and `i` for ingest. A node with no explicit roles will be a coordinating only node and marked with `-`. A node can have multiple roles. The master column has been adapted to return only whether a node is the current master (`*`) or not (`-`).

### Changes to cat field data API

The cat field data endpoint adds a row per field instead of a column per field.

The `total` field has been removed from the field data API. Total field data usage per node can be got by cat nodes API.
