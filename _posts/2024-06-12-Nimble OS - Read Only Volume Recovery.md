---
title: Nimble OS - Read Only Volume Recovery
date: 2024-06-12 12:00:00 +/-TTTT
categories: [Hardware, Nimble OS]
tags: [hardware,nimble os]     # TAG names should always be lowercase
description: How to quickly and programmatically bring offline Nimble VVOLs back to life and the bug that required it.
---

## Background

I love Nimble, VMWare, and Veeam.  However, in the past several years I have run up against a nasty bug that I can only assume is the result of a perfect storm of storage snapshotting of VVOLs and VMWare snapshotting as initiated by Veeam's hot plug backup feature.

During Veeam's hot plug method, the original VM runs on a snapshot while the underlying original disk is mounted to the backup server VM itself to read the changed blocks.

## So what's the problem?

If, during this time it happens to mount several large disks of machines residing on other VVOL folders and attempts a snapshot, Nimble's VMWare plugin registers the VVOL folder utilization as several hundred percent and proactively marks all volumes as read only; halting all VMs in the process.  Such was the case for me.

VVOLs have some pretty awesome benefits but becomes problematic in this instance because every VM, every virtual disk, every swap file is itself a volume.  This adds up quickly across several hundred virtual machines; in our case ~950 or so volumes.  In order to online these volumes again, it requires manual intervention per volume via cli.

## Resolution

Perform the following to reduce the chance of running into this again:
* Verify backup job has ceased.
* Redistributed VMs across VVols if necessary.

Finally, ssh into the Nimble and issue the following command to find all volumes marked read-only and set them to online:

```bash
vol --list | awk 'NR>5 {print $1}' | while read VOL; do vol --edit $VOL --readonly no; done
```