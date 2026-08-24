# NCDA Deep Dive
# FlexCache Configuration, Validation, Monitoring, Troubleshooting and Flash Cache

This guide covers:

```text
FlexCache Architecture
FlexCache Configuration
Cache Validation
Performance Monitoring
Troubleshooting
Real Production Issues
Flash Cache
When To Use Flash Cache
How To Verify Flash Cache Is Working
```

---

# Part 1 - FlexCache Deployment Architecture

## Single Site

```text
Client
   |
   |
   V

FlexCache Volume
   |
   |
   V

Origin Volume
```

---

## Branch Office

```text
Sydney Office
     |
     |
     V

FlexCache Volume
     |
   WAN
     |
     V

Origin Volume
(Melbourne)
```

---

## Global Environment

```text
                    Origin Volume
                      Sydney
                          |
      ------------------------------------------
      |                    |                  |
      |                    |                  |
      V                    V                  V

  FlexCache           FlexCache          FlexCache
  Melbourne             Perth           Singapore
```

---

# FlexCache Requirements

Before configuring:

Verify:

```text
Cluster Peering
SVM Peering
Intercluster LIFs
ONTAP Supports FlexCache
Network Connectivity
```

---

# Step 1 - Verify Cluster Peering

```bash
cluster peer show
```

Expected:

```text
Available = true
Healthy = true
```

---

# Step 2 - Verify SVM Peering

```bash
vserver peer show
```

Expected:

```text
Peered
```

---

# Step 3 - Verify Intercluster LIFs

```bash
network interface show -role intercluster
```

Expected:

```text
Up/Up
```

---

# Step 4 - Verify Source Volume

Example:

```bash
volume show
```

Output:

```text
svm_prod
vol_engineering
```

This will become:

```text
Origin Volume
```

---

# Step 5 - Create FlexCache

Example:

```bash
volume flexcache create \
-vserver svm_branch \
-volume fc_engineering \
-origin-volume svm_prod:vol_engineering \
-size 5TB
```

---

# Result

```text
Origin Volume
      |
      +---- FlexCache Volume
```

created successfully.

---

# Verify FlexCache

```bash
volume flexcache show
```

Example:

```text
Volume          Type
-------------------------
fc_engineering  flexcache
```

---

# Detailed Verification

```bash
volume flexcache show -instance
```

Shows:

```text
Origin SVM
Origin Volume
Cache Volume
State
Caching Parameters
```

---

# Viewing Namespace

Verify junction path.

```bash
volume show -fields junction-path
```

Example:

```text
/engineering
```

---

# Mount Validation

Client Side

Linux

```bash
mount -t nfs server:/engineering /mnt
```

---

Windows

```text
\\servername\engineering
```

---

# How To Know FlexCache Is Actually Working

This is where most engineers stop too early.

Creating FlexCache is easy.

Proving it works is important.

---

# First Access

User accesses:

```text
50GB CAD File
```

---

Flow:

```text
Client
 ↓
FlexCache
 ↓
Origin
```

---

This is:

```text
Cache Miss
```

---

# Second Access

Same file.

```text
Client
 ↓
FlexCache
```

Only.

---

This is:

```text
Cache Hit
```

---

# Questions To Ask

```text
WAN usage reduced?

Read latency reduced?

Repeat reads faster?
```

If yes:

```text
FlexCache working.
```

---

# Commands To Monitor

Show Statistics

```bash
statistics show
```

---

Advanced Statistics

```bash
statistics start
```

---

```bash
statistics stop
```

---

```bash
statistics show
```

---

# Node Performance

```bash
statistics show -object volume
```

---

# Client Latency

```bash
statistics show -object nfs
```

---

SMB

```bash
statistics show -object cifs
```

---

# Indicators FlexCache Is Working

You should observe:

```text
Repeated Reads Faster

WAN Utilization Lower

Origin CPU Lower

Origin Read Operations Reduced
```

---

# Real World Example

Before FlexCache

```text
Melbourne CAD Server

500 users

WAN Traffic:
1.2 Gbps
```

---

After FlexCache

```text
WAN Traffic:
300 Mbps
```

because cached files become local.

---

# Troubleshooting FlexCache

---

# Problem 1

Users Say:

```text
First Access Is Slow
```

Answer:

```text
Expected
```

Reason:

```text
Cache Miss
```

---

# Problem 2

Every Access Is Slow

Check:

```bash
volume flexcache show
```

---

Check:

```bash
cluster peer show
```

---

Check:

```bash
vserver peer show
```

---

Possible Causes

```text
Peering Broken

WAN Link Issues

Origin Unreachable
```

---

# Problem 3

FlexCache Exists But No Performance Improvement

Check:

```text
Read Percentage
```

---

FlexCache benefits:

