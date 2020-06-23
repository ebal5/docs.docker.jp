---
title: "Swarm へのデプロイ"
keywords: swarm, swarm services, stacks
description: Learn how to describe and deploy a simple application on Docker Swarm.
redirect_from:
- /get-started/part4/
---

{% comment %}
## Prerequisites
{% endcomment %}
{: #prerequisites }
## 前提条件

{% comment %}
- Download and install Docker Desktop as described in [Orientation and setup](index.md).
- Work through containerizing an application in [Part 2](part2.md).
- Make sure that Swarm is enabled on your Docker Desktop by typing `docker system info`, and looking for a message `Swarm: active` (you might have to scroll up a little).
{% endcomment %}
- [概要](index.md) にて説明している Docker Desktop をダウンロードしインストールしていること。
- [クイックスタート 2 部](part2.md) においてアプリケーションのコンテナー化を一通り学んでいること。
- Docker Desktop 上において Swarm が有効になっていること。
  これは `docker system info` を入力し、メッセージ内に `Swarm: active` があることで確認（多少スクロールして見ることが必要）。

  {% comment %}
  If Swarm isn't running, simply type `docker swarm init` in a shell prompt to set it up.
  {% endcomment %}
  Swarm が起動していない場合は、シェルプロンプトから単純に `docker swarm init` と入力して起動します。

{% comment %}
## Introduction
{% endcomment %}
{: #introduction }
## はじめに

{% comment %}
Now that we've demonstrated that the individual components of our application run as stand-alone containers and shown how to deploy it using Kubernetes, let's look at how to arrange for them to be managed by Docker Swarm. Swarm provides many tools for scaling, networking, securing and maintaining your containerized applications, above and beyond the abilities of containers themselves.
{% endcomment %}
ここまでにはアプリケーションの個々のコンポーネントが、スタンドアロンなコンテナーとして実行され、それを Kubernetes を用いてデプロイするところまでを見てきました。
今度は Docker Swarm によってどのように管理していくのかを見ていきます。
Swarm はコンテナー化されたアプリケーションに対して、そのコンテナーの能力を超えて、スケール、ネットワーク、セキュリティ、保守を行うツールを数多く提供します。

{% comment %}
In order to validate that our containerized application works well on Swarm, we'll use Docker Desktop's built in Swarm environment right on our development machine to deploy our application, before handing it off to run on a full Swarm cluster in production. The Swarm environment created by Docker Desktop is _fully featured_, meaning it has all the Swarm features your app will enjoy on a real cluster, accessible from the convenience of your development machine.
{% endcomment %}
Swarm 上においてコンテナー化アプリケーションが適正に動作していることを確認するために、まずは開発マシン上にて Swarm 環境化の Docker Desktop を用いてアプリケーションデプロイを行います。
完全な Swarm クラスターによる本番環境での稼動は、その次に扱っていくことにします。
Docker Desktop によって構築されている Swarm 環境は **完全な機能** を有しています。
つまりそこには Swarm の全機能が含まれていて、実際のクラスター上でアプリを動作させることができ、開発マシンから容易にアクセスすることができます。

{% comment %}
## Describe apps using stack files
{% endcomment %}
{: #describe-apps-using-stack-files }
## スタックファイルを用いたアプリ記述

{% comment %}
Swarm never creates individual containers like we did in the previous step of this tutorial. Instead, all Swarm workloads are scheduled as _services_, which are scalable groups of containers with added networking features maintained automatically by Swarm. Furthermore, all Swarm objects can and should be described in manifests called _stack files_. These YAML files describe all the components and configurations of your Swarm app, and can be used to easily create and destroy your app in any Swarm environment.
{% endcomment %}
本チュートリアルの以前の手順とは違って、Swarm では個別にコンテナーを生成することはしません。
そのかわり、Swarm 全体を **サービス** としてスケジューリングします。
これはスケール変更可能なコンテナーのグループであり、さらに Swarm によって自動的に管理されるネットワーク機能が付け加えられています。
そして Swarm によるオブジェクトは、すべて **スタックファイル**（stack file）と呼ばれるファイル内に記述されます。
この YAML ファイルに Swarm アプリのコンポーネントや設定をすべて記述します。
こうして Swarm 環境内でのアプリケーションの生成と削除が容易にできるようになります。

{% comment %}
Let's write a simple stack file to run and manage our bulletin board. Place the following in a file called `bb-stack.yaml`:
{% endcomment %}
では掲示板アプリを起動し管理するための単純なスタックファイルを記述します。
`bb-stack.yaml` というファイルに以下の内容を記述してください。

```yaml
version: '3.7'

services:
  bb-app:
    image: bulletinboard:1.0
    ports:
      - "8000:8080"
```

{% comment %}
In this Swarm YAML file, we have just one object: a `service`, describing a scalable group of identical containers. In this case, you'll get just one container (the default), and that container will be based on your `bulletinboard:1.0` image created in [Part 2](part2.md) of the Quickstart tutorial. In addition, We've asked Swarm to forward all traffic arriving at port 8000 on our development machine to port 8080 inside our bulletin board container.
{% endcomment %}
この Swarm YAML ファイルには `service` という１つのオブジェクトがあるのみです。
これはスケール変更可能な同一コンテナーのグループを表わします。
今の場合、（デフォルトにより）ただ１つのコンテナーを生成します。
ベースとするイメージは、クイックスタートチュートリアルの [2 部](part2.md) の手順にて利用した `bulletinboard:1.0` です。
そして開発マシン上のポート 8000 にきたトラフィックを、掲示板アプリコンテナー内のポート 8080 にフォワードするように指定しています。

{% comment %}
> **Kubernetes Services and Swarm Services are very different!** Despite the similar name, the two orchestrators mean very different things by the term 'service'. In Swarm, a service provides both scheduling _and_ networking facilities, creating containers and providing tools for routing traffic to them. In Kubernetes, scheduling and networking are handled separately: _deployments_ (or other controllers) handle the scheduling of containers as pods, while _services_ are responsible only for adding networking features to those pods.
{% endcomment %}
> **Kubernetes サービスと Swarm サービスはまったく違います。**
> この 2 つのオーケストレーターはサービスという同じ名前を持っていますが、サービスの意味するところがまったく違います。
> Swarm の場合はスケジューリング機能とネットワーク機能が **両方ともに** 提供されます。
> またコンテナーの生成と、これに向けてトラフィック送信を行うツールが提供されます。
> Kubernetes の場合、スケジューリング機能とネットワーク機能は別々に扱われています。
> **デプロイ機能**（あるいは他の制御機能）が、コンテナーのスケジューリングをポッド（pod）として扱います。
> また **サービス機能** というものが、そのポッドに対してのネットワーク機能付与のみを行います。

{% comment %}
## Deploy and check your application
{% endcomment %}
{: #deploy-and-check-your-application }
## アプリケーションのデプロイと確認

{% comment %}
1.  Deploy your application to Swarm:
{% endcomment %}
1.  Swarm にアプリケーションをデプロイします。

    ```shell
    docker stack deploy -c bb-stack.yaml demo
    ```

    {% comment %}
    If all goes well, Swarm will report creating all your stack objects with no complaints:
    {% endcomment %}
    すべて正常であればエラーはなく、スタックオブジェクトがすべて生成できたことを表示します。

    ```shell
    Creating network demo_default
    Creating service demo_bb-app
    ```

    {% comment %}
    Notice that in addition to your service, Swarm also creates a Docker network by default to isolate the containers deployed as part of your stack.
    {% endcomment %}
    サービスが生成されると同時に、Swarm はデフォルトで Docker ネットワークを生成し、スタックの一部分としてデプロイされたコンテナーを分離し実行します。

{% comment %}
2.  Make sure everything worked by listing your service:
{% endcomment %}
2.  サービス一覧を表示して、正常にすべてが動作していることを確認します。

    ```shell
    docker service ls
    ```

    {% comment %}
    If all has gone well, your service will report with 1/1 of its replicas created:
    {% endcomment %}
    すべて正常に動作していれば、サービスからレプリカ 1/1 が生成されたことが表示されます。

    ```shell
    ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
    il7elwunymbs        demo_bb-app         replicated          1/1                 bulletinboard:1.0   *:8000->8080/tcp
    ```

    {% comment %}
    This indicates 1/1 containers you asked for as part of your services are up and running. Also, we see that port 8000 on your development machine is getting forwarded to port 8080 in your bulletin board container.
    {% endcomment %}
    これはつまり、サービスの一部分として生成が指示された 1/1 のコンテナーが起動していることが示されています。
    また開発マシンの 8000 ポートが、掲示板アプリの 8080 ポートにフォワード設定されていることもわかります。

{% comment %}
3.  Open a browser and visit your bulletin board at `localhost:8000`; you should see your bulletin board, the same as when we ran it as a stand-alone container in Part 2 of the Quickstart tutorial.
{% endcomment %}
3.  ブラウザーを開いて `localhost:30001` にアクセスし掲示板アプリを開きます。
    ここでも掲示板アプリを見ることができます。
    これはクイックスタートチュートリアルの 2 部において、スタンドアロンなコンテナー上に起動させたアプリと同一のものです。

{% comment %}
4.  Once satisfied, tear down your application:
{% endcomment %}
4.  結果を確認できたらアプリケーションを削除します。

    ```shell
    docker stack rm demo
    ```

{% comment %}
## Conclusion
{% endcomment %}
{: #conclusion }
## まとめ

{% comment %}
At this point, we have successfully used Docker Desktop to deploy our application to a fully-featured Swarm environment on our development machine. We haven't done much with Swarm yet, but the door is now open: you can begin adding other components to your app and taking advantage of all the features and power of Swarm, right on your own machine.
{% endcomment %}
開発マシン上にて Swarm の全機能を実現した環境に、Docker Desktop を用いてアプリケーションをデプロイすることに成功しました。
Swarm を使った作業には、まだまだ多くのことがあります。
これは入り口に入ったばかりということです。
アプリケーションに別のコンポーネントを加えてみれば、Swarm の優れた機能や性能が見えてきます。
それをまさに開発マシン上で確認できます。

{% comment %}
In addition to deploying to Swarm, we have also described our application as a stack file. This simple text file contains everything we need to create our application in a running state; we can check it into version control and share it with our colleagues, allowing us to distribute our applications to other clusters (like the testing and production clusters that probably come after our development environments) easily.
{% endcomment %}
Swarm へのデプロイに加えて、アプリケーションをスタックファイルとして記述しました。
この単純なテキストファイルが、アプリケーションを生成して実行するために必要なものをすべて含んでいるわけです。
このファイルをバージョン管理システムにアップして、仲間と共有してみましょう。
そうすればこのアプリケーションを別の担当者（おそらく開発担当者から受け渡す先として、テスト環境担当者や本番環境担当者）に容易に配布できることになります。

{% comment %}
## Swarm and CLI references
{% endcomment %}
{: #swarm-and-cli-references }
## Swarm と CLI リファレンス

{% comment %}
Further documentation for all new Swarm objects and CLI commands used in this article are available here:
{% endcomment %}
本文において新たに用いた Swarm オブジェクトについての詳細は、以下を参照してください。

 {% comment %}
 - [Swarm Services](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)
 - [Swarm Stacks](https://docs.docker.com/engine/swarm/stack-deploy/)
 - [`docker stack *`](https://docs.docker.com/engine/reference/commandline/stack/)
 - [`docker service *`](https://docs.docker.com/engine/reference/commandline/service/)
 {% endcomment %}
 - [Swarm サービス](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)
 - [Swarm スタック](https://docs.docker.com/engine/swarm/stack-deploy/)
 - [`docker stack *`](https://docs.docker.com/engine/reference/commandline/stack/)
 - [`docker service *`](https://docs.docker.com/engine/reference/commandline/service/)
