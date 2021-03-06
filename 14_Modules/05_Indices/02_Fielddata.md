## 字段数据 Fielddata

字段数据缓存主要用于在字段上进行排序或计算聚合。 它将所有的字段值加载到内存中，以便为这些值提供快速的基于文档的访问。 字段数据缓存对于字段的构建可能很花费，所以建议有足够的内存来分配它，并保持它的加载。


可以使用`indices.fielddata.cache.size`来控制用于字段数据缓存的内存量。 注意：重新加载不适合你的缓存的字段数据将是代替很大的，性能会差。

`indices.fielddata.cache.size`
    
    字段数据高速缓存的最大大小，例如节点堆空间的“30％”或绝对值，例如“12GB”。 默认为不限制大小。 另请参阅[字段数据电路断路器]。

![Note](/images/icons/note.png)
这些设置是静态设置一定要配置在集群中的每个节点上
#### Monitoring field data

您可以使用[节点统计API](cluster-nodes-stats.html)监视现场数据以及现场数据断路器的内存使用情况，
