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
collection-0-601483983079373625.wt
diagnostic.data
index-1-601483983079373625.wt
_mdb_catalog.wt
mongod.lock
sizeStorer.wt
storage.bson
WiredTiger
WiredTigerLAS.wt
WiredTiger.lock
WiredTiger.turtle
WiredTiger.wt
```

The important files from that list, in terms of database structure and durability are:

### WiredTiger metadata file

The WiredTiger metadata file is called `WiredTiger.wt`. It tracks information about which collections and indexes (referred to from now on as tables) exist in the database, as well as the most recent durable update (checkpoint) in each of those tables. This is a WiredTiger table that is owned by WiredTiger.

### The WiredTiger turtle file

The WiredTiger turtle file is called `WiredTiger.turtle`. It contains the metadata for the WiredTiger metadata file described above. The turtle file is necessary for WiredTiger to track the most recent consistent durable point in time across all tables. This is a text file that is owned by WiredTiger.

### MongoDB metadata file

The MongoDB metadata file is called `_mdb_catalog.wt`. It contains information that MongoDB uses to keep metadata for MongoDB collections and indexes, including which WiredTiger table maps to each MongoDB collection or index. This is a WiredTiger table that is owned by MongoDB.

### The WiredTiger version file

The WiredTiger version file is called `WiredTiger`. It contains the version of WiredTiger used to create the database. It is used by WiredTiger on startup to determine whether a database is present. This is a text file that is owned by WiredTiger.

### The lock files

The lock files are called `WiredTiger.lock` and `mongod.lock`. They are used by MongoDB and WiredTiger to ensure that only a single process accesses the database at any particular time. These files don't hold useful information.

### The MongoDB size tracking file

The size tracking file is called `sizeStorer.wt`. It contains information about the amount of data currently in each MongoDB collection file. This is a WiredTiger table  that is owned by MongoDB.

### MongoDB collection and index files

The MongoDB collection files are called `collection-X-XXX.wt` and index files are called `index-X-XXX.wt`, where the `X` is replaced by an integer. Each MongoDB collection or index is stored in a separate WiredTiger table. The mapping between MongoDB collection names and WiredTiger table names is stored in the MongoDB metadata file. If MongoDB is started with the `--directoryPerDB` option these files may appear in a subdirectory.

### WiredTiger journal files

If a database is started without the --nojournal option the database will also contain a `journal` subdirectory that has the following content:

```
WiredTigerLog.0000000001
WiredTigerPreplog.0000000001
WiredTigerPreplog.0000000002
```

The imporant file here is the `WiredTigerLog.XX` file, which contains the [Write Ahead Log](https://en.wikipedia.org/wiki/Write-ahead_logging) (WAL) as maintained by WiredTiger.

# Solutions

## Resync from a secondary
## Revert to a recent backup of data
## Run WiredTiger salvage utility on WiredTiger metadata file.
## Rebuild the metadata using a custom set of steps

# Related Articles

http://source.wiredtiger.com/develop/checkpoint.html

http://source.wiredtiger.com/develop/durability.html
