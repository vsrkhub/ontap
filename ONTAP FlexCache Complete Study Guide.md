# NCDA Deep Dive
# FlexCache Architecture
# ONTAP FlexCache Complete Study Guide

---

# What is FlexCache?

FlexCache is a distributed caching technology that allows ONTAP to present a local copy (cache) of data that physically resides on another volume.

Think of it as:

```text
Primary Data
      |
      v
Origin Volume
      |
      |
      v
FlexCache Volume
```

Users access the FlexCache volume.

ONTAP retrieves data from:

```text
Origin Volume
```

and caches frequently used blocks locally.

---

# Why Does FlexCache Exist?

Primary goals:

```text
Reduce WAN Traffic
Reduce Latency
Accelerate Remote Reads
Scale Global NAS
Increase Read Performance
```

---

# Traditional Access

Without FlexCache:

```text
Sydney User
      |
      +----------------------+
                             |
                             v
                    Melbourne Storage
```

Every read traverses the WAN.

---

# FlexCache Access

With FlexCache:

```text
Origin Volume
(Melbourne)
      |
      |
      v
------------------------
FlexCache Volume
(Sydney)
------------------------
      ^
      |
Sydney Users
```

Frequently accessed blocks remain local.

---

# Core FlexCache Architecture

## Components

### Origin Volume

The authoritative data source.

Contains:

```text
Production Data
Snapshots
Permissions
Namespace
```

---

### FlexCache Volume

The cached copy.

Contains:

```text
Frequently Accessed Data
Metadata
Cached Blocks
```

---

### Client

Accesses:

```text
FlexCache
```

instead of directly accessing Origin.

---

# Architecture Diagram

```text
                CLIENT

                    |
                    |
                    v

          FLEXCACHE VOLUME

                    |
                    |
      Metadata + Cached Data

                    |
                    |
                    v

             ORIGIN VOLUME

                    |
                    |
                    v

             PRIMARY STORAGE
```

---

# Multi-Site Architecture

Example

```text
Sydney
Melbourne
Perth
Singapore
```

---

## Design

```text
                Origin Volume
                   Sydney
                      |
        --------------------------------
        |              |              |
        |              |              |
        v              v              v

    FlexCache      FlexCache      FlexCache
    Melbourne         Perth      Singapore
```

---

# Why Customers Deploy FlexCache

---

## Engineering Teams

Large CAD files.

Example:

```text
50 GB Drawing
```

Rather than every office reading:

```text
Across WAN
```

they read:

```text
Local FlexCache
```

---

## Media Companies

Large media assets.

---

## Global Software Development

Large source repositories.

---

## Home Directories

Distributed user access.

---

# Read Workflow

This is VERY important.

---

## Cache Miss

User requests:

```text
FileA
```

---

Not in cache.

Flow:

```text
Client
  |
FlexCache
  |
Origin Volume
```

---

Result:

```text
Data Retrieved
Data Cached
Data Returned
```

---

# Cache Hit

Same file requested later.

Flow:

```text
Client
  |
FlexCache
```

Only.

---

Origin volume not contacted.

---

Result:

```text
Lower Latency
Less WAN Traffic
```

---

# Write Workflow

NCDA favorite topic.

---

# Important Rule

Origin Volume Remains Authoritative

---

Write goes to:

```text
Origin Volume
```

---

Flow

```text
Client
   |
FlexCache
   |
Origin Volume
```

---

Origin commits write.

---

Cache updated.

---

# Exam Question

### Q

Who owns authoritative data?

Answer:

```text
Origin Volume
```

---

# FlexCache Metadata

FlexCache stores:

```text
Cached Data
Namespace Metadata
Directory Information
```

---

FlexCache does NOT replace:

```text
Origin Metadata
Origin Snapshots
Origin Data Ownership
```

---

# FlexCache and SnapMirror

Another NCDA favorite.

---

FlexCache:

```text
Performance
```

---

SnapMirror:

```text
Data Protection
```

---

Do not confuse them.

---

# FlexCache vs SnapMirror

## FlexCache

Purpose

```text
Read Acceleration
```

---

Data

```text
Single Source
```

---

Updates

```text
Realtime Access
```

---

## SnapMirror

Purpose

```text
Disaster Recovery
Replication
```

---

Data

```text
Multiple Copies
```

---

Updates

```text
Replication Schedule
```

---

# FlexCache vs FlexClone

---

## FlexClone

```text
Writable Clone
```

Independent object.

---

## FlexCache

```text
Cached View
```

Depends on origin.

---

# FlexCache vs FlashPool

---

## FlashPool

Caches:

```text
Blocks
```

Within:

```text
Same Aggregate
```

---

## FlexCache

Caches:

```text
Volume Data
```

Across:

```text
Clusters
Sites
Datacenters
```

---

# FlexCache Topologies

---

# Single FlexCache

```text
Origin
  |
  +----- FlexCache
```

---

# Fan-Out FlexCache

```text
Origin
  |
 ------------------------
 |          |           |
 v          v           v

FC1        FC2         FC3
```

Most common exam scenario.

---

# Cascaded FlexCache

Less common.

```text
Origin
   |
   v
Cache1
   |
   v
Cache2
```

Understand conceptually.

---

