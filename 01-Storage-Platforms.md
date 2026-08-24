# NCDA (NS0-165) Study Guide
# Domain 1 - Storage Platforms

**Objective:** Master ONTAP platform architecture, hardware, clustering, HA, scaling, upgrades, and fundamental storage concepts required for the NCDA exam.

---

# Chapter 1 - ONTAP Architecture Fundamentals

## Core Storage Hierarchy

Everything in ONTAP follows this hierarchy:

```text
Physical Disks
      ↓
RAID Groups
      ↓
Aggregates (Local Tiers)
      ↓
FlexVol Volumes
      ↓
Files / LUNs
      ↓
Hosts
```

---

## Physical Disks

Disks are the lowest storage component.

Types:

### HDD

Characteristics:

- Lowest cost
- High capacity
- High latency
- Archive workloads

Typical speeds:

```text
7200 RPM
10000 RPM
15000 RPM
```

---

### SSD

Characteristics:

- Low latency
- High IOPS
- Flash media
- Consistent performance

Used in:

```text
AFF
ASA
Performance Tier
```

---

## RAID Groups

A RAID group is a collection of disks protected by parity.

Example:

```text
23 SSDs
      ↓
1 RAID Group
```

or

```text
48 HDDs
      ↓
Multiple RAID Groups
```

---

## Aggregates (Local Tiers)

An Aggregate owns disks.

Modern ONTAP documentation frequently refers to an Aggregate as a Local Tier.

Example:

```text
aggr1
```

Contains:

```text
SSD RAID Groups
```

Volumes are created inside aggregates.

---

### Important Exam Fact

Question:

What object directly owns disks?

Answer:

```text
Aggregate
```

NOT Volume.

---

## FlexVol Volumes

Logical containers created within aggregates.

Example:

```text
vol_nfs_prod
vol_vmware_datastore1
vol_smb_hr
```

Used for:

- Files
- Shares
- LUNs
- Snapshots

---

## LUNs

Block devices created inside volumes.

Used by:

```text
VMware
Windows Clusters
Databases
Linux Servers
```

---

# Chapter 2 - NetApp Platforms

---

# AFF Systems

AFF = All Flash FAS

Built for:

```text
Performance
Virtualization
Databases
VDI
Mission Critical Applications
```

---

## AFF Characteristics

### Storage Media

```text
100% SSD
```

### Advantages

- Extremely low latency
- High throughput
- High IOPS
- Storage efficiency

### Typical Latency

```text
Sub-millisecond
```

---

## AFF Use Cases

### VMware

Large vSphere clusters

### SQL

Mission-critical databases

### Oracle

High performance workloads

### VDI

Boot storms

---

## Exam Questions

### Q1

Customer needs lowest latency possible.

Answer:

```text
AFF
```

---

### Q2

Customer prioritizes performance over capacity.

Answer:

```text
AFF
```

---

# FAS Systems

---

## Definition

Hybrid platform offering balance between:

```text
Performance
Capacity
Cost
```

---

## Characteristics

May use:

```text
SSD
SAS HDD
NL-SAS HDD
```

---

## Advantages

### High Capacity

PB-scale deployments.

### Lower Cost

Less expensive than AFF.

### Flexible

NAS
SAN
Backup

---

## Typical Workloads

- Home Directories
- Archive Storage
- Backup Repositories
- General NAS

---

## Exam Questions

### Q3

Customer needs 5PB archive platform.

Answer:

```text
FAS
```

---

### Q4

Customer needs lowest cost/TB.

Answer:

```text
FAS
```

---

# ASA Systems

---

## All SAN Array

Designed specifically for SAN.

Protocols:

```text
iSCSI
FC
NVMe/TCP
NVMe/FC
```

---

## Characteristics

### SAN Only

No NAS protocols.

### Symmetric Access

All paths active.

### Simpler SAN Design

Reduced complexity.

---

## Typical Workloads

```text
Databases
VMware
Enterprise SAN
Mission Critical Applications
```

---

## Exam Questions

### Q5

Customer uses only FC and iSCSI.

Recommended Platform?

Answer:

```text
ASA
```

---

### Q6

Customer requires NAS and SAN.

Answer:

```text
AFF or FAS
```

---

# Chapter 3 - Cluster Architecture

