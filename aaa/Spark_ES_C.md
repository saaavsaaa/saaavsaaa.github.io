Spark同步到ElasticSearch(EsSparkSQL)可用的所有配置属性
详见:https://www.elastic.co/guide/en/elasticsearch/hadoop/current/configuration.html

注意：
	不同版本的ES支持不一样
	有些配置之间会互相影响
	个别配置使用时需要注意(副作用很强)

es.resource
es.resource.read
es.resource.write

es.nodes
es.port

es.nodes.path.prefix
es.query

es.input.json
es.write.operation:index ,create,update,upsert

es.output.json

es.ingest.pipeline

es.mapping.id
es.mapping.parent

es.mapping.join
es.mapping.routing

es.mapping.version
es.mapping.version.type

es.mapping.ttl
es.mapping.timestamp

es.mapping.include
es.mapping.exclude

es.mapping.date.rich
es.read.field.include
es.read.field.exclude
es.read.field.as.array.include
es.read.field.as.array.exclude

es.read.metadata
es.read.metadata.field
es.read.metadata.version

es.update.script.inline
es.update.script.file
es.update.script.stored
es.update.script.lang
es.update.script.params
es.update.script.params.json
es.update.retry.on.conflict

es.index.auto.create
es.index.read.missing.as.empty
es.field.read.empty.as.null
es.field.read.validate.presence:warn ,ignore,strict
es.read.source.filter

es.index.read.allow.red.status
es.input.max.docs.per.partition
es.input.use.sliced.partitions

es.nodes.discovery
es.nodes.client.only
es.nodes.data.only
es.nodes.ingest.only
es.nodes.wan.only
es.nodes.resolve.hostname

es.http.timeout
es.http.retries
es.scroll.keepalive
es.scroll.size
es.scroll.limit

es.action.heart.beat.lead

es.net.http.header.[HEADER-NAME]

es.keystore.location

es.net.http.auth.user
es.net.http.auth.pass

es.net.ssl
es.net.ssl.keystore.location
es.net.ssl.keystore.pass
es.net.ssl.keystore.type
es.net.ssl.truststore.location
es.net.ssl.truststore.pass
es.net.ssl.cert.allow.self.signed
es.net.ssl.protocol

es.net.proxy.http.host
es.net.proxy.http.port
es.net.proxy.http.user
es.net.proxy.http.pass
es.net.proxy.http.use.system.props
es.net.proxy.https.host
es.net.proxy.https.port
es.net.proxy.https.user
es.net.proxy.https.pass
es.net.proxy.https.use.system.props
es.net.proxy.socks.host
es.net.proxy.socks.port
es.net.proxy.socks.user
es.net.proxy.socks.pass
es.net.proxy.socks.use.system.props

es.batch.size.bytes
es.batch.size.entries
es.batch.write.refresh
es.batch.write.retry.count
es.batch.write.retry.wait
es.ser.reader.value.class
es.ser.writer.value.class

-----
微信公众号：

![Image](/ppp/0.png)
