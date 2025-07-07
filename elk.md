### 1. **What is the ELK Stack?**

The **ELK Stack** is a powerful set of open-source tools used for **searching, analyzing, and visualizing log data in real time**.

---

### 🔤 ELK stands for:

| Component         | Purpose                                                                |
| ----------------- | ---------------------------------------------------------------------- |
| **Elasticsearch** | A distributed search and analytics engine — stores and indexes logs.   |
| **Logstash**      | A data processing pipeline — collects, parses, and transforms logs.    |
| **Kibana**        | A visualization UI — lets you explore and monitor logs via dashboards. |

---

### 🔄 How It Works (Basic Flow)

```
Application Logs → Logstash → Elasticsearch → Kibana
```

* **Logstash** ingests and processes logs (e.g., from files, syslog, Beats, Kafka).
* **Elasticsearch** stores and indexes the structured data.
* **Kibana** lets you search logs, create dashboards, and set up alerts.

---

### 2. Solutions to Reduce Data Loss in ELK Stack

#### 1. **Use Filebeat with Spooling and Retry**

* **Filebeat** buffers logs on disk before sending them.
* If Logstash or Elasticsearch is down, Filebeat will **retry** later.

✅ Filebeat ensures logs aren't dropped if the destination is temporarily unreachable.

---

#### 2. **Enable Persistent Queues in Logstash**

* By default, Logstash uses an **in-memory queue** (data loss risk if it crashes).
* Use **persistent queues** to write events to disk before processing:

✅ Ensures data survives Logstash crashes or restarts.

---

### 3. **Buffer with Kafka Between Components**

* Insert **Kafka** between Filebeat and Logstash or Elasticsearch.
* Kafka acts as a durable message broker — logs are stored until consumers are ready.

✅ Decouples ingestion from processing, absorbs spikes, and provides guaranteed delivery.

---

### 4. **Use Elasticsearch Bulk Indexing with Retry**

* If Logstash sends directly to Elasticsearch, enable **bulk retry logic**.
* In case of Elasticsearch overload or crash:

✅ Reduces the chance of dropped logs due to transient ES errors.

---

### 5. **Set Up Monitoring and Alerts**

* Use **X-Pack Monitoring** or open-source equivalents to alert on:

  * Logstash queue growth
  * Elasticsearch disk space
  * Filebeat output errors

✅ Early alerts help catch problems before data is lost.

---

### 6. **Use Durable Storage for Elasticsearch**

* Ensure **high-performance, persistent volumes** for Elasticsearch nodes.
* Set proper replica counts:

✅ Protects data from disk failure or node loss.

---

###  3. **Why Is Elasticsearch Fast?**

---

#### 1. **Inverted Indexes** (like a book index)

* Instead of scanning every document, Elasticsearch builds an **inverted index**, mapping each word → list of documents it appears in.
* This makes searching **lightning fast**, even on millions of records.

📘 Example:
Searching for `"error"` directly finds matching log entries without scanning all documents.

---

#### 2. **Distributed & Sharded Architecture**

* Data is **split across shards**, and shards are spread across multiple nodes.
* Queries run **in parallel** on multiple nodes and return combined results.

🔁 Result: Scalability + parallelism = speed.

---

#### 3. **Lucene Under the Hood**

* Elasticsearch is built on top of **Apache Lucene**, a high-performance search library written in Java.
* Lucene handles indexing, scoring, and fast full-text searching.

---

#### 4. **Real-Time Indexing (Near Real Time)**

* Documents become searchable within **1 second** of indexing (default refresh interval).
* Ideal for log and metrics systems where fresh data matters.

---

#### 5. **Memory-Efficient Caching**

* Caches:

  * Filter results (bitsets)
  * Field data
  * Frequent query terms
* Uses **doc values** for fast field-based sorting and aggregations.

---

#### 6. **Optimized Query DSL & Execution**

* Elasticsearch has a powerful and optimized **Query DSL** (Domain Specific Language).
* Filters are **cached**, and queries are **short-circuited** when results are found.

---

### 4. **What Is a Shard?**

A **shard** is a **horizontal partition of your data** — a smaller piece of an Elasticsearch index.

* Each index is **split into shards** to scale horizontally.
* Each shard is a **Lucene index** — it stores and indexes documents.
* A shard runs as an **independent search and index engine**.

---

### 5. **What Is a Replica?**

A **replica** is a **copy of a shard**.

* Improves **high availability** (if a node fails, the replica takes over).
* Allows **parallel searches** to improve performance (load-balanced queries).
* You can have **zero or more replicas** per primary shard.

---

#### 🔧 How It Works Together

| Term              | Description                                 |
| ----------------- | ------------------------------------------- |
| **Primary Shard** | The original, writeable shard.              |
| **Replica Shard** | A read-only copy of the primary shard.      |
| **Total Shards**  | `primary_shards × (1 + number_of_replicas)` |

Elasticsearch auto-assigns shards to nodes — no manual placement needed.

---

### 6. **How Does ELK Know Where Shards Are?**

Elasticsearch uses a **central cluster state**, maintained by the **master node**, to track:

* Which indices exist
* How many primary and replica shards they have
* **Which node each shard is allocated to**
* The health and status of each node and shard

---

#### 🔄 Here's How It Works:

1. **Master Node Maintains Cluster State**

   * It holds the **metadata** for all shard allocations.
   * When an index is created or updated, the master updates the cluster state.

2. **Cluster State Is Broadcast**

   * This state is shared with all nodes in the cluster.
   * Each node knows which shards are where — both locally and remotely.

3. **Search Request Flow**

   * When a search comes in:

     1. It hits a **coordinating node** (can be any node).
     2. That node consults the **cluster state** to know:

        * Which shards are involved in the query
        * Which **nodes** have those shards (primary or replica)
     3. It sends parallel requests to the right nodes, gathers results, and merges them.

---

### 6. **Does Elasticsearch do rebalancing?**

#### 🔄 What Is Rebalancing in Elasticsearch?

**Rebalancing** means moving shards between nodes to:

* Avoid overloading certain nodes
* Evenly distribute disk usage and search load
* Maintain high availability after a node joins or leaves

---

## ⚙️ When Does Elasticsearch Rebalance?

Elasticsearch will **automatically rebalance** when:

| Scenario                                | Result                                      |
| --------------------------------------- | ------------------------------------------- |
| A new node is added                     | Some shards are moved to the new node       |
| A node fails                            | Shards are reassigned to other nodes        |
| Disk usage is imbalanced                | Shards may move off “heavy” nodes           |
| Cluster settings (like replicas) change | Elasticsearch tries to spread shards evenly |

---

## 📁 How It Works

* The **master node** constantly evaluates the **cluster state** and shard placement.
* Uses a **Shard Allocation Decider** to decide:

  * Where to place shards
  * When to move them
* Shards are moved **one at a time** to avoid disrupting performance.

---

