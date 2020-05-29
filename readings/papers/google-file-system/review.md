# Review

## Motivations

- Google wanted to enable web crawling and indexing of entire internet.
- Also had large logs of web searches etc. to later analyze
- Wanted to a global storage system so all orgs. can play in the sandbox and work with the same data

### Remember

- Want to shard (harness the power of 100s of commodity machines)
- Prioritize throughput
- Many concurrent client requests, don't want to slow

### Abbreviation Key

- `P` = primary
- `S` = secondary
- `L` = leader
- `C` = client
- `CH` = chunk server
- `V` = volatile (in-memory)
- `NV` = non-volatile (written to disk, persistent)
- `V#` = version number
- `->` = mapping

## Leader

The leader contains the following:

Note: some information is kept in-memory and other is kept both in-memory and written to disk.

- list of `CH`s (`volatile`)
- `V#`s (`non-volatile`)
  - the `V#` is used to determine which chunk server is most-recent. Writing the `V#` to disk enables the `L` to determine which `CH` are most-recent and up-to-date when it recovers from failure.
- Which `CH` are `P` (`volatile`)
- `P` who contains the current lease (`non-volatile`)
  - the lease is GFS's solution to split-brain problem. Suppose a network partition where `L` can't communicate w/`CH`s (no heartbeat messsages ack.). W/out leasing (time-expiry) the `L` will assign a new `CH` as a `P` - all the while, the existing `P` is still functioning and serving client requests! Solution: leasing guarantees safe assignment of a new `P` because it is an understood contract of both `L` and primary `CH`.
- Log, append-only (`non-volatile`)

## Chunk Server

- 64mb files on a linux server
- Sequential appends to an offset (space in sequence)

## Reading

- `C` requests location of a filename and byte range to read
- calls `L`, `L` and GFS library conspire to return a list of `CH` that contain the file byte range to `C`
- internally, `L`:
  - filename `->` array of bytes
  - array of bytes `->` table containing `CH`
  - returns list of `CH` that contain filename
- `C` performs reads using the list of `CH`
- `CH` returns data according to byte range client requests

## Writing, specifically Record-Append

Consider the scenario: The leader recovers from failure

- leader replays its state from last checkpoint up to the most-recent record in its append log
- leader sends a heartbeat to `CH` requesting their version numbers
- diffs the `V#` with those that exist on disk

- `L` gathers set of `P` and replicas (secondaries, `S`)
- returns list to client (`C`)
- `C` writes to `P`
- `P` appends write to offset and distributes write to `S`
- `S` may, or may not write the offset.
- If all `S` write - they reply with an ack. to the `P` and `P` replies `"yes"` to `C`
- If not all `S` ack. then `"no"` is returned
- Client retries if `"no` (failure). Eventually all data would be written through repeated retries from client

* Good properties for successful writes
* Not so good properties for unsuccessful writes
  - however, client is fully aware of partial success
  - leaves responsibility of client to enforce consistency and synchronization (that's if the client application needs this!)
    - client application might enforce that only one client can write to the primary at one time.

## Review

- Weaker consistency
  - Justificiation: throughput, performance needs, scale of concurrent requests

## Limitations

- Single leader could server hundreds, but not thousands of requests. Scale of req. became problem. Most info is kept in-memory, can increase RAM, but only so much.

## Considerations

What would we need to consider if we wanted strong(er) consistency from a system like GFS?

- Two-phase commit (2PC) for record-appends between `P` and `S`s
- Leasing for secondaries (protocol for when it's okay to server requests)
- Duplicate request detection - `L` enforces duplicates requests are not served
- Leader election for new `P` if it fails, and re-synchronization so the new primary is up-to-date with all secondaries
- Some mechanism for removing stale, faulty secondaries so stale and skewed data cannot be served or written

# Pasta

## History

- GFS did not present any new ideas at the time of publish and presentation at the SOSP conference (Symposium on Operating Systems). GFS is notable because it used academic systems-building research and applied it to the real world.
- Organization-wide data storage system at scale - how could we build a global storage system that all teams within Google could write to and read from?

## Notes

- Built for sequential reads (not random access) of large files (GB-TB sized), a MapReduce workflow (large dataset processing where missing or corrupted rows doesn't matter)
- Loose on data consistency (will anyone notice if one google search query is missing in a corpus of 20,000?)

Leader

- reads and writes never go through the leader, the leader simply tells clients where chunks are located
  - client caches filename and chunk index so subsequent reads and writes do not go through leader, until cached info expires.
  - reduces leader, client interaction

## Clear up

Q: Why use a single leader (with replicated backups)?
A:

Q: What allows for the metadata to be kept in-memory? Why?
A: GFS keeps chunk location information in-memory because it removes the problem of keeping the leader and chunkservers in sync. Chunkservers fail, are join, and leave all the time. It's unnecessary to maintain a consistent view of chunks on leader.
Leader polls the chunkservers for chunk information on startup and periodically.

So the leader maintains the metadata information as a log? or as a table? or both? which is in-memory?

Q: leader maintains append-only metadata log and guarantees append-at-least-once semantics.

- Why have a log? What are the implications of append-at-least once semantics?
  A:

* Proof of record (ordering) for concurrent writes.
* Metadata log must be persisted and replicated
* If a leader fails and needs to be brought back up, the leader can replay the operation log to recover filesystem state.
*

Q: checksums to verify if the file is an original or is a copy?
A:

Q: the benefits of leasing
A:

Q: how does GFS handle concurrent writes?

- how concurrent writes to the same chunk are handled
  A:

Q: how does GFS handle split brain (multiple leaders?)
A:

## Notes

- Uses a single leader scheme (the leader is replicated for backup)
- Files are 64 mb chunked and stored on "chunk servers"
  - reduces size of metadata that needs to be stored on master
- separates responsibility of locating and storing files (master and chunk servers)
- clients updating chunks go through the leader, updates are append-at-least-once
