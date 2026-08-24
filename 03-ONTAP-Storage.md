# 03-ONTAP-Storage.md
# NCDA (NS0-165) Complete Storage Domain Guide

---

# Domain Objectives

This section covers:

```text
WAFL
RAID-DP / RAID-TEC
Disks
RAID Groups
Aggregates
Root Aggregates
FlexVol
FlexGroup
Junction Paths
Namespaces
Volume Guarantees
Fractional Reserve
Snapshot Technology
FlexClone
Volume Move
Storage Efficiency
FabricPool
Storage Troubleshooting
Capacity Management
```

---

# Chapter 1 - WAFL (Write Anywhere File Layout)

---

## What is WAFL?

WAFL stands for:

```text
Write Anywhere File Layout
```

WAFL is the filesystem architecture used by ONTAP.

Unlike traditional filesystems:

```text
NTFS
EXT4
XFS
```

WAFL never overwrites existing blocks directly.

---

## Traditional Filesystem Example

Initial file:

```text
Block1
Block2
Block3
```

Modify Block2

Result:

```text
Block2 overwritten
```

Old copy lost.

---

## WAFL Example

Modify Block2

Result:

```text
New Block Written Elsewhere
Metadata Updated
```

Old Block remains.

---

## Benefits of WAFL

Supports:

```text
Snapshots
FlexClone
Fast Recovery
Consistency Points
Rapid Restore
Volume Mobility
```

---

## NCDA Exam Point

Question:

```text
Why are ONTAP snapshots so efficient?
```

Answer:

```text
WAFL only updates metadata pointers.
```

---

# Chapter 2 - Consistency Points (CP)

---

## What is a Consistency Point?

WAFL collects writes in memory.

Periodically commits them to disk.

This process is called:

```text
Consistency Point
```

---

## Workflow

```text
Client Write
     ↓
NVRAM
     ↓
Memory
     ↓
Consistency Point
     ↓
Disk
```

---

## Why Important?

Provides:

```text
Performance
Consistency
Crash Recovery
```

---

## NCDA Question

If a node crashes after acknowledging a write:

```text
NVRAM protects the data
```

---

# Chapter 3 - Disks

---

## Disk Types

### SSD

Characteristics

```text
Low Latency
High IOPS
Flash
```

---

### HDD

Characteristics

```text
High Capacity
Lower Performance
Lower Cost
```

---

## Disk Ownership

Every disk belongs to:

```text
One Node
```

Verify:

```bash
storage disk show
```

---

# Chapter 4 - RAID

---

## RAID-DP

Default ONTAP RAID.

Protection:

```text
2 Disk Failures
```

---

## RAID-TEC

Protection:

```text
3 Disk Failures
```

---

## RAID Group Structure

Example

```text
20 SSD Data Disks
2 SSD Parity Disks
```

---

## Commands

Show RAID Layout

```bash
storage aggregate show-status
```

---

Show Aggregate

```bash
storage aggregate show
```

---

## Exam Questions

Which RAID type survives 2 failures?

```text
RAID-DP
```

---

Which RAID survives 3 failures?

```text
RAID-TEC
```

---

# Chapter 5 - Aggregates (Local Tiers)

---

## What is an Aggregate?

Aggregate = Collection of RAID Groups.

Contains:

```text
Disks
RAID Groups
Volumes
```

---

## Modern Name

```text
Local Tier
```

---

## Aggregate Diagram

```text
aggr1

├── RAID Group
├── RAID Group
├── RAID Group
│
├── vol1
├── vol2
└── vol3
```

---

## Aggregate Ownership

Important:

```text
Aggregate owns disks
```

NOT:

```text
Volume
```

---

## Commands

Show Aggregates

```bash
storage aggregate show
```

---

Capacity

```bash
storage aggregate show-space
```

---

Ownership

```bash
storage aggregate show -fields home-name
```

---

# Aggregate States

## Online

Serving data.

---

## Offline

Unavailable.

---

## Restricted

Maintenance state.

---

## Exam Trap

Question:

```text
Can a Volume exist outside an Aggregate?
```

Answer

```text
No
```

---

# Chapter 6 - Root Aggregate

---

## Purpose

Stores:

```text
ONTAP Configuration
Logs
Core Files
Metadata
```

---

## Example

```text
aggr0
```

---

## Important Rule

Do not store user data here.

---

## Why?

Root aggregate supports:

```text
Node Operations
Boot Process
Configuration Database
```

---

## Consequences If Root Aggregate Fails

```text
Boot Problems
Veto During Giveback
Node Stability Issues
```

---

