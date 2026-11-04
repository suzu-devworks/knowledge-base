# Git Flow

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Git Flow Example](#git-flow-example)
- [References](#references)

## Overview

Vincent Driessen proposes "A successful Git branching model" as a development model that utilizes Git branches.

- [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)

There are also antis.

## Git Flow Example

```mermaid
%%{init: { 'logLevel': 'debug', 'theme': 'default', 'gitGraph': { 'mainBranchName': 'master' } } }%%
gitGraph
%% Initial setup
commit id: "Tag 0.1"

%% Branching out to develop
branch develop
checkout develop
commit
commit
commit

%% Feature branches
branch feature/future
checkout feature/future
commit id: "Feature for future"

checkout develop
branch feature/major
checkout feature/major
commit id: "Major feature"
commit
commit

checkout feature/future
commit

checkout develop
merge feature/major

checkout feature/future
commit
commit
commit

checkout develop
merge feature/future

%% Release branch 1.0
branch release/1.0
checkout release/1.0
commit id: "Start release 1.0"
commit id: "Only bugfixes"
commit

%% Hotfix branch
checkout master
branch hotfix/0.2
checkout hotfix/0.2
commit id: "hotfix 0.2"

checkout master
merge hotfix/0.2 tag: "Tag 0.2"

checkout develop
merge hotfix/0.2

%% Merging release back to develop and master
checkout develop
merge release/1.0

checkout master
merge release/1.0 tag: "Tag 1.0"

checkout develop
commit

```

| Repository configuration | Description |
| :-: | :-: |
| master | Holds the HEAD of released source code. Once the develop source code is stable, create a Release branch and prepare for release. Once a release is ready, it will be merged from the Release branch into master and tagged with a release number. |
| develop | Main branch for development. Keep the HEAD of the latest source code for the next release. Sometimes called an "integration branch," this is also the source for automated builds. |
| feature/xxxx | Branches for implementing major features. Create a Feature branch for each task, such as implementing a feature or fixing a bug. Basically, derive from develop and merge into develop. I will eventually delete it when I'm done using it. |
| release-\* | Branches for preparing for release. By creating a Release branch, you can separate code for release preparation and development for the next version. Be sure to derive from develop and merge into master and develop. I will eventually delete it when I'm done using it. |
| hotfix-\* | Branch for making emergency fixes to released software. Fixes in this branch are released immediately, so the hotfix branch is merged into the master branch that manages releases. I will eventually delete it when I'm done using it. |

## References

- [FeatureBranch - Martin Fowler](https://martinfowler.com/bliki/FeatureBranch.html)
