# Problem

When a user attempts to open their database the open fails with an error message indicating that there is data corruption. Or a user encounters an error while using their database indicating a checksum mismatch was detected or an I/O operation failed unexpectedly.

Such failures are commonly caused by:
* Corruption by the underlying file I/O subsystem
* Misconfigured I/O subsystem - so that it does not allow adequate safety in the face of hard stop and system crashes.
* Bugs in WiredTiger storage engine
* Bugs in MongoDB integration layer

# Background


# Solutions

## Resync from a secondary
## Revert to a recent backup of data
## Run WiredTiger salvage utility on WiredTiger metadata file.
## Rebuild the metadata using a custom set of steps


# Related Articles

http://source.wiredtiger.com/develop/checkpoint.html

http://source.wiredtiger.com/develop/durability.html
