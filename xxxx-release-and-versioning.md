# OIP-xxxx: eWallet Release & Versioning

## Abstract

This document specifies a procedure for releasing and versioning to allow for stable production environment. OmiseGO eWallet Server (hereafter "eWallet Server") is meant to be deployed by a various entities, which may range from a personal store to an enterprise (hereafter "Provider") and thus require a stable operation and smooth upgrade path that maximizes compatibilities.

## Overview

The eWallet release is consist of a stable release and a development release. A stable release is meant to be run by a Provider that require stability and compatibility, while a development release is meant for development and testing propose. The stable releases should be versioned using [SemVer](https://semver.org/).

## Specifications

### Stable Release

A stable release is a release that provides compatibility and stability guarantee and must pass an extensive end-to-end testing and migration testing from supported versions prior to release.

| Test subject        | Required |
| ------------------- | -------- |
| Linting             | **Yes**  |
| Unit testing        | **Yes**  |
| Integration testing | **Yes**  |
| End to end testing  | **Yes**  |
| Migration testing   | **Yes**  |

A stable release will be provided as a Docker image based on Alpine Linux. The Docker image will be tagged with the release version (e.g. `v1.0.3`) and will use a `:latest` tag to refer to the latest stable release.

Only the latest `MAJOR.MINOR.PATCH` version of the last 2 `MAJOR.MINOR` versions will be supported. However, we may provide security fixes to non-supported versions as the team see fit. For demostration purpose, given that we released the following versions at a specific date, our support condition shall be:

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

In the figure above, version `1.0.2` is released as a security fix outside of the support period. This release is considered a hotfix release. However, even though version `1.0.2` is released after the supported version `1.1.1`, the version `1.0.2` won't be guarantee to receive any further updates. Provider is encouraged to upgrade to latest supported version.

### Development Release

A development release is a release that provides bleeding edge features that may still be unstable, as the branch is only required to pass unit testing and integration testing. Usage of development release in a production environment is highly discouraged, and only recommended for testing and evaluation purpose.

| Test subject        | Required |
| ------------------- | -------- |
| Linting             | **Yes**  |
| Unit testing        | **Yes**  |
| Integration testing | **Yes**  |
| End to end testing  | No       |
| Migration testing   | No       |

The development release will be provided as a binary release and a Docker image based on Alpine Linux. The Docker image will be tagged with commit ID and will have a `:dev` tag to refer to the latest development release.

### Branching Model

The branching model will utilize a `master` branch, a stable version branch and a feature branch. To demostrate:

```
feature2    -o---------o----------o
                                   \
                                    \
feature1    -o                       \
              \                       \
               v                       v
master      ----o-----------------------o-------------------------------->
                             \                           \
                              v                           \
stable-1.0                     o-----------o----o----------^------------->
                            v1.0.0           v1.0.1         \
                                                             v
stable-1.1                                                    o---------->
                                                            v1.1.0
```

#### Feature Branches

Feature branches, such as `feature2` and `feature1`, are development branches that implements new feature or change existing functionality. These branches may be long-running and should be merged to `master` only when the feature is considered stable enough for public consumption. Feature branches should regularly sync up with master to avoid too much code diversion. Feature branches may have its own public testing environment for each branch with its data copied from staging on each deploy.

#### `master` branch

The `master` branch is the branch that development release will be based on. While we required that feature branch must be stable enough before it could be merged into `master`, the `master` branch may still not stable enough to use in production environment. The `master` branch is where a stable branch will be branched from when it is considered stable enough. Before branching into stable, `master` may undergo a period of feature freeze, in which new features are not allowed to be merged onto the master branch, and only stability or bugfixes are accepted. The master branch will be deployed automatically to the OmiseGO staging environment for internal and public testing.

#### Stable version branches

Stable version branches such as `stable-1.0` and `stable-1.1`, are branches that correspond to the `MAJOR.MINOR` release versions. The branch is branched from `master` when the `master` branch is considered enable enough. Stable version branches are solely for maintenance purpose and will not accept any new features except for bug or security fixes.
