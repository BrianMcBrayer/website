---
linktitle: 发行版本历史
title: 发行版本
type: docs
---
<!-- 
linktitle: Release History
title: Releases
type: docs
-->


<!-- overview -->

<!-- 
The Kubernetes project maintains release branches for the most recent three minor releases ({{< skew currentVersion >}}, {{< skew currentVersionAddMinor -1 >}}, {{< skew currentVersionAddMinor -2 >}}).  Kubernetes 1.19 and newer receive [approximately 1 year of patch support](/releases/patch-releases/#support-period). Kubernetes 1.18 and older received approximately 9 months of patch support.
-->
Kubernetes 项目维护最近三个次要版本（{{< skew currentVersion >}}、{{< skew currentVersionAddMinor -1 >}}、{{< skew currentVersionAddMinor -2 >}}）的发布分支。
Kubernetes 1.19 和更新版本获得[大约 1 年的补丁支持](/zh/releases/patch-releases/#support-period)。
Kubernetes 1.18 及更早版本获得了大约 9 个月的补丁支持周期。

<!-- 
Kubernetes versions are expressed as **x.y.z**,
where **x** is the major version, **y** is the minor version, and **z** is the patch version, following [Semantic Versioning](https://semver.org/) terminology.
More information in the [version skew policy](/releases/version-skew-policy/) document.
-->

Kubernetes 版本表示为 **x.y.z**，
其中 **x** 是主要版本，**y** 是次要版本，**z** 是补丁版本，遵循[语义版本控制](https://semver.org/)术语。

更多信息在[版本偏差策略](/zh/releases/version-skew-policy/)文档中。

<!-- body -->

<!-- ## Release History -->
## 发行版本历史

{{< release-data >}}

<!-- ## Upcoming Release -->
## 未来的发行版本

<!-- 
Check out the [schedule](https://github.com/kubernetes/sig-release/tree/master/releases/release-{{< skew  currentVersionAddMinor 1 >}}) for the upcoming **{{< skew  currentVersionAddMinor 1 >}}** Kubernetes release!
-->
查看[时间表](https://github.com/kubernetes/sig-release/tree/master/releases/release-{{< skew currentVersionAddMinor 1 >}})，
Kubernetes **{{< skew nextMinorVersion >}}** 版本即将发行！

<!-- ## Helpful Resources -->
## 有用的资源
