# NCDA Deep Dive
# HA Takeover and Giveback in Multi-Node Clusters

This is one of the highest-value NCDA topics because it combines:

```text
HA
Cluster Architecture
Aggregate Ownership
Quorum
ARL
Maintenance
ONTAP Upgrades
Failure Recovery
Troubleshooting
```

Many exam questions are based on a 4-node or 8-node cluster where candidates incorrectly assume that any node can take over another node.

---

# Multi-Node Cluster Architecture

## 4 Node Example

```text
Cluster

HA Pair 1
---------
NodeA <----> NodeB

HA Pair 2
---------
NodeC <----> NodeD
```

Important:

```text
NodeA only takes over NodeB
NodeB only takes over NodeA

NodeC only takes over NodeD
NodeD only takes over NodeC
```

---

## 8 Node Example

```text
NodeA <-> NodeB
NodeC <-> NodeD
NodeE <-> NodeF
NodeG <-> NodeH
```

Each node has exactly:

```text
One HA Partner
```

---

# Biggest NCDA Exam Trap

Question:

```text
Can NodeA take over NodeD?
```

Answer:

```text
NO
```

Takeover occurs only between HA partners.

---

# Aggregate Ownership

Before Failure:

```text
NodeA owns:
aggr_A_root
aggr_A_data

NodeB owns:
aggr_B_root
aggr_B_data
```

---

## After NodeA Takeover

```text
NodeB owns:

aggr_A_root
aggr_A_data
aggr_B_root
aggr_B_data
```

---

Important:

```text
Aggregate ownership changes.

Data is NOT copied.
```

---

# What Happens During Takeover?

## Step 1

Node detects partner failure.

Causes:

```text
Power loss
Panic
Hardware failure
Manual takeover
Upgrade
Maintenance
```

---

## Step 2

Partner initiates takeover.

```text
storage failover takeover
```

---

## Step 3

Aggregate ownership transfers.

---

## Step 4

LIFs migrate if needed.

---

## Step 5

Clients continue serving I/O.

---

# What Happens During Giveback?

Once failed node returns:

```text
NodeA Rejoins Cluster
```

---

## Giveback Process

### Before

```text
NodeB owns:

aggr_A
aggr_B
```

---

### After Giveback

```text
NodeA owns aggr_A

NodeB owns aggr_B
```

---

# Important Terms

## CFO

Controller Failover

Protects:

```text
Root Aggregates
```

---

## SFO

Storage Failover

Protects:

```text
Data Aggregates
```

---

NCDA normally focuses on:

```text
Storage Failover
```

---

# Commands Every NCDA Candidate Must Know

---

# 1. Check HA Status

```bash
storage failover show
```

Example:

```text
Takeover Enabled: true
Giveback Possible: true
```

---

Used For:

```text
Health Check
Maintenance
Upgrade Planning
Troubleshooting
```

---

# Common Options

```bash
storage failover show -fields *
```

Display detailed fields.

---

```bash
storage failover show -node node1
```

Specific node.

---

# 2. Perform Manual Takeover

```bash
storage failover takeover
```

---

Specific Partner:

```bash
storage failover takeover -ofnode nodeA
```

---

# Common Options

```bash
storage failover takeover -ofnode nodeA
```

Take over NodeA.

---

```bash
storage failover takeover -bypass-optimization true
```

Used in special situations.

Exam awareness only.

---

# 3. Perform Giveback

```bash
storage failover giveback
```

---

Specific Node

```bash
storage failover giveback -ofnode nodeA
```

---

# 4. Check Giveback Status

```bash
storage failover show-giveback
```

---

Displays:

```text
Progress
Restrictions
Blockers
```

---

# 5. Monitor Giveback

```bash
storage failover show
```

---

Look For:

```text
Giveback Possible
Giveback Pending
```

---

# 6. Verify Aggregate Ownership

```bash
storage aggregate show
```

---

Key Fields:

```text
Owner
Node
State
```

---

# 7. Verify Aggregate Relocation

```bash
storage aggregate relocation show
```

---

# 8. Start Aggregate Relocation

```bash
storage aggregate relocation start
```

---

Specific Aggregate

```bash
storage aggregate relocation start \
-aggregate aggr1
```

---

# 9. Check Node Health

```bash
system node show
```

---

# 10. Check Cluster Health

```bash
system health show
```

---

# 11. Investigate Events

```bash
event log show
```

---

# Planned Maintenance Scenario

## Goal

Upgrade NodeA

---

Step 1

Verify Cluster

```bash
cluster show
```

---

Step 2

Verify HA

```bash
storage failover show
```

---

Step 3

Verify Health

```bash
system health show
```

---

Step 4

Perform Takeover

```bash
storage failover takeover -ofnode nodeA
```

---

Result

```text
NodeB owns NodeA aggregates
```

