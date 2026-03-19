# ⚡ Elasticsearch – Complete Guide
A comprehensive guide for learning Elasticsearch, a RESTful API-based database management system for search engine needs.

---

## 📋 Table of Contents

- [What is Elasticsearch](#-what-is-elasticsearch)
- [Installation](#-installation)
- [Running Elasticsearch](#-running-elasticsearch)
- [Data Modeling](#-data-modeling)
- [Data Types](#-data-types)
- [Index](#-index)
- [Mapping](#-mapping)
- [CRUD Operations](#-crud-operations)
- [Search API](#-search-api)
- [Query DSL](#-query-dsl)
- [Advanced Fields](#-advanced-fields)
- [Alias & Reindex](#-alias--reindex)
- [Snapshot & Restore](#-snapshot--restore)
- [Cat API](#-cat-api)

---

## 🔥 What is Elasticsearch

Elasticsearch is a **RESTful API-based database management system** designed for search engine needs. It is one of the most popular search databases used by large companies worldwide, and is free and open source — available at [https://github.com/elastic/elasticsearch](https://github.com/elastic/elasticsearch) and downloadable at [https://www.elastic.co/elasticsearch/](https://www.elastic.co/elasticsearch/).

- #### Apache Lucene

  At the core of Elasticsearch is **Apache Lucene**, a highly popular Information Retrieval library written in Java. All features in Elasticsearch use Apache Lucene under the hood.
  
  > More info: [https://lucene.apache.org/](https://lucene.apache.org/)

- #### Elasticsearch Clients

  Since Elasticsearch communicates via RESTful API (HTTP), any HTTP client can be used:
  
  | Client | URL |
  |--------|-----|
  | Postman | https://www.postman.com/ |
  | Insomnia | https://insomnia.rest/ |
  | JetBrains HTTP Client | https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html |
  | VS Code REST Client | https://marketplace.visualstudio.com/items?itemName=humao.rest-client |
  | curl | https://curl.se/ |

---

## 📦 Installation

- #### Download

  Download Elasticsearch in **Archive File** format from the official page:
  
  > https://www.elastic.co/downloads/elasticsearch

- #### Configuration

  All configurations are stored in `config/elasticsearch.yml` (YAML format). Elasticsearch has a built-in feature called **X-Pack** (closed source, made by Elastic). For learning purposes, it's recommended to disable it:
  
  ```yaml
  # config/elasticsearch.yml
  cluster.name: dzarurizky
  node.name: dzaru-1
  xpack.security.enabled: false
  http.port: 9200
  path.data: data
  path.logs: logs
  ```

---

## ▶️ Running Elasticsearch

- #### Start the Server

  ```bash
  ./bin/elasticsearch
  ```

- #### Verify

  ```bash
  curl http://localhost:9200
  ```
  
  `Expected output:`
  
  ```json
  {
    "name": "dzaru-1",
    "cluster_name": "dzarurizky",
    "version": {
      "number": "9.3.1",
      "lucene_version": "10.3.2"
    },
    "tagline": "You Know, for Search"
  }
  ```
  
  > To stop Elasticsearch, press `Ctrl + C`

---

## 🗂️ Data Modeling

When modeling data with Elasticsearch, unlike relational databases which follow **database normalization**, you should model based on **how the application performs searches**.

- #### Schema Flexibility

  - Elasticsearch supports dynamic schema — you can insert data without defining a schema first
  - Once a field type is set, it **cannot be changed**, only new fields can be added
  - Primary key field is always `_id` (type: string), and only **one** primary key per document

- #### Embedded vs Reference

  | Strategy | Use When |
  |----------|----------|
  | **Embedded** | Documents depend on each other; embedded data is always needed when fetching the main document |
  | **Reference** | Documents can stand alone; you need to manipulate reference data directly; reference data is not always needed |

---

## 📊 Data Types

- #### Basic Types

  | Type | Description |
  |------|-------------|
  | `binary` | Binary data in Base64 encoded string |
  | `boolean` | `true` or `false` |
  | `date` | Date and time up to milliseconds |
  | `date_nanos` | Date and time up to nanoseconds |
  | `ip` | IPv4 or IPv6 |
  | `keyword` | Structured text (id, email, hostname, zipcode, etc.) |
  | `text` | Free text |
  | `version` | Semantic version data (https://semver.org/) |

- #### Numeric Types

  | Type | Description |
  |------|-------------|
  | `long` | 64-bit integer (-2^63 to 2^62-1) |
  | `integer` | 32-bit integer (-2^31 to 2^31-1) |
  | `short` | 16-bit integer (-32768 to 32764) |
  | `byte` | 8-bit integer (-128 to 127) |
  | `double` | 64-bit IEEE 754 floating point |
  | `float` | 32-bit IEEE 754 floating point |
  | `half_float` | Half 16-bit IEEE 754 floating point |
  | `scaled_float` | Floating point stored as long |
  | `unsigned_long` | 64-bit integer (0 to 2^64-1) |

- #### Range Types

  | Type | Description |
  |------|-------------|
  | `integer_range` | Min/max range for integer |
  | `float_range` | Min/max range for float |
  | `long_range` | Min/max range for long |
  | `double_range` | Min/max range for double |
  | `date_range` | Min/max range for date |
  | `ip_range` | Min/max range for ip |

- #### Dynamic Field Mapping

  | JSON Type | Elasticsearch Type |
  |-----------|--------------------|
  | `null` | No field added |
  | `true` / `false` | `boolean` |
  | `double` | `float` |
  | `long` | `long` |
  | `array` | Depends on first element's type |
  | `string` | `date`, `float`, `long`, or `text` — auto-detected |

---

## 🗃️ Index

An **Index** is equivalent to a table in a relational database.

- #### Naming Rules

  - Must be **lowercase**
  - No special characters except `-`, `+`, and `_` (not at the start)
  - Max **255 bytes**
  - When using multiple apps, prefix the index name: `appname_indexname`

- #### Create Index

  ```http
  PUT http://localhost:9200/customers
  ```

- #### List All Indexes

  ```http
  GET http://localhost:9200/_cat/indices?v
  ```

- #### Delete Index

  ```http
  DELETE http://localhost:9200/customers
  ```
  
  > ⚠️ Deleting an index will remove **all documents** inside it.

---

## 🗺️ Mapping

**Mapping** is the schema definition for an index. While Elasticsearch has Dynamic Mapping (auto-detect), it is **recommended to define mappings manually**.

- #### Define Mapping

  ```http
  PUT http://localhost:9200/customers_v2/_mapping
  ```
  
  ```json
  {
    "numeric_detection": true,
    "date_detection": true,
    "dynamic_date_formats": [
      "yyyy-MM-dd HH:mm:ss",
      "yyyy-MM-dd",
      "yyyy/MM/dd HH:mm:ss",
      "yyyy/MM/dd"
    ],
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

- #### Object Field

  For nested objects, define properties within properties:
  
  ```json
  {
    "properties": {
      "address": {
        "properties": {
          "street":   { "type": "text" },
          "city":     { "type": "text" },
          "province": { "type": "text" },
          "country":  { "type": "text" },
          "zip_code": { "type": "keyword" }
        }
      }
    }
  }
  ```

- #### Array Field

  Arrays don't need a special type — just define the type of the array's content:
  
  ```json
  {
    "properties": {
      "hobbies": { "type": "text" }
    }
  }
  ```

- #### Multi Fields

  Define multiple types for a single field (useful for both full-text search and sorting/filtering):
  
  ```json
  {
    "properties": {
      "name": {
        "type": "text",
        "fields": {
          "raw": { "type": "keyword" }
        }
      }
    }
  }
  ```
  
  > Access sub-field using dot notation: `name.raw`
  
  After adding Multi Fields to existing documents, run `_update_by_query` to reindex:
  
  ```http
  POST http://localhost:9200/products/_update_by_query
  ```
  ```json
  { "query": { "match_all": {} } }
  ```

- #### Nested Field

  Use `nested` type when you need accurate querying on arrays of objects (prevents false matches due to Lucene flattening):
  
  ```json
  {
    "properties": {
      "children": {
        "type": "nested",
        "properties": {
          "first_name": { "type": "text" },
          "last_name":  { "type": "text" }
        }
      }
    }
  }
  ```
  
  > ⚠️ Nested queries are expensive — use them wisely.

- #### Flattened Field

  For dynamically-structured objects where you don't know all the fields in advance, use `flattened` type. All values are stored as keywords:
  
  ```json
  {
    "properties": {
      "labels": { "type": "flattened" }
    }
  }
  ```

---

## ✏️ CRUD Operations

- #### Create API

  Safe create — returns error (409 Conflict) if document already exists:
  
  ```http
  POST http://localhost:9200/customers/_create/dzaru
  ```
  ```json
  {
    "name": "Dzaru Rizky Fathan Fortuna",
    "register_at": "2025-03-03 12:00:00"
  }
  ```

- #### Index API

  Create or replace — replaces existing document without error:
  
  ```http
  POST http://localhost:9200/products/_doc/3
  ```
  ```json
  { "name": "Pop Mie Rasa Bakso", "price": 2500 }
  ```

- #### Get API

  Retrieve a document by ID (returns 404 if not found):
  
  ```http
  GET http://localhost:9200/customers/_doc/dzaru
  ```

- #### Get Source API

  Retrieve only the document data, without metadata:
  
  ```http
  GET http://localhost:9200/customers/_source/dzaru
  ```

- #### Check Exists

  Returns `200` if exists, `404` if not:
  
  ```http
  HEAD http://localhost:9200/customers/_source/dzaru
  ```

- #### Multi Get API

  Retrieve multiple documents in a single request:
  
  ```http
  POST http://localhost:9200/_mget
  ```
  ```json
  {
    "docs": [
      { "_id": "1",     "_index": "orders" },
      { "_id": "dzaru", "_index": "customers" },
      { "_id": "3",     "_index": "products" }
    ]
  }
  ```

- #### Update API

  Update only specific fields without replacing the entire document:
  
  ```http
  POST http://localhost:9200/products/_update/3
  ```
  ```json
  { "doc": { "price": 5000 } }
  ```

- #### Delete API

  Delete a document by ID (returns 404 if not found):
  
  ```http
  DELETE http://localhost:9200/customers/_doc/spammer
  ```

- #### Bulk API

  Perform multiple operations (create, index, update, delete) in a single request:
  
  ```http
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

- #### Delete by Query API

  Delete multiple documents matching a query:
  
  ```http
  POST http://localhost:9200/categories/_delete_by_query
  ```
  ```json
  { "query": { "match": { "name": "wrong" } } }
  ```

- #### Select Fields

  Use `_source_includes` and `_source_excludes` to control returned fields:
  
  ```http
  GET http://localhost:9200/order/_search?_source_includes=total,customer_id
  GET http://localhost:9200/products/_search?_source_excludes=price
  ```

---

## 🔍 Search API

```http
POST http://localhost:9200/<index_name>/_search
```

- #### Pagination

  ```http
  GET http://localhost:9200/products/_search?size=10&from=0
  ```
  
  | Parameter | Description | Default |
  |-----------|-------------|---------|
  | `size` | Number of documents per page | 10 |
  | `from` | Starting document offset | 0 |

- #### Sorting

  ```http
  POST http://localhost:9200/products/_search?sort=price:asc
  ```
  
  Multiple sort fields: `field1:asc,field2:desc`

---

## 🔎 Query DSL

- #### Match All

  Returns all documents (default when no query is specified):
  
  ```json
  {
    "query": { "match_all": {} },
    "size": 5,
    "from": 0,
    "sort": [{ "username": { "order": "desc" } }]
  }
  ```

- #### Term Query

  Exact value match. Best for `keyword`, `boolean`, `number`, `date` — **not suitable for `text` fields**:
  
  ```json
  {
    "query": {
      "term": { "gender": "Female" }
    }
  }
  ```

- #### Terms Query

  Equivalent to SQL `IN` operator (also not suitable for `text` fields):
  
  ```json
  {
    "query": {
      "terms": {
        "username": ["username1", "username2", "username3"]
      }
    }
  }
  ```

- #### Match Query

  Full-text search using the same Text Analysis as the field. Best for `text` fields:
  
  ```json
  {
    "query": {
      "match": { "banks.name": "BCA" }
    }
  }
  ```
  
  With operator:
  
  ```json
  {
    "query": {
      "match": {
        "banks.name": {
          "query": "BCA DIGITAL",
          "operator": "AND"
        }
      }
    }
  }
  ```
  
  > Default operator is `OR`. StandardAnalyzer tokenizes text (e.g., `eko@example.com` → `eko`, `example`, `com`).

- #### Nested Query

  Required for searching within `nested` type fields:
  
  ```json
  {
    "query": {
      "nested": {
        "path": "children",
        "query": {
          "bool": {
            "must": [
              { "match": { "children.first_name": "Dzaru" } },
              { "match": { "children.last_name": "Fathan" } }
            ]
          }
        }
      }
    }
  }
  ```

- #### Boolean Query

  Combine multiple queries across multiple fields:
  
  | Clause | Description |
  |--------|-------------|
  | `must` | All conditions must match (affects score) |
  | `filter` | All conditions must match (does NOT affect score, faster) |
  | `must_not` | All conditions must NOT match |
  | `should` | At least one condition should match (affects score) |
  
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
  
  > When `must` or `filter` is present, `should` default minimum is `0`. Set `minimum_should_match` to require at least N `should` conditions to match.

- #### Boost Score

  Adjust relevance weight per query clause:
  
  ```json
  {
    "query": {
      "bool": {
        "should": [
          { "term": { "banks.name": { "value": "bca", "boost": 0 } } },
          { "term": { "banks.name": { "value": "bni", "boost": 2 } } }
        ]
      }
    }
  }
  ```

- #### Explain API

  Understand how a document's score was calculated:
  
  ```http
  POST http://localhost:9200/customer/_explain/username126
  ```
  ```json
  {
    "query": {
      "bool": {
        "must": [{ "term": { "hobbies": "gaming" } }]
      }
    }
  }
  ```

- #### Scoring Algorithm

  Elasticsearch uses **BM25** (Okapi BM25) to calculate relevance scores — an improvement over the older TF-IDF algorithm that prevents keyword stuffing.
  
  > Reference: https://www.elastic.co/blog/practical-bm25-part-2-the-bm25-algorithm-and-its-variables

---

## 🔄 Search After

Elasticsearch limits search results to **10,000 documents** by default due to the **Deep Paging Problem** in distributed systems.

- #### Search After (Recommended)

  Use `search_after` with a sort field to paginate beyond the 10K limit:
  
  ```json
  {
    "size": 100,
    "from": 0,
    "query": { "match_all": {} },
    "sort": [{ "id": { "order": "asc" } }],
    "search_after": ["10087"]
  }
  ```
  
  > Use the last document's sort value as the `search_after` value for the next page.

- #### Scroll API

  An older approach for retrieving all documents — **no longer recommended**. Use Search After instead.
  
  > Reference: https://www.elastic.co/guide/en/elasticsearch/reference/current/scroll-api.html

---

## 🔀 Alias & Reindex

- #### Why Use Alias?

  In Elasticsearch, once a field type is set it cannot be changed — you must **create a new index** for schema changes. Alias lets clients always point to the correct index without changing their code.

- #### Create / Remove Alias

  ```http
  POST http://localhost:9200/_aliases
  ```
  ```json
  {
    "actions": [
      { "add":    { "alias": "customer", "index": "customers_v2" } },
      { "remove": { "alias": "customer", "index": "customers" } }
    ]
  }
  ```

- #### List All Aliases

  ```http
  GET http://localhost:9200/_aliases
  ```

- #### Reindex API

  Move data from one index to another:
  
  ```http
  POST http://localhost:9200/_reindex
  ```
  ```json
  {
    "source": { "index": "orders" },
    "dest":   { "index": "orders_v2" }
  }
  ```

---

## 💾 Snapshot & Restore

- #### Configure Repository Path

  Add the following to `config/elasticsearch.yml` (requires restart):
  
  ```yaml
  path.repo: ["snapshots"]
  ```

- #### Create Repository

  ```http
  PUT http://localhost:9200/_snapshot/first_backup
  ```
  ```json
  {
    "type": "fs",
    "settings": { "location": "first_backup" }
  }
  ```

- #### Create Snapshot

  ```http
  PUT http://localhost:9200/_snapshot/first_backup/snapshot1
  ```
  ```json
  {
    "indices": [],
    "metadata": {
      "taken_by": "Dzaru Rizky Fathan Fortuna",
      "taken_because": "backup before upgrading"
    }
  }
  ```
  
  > Leaving `indices` empty backs up **all indexes**.

- #### Restore Snapshot

  Before restoring, the target index must be **closed**:
  
  ```http
  POST http://localhost:9200/categories/_close
  POST http://localhost:9200/_snapshot/first_backup/snapshot1/_restore
  ```
  ```json
  { "indices": ["categories"] }
  ```
  
  Then reopen the index:
  
  ```http
  POST http://localhost:9200/categories/_open
  ```

- #### Delete Snapshot / Repository

  ```http
  DELETE http://localhost:9200/_snapshot/first_backup/snapshot1
  DELETE http://localhost:9200/_snapshot/first_backup
  ```

---

## 🐱 Cat API

CAT (Compact and Aligned Text) APIs are for **administrative use** — monitoring and maintaining Elasticsearch clusters.

```http
GET http://localhost:9200/_cat
```

| API | Description |
|-----|-------------|
| `/_cat/indices?v` | List all indexes with stats |
| `/_cat/aliases?v` | List all aliases |
| `/_cat/nodes?v` | List all nodes |
| `/_cat/health?v` | Cluster health |
| `/_cat/shards?v` | Shard details |
| `/_cat/snapshots?v` | Snapshot info |

`Example output for /_cat/indices?v:`

```
health status index        pri rep docs.count store.size
yellow open   customers_v2   1   1       1000    331.1kb
yellow open   categories     1   1      20000    864.2kb
yellow open   products       1   1          3      6.1kb
```

---

## 💡 Best Practices

- #### Index Design
  - Prefix index names with the app name to avoid conflicts: `appname_indexname`
  - Plan schema carefully — field types cannot be changed after creation
  - Use aliases so clients don't need to change when you create new indexes

- #### Mapping
  - Always define mappings explicitly instead of relying on Dynamic Mapping
  - Use `keyword` for structured data (IDs, emails, statuses) and `text` for full-text search
  - Use `nested` type only when necessary — it's expensive; prefer flat objects when possible
  - Use Multi Fields (`fields`) to support both full-text search and sorting on the same field

- #### Querying
  - Use `Term`/`Terms` for exact matches on `keyword`, numeric, and date fields
  - Use `Match` for full-text search on `text` fields
  - Prefer `filter` over `must` in Boolean queries when relevance score is not needed (faster, cacheable)
  - Use `Search After` instead of deep `from`/`size` pagination for large datasets

- #### Performance
  - Run `_update_by_query` after adding new field mappings to reindex existing documents
  - Use `Bulk API` for large-scale insert/update/delete operations
  - Avoid storing unnecessary fields in `_source` for large indexes

- #### Data Safety
  - Set up a `Snapshot Repository` pointing outside `/tmp` to persist backups
  - Take snapshots regularly before making schema changes
  - Always close an index before restoring a snapshot, then reopen after