---

# ONTAP Cluster

Cluster provides:

```text
Scalability
Availability
Management
```

---

## Cluster Components

```text
Node
HA Pair
Cluster
```

---

## Node

A single controller.

Contains:

```text
CPU
Memory
NVRAM
Network Ports
Storage Access
```

---

## HA Pair

Two nodes.

Example:

```text
Node1
Node2
```

becomes:

```text
1 HA Pair
```

---

## Cluster Example

```text
A+B
C+D
```

Equals:

```text
4 Nodes
2 HA Pairs
1 Cluster
```

---

# Cluster Networking

Cluster nodes communicate using:

```text
Cluster LIFs
```

Purpose:

```text
Node-to-node communication
```

---

# Why Clustering Exists

Without clustering:

```text
Storage Silo
```

With clustering:

```text
Single Storage Pool
```

Benefits:

- Scale out
- Non-disruptive operations
- Volume movement
- Centralized administration

---

# Exam Questions

### Q7

Can a cluster consist of one node?

Generally production ONTAP clusters use HA pairs.

NCDA answer:

```text
No HA protection
```

---

### Q8

Why cluster nodes together?

```text
Scalability and high availability
```

---

# Chapter 4 - High Availability (HA)

---

# HA Overview

HA protects against controller failures.

---

# Takeover

When one node fails.

Example:

```text
NodeA fails
```

Partner:

```text
NodeB takes ownership
```

---

## Result

Clients continue accessing data.

---

# Giveback

After repair:

```text
NodeA returns
```

Ownership restored.

---

# HA Components

### NVRAM Synchronization

Partner continuously mirrors writes.

---

### Heartbeat

Nodes monitor partner health.

---

### Shared Storage Access

Both nodes can access storage.

---

# Commands

```bash
storage failover show
```

Check HA status.

---

```bash
storage failover takeover
```

Manual takeover.

---

```bash
storage failover giveback
```

Manual giveback.

---

# Exam Questions

### Q9

Node fails.

What maintains data access?

Answer:

```text
Takeover
```

---

### Q10

After repair, restore ownership?

Answer:

```text
Giveback
```

---

### Q11

What technology mirrors writes between partners?

Answer:

```text
NVRAM Mirroring
```

---

# HA Failure Scenarios

---

## Scenario 1

NodeA fails.

Expected behavior:

```text
NodeB assumes ownership
```

No outage.

---

## Scenario 2

Giveback fails.

Possible causes:

```text
Aggregate relocation issue
Hardware issue
Partner unhealthy
```

---

## Scenario 3

Cluster reports:

```text
Takeover possible = false
```

Investigate:

```bash
storage failover show
```

---

# Chapter 5 - Scaling ONTAP

---

# Scale Up

Add resources to existing controller.

Examples:

```text
Disks
Shelves
Memory
```

---

# Scale Out

Add nodes.

Example:

```text
2 Node Cluster
```

becomes

```text
4 Node Cluster
```

---

## Benefits

### More Capacity

Additional aggregates.

### More Performance

Additional CPUs.

### Higher Throughput

More networking.

---

# Exam Questions

### Q12

Customer needs more storage only.

Possible solution?

```text
Add shelves
```

---

### Q13

Customer needs more CPU and memory.

Answer:

```text
Add nodes
```

---

# Cluster Expansion Workflow

---

## Steps

1. Rack controllers
2. Cable networking
3. Join cluster
4. Verify health
5. Add storage

---

## Validation Commands

```bash
cluster show
```

```bash
system node show
```

```bash
storage aggregate show
```

---

# Chapter 6 - ONTAP Select

---

# What Is ONTAP Select?

Software-defined ONTAP.

Runs on:

```text
VMware
KVM
Hypervisors
```

---

## Benefits

No dedicated hardware requirements.

Can run on commodity servers.

---

## Use Cases

### Edge Locations

Retail stores.

### Remote Sites

Branch offices.

### Lab Environments

Testing.

---

## Limitations

Compared with AFF/FAS:

```text
Lower performance
Depends on host hardware
```

---

# Exam Questions

### Q14

Customer wants ONTAP without buying controllers.

Answer:

```text
ONTAP Select
```

---

# Chapter 7 - Cloud Volumes ONTAP

