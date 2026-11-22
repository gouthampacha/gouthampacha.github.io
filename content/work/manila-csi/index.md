---
title: "Manila CSI: Shared Storage for Kubernetes"
description: "Kubernetes needed ReadWriteMany volumes; OpenStack had the filesystems. The driver that connects them had to absorb a real impedance mismatch."
date: 2019-05-01
showDate: false
showAuthor: false
showReadingTime: true
showTableOfContents: false
tags: ["openstack", "kubernetes", "csi", "manila"]
---

Kubernetes had no good answer for ReadWriteMany storage. Block volumes are ReadWriteOnce by nature; shared filesystems are what database config clusters, CI artifact stores, and AI training pipelines actually need. OpenStack clouds had a service for exactly this, Manila, and Kubernetes had a standard plug-in interface, the Container Storage Interface. What was missing was the bridge, and it wasn't a thin one: CSI is a demanding contract (dynamic provisioning, snapshot and clone semantics, node publish/unpublish, topology, resize, all over gRPC), while Manila's abstractions don't map one-to-one onto it. A share has access rules rather than mount credentials, share servers rather than fixed endpoints, and semantics that vary by protocol. The design question was where to absorb that mismatch: hide it inside the driver, or expose it and let users tune it.

The [Manila CSI driver](https://github.com/kubernetes/cloud-provider-openstack/tree/master/docs/manila-csi-plugin) lives in the cloud-provider-openstack repository. My contributions were on the design side, working through how the CSI contract maps onto Manila's model: the driver provisions Manila shares on behalf of PersistentVolumeClaims, mapping access levels to Kubernetes access modes; protocol-specific helpers (NFS, CephFS, later SMB and VirtIOFS) keep the shared logic in one place while handling mount-time differences; and multi-tenancy is preserved end to end, since each claim's share gets its own access rules, so one namespace can't read another's storage.

I continue to maintain this CSI integration and review contributions to it.

ReadWriteMany persistent volumes in Kubernetes backed by OpenStack shared storage: multi-tenant NFS and CephFS out of the box. The stack has been the subject of talks at three summits (Denver 2019, Shanghai 2019, Berlin 2022), and it's the integration that put Manila in front of the Kubernetes ecosystem rather than just the OpenStack one.
