# ONTAP Management Tools
# On-Prem, Cloud, ONTAP Select, FabricPool, Cloud Volumes ONTAP

One of the most common NCDA interview questions is:

```text
What management tools are used for ONTAP?
What tools are used for cloud?
What tools are used for FabricPool?
What manages ONTAP Select?
```

The answer depends on:

```text
Physical ONTAP
Cloud ONTAP
ONTAP Select
StorageGRID
FabricPool
Cloud Insights
Active IQ
```

---

# Complete ONTAP Ecosystem

```text
               Active IQ
                    |
                    |
                    V

         +-------------------+
         | ONTAP Clusters    |
         +-------------------+

            /           \
           /             \

 On-Prem ONTAP      Cloud ONTAP
 AFF/FAS/ASA          CVO

           \             /
            \           /

             BlueXP

```

---

# Primary Management Tools

## System Manager

Built into every ONTAP cluster.

Access

```text
https://cluster-mgmt-ip
```

Used for:

```text
Volumes
Aggregates
SVMs
Snapshots
Networking
Performance
Users
LIFs
Protocols
```

---

# Example Use Cases

```text
Create Volumes
View Performance
Check Capacity
Configure NFS
Configure SMB
```

---

# NCDA Fact

System Manager is:

```text
Primary GUI
```

for ONTAP.

---

# ONTAP CLI

Still heavily used by administrators.

Access

```bash
ssh admin@cluster
```

---

Used for:

```text
Troubleshooting

Performance

Automation

Advanced Features

Upgrades

Networking
```

---

# NCDA Reality

Many advanced operations still use:

```text
CLI
```

before GUI.

---

# Active IQ Digital Advisor

Previously:

```text
Active IQ
```

Purpose:

```text
Health Monitoring
Risk Detection
Upgrade Planning
Firmware Guidance
Support Cases
```

---

Administrators use it to identify:

```text
Known Bugs
Best Practices
Capacity Risks
End Of Support Hardware
```

---

# Active IQ Unified Manager (AIQUM)

Purpose:

```text
Monitoring
Capacity Planning
Historical Reporting
Performance Analytics
```

---

Provides:

```text
Months/Years of History

Performance Trends

Forecasting

Alerting
```

---

# NCDA Favorite Question

Which tool provides:

```text
Historical Performance Data?
```

Answer:

```text
Active IQ Unified Manager
```

---

# BlueXP

Previously:

```text
Cloud Manager
```

This is the cloud management platform.

---

Used to manage:

```text
Cloud Volumes ONTAP

FabricPool

StorageGRID

Backup

Tiering

Cloud Volumes Services
```

---

Architecture

```text
BlueXP

      |
      +---- On-Prem ONTAP

      |
      +---- Cloud Volumes ONTAP

      |
      +---- StorageGRID

      |
      +---- AWS

      |
      +---- Azure

      |
      +---- GCP
```

---

# Why BlueXP Is Important

Single pane of glass for:

```text
Hybrid Cloud
```

---

# Cloud Volumes ONTAP (CVO)

Cloud version of ONTAP.

Runs in:

```text
AWS

Azure

Google Cloud
```

---

Managed Using:

```text
BlueXP
```

and

```text
System Manager
```

---

# Example

AWS

```text
Cloud Volumes ONTAP

      ↓

BlueXP
```

---

# ONTAP Select

Very common exam topic.

---

# What Is ONTAP Select?

Software Defined ONTAP.

Runs on:

```text
VMware ESXi

KVM
```

instead of NetApp hardware.

---

Architecture

```text
ESXi

  |
  +---- ONTAP Select VM
```

---

Managed Using

```text
System Manager

CLI

BlueXP
```

---

Deployment Tool

```text
ONTAP Deploy
```

---

# NCDA Question

### What deploys ONTAP Select?

Answer

```text
ONTAP Deploy
```

---

# ONTAP Deploy

Specific purpose:

```text
Deploy

Monitor

License

Manage

ONTAP Select
```

---

Most important ONTAP Select tool.

