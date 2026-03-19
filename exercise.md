# 🛒 Elasticsearch Hands-On Practice — E-Commerce Search Engine

A complete hands-on scenario that covers all core Elasticsearch concepts: index creation, mapping, CRUD, querying, aliases, reindex, snapshots, and the Cat API — all in one real-world e-commerce use case.

---

## 📖 Scenario

You are building a search engine for an e-commerce platform called **TokoBaju** — an online fashion store. You will manage:

- **Products** — clothing items with price, category, tags, and stock
- **Customers** — registered buyers with profile and bank info
- **Orders** — purchase records linking customers and products

By the end of this practice, you will have a fully functional search system covering every topic in the Elasticsearch guide.

---

## 🗂️ Table of Contents

1. [Setup & Verify](#1-setup--verify)
2. [Create Indexes & Mappings](#2-create-indexes--mappings)
3. [Insert Data (CRUD)](#3-insert-data-crud)
4. [Search & Query DSL](#4-search--query-dsl)
5. [Alias & Reindex](#5-alias--reindex)
6. [Snapshot & Restore](#6-snapshot--restore)
7. [Monitor with Cat API](#7-monitor-with-cat-api)
8. [Challenge Tasks](#-challenge-tasks)

---

## 1. Setup & Verify

Before starting, make sure Elasticsearch is running.

```bash
./bin/elasticsearch
```

Verify the server is up:

```bash
curl http://localhost:9200
```

Expected output:

```json
{
  "name": "dzaru-1",
  "cluster_name": "dzarurizky",
  "tagline": "You Know, for Search"
}
```

---

## 2. Create Indexes & Mappings

> 💡 **Why manual mapping?** Dynamic mapping may guess the wrong types. Always define your schema explicitly for production use.

### 2a. Create `tokobaju_products_v1` Index

```http
PUT http://localhost:9200/tokobaju_products_v1
```

Define the mapping:

```http
PUT http://localhost:9200/tokobaju_products_v1/_mapping
```

```json
{
  "properties": {
    "name":        { "type": "text", "fields": { "raw": { "type": "keyword" } } },
    "description": { "type": "text" },
    "category":    { "type": "keyword" },
    "brand":       { "type": "keyword" },
    "price":       { "type": "integer" },
    "stock":       { "type": "integer" },
    "tags":        { "type": "keyword" },
    "sizes":       { "type": "keyword" },
    "created_at":  { "type": "date", "format": "yyyy-MM-dd" }
  }
}
```

> 📌 `name` uses **Multi Fields** — `name` for full-text search, `name.raw` for exact match / sorting.

### 2b. Create `tokobaju_customers_v1` Index

```http
PUT http://localhost:9200/tokobaju_customers_v1
```

```http
PUT http://localhost:9200/tokobaju_customers_v1/_mapping
```

```json
{
  "properties": {
    "username":   { "type": "keyword" },
    "full_name":  { "type": "text" },
    "email":      { "type": "keyword" },
    "gender":     { "type": "keyword" },
    "birth_date": { "type": "date", "format": "yyyy-MM-dd" },
    "address": {
      "properties": {
        "street":   { "type": "text" },
        "city":     { "type": "keyword" },
        "province": { "type": "keyword" },
        "zip_code": { "type": "keyword" }
      }
    },
    "banks": {
      "type": "nested",
      "properties": {
        "name":           { "type": "keyword" },
        "account_number": { "type": "keyword" }
      }
    },
    "labels": { "type": "flattened" }
  }
}
```

> 📌 `banks` uses **Nested** type because each customer can have multiple banks and we need accurate per-bank queries.  
> 📌 `labels` uses **Flattened** type for arbitrary dynamic tags (e.g., `{ "vip": true, "segment": "fashion-lover" }`).

### 2c. Create `tokobaju_orders_v1` Index

```http
PUT http://localhost:9200/tokobaju_orders_v1
```

```http
PUT http://localhost:9200/tokobaju_orders_v1/_mapping
```

```json
{
  "properties": {
    "order_code":  { "type": "keyword" },
    "customer_id": { "type": "keyword" },
    "total":       { "type": "long" },
    "status":      { "type": "keyword" },
    "ordered_at":  { "type": "date", "format": "yyyy-MM-dd HH:mm:ss" },
    "items": {
      "type": "nested",
      "properties": {
        "product_id": { "type": "keyword" },
        "name":       { "type": "text" },
        "qty":        { "type": "integer" },
        "price":      { "type": "integer" }
      }
    }
  }
}
```

---

## 3. Insert Data (CRUD)

### 3a. Bulk Insert Products

```http
POST http://localhost:9200/_bulk
```

```
{ "create": { "_index": "tokobaju_products_v1", "_id": "prod-1" } }
{ "name": "Kaos Polos Oversize", "description": "Kaos oversize bahan cotton combed 30s, adem dan nyaman", "category": "kaos", "brand": "BasicWear", "price": 85000, "stock": 150, "tags": ["oversize", "casual", "cotton"], "sizes": ["S", "M", "L", "XL", "XXL"], "created_at": "2024-01-10" }
{ "create": { "_index": "tokobaju_products_v1", "_id": "prod-2" } }
{ "name": "Kemeja Flannel Kotak Premium", "description": "Kemeja flannel motif kotak, cocok untuk casual dan semi-formal", "category": "kemeja", "brand": "FlannelCo", "price": 175000, "stock": 80, "tags": ["flannel", "casual", "premium"], "sizes": ["M", "L", "XL"], "created_at": "2024-01-15" }
{ "create": { "_index": "tokobaju_products_v1", "_id": "prod-3" } }
{ "name": "Celana Jogger Pria", "description": "Celana jogger pria bahan fleece tebal, cocok untuk olahraga dan santai", "category": "celana", "brand": "SportMax", "price": 130000, "stock": 200, "tags": ["jogger", "sport", "fleece"], "sizes": ["S", "M", "L", "XL"], "created_at": "2024-02-01" }
{ "create": { "_index": "tokobaju_products_v1", "_id": "prod-4" } }
{ "name": "Dress Batik Modern", "description": "Dress batik motif modern dengan bahan yang ringan dan elegan", "category": "dress", "brand": "BatikNusantara", "price": 250000, "stock": 60, "tags": ["batik", "formal", "elegant"], "sizes": ["S", "M", "L"], "created_at": "2024-02-10" }
{ "create": { "_index": "tokobaju_products_v1", "_id": "prod-5" } }
{ "name": "Hoodie Polos Premium", "description": "Hoodie polos bahan fleece premium, tersedia berbagai warna", "category": "hoodie", "brand": "BasicWear", "price": 195000, "stock": 120, "tags": ["hoodie", "casual", "premium", "oversize"], "sizes": ["M", "L", "XL", "XXL"], "created_at": "2024-03-01" }
```

### 3b. Bulk Insert Customers

```http
POST http://localhost:9200/_bulk
```

```
{ "create": { "_index": "tokobaju_customers_v1", "_id": "cust-1" } }
{ "username": "dzarurizky", "full_name": "Dzaru Rizky Fathan Fortuna", "email": "dzaru@example.com", "gender": "Male", "birth_date": "1998-03-15", "address": { "street": "Jl. Merdeka No. 10", "city": "Surabaya", "province": "Jawa Timur", "zip_code": "60111" }, "banks": [{ "name": "bca", "account_number": "1234567890" }, { "name": "bni", "account_number": "0987654321" }], "labels": { "vip": true, "segment": "fashion-lover" } }
{ "create": { "_index": "tokobaju_customers_v1", "_id": "cust-2" } }
{ "username": "sariindah", "full_name": "Sari Indah Permata", "email": "sari@example.com", "gender": "Female", "birth_date": "2000-07-22", "address": { "street": "Jl. Kenanga No. 5", "city": "Malang", "province": "Jawa Timur", "zip_code": "65141" }, "banks": [{ "name": "mandiri", "account_number": "1122334455" }], "labels": { "vip": false, "segment": "casual-shopper" } }
{ "create": { "_index": "tokobaju_customers_v1", "_id": "cust-3" } }
{ "username": "budiprakoso", "full_name": "Budi Prakoso Santoso", "email": "budi@example.com", "gender": "Male", "birth_date": "1995-11-08", "address": { "street": "Jl. Mawar No. 3", "city": "Bandung", "province": "Jawa Barat", "zip_code": "40111" }, "banks": [{ "name": "bca", "account_number": "5566778899" }, { "name": "bri", "account_number": "9988776655" }], "labels": { "vip": true, "segment": "premium-buyer" } }
```

### 3c. Insert a Single Order (Create API)

```http
POST http://localhost:9200/tokobaju_orders_v1/_create/order-001
```

```json
{
  "order_code": "ORD-2024-001",
  "customer_id": "cust-1",
  "total": 280000,
  "status": "completed",
  "ordered_at": "2024-03-10 14:30:00",
  "items": [
    { "product_id": "prod-1", "name": "Kaos Polos Oversize", "qty": 2, "price": 85000 },
    { "product_id": "prod-5", "name": "Hoodie Polos Premium", "qty": 1, "price": 195000 }
  ]
}
```

> ❓ **Try this**: Run the same request again. What HTTP status code do you get and why?

### 3d. Update a Product's Stock

```http
POST http://localhost:9200/tokobaju_products_v1/_update/prod-1
```

```json
{
  "doc": { "stock": 130 }
}
```

### 3e. Retrieve a Document

```http
GET http://localhost:9200/tokobaju_products_v1/_doc/prod-1
```

Get only the source data (no metadata):

```http
GET http://localhost:9200/tokobaju_products_v1/_source/prod-1
```

Get only selected fields:

```http
GET http://localhost:9200/tokobaju_products_v1/_search?_source_includes=name,price,stock
```

### 3f. Multi Get

```http
POST http://localhost:9200/_mget
```

```json
{
  "docs": [
    { "_index": "tokobaju_products_v1",  "_id": "prod-1" },
    { "_index": "tokobaju_products_v1",  "_id": "prod-3" },
    { "_index": "tokobaju_customers_v1", "_id": "cust-1" }
  ]
}
```

### 3g. Delete a Document

```http
DELETE http://localhost:9200/tokobaju_customers_v1/_doc/cust-999
```

> ❓ **What happens** when you try to delete a document that doesn't exist?

---

## 4. Search & Query DSL

### 4a. Match All (with Pagination & Sorting)

Return all products, sorted by price descending:

```http
POST http://localhost:9200/tokobaju_products_v1/_search
```

```json
{
  "query": { "match_all": {} },
  "size": 10,
  "from": 0,
  "sort": [{ "price": { "order": "desc" } }]
}
```

### 4b. Term Query — Exact Match

Find all products in category `kemeja`:

```json
{
  "query": {
    "term": { "category": "kemeja" }
  }
}
```

### 4c. Terms Query — Multiple Values

Find products in category `kaos` or `hoodie`:

```json
{
  "query": {
    "terms": { "category": ["kaos", "hoodie"] }
  }
}
```

### 4d. Match Query — Full-Text Search

Search products by description keyword `casual`:

```json
{
  "query": {
    "match": { "description": "casual" }
  }
}
```

Search with `AND` operator — must contain both words:

```json
{
  "query": {
    "match": {
      "description": {
        "query": "cotton casual",
        "operator": "AND"
      }
    }
  }
}
```

### 4e. Boolean Query — Combine Conditions

Find products that:
- **Must** be in `kaos` or `hoodie` category
- **Filter** price between 80,000 and 200,000 and stock > 0
- **Should** be from brand `BasicWear` (boosted relevance)

```json
{
  "query": {
    "bool": {
      "must": [
        { "terms": { "category": ["kaos", "hoodie"] } }
      ],
      "filter": [
        { "range": { "price": { "gte": 80000, "lte": 200000 } } },
        { "range": { "stock": { "gt": 0 } } }
      ],
      "should": [
        { "term": { "brand": { "value": "BasicWear", "boost": 2 } } }
      ]
    }
  }
}
```

> 💡 Use `filter` instead of `must` for range conditions — it's faster and cacheable since it doesn't affect the relevance score.

### 4f. Nested Query — Search Inside Banks

Find customers who have a `bca` bank account:

```http
POST http://localhost:9200/tokobaju_customers_v1/_search
```

```json
{
  "query": {
    "nested": {
      "path": "banks",
      "query": {
        "term": { "banks.name": "bca" }
      }
    }
  }
}
```

Find customers with **both** `bca` and `bni` accounts:

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "nested": {
            "path": "banks",
            "query": { "term": { "banks.name": "bca" } }
          }
        },
        {
          "nested": {
            "path": "banks",
            "query": { "term": { "banks.name": "bni" } }
          }
        }
      ]
    }
  }
}
```

### 4g. Nested Query — Search Inside Order Items

Find orders that contain product `prod-1`:

```http
POST http://localhost:9200/tokobaju_orders_v1/_search
```

```json
{
  "query": {
    "nested": {
      "path": "items",
      "query": {
        "term": { "items.product_id": "prod-1" }
      }
    }
  }
}
```

### 4h. Search After (Pagination Beyond 10K)

First page:

```json
{
  "size": 2,
  "query": { "match_all": {} },
  "sort": [{ "price": { "order": "asc" } }]
}
```

Next page — use the last document's `price` as `search_after`:

```json
{
  "size": 2,
  "query": { "match_all": {} },
  "sort": [{ "price": { "order": "asc" } }],
  "search_after": [130000]
}
```

### 4i. Explain API — Understand Score Calculation

```http
POST http://localhost:9200/tokobaju_products_v1/_explain/prod-5
```

```json
{
  "query": {
    "match": { "description": "premium" }
  }
}
```

---

## 5. Alias & Reindex

### 5a. Why We Need This

Suppose you need to add a new field `discount_price` to `tokobaju_products`. Since field types can't be changed, you must create a new index version and migrate data.

### 5b. Create Aliases for All Indexes

```http
POST http://localhost:9200/_aliases
```

```json
{
  "actions": [
    { "add": { "alias": "products",  "index": "tokobaju_products_v1" } },
    { "add": { "alias": "customers", "index": "tokobaju_customers_v1" } },
    { "add": { "alias": "orders",    "index": "tokobaju_orders_v1" } }
  ]
}
```

From now on, use alias names (e.g., `products`) in all your queries.

### 5c. Create New Index Version with Additional Field

```http
PUT http://localhost:9200/tokobaju_products_v2
```

```http
PUT http://localhost:9200/tokobaju_products_v2/_mapping
```

```json
{
  "properties": {
    "name":           { "type": "text", "fields": { "raw": { "type": "keyword" } } },
    "description":    { "type": "text" },
    "category":       { "type": "keyword" },
    "brand":          { "type": "keyword" },
    "price":          { "type": "integer" },
    "discount_price": { "type": "integer" },
    "stock":          { "type": "integer" },
    "tags":           { "type": "keyword" },
    "sizes":          { "type": "keyword" },
    "created_at":     { "type": "date", "format": "yyyy-MM-dd" }
  }
}
```

### 5d. Reindex Data

```http
POST http://localhost:9200/_reindex
```

```json
{
  "source": { "index": "tokobaju_products_v1" },
  "dest":   { "index": "tokobaju_products_v2" }
}
```

### 5e. Switch the Alias

```http
POST http://localhost:9200/_aliases
```

```json
{
  "actions": [
    { "add":    { "alias": "products", "index": "tokobaju_products_v2" } },
    { "remove": { "alias": "products", "index": "tokobaju_products_v1" } }
  ]
}
```

> ✅ All clients using the `products` alias are now automatically on v2 — no code changes needed.

---

## 6. Snapshot & Restore

### 6a. Configure Snapshot Path

Add to `config/elasticsearch.yml` and restart:

```yaml
path.repo: ["snapshots"]
```

### 6b. Create a Snapshot Repository

```http
PUT http://localhost:9200/_snapshot/tokobaju_backup
```

```json
{
  "type": "fs",
  "settings": { "location": "tokobaju_backup" }
}
```

### 6c. Take a Snapshot

```http
PUT http://localhost:9200/_snapshot/tokobaju_backup/snapshot_v1
```

```json
{
  "indices": [],
  "metadata": {
    "taken_by": "Dzaru Rizky Fathan Fortuna",
    "taken_because": "backup before schema migration to v2"
  }
}
```

> 📌 Leaving `indices` as an empty array backs up **all** indexes.

### 6d. Simulate Data Loss & Restore

Delete the products index to simulate a disaster:

```http
DELETE http://localhost:9200/tokobaju_products_v1
```

Close the index before restoring (required):

```http
POST http://localhost:9200/tokobaju_products_v1/_close
```

Restore from snapshot:

```http
POST http://localhost:9200/_snapshot/tokobaju_backup/snapshot_v1/_restore
```

```json
{
  "indices": ["tokobaju_products_v1"]
}
```

Reopen the index:

```http
POST http://localhost:9200/tokobaju_products_v1/_open
```

Verify the data is back:

```http
GET http://localhost:9200/tokobaju_products_v1/_doc/prod-1
```

---

## 7. Monitor with Cat API

Check cluster health:

```http
GET http://localhost:9200/_cat/health?v
```

List all indexes with document count and size:

```http
GET http://localhost:9200/_cat/indices?v
```

List all aliases:

```http
GET http://localhost:9200/_cat/aliases?v
```

Check all nodes in the cluster:

```http
GET http://localhost:9200/_cat/nodes?v
```

Check shard distribution:

```http
GET http://localhost:9200/_cat/shards?v
```

List all snapshots:

```http
GET http://localhost:9200/_cat/snapshots/tokobaju_backup?v
```

Expected output for `/_cat/indices?v`:

```
health status index                  pri rep docs.count store.size
yellow open   tokobaju_products_v1     1   1          5     12.3kb
yellow open   tokobaju_products_v2     1   1          5     13.1kb
yellow open   tokobaju_customers_v1    1   1          3      6.8kb
yellow open   tokobaju_orders_v1       1   1          1      4.2kb
```

> ⚠️ `yellow` status is normal on a single-node setup — replica shards cannot be assigned without another node.

---

## 🏆 Challenge Tasks

Once you've completed the steps above, try these on your own:

1. **Add a new order** for `cust-2` containing 1 `Dress Batik Modern` and 2 `Celana Jogger Pria`. Calculate the correct `total`.

2. **Search for all female customers** from `Jawa Timur` province using a Boolean query with `filter`.

3. **Find all completed orders** where the total is greater than `200,000`.

4. **Search products** that have tag `premium` AND are sorted by `name.raw` alphabetically.

5. **Delete all products** with `stock` equal to `0` using the Delete by Query API.

6. **Add a `rating` field** (type `float`) to the products mapping by creating `tokobaju_products_v3`, reindexing the data, and switching the `products` alias to v3.

7. **Take a new snapshot** called `snapshot_v2` after completing all challenges, then verify it appears in the Cat API.

---

## ✅ Concepts Covered

| Concept | Where Practiced |
|---|---|
| Index creation & deletion | Step 2, 5c |
| Manual Mapping | Step 2a, 2b, 2c |
| Multi Fields | Step 2a — `name.raw` |
| Object Fields | Step 2b — `address` |
| Nested Fields | Step 2b — `banks`, Step 2c — `items` |
| Flattened Fields | Step 2b — `labels` |
| Create API | Step 3c |
| Index API (via Bulk) | Step 3a |
| Get / Get Source API | Step 3e |
| Update API | Step 3d |
| Delete API | Step 3g |
| Bulk API | Step 3a, 3b |
| Multi Get API | Step 3f |
| Match All Query | Step 4a |
| Term / Terms Query | Step 4b, 4c |
| Match Query | Step 4d |
| Boolean Query | Step 4e |
| Nested Query | Step 4f, 4g |
| Boost Score | Step 4e |
| Search After | Step 4h |
| Explain API | Step 4i |
| Alias — Create & Switch | Step 5b, 5e |
| Reindex API | Step 5d |
| Snapshot & Restore | Step 6 |
| Cat API | Step 7 |
