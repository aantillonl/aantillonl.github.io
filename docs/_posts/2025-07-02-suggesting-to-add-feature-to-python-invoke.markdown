---
layout: post
title:  "Suggesting To Add Feature To Python Invoke"
date:   2025-07-02 11:00:00 -0500
---

After using [Invoke](https://www.pyinvoke.org) for a small set of automation tasks,
I realized that when a task is called as a pre-requisite from another task,
the pre-requisite task doesn't have any context of whether it was run directly
or as a pre-requisite from another task. Having such contest could be helpful to
pass around data from tasks to pre-requisite tasks.

A [Pull Request has been open on GitHub](https://github.com/pyinvoke/invoke/pull/1041)