---

# What Is CVO?

Software-defined ONTAP in cloud.

Platforms:

```text
Azure
AWS
Google Cloud
```

---

## Benefits

Same ONTAP features:

```text
Snapshots
SnapMirror
Replication
Security
```

---

## Common Architecture

```text
On-Prem ONTAP
        ↓
SnapMirror
        ↓
Cloud Volumes ONTAP
```

---

## Typical Use Cases

### DR

Cloud disaster recovery.

### Backup

Cloud copy.

### Cloud Migration

Hybrid cloud deployments.

---

# Exam Questions

### Q15

Customer wants ONTAP in Azure.

Answer:

```text
Cloud Volumes ONTAP
```

---

### Q16

Customer needs cloud-based SnapMirror target.

Answer:

```text
Cloud Volumes ONTAP
```

---

# Chapter 8 - ONTAP Upgrades

---

# Upgrade Types

### Automated

Recommended.

### Manual

Rare.

---

# Upgrade Workflow

```bash
cluster image package get
```

Upload image.

---

```bash
cluster image validate
```

Run prechecks.

---

```bash
cluster image update
```

Perform upgrade.

---

# What Validation Checks

Validation verifies:

```text
Cluster Health
Storage Health
Networking Health
HA Health
Disk Health
Compatibility
```

---

# Rolling Upgrade

Nodes upgraded one at a time.

Benefits:

```text
No downtime
```

---

# Upgrade Sequence

```text
Node A Upgrade
      ↓
Giveback
      ↓
Node B Upgrade
```

---

# Common Upgrade Blockers

### Unhealthy Node

```bash
system health show
```

---

### HA Problems

```bash
storage failover show
```

---

### Missing Disk Ownership

```bash
storage disk show
```

---

### Cluster Port Issues

```bash
network port show
```

---

# Upgrade Exam Traps

### Trap #1

Upgrade image uploaded.

Can upgrade begin immediately?

Wrong Answer:

```text
Yes
```

Correct:

```text
Run Validation First
```

---

### Trap #2

Node unhealthy.

Upgrade anyway?

Correct:

```text
No
```

---

# Troubleshooting Scenarios

---

# Scenario 1

Cluster cannot add new node.

Check:

```bash
cluster show
system node show
```

Investigate:

- Cluster network
- Licensing
- Node health

---

# Scenario 2

After expansion node appears healthy but no storage visible.

Check:

```bash
storage aggregate show
storage disk show
```

Potential cause:

```text
Unassigned disks
```

---

# Scenario 3

Node takeover occurred unexpectedly.

Investigate:

```bash
event log show
storage failover show
```

Possible causes:

```text
Power failure
Hardware failure
Network issues
```

---

# Scenario 4

Upgrade validation fails.

Investigate:

```bash
cluster image validate
system health show
```

Potential causes:

```text
Cluster health alerts
Disk issues
HA issues
```

---

# Storage Platforms Exam-Cram Sheet

## Memorize

```text
AFF = Performance
FAS = Capacity
ASA = SAN
```

---

```text
Aggregate owns disks
Volume owns files/LUNs
```

---

```text
Takeover = Partner assumes ownership
Giveback = Ownership returns
```

---

```text
Scale Up = Add resources
Scale Out = Add nodes
```

---

```text
ONTAP Select = Software-defined ONTAP
```

---

```text
CVO = ONTAP in Public Cloud
```

---

```text
Upgrade Steps

Get Package
Validate
Update
```

---

# Mini Practice Questions

### 1

What object directly contains disks?

**Answer:** Aggregate

---

### 2

Which platform is SAN-focused?

**Answer:** ASA

---

### 3

Which platform is software-defined?

**Answer:** ONTAP Select

---

### 4

Which platform runs in Azure?

**Answer:** Cloud Volumes ONTAP

---

### 5

What protects against node failure?

**Answer:** HA

---

### 6

What restores ownership after takeover?

**Answer:** Giveback

---

### 7

What is the purpose of a cluster?

**Answer:** Scale and availability

---

### 8

What is the first step before an upgrade?

**Answer:** Upload image package

---

### 9

What should always be run before upgrade?

**Answer:** Validation

---

### 10

What object contains FlexVol volumes?

**Answer:** Aggregate
