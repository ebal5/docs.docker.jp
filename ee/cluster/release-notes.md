---
title: Docker Cluster リリースノート
description: Learn about the new features, bug fixes, and breaking changes for Docker Cluster.
keywords: cluster, whats new, release notes
---

>{% include enterprise_label_shortform.md %}

{% comment %}
This page provides information about Docker Cluster versions. 
{% endcomment %}
このページでは Docker Cluster の各バージョン情報を示します。

{% comment %}
# Version 1
{% endcomment %}
{: #version-1 }
# バージョン 1


## 1.2.0
(2019-10-8)

### Features

- Added new env variable type which allows users to supply cluster variable values as an environment variable (DCIS-509)

### Fixes

- Fixed an issue where errors in the cluster commands would return exit code of 0 (DCIS-508)

- New error message displayed when a docker login is required:
```Checking for licenses on Docker Hub
   Error: no Hub login info found; please run 'docker login' first
```

{% comment %}
## Version 1.1.0 
(2019-09-03)
{% endcomment %}
{: #version-110 }
## バージョン 1.1.0 
(2019-09-03)

{% comment %}
### What's new
{% endcomment %}
{: #whats-new }
### 新機能

{% comment %}
* Support for Azure cloud provider
* Support for RHEL 8 operating system
{% endcomment %}
* Azure cloud プロバイダーへの対応。
* オペレーティングシステム RHEL 8 への対応。

{% comment %}
### Bug Fixes
{% endcomment %}
{: #bug-fixes }
### バグフィックス

{% comment %}
* Container file permissions on linux (DCIS-346)
* License check failure with non-Docker subscriptions (DCIS-465)
{% endcomment %}
* linux 上での Container ファイルのパーミッション。(DCIS-346)
* License check failure with non-Docker subscriptions (DCIS-465)

{% comment %}
## Version 1.0.1 
(2019-07-19)
{% endcomment %}
{: #version-101 }
## バージョン 1.0.1 
(2019-07-19)

{% comment %}
### What's new
{% endcomment %}
{: #whats-new-1 }
### 新機能

{% comment %}
* Support for SLES 12.4
* Support for Windows Server 2016
{% endcomment %}
* SLES 12.4 への対応。
* Windows Server 2016 への対応。

{% comment %}
## Version 1.0 
(2019-06-25)
{% endcomment %}
{: #version-10 }
## バージョン 1.0 
(2019-06-25)

{% comment %}
First major release.
{% endcomment %}
メジャーリリース。



























































