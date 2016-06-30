# Performance problems related to high cache utilization

## Problem

* Performance declines, including
  * slower operation rates
  * logged slow operations
  * queuing indicated by **active readers/writers** and
    **concurrentTransactions read/write out** (aka "tickets")
  * cache utilization exceeds 95% (**bytes currently in the cache** / **maximum bytes configured**)

## Background

Generally WiredTiger aims to limit cache utilization to 80% of the configured maximum
because the purpose of the cache is to provide a buffer between application
operations and disk i/o.  When the WiredTiger cache usage exceeds 95% of the configured
maximum performance decreases and significant operation latencies will result.
Two thresholds are of note:

* When the cache reaches 95% full, application threads will start doing
  evictions, increasing operation latencies. This is tracked by the
  **application threads doing evictions** metric.

* When the cache reaches 100% full, application threads may have to wait for
  the cache, also increasing operation latencies.
  [Q: **pthread mutex calls** metric?]

See [ReferenceMemoryUse](ReferenceMemoryUse.md).

## Diagnosis and remediation

### WiredTiger cache eviction algorithmic issue

* Diagnosis: high rate of **pages walked for eviction** (typically many
  millions per second) relative to pages evicted, which is sum of
  **modified pages evicted** and **unmodified pages evicted** (typically
  thousands or tens of thousands per second)

* Remediation: a specific diagnosis may be difficult. One or more of
  the following may apply:

    * [SERVER-23622](https://jira.mongodb.org/browse/SERVER-23622):
      issue appears to be triggered on the node that a secondary is
      replicating from if the secondary is delayed or lagging.

    * [SERVER-22831](https://jira.mongodb.org/browse/SERVER-22831):
      issue related to idle table. Fixed in 3.0.12, 3.2.5, 3.3.3.

    * [CS-31295](https://jira.mongodb.org/browse/CS-31295): SERVER
      and/or WiredTiger issues TBD.

### Disk write bottleneck

* Diagnosis: high proportion of dirty data in cache; *consistent* high disk
  utilization (bursts of 100% utilization, typically coinciding with checkpoints,
  are normal and expected)

* Remediation: improve storage performance; throttle workload

### CPU bottleneck related to reconciliation

* Diagnosis: ...

* Remediation: ...

### Extended transaction during checkpoints

* Symptoms: cache fills during checkpoints (**checkpoint currently
  running**), accompanied by large value for **pages pinned by
  checkpoint**)

* Remediation: ...

### Extended transaction outside checkpoints

* Symptoms: cache fills at times not necessarily correlated
  with checkpoints, accompanied by large values for **pages
  pinned** (note distinction between this metric and **pages
  pinned by checkpoint**).

* Remediation: escalate for further investigation as mongod
  is designed to avoid long-running transactions that can
  create this situation.
