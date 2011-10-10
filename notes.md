# Notes

## Requirements

The system must provide access primarily to the current version of a file, while 
still providing access to old/deleted versions. Also needs to provide efficient 
synchronisation with multiple sources.

### Version Control
**Policies:** The system should support user-configurable policies, good examples 
being the number of versions kept (i.e. the last 10 modifications), the amount of 
time kept (i.e. versions from the last 10 days) or no previous versions (no 
history at all).

**Speed:** The system should be able to handle large files. *Really* large files. 
Ideally use a delta algorithm that does not require loading the entire file into 
memory. The `rsync` algorithm, with chunking and rolling checksums, is an excellent 
example.

**Storage:** The system should de-duplicate data wherever possible, to reduce the 
storage consumed by version history.

**Offline:** The system should still be able to function offline, merging it's 
changes to other locations when re-connected to the network.

### Synchronisation

The system must provide simple bi-directional synchronisation between two hosts in 
an efficient manner.

**Low-latency:** A low-latency communications channel is assumed. Broadband or 
local area networks typically fit this description and can be assumed for *most* 
people.

**Delta Compression:** The system should use delta compression to reduce the 
amount of data sent between systems. Additional improvements should also be 
considered.

## Prior Work

Unsurprisingly, Version Control Systems (VCS), particularly those of a distributed nature, 
introduced many benefits to delta compression and associated algorithms.

Git is a prominent DVCS (Distributed VCS) that uses a simple blob structure to represent 
file versions. These blobs are themselves stored in files. This simple approach gives Git
speed, efficiency and is particularly suited to distributed development. However, this 
approach assumes the files under version control are small and will fit into memory.

Some modifications have been attempted to alleviate this, so far none are satisfactory. 
The `git-annex` project removes files from the Git repository and replaces them with
symbolic links. This solves nothing, it simply prevents Git from choking on large files.

The `bup` project aims to provide efficient backups of *very* large files/filesystems, 
while still using the Git packfile format. This allows for Git compatibility; they have 
also implemented a modified `rsync` algorithm for efficiently handling large files. This 
approach introduces other problems, mainly with Git's handling of many large packfiles.

## How Version Control Works

The following is a personal interpretation. It may have inconsistencies or be entirely wrong.

Basically, dirty bits used to identify pages in memory that had been modified in databases. This 
worked well enough, but couldn't be applied to files because there is no guarantee a modification
is contained to a particular page. Instead, the initial approach was to log every modification 
to a file, which could then be used to reconstruct it. This is essentially what delta compression 
does, except delta compression is an awful lot more efficient.

A differencing algorithms find the difference between two versions of the same file. A delta 
file is the encoded output of such an algorithm. These algorithms require two versions of a file, 
a reference file (the state the file was previously in) and a version file (the state of the file 
with modifications).

Similarly, to reconstruct a file, we need the inverse: a delta file and a reference file. There 
are essentially two options for reference files: we can store them locally or we can request them
from the server. Server requests add latency and may not always be guaranteed, so are best 
avoided. A small cache of reference files would be more suitable and has one of the simplest 
implementations.

### The Client
When the client notifies the server of a new file to be backed up, it includes that file. After 
this initial step, the client and the server both have a reference file. Modification on either 
results in a delta file, which is then transmitted and reconstructed at the other end. In other 
words both client and server transmit only delta information after the initial backup.

When the reference file cache is full, the least recently used (LRU) is removed. If there is more 
than one such file, the file with the largest ratio of delta size to original file size is 
deleted. This allows the most inefficient LRU file to be removed, increasing the efficiency 
of the store.

### The Server

Hmm...
