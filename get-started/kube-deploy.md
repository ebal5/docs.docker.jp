---
title: "Kubernetes へのデプロイ"
keywords: kubernetes, pods, deployments, kubernetes services
description: Kubernetes 上での簡単なアプリケーションを構築してデプロイする方法を学びます。
---

{% comment %}
## Prerequisites
{% endcomment %}
{: #prerequisites }
## 前提条件

{% comment %}
- Download and install Docker Desktop as described in [Orientation and setup](index.md).
- Work through containerizing an application in [Part 2](part2.md).
- Make sure that Kubernetes is enabled on your Docker Desktop:
  - **Mac**: Click the Docker icon in your menu bar, navigate to **Preferences** and make sure there's a green light beside 'Kubernetes'.
  - **Windows**: Click the Docker icon in the system tray and navigate to **Settings** and make sure there's a green light beside 'Kubernetes'.
{% endcomment %}
- [概要](index.md) にて説明している Docker Desktop をダウンロードしインストールしていること。
- [クイックスタート 2 部](part2.md) においてアプリケーションのコンテナー化を一通り学んでいること。
- Docker Desktop 上において Kubernetes が有効になっていること。
  - **Mac**: メニューバーの Docker アイコンをクリック、**Preferences** を実行し 'Kubernetes' の横がグリーンに点灯していること。
  - **Windows**: システムトレイの Docker アイコンをクリック、**Settings** を実行し 'Kubernetes' の横がグリーンに点灯していること。

  {% comment %}
  If Kubernetes isn't running, follow the instructions in [Orchestration](orchestration.md) of this tutorial to finish setting it up.
  {% endcomment %}
  Kubernetes が起動していない場合は、本チュートリアルの [概要](orchestration.md) に示す手順に従って設定を行ってください。

{% comment %}
## Introduction
{% endcomment %}
{: #introduction }
## はじめに

{% comment %}
Now that we've demonstrated that the individual components of our application run as stand-alone containers, it's time to arrange for them to be managed by an orchestrator like Kubernetes. Kubernetes provides many tools for scaling, networking, securing and maintaining your containerized applications, above and beyond the abilities of containers themselves.
{% endcomment %}
ここまでにアプリケーションの個々のコンポーネントが、スタンドアロンなコンテナーとして実行される様子を見てきました。
ここからはそれらを整理して、Kubernetes のようなオーケストレーターによって管理してくことにします。
Kubernetes はコンテナー化されたアプリケーションに対して、そのコンテナーの能力を超えて、スケール、ネットワーク、セキュリティ、保守を行うツールを数多く提供します。

{% comment %}
In order to validate that our containerized application works well on Kubernetes, we'll use Docker Desktop's built in Kubernetes environment right on our development machine to deploy our application, before handing it off to run on a full Kubernetes cluster in production. The Kubernetes environment created by Docker Desktop is _fully featured_, meaning it has all the Kubernetes features your app will enjoy on a real cluster, accessible from the convenience of your development machine.
{% endcomment %}
Kubernetes 上においてコンテナー化アプリケーションが適正に動作していることを確認するために、まずは開発マシン上にて Kubernetes 環境化の Docker Desktop を用いてアプリケーションデプロイを行います。
完全な Kubernetes クラスターによる本番環境での稼動は、その次に扱っていくことにします。
Docker Desktop によって構築されている Kubernetes 環境は **完全な機能** を有しています。
つまりそこには Kubernetes の全機能が含まれていて、実際のクラスター上でアプリを動作させることができ、開発マシンから容易にアクセスすることができます。

{% comment %}
## Describing apps using Kubernetes YAML
{% endcomment %}
{: #describing-apps-using-kubernetes-yaml }
## Kubernetes YAML を用いたアプリ記述

{% comment %}
All containers in Kubernetes are scheduled as _pods_, which are groups of co-located containers that share some resources. Furthermore, in a realistic application we almost never create individual pods; instead, most of our workloads are scheduled as _deployments_, which are scalable groups of pods maintained automatically by Kubernetes. Lastly, all Kubernetes objects can and should be described in manifests called _Kubernetes YAML_ files. These YAML files describe all the components and configurations of your Kubernetes app, and can be used to easily create and destroy your app in any Kubernetes environment.
{% endcomment %}
Kubernetes 内の全コンテナーは **ポッド**（pod）としてスケジューリングされます。
このポッドとは、リソースを共有する複数コンテナーのグループのことです。
なお現実のアプリケーションにおいては、ポッドを１つずつ生成するようなことはしません。
たいていのアプリケーション開発では **deployments** としてスケジューリングされます。
これは Kubernetes によって自動的に管理される、スケール変更可能なポッドのグループのことです。
そして Kubernetes によるオブジェクトは、すべて **Kubernetes YAML** ファイルと呼ばれるファイル内に記述されます。
この YAML ファイルに Kubernetes アプリのコンポーネントや設定をすべて記述します。
こうして Kubernetes 環境内でのアプリケーションの生成と削除が容易にできるようになります。

{% comment %}
1.  You already wrote a very basic Kubernetes YAML file in the Orchestration overview part of this tutorial. Now, let's write a slightly more sophisticated YAML file to run and manage our bulletin board. Place the following in a file called `bb.yaml`:
{% endcomment %}
1.  本チュートリアルのオーケストレーション概要の部において、すでに基本的な Kubernetes YAML ファイルを作り出しました。
    そこでこの YAML ファイルをもう少し洗練されたものにして、掲示板アプリの実行と管理を行っていきます。
    `bb.yaml` というファイルに以下の内容を記述してください。

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: bb-demo
      namespace: default
    spec:
      replicas: 1
      selector:
        matchLabels:
          bb: web
      template:
        metadata:
          labels:
            bb: web
        spec:
          containers:
          - name: bb-site
            image: bulletinboard:1.0
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: bb-entrypoint
      namespace: default
    spec:
      type: NodePort
      selector:
        bb: web
      ports:
      - port: 8080
        targetPort: 8080
        nodePort: 30001
    ```

    {% comment %}
    In this Kubernetes YAML file, we have two objects, separated by the `---`:
    - A `Deployment`, describing a scalable group of identical pods. In this case, you'll get just one `replica`, or copy of your pod, and that pod (which is described under the `template:` key) has just one container in it, based off of your `bulletinboard:1.0` image from the previous step in this tutorial.
    - A `NodePort` service, which will route traffic from port 30001 on your host to port 8080 inside the pods it routes to, allowing you to reach your bulletin board from the network.
    {% endcomment %}
    この Kubernetes YAML ファイルには二つのオブジェクトを定義しています。
    それらは `---` で区切られています。
    - `Deployment` は、スケール変更可能な同一ポッドのグループを表わします。
      この例では `replica` すなわちポッドのコピーを１つだけ用意します。
      そしてこのポッドは、その中に１つだけコンテナーを持ちます（このことは `template:` キー配下により示されます）。
      ベースとするイメージは、本チュートリアルの以前の手順にて利用した `bulletinboard:1.0` です。
    - `NodePort` サービスは、ホストのポート 30001 からのトラフィックを処理して、ポッド内の 8080 ポートへ接続します。
      こうしてネットワーク上から掲示板アプリへアクセスできるようになります。

    {% comment %}
    Also, notice that while Kubernetes YAML can appear long and complicated at first, it almost always follows the same pattern:
    - The `apiVersion`, which indicates the Kubernetes API that parses this object
    - The `kind` indicating what sort of object this is
    - Some `metadata` applying things like names to your objects
    - The `spec` specifying all the parameters and configurations of your object.
    {% endcomment %}
    また Kubernetes YAML は、はじめのうちは長く複雑に見えがちですが、たいていは同一パターンを持っています。
    - `apiVersion`、オブジェクトを扱うための Kubernetes API を表わします。
    - `kind` は、このオブジェクトがどういう種類のものかを表わします。
    - `metadata` は、たとえば name などの情報をオブジェクトに適用します。
    - `spec` は、オブジェクトに対するパラメーターや設定項目を指定します。

{% comment %}
## Deploy and check your application
{% endcomment %}
{: #deploy-and-check-your-application }
## アプリケーションのデプロイと確認

{% comment %}
1.  In a terminal, navigate to where you created `bb.yaml` and deploy your application to Kubernetes:
{% endcomment %}
1.  端末にて `bb.yaml` を生成したディレクトリに移動し、Kubernetes にアプリケーションをデプロイします。

    ```shell
    kubectl apply -f bb.yaml
    ```

    {% comment %}
    you should see output that looks like the following, indicating your Kubernetes objects were created successfully:
    {% endcomment %}
    以下のような出力が得られます。
    これは Kubernetes オブジェクトが正常に生成されたことを示しています。

    ```shell
    deployment.apps/bb-demo created
    service/bb-entrypoint created
    ```

{% comment %}
2.  Make sure everything worked by listing your deployments:
{% endcomment %}
2.  デプロイ結果の一覧を表示して、正常にすべてが動作していることを確認します。

    ```shell
    kubectl get deployments
    ```

    {% comment %}
    if all is well, your deployment should be listed as follows:
    {% endcomment %}
    すべて正常であれば、以下のようにデプロイ結果が一覧表示されます。

    ```shell
    NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    bb-demo   1         1         1            1           48s
    ```

    {% comment %}
    This indicates all one of the pods you asked for in your YAML are up and running. Do the same check for your services:
    {% endcomment %}
    YAML ファイル内に指定したポッドの個々が、すべて起動していることがわかります。
    同様にサービスに対しても確認します。

    ```shell
    kubectl get services

    NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    bb-entrypoint   NodePort    10.106.145.116   <none>        8080:30001/TCP   53s
    kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP          138d
    ```

    {% comment %}
    In addition to the default `kubernetes` service, we see our `bb-entrypoint` service, accepting traffic on port 30001/TCP.
    {% endcomment %}
    デフォルトの `kubernetes` サービスに加えて `bb-entrypoint` サービスが表示されます。
    このサービスはトラフィックをポート 30001/TCP から受け付けます。

{% comment %}
3.  Open a browser and visit your bulletin board at `localhost:30001`; you should see your bulletin board, the same as when we ran it as a stand-alone container in [Part 2](part2.md) of the Quickstart tutorial.
{% endcomment %}
3.  ブラウザーを開いて `localhost:30001` にアクセスし掲示板アプリを開きます。
    ここでも掲示板アプリを見ることができます。
    これはクイックスタートチュートリアルの [2 部](part2.md) において、スタンドアロンなコンテナー上に起動させたアプリと同一のものです。

{% comment %}
4.  Once satisfied, tear down your application:
{% endcomment %}
4.  結果を確認できたらアプリケーションを削除します。

    ```shell
    kubectl delete -f bb.yaml
    ```

{% comment %}
## Conclusion
{% endcomment %}
{: #conclusion }
## まとめ

{% comment %}
At this point, we have successfully used Docker Desktop to deploy our application to a fully-featured Kubernetes environment on our development machine. We haven't done much with Kubernetes yet, but the door is now open; you can begin adding other components to your app and taking advantage of all the features and power of Kubernetes, right on your own machine.
{% endcomment %}
ここまで、開発マシン上にて Kubernetes の全機能を実現した環境上に、Docker Desktop を用いてアプリケーションをデプロイすることに成功しました。
Kubernetes を使った作業には、まだまだ多くのことがあります。
これは入り口に入ったばかりということです。
アプリケーションに別のコンポーネントを加えてみれば、Kubernetes の優れた機能や性能が見えてきます。
それをまさに開発マシン上で確認できます。

{% comment %}
In addition to deploying to Kubernetes, we have also described our application as a Kubernetes YAML file. This simple text file contains everything we need to create our application in a running state. We can check it into version control and share it with our colleagues, allowing us to distribute our applications to other clusters (like the testing and production clusters that probably come after our development environments) easily.
{% endcomment %}
Kubernetes へのデプロイに加えて、アプリケーションを Kubernetes YAML ファイルとして記述しました。
この単純なテキストファイルが、アプリケーションを生成して実行するために必要なものをすべて含んでいるわけです。
このファイルをバージョン管理システムにアップして、仲間と共有してみましょう。
そうすればこのアプリケーションを別のクラスター（おそらく開発環境の次に作り出すものとしてテスト環境クラスターや本番環境クラスター）上に容易に実現できることになります。

{% comment %}
## Kubernetes references
{% endcomment %}
{: #kubernetes-references }
## Kubernetes リファレンス

{% comment %}
Further documentation for all new Kubernetes objects used in this article are available here:
{% endcomment %}
本文において新たに用いた Kubernetes オブジェクトについての詳細は、以下を参照してください。

{% comment %}
 - [Kubernetes Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)
 - [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
 - [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)
{% endcomment %}
 - [Kubernetes ポッド](https://kubernetes.io/docs/concepts/workloads/pods/pod/)
 - [Kubernetes デプロイメント](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
 - [Kubernetes サービス](https://kubernetes.io/docs/concepts/services-networking/service/)
