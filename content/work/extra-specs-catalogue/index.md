---
title: "Extra-Specs and the Storage Service Catalogue"
description: "Heterogeneous backends, one API. The trick was making share types a service catalogue: operators describe offerings, the scheduler matches them, and the rest is optional."
date: 2015-09-01
showDate: false
showAuthor: false
showReadingTime: true
showTableOfContents: false
tags: ["openstack", "manila", "design"]
---

Manila from the beginning faced a problem most OpenStack services didn't: the backends underneath it were wildly different. One operator runs CephFS, another a NetApp filer, another a purpose-built NFS cluster, and each offers a different set of capabilities: replication or not, snapshots or not, different performance tiers, different protocols. Hard-code those differences into the API and every new backend feature needs an API change. Expose them raw and users need to know storage internals to request a share.

The design that came out of those early discussions became Manila's [share types and extra-specs](https://docs.openstack.org/manila/latest/admin/shared-file-systems-share-types.html), and the model off of the API is simple: **without asking, you don't get it.**

A share type carries extra-specs, key-value pairs describing what a share of that type gets: driver, protocol, replication, snapshot support, QoS. Operators compose these into a catalogue, "gold NFS with replication," "fast CephFS," and so on. It's a way for administrators to ask for, and cater to, capabilities they know they have (or don't). The scheduler matches required extra-specs against what backends actually report through the [capabilities API](https://docs.openstack.org/manila/latest/admin/capabilities_and_extra_specs.html), so a request lands only where it can be served. End users see the effect immediately: the API turns down requests the cloud can't serve, and the capability is reflected in the resource itself.

The subtlety that makes the whole thing work is the "optional" distinction, and it matters as much as the required ones. Optional extra-specs are conveyed to the backend without constraining placement, letting offerings carry hints rather than hard filters. Required specs decide where a share can live; optional specs tell the backend what the operator intends for it. Without that split, every capability hint would silently become a scheduling constraint, and the catalogue would collapse into whatever the smallest backend could satisfy.

The catalogue framing turned a heterogeneous storage fleet into a self-service menu. Operators differentiate their offerings without API changes; users pick a service level instead of learning vendor internals; the scheduler absorbs the mismatch. It's a 2015 design that still carries the service today, which is about as well as an abstraction can age.