```text
Read Heavy Workloads
```

Not:

```text
Write Heavy Databases
```

---

# Problem 4

Users Cannot Access Volume

Verify:

```bash
volume show -fields junction-path
```

---

Common Cause:

```text
Missing Junction Path
```

---

# Problem 5

Origin Cluster Overloaded

Check:

```text
Cache Warm Up
```

---

Reason

```text
Every requested block initially comes from origin.
```

---

# Problem 6

Cache Not Updating

Verify:

```bash
volume flexcache show -instance
```

---

Investigate

```text
Origin Reachability

Intercluster LIFs

Peering
```

---

# When Should You Use FlexCache?

Excellent Candidates

```text
Engineering Files

Home Directories

Video Editing

Software Repositories

CAD/CAM Data

Remote Offices
```

---

Poor Candidates

```text
High Write Databases

Transactional Systems

Low Latency SAN Databases
```

---

# Part 2 - Flash Cache

Many administrators confuse:

```text
FlashCache

FlashPool

FlexCache
```

---

# Flash Cache

Flash Cache is:

```text
Controller Based Cache
```

Not:

```text
Volume Cache
```

Not:

```text
Aggregate Cache
```

---

# Architecture

```text
Client
   |
   V

Controller

+----------------+
| Flash Cache    |
+----------------+

      |
      V

Aggregate
```

---

# What Does Flash Cache Cache?

Mostly:

```text
Read Data

Random Read Data

Frequently Accessed Blocks
```

---

# Ideal Environment

Excellent Choices

```text
NFS

VMware

Home Directories

VDI

Engineering Data

Read Heavy NAS
```

---

Poor Candidates

```text
Sequential Backup Workloads

Media Streams

Large Sequential Reads
```

---

# Flash Cache vs Flash Pool

## Flash Cache

Located:

```text
Controller
```

---

Purpose:

```text
Read Cache
```

---

## Flash Pool

Located:

```text
Aggregate
```

---

Purpose:

```text
SSD Tiering
```

---

# How To Determine Flash Cache Is Working

This is a favorite interview question.

---

# Check Cache Statistics

```bash
statistics show
```

---

Advanced

```bash
statistics start -object wafl
```

---

```bash
statistics stop
```

---

Review:

```bash
statistics show
```

---

Look For

```text
Cache Hits

Cache Misses

Read Reduction
```

---

# What Is A Good Sign?

Cache Hit Rate Increasing.

Example:

```text
Hit Rate

85%
```

Excellent.

---

Example:

```text
Hit Rate

5%
```

Not effective.

---

# Real Example

Before Flash Cache

```text
Disk Reads

50,000 IOPS
```

---

After Warmup

```text
Disk Reads

10,000 IOPS
```

---

Cache Hit

```text
40,000 IOPS
```

---

Result

```text
Lower Latency

Reduced Disk Activity

Better Performance
```

---

# How To Troubleshoot Flash Cache

---

# Problem

No Improvement

---

Check:

```text
Sequential Workload?
```

---

Reason

```text
Sequential workloads don't benefit much.
```

---

# Problem

Low Hit Ratio

---

Check:

```text
Working Set Larger Than Cache
```

---

Example

```text
500GB Working Set

64GB Cache
```

---

Hit rate may remain low.

---

# Problem

Performance Still Poor

Check:

```text
CPU

Network

Aggregate Latency
```

Flash Cache cannot solve:

```text
CPU Bottlenecks
```

---

# NCDA Exam Scenarios

### Scenario 1

Remote office repeatedly opens same files.

Best solution:

```text
FlexCache
```

---

### Scenario 2

Users read same VM files repeatedly.

Best solution:

```text
Flash Cache
```

---

### Scenario 3

Need DR copy.

Best solution:

```text
SnapMirror
```

---

### Scenario 4

Need WAN reduction.

Best solution:

```text
FlexCache
```

---

### Scenario 5

Need lower HDD reads.

Best solution:

```text
Flash Cache
```

---

# NCDA Memory Dump

```text
FlexCache
=========

Volume Based Cache

Best For:
Remote Offices

Benefits:
WAN Reduction
Read Acceleration
Distributed NAS

Commands:

volume flexcache show

volume flexcache show -instance

cluster peer show

vserver peer show


Flash Cache
===========

Controller Cache

Best For:
Read Heavy NAS

Benefits:
Reduce Disk Reads
Reduce Latency

Look For:
Cache Hits
Cache Misses
Hit Rates


FlexCache
=========
Remote-Site Performance

Flash Cache
===========
Local Read Acceleration

SnapMirror
==========
Disaster Recovery

Flash Pool
==========
SSD Aggregate Tier
```

--- 
END OF FLEXCACHE + FLASH CACHE OPERATIONS GUIDE
