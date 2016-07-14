# Stalls During Checkpoints

## Problem

During checkpoint operations, which happen periodically in MongoDB, certain operations become slow.  The affected operations can include ordinary queries or updates, but may also involve collection creates / drops and server status (including FTDC sampling).

## Background

A background thread in MongoDB triggers a WiredTiger checkpoint every 60 seconds, or every time 2GB of WiredTiger log records are written (whichever is sooner). To complete a checkpoint, WiredTiger walks its cache and writes a version of every page that has been updated to data files.  A checkpoint uses a transaction internally to read a consistent version of data in memory.  Certain operations are blocked during a checkpoint (such as dropping the WiredTiger table for a collection or index), these report success immedately, and are queued internally and retried.

## Diagnosis and Remediation

The following diagnosis steps involve inspecting statistics generated over time. See StatisticsHOWTO for general information.

### Ensure I/O is provisioned appropriately

1.  Look at *most recent checkpoint time (ms)* from db.serverStatus() to see how long checkpoints are taking to complete

2.  Look at *tracked dirty bytes in cache* from db.serverStatus() to see how much work each checkpoint has to do

Remediation: faster disk / higher provisioned IOPS

### Checkpoint transaction blocking eviction

1. Look at *transaction IDs pinned by a checkpoint* to see how many transactions start while a checkpoint is active. If this value plateaus at high value (e.g., over 100 thousand) AND
2. With *bytes currently in the cache* being similar to *maximum bytes configured* AND
3. Periodically high values for *checkpoint blocked page eviction* or *pages selected for eviction unable to be evicted*

Remediation: faster disk / smaller cache / throttle updates.

If a smaller cache size is unacceptable because of the impact on read performance, consider running

```
mongod ... --wiredTigerEngineConfigString "eviction_dirty_target=10,eviction_dirty_trigger=20"
```

This may help by allowing reads to make use of the full cache size but constraining updates to a small fraction of the cache (in this case, between 10% and 20% of the cache size).  That in turn allows checkpoints to complete more quickly, reducing its impact on application operations.

### Reads and writes blocked by the filesystem

The EXT3/4 filesystem can block all I/O for long periods during high write loads. To isolate I/O blocking as the cause, inspect the output of iostat on POSIX, and look for high or spiking average queue depth (avgqu-sz) and/or long average wait time (await).

Remediation: Avoid EXT3/4, our production notes recommend XFS.

# Related articles

SERVER-22209, SERVER-22199, SERVER-21691, SERVER-21652, SERVER-18314, SERVER-16176, CS-26971

http://source.wiredtiger.com/develop/tune_cache.html#cache_eviction
