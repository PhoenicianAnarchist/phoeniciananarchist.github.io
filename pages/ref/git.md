---
layout: page
title: Git Branching Strategy
permalink: /ref/git
categories: ref meta
---

## References

- nvie : [git flow][]
- stackoverflow : [deleting branches][]

## Permanent Branches

Two main branches exist, `master` and `dev`. These are long-lived branches and
will exist for the lifetime of the project.

The `master` branch contains stable releases which are tagged with a version
number. While not guaranteed, the `dev` branch should be decently stable enough
that a "nightly" build can be taken from it.

```
v [0.0.0]   v [0.1.0]
o-----------o (master)
 \         /
  o---o---o---o---o (dev)
```

## Feature Branches

A number of commits implemented a given feature should be grouped together with
a feature branch. This makes it easier to isolate the development of a given
feature and roll-back/remove it if needed.

These branches are always merged back into `dev` with the `--no-ff` flag to
preserve the branch structure.

```
>---o-------------------o---> (dev)
     \                 /
      o---o---o---o---o (feature x)
```

## Release Preparation

The `dev` branch should not be merged directly into `master`, but rather via a
short-lived intermediate branch `release-*`.

This release branch exists for finalising the upcoming release and determining
the version number increment. No new features may be added, only bug-fixes and
meta-data.

```
v [0.0.0]           v [0.1.0]
o-------------------o (master)
 \                 /
  \           o---o (release-*)
   \         /     \
    o---o---o-------o---o---o (dev)
```

Creating a `release-*` branch:

```
$ git checkout dev
$ git checkout -b release-<date>
```

The future release is then checked and tested, and any bug fixes are committed.

The version increment is determined [`/ref/semver`](/ref/semver) and any
meta-data is updated where necessary.

```
$ git checkout master
$ git merge --no-ff release-<date>
$ git tag <version>
```

```
$ git checkout dev
$ git merge --no-ff release-<date>
```

After this, the `release-*` branch is to be deleted.

## Critical Fixes

If a critical bug in a release needs an immediate patch, a `hotfix-*` branch is
created. Branching from the relevant commit on `master`, merging back into
`master` as well as `dev`.

```
v [0.0.0]               v [0.1.0]   v [0.1.1]
o-----------------------o-----------o (master)
 \                     / \         /
  \                   /   o-------o (hotfix-*)
   \                 /             \
    \           o---o               \
     \         /     \               \
      o---o---o-------o---o---o-------o (dev)
```

If there is a `release-*` branch currently active, `hotfix-*` is merged into
`release-*` instead, and will eventually make it back into `dev`.

```
v [0.0.0]               v [0.1.0]   v [0.1.1]
o-----------------------o-----------o (master)
 \                     / \         /
  \                   /   o-------o (hotfix-*)
   \                 /             \
    \           o---o           o---o---o (release-*)
     \         /     \         /             
      o---o---o-------o---o---o-------o (dev)
```

---

## Deleting Branches

When deleting a branch, there are actually _three_ branches to consider:

The local branch:

`git branch --delete <branch>`

The local remote-tracking branch:

`git branch --delete --remote <remote>/<branch>`

The remote branch:

`git push <remote> --delete <branch>`

[git flow]: <https://nvie.com/posts/a-successful-git-branching-model/>
[deleting branches]: <https://stackoverflow.com/questions/2003505/how-do-i-delete-a-git-branch-locally-and-remotely/23961231#23961231>
