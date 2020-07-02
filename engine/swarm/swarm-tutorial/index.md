---
description: Getting Started tutorial for Docker Engine swarm mode
keywords: tutorial, cluster management, swarm mode
title: Swarm モードをはじめよう
toc_max: 4
---

{% comment %}
This tutorial introduces you to the features of Docker Engine Swarm mode. You
may want to familiarize yourself with the [key concepts](../key-concepts.md)
before you begin.
{% endcomment %}
このチュートリアルは、Docker Engine の Swarm モード機能を説明するものです。
これをはじめる前に [重要な考え方](../key-concepts.md) を読んでおいてください。

{% comment %}
The tutorial guides you through the following activities:
{% endcomment %}
このチュートリアルでは、以下のような作業を進めていきます。

{% comment %}
* initializing a cluster of Docker Engines in swarm mode
* adding nodes to the swarm
* deploying application services to the swarm
* managing the swarm once you have everything running
{% endcomment %}
* Swarm モード内で Docker Engine のクラスターを初期化します。
* Swarm にノードを追加します。
* Swarm に対してアプリケーションサービスをデプロイします。
* 正常に動作させることができたら Swarm の管理を行います。

{% comment %}
This tutorial uses Docker Engine CLI commands entered on the command line of a
terminal window.
{% endcomment %}
このチュートリアルでは、端末画面上のコマンドラインから Docker Engine CLI コマンドを利用します。

{% comment %}
If you are brand new to Docker, see [About Docker Engine](../../index.md).
{% endcomment %}
Docker を初めて使う場合は、[Docker Engine について](../../index.md) を参照してください。

