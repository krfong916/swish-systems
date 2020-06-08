## Overview

- To avoid split brain - VMWare FT uses a singular test-and-set server for determining a new primary during failure.
- The system model: the only I/O are the network packets from the client. We care about the timing of executing processes so the P/B do not deviate. P/B must execute packets at EXACTLY the same time.
- Log entries channel

### How does VMWare FT avoid split-brain?

The P/B use the log entries channel to exchange messages to indicate liveness. When a B receives no entries within a given window of time - it backup must go live. It is important that the B makes the distinction between a failure due to network partition and failure due to the P crashing. Suppose a backup detects the primary has failed.

Two ways to fail:

1. fail-stop (the primary completely fails)
2. both machines are up and running, but there's something funny happening on the network. P/B cannot talk to each other but can talk to clients - both will assume leadership, no longer publish log events, and will diverge - split-brain disaster if we let the P/B go live if a network error. To address the problem of split brain, VMWare FT uses a test-and-set service.

#### Test and Set

A test-and-set service guarantees only one VM is made primary in the presence of failure. If P/B are both up and we're experiencing a network partition, then both communicate with a test-and-set server. The first to write to the service is declared the leader. This test-and-set service sounds like an SOP (and you'd be correct), but itself is replicated and FT.

### Output Requirement

Output Requirement - the backup should produce the same output as the primary
Output Rule - a primary delays sending network packets until the backup has acknowledged the request (a write, read, update, etc.)
VMWare FT has a strict level of consistency - the internal file systems of the VMs cannot deviate. As a solution, the P and B share the same virtual disk. This provides safety of the output rule and requirement because the content of the disks will naturally be correct. Alternatively, the P and B could each have their own virtual disk, however the disks must be kept in-sync - this is a much harder problem to get right.

### Log Entry Channel

The log entry channel is used by the P and B to communicate liveness to each other.
Exchanging messages can take 5ms-10ms (possibly) if geograph. distributed.
For low intensity services, the amount of chit-chat is not a problem, but if millions of req. per second this is damaging for perf.
Performance thorn - the synchronous wait - can't let primary too far ahead of the backup, so every replication at some point the primary has to stall and wait for the backup -> real limit on performance
This is a big reason why we use a different replication scheme
application-level replication scheme
or a scheme that knows to halt only for write ops. and not every op. i.e. is fine with read-only ops.

we cannot rely on the user-level software to detect duplicate packets in the event of failover<span id="f1">[¹](#1)</span>
Really, any replication system (where cutover happens) it's impossible to guarantee a duplicate packet is not generated in the event of failover. Everybody errs on the side of generating duplicate output and, at some level, the client-side of replication schemes needs some duplicate detection scheme.

## Footnotes

<span id="1">¹</span> Footnote.[⏎](#f1)<br> suppose a primary sends a packet, fails over, we require the backup to be up to date with the primary. If the backup is behind processing writes from the log, it may end up sending the same packet to the client twice, once from the primary and once from the backup. If the lower-level does not detect that we've sent a packet already, we may be fortunate to be communicating to the client via TCP. The client software would detect duplicate packets and discard the packet from the backup. All this is to say, we cannot rely on the higher-level software of the client to guarantee duplicates are not sent

## Misc. Notes

#### What is a hypervisor? and what is a VM?

A hypervisor is software that emulates a computer. The guest OS and applications that execute inside the hypervisor is called a VM.

#### If a primary VM fails, what is the process of reliably failing over to a VM backup, and correctly apply the inputs and non-determinism? i.e. how does a VMWare FT handle failure? Does VMWare FT assume a hit in performance?

VMWare FT VMs use UDP heartbeats to check on each other. If the primary fails, the backup advertises its MAC address to the physical network so it can be used as the new primary. During the time that the primary fails and the backup promotes itself as primary, the backup must download and apply all updates it hasn't

#### What is VMWare FT? and what is unique about the design of VMWare FT compared to similar VMs?

no use of epochs

VMWare FT follows a stricter level of consistency - when the primary failsover - the backup (replica) is required to produce the same output as would the primary. To enforce this level of consistency, the primary does not send a network packet (a response) until the backup receives and acknowledges the request.
Primary sends packet to backup, waits for ack, and only then can it release the packet to the network back to the client. Backup just halts the output, not execution itself

An alternate design would have a separate virtual disk for the primary and backup. If this were so,
