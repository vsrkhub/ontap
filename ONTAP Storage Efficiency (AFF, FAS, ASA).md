# NCDA Deep Dive
# ONTAP Storage Efficiency (AFF, FAS, ASA)
# Deduplication, Compression, Compaction, Thin Provisioning

---

# Why Storage Efficiency Exists

Storage efficiency is one of ONTAP's biggest differentiators.

Goals:

```text
Reduce Physical Storage Consumption
Increase Effective Capacity
Reduce Cost/TB
Improve Flash Utilization
Reduce SSD Wear
```

Example:

```text
100TB Physical

With Efficiency

200TB+
Logical Capacity
```

---

# Storage Efficiency Stack

Think of ONTAP efficiency as layers.

```text
User Data
    ↓
Deduplication
    ↓
Compression
    ↓
Compaction
    ↓
Thin Provisioning
    ↓
Physical Storage
```

---

# NCDA Exam Tip

Many candidates think:

```text
Deduplication
Compression
Compaction
```

are the same.

They are completely different technologies.

---

# Storage Efficiency Technologies

```text
Deduplication
Compression
Compaction
Thin Provisioning
Data Reduction
```

---

# What Is Enabled By Default?

Depends on:

```text
ONTAP Version
AFF
ASA
FAS
Volume Type
```

---

## Modern AFF Systems

Typically enabled by default:

```text
Inline Deduplication
Inline Compression
Inline Compaction
Background Efficiency
```

---

## Modern ASA Platforms

Usually inherit AFF data reduction behavior.

```text
Inline Dedupe
Compression
Compaction
```

enabled aggressively.

---

## Traditional FAS

Historically:

```text
Efficiency Optional
```

Must verify.

Check:

```bash
volume efficiency show
```

---

# Verify Efficiency

```bash
volume efficiency show
```

Shows:

```text
Dedupe
Compression
Compaction
Policy
Status
Savings
```

---

# Deduplication Deep Dive

---

# What Is Deduplication?

Dedupe identifies identical blocks.

Stores:

```text
One Physical Copy
```

References:

```text
Many Logical Copies
```

---

# Example

Without Dedupe

```text
File A

Block X

File B

Block X

File C

Block X
```

Stored:

```text
3 copies
```

---

With Dedupe

Stored:

```text
1 copy
```

Referenced:

```text
3 times
```

---

# Real VMware Example

100 Windows VMs

Each VM has:

```text
Windows OS
DLL Files
System Files
```

Most blocks are identical.

Without Dedupe:

```text
100 copies stored
```

With Dedupe:

```text
Store once
Reference 100 times
```

---

# Types of Deduplication

---

## Inline Dedupe

Occurs:

```text
Before write committed
```

Workflow:

```text
Write Arrives
↓
Compared
↓
Duplicate?
↓
Don't write new block
```

---

## Background Deduplication

Runs after data written.

Workflow:

```text
Data Written
↓
Scanner Runs
↓
Blocks Compared
↓
Duplicates Removed
```

---

# NCDA Question

Inline dedupe occurs:

```text
Before blocks written
```

---

Background dedupe occurs:

```text
After blocks written
```

---

# Compression Deep Dive

---

# What Is Compression?

Compression reduces block size.

Unlike dedupe:

```text
No identical block required.
```

---

Example

Original

```text
100 KB
```

Compressed

```text
40 KB
```

---

Stored:

```text
40 KB
```

instead of:

```text
100 KB
```

---

# Compression Works Well For

```text
Text
Logs
Databases
Virtual Machines
```

---

# Compression Works Poorly For

```text
Encrypted Files
Already Compressed ZIP Files
JPEG Images
MP4 Files
```

---

# Types of Compression

## Inline Compression

Compress before writing.

---

## Post-process Compression

Compress later.

---

# NCDA Exam Trap

Question:

```text
Compression removes duplicate blocks.
```

Answer:

```text
False
```

That's dedupe.

---

# Compaction Deep Dive

---

# What Is Compaction?

Compaction combines partially-filled blocks.

---

Imagine:

```text
Block A = 20% Full
Block B = 25% Full
Block C = 30% Full
```

Without compaction:

```text
3 blocks used
```

---

With compaction:

```text
Pack together
```

Result:

```text
1 block used
```

---

# Why Compaction Exists

Flash systems frequently store:

```text
Small Random Writes
Small Files
Database Updates
Metadata
```

Compaction dramatically increases efficiency.

---

# AFF Loves Compaction

AFF workloads generate:

```text
Lots of Small IO
```

Compaction performs extremely well.

---

# Small File Handling

---

# Traditional File System

Small file:

```text
4KB
```

written into:

```text
8KB block
```

Result:

