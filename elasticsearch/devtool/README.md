# Elasticsearch Dev Tools

> Collection of Elasticsearch APIs and Kibana Dev Tools commands for daily administration, troubleshooting, monitoring, and log management.

---

## 📑 Table of Contents

* [Cluster](#-cluster)
* [Nodes](#-nodes)
* [Indices](#-indices)
* [Index Templates](#-index-templates)
* [Aliases](#-aliases)
* [ILM](#-ilm)
* [Rollover](#-rollover)
* [Reindex](#-reindex)
* [Troubleshooting](#-troubleshooting)

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
GET _cat/allocation?v
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
        "alias": "logiis17x-alias",
        "is_write_index": true
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

# Rollover

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

# Reindex

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
