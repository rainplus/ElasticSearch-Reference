## 5.1的版本变更

### Indices API changes

#### Alias names are validated against (most of) the rules for index names

Alias names are now validated against almost the same set of rules that validate index names. The only difference is that aliases are allowed to have uppercase characters. That means that aliases may not:

  * Start with `_`, `-`, or `+`
  * Contain `#`, `\`, `/`, `*`, `?`, `"`, `<`, `>`, `|`, ` `, `,`
  * Be longer than 100 UTF-8 encoded bytes 
  * Be exactly `.` or `..`



Aliases created in versions before 5.1.0 are still supported but no new aliases can be added that violate those rules. Since modifying an alias in elasticsearch is removing it and recreating it atomically using the `_aliases` API, modifying aliases with invalid names is also no longer supported.

### Java API changes

#### Log4j dependency has been upgraded

The Log4j dependency has been upgraded from version 2.6.2 to version 2.7. If you’re using the transport client in your application, you should update your Log4j dependencies accordingly.

#### Local discovery has been removed

Local discovery has been removed; this discovery implementation was used internally in the tribe service and for tests that ran multiple nodes inside the same JVM. This means that setting `discovery.type` to `local` will fail on startup.

### Plugin API changes

#### UnicastHostsProvider now pull based

Plugging in a `UnicastHostsProvider` for zen discovery is now pull based. Implement a `DiscoveryPlugin` and override the `getZenHostsProviders` method. Selecting a hosts provider is also now done with a separate setting, `discovery.zen.hosts_provider`.

#### ZenPing and MasterElectService pluggability removed

These classes are no longer pluggable. Either implement your own discovery, or extend from ZenDiscovery and customize as necessary.

#### onModule support removed

Plugins could formerly implement methods of the name `onModule` which took a single Guice module. All the uses of onModule for plugging in custom behavior have now been converted to pull based plugins, and hooks for onModule have been removed.

### Other API changes

#### Indices stats and node stats API unrecognized metrics

The indices stats and node stats APIs allow querying Elasticsearch for a variety of metrics. Previous versions of Elasticsearch would silently accept unrecognized metrics (e.g., typos like). In 5.1.0, this is no longer the case; unrecognized metrics will cause the request to fail. There is one exception to this which is the percolate metric which was removed in 5.0.0 but requests for these will only produce a warning in the 5.x series starting with 5.1.0 and will fail like any other unrecognized metric in 6.0.0.
