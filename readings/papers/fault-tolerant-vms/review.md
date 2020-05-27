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
