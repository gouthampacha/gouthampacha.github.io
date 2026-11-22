---
title: "One Export Path: PROXY Protocol for NFS"
description: "Active/Active gateways gave NFS scale, but users saw a moving target of export locations. A single stable path, with client IPs preserved end to end, took PROXY protocol."
date: 2024-06-01
showDate: false
showAuthor: false
showReadingTime: true
showTableOfContents: false
tags: ["openstack", "ceph", "nfs", "manila"]
---

Once CephFS NFS gateways run Active/Active behind an ingress, the infrastructure is scalable but the user experience isn't. As gateways come and go, the export locations a user sees keep changing: remount your share on the new address, update your automation, redo your firewall rules. For a cloud service, "the storage moved, please re-mount" is a broken promise. What users want is one stable export path per share, with the ingress handling load balancing across the gateway cluster behind it.

But a single path through the ingress creates an accounting problem. NFS access control in Manila is IP-based: access rules say which client IPs may mount a share. When traffic flows through the ingress, the backend Ganesha servers no longer see the client's address, only the ingress's. Enforce access rules there and every client appears to come from one IP; enforce them nowhere and the multi-tenant isolation the access rules provide is gone. The ingress has to convey the real client IP to the backends, transparently, without changing anything about how NFS clients speak NFS.

[PROXY protocol](https://www.haproxy.com/blog/haproxy/proxy-protocol/) (v2) is the bridge. The ingress prepends a small header carrying the client's address to each connection; the backend Ganesha servers understand it and enforce access rules against the real client IP. Users get a single export location per share, stable across gateway scaling events. The ingress load balances across the Active/Active gateway cluster. IP-based access control keeps working end to end, because the backend sees the true source. And nothing changes for the NFS client: standard NFS, standard mounts.

The consumption model for CephFS NFS went from "infrastructure you have to track" to "an address you can put in /etc/fstab and forget." Scalability events behind the ingress became invisible to users, without giving up the per-client IP enforcement that multi-tenant isolation depends on.
