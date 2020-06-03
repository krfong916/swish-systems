## Notes

VMWare FT follows a stricter level of consistency - when the primary failsover - the backup (replica) is required to produce the same output as would the primary. To enforce this level of consistency, the primary does not send a network packet (a response) until the backup receives and acknowledges the request.

An alternate design would have a separate virtual disk for the primary and backup. If this were so,

Output Requirement and Output Rule
Output Requirement - the backup should produce the same output as the primary
Output Rule - a primary delays sending network packets until the backup has acknowledged the request (a write, read, update, etc.)

#### What is VMWare FT? and what is unique about the design of VMWare FT compared to similar VMs?

no use of epochs

#### What is a hypervisor? and what is a VM?

A hypervisor is software that emulates a computer. The guest OS and applications that execute inside the hypervisor is called a VM.

#### If a primary VM fails, what is the process of reliably failing over to a VM backup, and correctly apply the inputs and non-determinism? i.e. how does a VMWare FT handle failure? Does VMWare FT assume a hit in performance?

VMWare FT VMs use UDP heartbeat to check on each other. If the primary fails, the backup advertises its MAC address to the physical network so it can be used as the new primary. During the time that the primary fails and the backup promotes itself as primary, the backup must download and apply all updates it hasn't

#### How does VMWare FT avoid split-brain?

A backup must go live when there is a failure, and only when a failure occurs to avoid split brain. Suppose a backup detects the primary has failed. Two events could have happened, either the primary had indeed failed, or all network connectivity had been lost between the servers and the primary is still operational. In both events, we now have two VMs declaring themselves as primary (not good). To address the problem of split brain, the primary and backup share the same virtual disk.

This provides safety of the output rule and requirement because

the content of the disks will naturally be correct. The alternative design is that each has their own virtual disk and must be kept in-sync.

deterministic replay
log channel and log buffer - primary and backup
disk I/O issues and non-determinism i.e. writing to the same disk location

lessons learned

primary sends packet to backup, waits for ack, and only then can it release the packet to the network back to the client. primary just halts the output, not execution itself
5ms-10ms possibly if geograph. distributed
for low intensity services, not a problem
but if millions of req. per second this is damaging for perf.
performance thorn - the synchronous wait - can't let primary too far ahead of the backup, so every replication
at some point the primary has to stall and wait for the backup
-> real limit on performance

this is a big reason why we use a different replication scheme
application-level replication scheme
or a scheme that knows to halt only for write ops. and not every op. i.e. is fine with read-only ops.

we cannot rely on the user-level software to detect duplicate packets in the event of failover<span id="f1">[¹](#1)</span>
Really, any replication system (where cutover happens) it's impossible to guarantee a duplicate packet is not generated in the event of failover. Everybody errs on the side of generating duplicate output and, at some level, the client-side of replication schemes needs some duplicate detection scheme.

Two ways to fail:

1. fail-stop (the primary completely fails)
2. both machines are up and running, but there's something funny happening on the network. P/B cannot talk to each other but can talk to clients - both will assume leadership, no longer publish log events, and will diverge - split-brain disaster if we let the P/B go live if a network error.

How to deal?
You cannot tell the type of failure. How about a Test-and-set "lock" service? If P/B are both up and we're experiencing a network partition, then communicate with a test-and-set server. First to write is the leader. This test-and-set service sounds like an SOP, but itself is replicated and FT.

## Footnotes

<span id="1">¹</span> Footnote.[⏎](#f1)<br> suppose a primary sends a packet, fails over, we require the backup to be up to date with the primary. If the backup is behind processing writes from the log, it may end up sending the same packet to the client twice, once from the primary and once from the backup. If the lower-level does not detect that we've sent a packet already, we may be fortunate to be communicating to the client via TCP. The client software would detect duplicate packets and discard the packet from the backup. All this is to say, we cannot rely on the higher-level software of the client to guarantee duplicates are not sent
