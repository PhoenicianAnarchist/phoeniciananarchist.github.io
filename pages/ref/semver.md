---
layout: page
title: Semantic Versioning
permalink: /ref/semver
categories: ref meta
---

A version number will be in the format `<major>.<minor>.<patch>`.

- `<major>`
: Any change in the public API which is not backwards compatible.

- `<minor>`
: Any additional functionality which is backwards-compatible (including
  deprecations).

- `<patch>`
: For (backwards compatible) bug-fixes and other minor/meta changes.

When one number is incremented, all subsequent numbers are reset to `0`.

Major version `0` (`0.y.z`) is used for initial development and everything at
this point should be considered subject to change. The public API should only
be considered to be stable from major version `1` (`1.y.z`)onwards.
