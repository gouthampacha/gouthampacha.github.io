---
title: "Share Replication in Manila"
description: "Shared filesystems had no answer to a backend failure until replication. Getting the semantics right, especially across failure domains, was the work."
date: 2016-04-01
showDate: false
showAuthor: false
showReadingTime: true
showTableOfContents: false
tags: ["openstack", "storage", "manila", "ha"]
---

Before replication, Manila had no answer for a failed storage backend except waiting for it to come back. For shared filesystems serving databases and CI systems, that's a gap: the share is the workload's working state, and the working state lives in exactly one place.

Replicating a share sounds like a solved problem, since block storage and databases had done it. But shares carry complications of their own. Replicas have to live in different failure domains or they don't protect anything, which means the scheduler needs to understand placement not just at creation time but across promotion and resync. Access rules have to track replicas individually, since each replica may be reachable on a different network. And the API needs honest states, "in sync," "out of sync," "error," rather than pretending replication is instantaneous, because failover decisions depend on knowing what a replica actually has.

I was a founding contributor to [share replication](https://docs.openstack.org/manila/latest/admin/shared-file-systems-share-replication.html), working through those semantics, which the feature still uses today: replica relationships per share with explicit states for sync and promotion, scheduler placement across failure domains so replicas don't share a fate, per-replica access-rule handling, and manage and unmanage operations so existing replicated filesystems could be adopted rather than rebuilt.

The result was high availability and disaster recovery for shared storage in OpenStack: promote a replica, keep serving. Geo-replicated setups across data centers became a supported pattern rather than a hand-built one. And the semantics settled here became the foundation the later HA work built on; replication is where Manila first had to think hard about failure domains, and that thinking paid off repeatedly.