```text
4KB wasted
```

---

# ONTAP Compaction

Multiple small writes packed together.

Example:

```text
3KB
2KB
1KB
```

Packed into:

```text
Single physical block
```

---

# Example Workloads

Excellent Savings

```text
Home Directories
VMware Metadata
Office Documents
Database Logs
VMDK Metadata
```

---

# AFF Data Reduction

AFF achieves savings through:

```text
Inline Dedupe
Inline Compression
Inline Compaction
Background Passes
```

All operate together.

---

# Exam Question

What provides greatest savings for small files?

Answer:

```text
Compaction
```

---

# Thin Provisioning

---

# What Is Thin Provisioning?

Logical size exceeds physical size.

---

Example

Aggregate:

```text
10 TB
```

---

Volumes Created:

```text
Volume1 = 10 TB

Volume2 = 10 TB

Volume3 = 10 TB
```

Total:

```text
30 TB Logical
```

---

Physical:

```text
10 TB
```

---

# Benefits

```text
Higher Utilization
Reduced Upfront Costs
More Flexibility
```

---

# Risks

Oversubscription.

---

Example

All users suddenly consume space.

Result:

```text
Aggregate Full
```

---

# Check Guarantee

```bash
volume show -fields space-guarantee
```

---

Most modern environments:

```text
Guarantee = None
```

---

# AFF vs FAS Efficiency

---

# AFF

Storage efficiency optimized.

Technologies:

```text
Inline Dedupe
Inline Compression
Inline Compaction
Background Dedupe
Background Compression
```

Expected savings:

```text
2x-5x+
```

depending on workload.

---

# FAS

Depends on workload.

Generally:

```text
Lower efficiency benefit
```

than AFF.

Especially when:

```text
Large Video Files
Archive Data
Backup Data
```

---

# Why AFF Gets Better Savings

Flash systems:

```text
Small Blocks
Random IO
Virtual Machines
```

These workloads benefit greatly from:

```text
Compaction
Compression
Dedupe
```

---

# Cross-Volume Deduplication

---

# Traditional Dedupe

Only inside volume.

Example:

```text
VOL1
```

Blocks compared only within VOL1.

---

# Cross Volume Dedupe

Can compare:

```text
VOL1
VOL2
VOL3
```

inside same aggregate.

---

Potentially higher savings.

---

# Storage Efficiency Policies

---

Show:

```bash
volume efficiency policy show
```

---

Assign:

```bash
volume efficiency modify
```

---

View Status:

```bash
volume efficiency show
```

---

# Data Reduction Order

Simplified View

```text
Write Arrives
     ↓
Inline Dedupe
     ↓
Inline Compression
     ↓
Inline Compaction
     ↓
Write Storage
     ↓
Background Efficiency
```

---

# Commands You Must Know

---

Check Efficiency

```bash
volume efficiency show
```

---

Start Efficiency

```bash
volume efficiency start
```

---

View Policy

```bash
volume efficiency policy show
```

---

Volume Space

```bash
volume show-space
```

---

Volume Footprint

```bash
volume show-footprint
```

---

Aggregate Capacity

```bash
storage aggregate show-space
```

---

# Troubleshooting Efficiency

---

## Savings Lower Than Expected

Check:

```bash
volume efficiency show
```

---

Possible causes:

```text
Efficiency Disabled
Already Compressed Data
Encrypted Data
Video Files
Zip Files
```

---

## Aggregate Full Despite Dedupe

Reason:

```text
Thin Provisioning Oversubscription
```

---

## No Compression Savings

Likely:

```text
Data Already Compressed
```

---

## No Dedupe Savings

Likely:

```text
Highly Unique Data
```

---

# NCDA Exam Traps

---

## Trap 1

Dedupe compresses data.

False.

---

## Trap 2

Compression removes duplicate blocks.

False.

---

## Trap 3

Compaction shrinks data.

False.

Compaction packs small blocks.

---

## Trap 4

Thin Provisioning saves capacity through dedupe.

False.

Thin provisioning is space allocation.

---

## Trap 5

Snapshots are a storage efficiency feature.

Partially true.

Snapshots leverage WAFL efficiency but are not dedupe/compression technologies.

---

# Memory Dump

```text
Deduplication
=============
Removes duplicate blocks

Compression
===========
Shrinks block size

Compaction
==========
Packs partially used blocks

Thin Provisioning
=================
Allocates logical space
without reserving physical space

AFF
===
Best efficiency savings

FAS
===
Depends heavily on workload

Small Files
===========
Compaction provides biggest gains

Most Important Commands
=======================

volume efficiency show

volume efficiency start

volume efficiency policy show

volume show-space

volume show-footprint

storage aggregate show-space
```