# Chapter 7 - FlexVol Volumes

---

## What is a FlexVol?

Primary storage container.

---

## Structure

```text
Aggregate
      ↓
Volume
      ↓
Files / LUNs
```

---

## FlexVol Benefits

```text
Thin Provisioning
Snapshots
Clone Support
Volume Move
Autosize
```

---

## Commands

Create

```bash
volume create
```

---

Show

```bash
volume show
```

---

Modify

```bash
volume modify
```

---

Delete

```bash
volume delete
```

---

# Volume Settings

---

## Security Style

### UNIX

Used by:

```text
NFS
```

---

### NTFS

Used by:

```text
SMB
```

---

### Mixed

Mixed protocol environment.

---

Check

```bash
volume show -fields security-style
```

---

# Junction Paths

---

## What is a Junction Path?

Path inside SVM namespace.

Example:

```text
/finance
/hr
/home
```

---

## Namespace Example

```text
SVM Root
  |
  +-- finance
  |
  +-- hr
  |
  +-- engineering
```

---

## Commands

Show Path

```bash
volume show -fields junction-path
```

---

Mount Volume

```bash
volume mount
```

---

Unmount

```bash
volume unmount
```

---

## Common Issue

Volume exists.

Client cannot access.

Potential cause:

```text
No Junction Path
```

---

# Chapter 8 - FlexGroup

---

## What is a FlexGroup?

Scale-out volume.

Built from multiple constituents.

---

Example

```text
FlexGroup

├─ constituent1
├─ constituent2
├─ constituent3
└─ constituent4
```

---

## Benefits

```text
Massive Scale
Billions of Files
Higher Throughput
Parallelism
```

---

## FlexVol vs FlexGroup

### FlexVol

Single volume.

---

### FlexGroup

Multiple constituents.

---

## Use Cases

```text
AI
Research
Media Archives
Large NAS
```

---

# Chapter 9 - Space Guarantees

---

## Guarantee Types

### None

Thin Provisioned.

Most common.

---

### Volume

Reserves full volume size.

---

### File

Used mainly for LUN environments.

---

## Check

```bash
volume show -fields space-guarantee
```

---

## NCDA Exam

Question:

```text
Volume larger than physical storage?
```

Answer:

```text
Thin Provisioning
```

---

# Chapter 10 - Snapshot Reserve

---

## Purpose

Reserved space for snapshots.

---

Example

Volume:

```text
100 GB
```

Snapshot Reserve:

```text
5%
```

Snapshots get:

```text
5 GB
```

---

## Commands

```bash
volume show -fields percent-snapshot-space
```

---

## Problem

High snapshot usage.

Result:

```text
User Space Reduced
```

---

# Chapter 11 - Fractional Reserve

---

## What Is It?

Reserve overwrite space.

Primarily SAN related.

---

## Purpose

Protect LUN overwrites.

---

## Values

```text
0%
100%
```

---

## Check

```bash
volume show -fields fractional-reserve
```

---

## NCDA Question

Which feature guarantees overwrite space?

```text
Fractional Reserve
```

---

# Chapter 12 - Snapshot Technology

---

## What is a Snapshot?

Read-only point-in-time copy.

---

## Characteristics

```text
Instant
Read Only
Space Efficient
Metadata Based
```

---

## Important NCDA Fact

Snapshot is NOT:

```text
Full Copy
```

---

Snapshot IS:

```text
Metadata Pointer
```

---

## Commands

Show

```bash
volume snapshot show
```

---

Create

```bash
volume snapshot create
```

---

Delete

```bash
volume snapshot delete
```

---

Restore

```bash
volume snapshot restore
```

---

# Snapshot Deletion Logic

Blocks are deleted only when:

```text
No Active File Uses Block
AND
No Snapshot References Block
```

---

# Chapter 13 - FlexClone

---

## What is FlexClone?

Writable clone.

Created from:

```text
Volume
Snapshot
LUN
```

---

## Benefits

```text
Instant
Space Efficient
Fast
```

---

## Example

100 TB Volume.

Create FlexClone.

Initial consumption:

```text
Near Zero Additional Space
```

---

## Commands

```bash
volume clone create
```

---

```bash
volume clone show
```

---

## Use Cases

```text
Dev/Test
Backup Validation
DB Testing
VM Cloning
```

---

# Chapter 14 - Storage Efficiency

---

Components:

```text
Deduplication
Compression
Compaction
Thin Provisioning
```

---

## Deduplication

Eliminates duplicate blocks.

---

Example

```text
100 Windows Servers
```

Store common blocks once.

---

