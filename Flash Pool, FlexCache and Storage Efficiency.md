# NCDA Deep Dive
# Flash Pool, FlexCache and Storage Efficiency

This is an advanced ONTAP topic that many administrators misunderstand.

The question is NOT:

```text
Does FlashPool/FlexCache provide efficiency?
```

The question is:

```text
What happens to dedupe, compression, compaction,
snapshots and efficiency accounting when FlashPool
or FlexCache are involved?
```

---

# Part 1 - Flash Pool

---

# What Is Flash Pool?

Flash Pool is a:

```text
Hybrid Aggregate Technology
```

that combines:

```text
HDD Tier
+
SSD Cache Tier
```

Example:

```text
Aggregate

20 x NL-SAS

+
4 x SSD
```

---

# Flash Pool Is NOT Storage Efficiency

Major exam point:

```text
Flash Pool ≠ Dedupe
Flash Pool ≠ Compression
Flash Pool ≠ Compaction
```

Flash Pool provides:

```text
Performance
Caching
Reduced HDD Reads
Reduced HDD Writes
```

NOT:

```text
Capacity Savings
```

---

# Architecture

```text
Client
   |
   v
ONTAP
   |
   +---- SSD Cache
   |
   +---- HDD Storage
```

---

# What Data Goes Into Flash Pool?

Depending on policy:

```text
Hot Data
Frequently Accessed Data
Random Reads
```

---

# Does Flash Pool Affect Dedupe?

Answer:

```text
No
```

Deduplication operates on:

```text
User Data Blocks
```

stored in the aggregate.

Flash Pool only caches blocks.

---

Example

Data:

```text
50GB duplicated files
```

Dedupe savings:

```text
Remains identical
```

whether Flash Pool exists or not.

---

# Does Flash Pool Affect Compression?

```text
No
```

Compressed blocks remain compressed.

Flash Pool caches:

```text
Compressed WAFL Blocks
```

---

# Does Flash Pool Affect Compaction?

```text
No
```

Compacted blocks remain compacted.

Flash Pool merely caches.

---

# Does Flash Pool Increase Efficiency Savings?

No.

Example:

Before Flash Pool

```text
Logical: 100TB

Physical: 40TB
```

Savings:

```text
60TB
```

---

After Flash Pool

```text
Logical: 100TB

Physical: 40TB
```

Savings:

```text
60TB
```

unchanged.

---

# What Changes?

Performance.

Not efficiency.

---

# NCDA Exam Trap

Question:

```text
Flash Pool improves storage efficiency savings.
```

Answer:

```text
FALSE
```

---

# Flash Pool and Snapshot Behavior

Snapshots still operate normally.

```text
No changes
```

Flash Pool does not alter:

```text
Snapshot Retention
Snapshot Reserve
Snapshot Metadata
```

---

# Flash Pool and Volume Move

Suppose:

```text
Volume moved
```

from

```text
Flash Pool Aggregate
```

to

```text
Non-FlashPool Aggregate
```

Result:

```text
Performance characteristics change

Efficiency remains unchanged
```

---

# Why Customers Confuse Flash Pool With Efficiency

Because customers often see:

```text
Less HDD Activity
Less HDD Usage
```

and believe:

```text
Storage Efficiency Improved
```

Actually:

```text
Performance Improved
```

---

# Part 2 - FlexCache

---

# What Is FlexCache?

A FlexCache volume is:

```text
A Cache Of Another Volume
```

---

Original Volume

```text
Origin Volume
```

---

Remote Site

```text
FlexCache Volume
```

---

Architecture

```text
Origin Volume
      |
      |
      v
FlexCache
```

---

# Purpose

Reduce latency.

---

Use Cases

```text
Remote Offices
Engineering Sites
Global NAS
Large File Shares
```

---

# Important Concept

FlexCache:

```text
Does NOT own primary data
```

Origin volume remains authoritative.

---

# Does FlexCache Affect Deduplication?

Generally:

```text
No
```

Deduplication occurs on:

```text
Origin Volume
```

---

FlexCache caches:

```text
Frequently Accessed Blocks
```

---

# Does FlexCache Affect Compression?

No.

Compressed blocks remain compressed.

---

# Does FlexCache Affect Compaction?

No.

Compaction remains on origin storage.

---

# Does FlexCache Create More Physical Consumption?

Yes.

This is where administrators get surprised.

---

Example

Origin Volume

```text
10TB Logical

4TB Physical
```

---

Create FlexCache

Now:

