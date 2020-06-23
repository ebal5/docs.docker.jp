---
description: How to run more than one process in a container
keywords: docker, supervisor, process management
redirect_from:
- /engine/articles/using_supervisord/
- /engine/admin/using_supervisord/
- /engine/admin/multi-service_container/
title: コンテナー内での複数サービス起動
---

{% comment %}
A container's main running process is the `ENTRYPOINT` and/or `CMD` at the
end of the `Dockerfile`. It is generally recommended that you separate areas of
concern by using one service per container. That service may fork into multiple
processes (for example, Apache web server starts multiple worker processes).
It's ok to have multiple processes, but to get the most benefit out of Docker,
avoid one container being responsible for multiple aspects of your overall
application. You can connect multiple containers using user-defined networks and
shared volumes.
{% endcomment %}
コンテナーの主となる実行プロセスは、`Dockerfile` の最終部分に指定される `ENTRYPOINT` や `CMD` です。
1 つのコンテナーには 1 つのサービスを割り当てるということにすれば、気にかける箇所が絞られるので、一般的にはこれが推奨されます。
ただそのサービスからは、複数のプロセスがフォークされることもあります（たとえば Apache ウェブサーバーでは複数のワーカープロセスが起動されます）。
マルチプロセスとなることは、まったく問題ありません。
一方で、アプリケーションが持ついくつもの役割を 1 つのコンテナーに持たせることは、Docker の優れた機能を利用する観点からは避けるべきです。
コンテナーを複数にするのであれば、ユーザー定義のネットワークや共有ボリュームを利用して接続します。

{% comment %}
The container's main process is responsible for managing all processes that it
starts. In some cases, the main process isn't well-designed, and doesn't handle
"reaping" (stopping) child processes gracefully when the container exits. If
your process falls into this category, you can use the `--init` option when you
run the container. The `--init` flag inserts a tiny init-process into the
container as the main process, and handles reaping of all processes when the
container exits. Handling such processes this way is superior to using a
full-fledged init process such as `sysvinit`, `upstart`, or `systemd` to handle
process lifecycle within your container.
{% endcomment %}
コンテナーのメインプロセスは、コンテナーそのものが起動させるプロセスすべてを管理するためにあります。
メインプロセスが十分に機能していないことが原因で、コンテナー終了時に子プロセスを適切に停止できないことがあります。
起動プロセスがこの手の事態に陥った場合は、コンテナー起動時に `--init` オプションを指定してみてください。
この `--init` フラグは、コンテナーのメインプロセスとして、非常に小さな初期化プロセスを埋め込みます。
この小さなプロセスが、コンテナー終了時の子プロセス停止を受け持つことになります。
子プロセスの扱いをこのようにするのは、本格的な初期化プロセス、たとえば `sysvinit`、`upstart`、`systemd` に比べて、コンテナー内部のプロセスのライフサイクルを適切に扱うことができるからです。

{% comment %}
If you need to run more than one service within a container, you can accomplish
this in a few different ways.
{% endcomment %}
1 つのコンテナー内に複数のサービスを起動させる必要があるなら、方法はいくつかあります。

