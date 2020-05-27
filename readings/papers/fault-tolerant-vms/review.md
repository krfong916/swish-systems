## Notes

Output rule - a primary delays sending network packets until the backup has acknowledged the request (a write, read, update, etc.)

The primary and backup VM share virtual disks. We can safely failover because the content of the disks will naturally be correct. The alternative design is that each has their own virtual disk and must be kept in-sync.

## Elaborate

what is a FT VM and why is it useful?
A Hypervisor is
A VM is
A fault tolerant VM is

output requirement v. output rule
primary and backup failover, how to avoid split-brain
advantage of a shared storage
log channel and log buffer - primary and backup
disk I/O issues and non-determinism i.e. writing to the same disk location
