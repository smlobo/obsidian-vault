|Database / data-system type|Open-source implementations|Commercial / vendor-led|Hyperscaler services|
|---|---|---|---|
|**Relational SQL / RDBMS**|PostgreSQL, MySQL, MariaDB|Oracle Database, Microsoft SQL Server, IBM Db2|Amazon RDS, Google Cloud SQL, Azure SQL|
|**Distributed SQL / NewSQL**|YugabyteDB, TiDB, rqlite, dqlite|CockroachDB|Google Spanner, Amazon Aurora, Azure Cosmos DB|
|**Embedded relational / analytical**|SQLite, DuckDB, dqlite, rqlite|—|—|
|**Key–value stores**|Valkey, Redis, etcd, Memcached|Aerospike, Consul KV|Amazon DynamoDB, Azure Cosmos DB|
|**Embedded storage engines**|RocksDB, LevelDB, LMDB, FoundationDB|Berkeley DB|—|
|**Document databases**|Apache CouchDB|MongoDB, Couchbase, ArangoDB|Google Firestore, Amazon DocumentDB, Azure Cosmos DB|
|**Wide-column stores**|Apache Cassandra, Apache HBase|ScyllaDB|Google Bigtable, Amazon Keyspaces|
|**Graph databases**|JanusGraph, NebulaGraph|Neo4j, TigerGraph, ArangoDB|Amazon Neptune, Azure Cosmos DB Gremlin API|
|**Time-series databases**|Prometheus, VictoriaMetrics|TimescaleDB, InfluxDB|Amazon Timestream, Azure Data Explorer, Google Bigtable|
|**Search / indexing systems**|OpenSearch, Apache Solr|Elasticsearch, Algolia|Azure AI Search, Amazon OpenSearch Service|
|**Vector databases and indexes**|Milvus, Qdrant, pgvector|Pinecone, Weaviate|Vertex AI Vector Search, Azure AI Search, Amazon OpenSearch Service|
|**Analytical databases / OLAP**|ClickHouse, Apache Druid, DuckDB|Snowflake|Google BigQuery, Amazon Redshift, Azure Synapse Analytics|
|**Data warehouses**|ClickHouse and Trino-based stacks|Snowflake, Teradata|BigQuery, Redshift, Synapse|
|**Data lakes**|Hadoop/HDFS; object storage combined with open formats|Vendor-managed data platforms|Amazon S3, Google Cloud Storage, Azure Data Lake Storage|
|**Lakehouse table formats**|Apache Iceberg, Delta Lake, Apache Hudi|Databricks|Google BigLake, Microsoft Fabric, AWS lakehouse services|
|**Stream / event-log systems**|Apache Kafka, Apache Pulsar|Redpanda, Confluent Platform|Amazon Kinesis, Google Pub/Sub, Azure Event Hubs|
|**Caches / in-memory databases**|Valkey, Redis, Memcached|Hazelcast Enterprise, Redis Enterprise|Amazon ElastiCache, Google Memorystore, Azure Managed Redis|
|**Multi-model databases**|PostgreSQL with extensions|Oracle Database, MongoDB, ArangoDB, Couchbase|Azure Cosmos DB, Amazon Neptune, Google Spanner|
|**Object databases**|ObjectDB alternatives and language-specific stores|ObjectDB, InterSystems IRIS|Usually delivered as part of broader managed platforms|
|**Ledger / immutable databases**|immudb, Amazon QLDB-compatible patterns|Enterprise blockchain/ledger products|Cloud ledger or blockchain services|
|**Geospatial databases**|PostGIS, SpatiaLite|Oracle Spatial, Esri-backed systems|BigQuery GIS, Azure Cosmos DB geospatial features|
|**Distributed filesystem / object storage**|Ceph, MinIO, HDFS|WEKA, Pure Storage and similar platforms|Amazon S3, Google Cloud Storage, Azure Blob Storage|

A useful mental model is:

| Workload question                                                   | Typical family     |
| ------------------------------------------------------------------- | ------------------ |
| “I need transactions, joins and constraints.”                       | Relational SQL     |
| “I need SQL across regions or a fault-tolerant cluster.”            | Distributed SQL    |
| “I know the key and need the value extremely quickly.”              | Key–value          |
| “My records are naturally JSON and vary in shape.”                  | Document           |
| “I need enormous partitioned tables with predictable access paths.” | Wide-column        |
| “Relationships and traversals are the main query.”                  | Graph              |
| “Most records are measurements indexed by time.”                    | Time-series        |
| “I need keyword or relevance search.”                               | Search index       |
| “I need semantic similarity over embeddings.”                       | Vector             |
| “I need large-scale aggregations and BI.”                           | Warehouse / OLAP   |
| “I want inexpensive storage for raw data.”                          | Data lake          |
| “I want lake storage with database-like tables and transactions.”   | Lakehouse          |
| “I need an ordered stream of events.”                               | Stream / event log |
| “The database must run inside the application process.”             | Embedded database  |