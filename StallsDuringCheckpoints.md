# Stalls During Checkpoints

## Problem

During checkpoint operations, which happen periodically in MongoDB, certain
operations become slow.  The affected operations can include ordinary queries
or updates, but may also involve collection creates / drops and server status
(including FTDC sampling).

## Background

A background thread in MongoDB triggers WiredTiger checkpoints every 60
seconds, or every time 2GB of log records are written (whichever is sooner).
 To complete a checkpoint, WiredTiger walks its cache and writes a version of
every dirty page to data files.  A checkpoint uses a transaction internally to
read a consistent version of data in memory.  Certain operations are blocked
during a checkpoint (such as dropping the WiredTiger table for a collection or
index), these report success immedately, and are queued internally and retried.

## Diagnosis and Remediation

### Ensure I/O is provisioned appropriately

1.  Look at “most recent checkpoint time (ms)” to see how long checkpoints are
    taking to complete

2.  Look at “tracked dirty bytes in cache” vs total cache size to see how much
    work checkpoint has to do

Remediation: faster disk / higher provisioned IOPS

### Checkpoint transaction blocking eviction

High (over 100K) *transaction IDs pinned by a checkpoint* to see how many
transactions start while a checkpoint is active

Remediation: faster disk / smaller cache / throttle updates

### Reads and writes blocked by the filesystem

The EXT3/4 filesystem can block all I/O for long periods during high write
loads.

Remediation: Avoid EXT3/4, our production notes recommend XFS.

# Related articles

SERVER-22209, SERVER-22199, SERVER-21691, SERVER-21652, SERVER-18314, SERVER-16176, CS-26971
