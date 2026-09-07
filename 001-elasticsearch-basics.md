# ⚡ Elasticsearch Basics

A practical reference guide for learning Elasticsearch from scratch — covering installation, data modeling, field types, indexes and mappings, the full document CRUD surface, the Search API, Query DSL, deep pagination, aliases and reindexing, snapshots, and the Cat APIs, worked through hands-on over plain HTTP.

---

## 📋 Table of Contents

- [Introduction](#-introduction)
  - [What Is Elasticsearch?](#what-is-elasticsearch)
  - [Apache Lucene](#apache-lucene)
  - [Terminology at a Glance](#terminology-at-a-glance)
  - [Elasticsearch Clients](#elasticsearch-clients)
- [Installation & Setup](#-installation--setup)
  - [Downloading Elasticsearch](#downloading-elasticsearch)
  - [Configuration](#configuration)
  - [Running the Server](#running-the-server)
  - [Verifying the Installation](#verifying-the-installation)
- [Data Modeling](#-data-modeling)
  - [Search-Driven Design](#search-driven-design)
  - [Schema Flexibility](#schema-flexibility)
  - [The Document ID (_id)](#the-document-id-_id)
  - [Embedded vs Reference](#embedded-vs-reference)
- [Field Data Types](#-field-data-types)
  - [Basic Types](#basic-types)
  - [Numeric Types](#numeric-types)
  - [Range Types](#range-types)
  - [Dynamic Field Mapping](#dynamic-field-mapping)
- [Index](#-index)
  - [Naming Rules](#naming-rules)
  - [Index APIs](#index-apis)
  - [Creating an Index](#creating-an-index)
  - [Listing Indexes](#listing-indexes)
  - [Deleting an Index](#deleting-an-index)
- [Mapping](#-mapping)
  - [Defining a Mapping](#defining-a-mapping)
  - [Keyword vs Text](#keyword-vs-text)
  - [Object Fields](#object-fields)
  - [Array Fields](#array-fields)
  - [Multi Fields](#multi-fields)
  - [Nested Fields](#nested-fields)
  - [Flattened Fields](#flattened-fields)
- [CRUD Operations](#-crud-operations)
  - [Document APIs](#document-apis)
  - [Create API](#create-api)
  - [Index API](#index-api)
  - [Get API](#get-api)
  - [Multi Get API](#multi-get-api)
  - [Update API](#update-api)
  - [Delete API](#delete-api)
  - [Bulk API](#bulk-api)
  - [Delete by Query API](#delete-by-query-api)
  - [Selecting Fields](#selecting-fields)
- [Search API](#-search-api)
  - [Running a Search](#running-a-search)
  - [Pagination](#pagination)
  - [Sorting](#sorting)
- [Query DSL](#-query-dsl)
  - [Match All](#match-all)
  - [Term Query](#term-query)
  - [Terms Query](#terms-query)
  - [Match Query](#match-query)
  - [Nested Query](#nested-query)
  - [Boolean Query](#boolean-query)
  - [Boost Score](#boost-score)
  - [Explain API](#explain-api)
  - [Relevance Scoring](#relevance-scoring)
- [Deep Pagination](#-deep-pagination)
  - [The Deep Paging Problem](#the-deep-paging-problem)
  - [Search After](#search-after)
  - [Scroll API](#scroll-api)
- [Alias & Reindex](#-alias--reindex)
  - [Why Aliases Matter](#why-aliases-matter)
  - [Creating & Removing an Alias](#creating--removing-an-alias)
  - [Listing Aliases](#listing-aliases)
  - [Reindex API](#reindex-api)
- [Snapshot & Restore](#-snapshot--restore)
  - [Configuring a Repository](#configuring-a-repository)
  - [Creating a Repository](#creating-a-repository)
  - [Creating a Snapshot](#creating-a-snapshot)
  - [Restoring a Snapshot](#restoring-a-snapshot)
  - [Deleting Snapshots & Repositories](#deleting-snapshots--repositories)
- [Cat API](#-cat-api)
- [Search Execution Flow](#-search-execution-flow)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)

---

## 🎯 Introduction

### What Is Elasticsearch?

- **Elasticsearch** is a free and open-source database management system built for **search engine** workloads
- It is **document-based** — data is stored as JSON documents, not rows in a table
- Every operation goes through a **RESTful API over HTTP**, so there is no dedicated shell or wire protocol to learn
- It is one of the most widely deployed search databases, used by large companies worldwide
- Documents are queried with **Query DSL**, a JSON query language, not SQL

> Reference: [elastic.co/elasticsearch](https://www.elastic.co/elasticsearch/) · [github.com/elastic/elasticsearch](https://github.com/elastic/elasticsearch)

### Apache Lucene

At the core of Elasticsearch sits **Apache Lucene** — a mature Information Retrieval library written in Java. Every feature in this guide is Lucene underneath: the inverted index, text analysis, and relevance scoring all belong to Lucene.

> **Key Insight:** Elasticsearch is best understood as a *distributed layer* over Lucene. Lucene gives you a single searchable index on one machine; Elasticsearch adds sharding, replication, the REST API, and the cluster. Most surprising behaviour in this guide — [immutable field types](#schema-flexibility), the [nested-field flattening problem](#nested-fields), the [10,000-document limit](#the-deep-paging-problem) — traces back to that split.

> Reference: [lucene.apache.org](https://lucene.apache.org/)

### Terminology at a Glance

The mental model is close to a relational database, only the names — and the flexibility — change:

| Relational database | Elasticsearch |
| --- | --- |
| Database | **Cluster** |
| Table | **Index** |
| Row / record | **Document** |
| Column | **Field** |
| Primary key | **`_id` field** |
| Schema | **Mapping** |
| `JOIN` | **Embedded object** or a manual **reference** |
| Partition | **Shard** |

### Elasticsearch Clients

Because Elasticsearch speaks plain HTTP, **any** HTTP client works — there is no official shell to install:

| Client | Best for | URL |
| --- | --- | --- |
| **Postman** | GUI client — build and save request collections | [postman.com](https://www.postman.com/) |
| **Insomnia** | Lightweight GUI alternative to Postman | [insomnia.rest](https://insomnia.rest/) |
| **JetBrains HTTP Client** | Requests as `.http` files, versioned next to your code | [jetbrains.com](https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html) |
| **VS Code REST Client** | The same idea inside VS Code | [marketplace.visualstudio.com](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) |
| **curl** | Terminal client for servers, headless systems, and scripting | [curl.se](https://curl.se/) |

Elasticsearch listens on port **`9200`** by default, which is what every example below assumes.

---

## 📦 Installation & Setup

### Downloading Elasticsearch

- Download Elasticsearch as an **archive file**
- Extract it anywhere on your computer
- Inside the archive you'll find `bin/elasticsearch` — the server process itself

> Reference: [elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch)

### Configuration

All configuration lives in `config/elasticsearch.yml`, in YAML format. Elasticsearch ships with **X-Pack**, a closed-source feature bundle made by Elastic; for learning purposes it is easiest to turn its security layer off:

```yaml
# config/elasticsearch.yml
cluster.name: dzarurizky
node.name: dzaru-1
xpack.security.enabled: false
http.port: 9200
path.data: data
path.logs: logs
```

> ⚠️ **Warning:** `xpack.security.enabled: false` disables authentication, authorization, and TLS entirely — anyone who can reach port `9200` gets full control of the cluster. It is a learning setting, never a deployment setting.

### Running the Server

```bash
# Start the Elasticsearch server
./bin/elasticsearch
```

Stop the server with `Ctrl + C`.

> **Note:** `path.data` is where every index physically lives, and it is locked by whichever process owns it — a second node pointed at the same folder will refuse to start.

### Verifying the Installation

```bash
curl http://localhost:9200
```

The root endpoint returns the node's identity, which doubles as a health check:

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

> **Tip:** the `lucene_version` in that response is worth noting — it tells you which Lucene release is doing the actual work, and it is what release notes on analysis or scoring changes refer to.

---

## 🧩 Data Modeling

### Search-Driven Design

Relational databases are modeled around **normalization** — eliminate duplication, then join at read time. Elasticsearch is modeled around **how the application searches**: shape each document so that one query can answer one question, even if that means storing the same value twice.

> **Key Insight:** there are no joins. That single constraint is what turns modeling from a normalization exercise into a query-planning exercise — you decide the shape by listing the searches your application performs, not the entities it owns.

### Schema Flexibility

- **No predefined schema required** — index a document and Elasticsearch infers the fields via [Dynamic Mapping](#dynamic-field-mapping)
- **New fields can be added** to an existing index at any time
- **Existing field types cannot be changed** — the type is fixed once the first document sets it
- **Best practice** — define the [mapping](#-mapping) explicitly up front rather than letting it be guessed

> **Gotcha:** field-type immutability is the most consequential rule in Elasticsearch. Changing `price` from `text` to `long` is not an `ALTER TABLE` — it means creating a new index, [reindexing](#reindex-api) into it, and swapping an [alias](#-alias--reindex). Plan the mapping before the first document, not after the first million.

### The Document ID (_id)

- **Always present** — every document is addressed by `_id`
- **Always a string**, regardless of what you pass in
- **Single field only** — composite primary keys do not exist
- **Auto-generated** when you use the [Index API](#index-api) without an ID

> **Tip:** supplying your own `_id` (a username, an order number) makes writes idempotent — re-indexing the same document overwrites rather than duplicates. Letting Elasticsearch generate one is faster, because it can skip the "does this ID already exist?" lookup.

### Embedded vs Reference

| Use **Embedded** when | Use **Reference** when |
| --- | --- |
| Documents are interdependent | Documents can stand alone |
| The nested data is always needed when reading the parent | The related data is not always needed when reading |
| You never need to update the inner data on its own | You need to manipulate the referenced data directly |

> **Note:** references are not joins. Nothing enforces that `customer_id` points at a real customer, and reading both sides means two round trips from your application. That cost is the whole reason to embed when you can.

---

## 📊 Field Data Types

### Basic Types

| Type | Description |
| --- | --- |
| `binary` | Binary data as a Base64-encoded string |
| `boolean` | `true` or `false` |
| `date` | Date and time down to milliseconds |
| `date_nanos` | Date and time down to nanoseconds |
| `ip` | An IPv4 or IPv6 address |
| `keyword` | Structured text — id, email, hostname, zip code, status |
| `text` | Free text, analyzed for full-text search |
| `version` | A semantic version string |

> Reference: [semver.org](https://semver.org/)

### Numeric Types

| Type | Description |
| --- | --- |
| `long` | 64-bit signed integer (−2^63 to 2^63−1) |
| `integer` | 32-bit signed integer (−2^31 to 2^31−1) |
| `short` | 16-bit signed integer (−32,768 to 32,767) |
| `byte` | 8-bit signed integer (−128 to 127) |
| `double` | 64-bit IEEE 754 floating point |
| `float` | 32-bit IEEE 754 floating point |
| `half_float` | 16-bit IEEE 754 floating point |
| `scaled_float` | Floating point stored internally as a `long` with a fixed scaling factor |
| `unsigned_long` | 64-bit unsigned integer (0 to 2^64−1) |

> **Tip:** pick the smallest type that fits. Numeric range queries get cheaper as the field gets narrower, and `scaled_float` is the right choice for money — store `19.99` with `scaling_factor: 100` and you get exact arithmetic instead of binary floating-point drift.

### Range Types

| Type | Description |
| --- | --- |
| `integer_range` | Min/max range of integers |
| `long_range` | Min/max range of longs |
| `float_range` | Min/max range of floats |
| `double_range` | Min/max range of doubles |
| `date_range` | Min/max range of dates |
| `ip_range` | Min/max range of IP addresses |

> **Note:** a range type stores the *interval itself* in one field — a hotel booking, a promo window, a subnet. That lets you ask "which intervals contain this point?", which is the inverse of what a normal range query does.

### Dynamic Field Mapping

When no mapping is defined, Elasticsearch infers the type from the first value it sees:

| JSON type | Elasticsearch type |
| --- | --- |
| `null` | No field added |
| `true` / `false` | `boolean` |
| `double` | `float` |
| `long` | `long` |
| `array` | Determined by the first non-null element |
| `string` | `date`, `float`, `long` or `text` — auto-detected |

> **Gotcha:** dynamic mapping guesses from **one** document. A field that arrives as `"12345"` in the first document becomes `long`, and the next document sending `"ABC-123"` is rejected outright. Worse, the guess is permanent — see [Schema Flexibility](#schema-flexibility).

---

## 📁 Index

An **index** is the equivalent of a table in a relational database — a named collection of documents that share a [mapping](#-mapping).

### Naming Rules

- Must be **lowercase**
- No special characters except `-`, `+` and `_`, and none of them at the start
- Maximum **255 bytes**
- When several applications share a cluster, prefix the name: `appname_indexname`

### Index APIs

| Request | Description |
| --- | --- |
| `PUT /<index>` | Create an index |
| `GET /<index>` | Get an index's settings and mapping |
| `DELETE /<index>` | Delete an index and everything in it |
| `GET /_cat/indices?v` | List all indexes with their stats |
| `POST /<index>/_close` | Close an index — it stays on disk but rejects reads and writes |
| `POST /<index>/_open` | Reopen a closed index |

### Creating an Index

```http
PUT http://localhost:9200/customers
```

### Listing Indexes

```http
GET http://localhost:9200/_cat/indices?v
```

### Deleting an Index

```http
DELETE http://localhost:9200/customers
```

> ⚠️ **Warning:** deleting an index removes **every document** inside it, and there is no confirmation prompt and no undo. Take a [snapshot](#-snapshot--restore) first.

---

## 🧭 Mapping

**Mapping** is the schema definition for an index — which fields exist, and what type each one is. Dynamic Mapping will invent one for you, but defining it manually is strongly recommended.

### Defining a Mapping

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

> **Note:** `_mapping` only ever **adds**. New fields are accepted; changing the type of an existing one is rejected. The three detection settings above apply to fields you *didn't* declare — they are the rules dynamic mapping follows for everything not listed in `properties`.

### Keyword vs Text

This is the first decision to get right for every string field, and the two types behave nothing alike:

| | `keyword` | `text` |
| --- | --- | --- |
| Analyzed? | No — stored verbatim as one token | Yes — split into terms by an analyzer |
| Query with | [`term`](#term-query) / [`terms`](#terms-query) | [`match`](#match-query) |
| Sort & aggregate | Yes | No, not by default |
| Good for | IDs, emails, statuses, tags, hostnames | Titles, descriptions, comments, article bodies |

> **Key Insight:** `text` fields are analyzed at *both* index time and query time, and that is what makes `match` work — `"BCA Digital"` becomes the terms `bca` and `digital` on the way in, and the query is analyzed the same way on the way out so the two can meet. A `keyword` field skips analysis entirely, which is exactly why it can be sorted on and a `text` field cannot. When you need both behaviours on one field, see [Multi Fields](#multi-fields).

### Object Fields

For nested objects, declare `properties` inside `properties`:

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

Query the inner fields with dot notation: `address.city`.

### Array Fields

Arrays need no special type — declare the type of the array's *contents*:

```json
{
  "properties": {
    "hobbies": { "type": "text" }
  }
}
```

> **Note:** every field in Elasticsearch is implicitly multi-valued. `"hobbies": "gaming"` and `"hobbies": ["gaming", "reading"]` fit the same mapping, because the inverted index simply records more terms for the second one.

### Multi Fields

Index one source field as several types at once — the standard fix for needing full-text search *and* sorting on the same value:

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

Search `name` with `match`, and sort or aggregate on `name.raw` with dot notation.

After adding a multi-field to an index that already holds documents, reindex them in place so the new sub-field is populated:

```http
POST http://localhost:9200/products/_update_by_query
```

```json
{ "query": { "match_all": {} } }
```

> **Gotcha:** a new mapping applies only to documents indexed *after* it. Existing documents keep whatever was indexed at write time, so `name.raw` stays empty until `_update_by_query` rewrites them — the query returns results, just silently incomplete ones.

### Nested Fields

Use the `nested` type when you need accurate queries against an **array of objects**:

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

> **Key Insight:** a plain object array is *flattened* by Lucene — `[{first:"Dzaru", last:"Fathan"}, {first:"Budi", last:"Nugraha"}]` is stored as `first: [Dzaru, Budi]`, `last: [Fathan, Nugraha]`, and the relationship between the pairs is gone. Searching for `first:Dzaru AND last:Nugraha` then matches a person who does not exist. The `nested` type fixes it by indexing each array element as its own hidden Lucene document.

> ⚠️ **Warning:** that fix is not free — one document with 50 nested children is 51 documents on disk, and every query against them needs a [`nested` query](#nested-query) to join back. Reach for `nested` only when a cross-field false match would actually be wrong.

### Flattened Fields

For objects whose keys you cannot know in advance — user-defined labels, arbitrary metadata — use `flattened`:

```json
{
  "properties": {
    "labels": { "type": "flattened" }
  }
}
```

> **Tip:** `flattened` exists to prevent **mapping explosion**. Without it, a thousand distinct user-supplied keys become a thousand mapped fields, and cluster state grows with every new one. A flattened field is a single entry in the mapping no matter how many keys arrive — the trade-off being that every leaf value is indexed as a `keyword`, so numeric and date queries on them are off the table.

---

## 📝 CRUD Operations

### Document APIs

| Request | Description |
| --- | --- |
| `POST /<index>/_create/<id>` | Create a document, failing if the ID exists |
| `POST /<index>/_doc/<id>` | Create or replace a document |
| `GET /<index>/_doc/<id>` | Get a document with its metadata |
| `GET /<index>/_source/<id>` | Get only the document body |
| `HEAD /<index>/_doc/<id>` | Check whether a document exists |
| `POST /_mget` | Get many documents in one request |
| `POST /<index>/_update/<id>` | Update selected fields of a document |
| `DELETE /<index>/_doc/<id>` | Delete a document by ID |
| `POST /_bulk` | Mix many write operations into one request |
| `POST /<index>/_delete_by_query` | Delete every document matching a query |

### Create API

A safe create — returns `409 Conflict` if the document already exists:

```http
POST http://localhost:9200/customers/_create/dzaru
```

```json
{
  "name": "Dzaru Rizky Fathan Fortuna",
  "register_at": "2025-03-03 12:00:00"
}
```

### Index API

Create **or replace** — overwrites an existing document without complaint:

```http
POST http://localhost:9200/products/_doc/3
```

```json
{ "name": "Pop Mie Rasa Bakso", "price": 2500 }
```

> **Gotcha:** `_doc` replaces the *whole* document — every field you leave out is gone. `_create` refuses to overwrite, and [`_update`](#update-api) only touches the fields you name. Reach for `_doc` deliberately, never by accident.

### Get API

Retrieve a document by ID, returning `404` when it does not exist:

```http
GET http://localhost:9200/customers/_doc/dzaru
```

Retrieve just the document body, with no metadata wrapper:

```http
GET http://localhost:9200/customers/_source/dzaru
```

Check for existence without transferring the document — `200` if it exists, `404` if not:

```http
HEAD http://localhost:9200/customers/_doc/dzaru
```

> **Note:** the Get API is *real-time* — it can read a document the instant it is written. Search is not: a new document becomes searchable only after the next refresh, one second later by default. That is why a document you just indexed appears under `_doc/<id>` but is missing from `_search`.

### Multi Get API

Fetch documents from several indexes in a single round trip:

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

### Update API

Change specific fields without replacing the whole document:

```http
POST http://localhost:9200/products/_update/3
```

```json
{ "doc": { "price": 5000 } }
```

> **Key Insight:** partial update is a convenience, not a capability of the storage engine. Lucene documents are immutable, so Elasticsearch fetches the old document, merges your `doc` into it, indexes the result, and marks the old one deleted. The API is partial; the write underneath never is.

### Delete API

```http
DELETE http://localhost:9200/customers/_doc/spammer
```

### Bulk API

Mix create, index, update and delete operations into a single request. The body is **newline-delimited JSON** — an action line, then a source line, repeated:

```json
{ "create": { "_index": "customers", "_id": "joko" } }
{ "name": "Joko Morro", "register_at": "2023-10-10 00:00:00" }
{ "index": { "_index": "customers", "_id": "budi" } }
{ "name": "Budi Nugraha", "register_at": "2023-10-10 00:00:00" }
{ "update": { "_index": "products", "_id": "1" } }
{ "doc": { "price": 2500 } }
{ "delete": { "_index": "customers", "_id": "spammer" } }
```

> **Gotcha:** two rules about that body trip up everyone. Each line must be a **single line** of JSON — no pretty-printing — and the body must end with a **trailing newline**. Also note `delete` takes no source line, since there is nothing to send.

> **Tip:** a `200` response does not mean every operation succeeded. Bulk reports per-item results, so check the top-level `errors` flag and then the individual `items` — a partial failure looks like a success from the status code alone.

### Delete by Query API

Delete every document matching a [query](#-query-dsl):

```http
POST http://localhost:9200/categories/_delete_by_query
```

```json
{ "query": { "match": { "name": "wrong" } } }
```

> ⚠️ **Warning:** run the same query through `_search` first and look at what comes back. `{ "query": { "match_all": {} } }` here empties the index, and deleted documents cannot be recovered.

### Selecting Fields

Control which fields come back with `_source_includes` and `_source_excludes`:

```http
GET http://localhost:9200/order/_search?_source_includes=total,customer_id
GET http://localhost:9200/products/_search?_source_excludes=price
```

> **Tip:** this trims the response, not the work — Elasticsearch still finds and loads the full document before filtering it. The win is network and parsing cost, which is substantial on wide documents and large result sets.

---

## 🔍 Search API

### Running a Search

```http
POST http://localhost:9200/<index_name>/_search
```

With no query in the body, the Search API defaults to [`match_all`](#match-all).

### Pagination

```http
GET http://localhost:9200/products/_search?size=10&from=0
```

| Parameter | Description | Default |
| --- | --- | --- |
| `size` | Number of documents per page | `10` |
| `from` | Starting document offset | `0` |

> **Note:** `from` and `size` work up to a ceiling of 10,000 documents. Past that you need [Search After](#search-after) — the reason why is [the deep paging problem](#the-deep-paging-problem).

### Sorting

```http
POST http://localhost:9200/products/_search?sort=price:asc
```

Several sort fields are comma-separated, applied left to right: `field1:asc,field2:desc`.

> **Gotcha:** sorting on a `text` field fails. Sorting needs the value whole, and analysis has already shredded it into terms — sort on a `keyword` field, or on the `keyword` half of a [multi-field](#multi-fields) such as `name.raw`.

---

## 🔎 Query DSL

### Match All

Returns every document — the default when no query is given:

```json
{
  "query": { "match_all": {} },
  "size": 5,
  "from": 0,
  "sort": [{ "username": { "order": "desc" } }]
}
```

### Term Query

An exact, unanalyzed value match. Right for `keyword`, `boolean`, numeric and `date` fields — **not** for `text`:

```json
{
  "query": {
    "term": { "gender": "Female" }
  }
}
```

> **Gotcha:** `term` on a `text` field is the single most common Query DSL mistake. The query value is *not* analyzed while the indexed field *was*, so searching `text` for `"Female"` finds nothing — the index holds the lowercased term `female`. Use [`match`](#match-query) on `text`, `term` on `keyword`.

### Terms Query

The equivalent of SQL's `IN` — also unanalyzed, so the same `keyword`-only rule applies:

```json
{
  "query": {
    "terms": {
      "username": ["username1", "username2", "username3"]
    }
  }
}
```

### Match Query

Full-text search. The query string is run through the **same analyzer as the field**, which is what makes it work on `text`:

```json
{
  "query": {
    "match": { "banks.name": "BCA" }
  }
}
```

By default the resulting terms are OR-ed together. Require all of them with `operator`:

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

> **Note:** the default `StandardAnalyzer` lowercases and splits on non-alphanumerics, so `eko@example.com` is indexed as three terms — `eko`, `example`, `com`. A `match` for `example` therefore hits that document, which is usually a surprise. If the whole address must match as one unit, the field should be a `keyword`.

### Nested Query

Required to search inside a [`nested`](#nested-fields) field — a plain `match` on `children.first_name` will not reach it:

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

> **Key Insight:** the `path` is what makes this correct. Both `must` clauses are evaluated against **one** hidden nested document at a time, so the match means "a single child named Dzaru Fathan" rather than "some child named Dzaru, and some child named Fathan". That is precisely the false match a plain object array cannot avoid.

### Boolean Query

Combine several queries across several fields:

| Clause | Description |
| --- | --- |
| `must` | Every condition must match, and each contributes to the score |
| `filter` | Every condition must match, but none affect the score |
| `must_not` | No condition may match |
| `should` | At least one condition should match, and matches raise the score |

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

> **Key Insight:** `must` and `filter` select exactly the same documents — the difference is scoring. Because `filter` computes no score, its results are cacheable and reused across queries, which makes it markedly faster. Use `must` only for the clauses that should influence ranking, and `filter` for everything that is merely a condition.

> **Gotcha:** `minimum_should_match` defaults to `1` when `should` stands alone, but to `0` as soon as a `must` or `filter` is present — at which point `should` becomes a pure score booster that no longer filters anything. Set it explicitly whenever you mean "and at least one of these".

### Boost Score

Weight individual clauses to control relevance:

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

> **Note:** `boost` multiplies a clause's contribution, so `2` doubles it and `0` cancels it. A `boost: 0` clause still *matches* — it just adds nothing to the score, which is not the same as excluding the document.

### Explain API

Ask Elasticsearch to show exactly how one document's score was computed:

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

> **Tip:** `_explain` is the honest answer to "why did this document rank there?" — and to "why did it not match at all?". The response breaks the score down clause by clause, term frequency included, which is far quicker than guessing at the analyzer.

### Relevance Scoring

Elasticsearch ranks results with **BM25** (Okapi BM25), the successor to TF-IDF. It weighs three things: how often the term appears in the document, how rare the term is across the index, and how long the document is.

> **Key Insight:** BM25's improvement over TF-IDF is **term-frequency saturation** — the tenth occurrence of a word adds far less than the second. That is what makes keyword stuffing ineffective, and it is why a short document mentioning your term twice can outrank a long one mentioning it twenty times.

> Reference: [elastic.co/blog — the BM25 algorithm and its variables](https://www.elastic.co/blog/practical-bm25-part-2-the-bm25-algorithm-and-its-variables)

---

## 🔄 Deep Pagination

### The Deep Paging Problem

Search results are capped at **10,000 documents** by default, via the `index.max_result_window` setting.

> **Key Insight:** the limit is a property of distributed search, not an arbitrary quota. To return page 500 of a 10-shard index, every shard must produce its own top 5,010 hits and ship them to the coordinating node, which sorts 50,100 candidates and discards all but ten. Cost grows with `from`, not with `size` — which is why raising the ceiling trades a clear error for a slow cluster.

### Search After

The supported way past the limit: sort the results, then ask for what comes *after* the last row you saw.

```json
{
  "size": 100,
  "query": { "match_all": {} },
  "sort": [{ "id": { "order": "asc" } }],
  "search_after": ["10087"]
}
```

Take the `sort` values of the last hit in each response and pass them as `search_after` for the next page.

> **Gotcha:** three requirements are easy to miss. `from` must be `0` or omitted — Elasticsearch rejects the combination outright. The `sort` must be **deterministic**, so add a unique tiebreaker such as `_id` when the primary sort field can repeat. And the values in `search_after` must line up with the `sort` array, in the same order and the same number.

> **Tip:** for a consistent snapshot across pages, open a **point in time** (`POST /<index>/_pit?keep_alive=1m`) and search against that PIT id. Without one, documents indexed mid-pagination can shift rows between requests and you may see a hit twice, or never.

### Scroll API

The older way to walk an entire index. It still works, but it is **no longer recommended** — Search After with a point in time supersedes it.

> Reference: [elastic.co — scroll API](https://www.elastic.co/guide/en/elasticsearch/reference/current/scroll-api.html)

---

## 🔀 Alias & Reindex

### Why Aliases Matter

Since a field type [cannot be changed](#schema-flexibility), every schema change means a **new index**. An alias is a stable name pointing at whichever index is current, so clients never have to know that `customers` is really `customers_v2` today and `customers_v3` next month.

### Creating & Removing an Alias

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

> **Key Insight:** every action in one `_aliases` call is applied **atomically**. That is the whole point — add and remove together and the alias never briefly points at both indexes or at none, so a schema migration becomes a zero-downtime cutover.

### Listing Aliases

```http
GET http://localhost:9200/_aliases
```

### Reindex API

Copy documents from one index into another, applying the destination's mapping as they land:

```http
POST http://localhost:9200/_reindex
```

```json
{
  "source": { "index": "orders" },
  "dest":   { "index": "orders_v2" }
}
```

> **Note:** create the destination index **with its new mapping first**. Reindexing into an index that does not exist yet lets [dynamic mapping](#dynamic-field-mapping) guess the types all over again — which defeats the entire reason for reindexing.

> **Tip:** on a large index, add `?wait_for_completion=false`. Elasticsearch returns a task id immediately and you track progress with `GET /_tasks/<task_id>` instead of holding an HTTP connection open for an hour.

---

## 💾 Snapshot & Restore

### Configuring a Repository

A repository is where backups are written. Its location must be allow-listed in `config/elasticsearch.yml`:

```yaml
path.repo: ["snapshots"]
```

> ⚠️ **Warning:** this setting requires a **restart** to take effect, and every path is relative to it. Point it somewhere durable — a directory under `/tmp` disappears on reboot, taking every backup with it.

### Creating a Repository

```http
PUT http://localhost:9200/_snapshot/first_backup
```

```json
{
  "type": "fs",
  "settings": { "location": "first_backup" }
}
```

> **Note:** `type: "fs"` is a shared filesystem repository — on a multi-node cluster the path must be mounted on **every** node, or snapshots of the shards living elsewhere will fail. Production clusters typically use an object-store repository instead.

### Creating a Snapshot

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

Leaving `indices` empty snapshots **all** indexes.

> **Tip:** snapshots are **incremental**. The second snapshot into the same repository only stores the Lucene segments that changed since the first, so taking them often is far cheaper than it looks — and each one is still restorable on its own.

### Restoring a Snapshot

An index cannot be restored while it is open, so close it first:

```http
POST http://localhost:9200/categories/_close
POST http://localhost:9200/_snapshot/first_backup/snapshot1/_restore
```

```json
{ "indices": ["categories"] }
```

Then reopen it:

```http
POST http://localhost:9200/categories/_open
```

> **Tip:** closing the live index means downtime. Add `rename_pattern` and `rename_replacement` to the restore body and the snapshot lands in a *differently named* index instead — the safe way to verify a backup, or to recover a few documents, without touching production data.

### Deleting Snapshots & Repositories

```http
DELETE http://localhost:9200/_snapshot/first_backup/snapshot1
DELETE http://localhost:9200/_snapshot/first_backup
```

> **Note:** deleting a *repository* only removes Elasticsearch's registration of it — the files stay on disk. Deleting a *snapshot* does delete data, and because snapshots are incremental, it removes only the segments no remaining snapshot still needs.

---

## 🐱 Cat API

The **CAT** (Compact and Aligned Text) APIs return human-readable tables instead of JSON. They exist for administration — monitoring and maintaining a cluster from a terminal.

```http
GET http://localhost:9200/_cat
```

| API | Description |
| --- | --- |
| `/_cat/indices?v` | List all indexes with their stats |
| `/_cat/aliases?v` | List all aliases and the indexes behind them |
| `/_cat/nodes?v` | List all nodes in the cluster |
| `/_cat/health?v` | Cluster health |
| `/_cat/shards?v` | Per-shard detail, including where each one lives |
| `/_cat/snapshots?v` | Snapshot information |

Example output for `/_cat/indices?v`:

```
health status index        pri rep docs.count store.size
yellow open   customers_v2   1   1       1000    331.1kb
yellow open   categories     1   1      20000    864.2kb
yellow open   products       1   1          3      6.1kb
```

> **Note:** `?v` adds the header row — without it you get bare columns. `yellow` health on a single-node cluster is normal, not a problem: each index asks for one replica (`rep 1`) and there is no second node to put it on.

> **Tip:** the CAT APIs are built for humans and their columns are not a stable contract. For scripts and dashboards, use the JSON APIs (`GET /_cluster/health`, `GET /<index>/_stats`) instead.

---

## 🔁 Search Execution Flow

Knowing the order in which Elasticsearch processes a search is what makes the rest of this guide click. Every request runs two phases:

```
Client (HTTP request)
  ↓
Coordinating node                 the node you happened to call
  ↓
── Query phase ──────────────────  broadcast to every shard
  ↓
Text Analysis                     char filters → tokenizer → token filters
  ↓                               (the query string, analyzed like the field was)
Inverted Index Lookup             matching doc ids, per shard
  ↓
Filter context                    filter, must_not → no score, cacheable
  ↓
Score context                     must, should → BM25 relevance
  ↓
Per-shard sort + from/size        each shard returns its own top (from + size)
  ↓
Coordinating node merge           sorts the candidates, keeps the real top N
  ↓
── Fetch phase ──────────────────  only the surviving N documents
  ↓
_source retrieval + projection    _source_includes / _source_excludes
  ↓
Response
```

This explains the design decisions throughout this guide:

| Question | Answer |
| --- | --- |
| Why does `term` fail on a `text` field? | Analysis happens at index time *and* query time. `term` skips the query-time half, so its raw value never matches the analyzed terms in the index |
| Why is `filter` faster than `must`? | It runs in filter context, where no score is computed — so the result set can be cached and reused across queries |
| Why can't a `text` field be sorted on? | Sorting needs one whole value per document, and analysis already replaced it with a bag of terms |
| Why is deep `from` slow? | Each shard must build its own top `from + size` before the merge, so the work grows with the offset, not the page size |
| Why does `_source_excludes` not speed up the search? | It applies in the fetch phase, after the matching documents are already found and loaded |
| Why is a just-indexed document missing from `_search`? | Search reads Lucene segments, which are only made visible by a refresh — one second later by default. `GET /_doc/<id>` bypasses that and is real-time |

---

## 🎯 Quick Reference

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| **Index** | Container for documents — a table | `PUT /customers` |
| **Mapping** | Field schema for an index | `PUT /customers/_mapping` |
| **Keyword vs Text** | Exact values vs full-text | `"type": "keyword"` / `"type": "text"` |
| **Multi field** | One value, several types | `"fields": { "raw": { "type": "keyword" } }` |
| **Nested** | Accurate queries on object arrays | `"type": "nested"` + `nested` query |
| **Flattened** | Unknown keys without mapping explosion | `"type": "flattened"` |
| **Create** | Add a document, fail on conflict | `POST /<index>/_create/<id>` |
| **Index doc** | Add or replace a document | `POST /<index>/_doc/<id>` |
| **Get** | Read one document, real-time | `GET /<index>/_doc/<id>` |
| **Update** | Change selected fields | `POST /<index>/_update/<id>` |
| **Delete** | Remove a document | `DELETE /<index>/_doc/<id>` |
| **Bulk** | Many writes, one round trip | `POST /_bulk` (newline-delimited JSON) |
| **Search** | Run a query | `POST /<index>/_search` |
| **Pagination** | Page through results | `?size=10&from=0` |
| **Exact match** | Unanalyzed match on `keyword` | `term`, `terms` |
| **Full-text** | Analyzed match on `text` | `match` with `operator: AND` / `OR` |
| **Combine** | Several conditions at once | `bool` with `must`, `filter`, `must_not`, `should` |
| **Relevance** | Rank the results | BM25, tuned with `boost` |
| **Debug scoring** | See how a score was built | `POST /<index>/_explain/<id>` |
| **Deep paging** | Past 10,000 documents | `search_after` + a deterministic `sort` |
| **Alias** | Stable name over a changing index | `POST /_aliases` (atomic add + remove) |
| **Reindex** | Copy into a new mapping | `POST /_reindex` |
| **Snapshot** | Back up the data | `PUT /_snapshot/<repo>/<snapshot>` |
| **Restore** | Bring the data back | `POST /_snapshot/<repo>/<snapshot>/_restore` |
| **Administration** | Inspect the cluster | `GET /_cat/indices?v`, `/_cat/health?v` |

---

## 💡 Best Practices

**✅ Do This**

- **Model around your searches, not around normalization** — list the queries the application runs, then shape documents so one query answers one question
- **Define the mapping explicitly before the first document** — field types are permanent, and dynamic mapping guesses them from a single value
- **Use `keyword` for structured data** (IDs, emails, statuses, tags) and `text` for anything a human types and searches
- **Add a multi-field when you need both** — `text` for `match`, a `keyword` sub-field for sorting and aggregating
- **Prefer `filter` over `must`** for every clause that is a condition rather than a ranking signal — it skips scoring and is cacheable
- **Set `minimum_should_match` explicitly** whenever a `should` block sits next to a `must` or `filter`
- **Use `search_after` with a deterministic sort** for large result sets, and open a point in time when consistency across pages matters
- **Front every index with an alias** and swap it atomically, so schema migrations need no client changes
- **Create the destination index with its mapping before reindexing** into it
- **Run `_update_by_query` after adding a field mapping** so existing documents pick up the new sub-field
- **Use the Bulk API for large writes**, and check the `errors` flag and per-item results rather than trusting the HTTP status
- **Reach for `_explain` instead of guessing** when a document ranks oddly or fails to match
- **Point `path.repo` at durable storage, snapshot before every schema change**, and test a restore into a renamed index
- **Turn `xpack.security.enabled` back on outside of learning** — with TLS, real credentials, and the port firewalled

**❌ Avoid This**

- **Using `term` or `terms` on a `text` field** — the query value is not analyzed while the field was, so it matches nothing
- **Sorting or aggregating on a `text` field** — sort on a `keyword` field or a `keyword` multi-field instead
- **Relying on dynamic mapping in production** — one unlucky first document fixes the wrong type permanently
- **Running `_delete_by_query` without previewing the query through `_search`** — there is no confirmation and no undo
- **Reaching for `_doc` when you meant `_update`** — indexing a document replaces it entirely, dropping every field you left out
- **Pretty-printing a Bulk body, or forgetting its trailing newline** — both make the request invalid
- **Paging deep with `from` and `size`** — cost grows with the offset, which is what the 10,000-document window is protecting you from
- **Raising `index.max_result_window` to work around that limit** — it trades a clear error for a slow, memory-hungry cluster
- **Combining `from` with `search_after`** — `from` must be `0`, or the request is rejected
- **Defaulting to `nested` for every object array** — each element becomes its own hidden document and every query needs a `nested` wrapper
- **Letting user-supplied keys become mapped fields** — use `flattened` before mapping explosion bloats the cluster state
- **Assuming a document is searchable the moment it is written** — search waits for a refresh; only `GET /_doc/<id>` is real-time
- **Leaving Elasticsearch open on port `9200` with security disabled** — the learning configuration is not a deployment configuration

> Reference: [elastic.co/guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html) · [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) · [Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)
