---
title: "オーケストレーション"
keywords: orchestration, deploy, kubernetes, swarm,
description: Get oriented on some basics of Docker and install Docker Desktop.
---

{% comment %}
The portability and reproducibility of a containerized process mean we have an opportunity to move and scale our containerized applications across clouds and datacenters. Containers effectively guarantee that those applications run the same way anywhere, allowing us to quickly and easily take advantage of all these environments. Furthermore, as we scale our applications up, we'll want some tooling to help automate the maintenance of those applications, able to replace failed containers automatically, and manage the rollout of updates and reconfigurations of those containers during their lifecycle.
{% endcomment %}
コンテナー化プロセスが可搬性や再生産性に優れているということは、つまりコンテナー化アプリケーションをクラウドやデータセンター間において移行させスケール変更が容易になるということです。
コンテナーというものは、アプリケーションをどこで動作させても同じように動くことが保証されるものであり、すべての環境下においてその性能をすばやく簡単に利用することができます。
アプリケーションをスケールアップするときには、アプリケーションのメンテナンスを自動化するツールが必要になってきます。
また動作不良をおこしたコンテナーを自動的に置き換えたり、コンテナー稼働中でのアップデートの適用や再設定の管理ができることが求められます。

{% comment %}
Tools to manage, scale, and maintain containerized applications are called _orchestrators_, and the most common examples of these are _Kubernetes_ and _Docker Swarm_. Development environment deployments of both of these
orchestrators are provided by Docker Desktop, which we'll use throughout
this guide to create our first orchestrated, containerized application.
{% endcomment %}
コンテナー化アプリケーションを管理しスケール変更や保守を行うツールのことを **オーケストレーター（orchestrator）** と呼びます。
その具体例が **Kubernetes** と **Docker Swarm** です。
この２つのオーケストレーターによる開発環境デプロイ機能は、Docker Desktop によって提供されています。
本ガイドでは、これからオーケストレーションされた、コンテナー化アプリケーションを生成するために、このオーケストレーターを利用していきます。

{% comment %}
The advanced modules teach you how to:
{% endcomment %}
この高度なモジュールを利用して以下のことを学んでいきます。

{% comment %}
1. [Set up and use a Kubernetes environment on your development machine](kube-deploy.md)
2. [Set up and use a Swarm environment on your development machine](swarm-deploy.md)
{% endcomment %}
1. [開発環境上にて Kubernetes 環境を構築して利用する](kube-deploy.md)
2. [開発環境上にて Swarm 環境を構築して利用する](swarm-deploy.md)

