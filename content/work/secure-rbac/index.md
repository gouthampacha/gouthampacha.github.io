---
title: "Secure RBAC in Manila"
description: "OpenStack's authorization model was binary: admin, or everyone else. Rebuilding it meant auditing 200+ API endpoints without breaking a single running cloud."
date: 2021-06-01
showDate: false
showAuthor: false
showReadingTime: true
showTableOfContents: false
tags: ["openstack", "security", "manila"]
---

Manila's authorization model was inherited from OpenStack's early days: a binary "admin vs. everyone else" split. Any user with a plain "member" role in a project could delete that project's resources, including shared storage actively serving databases and CI systems. There was no read-only access for auditors, and no distinction between cloud-wide operations and project-scoped ones. For shared filesystems this is worse than it sounds: a share isn't a private object, it's often mounted by workloads in *other* projects. Delete the share, and you take down someone else's running system.

The fix, OpenStack's Secure RBAC model with system/project scopes and a reader role, was defined at the platform level. But implementing it in Manila meant touching every API endpoint, 200+ of them, and doing it *while clouds were running*. Every deployment in the world would upgrade through the transition: break backward compatibility, and you brick production clouds on upgrade day; preserve it blindly, and you preserve the security holes you're trying to close. There was also a subtle trap, in that Manila's legacy `context_is_admin` check could silently bypass the new policy engine, meaning the old shortcut would override the new security model without anyone noticing.

The implementation took three years and [44 changes](https://review.opendev.org/c/openstack/manila/+/916025) (2021–2024):

- All 200+ API endpoints audited and assigned their correct scope: system admin, project member, or reader
- Policy definitions migrated from static JSON files (which drift and get hand-edited in production) to code
- The reader role implemented across every GET API, giving auditors access without write rights
- A backward-compatibility layer so existing deployments kept working across upgrade cycles
- The `context_is_admin` bypass fixed so the new policy engine was actually in charge
- 404 rather than 403 for unauthorized resources, so an attacker can't even confirm a resource exists

The culminating change flipped enforcement on across all Manila APIs: the largest security improvement in the project's history, and it shipped in Red Hat's product to every customer. Auditors got real read-only access, cloud operators got scoped admin roles, and the "any member can delete the share under your database" class of risk went away.
