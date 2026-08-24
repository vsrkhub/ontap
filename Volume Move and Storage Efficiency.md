# NCDA Deep Dive
# Volume Move and Storage Efficiency
# What Happens to Dedupe, Compression, Compaction During Volume Moves

---

# Why This Topic Matters

A common NCDA misconception is:

```text
Volume Move simply copies a volume.
```

In reality, Volume Move interacts heavily with:

```text
Storage Efficiency
Aggregate Layout
Compaction
Compression
Dedupe Metadata
Cross-Volume Dedupe
FabricPool
Snapshot Metadata
```

Many NCDA and real-world troubleshooting questions are centered around:

```text
Why did savings change after a volume move?
Why did capacity increase after migration?
Why did efficiency scans start after migration?
```

---

# What Happens During a Volume Move?

## High-Level Process

```text
Source Aggregate
        |
        | Volume Move
        v
Destination Aggregate
```

Example:

```text
aggr1
  |
  +-- vol_prod
```

Move:

```bash
volume move start \
-vserver svm1 \
-volume vol_prod \
-destination-aggregate aggr2
```

Result:

```text
aggr2
  |
  +-- vol_prod
```

---

# Is Volume Move Logical or Physical?

Volume Move performs:

```text
Physical Data Migration
```

Unlike:

```text
ARL (Aggregate Relocation)
```

which moves:

```text
Ownership Only
```

---

# What Happens to Storage Efficiency?

Most candidates get this wrong.

---

# Snapshot Retention

During move:

```text
Snapshots are preserved.
```

---

Example

Before

```text
VOL1

100 Snapshots
```

After move

```text
VOL1

100 Snapshots
```

Still exists.

---

# Deduplication Metadata

Preserved.

---

Example

Before move:

```text
Dedupe Savings = 45%
```

After move:

```text
Savings preserved
```

Usually yes.

---

# Compression Metadata

Preserved.

Compressed blocks remain compressed.

---

# Compaction

Compaction is different.

During volume movement:

```text
Data may be rewritten
```

which can provide opportunities for:

```text
Repacking
Reorganization
Optimization
```

depending on ONTAP release.

---

# Cross-Volume Dedupe Considerations

One of the most important advanced topics.

---

## Before Move

Aggregate

```text
aggr1

VOL1
VOL2
VOL3
```

Cross-volume dedupe exists.

---

Benefits:

```text
VOL1 shares blocks with VOL2
VOL2 shares blocks with VOL3
```

---

## Move VOL1

After Move

```text
aggr2

VOL1
```

---

Possible Result

```text
Cross-Volume Savings Lost
```

because:

```text
VOL1 no longer resides in same aggregate.
```

---

# Exam Question

### Question

A volume move completes.

Storage efficiency falls slightly.

Possible reason?

Answer:

```text
Loss of cross-volume deduplication opportunities.
```

---

# Aggregate Change Effects

When moving between:

```text
AFF → AFF
```

Usually:

```text
Minimal changes
```

---

When moving:

```text
FAS → AFF
```

You may gain:

```text
Compression
Compaction
Better inline efficiency
```

after future scans.

---

When moving:

```text
AFF → FAS
```

You may lose:

```text
Certain efficiency advantages
```

depending on policies.

---

# Storage Efficiency Processing

---

# Inline Efficiency

Occurs:

```text
During Writes
```

Components:

```text
Inline Dedupe
Inline Compression
Inline Compaction
```

---

# Background Efficiency

Occurs:

```text
After Data Written
```

Components:

```text
Post Process Dedupe
Post Process Compression
Scanner Jobs
```

---

Show Status

```bash
volume efficiency show
```

---

# Example Output

```text
Volume
State
Compression
Compaction
Inline Dedupe
Policy
Savings
```

---

# What Is Enabled By Default?

This depends on:

```text
Platform
ONTAP Version
Volume Type
```

---

# Modern AFF Systems

Typically enabled by default:

```text
Inline Deduplication

Inline Compression

Inline Compaction

Background Optimization
```

---

# Modern ASA

Generally follows AFF behavior.

Typically:

```text
Always-On Efficiency
```

for SAN workloads.

---

# FAS Systems

Historically:

```text
Efficiency features may exist
but not always enabled automatically.
```

Always verify.

---

Check:

```bash
volume efficiency show
```

Never assume.

---

# What Usually Requires Verification?

Administrators should verify:

```text
Dedupe
Compression
Compaction
Policies
Schedules
```

---

# Commands You Must Know

---

# Show Efficiency

```bash
volume efficiency show
```

---

# Detailed Efficiency

```bash
volume efficiency show -instance
```

---

# Start Scan

```bash
volume efficiency start
```

---

# Stop Scan

```bash
volume efficiency stop
```

---

# Show Policy

```bash
volume efficiency policy show
```

---

# Assign Policy

```bash
volume efficiency modify
```

---

