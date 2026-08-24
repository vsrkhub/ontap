# FlexClone and Volume Move
# What Happens When Moving a Parent Volume or a FlexClone Volume?

This is an advanced ONTAP topic and commonly appears in:

```text
Storage Migrations
Technology Refresh
Storage Rebalancing
Capacity Management
Dev/Test Environments
```

---

# First Understand FlexClone

A FlexClone is:

```text
Writable
Space Efficient
Snapshot Based
```

created from:

```text
Volume
Snapshot
LUN
```

---

## Example

Parent Volume

```text
vol_prod
```

Snapshot

```text
vol_prod:snap1
```

Clone

```text
vol_dev
```

---

Architecture

```text
vol_prod
      |
      |
      +---- snap1
                |
                |
                +---- vol_dev
```

---

# Storage Consumption

Initially:

```text
vol_prod = 10TB

vol_dev = FlexClone
```

Additional consumption:

```text
Almost Zero
```

because both share blocks.

---

# Shared Block Relationship

```text
Parent Volume

      A
      B
      C
      D
```

Clone Created

```text
Parent
      A
      B
      C
      D

Clone
      A
      B
      C
      D
```

Physical storage:

```text
1 copy
```

Logical storage:

```text
2 copies
```

---

# What Happens During Volume Move?

Question:

```text
Can I move a FlexClone?
```

Answer:

```text
Yes
```

---

Question:

```text
Can I move the parent?
```

Answer:

```text
Yes
```

---

Question:

```text
Will the shared block relationship survive?
```

Answer:

```text
Usually Yes
```

This is the important part.

---

# Scenario 1
# Move Parent Volume

Initial

```text
aggr1

vol_prod
      |
      +---- vol_dev
```

---

Move Parent

```bash
volume move start \
-vserver svm1 \
-volume vol_prod \
-destination-aggregate aggr2
```

---

Result

```text
aggr2

vol_prod
      |
      +---- vol_dev
```

Clone relationship preserved.

---

# Scenario 2
# Move Clone Volume

Initial

```text
aggr1

vol_prod
vol_dev
```

---

Move Clone

```bash
volume move start \
-vserver svm1 \
-volume vol_dev \
-destination-aggregate aggr2
```

---

Result

```text
aggr1

vol_prod


aggr2

vol_dev
```

---

# What Happens To Shared Blocks?

This is where it gets interesting.

Before move:

```text
Parent and Clone share blocks.
```

---

After move:

```text
Different Aggregates
```

cannot physically share the same aggregate blocks.

Therefore ONTAP may:

```text
Split sharing relationships
Copy required blocks
Consume more capacity
```

---

# Exam Rule

If:

```text
Parent and Clone stay
within same aggregate
```

sharing is preserved.

---

If:

```text
Moved to different aggregate
```

expect:

```text
Additional physical space
```

to be required.

---

# Common Real World Surprise

Before Move

```text
Parent = 20TB

Clone = 20TB

Physical Usage = 21TB
```

---

After Clone Move

```text
Parent = 20TB

Clone = 20TB

Physical Usage = 40TB+
```

because block sharing opportunities may be lost.

---

# How To Check Clone Relationships

Show Clones

```bash
volume clone show
```

---

Detailed

```bash
volume clone show -instance
```

---

Example

```text
Volume: vol_dev

Parent Volume: vol_prod

Parent Snapshot: snap1
```

---

# How To Check Space Savings

Show Footprint

```bash
volume show-footprint
```

---

Show Space

```bash
volume show-space
```

---

Show Clone Information

```bash
volume clone show
```

---

# How To Move A FlexClone

Step 1

Check Clone

```bash
volume clone show
```

---

Step 2

Check Space

```bash
volume show-footprint
```

---

Step 3

Start Move

```bash
volume move start \
-vserver svm1 \
-volume vol_dev \
-destination-aggregate aggr2
```

---

Step 4

Monitor

```bash
volume move show
```

---

Step 5

Verify

```bash
volume clone show
```

---

Step 6

Verify Space Usage

```bash
volume show-footprint
```

---

# What Is Clone Split?

Another NCDA and real-world topic.

---

## Clone Split

Convert:

```text
Clone
```

into:

```text
Independent Volume
```

---

Before

```text
Parent
   |
Clone
```

Shared blocks.

---

After Split

```text
Parent

Clone
```

No shared blocks.

---

# Commands

Show Status

```bash
volume clone show
```

---

Start Split

```bash
volume clone split start
```

---

Monitor

```bash
volume clone split show
```

---

Stop

```bash
volume clone split stop
```

---

# Why Perform Clone Split?

Examples

```text
Long Term Retention

Storage Migration

Independent Testing

Remove Parent Dependency
```

---

# Volume Move vs Clone Split

---

## Volume Move

Moves:

```text
Volume Location
```

---

Relationship remains if possible.

---

## Clone Split

Changes:

```text
Storage Relationship
```

---

No sharing remains.

---

# Storage Efficiency Impact

Before Split

```text
50TB Parent

50TB Clone
```

Physical

```text
52TB
```

---

After Split

```text
100TB+
```

Possible.

---

# Troubleshooting Scenario 1

Clone Move Fails

Check

```bash
volume move show
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

# Troubleshooting Scenario 2

Storage Usage Increased After Move

Most likely:

```text
Shared Block Relationships Reduced
```

---

Check

```bash
volume show-footprint
```

---

# Troubleshooting Scenario 3

Clone Split Appears Stuck

Check

```bash
volume clone split show
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

# Troubleshooting Scenario 4

Unexpected Aggregate Full

Check

```bash
volume clone show
```

---

Possible Cause

```text
Clone Split Consumed Additional Space
```

---

# Advanced Example

Initial

```text
aggr1

vol_prod = 20TB

vol_dev = clone
```

Physical Usage

```text
21TB
```

---

Move Clone

```text
aggr2
```

---

Expected Result

```text
Shared Blocks Recreated As Needed

Capacity Consumption Increases
```

---

# NCDA Exam Questions

### Q1

What is a FlexClone?

```text
Writable space-efficient clone.
```

---

### Q2

Does FlexClone initially consume full space?

```text
No
```

---

### Q3

Can a FlexClone be volume moved?

```text
Yes
```

---

### Q4

Can a parent volume be volume moved?

```text
Yes
```

---

### Q5

What command shows clones?

```bash
volume clone show
```

---

### Q6

What command shows clone details?

```bash
volume clone show -instance
```

---

### Q7

What command starts a volume move?

```bash
volume move start
```

---

### Q8

What command monitors a move?

```bash
volume move show
```

---

### Q9

What command starts clone split?

```bash
volume clone split start
```

---

### Q10

What command monitors split progress?

```bash
volume clone split show
```

---

### Q11

Can moving a clone increase storage usage?

```text
Yes
```

---

### Q12

Why?

```text
Shared blocks may no longer remain shared across aggregates.
```

---

### Q13

What command shows actual footprint?

```bash
volume show-footprint
```

---

### Q14

What is clone split?

```text
Making a clone fully independent.
```

---

### Q15

What happens to storage efficiency after a clone split?

```text
Capacity consumption increases significantly.
```

---

# NCDA Memory Dump

```text
FlexClone
=========

Writable
Space Efficient
Snapshot Based

Commands
=========

volume clone show

volume clone show -instance

volume clone split start

volume clone split show

volume move start

volume move show

Key Rule
=========

Move Parent?
YES

Move Clone?
YES

Clone Split?
YES

Move Across Aggregates?
Possible Capacity Increase

Clone Split?
Maximum Capacity Increase

Check Actual Savings
====================

volume show-footprint
```
