# NCDA Deep Dive
# Cluster Size, HA Takeover, Giveback, Aggregate Relocation (ARL)

---

# Cluster Fundamentals

## What Is An ONTAP Cluster?

A cluster is a group of storage controllers (nodes) that operate as a single logical storage system.

Benefits:

```text
High Availability
Scalability
Non-Disruptive Operations
Centralized Management
Workload Mobility
```

---

# How Many Nodes Can An ONTAP Cluster Have?

## Common Configurations

### 2 Node Cluster

```text
Node A <> Node B
```

Most common SMB/Enterprise deployment.

Provides:

```text
1 HA Pair
```

---

### 4 Node Cluster

```text
Node A <> Node B
Node C <> Node D
```

Provides:

```text
2 HA Pairs
```

---

### 8 Node Cluster

```text
4 HA Pairs
```

Used in larger environments.

---

### 12 Node Cluster

```text
6 HA Pairs
```

---

### 24 Node Cluster

Maximum traditional ONTAP NAS cluster size.

```text
12 HA Pairs
24 Controllers
```

Must know this for NCDA.

---

# Important NCDA Fact

Always think:

```text
Cluster
|
+-- HA Pair
     |
     +-- Node A
     +-- Node B
```

---

# HA Architecture

## What Is High Availability (HA) ?

HA allows one node to continue serving data if its partner fails.

Example:

```text
Node A Failure
      ↓
Node B Takes Ownership
```

Client access continues.

---

# How Does HA Work?

Several technologies work together.

---

## NVRAM Mirroring

Every write is mirrored to partner node.

Example:

```text
Write Arrives
      ↓
Node A NVRAM
      ↓
Node B NVRAM Copy
```

If Node A dies:

```text
Node B has write history
```

---

## Heartbeat

Nodes continuously monitor each other.

Checks:

```text
CPU Health
Node Availability
Communication
Storage Access
```

---

## Shared Storage Access

Both nodes can access storage.

During takeover:

```text
Partner reads disks
```

without interruption.

---

# Takeover

## Definition

Takeover occurs when one node assumes ownership of the partner's storage resources.

---

# Normal Ownership

```text
Node A -> aggr1
Node B -> aggr2
```

---

# After Takeover

```text
Node B

Owns:
aggr1
aggr2
```

---

# Why Takeover Happens

### Planned

Administrator initiates.

### Unplanned

Node failure.

---

# Commands

## Check HA State

```bash
storage failover show
```

Example:

```text
Takeover Possible : true
Giveback Possible : true
```

---

## Force Takeover

```bash
storage failover takeover
```

---

## Takeover Of Specific Node

```bash
storage failover takeover -ofnode node1
```

---

# Giveback

## Definition

Returns aggregate ownership back to the original node.

---

# Example

Before Failure

```text
NodeA -> aggr1
NodeB -> aggr2
```

---

After Failure

```text
NodeB

aggr1
aggr2
```

---

After Giveback

```text
NodeA -> aggr1
NodeB -> aggr2
```

---

# Giveback Commands

## Check Giveback Status

```bash
storage failover show-giveback
```

---

## Start Giveback

```bash
storage failover giveback
```

---

## Giveback To Specific Node

```bash
storage failover giveback -ofnode node1
```

---

# Aggregate Relocation (ARL)

## What Is ARL?

Aggregate Relocation.

Moves aggregate ownership without moving data.

Commonly used during:

```text
ONTAP Upgrades
Controller Replacement
Maintenance
Hardware Refresh
```

---

## Important Difference

### Aggregate Relocation

Moves ownership.

```text
Aggregate remains in place
```

---

### Volume Move

Moves data.

```text
Volume physically migrates
```

---

# ARL Commands

Show Aggregates

```bash
storage aggregate show
```

---

Relocate Aggregate

```bash
storage aggregate relocation start
```

---

Check Relocation

```bash
storage aggregate relocation show
```

---

# Planned Takeover Scenario

## Scenario

Need to upgrade Node A.

---

### Step 1

Verify HA

```bash
storage failover show
```

---

### Step 2

Perform Takeover

```bash
storage failover takeover -ofnode nodeA
```

---

### Result

```text
Node B serves all workloads
```

---

### Step 3

Upgrade Node A

---

### Step 4

Perform Giveback

```bash
storage failover giveback -ofnode nodeA
```

---

### Result

```text
Workloads return
```

---

# Unplanned Takeover Scenario

## Scenario

Node A Power Failure

Normal

```text
NodeA -> aggr1
NodeB -> aggr2
```

---

Node A crashes

```text
NodeB
|
+-- aggr1
+-- aggr2
```

---

Users continue access.

This is one of the most tested NCDA concepts.

---

# Giveback Failure Scenario

## Example

Node recovered but giveback fails.