# Show Savings

```bash
volume show-footprint
```

---

# Show Space

```bash
volume show-space
```

---

# Interpreting show-footprint

Very important command.

---

Example

```text
Volume Size = 10TB

Logical Used = 8TB

Physical Used = 3TB
```

Storage savings:

```text
5TB
```

via:

```text
Compression
Dedupe
Compaction
```

---

# Volume Move and Efficiency Scanners

After large migrations:

ONTAP may run:

```text
Efficiency Scans
```

to optimize placement.

---

Check jobs:

```bash
job show
```

---

Check efficiency:

```bash
volume efficiency show
```

---

# Volume Move Optimization

---

# Is Data Rehydrated?

Generally:

```text
No
```

Compressed blocks remain compressed.

---

# Are Snapshots Lost?

```text
No
```

Snapshots move with the volume.

---

# Is Dedupe Lost?

```text
No
```

Volume-local dedupe remains.

---

# Can Cross-Volume Savings Change?

```text
Yes
```

This is a common reason for footprint changes.

---

# FabricPool Interaction

---

Suppose:

```text
VOL1
```

moves from:

```text
FabricPool Aggregate
```

to:

```text
Non-FabricPool Aggregate
```

Potential results:

```text
Cold Data Behaviour Changes
Tiering Policies Re-Evaluated
```

---

Check:

```bash
volume show -fields tiering-policy
```

---

# AFF Detailed Behavior

AFF benefits most from:

```text
Inline Dedupe
Inline Compression
Inline Compaction
```

because workloads usually contain:

```text
VMs
Databases
Home Directories
VDI
```

Many:

```text
Small Writes
Repeated Blocks
Small Files
```

---

# Small File Example

Files:

```text
4 KB
3 KB
2 KB
```

Without compaction:

```text
Separate physical storage
```

---

With compaction:

```text
Packed together
```

Result:

```text
Higher SSD utilization
```

---

# Why AFF Shows Better Savings

Most AFF workloads contain:

```text
Small Random Writes
OS Images
VM Templates
Repeated Data
Metadata
```

All ideal for:

```text
Dedupe
Compression
Compaction
```

---

# FAS Detailed Behavior

Many FAS environments store:

```text
Backup Repositories
Media Data
Video
Archive Data
```

These workloads may already be:

```text
Compressed
Unique
Large Sequential Files
```

Result:

```text
Lower efficiency savings
```

---

# Troubleshooting Storage Efficiency

---

# Issue

Storage efficiency appears disabled.

Check:

```bash
volume efficiency show
```

---

# Issue

Savings lower than expected.

Investigate:

```text
Encrypted Data
Zip Files
Media Files
Already Compressed Content
```

---

# Issue

Volume move caused footprint increase.

Possible reasons:

```text
Lost Cross-Volume Dedupe

Different Aggregate Layout

New Efficiency Scan Pending
```

---

# Issue

Aggregate Full Despite Efficiency

Cause:

```text
Thin Provisioning Oversubscription
```

---

# NCDA Exam Traps

---

## Trap 1

Volume Move removes snapshots.

```text
FALSE
```

---

## Trap 2

Volume Move equals ARL.

```text
FALSE
```

---

## Trap 3

Compaction removes duplicate blocks.

```text
FALSE
```

Dedupe does.

---

## Trap 4

Compression and Dedupe are identical.

```text
FALSE
```

---

## Trap 5

Moving a volume guarantees identical savings afterwards.

```text
FALSE
```

Cross-volume dedupe effects may differ.

---

# Advanced NCDA Questions

### Q1

What command shows detailed efficiency status?

```bash
volume efficiency show -instance
```

---

### Q2

What command displays logical vs physical usage?

```bash
volume show-footprint
```

---

### Q3

Do snapshots move with a volume move?

```text
Yes
```

---

### Q4

Does ARL move data?

```text
No
```

---

### Q5

Does Volume Move move data?

```text
Yes
```

---

### Q6

Can moving a volume impact cross-volume dedupe?

```text
Yes
```

---

### Q7

What efficiency technology provides the largest benefit for small files?

```text
Compaction
```

---

### Q8

What efficiency technology removes duplicate blocks?

```text
Deduplication
```

---

### Q9

What efficiency technology shrinks blocks?

```text
Compression
```

---

### Q10

What command should be checked after a move if savings appear lower?

```bash
volume show-footprint
```

---

# Memory Dump

```text
Volume Move
===========
Moves Actual Data

ARL
===
Moves Ownership

Snapshots
=========
Move With Volume

Dedupe Metadata
===============
Preserved

Compression
===========
Preserved

Cross-Volume Dedupe
===================
May Change

Best Commands
=============

volume efficiency show

volume efficiency show -instance

volume show-space

volume show-footprint

volume move show

volume move start

job show

storage aggregate show-space
```

---
END OF STORAGE EFFICIENCY DEEP DIVE
