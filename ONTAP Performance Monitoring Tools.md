# ONTAP Performance Monitoring Tools
# NCDA / Real-World Storage Administrator Guide

A common NCDA question is:

```text
Which tool should I use to monitor ONTAP performance?
```

The answer depends on:

```text
Cluster
Node
Volume
LUN
Protocol
Historical Analysis
Real-Time Troubleshooting
Capacity Planning
```

---

# ONTAP Performance Monitoring Tools

## Native ONTAP Tools

```text
System Manager

CLI Statistics

QoS Statistics

EMS

Active IQ Unified Manager

Active IQ Digital Advisor

Performance Archive Logs
```

---

# Third Party Tools

```text
Grafana

Splunk

SolarWinds

SCOM

Cloud Insights

Dynatrace
```

---

# 1. ONTAP System Manager

Most commonly used today.

---

## Features

```text
Volume Latency

Node Utilization

Aggregate Performance

Protocol Performance

Network Throughput

IOPS

Capacity

Alerts
```

---

## System Manager Views

### Cluster Dashboard

Shows:

```text
Health

Capacity

IOPS

Throughput

Latency
```

---

### Performance View

Shows:

```text
Read IOPS

Write IOPS

Response Time

CPU Usage

Network Usage
```

---

## Use Cases

```text
Quick Health Check

Daily Monitoring

Operations Teams
```

---

# 2. CLI Performance Monitoring

Most engineers use CLI during troubleshooting.

---

# Real-Time Statistics

Command:

```bash
statistics show
```

---

Shows:

```text
Counters
Performance Objects
Metrics
```

---

# Discover Available Objects

```bash
statistics catalog object show
```

---

Examples

```text
volume

aggregate

wafl

nfs

cifs

lun

fcp

disk
```

---

# View Volume Statistics

```bash
statistics show -object volume
```

---

# View NFS Statistics

```bash
statistics show -object nfs
```

---

# View CIFS Statistics

```bash
statistics show -object cifs
```

---

# View WAFL Statistics

```bash
statistics show -object wafl
```

---

# Performance Collection

Start

```bash
statistics start
```

---

Stop

```bash
statistics stop
```

---

Display

```bash
statistics show
```

---

# 3. QoS Statistics

One of the most important troubleshooting tools.

---

# Volume Latency

```bash
qos statistics volume latency show
```

---

Example

```text
Volume

Latency

IOPS
```

---

# LUN Latency

```bash
qos statistics lun latency show
```

---

# Workload Latency

```bash
qos statistics workload performance show
```

---

# Why Important?

Many customer cases start with:

```text
Application Slow
```

First command:

```bash
qos statistics volume latency show
```

---

# Latency Categories

Example:

```text
Network

Cluster

Data

Disk

CPU
```

---

# Exam Question

### Q

Which command shows ONTAP latency?

```bash
qos statistics volume latency show
```

---

# 4. Statistics Objects

You should know these.

---

# WAFL

```bash
statistics show -object wafl
```

Shows:

```text
Consistency Points

Cache Activity

File System Activity
```

---

# Volume

```bash
statistics show -object volume
```

Shows:

```text
IOPS

Throughput

Latency
```

---

# Aggregate

```bash
statistics show -object aggregate
```

Shows:

```text
Aggregate Utilization

Reads

Writes
```

---

# Disk

```bash
statistics show -object disk
```

Shows:

```text
Disk Busy Time

Queue Depth

Response Time
```

---

# NFS

```bash
statistics show -object nfs
```

Shows:

```text
NFS Operations

Read Counts

Write Counts
```

---

# CIFS

```bash
statistics show -object cifs
```

Shows:

```text
SMB Operations

Session Activity
```

---

# SAN

## LUN

```bash
statistics show -object lun
```

---

## FCP

```bash
statistics show -object fcp
```

---

## iSCSI

```bash
statistics show -object iscsi
```

---

# 5. Active IQ Unified Manager (AIQUM)

Enterprise monitoring platform.

---

# Purpose

Provides:

```text
Historical Monitoring

Predictive Alerts

Capacity Forecasting

Performance Trending

Root Cause Analysis
```

