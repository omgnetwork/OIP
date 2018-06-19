# OIP-xxxx: eWallet Release & Versioning

## Abstract

This document specifies a procedure for releasing and versioning to allow for stable production environment. OmiseGO eWallet Server (hereafter "eWallet Server") is meant to be deployed by a various entities, which may range from a personal store to an enterprise (hereafter "Provider") and thus require a stable operation and smooth upgrade path that maximizes compatibilities.

## Overview

The eWallet release is consist of a stable release and a nightly release. A stable release is meant to be run by a Provider that require stability and compatibility, while a nightly release is meant for development and testing propose. The stable releases should be versioned using [SemVer](https://semver.org/).

## Specifications

### Stable Release

A stable release is a release that provides compatibility and stability guarantee and thus require an extensive end-to-end testing prior to release. Due to this, **we will only provide a stable release as a Docker image based on Alpine Linux**. The Docker image will be tagged with the release version and should use a `:latest` tag to refer to the latest stable release.

Only a the latest `MAJOR.MINOR.PATCH` version of the last 2 `MAJOR.MINOR` versions will be supported. However, we may provide security fixes to a non-supported versions as the team see fit. As an example, we shall support the following versions released in the past:

| Version | Release Date | Support                                    |
| ------- | ------------ | ------------------------------------------ |
| 1.2.2   | 2018-06-12   | **Supported**, latest `1.2`                |
| 1.2.1   | 2018-04-01   | Not supported, not latest `1.2`            |
| 1.2.0   | 2018-01-01   | Not supported, not latest `1.2`            |
| 1.1.1   | 2017-06-20   | **Supported**, latest `1.1`                |
| 1.1.0   | 2017-06-20   | Not supported, not latest `1.1`            |
| 1.0.2   | 2018-02-01   | Not supported, `1.0` discontinued (hotfix) |
| 1.0.1   | 2017-08-01   | Not supported, `1.0` discontinued          |
| 1.0.0   | 2016-08-01   | Not supported, `1.0` discontinued          |

In the above table, `1.0.2` is released as a security fix outside of the support period. However, even though it is released after the supported version `1.1.1`, the version `1.0.2` won't have any guarantee that it will receive any further support.

A stable release require that end-to-end tests and migration test from the supported versions are passed.

### Nightly Release

A nightly release is a release that provides bleeding edge feature at a cost of stability and incompatibility. Thus, a nightly release is only recommended for testing and evaluation purpose. The nightly release will be provided as a binary release and a Docker image based on Alpine Linux. The Docker image will be tagged with commit ID and should use a `:nightly` tag to refer to the latest nightly release.

A nightly release shall require unit testing and integration testing.

### Branching Model

Stable branch shall be separated from the mainline development branch with changes selectively backported when a feature is considered stable enough. Backporting shall be done via a **Pull Request** against the stable branch and shall be done on a per-feature basis.

Once a stable release will happen, a stable release shall have its own branch in a form of `stable-MAJOR.MINOR` and tag with `vMAJOR.MINOR.PATCH` as appropriate.

```
          a1    b1   a2   b2   a3   c1   c2   c3   a4   b3
develop ---o----o----o----o----o----o----o----o----o----o---------->
           |    |    |    |    |    |    |    |
           +--= | =--+    |    |    +----+----+
                |    |    |    |              |
                \--= | =--+--= | =--+         |
                     |         |    |         |
                     v         v    v         v
 stable -------------o---------o----o---------o-------------------->
                  {a1,a2} \   {a3} {b1,b2}   {c1,c2,c3} \
                           \    \                        \
                            \    +------------            \
                             \                \            \
 stable-1.0                   t----------o-----t------------------->
                         v1.0.0 {a1,a2}     v1.0.1 {a3}      \
                                                              \
 stable-1.1                                                    t--->
                                                             v1.1.0
```

In this chart, changes are selectively backported from `develop` to `stable` in a per-feature basis (`{a1,a2}`, `{b1,b2}`, ...) This process can be done by cherry-picking changes and fixing conflicts as necessary.

Once we made a release, we branch directly from the `HEAD` of stable branch at a time into a release branch (`stable-1.0`, `stable-1.1`, ...) and only backport fixes for features that are already released within that `MAJOR.MINOR` version.

In the chart, there's a commit between `v1.0.0` and `v1.0.1` in addition to back-porting the fix from stable branch, this is a **hotfix** for an issue that only exists within the `1.0.0` release but not in the stable release.
