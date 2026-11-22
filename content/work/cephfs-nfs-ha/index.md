---
title: "High-Availability NFS for CephFS"
description: "Active/Passive NFS gateways under Pacemaker capped throughput and left a single point of failure. The answer was Active/Active gateways, orchestrated by Ceph, load balanced by HAProxy."
date: 2019-06-01
showDate: false
showAuthor: false
showReadingTime: true
showTableOfContents: false
tags: ["openstack", "ceph", "storage", "manila"]
---

NFS gateways (NFS-Ganesha) are the compatibility layer that lets CephFS serve standard NFS clients. For a long time, the available high-availability story for those gateways was Active/Passive under Pacemaker: one node serves traffic, a standby takes over on failure. That model has two structural problems. The active node is a bottleneck, since all NFS traffic for the deployment flows through a single gateway, so throughput is capped by one host. And failover isn't free: during the Pacemaker transition, clients stall. For large production environments, telcos in particular, both the ceiling and the stall are disqualifying at scale.

The fix on paper is simple: run multiple active gateways and load balance across them. In practice, Active/Active NFS is hard. Ganesha clusters need their export configurations kept in sync across nodes; Ceph's own tooling (Cephadm and the Ceph orchestrator) had to own the gateway lifecycle, which meant Manila giving up direct control of a component it had always commanded itself, over dbus and SSH. And a cloud service can't hand-manage gateway topology: whatever the architecture was, operators had to be able to scale and repair it through APIs, not console sessions.

The architecture that came out of this work:

- The Ceph orchestrator (Cephadm) deploys and scales NFS-Ganesha containers, so gateway clusters are managed like any other Ceph service
- Gateways run Active/Active behind HAProxy, so NFS throughput scales horizontally and no single node is a bottleneck or a single point of failure
- The [Ensure Shares API](https://specs.openstack.org/openstack/manila-specs/specs/dalmatian/add-ensure-shares-api.html) (`POST /v2/services/ensure`, [API reference](https://docs.openstack.org/api-ref/shared-file-system/)), which I designed and Carlos Eduardo da Silva implemented, lets operators trigger export recovery or reconfiguration on a specific backend, on demand, with zero control-plane restarts. I later extended it for CephFS NFS with the transition logic that reconciles exports as Ganesha clusters scale
- The [resource locks framework](https://www.openstack.org/blog/new-in-openstack-bobcat-manila-team-introduces-resource-lock-framework/) from Bobcat made it safe to attach shares to VMs: Nova places deletion locks so a running workload's storage can't be destroyed out from under it

This is the architecture running in production at major US telecoms through Red Hat's product: horizontally scalable, Active/Active NFS on CephFS, with no failover stalls and no single-host ceiling. The whole distance came from one decision: give the gateway lifecycle to the orchestrator that already knew how to run Ceph services, and keep Manila's authority where it belongs, in the API layer that makes the result operable.