# FlexCache and Storage Efficiency

Important exam point.

---

## Dedupe

Runs on:

```text
Origin Volume
```

---

## Compression

Runs on:

```text
Origin Volume
```

---

## Compaction

Runs on:

```text
Origin Volume
```

---

FlexCache itself:

```text
Does NOT improve efficiency savings.
```

---

# FlexCache Benefits

---

## Reduced WAN Usage

Frequently read blocks become local.

---

## Faster Access

Data served locally.

---

## Less Load On Origin

Repeated reads avoided.

---

## Scalability

Global namespaces become easier.

---

# FlexCache Limitations

---

## Not DR

FlexCache is NOT:

```text
Backup
Disaster Recovery
Replication
```

---

## Origin Dependency

Origin remains authoritative.

---

## Additional Storage Required

Cache consumes space.

---

# Commands

---

## Show FlexCache

```bash
volume flexcache show
```

---

## Detailed Info

```bash
volume flexcache show -instance
```

---

## Show Volumes

```bash
volume show
```

---

## Show Junction Paths

```bash
volume show -fields junction-path
```

---

## Performance Statistics

```bash
statistics show
```

---

# Troubleshooting Scenarios

---

# Scenario 1

User reports:

```text
Slow initial file access
```

Reason:

```text
Cache Miss
```

---

# Scenario 2

User reports:

```text
Subsequent access very fast
```

Reason:

```text
Cache Hit
```

---

# Scenario 3

FlexCache exists.

Origin heavily loaded.

Reason:

```text
Cache warming phase
```

---

# Scenario 4

Branch office slow.

Investigate:

```text
Cache Hit Ratio
WAN Latency
Origin Reachability
```

---

# Scenario 5

Customer expects FlexCache to provide DR.

Incorrect design.

Need:

```text
SnapMirror
```

---

# NCDA Exam Traps

---

## Trap #1

FlexCache is a backup.

```text
FALSE
```

---

## Trap #2

FlexCache replaces SnapMirror.

```text
FALSE
```

---

## Trap #3

FlexCache owns production data.

```text
FALSE
```

---

## Trap #4

FlexCache improves dedupe savings.

```text
FALSE
```

---

## Trap #5

Writes are permanently stored only in FlexCache.

```text
FALSE
```

Origin remains authoritative.

---

# 30 NCDA FlexCache Questions

### Q1

What is the authoritative data source?

```text
Origin Volume
```

---

### Q2

What is the primary purpose of FlexCache?

```text
Reduce latency
```

---

### Q3

What command displays FlexCache volumes?

```bash
volume flexcache show
```

---

### Q4

Does FlexCache replace SnapMirror?

```text
No
```

---

### Q5

Does FlexCache provide DR?

```text
No
```

---

### Q6

What happens during a cache miss?

```text
Data retrieved from origin
```

---

### Q7

What happens during a cache hit?

```text
Data served locally
```

---

### Q8

What type of workload benefits most?

```text
Read-heavy
```

---

### Q9

Can multiple FlexCaches exist for one origin?

```text
Yes
```

---

### Q10

What is fan-out topology?

```text
One origin
Multiple FlexCaches
```

---

### Q11

Who owns snapshots?

```text
Origin Volume
```

---

### Q12

Does FlexCache store authoritative data?

```text
No
```

---

### Q13

Does FlexCache reduce WAN traffic?

```text
Yes
```

---

### Q14

Can FlexCache span clusters?

```text
Yes
```

---

### Q15

Can FlexCache span sites?

```text
Yes
```

---

### Q16

Does FlexCache improve read latency?

```text
Yes
```

---

### Q17

Is FlexCache a clone?

```text
No
```

---

### Q18

What technology is used for writable copies?

```text
FlexClone
```

---

### Q19

Does FlexCache improve dedupe savings?

```text
No
```

---

### Q20

Does FlexCache cache reads?

```text
Yes
```

---

### Q21

Does FlexCache cache metadata?

```text
Yes
```

---

### Q22

What command shows detailed FlexCache info?

```bash
volume flexcache show -instance
```

---

### Q23

Can users access a FlexCache directly?

```text
Yes
```

---

### Q24

What is a common use case?

```text
Remote Office Access
```

---

### Q25

What WAN benefit exists?

```text
Reduced traffic
```

---

### Q26

FlexCache belongs to which category?

```text
Performance Acceleration
```

---

### Q27

What remains authoritative?

```text
Origin Volume
```

---

### Q28

Which technology provides replication?

```text
SnapMirror
```

---

### Q29

Which technology provides caching?

```text
FlexCache
```

---

### Q30

Most important FlexCache exam answer?

```text
Origin Volume Is Always Authoritative
```

---

# NCDA Memory Dump

```text
FlexCache
=========

Purpose:
Read Acceleration

Authoritative Data:
Origin Volume

Cache Miss:
Origin Access

Cache Hit:
Local Access

Not:
Backup
DR
Replication

Benefits:
Lower Latency
Lower WAN Traffic
Remote Site Performance

Commands:

volume flexcache show

volume flexcache show -instance

volume show

volume show -fields junction-path

Most Common Exam Question:

Who owns authoritative data?

Answer:
Origin Volume
```

---
END OF FLEXCACHE GUIDE
