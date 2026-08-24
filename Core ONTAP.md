# NCDA (NS0-165) Study Guide
# Domain 2 - Core ONTAP

**Objective:** Master ONTAP cluster administration, Storage Virtual Machines (SVMs), administrative roles, quorum, high availability concepts, system management, and core operational workflows.

---

# Chapter 1 - Core ONTAP Architecture

Core ONTAP is the control plane of the storage environment.

It includes:

```text
Cluster
Nodes
SVMs
LIFs
Volumes
Aggregates
Protocols
Users
Monitoring
```

Everything an administrator manages is ultimately controlled through ONTAP.

---

# Chapter 2 - ONTAP Management Interfaces

---

## System Manager

GUI-based administration.

Capabilities:

```text
Monitoring
Volume Management
LIF Management
Networking
Security
SnapMirror
Performance
Firmware Upgrades
```

---

### Advantages

- Easy to use
- Modern interface
- Good visual monitoring
- Rapid troubleshooting

---

### Exam Tip

System Manager can perform most cluster administration tasks.

---

## CLI

Preferred by advanced administrators.

Especially useful for:

```text
Troubleshooting
Automation
Advanced configuration
Bulk operations
```

---

### Privilege Levels

```bash
set -privilege admin
```

Default level.

---

```bash
set -privilege advanced
```

Advanced commands.

---

```bash
set -privilege diagnostic
```

Restricted level.

---

## Common CLI Commands

```bash
version
```

Display ONTAP version.

---

```bash
cluster show
```

Display cluster members.

---

```bash
system node show
```

Display node information.

---

```bash
event log show
```

Display event logs.

---

```bash
system health show
```

Display health status.

---

# Chapter 3 - Cluster Administration

---

# Cluster Identity

Cluster identity includes:

```text
Cluster Name
Management IP
Cluster UUID
```

---

## Commands

```bash
cluster identity show
```

---

Example Output

```text
Cluster Name : PROD_CLUSTER
Cluster UUID : xxxxxxxx
```

---

# Cluster Nodes

Each node participates in:

```text
Storage
Networking
Failover
Management
```

---

## Commands

```bash
cluster show
```

---

```bash
system node show
```

---

## Node Health

Always verify node health.

```bash
system health show
```

---

```bash
system health alert show
```

---

### Exam Scenario

Customer reports random failures.

First action:

```bash
system health show
```

---

# Cluster Time

Critical for:

```text
Kerberos
AD Authentication
Certificates
SnapMirror
```

---

## Commands

```bash
cluster time-service ntp server show
```

---

```bash
cluster date show
```

---

### Exam Trap

Incorrect NTP settings can cause:

```text
AD login failures
Kerberos failures
Certificate issues
```

---

# Chapter 4 - Storage Virtual Machines (SVMs)

---

# What is an SVM?

Storage Virtual Machine.

Provides a logical storage boundary.

Think:

```text
Virtual Storage Server
```

---

## SVM Components

```text
Volumes
LIFs
Shares
Exports
Protocols
Users
Security
```

---

## Example

```text
Cluster
|
+--SVM_FINANCE
|
+--SVM_HR
|
+--SVM_ENGINEERING
```

---

## Benefits

### Multi-Tenancy

Different departments isolated.

---

### Security

Separate authentication.

---

### Administrative Delegation

SVM admins manage own environments.

---

## Commands

```bash
vserver show
```

---

```bash
vserver create
```

---

```bash
vserver modify
```

---

```bash
vserver delete
```

---

# SVM State

Possible states:

```text
Running
Stopped
Starting
```

---

## Commands

```bash
vserver show
```

---

### Exam Question

Users cannot access data.

One possibility:

```text
SVM stopped
```

---

# Data LIF Ownership

Data LIFs belong to:

```text
SVM
```

Not the cluster.

---

## Commands

```bash
network interface show
```

---

# Supported Protocols Per SVM

An SVM may support:

```text
NFS
SMB
iSCSI
FC
NVMe
S3
```

---

### Exam Scenario

Company requires both NFS and SMB.

Solution:

```text
Single multiprotocol SVM
```

---

# Chapter 5 - Administrative Roles and RBAC

---

# Role-Based Access Control

RBAC controls administrative permissions.

---

## Built-In Roles

```text
admin
readonly
backup
vsadmin
vsadmin-volume
```

---

# Cluster Administrator

Controls:

```text
Nodes
Aggregates
Networking
HA
Upgrades
Licensing
```

---

# SVM Administrator

Controls:

```text
Volumes
Shares
Exports
Users
Protocols
```

---

Cannot manage:

```text
Nodes
Aggregates
Cluster Networking
HA
```

---

## Commands

```bash
security login show
```

---

```bash
security login role show
```

---

```bash
security login create
```

---

### Exam Question

User must administer only CIFS shares.

Answer:

```text
SVM Administrator
```

---

# Chapter 6 - Cluster Quorum

---

# What is Quorum?

Quorum ensures cluster consistency.

A majority of nodes must communicate.

---

## Purpose

Protects against:

```text
Split Brain
Cluster Corruption
Inconsistent Metadata
```

---

## Example

4 Node Cluster

```text
Node1
Node2
Node3
Node4
```

If majority remain reachable:

```text
Quorum maintained
```

---

# Commands

```bash
cluster ring show
```

---

```bash
cluster show
```

---

### Exam Question

Why is quorum important?

Answer:

```text
Prevent split-brain conditions
```

---

# Chapter 7 - Event Management System (EMS)

---

