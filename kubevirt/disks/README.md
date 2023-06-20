# Kubevirt Disks

These files are the declarative way to pre-seed disk images onto a node. They used the CDI HTTP import process which is mush slower than a direct upload (I really need to open a ticket about it). 
They additionally force their volumes to bind without waiting for first consumer, but that should be removed when using these in a multi-node setup.