Check:

```bash
storage failover show
```

---

Possible causes

```text
Partner not healthy
Aggregates offline
Network issues
Hardware issues
```

---

# Cluster Commands You Must Know

---

## Cluster Membership

```bash
cluster show
```

Shows

```text
Nodes
Eligibility
Health
```

---

## Node Information

```bash
system node show
```

Shows

```text
Node Name
Uptime
Model
Health
```

---

## Cluster Identity

```bash
cluster identity show
```

Shows

```text
Cluster Name
UUID
```

---

## HA Status

```bash
storage failover show
```

Shows

```text
Takeover State
Giveback State
Partner Health
```

---

## Aggregate Status

```bash
storage aggregate show
```

Shows

```text
Aggregate Ownership
State
Capacity
```

---

## Cluster Health

```bash
system health show
```

Shows

```text
Cluster Health Status
```

---

## Cluster Communication

```bash
cluster ring show
```

Shows

```text
Ring Health
Node Communication
VLDB Synchronization
```

---

## Cluster Connectivity

```bash
cluster ping-cluster -node local
```

Shows

```text
Node-to-Node Reachability
Packet Loss
Latency
```

---

# NCDA Exam Traps

## Trap 1

Question:

Takeover moves volumes.

Answer:

```text
False
```

Takeover moves aggregate ownership.

---

## Trap 2

Question:

Giveback moves data.

Answer:

```text
False
```

Returns ownership.

---

## Trap 3

Question:

Volume Move and ARL are same.

Answer:

```text
False
```

---

## Trap 4

Question:

Node failure means data unavailable.

Answer:

```text
False
```

Partner assumes ownership.

---

## Trap 5

Question:

Takeover requires storage migration.

Answer:

```text
False
```

---

# 30 NCDA Questions
# Cluster + Takeover + Giveback

### Q1

What is the maximum traditional ONTAP cluster size?

**Answer**

```text
24 nodes
```

---

### Q2

How many HA pairs in an 8-node cluster?

```text
4
```

---

### Q3

How many HA pairs in a 24-node cluster?

```text
12
```

---

### Q4

What command displays cluster nodes?

```bash
cluster show
```

---

### Q5

What command displays node information?

```bash
system node show
```

---

### Q6

What command displays HA status?

```bash
storage failover show
```

---

### Q7

What is takeover?

```text
Partner assumes aggregate ownership.
```

---

### Q8

What is giveback?

```text
Ownership returned to original node.
```

---

### Q9

What technology protects writes during takeover?

```text
NVRAM Mirroring
```

---

### Q10

Does takeover move volumes?

```text
No
```

---

### Q11

Does takeover move aggregate ownership?

```text
Yes
```

---

### Q12

What command performs takeover?

```bash
storage failover takeover
```

---

### Q13

What command performs giveback?

```bash
storage failover giveback
```

---

### Q14

What command shows giveback status?

```bash
storage failover show-giveback
```

---

### Q15

Node A crashes. Who serves data?

```text
Node B
```

---

### Q16

Which process automatically occurs after node failure?

```text
Takeover
```

---

### Q17

What should be checked before planned maintenance?

```bash
storage failover show
```

---

### Q18

What command shows aggregate ownership?

```bash
storage aggregate show
```

---

### Q19

What is ARL?

```text
Aggregate Relocation
```

---

### Q20

Does ARL move data?

```text
No
```

---

### Q21

What moves ownership without moving data?

```text
ARL
```

---

### Q22

What command starts aggregate relocation?

```bash
storage aggregate relocation start
```

---

### Q23

What is a common use of ARL?

```text
ONTAP Upgrade
```

---

### Q24

What command verifies cluster communication?

```bash
cluster ring show
```

---

### Q25

What command tests cluster connectivity?

```bash
cluster ping-cluster -node local
```

---

### Q26

What command displays cluster health?

```bash
system health show
```

---

### Q27

Node healthy but giveback unavailable. First command?

```bash
storage failover show
```

---

### Q28

Cluster network failure primarily impacts?

```text
Quorum
```

---

### Q29

Which LIF carries cluster communication?

```text
Cluster LIF
```

---

### Q30

What ONTAP feature allows storage access during controller failure?

```text
HA Takeover
```

---

# NCDA Memory Dump

```text
2 Nodes = 1 HA Pair

4 Nodes = 2 HA Pairs

8 Nodes = 4 HA Pairs

24 Nodes = 12 HA Pairs

Takeover
========
Partner assumes ownership

Giveback
========
Return ownership

ARL
===
Move ownership
No data movement

Volume Move
===========
Moves actual data

Most Important Commands

cluster show

system node show

storage failover show

storage failover takeover

storage failover giveback

storage aggregate show

cluster ring show

cluster ping-cluster -node local
```
