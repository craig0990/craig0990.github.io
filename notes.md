# Notes

## Requirements

Tge system must provide access primarily to the current version of a file, while 
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
