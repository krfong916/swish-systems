## Overview

- Raft as a library for handling application state consistency
- Why use a replicated log?
- Raft properties: odd-numbered clusters and majority overlap for quorum
- Fault Tolerance and recovery
- Log Reconciliation

The whole thing about replication is to address two questions:

1. **How do we avoid split brain?** We need to be careful of how we choose a new primary
2. **How do we build an automatic failover system that works correctly, even in the presence of flaky networks?**

## System Model

1. client sends request
2. Server (leader) receives request
3. Leader sends an AppendEntries RPC to servers in cluster (followers)
4. A majority of servers reply back with acknowledgement of replicating the entry in their log
5. The leader allows the application to execute/update the application state
6. A response from the leader is sent back to the client
