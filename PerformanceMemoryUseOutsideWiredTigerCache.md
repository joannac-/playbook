# Excess memory allocated outside the WiredTiger cache

## Problem

Generally memory usage outside the WiredTiger cache remains well bounded. However there are workloads that are known to consume large amounts of memory outside of the WiredTiger cache, and on occasion bugs that cause excess memory allocation have been found.

## Background

Users of MongoDB sometimes expect that the WiredTiger cache accounts of all memory used by MongoDB - that is not the case. The WiredTiger cache is a component of the WiredTiger storage engine that is used to facilitate access and updates to documents and indexes while minimizing interactions with the backing file system. The cache is generally the largest consumer of memory in a database system, but it is not the only consumer. The remainder of this article discusses how to identify when memory excessive amounts of memory are being allocated outside of the cache, and approaches for reducing the amount of memory being used.

See [ReferenceMemoryUse](ReferenceMemoryUse.md).

## Diagnosis and remediation

* Overall large memory use: **allocated minus wt cache** metric grows large
* High water mark reflecting a previous large allocation that has subsequently
  been freed: large **heap_size** and **total_free_bytes**.
* Memory fragmentation leading to performance issues or out of memory,
  accompanied by following (more-or-less equivalent):
  
  * **total free memory** (which is the sum of **pageheap_free_bytes**
    and **central_cache_free_bytes**) is large relative to
    **current_allocated_bytes**

  * **heap_size** minus **pageheap_unmapped_bytes** is large relative
    to **current_allocated_bytes**
  
  * **mem: resident** is large relative to **current_allocated_bytes**

  Ideally free memory tracked by tcmalloc should remain small so that
  unallocated memory can be used for kernel file cache. If free memory becomes
  large this can impact performance, and result in out-of-memory condition.

### Overall large memory usage

Causes: various, tbd

Remediation: smaller cache, identify offending query, more memory, cursor
timeouts; escalate within team for further analysis

### High water mark

At some point in the past a large amount of memory was allocated outside
the WiredTiger cache and then freed, resulting in a current large **heap_size** and
**total_free_bytes**.

* Diagnosis: examine memory usage history since last mongod restart
  (**uptime**) to find high-water mark for **current_allocated_bytes**

* Remediation: see section "Excess memory allocated outside the WiredTiger cache"

### Memory fragmentation

The workload has resulted in memory fragmentation: there is memory that has
been freed, but it cannot be reused to satisfy new allocation requests because
it is fragmented.

* Diagnosis: examine memory usage history since last mongod restart
  (**uptime**) to rule out a large **heap_size** due to a previous
  large **curent_allocated_bytes**

* Remediation: see SERVER-20306, SERVER-...
