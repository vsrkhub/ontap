# NetApp NCDA-165 Study Notes
## Compiled from Discussion Topics

---

# NetApp Management Tools

| Current Name | Previous Name | Purpose | When to Use |
|-------------|--------------|----------|------------|
| ONTAP System Manager | OnCommand System Manager | ONTAP Administration | Create volumes, SVMs, LUNs, SMB shares, NFS exports, snapshots, upgrades |
| Active IQ Unified Manager | OnCommand Unified Manager (OCUM) | Monitor ONTAP | Historical performance, IOPS, latency, capacity, alerts |
| Active IQ Digital Advisor | Active IQ | Advisory Analytics | Upgrade Advisor, security, AutoSupport analysis |
| Config Advisor | Config Advisor | Validation | Cabling, networking, health checks, best practices |
| Data Infrastructure Insights (DII) | Cloud Insights / OCI | Full Stack Observability | VMware, Kubernetes, applications, multi-vendor monitoring |
| NetApp Console | BlueXP | Cloud & Hybrid Cloud Management | Backup, DR, Cloud Volumes ONTAP, tiering |
| NetApp Console Agent | BlueXP Connector | Connectivity Bridge | Connect Console to ONTAP, StorageGRID and cloud resources |
| ONTAP Deploy | ONTAP Deploy | ONTAP Select Deployment | Deploy and manage ONTAP Select |
| Lifecycle Planning | BlueXP Economic Efficiency | Lifecycle Analysis | Refresh planning, capacity planning |
| Backup and Recovery | Cloud Backup | Backups | Cloud and hybrid backups |
| Replication | Replication | SnapMirror Management | Replication and DR |
| Disaster Recovery | VMware DR | DR Orchestration | VMware failover/failback |
| Cloud Tiering | Cloud Tiering | FabricPool Management | Tier cold data |
| Volume Caching | FlexCache Service | FlexCache Management | Remote office acceleration |

---

# Tool Memory Trick

```text
System Manager      = Configure
Unified Manager     = Monitor
Digital Advisor     = Advise
Config Advisor      = Validate
DII                 = Observe
NetApp Console      = Manage Cloud
Console Agent       = Connect
FabricPool          = Tier Cold Data
FlexCache           = Cache Hot Data
SnapMirror          = Replicate
Mediator            = Decide
```

---

# FlexCache Deep Dive

## What is FlexCache?

FlexCache creates a cached copy of data closer to users.

Example:

```text
US ONTAP Cluster
----------------
Volume: test1
(Origin)

       ^
       |
       |
FlexCache
       |
       v

Sydney ONTAP Cluster
--------------------
Volume: test1_cache
(Cache)
```

---

## First Read

```text
Sydney User
    |
    v
Sydney FlexCache
    |
Cache Miss
    |
    v
US Origin Volume
```

Data is fetched and cached.

---

## Second Read

```text
Sydney User
    |
    v
Sydney FlexCache
```

Data served locally.

---

## FlexCache vs SnapMirror

### FlexCache

```text
Purpose: Performance
```

- Caches data
- Hot data only
- Reduces latency
- Optimizes WAN traffic

### SnapMirror

```text
Purpose: Disaster Recovery
```

- Maintains copy
- DR and migration
- Recovery solution

---

## FlexCache and DFS

FlexCache does NOT redirect users.

Example:

```text
\\us01\share
```

Still goes to US.

Need:

```text
DFS
DNS GSLB
Aliases
```

to redirect users.

---

# DFS vs DNS

## DNS

DNS answers:

```text
Where is the server?
```

Example:

```text
us01.company.com
   |
   v
10.1.1.50
```

---

## DFS

DFS answers:

```text
Which file server should this user use?
```

Example:

```text
\\company\data\share
```

DFS Referral:

```text
Sydney User -> Sydney FlexCache

US User -> US Origin
```

---

# SnapMirror Evolution

## Traditional SnapMirror

```text
ONTAP --> ONTAP
```

Purpose:

```text
DR
Replication
```

---

## SnapVault

Purpose:

```text
Long-Term Retention
```

Examples:

```text
7 Daily
12 Weekly
36 Monthly
```

---

## XDP

Modern framework supporting:

```text
Mirror
Vault
Mirror-Vault
```

---

## SnapMirror Active Sync

Formerly:

```text
SMBC
```

Now:

```text
SnapMirror Active Sync
```

Purpose:

```text
SAN Business Continuity
```

Supports:

- FC
- iSCSI
- NVMe

Uses:

```text
Mediator
```

---

## SnapMirror Cloud

Purpose:

```text
ONTAP Volume
      |
      v
AWS S3

Azure Blob

Google Cloud

StorageGRID
```

Long-term backups and tape replacement.

---

## SnapMirror S3

Purpose:

```text
Bucket Protection
```

Example:

```text
ONTAP S3
      |
      v
AWS S3
StorageGRID
ONTAP S3
```

---

# SnapMirror Types

## Mirror

Purpose:

```text
DR
```

Policy:

```text
MirrorAllSnapshots
```

---

## Vault

Purpose:

```text
Backup
```

Retention focused.