```text
Additional cache blocks exist.
```

---

Consumption increases.

---

# Important Distinction

FlexCache improves:

```text
Performance
Read Latency
User Experience
```

---

FlexCache does NOT improve:

```text
Dedupe Savings
Compression Savings
Compaction Savings
```

---

# Can Efficiency Savings Appear Different?

Yes.

Because:

```text
Cached Blocks Consume Space
```

administrators may see:

```text
Higher Footprint
```

and think:

```text
Efficiency lost
```

---

Actually:

```text
Cache Space Added
```

not efficiency reduction.

---

# FlexCache and Storage Reporting

Scenario

Origin:

```text
Logical 100TB

Physical 40TB
```

---

FlexCache created.

---

Administrator sees:

```text
New Physical Space Usage
```

---

Reason:

```text
Cache Storage
```

not loss of dedupe.

---

# FlexCache and Snapshots

FlexCache does not become:

```text
Snapshot Master
```

Origin volume retains ownership.

---

# FlexCache and Volume Efficiency Commands

Common exam question.

---

Check origin:

```bash
volume efficiency show
```

---

Shows:

```text
Real Dedupe Savings
Real Compression Savings
```

---

Checking cache volume:

```bash
volume show
```

does not mean you are looking at origin efficiency.

---

# Flash Pool vs FlexCache

---

# Flash Pool

Caches

```text
Blocks
```

Uses

```text
SSD
```

Location

```text
Same Aggregate
```

Purpose

```text
Performance
```

---

# FlexCache

Caches

```text
Volume Data
```

Uses

```text
Another Volume
```

Location

```text
Local or Remote Site
```

Purpose

```text
Distributed Access
```

---

# Storage Efficiency Impact Comparison

| Feature | Dedupe | Compression | Compaction | Snapshot | Capacity Savings |
|----------|----------|----------|----------|----------|----------|
| Flash Pool | No Impact | No Impact | No Impact | No Impact | No |
| FlexCache | No Impact | No Impact | No Impact | Managed on Origin | No |

---

# AFF + Flash Pool

Common trick question.

Actually:

```text
AFF does NOT typically require Flash Pool.
```

Because:

```text
AFF already uses SSDs.
```

---

Flash Pool typically appears on:

```text
Hybrid FAS
```

systems.

---

# AFF + FlexCache

Very common.

Example:

```text
AFF in Sydney

FlexCache in Melbourne

FlexCache in Perth
```

Provides:

```text
Low Latency Reads
```

---

Efficiency remains:

```text
Origin Volume Efficiency
```

---

# Real-World Troubleshooting

---

## Customer Reports

```text
Flash Pool added.

Why didn't dedupe savings increase?
```

Answer:

```text
Flash Pool affects performance only.
```

---

## Customer Reports

```text
FlexCache added.

Physical footprint increased.
```

Answer:

```text
Cache blocks consume space.
```

---

## Customer Reports

```text
FlexCache reduced WAN traffic.

Why?
```

Answer:

```text
Frequently accessed data is served locally.
```

---

# NCDA Questions

### Q1

Does Flash Pool provide dedupe?

```text
No
```

---

### Q2

Does Flash Pool improve performance?

```text
Yes
```

---

### Q3

Does Flash Pool improve capacity savings?

```text
No
```

---

### Q4

Does FlexCache deduplicate data?

```text
No
```

---

### Q5

Where do dedupe savings occur?

```text
Origin Volume
```

---

### Q6

Does FlexCache compress data?

```text
No
```

---

### Q7

Can FlexCache consume additional storage?

```text
Yes
```

---

### Q8

Does Flash Pool affect snapshot behavior?

```text
No
```

---

### Q9

Does Flash Pool affect compaction?

```text
No
```

---

### Q10

What is the primary benefit of FlexCache?

```text
Reduced Read Latency
```

---

# Memory Dump

```text
Flash Pool
==========

Hybrid Aggregate
SSD Cache
Performance Only

No Impact On:

Dedupe
Compression
Compaction
Snapshot Savings


FlexCache
=========

Volume Cache

Origin Volume
      |
      +---- FlexCache

Benefits:

Read Performance
Remote Site Access
Lower WAN Usage

No Impact On:

Dedupe
Compression
Compaction

May Consume Additional Cache Space


Storage Efficiency
==================

Dedupe
Compression
Compaction

Operate On:

Origin Data

NOT On:

Flash Pool Cache

NOT On:

FlexCache Cache Logic
```
``