{% comment %}
## Enable Kubernetes
{% endcomment %}
{: #enable-kubernetes }
## Kubernetes の有効化

{% comment %}
Docker Desktop will set up Kubernetes for you quickly and easily. Follow the setup and validation instructions appropriate for your operating system:
{% endcomment %}
Docker Desktop では Kubernetes をすばやく簡単に設定することができます。
利用しているオペレーティングシステムに合わせて、以下の設定および確認手順を進めてください。

<ul class="nav nav-tabs">
  <li class="active"><a data-toggle="tab" href="#kubeosx">Mac</a></li>
  <li><a data-toggle="tab" href="#kubewin">Windows</a></li>
</ul>
<div class="tab-content">
  <div id="kubeosx" class="tab-pane fade in active">
{% capture local-content %}

### Mac

{% comment %}
1.  After installing Docker Desktop, you should see a Docker icon in your menu bar. Click on it, and navigate to **Preferences** > **Kubernetes**.
{% endcomment %}
1.  Docker Desktop のインストール後には、メニューバー上に Docker アイコンが表示されます。
    これをクリックして **Preferences** > **Kubernetes** を実行します。

{% comment %}
2.  Check the checkbox labeled **Enable Kubernetes**, and click **Apply & Restart**. Docker Desktop will automatically set up Kubernetes for you. You'll know that Kubernetes has been successfully enabled when you see a green light beside 'Kubernetes _running_' in the Preferences menu.
{% endcomment %}
2.  **Enable Kubernetes** と書かれたチェックボックスにチェックを入れて **Apply & Restart** をクリックします。
    Docker Desktop が自動的に Kubernetes を設定します。
    Preferences メニューにおいて 'Kubernetes _running_' と書かれた横にグリーンライトが表示されるので、これによって Kubernetes が正常に有効化されたことがわかります。

{% comment %}
3.  In order to confirm that Kubernetes is up and running, create a text file called `pod.yaml` with the following content:
{% endcomment %}
3.  Kubernetes が起動していることを確認するために、`pod.yaml` というテキストファイルを生成し、その内容を以下のようにします。

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: demo
    spec:
      containers:
      - name: testpod
        image: alpine:3.5
        command: ["ping", "8.8.8.8"]
    ```

    {% comment %}
    This describes a pod with a single container, isolating a simple ping to 8.8.8.8.
    {% endcomment %}
    これは１つのコンテナー内に１つのポッド（pod）を定義し、単純に 8.8.8.8 への ping を行います。

{% comment %}
4.  In a terminal, navigate to where you created `pod.yaml` and create your pod:
{% endcomment %}
4.  端末にて `pod.yaml` を生成したディレクトリに移動しポッドを生成します。

    ```shell
    kubectl apply -f pod.yaml
    ```

{% comment %}
5.  Check that your pod is up and running:
{% endcomment %}
5.  ポッドが起動して実行中であることを確認します。

    ```shell
    kubectl get pods
    ```

    {% comment %}
    You should see something like:
    {% endcomment %}
    たとえば以下のような表示になります。

    ```shell
    NAME      READY     STATUS    RESTARTS   AGE
    demo      1/1       Running   0          4s
    ```

{% comment %}
6.  Check that you get the logs you'd expect for a ping process:
{% endcomment %}
6.  ping 処理によって得られるログを確認してみます。

    ```shell
    kubectl logs demo
    ```

    {% comment %}
    You should see the output of a healthy ping process:
    {% endcomment %}
    ping プロセスが正常に実行されていれば、以下のような出力になります。

    ```shell
    PING 8.8.8.8 (8.8.8.8): 56 data bytes
    64 bytes from 8.8.8.8: seq=0 ttl=37 time=21.393 ms
    64 bytes from 8.8.8.8: seq=1 ttl=37 time=15.320 ms
    64 bytes from 8.8.8.8: seq=2 ttl=37 time=11.111 ms
    ...
    ```

{% comment %}
7.  Finally, tear down your test pod:
{% endcomment %}
7.  作業を終えたらテスト用ポッドを削除します。

    ```shell
    kubectl delete -f pod.yaml
    ```

{% endcapture %}
{{ local-content | markdownify }}

</div>
<div id="kubewin" class="tab-pane fade" markdown="1">
{% capture localwin-content %}

### Windows

{% comment %}
1.  After installing Docker Desktop, you should see a Docker icon in your system tray. Right-click on it, and navigate **Settings** > **Kubernetes**.
{% endcomment %}
1.  Docker Desktop のインストール後には、システムトレイ上に Docker アイコンが表示されます。
    アイコンを右クリックして **Settings** > **Kubernetes** を実行します。

{% comment %}
2.  Check the checkbox labeled **Enable Kubernetes**, and click **Apply & Restart**. Docker Desktop will automatically set up Kubernetes for you. You'll know that Kubernetes has been successfully enabled when you see a green light beside 'Kubernetes _running_' in the **Settings** menu.
{% endcomment %}
2.  **Enable Kubernetes** と書かれたチェックボックスにチェックを入れて **Apply & Restart** をクリックします。
    Docker Desktop が自動的に Kubernetes を設定します。
    **Settings** メニューにおいて 'Kubernetes _running_' と書かれた横にグリーンライトが表示されるので、これによって Kubernetes が正常に有効化されたことがわかります。

{% comment %}
3.  In order to confirm that Kubernetes is up and running, create a text file called `pod.yaml` with the following content:
{% endcomment %}
3.  Kubernetes が起動していることを確認するために、`pod.yaml` というテキストファイルを生成し、その内容を以下のようにします。

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: demo
    spec:
      containers:
      - name: testpod
        image: alpine:3.5
        command: ["ping", "8.8.8.8"]
    ```

    {% comment %}
    This describes a pod with a single container, isolating a simple ping to 8.8.8.8.
    {% endcomment %}
    これは１つのコンテナー内に１つのポッド（pod）を定義し、単純に 8.8.8.8 への ping を行います。

{% comment %}
4.  In PowerShell, navigate to where you created `pod.yaml` and create your pod:
{% endcomment %}
4.  PowerShell において `pod.yaml` を生成したディレクトリに移動しポッドを生成します。

    ```shell
    kubectl apply -f pod.yaml
    ```

{% comment %}
5.  Check that your pod is up and running:
{% endcomment %}
5.  ポッドが起動して実行中であることを確認します。

    ```shell
    kubectl get pods
    ```

    {% comment %}
    You should see something like:
    {% endcomment %}
    たとえば以下のような表示になります。

    ```shell
    NAME      READY     STATUS    RESTARTS   AGE
    demo      1/1       Running   0          4s
    ```

{% comment %}
6.  Check that you get the logs you'd expect for a ping process:
{% endcomment %}
6.  ping 処理によって得られるログを確認してみます。

    ```shell
    kubectl logs demo
    ```

    {% comment %}
    You should see the output of a healthy ping process:
    {% endcomment %}
    ping プロセスが正常に実行されていれば、以下のような出力になります。

    ```shell
    PING 8.8.8.8 (8.8.8.8): 56 data bytes
    64 bytes from 8.8.8.8: seq=0 ttl=37 time=21.393 ms
    64 bytes from 8.8.8.8: seq=1 ttl=37 time=15.320 ms
    64 bytes from 8.8.8.8: seq=2 ttl=37 time=11.111 ms
    ...
    ```

{% comment %}
7.  Finally, tear down your test pod:
{% endcomment %}
7.  作業を終えたらテスト用ポッドを削除します。

    ```shell
    kubectl delete -f pod.yaml
    ```

{% endcapture %}
{{ localwin-content | markdownify }}
</div>
<hr>
</div>

{% comment %}
## Enable Docker Swarm
{% endcomment %}
{: #enable-docker-swarm }
## Docker Swarm の有効化

{% comment %}
Docker Desktop runs primarily on Docker Engine, which has everything you need to run a Swarm built in. Follow the setup and validation instructions appropriate for your operating system:
{% endcomment %}
Docker Desktop は主に Docker Engine 上において動作します。
これがあれば、ビルトインの Swarm を動作させる機能がすべて含まれています。
利用オペレーティングシステムに応じて、以下の設定および確認手順に従ってください。

<ul class="nav nav-tabs">
  <li class="active"><a data-toggle="tab" href="#swarmosx">Mac</a></li>
  <li><a data-toggle="tab" href="#swarmwin">Windows</a></li>
</ul>
<div class="tab-content">
  <div id="swarmosx" class="tab-pane fade in active">
{% capture local-content %}

### Mac

{% comment %}
1.  Open a terminal, and initialize Docker Swarm mode:
{% endcomment %}
1.  端末を開いて Docker Swarm モードを初期化します。

    ```shell
    docker swarm init
    ```

    {% comment %}
    If all goes well, you should see a message similar to the following:
    {% endcomment %}
    正常に処理されると、以下のようなメッセージが出力されます。

    ```shell
    Swarm initialized: current node (tjjggogqpnpj2phbfbz8jd5oq) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join --token SWMTKN-1-3e0hh0jd5t4yjg209f4g5qpowbsczfahv2dea9a1ay2l8787cf-2h4ly330d0j917ocvzw30j5x9 192.168.65.3:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```

{% comment %}
2.  Run a simple Docker service that uses an alpine-based filesystem, and isolates a ping to 8.8.8.8:
{% endcomment %}
2.  単純な Docker サービスを実行します。
    これは alpine ベースのファイルシステムを利用するものであり、8.8.8.8 への ping プロセスを実行します。

    ```shell
    docker service create --name demo alpine:3.5 ping 8.8.8.8
    ```

{% comment %}
3.  Check that your service created one running container:
{% endcomment %}
3.  サービスから 1 つのコンテナーが生成されていることを確認します。

    ```shell
    docker service ps demo
    ```

    {% comment %}
    You should see something like:
    {% endcomment %}
    以下のような出力が得られます。

    ```shell
    ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
    463j2s3y4b5o        demo.1              alpine:3.5          docker-desktop      Running             Running 8 seconds ago
    ```

{% comment %}
4.  Check that you get the logs you'd expect for a ping process:
{% endcomment %}
4.  ログを参照して、期待どおりに ping プロセスが得られていることを確認します。

    ```shell
    docker service logs demo
    ```

    {% comment %}
    You should see the output of a healthy ping process:
    {% endcomment %}
    ping 処理が正常に処理されて以下のように出力されているはずです。

    ```shell
    demo.1.463j2s3y4b5o@docker-desktop    | PING 8.8.8.8 (8.8.8.8): 56 data bytes
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=0 ttl=37 time=13.005 ms
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=1 ttl=37 time=13.847 ms
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=2 ttl=37 time=41.296 ms
    ...
    ```

{% comment %}
5.  Finally, tear down your test service:
{% endcomment %}
5.  最後はテストしたサービスを終了させます。

    ```shell
    docker service rm demo
    ```

{% endcapture %}
{{ local-content | markdownify }}

</div>
<div id="swarmwin" class="tab-pane fade" markdown="1">
{% capture localwin-content %}

### Windows

{% comment %}
1.  Open a powershell, and initialize Docker Swarm mode:
{% endcomment %}
1.  powershell を開いて Docker Swarm モードを初期化します。

    ```shell
    docker swarm init
    ```

    {% comment %}
    If all goes well, you should see a message similar to the following:
    {% endcomment %}
    正常に処理されると、以下のようなメッセージが出力されます。

    ```shell
    Swarm initialized: current node (tjjggogqpnpj2phbfbz8jd5oq) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join --token SWMTKN-1-3e0hh0jd5t4yjg209f4g5qpowbsczfahv2dea9a1ay2l8787cf-2h4ly330d0j917ocvzw30j5x9 192.168.65.3:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```

{% comment %}
2.  Run a simple Docker service that uses an alpine-based filesystem, and isolates a ping to 8.8.8.8:
{% endcomment %}
2.  単純な Docker サービスを実行します。
    これは alpine ベースのファイルシステムを利用するものであり、8.8.8.8 への ping プロセスを実行します。

    ```shell
    docker service create --name demo alpine:3.5 ping 8.8.8.8
    ```

{% comment %}
3.  Check that your service created one running container:
{% endcomment %}
3.  サービスから 1 つのコンテナーが生成されていることを確認します。

    ```shell
    docker service ps demo
    ```

    {% comment %}
    You should see something like:
    {% endcomment %}
    以下のような出力が得られます。

    ```shell
    ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
    463j2s3y4b5o        demo.1              alpine:3.5          docker-desktop      Running             Running 8 seconds ago
    ```

{% comment %}
4.  Check that you get the logs you'd expect for a ping process:
{% endcomment %}
4.  ログを参照して、期待どおりに ping プロセスが得られていることを確認します。

    ```shell
    docker service logs demo
    ```

    {% comment %}
    You should see the output of a healthy ping process:
    {% endcomment %}
    ping 処理が正常に処理されて以下のように出力されているはずです。

    ```shell
    demo.1.463j2s3y4b5o@docker-desktop    | PING 8.8.8.8 (8.8.8.8): 56 data bytes
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=0 ttl=37 time=13.005 ms
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=1 ttl=37 time=13.847 ms
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=2 ttl=37 time=41.296 ms
    ...
    ```

{% comment %}
5.  Finally, tear down your test service:
{% endcomment %}
5.  最後はテストしたサービスを終了させます。

    ```shell
    docker service rm demo
    ```

{% endcapture %}
{{ localwin-content | markdownify }}
</div>
<hr>
</div>

{% comment %}
## Conclusion
{% endcomment %}
{: #conclusion }
## まとめ

{% comment %}
At this point, you've confirmed that you can run simple containerized workloads in Kubernetes and Swarm. The next step will be to write the Kubernetes yaml that describes how to run and manage these containers on Kubernetes.
{% endcomment %}
ここまで、Kubernetes や Swarm を使って単純なコンテナー化アプリが起動できることを確認しました。
次のステップでは Kubernetes 上にてコンテナーの起動と管理を行うための Kubenetes yaml の記述方法を学びます。

{% comment %}
[On to deploying to Kubernetes >>](kube-deploy.md){: class="button outline-btn" style="margin-bottom: 30px; margin-right: 200%"}
{% endcomment %}
[Kubernetes へのデプロイ >>](kube-deploy.md){: class="button outline-btn" style="margin-bottom: 30px; margin-right: 200%"}

{% comment %}
To learn how to write the stack file to help you run and manage containers on Swarm, see [Deploying to Swarm](swarm-deploy.md).
{% endcomment %}
Swarm 上にて、コンテナーの起動と管理を行うためのファイル記述方法については [Swarm へのデプロイ](swarm-deploy.md) を参照してください。

{% comment %}
## CLI references
{% endcomment %}
{: #cli-references }
## CLI リファレンス

{% comment %}
Further documentation for all CLI commands used in this article are available here:
{% endcomment %}
本説明において利用した CLI コマンドの詳細については、以下を参照してください。

- [`kubectl apply`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply)
- [`kubectl get`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get)
- [`kubectl logs`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs)
- [`kubectl delete`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete)
- [`docker swarm init`](https://docs.docker.com/engine/reference/commandline/swarm_init/)
- [`docker service *`](https://docs.docker.com/engine/reference/commandline/service/)
