![Logo](assets/devops-bootcamp-logo.png)

## BONUS - Databases

Database is an organized collection of information stored electronically that allows to be easily stored, searched, updated and deleted.

## Types of databases

1. **Key Value Databases** - unique key, no joins, fast. Best use case: caching or message queue

*Example*: Redis, Memcached, etcd

2. **Wide Column Databases** - without schema, no joins, scales good, Best use case: time-series, unstructured data

*Example*: Cassandra, HBase

3. **Document Databases** - without schema, no joins, faster to read than write. Best use case: mobile apps, game apps etc.

*Example*: MongoDB, DynamoDBm CouchDB

4. **Relational Databases** - structured data, schema is there, ACID - atomic, consistent, isolated, durable, more difficult to scale

*Example*: MySQL, Postresql, CockroachDB

5. **Graph Databases** - used to directly connect entities, easier to query. Best use case: graphs, detecting patterns, recommendations

*Example*: Neo4j, Dgraph

6. **Search Databases** - used to full text search in efficient way, based on indexes.

*Example*: Elasticsearch, Solr
