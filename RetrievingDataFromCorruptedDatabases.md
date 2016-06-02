# Problem

When a user attempts to open their database the open fails with an error message indicating that there is data corruption. Or a user encounters an error while using their database indicating a checksum mismatch was detected or an I/O operation failed unexpectedly.

Such failures are commonly caused by:
* Corruption by the underlying file I/O subsystem
* Misconfigured I/O subsystem - so that it does not allow adequate safety in the face of hard stop and system crashes.
* Bugs in WiredTiger storage engine
* Bugs in MongoDB integration layer

# Background

WiredTiger databases consist of multiple files, it is important to understand the roles of the different files that form part of a WiredTiger database in order to understand the different types of corruption that are possible and subsequently the approaches for recovering data from a corrupted database.

## Files in a WiredTiger database

Every WiredTiger database shares the same structure in terms of files on disk. A newly created MongoDB database consists of:

```
-rw-rw-r-- 1 alexg alexg 16384 Jun  2 00:28 collection-0-601483983079373625.wt
drwxrwxr-x 2 alexg alexg    47 Jun  2 00:28 diagnostic.data
-rw-rw-r-- 1 alexg alexg 16384 Jun  2 00:28 index-1-601483983079373625.wt
-rw-rw-r-- 1 alexg alexg 16384 Jun  2 00:28 _mdb_catalog.wt
-rw-r--r-- 1 alexg alexg     0 Jun  2 00:28 mongod.lock
-rw-rw-r-- 1 alexg alexg 16384 Jun  2 00:28 sizeStorer.wt
-rw-rw-r-- 1 alexg alexg    95 Jun  2 00:28 storage.bson
-rw-rw-r-- 1 alexg alexg    46 Jun  2 00:28 WiredTiger
-rw-rw-r-- 1 alexg alexg  4096 Jun  2 00:28 WiredTigerLAS.wt
-rw-rw-r-- 1 alexg alexg    21 Jun  2 00:28 WiredTiger.lock
-rw-rw-r-- 1 alexg alexg   907 Jun  2 00:28 WiredTiger.turtle
-rw-rw-r-- 1 alexg alexg 20480 Jun  2 00:28 WiredTiger.wt
```

If a database is started without the --nojournal option the database will also contain a `journal` subdirectory, that has the following content:
```
-rw-rw-r-- 1 alexg alexg 104857728 Jun  2 00:28 WiredTigerLog.0000000001
-rw-rw-r-- 1 alexg alexg 104857728 Jun  2 00:28 WiredTigerPreplog.0000000001
-rw-rw-r-- 1 alexg alexg 104857728 Jun  2 00:28 WiredTigerPreplog.0000000002

```

# Solutions

## Resync from a secondary
## Revert to a recent backup of data
## Run WiredTiger salvage utility on WiredTiger metadata file.
## Rebuild the metadata using a custom set of steps


# Related Articles

http://source.wiredtiger.com/develop/checkpoint.html

http://source.wiredtiger.com/develop/durability.html
