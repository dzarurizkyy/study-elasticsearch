# Study Elasticsearch ⚡

A comprehensive Elasticsearch reference guide covering installation, data modeling, index and mapping design, CRUD, Query DSL, aliases and reindexing, and snapshot & restore.

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

3. **Run the Elasticsearch Server**:

   ```bash
   # Start Elasticsearch
   ./bin/elasticsearch

   # Verify the server is running
   curl http://localhost:9200
   ```

   > **Windows**: Use `bin\elasticsearch.bat` instead of `bin/elasticsearch`
   >
   > **Stop**: Press `Ctrl + C`

4. **Use an HTTP Client** (choose one):

   `GUI Client`
   - Download Postman at `https://www.postman.com/` or Insomnia at `https://insomnia.rest/`
   - Default connection: `http://localhost:9200`

   `CLI Client`
   - Elasticsearch speaks plain HTTP, so `curl` is enough
   - Run: `curl http://localhost:9200/_cat/indices?v`

## List of Material 📚

- ⚡ **[Elasticsearch Basics](001-elasticsearch-basics.md)**

  Hands-on REST API guide covering data modeling, index and mapping design, the full document lifecycle (CRUD & bulk writes), Query DSL, deep pagination, aliases, and snapshot/restore.

  Filter for exact terms, boost relevance, then sort and page the results:

  ```json
  {
    "query": {
      "bool": {
        "filter": [
          { "term": { "gender": "Female" } }
        ],
        "must": [
          { "match": { "first_name": "Dzaru" } }
        ],
        "should": [
          { "term": { "banks.name": { "value": "bca", "boost": 2 } } }
        ],
        "minimum_should_match": 1
      }
    },
    "size": 10,
    "sort": [{ "username": { "order": "asc" } }]
  }
  ```

  Send requests over HTTP:

  ```bash
  # Define the mapping before the first document
  curl -X PUT http://localhost:9200/customers_v2/_mapping

  # Search an index
  curl -X POST http://localhost:9200/customers_v2/_search

  # Swap an alias atomically after reindexing
  curl -X POST http://localhost:9200/_aliases

  # Back up and restore an index
  curl -X PUT http://localhost:9200/_snapshot/first_backup/snapshot1
  curl -X POST http://localhost:9200/_snapshot/first_backup/snapshot1/_restore
  ```

## 📍 References

- [Udemy](https://www.udemy.com/course/belajar-elasticsearch)

## 👨‍💻 Contributors

- [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)