---

# StorageGRID

Object Storage Platform.

Frequently used with:

```text
FabricPool
```

---

Managed Using:

```text
StorageGRID Grid Manager
```

---

Provides:

```text
S3 Compatible Storage
```

---

# FabricPool

Most misunderstood topic.

---

# Is FabricPool Cloud?

Answer:

```text
Not necessarily.
```

---

FabricPool is:

```text
Tiering Technology
```

---

It can tier data to:

```text
StorageGRID

AWS S3

Azure Blob

Google Cloud Storage
```

---

# Architecture

```text
Hot Data

ONTAP Aggregate
     |
     |
Cold Blocks
     |
     V

Object Store
```

---

# On-Prem FabricPool

```text
AFF/FAS

     |
     |
StorageGRID
```

No public cloud.

Still FabricPool.

---

# Cloud FabricPool

```text
AFF

     |
     |
AWS S3
```

or

```text
Azure Blob
```

or

```text
Google Cloud Storage
```

---

# FabricPool Managed Using

---

ONTAP CLI

```bash
storage aggregate object-store show
```

---

System Manager

```text
Storage Tiers
```

---

BlueXP

```text
Tiering Service
```

---

# Cloud Insights

Enterprise observability tool.

Think:

```text
Storage Monitoring
+
Application Monitoring
```

---

Provides:

```text
VM Performance

Storage Performance

Container Monitoring

Cloud Monitoring
```

---

Used for:

```text
Performance Troubleshooting
```

---

# SnapCenter

Application-consistent protection.

Used for:

```text
SQL

Oracle

Exchange

VMware
```

---

Managed via:

```text
SnapCenter Server
```

---

# Typical Tool Matrix

## Physical AFF/FAS/ASA

Managed By

```text
System Manager

CLI

Active IQ

AIQUM
```

---

## Cloud Volumes ONTAP

Managed By

```text
BlueXP

System Manager

CLI
```

---

## ONTAP Select

Managed By

```text
ONTAP Deploy

System Manager

CLI

BlueXP
```

---

## StorageGRID

Managed By

```text
Grid Manager
```

---

## FabricPool

Managed By

```text
System Manager

CLI

BlueXP
```

---

## Monitoring

Managed By

```text
AIQUM

Cloud Insights

Active IQ
```

---

# Common NCDA Questions

### Q1

What is the primary ONTAP GUI?

```text
System Manager
```

---

### Q2

What is the primary ONTAP cloud management platform?

```text
BlueXP
```

---

### Q3

What deploys ONTAP Select?

```text
ONTAP Deploy
```

---

### Q4

What manages Cloud Volumes ONTAP?

```text
BlueXP
```

---

### Q5

What tool provides historical performance reporting?

```text
Active IQ Unified Manager
```

---

### Q6

What tool provides support and risk insights?

```text
Active IQ Digital Advisor
```

---

### Q7

Is FabricPool cloud-only?

```text
No
```

---

### Q8

Can FabricPool tier to StorageGRID?

```text
Yes
```

---

### Q9

What manages StorageGRID?

```text
Grid Manager
```

---

### Q10

What is the most common tool for hybrid cloud ONTAP environments?

```text
BlueXP
```

---

# Memory Dump

```text
System Manager
==============
Primary ONTAP GUI

CLI
===
Advanced Administration

BlueXP
======
Cloud Management

Cloud Volumes ONTAP

FabricPool

StorageGRID

Hybrid Cloud


ONTAP Deploy
============
Deploys ONTAP Select


AIQUM
=====
Historical Performance
Monitoring


Active IQ
=========
Health
Risk
Upgrades


Cloud Insights
==============
Performance Monitoring


StorageGRID
===========
Object Storage


FabricPool
==========
Tiering Technology

Targets:

StorageGRID
AWS S3
Azure Blob
GCS


ONTAP Select
============
Software Defined ONTAP

Runs On:

VMware
KVM

Managed By:

ONTAP Deploy
System Manager
CLI
BlueXP
```