---

## Mirror-Vault

Purpose:

```text
DR + Backup
```

Most common enterprise design.

---

# SnapMirror Labels

Labels determine what gets retained.

Source:

```text
hourly
daily
weekly
monthly
```

Destination Policy:

```text
daily   retain 30
weekly  retain 12
monthly retain 36
```

Matching occurs using labels.

---

# Mediator

## Purpose

Prevents:

```text
Split Brain
```

Acts as:

```text
Referee
```

---

## SnapMirror Active Sync

```text
Site A
   |
Mediator
   |
Site B
```

Decides:

```text
Which side owns LUNs
```

---

## MetroCluster

```text
Site A
   |
Mediator
   |
Site B
```

Decides:

```text
Which site survives
```

after failure.

---

# SnapMirror Active Sync vs MetroCluster

| Feature | SnapMirror Active Sync | MetroCluster |
|----------|-----------------------|-------------|
| Focus | SAN LUNs | Entire Site |
| Purpose | Business Continuity | Disaster Recovery |
| Mediator Role | Quorum | Witness |
| Protocols | FC/iSCSI/NVMe | NAS + SAN |
| Granularity | LUN | Site |

---

# FabricPool and Cloud Tiering

## FabricPool

Actual ONTAP Technology.

```text
SSD
 |
 v
Object Storage
```

Moves cold blocks.

---

## Cloud Tiering

Management Service.

```text
NetApp Console
       |
       v
FabricPool
```

---

# FlexCache and Volume Caching

## FlexCache

Actual ONTAP Technology.

## Volume Caching

NetApp Console Service.

```text
Volume Caching
       |
       v
FlexCache
```

---

# ONTAP 9.18.1 Highlights

## Security

### Cluster Network Encryption

Encrypts:

```text
Node <-> Node
```

cluster traffic.

---

### HA Traffic Encryption

Encrypts:

```text
HA Partner <-> HA Partner
```

traffic.

---

### Post Quantum Crypto

Supports:

```text
PQC SSL Algorithms
```

---

## SAN

### NVMe Copy Offload

Copy operation executed by storage instead of host.

Benefits:

```text
Lower CPU consumption
```

---

## Object Storage

### S3 Snapshot Restore

Supports:

```text
Single Object
Multiple Objects
Entire Bucket
```

restore.

---

### FlexCache + S3

Can now:

```text
Create NAS Buckets
on FlexCache Volumes
```

---

## Security

### ARP/AI for FlexGroup

Machine Learning based:

```text
Ransomware Detection
```

for FlexGroup.

---

# AFX Platform

## What is AFX?

New ultra-high-performance NetApp all-flash platform.

Supports:

```text
NAS
SAN
AI
Analytics
Databases
```

---

## Platform Comparison

| Platform | NAS | SAN |
|-----------|-----|-----|
| FAS | Yes | Yes |
| AFF | Yes | Yes |
| ASA | No (SAN-focused) | Yes |
| AFX | Yes | Yes |

---

# Disaggregated Storage

Traditional:

```text
Controller + SSDs
```

Same chassis.

Disaggregated:

```text
Controller Layer
      |
      v
Shared Storage Pool
```

Benefits:

```text
Scale Storage Independently
Scale Compute Independently
```

---

# FlexCache Health Troubleshooting

## Verify Volume

```bash
volume flexcache show
```

---

## Verify Cluster Peering

```bash
cluster peer show
```

---

## Verify SVM Peering

```bash
vserver peer show
```

---

## Verify Intercluster LIFs

```bash
network interface show -role intercluster
```

---

## Cluster Peer Ping

```bash
cluster peer ping -originating-node node1
```

Purpose:

```text
Validate Cluster Peering Connectivity
```

---

## Network Ping

```bash
network ping \
-lif ic-lif1 \
-destination <IP>
```

Purpose:

```text
Basic Network Reachability
```

---

# Most Important NCDA Exam Traps

| Question | Correct Answer |
|-----------|---------------|
| Upgrade Readiness | Digital Advisor |
| Historical Latency | Unified Manager |
| Create SMB Share | System Manager |
| Multi-Vendor Monitoring | DII |
| Tier Cold Data | FabricPool |
| Manage FabricPool | Cloud Tiering |
| Branch Office Performance | FlexCache |
| DR Replication | SnapMirror |
| SAN Business Continuity | SnapMirror Active Sync |
| Cloud Volumes ONTAP Management | NetApp Console |
| Validate Cabling | Config Advisor |
| ONTAP Select Deployment | ONTAP Deploy |

---

# Final Memory Sheet

```text
System Manager      = Configure

Unified Manager     = Monitor

Digital Advisor     = Recommend

Config Advisor      = Validate

DII                 = Observe

NetApp Console      = Manage

Console Agent       = Connect

FlexCache           = Cache

FabricPool          = Tier

SnapMirror          = Replicate

SnapVault           = Retain

Mirror-Vault        = DR + Backup

Mediator            = Decide

MetroCluster        = Site DR

SnapMirror Active Sync = SAN Business Continuity

DFS                 = Referrals

DNS                 = Resolution
```