---

Step 5

Upgrade NodeA

---

Step 6

Verify NodeA Online

```bash
system node show
```

---

Step 7

Perform Giveback

```bash
storage failover giveback -ofnode nodeA
```

---

Step 8

Verify Ownership

```bash
storage aggregate show
```

---

# Troubleshooting Scenario 1

## Takeover Not Possible

Command

```bash
storage failover show
```

---

Possible Causes

```text
Partner unhealthy
HA disabled
Cluster communication issue
Node booting
```

---

Checks

```bash
system health show

event log show

cluster ring show
```

---

# Troubleshooting Scenario 2

## Giveback Fails

Command

```bash
storage failover show-giveback
```

---

Possible Causes

```text
Aggregate offline

Network issue

Partner unhealthy

Node not booted

Version mismatch
```

---

Commands

```bash
storage aggregate show

system health show

event log show
```

---

# Troubleshooting Scenario 3

## Aggregates Not Returned

Verify

```bash
storage aggregate show
```

---

Find

```text
Current Owner
Home Owner
```

---

Common Issue

```text
Giveback incomplete
```

---

# Troubleshooting Scenario 4

## Unexpected Takeover

Check

```bash
event log show
```

---

Look For

```text
Panic
Power Failure
Watchdog Reset
Hardware Fault
```

---

# Troubleshooting Scenario 5

## Upgrade Stuck During ARL

Check

```bash
storage aggregate relocation show
```

---

Check

```bash
job show
```

---

Check

```bash
event log show
```

---

# Takeover vs ARL vs Volume Move

## Takeover

Moves:

```text
Aggregate Ownership
```

Used For:

```text
Failure
Maintenance
```

---

## ARL

Moves:

```text
Aggregate Ownership
```

Used For:

```text
Controller Upgrades
Maintenance
```

---

## Volume Move

Moves:

```text
Actual Volume Data
```

Used For:

```text
Rebalancing
Capacity Management
```

---

# 30 Practice Questions

### Q1

Can NodeA take over NodeC in a 4-node cluster?

**Answer**

```text
No
```

---

### Q2

Who can take over NodeA?

```text
Only NodeB
```

---

### Q3

What command shows HA status?

```bash
storage failover show
```

---

### Q4

What command performs takeover?

```bash
storage failover takeover
```

---

### Q5

What command performs giveback?

```bash
storage failover giveback
```

---

### Q6

What changes during takeover?

```text
Aggregate ownership
```

---

### Q7

Does takeover move data?

```text
No
```

---

### Q8

What technology protects writes during takeover?

```text
NVRAM Mirroring
```

---

### Q9

What command shows giveback status?

```bash
storage failover show-giveback
```

---

### Q10

What command verifies aggregate owners?

```bash
storage aggregate show
```

---

### Q11

Can a node have more than one HA partner?

```text
No
```

---

### Q12

How many HA partners does NodeA have?

```text
One
```

---

### Q13

A node crashes unexpectedly. What occurs automatically?

```text
Takeover
```

---

### Q14

What is returned during giveback?

```text
Aggregate ownership
```

---

### Q15

What command verifies cluster health?

```bash
system health show
```

---

### Q16

What command checks node status?

```bash
system node show
```

---

### Q17

What should be checked before planned takeover?

```bash
storage failover show
```

---

### Q18

What command helps investigate unexpected takeover?

```bash
event log show
```

---

### Q19

Can takeover occur across HA pairs?

```text
No
```

---

### Q20

Does ARL move data?

```text
No
```

---

### Q21

What command starts ARL?

```bash
storage aggregate relocation start
```

---

### Q22

What command checks ARL?

```bash
storage aggregate relocation show
```

---

### Q23

What command checks cluster communication?

```bash
cluster ring show
```

---

### Q24

Most common reason for giveback failure?

```text
Partner unhealthy
```

---

### Q25

What object is owned during takeover?

```text
Aggregate
```

---

### Q26

What component typically migrates along with takeover events?

```text
LIFs
```

---

### Q27

2-node cluster contains how many HA pairs?

```text
1
```

---

### Q28

8-node cluster contains how many HA pairs?

```text
4
```

---

### Q29

24-node cluster contains how many HA pairs?

```text
12
```

---

### Q30

Most important command before maintenance?

```bash
storage failover show
```

---

# NCDA Memory Dump

```text
Takeover
========
Partner assumes aggregate ownership

Giveback
========
Ownership returns home

NodeA ↔ NodeB
NodeC ↔ NodeD

Takeover never crosses HA pairs

Important Commands
==================

storage failover show

storage failover takeover

storage failover giveback

storage failover show-giveback

storage aggregate show

storage aggregate relocation show

storage aggregate relocation start

system node show

system health show

event log show

cluster ring show

cluster ping-cluster -node local
```
