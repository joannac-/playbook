# Excess memory allocated outside the WT cache

## Symptoms

Generally memory usage outside the WT cache should remain well bounded.
However there are exceptions, and on occasion bugs have been found in this
area.

* Overall large memory use: **allocated minus wt cache** metric grows large
* High water mark: large **heap_size** and **total_free_bytes**.
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

## Background

See [MemoryUseOverview].

## Diagnosis and remediation

### Overall large memory usage

Causes: various, tbd

Remediation: smaller cache, identify offending query, more memory, cursor
timeouts; escalate within team for further analysis

### High water mark

At some point in the past a large amount of memory was allocated outside
the WT cache and then freed, resulting in a current large **heap_size** and
**total_free_bytes**.

* Diagnosis: examine memory usage history since last mongod restart
  (**uptime**) to find high-water mark for **current_allocated_bytes**

* Remediation: see section "Excess memory allocated outside the WT cache"

### Memory fragmentation

The workload has resulted in memory fragmentation: there is memory that has
been freed, but it cannot be reused to satisfy new allocation requests because
it is fragmented.

* Diagnosis: examine memory usage history since last mongod restart
  (**uptime**) to rule out a large **heap_size** due to a previous
  large **curent_allocated_bytes**

* Remediation: see SERVER-..., SERVER-...
