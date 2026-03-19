# Study Elasticsearch ⚡
This repository contains a practical guide to learn Elasticsearch from installation to advanced querying, with hands-on scenarios to practice and strengthen your Elasticsearch skills.

## Installation 🔧
1. **Download Elasticsearch**:
   - Visit `https://www.elastic.co/downloads/elasticsearch`
   - Download the **Archive File** for your operating system
   - Extract the downloaded file

2. **Configure Elasticsearch**:
   ```yaml
   # config/elasticsearch.yml
   cluster.name: dzarurizky
   node.name: dzaru-1
   xpack.security.enabled: false
   http.port: 9200
   path.data: data
   path.logs: logs
   ```

3. **Setup & Run Elasticsearch**:
   ```bash
   # Start Elasticsearch
   ./bin/elasticsearch

   # Verify the server is running
   curl http://localhost:9200
   ```

   > **Windows**: Use `bin\elasticsearch.bat` instead of `bin/elasticsearch`
   >
   > **Stop**: Press `Ctrl + C`

## List of Material 📚

* 📘 **Index & Mapping**

  Contains core concepts including Index creation, Mapping definition, Data Types, Dynamic Field Mapping, Object Fields, Array Fields, Multi Fields, Nested Fields, and Flattened Fields.

  ```http
  # Create an index
  PUT http://localhost:9200/customers

  # List all indexes
  GET http://localhost:9200/_cat/indices?v

  # Delete an index
  DELETE http://localhost:9200/customers

  # Define mapping
  PUT http://localhost:9200/customers_v2/_mapping
  ```

  ```json
  {
    "numeric_detection": true,
    "date_detection": true,
    "properties": {
      "username":   { "type": "keyword" },
      "first_name": { "type": "text" },
      "last_name":  { "type": "text" },
      "email":      { "type": "keyword" },
      "gender":     { "type": "keyword" },
      "birth_date": { "type": "date", "format": "yyyy-MM-dd" }
    }
  }
  ```

* 📗 **CRUD Operations**

  Contains Create API, Index API, Get API, Update API, Delete API, Bulk API, Multi Get API, Delete by Query, and field selection with `_source_includes` / `_source_excludes`.

  ```http
  # Create (safe — 409 if exists)
  POST http://localhost:9200/customers/_create/dzaru

  # Index (create or replace)
  POST http://localhost:9200/products/_doc/3

  # Get a document
  GET http://localhost:9200/customers/_doc/dzaru

  # Update specific fields
  POST http://localhost:9200/products/_update/3

  # Delete a document
  DELETE http://localhost:9200/customers/_doc/spammer

  # Bulk operations
  POST http://localhost:9200/_bulk
  ```

  ```
  { "create": { "_index": "customers", "_id": "joko" } }
  { "name": "Joko Morro", "register_at": "2023-10-10 00:00:00" }
  { "index": { "_index": "customers", "_id": "budi" } }
  { "name": "Budi Nugraha", "register_at": "2023-10-10 00:00:00" }
  { "update": { "_index": "products", "_id": "1" } }
  { "doc": { "price": 2500 } }
  { "delete": { "_index": "customers", "_id": "spammer" } }
  ```

* 📙 **Query DSL**

  Contains Match All, Term Query, Terms Query, Match Query, Boolean Query (must / filter / must_not / should), Nested Query, Boost Score, Search After pagination, and the Explain API.

  ```http
  # Search
  POST http://localhost:9200/products/_search
  ```

  ```json
  {
    "query": {
      "bool": {
        "must": [
          { "term": { "hobbies": "gaming" } }
        ],
        "should": [
          { "term": { "banks.name": "bca" } },
          { "term": { "banks.name": "bni" } }
        ],
        "minimum_should_match": 1
      }
    }
  }
  ```

  `Search After (pagination beyond 10K limit):`

  ```json
  {
    "size": 100,
    "from": 0,
    "query": { "match_all": {} },
    "sort": [{ "id": { "order": "asc" } }],
    "search_after": ["10087"]
  }
  ```

* 📕 **Alias & Reindex**

  Covers creating and removing aliases, listing aliases, and the Reindex API for migrating data to a new index when schema changes are needed.

  ```http
  # Create / remove alias
  POST http://localhost:9200/_aliases

  # List all aliases
  GET http://localhost:9200/_aliases

  # Reindex data to a new index
  POST http://localhost:9200/_reindex
  ```

  ```json
  {
    "actions": [
      { "add":    { "alias": "customer", "index": "customers_v2" } },
      { "remove": { "alias": "customer", "index": "customers" } }
    ]
  }
  ```

* 📓 **Snapshot & Restore**

  Covers configuring a snapshot repository, creating and restoring snapshots, and deleting snapshots or repositories.

  ```yaml
  # config/elasticsearch.yml
  path.repo: ["snapshots"]
  ```

  ```http
  # Create repository
  PUT http://localhost:9200/_snapshot/first_backup

  # Create snapshot
  PUT http://localhost:9200/_snapshot/first_backup/snapshot1

  # Close index before restore
  POST http://localhost:9200/categories/_close

  # Restore snapshot
  POST http://localhost:9200/_snapshot/first_backup/snapshot1/_restore

  # Reopen index after restore
  POST http://localhost:9200/categories/_open
  ```

* 📒 **Cat API**

  Administrative APIs for monitoring and maintaining Elasticsearch clusters.

  ```http
  GET http://localhost:9200/_cat/indices?v
  GET http://localhost:9200/_cat/aliases?v
  GET http://localhost:9200/_cat/nodes?v
  GET http://localhost:9200/_cat/health?v
  GET http://localhost:9200/_cat/shards?v
  GET http://localhost:9200/_cat/snapshots?v
  ```

  `Example output for /_cat/indices?v:`

  ```
  health status index        pri rep docs.count store.size
  yellow open   customers_v2   1   1       1000    331.1kb
  yellow open   categories     1   1      20000    864.2kb
  yellow open   products       1   1          3      6.1kb
  ```

## 📍 References
* [Udemy](https://www.udemy.com/user/eko-kurniawan)

## 👨‍💻 Contributors
* [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)
