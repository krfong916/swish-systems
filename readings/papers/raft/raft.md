Extremely ambigious problem that I don't know how to do
What does kubernetes do?
What environment do we run golang run in?
How do we compile golang?
What is the network protocol that servers will use to talk?
How do we implement the network protocol with golang?

I assume there will be servers involved in the following configuration:

```
                        Server 1
                        Server 2
                        .
                        .
                        .
client -> webserver ->
                        Server n-1
```

1. A client writes a value to a webserver.
2. The webserver does some computation and forwards that request to servers 1..n-1.
3. We send values to those servers continually, asynchronously.
4. At some point in time we introduce a network partition meaning we're able to talk to some servers, and others we cannot
5. How do we detect a node is separated from the cluster? - gossip protocol?
6. The question is - then - how do new nodes get reintroduced in the cluster?
7. How do we discover these new nodes? gossip protocol?
8. How do we reach consensus about the current value and how does that get propagated to the new nodes? raft?

Run Go programs with goroutines inside Pods on Kubernetes minikube and make them discover each other using gossip protocol and let them reach a consensus on a variable change using raft / paxos.

Delete some pods and make the network undergo a partition and make a variable change and watch how they discover after the Pods come back up and they reach consensus again.

You'll learn the following with this exercise:

kubernetes
kubernetes networking
kubernetes operations
Gossip protocol
Raft / paxos implementation
golang
goroutines usage
Docker to containerize the go program
Raft is a consensus algorithm
differs from paxos in its:

- leader election
- log replication
- safety
- reduces number of states to be considered
- cluster membership uses overlapping majority to guarantee safety

## How to read a paper

1. Title
2. Keywords
3. Abstract
4. Conclusion
5. Read tables and figures including captions - this is what was done in the work
6. Read the introduction

- background needed and why study was done

7. Read the results and discussion

- heart of the paper

8. Read the experimental

- how they did the work, only get here if you're really interested and need to understand exactly what was done to better understand the meaning of the data and its interpretation

# Raft

## Overview

- strong leader:
  - log entries only flow from leader to other servers. this simplifies management of replicated log
- leader election:
  - uses randomized timers to elect leaders on top of the heartbeats. This helps resolve conflicts simply and rapidly
- membership changes:

  - uses _joint consensus_, the majorities of two diff configurations overlap during membership transitions. This overlap allows the cluster to continue operating normally during configuration changes

- prioritizes understandability so developers can develop an intuituin about it and the desirable properties
- Raft solves understandability of distributed consensus through decomposition of problem and simplifying the state space

raft has three states

- requests made to followers are routed to the leader. Followers do not make requests
- There is a single leader, the leader handles all requests

### Leader Election

servers are initially booted as followers as long as they receive communication from leaders or candidates. if there is no communication after an elapsed period of time, a follower will start an election to choose a new leader.

## Work

### State

### AppendEntries RPC

`appendEntries()`

- args:
  - term: leader's term
  - leaderId: so follower can redirect clients
  - prevLogIndex: index of log entry immediately preceding new ones
  - prevLogTerm: term of `prevLogIndex` entry
  - entries[]: log entries to store (empty for heartbeat)
  - leaderCommit: leader's `commitIndex`
- returns:
  - term: currentTerm, for leader to update itself.
  - successs: True if follower contained entry matching `prevLogIndex` and `prevLogTerm`
- pseudocode:
  - reply false if term < currentTerm
  - reply false if log doesn't contain an entry at prevLogIndex whose term matches prevLogTerm
  - If an existing entry conflicts with a new one (same index but different terms)
    - then delete the existing entry and all that follow
  - Append any new entries not already in the log
  - if leaderCommit > commitIndex
    - set commitIndex = min(leaderCommit, index of last new entry)

### RequestVote RPC

### Rules for Servers

## Background

## Results and Discussion

## Experimental

## replicated state machine problem

Definition: collection of servers compute identitical copies of the same state and continue operating if some servers are down in the cluster

GFS, HDFS have a single cluster leader and use a separate replicated machine to manage leader election and store config info -> like zookeeper or chubby

replicated log - commands read and executed in order so state is deterministic
a consensus algorithm's job is to keep the replicated log consistent.
a consensus module

- receives commands from a client and adds it to its log
- communicates with consensus modules on other servers to ensure that every state machine executes the log in the same order, regardless if it fails
- when commands are replicated and state machine executes in log order, the output is returned to the clients

### Consensus algorithm properties

- safety under network delays, partitions, data loss, reordering
- available if a majority is up - ex: can tolerate failure of 2 servers in a cluster of 5

## weakness of paxos

### single-decree paxos

authors posit systems should be built around a log.

## understandability of raft

- decomposition of tasks:
  - leader election
  - log replication
  - safety
  - membership changes

## raft

## evaluating raft
