# 📇 Indexing Strategy

This page describes the indexing design used in the **Couchbase Eventing Shadow Collection Pattern** for the `quantic` bucket.

The goal of this indexing strategy is to support:

* High-volume operational event ingestion
* Real-time event filtering and aggregation
* Multi-tenant analytics queries
* Time-series reporting workloads
* Eventing-driven shadow replication

---

# 🧠 Design Principles

## 1. Query-Driven Indexing

Indexes are designed based on actual query patterns rather than schema structure.

This ensures:

* minimal index fanout
* efficient filtering paths
* predictable query performance

---

## 2. Operational vs Analytical Separation

| Layer           | Purpose                                   |
| --------------- | ----------------------------------------- |
| `events`        | Operational ingestion + real-time queries |
| `events_shadow` | Durable analytics dataset (TTL removed)   |
| `eventing_meta` | Eventing counters + metadata              |

Operational workloads are optimized for write throughput, while analytics workloads are offloaded to shadow collections.

---

## 3. TTL-Aware Design

Operational data contains TTL (Time-To-Live) due to mobile ingestion patterns.

To prevent analytical data loss:

* TTL remains only in operational collection
* Shadow collection removes TTL during Eventing transformation
* Analytics reads from TTL-free dataset

---

# 📊 Primary Indexes

Primary indexes are used for development, debugging, and administrative access.

## `eventing_meta`

```sql
CREATE PRIMARY INDEX ON `quantic`.`core`.`eventing_meta`;
```

**Purpose:**

* Eventing counter inspection
* Debugging metadata state
* Administrative queries

---

## `events_shadow`

```sql
CREATE PRIMARY INDEX ON `quantic`.`core`.`events_shadow`;
```

**Purpose:**

* Validate shadow replication
* Analytics dataset inspection
* Reporting verification

---

## `events`

```sql
CREATE PRIMARY INDEX ON `quantic`.`core`.`events`;
```

**Purpose:**

* Operational debugging
* Development queries
* Ad hoc inspection

---

# ⚡ Secondary Indexes (Operational Events)

These indexes support high-performance filtering and analytics on the `events` collection.

---

## 👤 User + Session Tracking

```sql
CREATE INDEX `idx_core_events_user_session`
ON `quantic`.`core`.`events`(`userId`, `sessionId`);
```

**Use cases:**

* User journey reconstruction
* Session-based analysis
* Behavioral tracking

---

## 🏢 Multi-Tenant Filtering

```sql
CREATE INDEX `idx_core_events_tenant_eventType`
ON `quantic`.`core`.`events`(`tenant`, `eventType`);
```

**Use cases:**

* Tenant isolation
* Event segmentation
* Per-tenant analytics

---

## 📈 Event Type + Time Series

```sql
CREATE INDEX `idx_core_events_eventtype_ts`
ON `quantic`.`core`.`events`(`eventType`, `timestamp`);
```

**Use cases:**

* Event timelines
* Trend analysis
* Time-based aggregation

---

## ⏱️ Tenant Time-Series Queries

```sql
CREATE INDEX `idx_core_events_tenant_ts`
ON `quantic`.`core`.`events`(`tenant`, `timestamp`);
```

**Use cases:**

* Tenant activity windows
* Time-based dashboards
* SLA monitoring

---

## 🌍 Region + Device Analytics

```sql
CREATE INDEX `idx_core_events_region_device`
ON `quantic`.`core`.`events`(`region`, `device`);
```

**Use cases:**

* Geographic analysis
* Device segmentation
* Mobile usage insights

---

## 🚀 High-Cardinality Composite Index

```sql
CREATE INDEX `idx_core_events_tenant_eventType_ts`
ON `quantic`.`core`.`events`(`tenant`, `eventType`, `timestamp`);
```

**Use cases:**

* High-selectivity analytics queries
* Dashboard acceleration
* Filtered time-series reporting
* KPI computation

---

# 🪞 Shadow Collection Indexing

The `events_shadow` collection serves as the **analytics-ready dataset**.

Key characteristics:

* TTL removed during Eventing transformation
* stable historical retention
* reduced index churn
* optimized for analytics workloads

This ensures Analytics nodes receive a **durable dataset independent of operational expiration policies**.

---

# 📌 Index Usage Summary

| Index                            | Purpose                 |
| -------------------------------- | ----------------------- |
| `userId + sessionId`             | Session reconstruction  |
| `tenant + eventType`             | Multi-tenant filtering  |
| `eventType + timestamp`          | Event timelines         |
| `tenant + timestamp`             | Tenant analytics        |
| `region + device`                | Geo/device analytics    |
| `tenant + eventType + timestamp` | Full analytical slicing |

---

# 🧠 Why This Indexing Strategy Works

This design is optimized for:

* real-time event ingestion workloads
* multi-dimensional query patterns
* time-series analytics
* tenant isolation
* high-throughput mobile data ingestion

It aligns closely with Couchbase’s distributed indexing model in entity["company","Couchbase",""], ensuring scalability without sacrificing query performance.

---

# 🚀 Summary

The indexing strategy balances:

* operational performance
* analytics flexibility
* TTL-aware ingestion constraints
* shadow-based durability patterns

This ensures the system can support both:

* real-time event processing
* long-term analytical reporting
