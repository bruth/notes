## Architecture

- `mongos` - shard router
    - application talks to mongos, looks like a normal database
    - completely stateless
    - convention to run them on the app server (extension of the driver)
- config servers
    - run 3 of them.. 1 in dev is ok
    - changes use a 2 phase commit
    - if one goes down, read-only


## Sharding

- range-based partition on keys

```
// shard on email address
db.runCommand({ shardcollection: "test.users", key: {email: 1}});
```

### Chunks

- single chunk