---

# Stores Historical Data

Unlike CLI.

CLI =

```text
Current
Short-Term
```

---

AIQUM =

```text
Weeks

Months

Years
```

---

# Typical Dashboards

```text
Volume Performance

LUN Performance

SVM Performance

Node Performance

Latency Analysis
```

---

# Favorite NCDA Question

### Which tool provides long-term performance history?

Answer:

```text
Active IQ Unified Manager
```

---

# 6. Active IQ Digital Advisor

Previously:

```text
Active IQ
```

---

Purpose

```text
Support Insights

Upgrade Advice

Risk Detection

Health Monitoring
```

---

Not primarily a performance tool.

---

Helps identify:

```text
Configuration Risks

Firmware Issues

Known Bugs
```

---

# 7. EMS (Event Management System)

---

Performance-related events often appear here.

Check:

```bash
event log show
```

---

Examples

```text
Latency Events

Network Issues

Disk Errors

Controller Warnings
```

---

# 8. Performance Archive Logs

Advanced.

---

Command

```bash
statistics archive show
```

---

Purpose

```text
Historical Collection
```

---

Can be analyzed later.

---

# Common Real-World Workflow

---

## Customer Reports

```text
VMware Slow
```

---

Check

```bash
qos statistics volume latency show
```

---

Determine

```text
Latency?
```

---

If yes

Check:

```bash
statistics show -object disk
```

---

Determine:

```text
Disk Bottleneck?
```

---

Check

```bash
statistics show -object aggregate
```

---

Determine:

```text
Aggregate Busy?
```

---

Check

```bash
statistics show -object wafl
```

---

Determine:

```text
CP Pressure?
```

---

# Protocol Troubleshooting

---

# NFS Slow

Check

```bash
statistics show -object nfs
```

---

Check

```bash
qos statistics volume latency show
```

---

# SMB Slow

Check

```bash
statistics show -object cifs
```

---

Check

```bash
qos statistics volume latency show
```

---

# SAN Slow

Check

```bash
statistics show -object lun
```

---

Check

```bash
statistics show -object iscsi
```

or

```bash
statistics show -object fcp
```

---

# AFF Performance Monitoring

Most important metrics:

```text
Latency

IOPS

CPU

WAFL CPs

Cache Efficiency

NVMe Activity
```

---

# Flash Pool Monitoring

Check:

```bash
statistics show -object aggregate
```

---

Look for:

```text
Cache Hits

Cache Misses

Reduced HDD Reads
```

---

# FlexCache Monitoring

Check:

```bash
volume flexcache show
```

---

Performance indicators:

```text
Cache Hits

Reduced WAN Usage

Lower Read Latency
```

---

# NCDA Exam Questions

### Q1

Best tool for historical performance?

```text
Active IQ Unified Manager
```

---

### Q2

Best command for volume latency?

```bash
qos statistics volume latency show
```

---

### Q3

Command for NFS performance?

```bash
statistics show -object nfs
```

---

### Q4

Command for SMB performance?

```bash
statistics show -object cifs
```

---

### Q5

Command for aggregate performance?

```bash
statistics show -object aggregate
```

---

### Q6

Command for disk performance?

```bash
statistics show -object disk
```

---

### Q7

Command for LUN performance?

```bash
statistics show -object lun
```

---

### Q8

Which tool stores months of performance data?

```text
Active IQ Unified Manager
```

---

### Q9

Which command shows EMS events?

```bash
event log show
```

---

### Q10

First command when customer says storage is slow?

```bash
qos statistics volume latency show
```

---

# NCDA Memory Dump

```text
Real-Time
=========

qos statistics volume latency show

statistics show -object volume

statistics show -object aggregate

statistics show -object wafl

statistics show -object disk

statistics show -object nfs

statistics show -object cifs


Historical
==========

Active IQ Unified Manager


Health
======

System Manager

Active IQ Digital Advisor

event log show


Most Important Performance Command
==================================

qos statistics volume latency show


When Customer Says:
"My application is slow"

Start With:
============
qos statistics volume latency show
```
