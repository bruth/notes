## Consistency

- specified at the query level
    - pass `slaveok` flag
- eventual consistency
    - great for vast majority of use cases where it doesn't make a difference whether the changes are visible
- strong consistency
    - users who should see immediate updates can use read directly from the the node they just wrote to

## Durability

- "fire and forget"
    - async write, master responds right away
        - not necessarily written to the database when it responds
    - use `getLastError` command
        - applied "safe mode" in most drivers
- all masters will _rollback_ their transactions to the last common point across all master nodes and replay the lastest ones
    - any transactions that are rolled back will be sent to a rollback directory as BSON files

### `getLastError`

See if any errors happened, pass any number of flags for various behavior.

- wait until it is written to disk
    - for journaling `{j: true }` (100 ms)
    - no journaling `{fsync: true}` (1 min)
- wait until the write has been replicated to at least N nodes (still in memory)
    - replicate to 2 nodes - `{w: 2}`
    - replicate to N/2+1 nodes in replica set - `{w: "majority"}`
    - sometimes more appropriate than waiting to write to disk
        - still survives failure without needing to write to disk


## Replica Sets

### Tagging

- associate a tag with a replica set member
- set `getLastErrorModes` in configuration `settings` object
    - enables grouping members by tag for a given error mode


```json
{
    "_id": "some replica set",
    "members": [
        {"_id": 1, "host": "A", "tags": {"dc": "ny"}},
        {"_id": 2, "host": "B", "tags": {"dc": "ny"}},
        {"_id": 3, "host": "C", "tags": {"dc": "sf"}},
        {"_id": 4, "host": "D", "tags": {"dc": "sf"}},
        {"_id": 5, "host": "E", "tags": {"dc": "cloud"}},
    ],
    "settings": {
        "getLastErrorModes": {
            "veryImportant": {"dc": 3},
            "kindOfImportant": {"dc": 2},
        }
    }
}
```

_Reference: http://www.10gen.com/presentations/mongosv-2011/a-mongodb-replication-primer-replica-sets-in-practice_


### Priorities

- define a priority per node between 1 and 1000
- failover to highest priority
    - useful if one server is _beefier_ than the others
- use priority 0 to never become primary
- if a higher priority node _catches up_ it will win the primary election


### Slave Delay

- define an arbitrary amount of time a slave lags behind the master
- acts as a _curated_ node
    - descructive actions that happen to the primary can be removed or fixed before the slave replicates