{% comment %}
- Put all of your commands in a wrapper script, complete with testing and
  debugging information. Run the wrapper script as your `CMD`. This is a very
  naive example. First, the wrapper script:
{% endcomment %}
- 実行するコマンドをすべてラッパースクリプトに含めます。
  あらかじめテストやデバッグは行っておきます。
  そしてこのラッパースクリプトを `CMD` として実行します。
  以下は簡単な例です。
  まずはラッパースクリプトを生成します。

  ```bash
  #!/bin/bash

  # 1つめのプロセスを起動
  ./my_first_process -D
  status=$?
  if [ $status -ne 0 ]; then
    echo "Failed to start my_first_process: $status"
    exit $status
  fi

  # 2つめのプロセスを起動
  ./my_second_process -D
  status=$?
  if [ $status -ne 0 ]; then
    echo "Failed to start my_second_process: $status"
    exit $status
  fi

  # 単純なチェックとして 1分間隔で2つのプロセスの終了コードを確認します。
  # 1つのコンテナーに複数サービスを起動させたい場合に、このような部分が
  # 大変なところです。どちらかのプロセスの終了が検出されたら、コンテナー
  # はエラー終了するようにします。そうでなければ、60秒ごとに確認しながら
  # ループし続けます。

  while sleep 60; do
    ps aux |grep my_first_process |grep -q -v grep
    PROCESS_1_STATUS=$?
    ps aux |grep my_second_process |grep -q -v grep
    PROCESS_2_STATUS=$?
    # 上の2つのgrepが検索マッチすれば、どちらの終了ステータスともゼロ。
    # 2つともゼロでないなら何かがおかしい。
    if [ $PROCESS_1_STATUS -ne 0 -o $PROCESS_2_STATUS -ne 0 ]; then
      echo "One of the processes has already exited."
      exit 1
    fi
  done
  ```

  {% comment %}
  Next, the Dockerfile:
  {% endcomment %}
  Dockerfile は以下のような記述とします。

  ```dockerfile
  FROM ubuntu:latest
  COPY my_first_process my_first_process
  COPY my_second_process my_second_process
  COPY my_wrapper_script.sh my_wrapper_script.sh
  CMD ./my_wrapper_script.sh
  ```

{% comment %}
- If you have one main process that needs to start first and stay running but
  you temporarily need to run some other processes (perhaps to interact with
  the main process) then you can use bash's job control to facilitate that.
  First, the wrapper script:
{% endcomment %}
- 1 つのメインプロセスを起動させたら、そのまま起動し続ける場合です。
  一時的に別のプロセスをいくつか起動する（そしておそらくはメインプロセスと通信を行う）とします。
  この場合は bash のジョブ制御の機能を利用します。
  まずはラッパースクリプトを生成します。

  ```bash
  #!/bin/bash

  # ジョブ制御を有効にします。
  set -m

  # 1つめのプロセスをバックグラウンドで実行します。
  ./my_main_process &

  # ヘルパープロセスを実行します。
  ./my_helper_process

  # この my_helper_process は自分の処理を開始して終了するためには、
  # 1つめのプロセスの動きを知っておく必要があるかもしれません。


  # ここで1つめのプロセスをフォアグラウンド実行に戻してそのままとします。
  fg %1
  ```

  ```dockerfile
  FROM ubuntu:latest
  COPY my_main_process my_main_process
  COPY my_helper_process my_helper_process
  COPY my_wrapper_script.sh my_wrapper_script.sh
  CMD ./my_wrapper_script.sh
  ```

{% comment %}
- Use a process manager like `supervisord`. This is a moderately heavy-weight
  approach that requires you to package `supervisord` and its configuration in
  your image (or base your image on one that includes `supervisord`), along with
  the different applications it manages. Then you start `supervisord`, which
  manages your processes for you. Here is an example Dockerfile using this
  approach, that assumes the pre-written `supervisord.conf`, `my_first_process`,
  and `my_second_process` files all exist in the same directory as your
  Dockerfile.
{% endcomment %}
- `supervisord` のようなプロセスマネージャーを利用する場合です。
  これは少々面倒な方法です。
  これを行うためには、イメージ内に `supervisord` パッケージとその設定を含める必要があります。
  （あるいは `supervisord` が含まれているイメージをベースとします。）
  さらにそのパッケージが管理する別のアプリケーションが必要になってきます。
  その上で `supervisord` を起動させてプロセス管理を行います。
  以下はこの手法を利用する Dockerfile です。
  `supervisord.conf`、`my_first_process`、`my_second_process` の各ファイルは準備ができていて、Dockerfile と同一ディレクトリに存在しているとします。

  ```dockerfile
  FROM ubuntu:latest
  RUN apt-get update && apt-get install -y supervisor
  RUN mkdir -p /var/log/supervisor
  COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
  COPY my_first_process my_first_process
  COPY my_second_process my_second_process
  CMD ["/usr/bin/supervisord"]
  ```