{% comment %}
## Set up
{% endcomment %}
{: #set-up }
## セットアップ

{% comment %}
To run this tutorial, you need the following:
{% endcomment %}
このチュートリアルを実行するには、以下の項目が必要です。

{% comment %}
* [three Linux hosts which can communicate over a network, with Docker installed](#three-networked-host-machines)
* [Docker Engine 1.12 or later installed](#docker-engine-112-or-newer)
* [the IP address of the manager machine](#the-ip-address-of-the-manager-machine)
* [open ports between the hosts](#open-protocols-and-ports-between-the-hosts)
{% endcomment %}
* [3 つの Linux ホストに Docker がインストールされ、ネットワーク上で互いに通信可能であること。](#three-networked-host-machines)
* [Docker Engine 1.12 またはそれ以降がインストールされていること。](#docker-engine-112-or-newer)
* [マネージャーマシンの IP アドレス。](#the-ip-address-of-the-manager-machine)
* [ホスト間でポートが開放されていること。](#open-protocols-and-ports-between-the-hosts)

{% comment %}
### Three networked host machines
{% endcomment %}
{: #three-networked-host-machines }
### ネットワーク接続された 3 つのホストマシン

{% comment %}
This tutorial requires three Linux hosts which have Docker installed and can
communicate over a network. These can be physical machines, virtual machines,
Amazon EC2 instances, or hosted in some other way. You can even use Docker Machine
from a Linux, Mac, or Windows host. Check out
[Getting started - Swarms](../../../get-started/swarm-deploy.md#prerequisites)
for one possible set-up for the hosts.
{% endcomment %}
このチュートリアルでは、Docker がインストールされ、互いにネットワーク通信が可能である 3 つの Linux ホストを用います。
このホストは物理マシン、仮想マシン、Amazon EC2 インスタンス、その他のホストのいずれでもかまいません。
Docker Machine を利用するのであれば、Linux、Mac、Windows のいずれをホストすることもできます。
[Swarm をはじめよう](../../../get-started/swarm-deploy.md#prerequisites) を確認し、ホストとして設定可能なものを選んでください。

{% comment %}
One of these machines is a manager (called `manager1`) and two of them are
workers (`worker1` and `worker2`).
{% endcomment %}
このマシンのうちの 1 つをマネージャー（`manager1`）とし、残りの 2 つをワーカー（`worker1` と `worker2`）とします。


{% comment %}
>**Note**: You can follow many of the tutorial steps to test single-node swarm
as well, in which case you need only one host. Multi-node commands do not
work, but you can initialize a swarm, create services, and scale them.
{% endcomment %}
>**メモ**: このチュートリアルに示す手順の多くは、単一ノードの Swarm を確認する際にも用いることができます。
> その場合は、ホストはただ 1 つあれば十分です。
> 複数ノード用コマンドは動作しませんが、Swarm の初期化、サービス生成、スケール変更は行うことができます。

{% comment %}
###  Docker Engine 1.12 or newer
{% endcomment %}
{: #docker-engine-112-or-newer }
###  Docker Engine 1.12 またはそれ以降

{% comment %}
This tutorial requires Docker Engine 1.12 or newer on each of the host machines.
Install Docker Engine and verify that the Docker Engine daemon is running on
each of the machines. You can get the latest version of Docker Engine as
follows:
{% endcomment %}
このチュートリアルのホストマシンには、いずれも Docker Engine 1.12 またはそれ以降がインストールされていることが必要です。
適切な Docker Engine をインストールして Docker Engine デーモンが各マシンにおいて実行されていることを確認してください。
Docker Engine の最新版は以下から入手することができます。

{% comment %}
* [install Docker Engine on Linux machines](#install-docker-engine-on-linux-machines)
{% endcomment %}
* [Linux マシンへの Docker Engine インストール](#install-docker-engine-on-linux-machines)

{% comment %}
* [use Docker Desktop for Mac or Docker Desktop for Windows](#use-docker-desktop-for-mac-or-docker-desktop-for-windows)
{% endcomment %}
* [Docker Desktop for Mac または Docker Desktop for Windows の利用](#use-docker-desktop-for-mac-or-docker-desktop-for-windows)

{% comment %}
#### Install Docker Engine on Linux machines
{% endcomment %}
{: #install-docker-engine-on-linux-machines }
#### Linux マシンへの Docker Engine インストール

{% comment %}
If you are using Linux based physical computers or cloud-provided computers as
hosts, simply follow the [Linux install instructions](../../install/index.md)
for your platform. Spin up the three machines, and you are ready. You can test both
single-node and multi-node swarm scenarios on Linux machines.
{% endcomment %}
ホストとして利用するものが Linux ベースの物理コンピューターやクラウドコンピューターである場合は、各プラットフォームに合わせて単純に [Linux インストール手順](../../install/index.md) に従います。
3 つのマシンを設定したら準備が整いました。
Linux マシン上では単一ノード、複数ノードのいずれのシナリオでも作業していくことができます。

{% comment %}
#### Use Docker Desktop for Mac or Docker Desktop for Windows
{% endcomment %}
{: #use-docker-desktop-for-mac-or-docker-desktop-for-windows }
#### Docker Desktop for Mac または Docker Desktop for Windows の利用

{% comment %}
Alternatively, install the latest [Docker Desktop for Mac](../../../docker-for-mac/index.md) or
[Docker Desktop for Windows](../../../docker-for-windows/index.md) application on one
computer. You can test both single-node and multi-node swarm from this computer,
but you need to use Docker Machine to test the multi-node scenarios.
{% endcomment %}
別の方法として、手元のコンピューターに最新の [Docker Desktop for Mac](../../../docker-for-mac/index.md) または [Docker Desktop for Windows](../../../docker-for-windows/index.md) をインストールして利用することもできます。
このコンピューターからは、単一ノード、複数ノードのいずれの Swarm でもテストすることができます。
ただし複数ノードの例をテストするためには Docker Machine が必要です。

{% comment %}
* You can use Docker Desktop for Mac or Windows to test _single-node_ features of swarm
mode, including initializing a swarm with a single node, creating services,
and scaling services. Docker "Moby" on Hyperkit (Mac) or Hyper-V (Windows)
serve as the single swarm node.
{% endcomment %}
* Docker Desktop for Mac や Docker Desktop for Windows は、Swarm モードにおける **単一ノード** の確認や、単一ノードからなる Swarm の初期化、サービス生成、サービスのスケーリングを行うことができます。
Hyperkit (Mac) 上、または Hyper-V (Windows) 上の Docker "Moby" が単一 Swarm ノードを提供します。

<p />

{% comment %}
* Currently, you cannot use Docker Desktop for Mac or Docker Desktop for Windows alone to test a
_multi-node_ swarm. However, you can use the included version of
[Docker Machine](../../../machine/overview.md) to create the swarm nodes (see
[Get started with Docker Machine and a local VM](../../../machine/get-started.md)), then
follow the tutorial for all multi-node features. For this scenario, you run
commands from a Docker Desktop for Mac or Docker Desktop for Windows host, but that Docker host itself is
_not_ participating in the swarm. After you create the nodes, you can run all
swarm commands as shown from the Mac terminal or Windows PowerShell with
Docker Desktop for Mac or Docker Desktop for Windows running.
{% endcomment %}
* 現時点において Docker Desktop for Mac または Docker Desktop for Windows を単独で利用するだけでは、**複数ノード** Swarm を扱うことはできません。
しかし [Docker Machine](../../../machine/overview.md) が含まれているバージョンを利用すれば、Swarm ノードを生成することができます（[Docker Machine とローカル VM をはじめよう](../../../machine/get-started.md) を参照してください）。
そしてこのチュートリアルにに示す、複数ノードの操作を行うことができます。
このシナリオの場合、コマンドの実行は Docker Desktop for Mac または Docker Desktop for Windows のホストから行います。
ただし Docker ホストそのものは Swarm に参加しているわけでは **ありません**。
全ノードを生成した後は、Docker Desktop for Mac における Mac ターミナル、Docker Desktop for Windows における Windows PowerShell から、説明している Swarm コマンドはすべて実行することができます。

{% comment %}
### The IP address of the manager machine
{% endcomment %}
{: #the-ip-address-of-the-manager-machine }
### マネージャーマシンの IP アドレス

{% comment %}
The IP address must be assigned to a network interface available to the host
operating system. All nodes in the swarm need to connect to the manager at
the IP address.
{% endcomment %}
ホストオペレーティングシステムが利用するネットワークインターフェースに対しては、IP アドレスを割り振る必要があります。
Swarm 上のノードはすべて、マネージャーに対して IP アドレスにより接続します。

{% comment %}
Because other nodes contact the manager node on its IP address, you should use a
fixed IP address.
{% endcomment %}
他のノードからもマネージャーノードの IP アドレスを通じて接続するため、IP アドレスは固定しておく必要があります。

{% comment %}
You can run `ifconfig` on Linux or macOS to see a list of the
available network interfaces.
{% endcomment %}
Linux や macOS では `ifconfig` を実行すれば、利用可能なネットワークインターフェースの一覧を確認できます。

{% comment %}
If you are using Docker Machine, you can get the manager IP with either
`docker-machine ls` or `docker-machine ip <MACHINE-NAME>` &#8212; for example,
`docker-machine ip manager1`.
{% endcomment %}
Docker Machine を利用している場合は、`docker-machine ls` または `docker-machine ip <マシン名>`（たとえば `docker-machine ip manager1`） によってマネージャーの IP アドレスを確認することができます。

{% comment %}
The tutorial uses `manager1` : `192.168.99.100`.
{% endcomment %}
このチュートリアルでは `manager1` が `192.168.99.100` であるものとします。

{% comment %}
### Open protocols and ports between the hosts
{% endcomment %}
{: #open-protocols-and-ports-between-the-hosts }
### ホスト間でのプロトコルとポートの開放

{% comment %}
The following ports must be available. On some systems, these ports are open by default.
{% endcomment %}
以下のポートの利用が必要です。
システムの中には、デフォルトでこれらのポートはすでに開放されているものがあります。

{% comment %}
* **TCP port 2377** for cluster management communications
* **TCP** and **UDP port 7946** for communication among nodes
* **UDP port 4789** for overlay network traffic
{% endcomment %}
* **TCP ポートの 2377**、クラスター管理における通信のため
* **TCP** および **UDP ポートの 7946**、ノード間の通信のため
* **UDP ポートの 4789**、オーバーレイネットワークのトラフィックのため

{% comment %}
If you plan on creating an overlay network with encryption (`--opt encrypted`),
you also need to ensure **ip protocol 50** (**ESP**) traffic is allowed.
{% endcomment %}
オーバーレイネットワークを暗号化（`--opt encrypted`）込みで生成する場合は、**ip protocol 50** (**ESP**) を許可しておく必要があります。

{% comment %}
## What's next?
{% endcomment %}
{: #whats-next }
## 次にすることは

{% comment %}
After you have set up your environment, you are ready to [create a swarm](create-swarm.md).
{% endcomment %}
環境を整えたら、次に [Swarm の生成](create-swarm.md) を行います。
