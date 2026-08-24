# ONTAP Compaction Deep Dive
# What Happens When Blocks Are Larger Than 4KB?

This is where many ONTAP administrators misunderstand compaction.

A common misconception is:

```text
Compaction only works for files smaller than 4KB.
```

This is NOT correct.

Compaction operates on:

```text
Partially-used WAFL blocks
```

and not directly on files.

---

# First Understand WAFL Blocks

Historically WAFL stores data in:

```text
4KB logical blocks
```

Internally ONTAP works with:

```text
WAFL blocks
Physical SSD blocks
Compressed blocks
Compacted blocks
```

Compaction operates AFTER dedupe/compression decisions.

Think:

```text
Write
 ↓
Dedupe
 ↓
Compression
 ↓
Compaction
 ↓
SSD
```

---

# Small File Example

File:

```text
1 KB
```

Without compaction:

```text
4 KB WAFL Block

1 KB Used
3 KB Wasted
```

---

10 files:

```text
1 KB each
```

Without compaction:

```text
10 WAFL blocks

40 KB consumption
```

---

With compaction:

```text
10 KB actual data

packed together
```

May consume:

```text
~3 WAFL blocks
```

instead of:

```text
10 WAFL blocks
```

Huge savings.

---

# What About A 6KB File?

This is where NCDA-level understanding becomes interesting.

---

## Without Compression

File:

```text
6 KB
```

WAFL needs:

```text
Block1 = 4KB
Block2 = 2KB
```

Result:

```text
2KB unused in Block2
```

---

Compaction sees:

```text
Block2 only partially full
```

and may combine it with:

```text
Other partial blocks
```

from other writes.

---

Example

```text
FileA = remaining 2KB

FileB = remaining 1KB

FileC = remaining 1KB
```

---

Compaction may pack:

```text
2KB
1KB
1KB
```

into fewer physical blocks.

---

# What About Large Files?

Example:

```text
100 MB file
```

Many people assume:

```text
Compaction does nothing
```

Not entirely true.

---

Most large files rarely align perfectly.

Example:

```text
100 MB + 3 KB
```

Last block:

```text
3 KB Used
1 KB Empty
```

Compaction can still optimize:

```text
Last partially occupied blocks
```

---

# Database Workloads

Compaction shines here.

Example:

```text
Oracle
SQL
VMFS Metadata
VMDK Metadata
```

Workloads generate:

```text
Many small random updates
```

---

Example

A database page writes:

```text
512 B

1 KB

2 KB

3 KB
```

Repeated millions of times.

Without compaction:

```text
Lots of partially consumed blocks
```

---

With compaction:

```text
Packed efficiently
```

---

# VMware Example

Imagine:

```text
1000 virtual machines
```

Each VM generates:

```text
Metadata Updates
Logs
Directory Updates
Lock Files
```

These are often:

```text
Sub-4KB writes
```

---

Compaction becomes extremely valuable.

This is one reason AFF systems achieve:

```text
3:1
4:1
5:1+
```

effective data reduction.

---

# Interaction With Compression

Very important NCDA concept.

---

Suppose:

Original block:

```text
4 KB
```

Compression reduces to:

```text
2 KB
```

---

Now there is:

```text
2 KB free space
```

Compaction may place:

```text
Another compressed block
```

into the same physical storage area.

---

Therefore:

```text
Compression creates opportunities
for compaction.
```

---

# Example

File A

```text
4KB
```

Compressed

```text
2KB
```

---

File B

```text
4KB
```

Compressed

```text
1.5KB
```

---

Without compaction:

```text
Use 2 blocks
```

---

With compaction:

```text
Can potentially share physical space
```

Result:

```text
Even greater savings
```

---

# Interaction With Dedupe

Dedupe executes first.

Example:

```text
Block A
Block A
Block A
```

---

Dedupe:

```text
Store once
```

---

Compression:

```text
Shrink stored block
```

---

Compaction:

```text
Pack remaining free space
```

---

Think:

```text
Dedupe = Remove Copies

Compression = Shrink Data

Compaction = Remove Empty Space
```

---

# Why AFF Benefits More

AFF workloads typically contain:

```text
Small Random Reads
Small Random Writes
Metadata Updates
VM Workloads
Databases
```

---

These workloads naturally produce:

```text
Partially filled blocks
```

Compaction has lots to optimize.

---

# Why Archive FAS Sees Less Benefit

Archive workload:

```text
100GB Video
50GB Backup File
20GB ISO Image
```

---

Characteristics:

```text
Large Sequential Files
Large Blocks
Already Compressed Data
```

Result:

```text
Fewer partially utilized blocks
```

Compaction benefit is lower.

---

# Exam-Level Difference

## Dedupe

Question:

```text
What if two blocks are identical?
```

Answer:

```text
Store one copy
```

---

## Compression

Question:

```text
Can this block be shrunk?
```

Answer:

```text
Reduce size
```

---

## Compaction

Question:

```text
Do we have partially utilized space?
```

Answer:

```text
Pack it together
```

---

# Advanced Example

Imagine:

```text
Block1 = 1KB Used

Block2 = 2KB Used

Block3 = 1KB Used

Block4 = 4KB Used
```

---

Without Compaction:

```text
16KB allocated

8KB actually used
```

---

With Compaction:

```text
Partial blocks packed

8KB allocated

8KB used
```

---

# NCDA Interview Question

### Q

Will compaction help a 1TB video file?

Answer:

```text
Very little.

The file is mostly full blocks.
Only partial tail blocks may benefit.
```

---

### Q

Will compaction help millions of 1KB files?

Answer:

```text
Yes.

This is one of the ideal compaction workloads.
```

---

### Q

Does compaction require duplicate data?

Answer:

```text
No.

That's deduplication.
```

---

### Q

Does compaction compress data?

Answer:

```text
No.

Compaction packs partially consumed blocks.
Compression reduces block size.
```

---

# NCDA Memory Dump

```text
Compaction
==========

Works on:
Partially Filled Blocks

Best For:
Small Files
Databases
VM Metadata
Random Writes

Poor For:
Large Sequential Files
Video Files
Already Compressed Data

Order of Operations:
Dedupe
 ↓
Compression
 ↓
Compaction

Dedupe
======
Removes duplicate blocks

Compression
===========
Shrinks block size

Compaction
==========
Packs free space inside partially used blocks
```
