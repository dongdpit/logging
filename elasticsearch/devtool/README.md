# Elasticsearch Dev Tools

> Collection of Elasticsearch APIs and Kibana Dev Tools commands for daily administration, troubleshooting, monitoring, and log management.

---

## 📑 Table of Contents

* [Cluster](#-cluster)
* [Nodes](#-nodes)
* [Indices](#-indices)
* [Mappings](#-mappings)
* [Index Templates](#-index-templates)
* [Aliases](#-aliases)
* [ILM](#-ilm)
* [Rollover](#-rollover)
* [Search](#-search)
* [Documents](#-documents)
* [Reindex](#-reindex)
* [Ingest Pipeline](#-ingest-pipeline)
* [Shards](#-shards)
* [Disk & Allocation](#-disk--allocation)
* [Recovery](#-recovery)
* [Tasks](#-tasks)
* [Snapshots](#-snapshots)
* [Troubleshooting](#-troubleshooting)
* [Quick Reference](#-quick-reference)

---

# Cluster

```http
GET _cluster/health
GET _cluster/settings?include_defaults=true
GET _cluster/state
```

---

# Nodes

```http
GET _cat/nodes?v
GET _nodes/stats
GET _nodes/stats/jvm
GET _nodes/stats/os
GET _nodes/stats/fs
```

---

# Indices

## List Indices

```http
GET /_cat/indices?v
GET /_cat/indices?v&s=store.size:desc
GET /_cat/indices?v&h=store.size,uuid&s=store.size:desc
```

## Create Index
```http
PUT %3Clog-product-iis17x-%7Bnow%2Fd%7D-000001%3E
{ 
  "aliases": { 
    "logiis17x-alias": { 
      "is_write_index": true 
    } 
  } 
}
```

---


# Index Templates

Index Templates define default configuration for newly created indices.

A template can contain:

* Settings
* Mappings
* Aliases
* ILM configuration
* Shard configuration
* Replica configuration

## List/Get Templates

```http
GET _index_template
GET _index_template/logiis17x-template
```

## Create Template

```http
PUT _index_template/logiis17x_template
{
  "index_patterns": [
    "log-product-iis17x-*"
  ],
  "priority": 500,
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1
      "index.lifecycle.name": "iis-15days",
      "index.lifecycle.rollover_alias": "logiis17x-alias"
    },
    "mappings": {
      "properties": {
        "log_timestamp": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss||strict_date_optional_time||epoch_millis"
        }
      }
    }
  }
}
```

## Simulate Template

Use this API to determine which templates will be applied when an index is created.

```http
POST /_index_template/_simulate_index/log-product-iis17x-test-000001
```

---

# Aliases

Aliases provide a logical name for one or more indices.

Example:

```text
                 logiis17x-alias
                        │
                        ▼
             log-product-iis17x-000001
```

## List/Get Aliases

```http
GET _cat/aliases?v
GET /logiis17x-alias
```

## Add Alias

```http
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "log-product-iis17x-000001",
        "alias": "logiis17x-alias"
      }
    }
  ]
}
```

---

# ILM

## List/Get ILM Policies

```http
GET _ilm/policy
GET _ilm/policy/iis-15days

```

## Check Index ILM Status

```http
GET log-product-iis17x-000001/_ilm/explain
```

Useful fields:

```text
phase
action
step
step_info
failed_step
```

Use `_ilm/explain` when troubleshooting:

* Index not rolling over
* Index not being deleted
* ILM stuck
* Wrong rollover alias
* Incorrect policy configuration

---

# 🔄 Rollover

Typical architecture:

```text
                 logiis17x-alias
                        │
                        ▼
             log-product-iis17x-000001
```

After rollover:

```text
                 logiis17x-alias
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
log-product-iis17x-000001   log-product-iis17x-000002
```

## Check Rollover Alias

```http
GET _cat/aliases/logiis17x-alias?v
```

## Manual Rollover

```http
POST logiis17x-alias/_rollover
```

## Rollover with Conditions

```http
POST logiis17x-alias/_rollover
{
  "conditions": {
    "max_age": "1d",
    "max_primary_shard_size": "50gb"
  }
}
```

---

# 🔎 Search

## Match All

```http
GET log-product-iis17x-*/_search
{
  "query": {
    "match_all": {}
  }
}
```

## Match

```http
GET log-product-iis17x-*/_search
{
  "query": {
    "match": {
      "message": "error"
    }
  }
}
```

## Term

Use `term` for exact matching on `keyword` fields.

```http
GET log-product-iis17x-*/_search
{
  "query": {
    "term": {
      "status": "500"
    }
  }
}
```

## Search by Time

```http
GET log-product-iis17x-*/_search
{
  "query": {
    "range": {
      "log_timestamp": {
        "gte": "now-1h",
        "lte": "now"
      }
    }
  }
}
```

---

# 📄 Documents

## Count Documents

```http
GET log-product-iis17x-*/_count
```

## Insert Document

```http
POST log-product-iis17x-000001/_doc
{
  "log_timestamp": "2026-08-26 09:00:00",
  "message": "test"
}
```

## Get Document

```http
GET log-product-iis17x-000001/_doc/<document_id>
```

## Update Document

```http
POST log-product-iis17x-000001/_update/<document_id>
{
  "doc": {
    "message": "updated"
  }
}
```

## Delete Document

```http
DELETE log-product-iis17x-000001/_doc/<document_id>
```

---

# 🔀 Reindex

Use `_reindex` when migrating data to a new index.

Typical use case:

```text
Old Index
    │
    │ Wrong Mapping
    ▼
New Index
    │
    │ Correct Mapping
    ▼
Reindex
```

## Basic Reindex

```http
POST _reindex
{
  "source": {
    "index": "log-product-iis17x-old"
  },
  "dest": {
    "index": "log-product-iis17x-new"
  }
}
```

## Reindex with Query

```http
POST _reindex
{
  "source": {
    "index": "log-product-iis17x-old",
    "query": {
      "range": {
        "log_timestamp": {
          "gte": "now-15d"
        }
      }
    }
  },
  "dest": {
    "index": "log-product-iis17x-new"
  }
}
```

---

# 🧪 Ingest Pipeline

## List Pipelines

```http
GET _ingest/pipeline
```

## Get Pipeline

```http
GET _ingest/pipeline/iis-client-geoip
```

## Simulate Pipeline

```http
POST _ingest/pipeline/iis-client-geoip/_simulate
{
  "docs": [
    {
      "_source": {
        "client": {
          "ip": "8.8.8.8"
        }
      }
    }
  ]
}
```

Useful for troubleshooting:

* GeoIP
* Date processor
* Grok
* Rename
* Set
* Remove
* Script

---

# 🧩 Shards

## List Shards

```http
GET _cat/shards?v
```

## Check Unassigned Shards

```http
GET _cat/shards?v&h=index,shard,prirep,state,unassigned.reason
```

## Allocation Explain

Use this when a shard is `UNASSIGNED`.

```http
GET _cluster/allocation/explain
```

Common causes:

* Disk watermark
* Allocation rules
* Node eligibility
* Replica allocation
* Shard limit
* Awareness
* Data tier configuration

---

# 💾 Disk & Allocation

## Node Disk Allocation

```http
GET _cat/allocation?v
```

## Filesystem Statistics

```http
GET _nodes/stats/fs
```

## Largest Indices

```http
GET _cat/indices?v&s=store.size:desc
```

---

# 🔁 Recovery

## Recovery Status

```http
GET _cat/recovery?v
```

## Index Recovery

```http
GET log-product-iis17x-000001/_recovery
```

---

# ⚙️ Tasks

## List Tasks

```http
GET _tasks
```

## Detailed Tasks

```http
GET _tasks?detailed=true
```

## Reindex Tasks

```http
GET _tasks?actions=*reindex
```

Useful when monitoring:

* `_reindex`
* `_delete_by_query`
* Snapshot
* Recovery
* Forcemerge

---

# 📸 Snapshots

## List Repositories

```http
GET _snapshot
```

## List Snapshots

```http
GET _snapshot/<repository>/_all
```

## Snapshot Status

```http
GET _snapshot/<repository>/<snapshot>/_status
```

---

# 🛠 Troubleshooting

## General Troubleshooting Flow

```text
                    Elasticsearch Issue
                            │
                            ▼
                   GET _cluster/health
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
        Node               Disk              Shard
          │                 │                 │
    _cat/nodes       _cat/allocation    _cat/shards
          │                 │                 │
          ▼                 ▼                 ▼
    _nodes/stats       Disk Usage       allocation/explain
                            │
                            ▼
                          Index
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Mapping         Template         ILM
             │              │              │
         _mapping        simulate       _ilm/explain
                            │
                            ▼
                          Alias
                            │
                       _cat/aliases
```

---

# 🌐 IIS Log Troubleshooting

For IIS logging indices, use the following workflow:

```text
1. Check Index
       │
       ▼
2. Check Mapping
       │
       ▼
3. Check Index Template
       │
       ▼
4. Simulate Template
       │
       ▼
5. Check Alias
       │
       ▼
6. Check ILM
       │
       ▼
7. Check Rollover
       │
       ▼
8. Check Data
```

### Check Index

```http
GET _cat/indices/log-product-iis17x-*?v
```

### Check Mapping

```http
GET log-product-iis17x-*/_mapping
```

### Check Template

```http
GET _index_template/logiis17x-template
```

### Simulate Template

```http
POST /_index_template/_simulate_index/log-product-iis17x-test-000001
```

### Check Alias

```http
GET _cat/aliases/logiis17x-alias?v
```

### Check ILM

```http
GET log-product-iis17x-000001/_ilm/explain
```

### Test Search

```http
GET logiis17x-alias/_search
{
  "query": {
    "range": {
      "log_timestamp": {
        "gte": "now-1h"
      }
    }
  }
}
```

---

# ⚡ Quick Reference

| Purpose             | API                                            |
| ------------------- | ---------------------------------------------- |
| Cluster Health      | `GET _cluster/health`                          |
| Nodes               | `GET _cat/nodes?v`                             |
| Indices             | `GET _cat/indices?v`                           |
| Shards              | `GET _cat/shards?v`                            |
| Allocation          | `GET _cat/allocation?v`                        |
| Aliases             | `GET _cat/aliases?v`                           |
| Recovery            | `GET _cat/recovery?v`                          |
| Templates           | `GET _index_template`                          |
| Template Simulation | `POST _index_template/_simulate_index/<index>` |
| Mapping             | `GET <index>/_mapping`                         |
| Settings            | `GET <index>/_settings`                        |
| ILM Policies        | `GET _ilm/policy`                              |
| ILM Status          | `GET <index>/_ilm/explain`                     |
| Tasks               | `GET _tasks`                                   |
| Pipelines           | `GET _ingest/pipeline`                         |
| Snapshots           | `GET _snapshot`                                |

---

# ⚠️ Production Warning

The following operations can modify or delete production data:

```text
DELETE
_delete_by_query
_reindex
_rollover
PUT _index_template
PUT _cluster/settings
```

Before making changes, check:

```http
GET _cluster/health
```

After making changes, verify:

```http
GET _cluster/health
GET _cat/indices?v
GET _cat/shards?v
```

> **Always verify the target index before executing destructive operations.**
