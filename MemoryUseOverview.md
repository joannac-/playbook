# Memory usage overview

## Background

This section summarizes the organization of overall mongod process
memory usage, both outside and inside the tcmalloc
allocator. Indentation below indicates containment.

* **mem: virtual**: overall process virtual address space.
   * code and initialized data.
   * thread stacks, one per connection.
   * **generic heap_size**: virtual memory under control of the
     allocator.
      * **current_allocated_bytes**: memory allocated by mongod
         * **bytes currently in the cache**: WT cache memory
         * **allocated minus wt cache**: memory allocated outside
           the WT cache
      * freed memory used by mongod and then returned to the allocator
         * **central_cache_free_bytes**: small free regions
         * **pageheap_free_bytes**: large free regions backed by physical memory
         * **pageheap_unmapped_bytes** large free regions not backed by physical memory
