## History

- GFS did not present any new ideas at the time of publish and presentation at the Mass Systems conference. GFS is notable because it used academic systems-building research and applied it to the real world.
- Organization-wide data storage system at scale - how could we build a global storage system that all teams within Google could write to and read from?

## Notes

- Built for sequential reads (not random access) of large files (GB-TB sized), a MapReduce workflow (large dataset processing where missing or corrupted rows doesn't matter)
- Loose on data consistency (will anyone notice if one google search query is missing in a corpus of 20,000?)
- Uses a single leader scheme (the leader is certainly replicated many times)
- Files are chunked and stored on "chunk servers"

## Clear up

- leader maintains append-only log and guarantees append-at-least-once semantics. What are the implications of append-at-least once semantics?
- (?) checksums to verify if the file is an original or is a copy?
- the benefits of leasing
- how concurrent writes to the same chunk are handled

## Notes

- Chunks are 64mb
- Separates responsibility of file lookup and file storage. The leader is responsible for keeping record of file chunk location
- Is a single leader (with replicated backups) beneficial for GFS?
- Single leader responsible for file metadata, kept in-memory
- files are broken into chunks and stored on chunk servers
- responsibility of file location and file storage separate
- clients updating chunks go through the leader, updates are append-at-least-once
