---
title: "Filesystem Drivers: Lustre, VirtIOFS, SMB"
description: "Manila speaks to storage through drivers. New protocols mean new workloads: the Lustre driver, and ongoing VirtIOFS and CephFS SMB work."
date: 2025-10-01
showDate: false
showAuthor: false
showReadingTime: true
showTableOfContents: false
tags: ["openstack", "storage", "manila", "lustre", "ceph"]
---

Manila's value is that one API fronts many filesystem backends. Every protocol it speaks is a new class of workload it can serve: NFS for compatibility, CephFS for scale, and increasingly, protocols chosen for security boundaries or performance profiles that the originals can't meet. What's on the bench right now:

**[Lustre](https://specs.openstack.org/openstack/manila-specs/specs/hibiscus/lustre-driver.html)**. The Lustre driver for Manila, landing in the Hibiscus release, is my work. Lustre dominates high-performance computing: it's the filesystem behind national labs and large-scale research clusters. Exposing it through Manila turns an HPC storage system into a self-service cloud primitive, with quotas, access rules, and lifecycle management that HPC sites otherwise build by hand.

**VirtIOFS**. I'm working on VirtIOFS support, giving VMs and containers a hypervisor-mediated path to shared storage. Instead of tenants speaking NFS over a network they can touch, the host mounts the filesystem and hands it to the guest over virtio. I wrote up the trust-boundary argument in [ADMIN Magazine](https://www.admin-it.io/closing-the-cephfs-trust-gap-w/): it closes the gap where tenants would otherwise sit on the storage network itself.

**CephFS SMB**. I'm working on the SMB driver for CephFS, which matters for one stubborn reason: a huge share of enterprise file workloads still speak SMB, and Ceph's own SMB support needed a cloud API in front of it. With this, the same Ceph cluster serving CephFS-native and NFS exports can serve Windows estates too.

Each driver is also a security and isolation decision: who can reach the storage network, what identity the mount carries, what happens on eviction. Writing drivers is where Manila's promise ("any shared filesystem, safely, multi-tenant") gets kept or broken. It's also why this is where I keep spending time.