## EMS

Event Management System.

Records:

```text
Failures
Warnings
System Events
Network Issues
Storage Errors
```

---

## Commands

```bash
event log show
```

---

```bash
event catalog show
```

---

## Filtering

```bash
event log show -severity ERROR
```

---

### Troubleshooting

Many exam troubleshooting questions begin with:

```bash
event log show
```

---

# Chapter 8 - Job Management

---

## ONTAP Jobs

Background processes.

Examples:

```text
Volume Move
SnapMirror Update
Efficiency Scan
```

---

## Commands

```bash
job show
```

---

```bash
job history show
```

---

### Question

Volume move appears stuck.

Check:

```bash
job show
```

---

# Chapter 9 - Software Packages & Licenses

---

## Packages

Features often depend on licensing.

---

### Verify Licenses

```bash
system license show
```

---

### Verify Installed Packages

```bash
system license package show
```

---

### Exam Scenario

Feature unavailable.

Check:

```text
Licensing
```

before troubleshooting.

---

# Chapter 10 - ONTAP Health Monitoring

---

## Health Monitoring Categories

```text
Node
Storage
Network
Cluster
Hardware
MetroCluster
```

---

## Commands

```bash
system health show
```

---

```bash
system health alert show
```

---

## Common Alert Types

```text
Port Failure
Disk Failure
Node Failure
Shelf Failure
Power Issue
```

---

# Troubleshooting Scenarios

---

## Scenario 1

Users lose access to all shares.

Checks:

```bash
vserver show
network interface show
```

Possible root cause:

```text
SVM stopped
```

---

## Scenario 2

Administrator cannot log in.

Checks:

```bash
security login show
```

Possible causes:

```text
Account Locked
RBAC Restriction
Authentication Failure
```

---

## Scenario 3

SnapMirror suddenly failing.

Checks:

```bash
event log show
cluster date show
```

Possible issue:

```text
Time synchronization
```

---

## Scenario 4

Cluster reports unhealthy.

Checks:

```bash
system health show
system health alert show
```

---

## Scenario 5

Volume move never completes.

Checks:

```bash
job show
job history show
```

---

# Exam Traps

---

## Trap #1

SVM Admin cannot manage aggregates.

Correct.

Aggregates belong to cluster scope.

---

## Trap #2

Data LIFs belong to cluster.

Incorrect.

Data LIFs belong to SVM.

---

## Trap #3

Quorum protects storage performance.

Incorrect.

Quorum protects cluster consistency.

---

## Trap #4

System Manager replaces all CLI.

Incorrect.

Some operations still require CLI.

---

## Trap #5

EMS is only for hardware events.

Incorrect.

EMS logs software and configuration events too.

---

# 30 Practice Questions

---

### 1.

What command displays all SVMs?

**Answer**

```bash
vserver show
```

---

### 2.

What command displays cluster identity?

```bash
cluster identity show
```

---

### 3.

What command shows node information?

```bash
system node show
```

---

### 4.

Which role can administer aggregates?

**Answer:** Cluster Administrator

---

### 5.

Which object provides tenant isolation?

**Answer:** SVM

---

### 6.

Can an SVM own Data LIFs?

**Answer:** Yes

---

### 7.

Can an SVM own Aggregates?

**Answer:** No

---

### 8.

What command displays cluster health?

```bash
system health show
```

---

### 9.

What command displays health alerts?

```bash
system health alert show
```

---

### 10.

Which command displays EMS events?

```bash
event log show
```

---

### 11.

What does EMS stand for?

**Answer:** Event Management System

---

### 12.

Which object contains exports and shares?

**Answer:** SVM

---

### 13.

What is the purpose of quorum?

**Answer:** Prevent split-brain conditions

---

### 14.

Can one cluster host multiple SVMs?

**Answer:** Yes

---

### 15.

Which privilege level is default?

**Answer:** admin

---

### 16.

Which command enters advanced mode?

```bash
set -privilege advanced
```

---

### 17.

What command shows running jobs?

```bash
job show
```

---

### 18.

What command shows job history?

```bash
job history show
```

---

### 19.

Can System Manager perform SnapMirror operations?

**Answer:** Yes

---

### 20.

Which command shows system version?

```bash
version
```

---

### 21.

What command verifies licenses?

```bash
system license show
```

---

### 22.

What command shows login accounts?

```bash
security login show
```

---

### 23.

What command shows RBAC roles?

```bash
security login role show
```

---

### 24.

Which role should manage only an SVM?

**Answer:** vsadmin

---

### 25.

Who manages nodes and networking?

**Answer:** Cluster Administrator

---

### 26.

Can a single SVM support SMB and NFS simultaneously?

**Answer:** Yes

---

### 27.

Which service is critical for Kerberos authentication?

**Answer:** NTP/Time Synchronization

---

### 28.

What command displays cluster time?

```bash
cluster date show
```

---

### 29.

Where should you start if ONTAP reports unusual behavior?

```bash
event log show
```

---

### 30.

What command verifies NTP servers?

```bash
cluster time-service ntp server show
```

---

# Exam Cram Sheet

Memorize:

```text
SVM = Virtual Storage Server

Cluster Admin = Cluster Scope

SVM Admin = Tenant Scope

Data LIF = SVM Object

Quorum = Prevent Split Brain

EMS = Event Management System

System Health = First Troubleshooting Step

event log show = First Investigation Command

job show = Background Jobs

NTP = Required for AD/Kerberos Stability
```

**Next guide:** `03-ONTAP-Storage.md`