## Compression

Shrinks blocks.

---

## Compaction

Combines partially used blocks.

---

## Command

```bash
volume efficiency show
```

---

Run Efficiency

```bash
volume efficiency start
```

---

# Chapter 15 - Volume Move

---

## Purpose

Move volume between aggregates.

Non-disruptive.

---

## Commands

Start

```bash
volume move start
```

---

Status

```bash
volume move show
```

---

Cancel

```bash
volume move cancel
```

---

# Example

Before

```text
aggr1
 |
 +-- vol_prod
```

---

After

```text
aggr2
 |
 +-- vol_prod
```

---

# Chapter 16 - FabricPool

---

## Purpose

Move cold data.

---

## Architecture

```text
Performance Tier
      ↓
Capacity Tier
```

---

## Capacity Tier Examples

```text
AWS S3
Azure Blob
StorageGRID
```

---

## Tiering Policies

### None

Never tier.

---

### Snapshot-only

Tier snapshots.

---

### Auto

Automatic.

---

### All

Tier aggressively.

---

## Verify

```bash
volume show -fields tiering-policy
```

---

# Chapter 17 - Storage Troubleshooting

---

# Volume Full

Commands

```bash
volume show
```

---

```bash
volume autosize show
```

---

```bash
volume snapshot show
```

---

```bash
volume show-footprint
```

---

# Aggregate Full

Commands

```bash
storage aggregate show-space
```

---

Solutions

```text
Add Disks
Move Volume
Delete Data
```

---

# Snapshot Consuming Space

Check

```bash
volume snapshot show
```

---

Delete

```bash
volume snapshot delete
```

---

# Volume Move Stuck

Check

```bash
volume move show
```

---

```bash
job show
```

---

```bash
event log show
```

---

# 50 High-Value NCDA Questions

## Architecture

1. What owns disks?
   - Aggregate

2. What owns files?
   - Volume

3. What owns RAID groups?
   - Aggregate

4. What is WAFL?
   - Write Anywhere File Layout

5. Why are snapshots efficient?
   - Metadata pointers

## RAID

6. RAID-DP protects against?
   - 2 disk failures

7. RAID-TEC protects against?
   - 3 disk failures

8. Default ONTAP RAID?
   - RAID-DP

## Aggregates

9. Modern name for aggregate?
   - Local Tier

10. Command to display aggregates?
    - storage aggregate show

11. Aggregate contains?
    - Volumes

12. Volume contains?
    - Files/LUNs

## Volumes

13. Command to display volumes?
    - volume show

14. Can a volume span aggregates?
    - No

15. What feature grows a volume automatically?
    - Autosize

16. Most common guarantee?
    - none

17. What supports thin provisioning?
    - Guarantee none

## Namespace

18. Command to show junction path?
    - volume show -fields junction-path

19. What is a namespace?
    - Hierarchical path structure

20. Volume inaccessible but exists. Check?
    - Junction path

## Snapshots

21. Snapshot type?
    - Read-only

22. Restore command?
    - volume snapshot restore

23. Show command?
    - volume snapshot show

24. Snapshot is full copy?
    - No

25. Snapshot uses metadata?
    - Yes

## FlexClone

26. FlexClone is?
    - Writable clone

27. Clone command?
    - volume clone create

28. Major benefit?
    - Instant creation

## Efficiency

29. Removes duplicate blocks?
    - Deduplication

30. Shrinks blocks?
    - Compression

31. Packs partial blocks?
    - Compaction

32. Efficiency command?
    - volume efficiency show

## FlexGroup

33. Used for?
    - Massive scale NAS

34. Supports billions of files?
    - Yes

35. Multiple constituents?
    - Yes

## FabricPool

36. Moves what data?
    - Cold data

37. Capacity tier example?
    - AWS S3

38. Verify policy?
    - volume show -fields tiering-policy

## Troubleshooting

39. Aggregate full command?
    - storage aggregate show-space

40. Volume full command?
    - volume show

41. Snapshot consumption command?
    - volume snapshot show

42. Volume move status?
    - volume move show

## Advanced

43. Root aggregate stores?
    - ONTAP metadata

44. Root aggregate for user data?
    - No

45. Fractional reserve protects?
    - LUN overwrites

46. Volume move moves?
    - Volume data

47. ARL moves?
    - Aggregate ownership

48. FabricPool performance tier contains?
    - Hot data

49. FabricPool capacity tier contains?
    - Cold data

50. First command for storage troubleshooting?
    - volume show / storage aggregate show-space depending on issue

---
END OF 03-ONTAP-Storage.md
