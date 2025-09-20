# KOF Tracing

KOF integrates Jaeger for distributed tracing via OpenTelemetry.. By default, Jaeger runs with in-memory storage and a maximum of 100,000 traces. When the limit is reached, it evicts the oldest traces (FIFO). Note that in-memory traces are lost on pod restart.

 To solve this problem in production, you canuse a persistent backend (e.g., Cassandra, Elasticsearch, or a compatible VictoriaMetrics-Jaeger deployment) and set retention according to your requirements.

 For example, you can tell Jaeger to use Cassandra by adding the following to the `charts/kof-storage/values.yaml` file:

```yaml
jaeger:
  enabled: true
  spec:
    strategy: production
    storage:
      type: cassandra
      options:
        cassandra:
          servers: cassandra.kof.svc
          keyspace: jaeger_v1_dc1
          replication: "{'class':'NetworkTopologyStrategy','dc1':3}"
```