---
description: Initialize the swarm
keywords: tutorial, cluster management, swarm mode
title: Swarm の生成
notoc: true
---

{% comment %}
After you complete the [tutorial setup](index.md) steps, you're ready
to create a swarm. Make sure the Docker Engine daemon is started on the host
machines.
{% endcomment %}
[チュートリアルでのセットアップ](index.md) 手順をすべて終えたら、Swarm の生成を行います。
ホストマシン上において Docker Engine デーモンが起動済であることを確認してください。

{% comment %}
1.  Open a terminal and ssh into the machine where you want to run your manager
    node. This tutorial uses a machine named `manager1`. If you use Docker Machine,
    you can connect to it via SSH using the following command:
{% endcomment %}
1.  ターミナル画面を開き、マネージャーノードを起動させたいマシンに SSH 接続します。
    このチュートリアルでは `manager1` というマシン名にします。
    Docker Machine を利用している場合は、以下のコマンドを使って SSH 経由で接続することができます。

    ```bash
    $ docker-machine ssh manager1
    ```

{% comment %}
2.  Run the following command to create a new swarm:
{% endcomment %}
2.  以下のコマンドを実行して、新規に Swarm を生成します。

    ```bash
    $ docker swarm init --advertise-addr <MANAGER-IP>
    ```

    {% comment %}
    > **Note**: If you are using Docker Desktop for Mac or Docker Desktop for Windows to test
    > single-node swarm, simply run `docker swarm init` with no arguments. There is no
    > need to specify `--advertise-addr` in this case. To learn more, see the topic
    > on how to
    > [Use Docker Desktop or Mac or Docker Desktop for Windows](index.md#use-docker-desktop-for-mac-or-docker-desktop-for-windows)
    > with Swarm.
    {% endcomment %}
    > **メモ**: Docker Desktop for Mac または Docker Desktop for Windows 利用し単一ノードの Swarm を生成する場合は、引数なしで `docker swarm init` を実行します。
    > このときには `--advertise-addr` を指定する必要はありません。
    > 詳しくは Swarm を使った [Docker Desktop for Mac または Docker Desktop for Windows の利用](index.md#use-docker-desktop-for-mac-or-docker-desktop-for-windows) を参照してください。

    {% comment %}
    In the tutorial, the following command creates a swarm on the `manager1`
    machine:
    In the tutorial, the following command creates a swarm on the `manager1`
    machine:
    {% endcomment %}
    このチュートリアルでは、以下のコマンドにより `manager1` マシン上に Swarm が生成されます。

    ```bash
    $ docker swarm init --advertise-addr 192.168.99.100
    Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join \
        --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
        192.168.99.100:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```

    {% comment %}
    The `--advertise-addr` flag configures the manager node to publish its
    address as `192.168.99.100`. The other nodes in the swarm must be able
    to access the manager at the IP address.
    {% endcomment %}
    `--advertise-addr` フラグはマネージャーノードに対して、アドレス `192.168.99.100` を公開することを指示します。
    Swarm 上の他のノードは、マネージャーに対してこの IP アドレスを使ってアクセスしなければなりません。

    {% comment %}
    The output includes the commands to join new nodes to the swarm. Nodes will
    join as managers or workers depending on the value for the `--token`
    flag.
    {% endcomment %}
    出力メッセージには、Swarm に新たなノードを参加させるためのコマンドが示されています。
    参加させるノードをマネージャーとするかワーカーとするかは、`--token` フラグに与える値によって決まります。

{% comment %}
2.  Run `docker info` to view the current state of the swarm:
{% endcomment %}
2.  `docker info` を実行して、Swarm の現時点での状態を確認してみます。

    ```bash
    $ docker info

    Containers: 2
    Running: 0
    Paused: 0
    Stopped: 2
      ...snip...
    Swarm: active
      NodeID: dxn1zf6l61qsb1josjja83ngz
      Is Manager: true
      Managers: 1
      Nodes: 1
      ...snip...
    ```

{% comment %}
3.  Run the `docker node ls` command to view information about nodes:
{% endcomment %}
3.  `docker node ls` コマンドを実行して、ノードに関する情報を確認します。

    ```bash
    $ docker node ls

    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    dxn1zf6l61qsb1josjja83ngz *  manager1  Ready   Active        Leader

    ```

    {% comment %}
    The `*` next to the node ID indicates that you're currently connected on
    this node.
    {% endcomment %}
    ノード ID の横にある `*` が、現在接続しているノードを表わしています。

    {% comment %}
    Docker Engine swarm mode automatically names the node for the machine host
    name. The tutorial covers other columns in later steps.
    {% endcomment %}
    Docker Engine の Swarm モードはマシンホスト名にちなんで、自動的にノードに名称がつけられます。
    その他の出力カラムに関しては、後のステップにおいて説明します。

{% comment %}
## What's next?
{% endcomment %}
{: #whats-next }
## 次にすることは

{% comment %}
In the next section of the tutorial, we [add two more nodes](add-nodes.md) to
the cluster.
{% endcomment %}
本チュートリアルの次節では、クラスターに対して [さらに 2 つのノードを追加](add-nodes.md) します。
