---
title: "Removing Eventlet from OpenStack"
description: "A concurrency library from 2010 was making the entire project undeployable. Fixing it meant organizing a migration across dozens of projects before the deadline fixed itself."
date: 2024-06-01
showDate: false
showAuthor: false
showReadingTime: true
showTableOfContents: false
tags: ["openstack", "python", "governance"]
---

Eventlet is a Python concurrency library that OpenStack has depended on since 2010. It works by monkey-patching the standard library at runtime, swapping out sockets, threads, and time functions for cooperative green-thread versions. For fifteen years, that trade-off mostly worked.

Then it stopped working. Python 3.12 and 3.13 broke Eventlet's monkey-patching, and Linux distributions (RHEL 10, Ubuntu 24.04) shipped those versions. The deadline wasn't negotiable: when the operating systems OpenStack runs on no longer supported its concurrency model, OpenStack becomes undeployable. This wasn't a Manila problem. Every OpenStack service used Eventlet.

The technical fix, replacing green threads with native threading, is straightforward to state and painful to do. Each service had years of implicit assumptions baked in: code that relied on cooperative scheduling being "safe," green-thread-safe locks used as cross-thread locks, WSGI servers that only worked under Eventlet. And there was no single owner. OpenStack is dozens of independent project teams with their own priorities; nobody is obligated to spend their release cycle on someone else's existential problem. The real design question was organizational: how do you make a cross-project migration *somebody's* priority when it's *everybody's* problem?

The migration became a [community-wide goal](https://governance.openstack.org/tc/goals/) with its own goal champion. My contribution was organizational: I came up with the pop-up team concept, the structure that gave this work a home when it wasn't any single project's backlog, and I work with the goal champion to drive the effort to conclusion in the 2027.2 release. The team got its own channel and biweekly meetings, a multi-release timeline (Epoxy → Flamingo → Gazpacho and onward) so projects could migrate at their own pace instead of facing a big-bang deadline, and migration documentation plus threading-compatible launcher classes so each team wasn't rediscovering the same pitfalls. In Manila, the work meant deprecating the built-in Eventlet WSGI server, replacing green-thread calls with native threading, adding CI jobs that run the entire test suite under native threading, and writing the new WSGI module.

OpenStack stayed deployable on current operating systems, which is to say, it stayed alive. The Eventlet work was called out by name in [The Register's coverage](https://www.theregister.com/2025/10/01/openstack_flamingo_release/) of the Flamingo release. What I'm proudest of is that a volunteer federation of project teams completed an unfunded, unglamorous, project-wide migration on a schedule, because the goal had an owner, a structure, and a deadline that everyone could see coming.
