---
description: Docker デーモンの設定とトラブルシュート
keywords: docker, daemon, configuration, troubleshooting
redirect_from:
- /engine/articles/chef/
- /engine/articles/configuring/
- /engine/articles/dsc/
- /engine/articles/puppet/
- /engine/admin/configuring/
- /engine/admin/
- /engine/admin/ansible/
- /engine/admin/chef/
- /engine/admin/dsc/
- /engine/admin/puppet/
- /engine/userguide/
- /config/thirdparty/
- /config/thirdparty/ansible/
- /config/thirdparty/chef/
- /config/thirdparty/dsc/
- /config/thirdparty/puppet/

title: Docker デーモンの設定とトラブルシュート
---

{% comment %}
After successfully installing and starting Docker, the `dockerd` daemon
runs with its default configuration. This topic shows how to customize
the configuration, start the daemon manually, and troubleshoot and debug the
daemon if you run into issues.
{% endcomment %}
Docker を正常にインストールし Docker を起動すると、`dockerd` というデーモンがデフォルト設定により起動します。
ここではその設定のカスタマイズ方法、デーモンの手動起動、問題発生時のトラブルシュートやデバッグについて示します。

{% comment %}
## Start the daemon using operating system utilities
{% endcomment %}
{: #start-the-daemon-using-operating-system-utilities }
## OS ユーティリティーによるデーモンの起動

{% comment %}
On a typical installation the Docker daemon is started by a system utility,
not manually by a user. This makes it easier to automatically start Docker when
the machine reboots.
{% endcomment %}
標準的なインストールによる Docker デーモンは、ユーザーが手動で起動するものではなく、システムのユーティリティーによって起動されます。
これによりマシン再起動時に、Docker を自動起動させることが簡単になります。

{% comment %}
The command to start Docker depends on your operating system. Check the correct
page under [Install Docker](../../engine/install/index.md). To configure Docker
to start automatically at system boot, see
[Configure Docker to start on boot](../../engine/install/linux-postinstall.md#configure-docker-to-start-on-boot).
{% endcomment %}
Docker を起動させるコマンドはオペレーティングシステムによります。
[Docker のインストール](../../engine/install/index.md) の中から適切なページを確認してください。
システム起動時に Docker を自動起動するように設定する場合は [システムブート時の Docker 起動設定](../../engine/install/linux-postinstall.md#configure-docker-to-start-on-boot) を参照してください。

{% comment %}
## Start the daemon manually
{% endcomment %}
{: #start-the-daemon-manually }
## デーモンの手動起動

{% comment %}
If you don't want to use a system utility to manage the Docker daemon, or
just want to test things out, you can manually run it using the `dockerd`
command. You may need to use `sudo`, depending on your operating system
configuration.
{% endcomment %}
Docker デーモンをシステムユーティリティーによって管理することが望ましくない場合、あるいは何かをすぐにテストしたいといった場合、`dockerd` コマンドを利用してデーモンを手動起動することができます。
このときには `sudo` を必要とするかもしれませんが、そこはオペレーティングシステムでの設定によります。

{% comment %}
When you start Docker this way, it runs in the foreground and sends its logs
directly to your terminal.
{% endcomment %}
この方法によって Docker を起動した場合、Docker はフォアグラウンド実行され、出力ログは端末に直接出力されます。

```bash
$ dockerd

INFO[0000] +job init_networkdriver()
INFO[0000] +job serveapi(unix:///var/run/docker.sock)
INFO[0000] Listening for HTTP on unix (/var/run/docker.sock)
```

{% comment %}
To stop Docker when you have started it manually, issue a `Ctrl+C` in your
terminal.
{% endcomment %}
手動で起動させた Docker を停止するには、端末上で `Ctrl+C` を入力します。

{% comment %}
## Configure the Docker daemon
{% endcomment %}
{: #configure-the-docker-daemon }
## デーモンの設定

{% comment %}
There are two ways to configure the Docker daemon:
{% endcomment %}
Docker デーモンの設定には 2 つの方法があります。

{% comment %}
* Use a JSON configuration file. This is the preferred option, since it keeps
all configurations in a single place.
* Use flags when starting `dockerd`.
{% endcomment %}
* JSON 形式の設定ファイルを利用します。
  設定をすべて一箇所にとりまとめられるので、この方法が推奨されます。
* `dockerd` 起動時のフラグを利用します。

{% comment %}
You can use both of these options together as long as you don't specify the
same option both as a flag and in the JSON file. If that happens, the Docker
daemon won't start and prints an error message.
{% endcomment %}
同じオプションをフラグと JSON ファイルの両方に指定しない限り、フラグと JSON ファイルによる指定を併用することができます。
もし両方に指定してしまった場合、Docker デーモンは起動せず、エラーメッセージが出力されます。

{% comment %}
To configure the Docker daemon using a JSON file, create a file at
`/etc/docker/daemon.json` on Linux systems, or `C:\ProgramData\docker\config\daemon.json`
on Windows. On MacOS go to the whale in the taskbar > Preferences > Daemon > Advanced.
{% endcomment %}
JSON ファイルを使って Docker デーモンを設定する場合、Linux であればその設定ファイルを `/etc/docker/daemon.json` に、また Windows であれば `C:\ProgramData\docker\config\daemon.json` にファイルを生成します。
MacOS の場合は、タスクバー上の Docker アイコンをクリックして Preferences > Daemon > Advanced を実行して設定します。

{% comment %}
Here's what the configuration file looks like:
{% endcomment %}
設定ファイルは以下のようにします。

```json
{
  "debug": true,
  "tls": true,
  "tlscert": "/var/docker/server.pem",
  "tlskey": "/var/docker/serverkey.pem",
  "hosts": ["tcp://192.168.59.3:2376"]
}
```

{% comment %}
With this configuration the Docker daemon runs in debug mode, uses TLS, and
listens for traffic routed to `192.168.59.3` on port `2376`.
You can learn what configuration options are available in the
[dockerd reference docs](../../engine/reference/commandline/dockerd.md#daemon-configuration-file)
{% endcomment %}
この設定によってデーモンを起動すると、デバッグモードとなり TLS を利用し、`192.168.59.3` の `2376` ポートへのトラフィックを待ち受けるものとなります。
どのような設定オプションが利用可能であるかは、[dockerd リファレンスドキュメント](../../engine/reference/commandline/dockerd.md#daemon-configuration-file) を参照してください。

{% comment %}
You can also start the Docker daemon manually and configure it using flags.
This can be useful for troubleshooting problems.
{% endcomment %}
また Docker デーモンを手動で起動し、その際にフラグを使って設定することもできます。
これはトラブルを解決するときに活用できる方法です。

{% comment %}
Here's an example of how to manually start the Docker daemon, using the same
configurations as above:
{% endcomment %}
以下の例は Docker デーモンを手動で起動する方法であり、先ほどと同じ設定を行うものです。

```bash
dockerd --debug \
  --tls=true \
  --tlscert=/var/docker/server.pem \
  --tlskey=/var/docker/serverkey.pem \
  --host tcp://192.168.59.3:2376
```

{% comment %}
You can learn what configuration options are available in the
[dockerd reference docs](../../engine/reference/commandline/dockerd.md), or by running:
{% endcomment %}
どのような設定オプションが利用可能であるかは、[dockerd リファレンスドキュメント](../../engine/reference/commandline/dockerd.md) を参照するるか、あるいは以下を実行してみてください。

```
dockerd --help
```

{% comment %}
Many specific configuration options are discussed throughout the Docker
documentation. Some places to go next include:
{% endcomment %}
Docker ドキュメントでは、設定オプションを数多く取り扱っています。
その中から次に見ていくものとして、以下をあげておきます。

{% comment %}
- [Automatically start containers](../containers/start-containers-automatically.md)
- [Limit a container's resources](../containers/resource_constraints.md)
- [Configure storage drivers](../../storage/storagedriver/select-storage-driver.md)
- [Container security](../../engine/security/index.md)
{% endcomment %}
- [コンテナーの自動起動](../containers/start-containers-automatically.md)
- [コンテナーのリソース制限](../containers/resource_constraints.md)
- [ストレージドライバーの設定](../../storage/storagedriver/select-storage-driver.md)
- [コンテナーのセキュリティ](../../engine/security/index.md)

{% comment %}
## Docker daemon directory
{% endcomment %}
{: #docker-daemon-directory }
## Docker デーモンディレクトリ

{% comment %}
The Docker daemon persists all data in a single directory. This tracks everything
related to Docker, including containers, images, volumes, service definition,
and secrets.
{% endcomment %}
Docker デーモンは関連データをすべて、単一のディレクトリ配下に保持します。
ここに Docker に関連する、コンテナー、イメージ、ボリューム、サービス定義、機密情報などすべて管理しています。

{% comment %}
By default this directory is:
{% endcomment %}
このディレクトリはデフォルトでは以下のものです。

{% comment %}
* `/var/lib/docker` on Linux.
* `C:\ProgramData\docker` on Windows.
{% endcomment %}
* Linux では `/var/lib/docker`
* Windows では `C:\ProgramData\docker`

{% comment %}
You can configure the Docker daemon to use a different directory, using the
`data-root` configuration option.
{% endcomment %}
設定オプションの `data-root` というものを使えば、Docker デーモンが利用するディレクトリを別のところに変更することもできます。

{% comment %}
Since the state of a Docker daemon is kept on this directory, make sure
you use a dedicated directory for each daemon. If two daemons share the same
directory, for example, an NFS share, you are going to experience errors that
are difficult to troubleshoot.
{% endcomment %}
Docker デーモンの状態はこのディレクトリ内に保持されるので、デーモンごとに専用のディレクトリを利用することが必要です。
2 つあるデーモンが、たとえば NFS 共有による同一のディレクトリをともに利用していると、解決困難なエラーに出会うことになりかねません。

{% comment %}
## Troubleshoot the daemon
{% endcomment %}
{: #troubleshoot-the-daemon }
## デーモンのトラブルシューティング

{% comment %}
You can enable debugging on the daemon to learn about the runtime activity of
the daemon and to aid in troubleshooting. If the daemon is completely
non-responsive, you can also
[force a full stack trace](#force-a-stack-trace-to-be-logged) of all
threads to be added to the daemon log by sending the `SIGUSR` signal to the
Docker daemon.
{% endcomment %}
デーモンのデバッグ機能を有効にすれば、デーモン実行時の様子を見たり、トラブル解決に役立てたりすることができます。
デーモンが完全に応答しなくなった場合は、全スレッドに対して [スタックトレースの強制出力](#force-a-stack-trace-to-be-logged) が可能です。
これは Docker デーモンに対して `SIGUSR` シグナルを送信すれば、デーモンログに出力されます。

{% comment %}
### Troubleshoot conflicts between the `daemon.json` and startup scripts
{% endcomment %}
{: #troubleshoot-conflicts-between-the-daemonjson-and-startup-scripts }
### `daemon.json` と起動スクリプトの競合を解決する

{% comment %}
If you use a `daemon.json` file and also pass options to the `dockerd`
command manually or using start-up scripts, and these options conflict,
Docker fails to start with an error such as:
{% endcomment %}
`daemon.json` ファイル利用時であって、同じオプションを `dockerd` コマンドに対して手動で指定した場合、あるいは起動スクリプト実行をした場合、そのオプションが競合してしまい、以下のようなエラーが出力されてデーモン起動は失敗します。

```none
unable to configure the Docker daemon with file /etc/docker/daemon.json:
the following directives are specified both as a flag and in the configuration
file: hosts: (from flag: [unix:///var/run/docker.sock], from file: [tcp://127.0.0.1:2376])
```

{% comment %}
If you see an error similar to this one and you are starting the daemon manually with flags,
you may need to adjust your flags or the `daemon.json` to remove the conflict.
{% endcomment %}
上のようなエラーが出力され、デーモンの手動起動時にフラグ設定を行っているなら、そのフラグを正しくするか、あるいは `daemon.json` 内から競合しているフラグを削除します。

{% comment %}
> **Note**: If you see this specific error, continue to the
> [next section](#use-the-hosts-key-in-daemonjson-with-systemd) for a workaround.
{% endcomment %}
> **メモ**:
> 上のエラーが出力された場合、とりあえずの解決としては [次節](#use-the-hosts-key-in-daemonjson-with-systemd) に進んでください。

{% comment %}
If you are starting Docker using your operating system's init scripts, you may
need to override the defaults in these scripts in ways that are specific to the
operating system.
{% endcomment %}
オペレーティングシステムの初期化スクリプトにより Docker デーモンを起動している場合は、そのスクリプト内におけるデフォルト設定を上書きすることになるかもしれません。
その方法については、オペレーティングシステムが規定する方法に従ってください。

{% comment %}
#### Use the hosts key in daemon.json with systemd
{% endcomment %}
{: #use-the-hosts-key-in-daemonjson-with-systemd }
#### systemd を使った daemon.json 内での hosts キーの利用

{% comment %}
One notable example of a configuration conflict that is difficult to troubleshoot
is when you want to specify a different daemon address from
the default. Docker listens on a socket by default. On Debian and Ubuntu systems using `systemd`,
this means that a host flag `-H` is always used when starting `dockerd`. If you specify a
`hosts` entry in the `daemon.json`, this causes a configuration conflict (as in the above message)
and Docker fails to start.
{% endcomment %}
競合が発生してもわかりにくい例として、よくあるのが、デーモンのアドレスをデフォルトとは異なるものにした場合です。
Docker はデフォルトでソケットを待ち受けます。
Debian や Ubuntu は `systemd` を利用しているので、`dockerd` の起動時には必ず、ホスト指定に `-H` フラグを用います。
`daemon.json` ファイルに `host` 項目を指定した場合、設定の競合が発生して（上で示したメッセージが出力され）Docker の起動は失敗します。

{% comment %}
To work around this problem, create a new file `/etc/systemd/system/docker.service.d/docker.conf` with
the following contents, to remove the `-H` argument that is used when starting the daemon by default.
{% endcomment %}
この問題をとりあえず回避するには `/etc/systemd/system/docker.service.d/docker.conf` というファイルを生成し、内容を以下のようにします。
これはデフォルトにおいて、デーモン起動時に用いられる `-H` 引数を取り除くものです。

```none
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```

{% comment %}
There are other times when you might need to configure `systemd` with Docker, such as
[configuring a HTTP or HTTPS proxy](systemd.md#httphttps-proxy).
{% endcomment %}
Docker に関して `systemd` の設定を見直す必要が、他にも出てくるかもしれません。
たとえば [HTTP または HTTPS プロキシーの設定](systemd.md#httphttps-proxy) を行う必要がある場合です。

{% comment %}
> **Note**: If you override this option and then do not specify a `hosts` entry in the `daemon.json`
> or a `-H` flag when starting Docker manually, Docker fails to start.
{% endcomment %}
> **メモ**:
> このオプションを上のように設定したにもかかわらず、`daemon.json` の `hosts` 設定や、Docker 手動起動時の `-H` フラグを用いなかった場合には、Docker の起動は失敗します。

{% comment %}
Run `sudo systemctl daemon-reload` before attempting to start Docker. If Docker starts
successfully, it is now listening on the IP address specified in the `hosts` key of the
`daemon.json` instead of a socket.
{% endcomment %}
Docker を起動しようとする前には `sudo systemctl daemon-reload` を実行してください。
Docker が正常に起動したら、これ以降はソケットを待ち受けるのでなく、`daemon.json` の `hosts` キーに指定された IP アドレスからのトラフィックを待ち受けることになります。

{% comment %}
> **Important**: Setting `hosts` in the `daemon.json` is not supported on Docker Desktop for Windows
> or Docker Desktop for Mac.
{:.important}
{% endcomment %}
> **重要**:
> `daemon.json` において `hosts` の設定を行う方法は、Docker Desktop for Windows や Docker Desktop for Mac ではサポートされていません。
{:.important}



{% comment %}
### Out Of Memory Exceptions (OOME)
{% endcomment %}
{: #out-of-memory-exceptions-oome  }
### Out Of Memory Exceptions (OOME)

{% comment %}
If your containers attempt to use more memory than the system has available,
you may experience an Out Of Memory Exception (OOME) and a container, or the
Docker daemon, might be killed by the kernel OOM killer. To prevent this from
happening, ensure that your application runs on hosts with adequate memory and
see
[Understand the risks of running out of memory](../containers/resource_constraints.md#understand-the-risks-of-running-out-of-memory).
{% endcomment %}
コンテナーの利用するメモリ容量が、システムの残容量を超えた場合に、Out Of Memory Exception (OOME) が発生することがあります。
その場合コンテナーあるいは Docker デーモンは、カーネルの OOM キラーによって kill されるかもしれません。
このような発生を防ぐためには、ホスト上の適切なメモリ容量範囲内でアプリケーションが動作するようにしてください。
[out of memory となるリスクの理解](../containers/resource_constraints.md#understand-the-risks-of-running-out-of-memory) も確認してください。

{% comment %}
### Read the logs
{% endcomment %}
{: #read-the-logs }
### ログを読む

{% comment %}
The daemon logs may help you diagnose problems. The logs may be saved in one of
a few locations, depending on the operating system configuration and the logging
subsystem used:
{% endcomment %}
デーモンログは、問題の解決に役立つものです。
ログはいくつかの場所に保存されます。
これはオペレーティングシステムの設定や、利用しているログ管理システムによって定まります。

{% comment %}
| Operating system      | Location                                                                                 |
|:----------------------|:-----------------------------------------------------------------------------------------|
| RHEL, Oracle Linux    | `/var/log/messages`                                                                      |
| Debian                | `/var/log/daemon.log`                                                                    |
| Ubuntu 16.04+, CentOS | Use the command `journalctl -u docker.service`                                           |
| Ubuntu 14.10-         | `/var/log/upstart/docker.log`                                                            |
| macOS (Docker 18.01+) | `~/Library/Containers/com.docker.docker/Data/vms/0/console-ring`                         |
| macOS (Docker <18.01) | `~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/console-ring` |
| Windows               | `AppData\Local`                                                                          |
{% endcomment %}
| オペレーティングシステム  | 場所                                                                                     |
|:--------------------------|:-----------------------------------------------------------------------------------------|
| RHEL, Oracle Linux        | `/var/log/messages`                                                                      |
| Debian                    | `/var/log/daemon.log`                                                                    |
| Ubuntu 16.04 以上, CentOS | コマンド `journalctl -u docker.service` 実行                                             |
| Ubuntu 14.10 未満         | `/var/log/upstart/docker.log`                                                            |
| macOS (Docker 18.01 以上) | `~/Library/Containers/com.docker.docker/Data/vms/0/console-ring`                         |
| macOS (Docker 18.01 未満) | `~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/console-ring` |
| Windows                   | `AppData\Local`                                                                          |


{% comment %}
### Enable debugging
{% endcomment %}
{: #enable-debugging }
### デバッグの有効化

{% comment %}
There are two ways to enable debugging. The recommended approach is to set the
`debug` key to `true` in the `daemon.json` file. This method works for every
Docker platform.
{% endcomment %}
デバッグを有効にするには 2 通りあります。
お勧めの方法は `daemon.json` ファイルにおいて `debug` キーを `true` にすることです。
この方法はどの Docker プラットフォームでも動作します。

{% comment %}
1.  Edit the `daemon.json` file, which is usually located in `/etc/docker/`.
    You may need to create this file, if it does not yet exist. On macOS or
    Windows, do not edit the file directly. Instead, go to
    **Preferences** / **Daemon** / **Advanced**.
{% endcomment %}
1.  `daemon.json` ファイルを編集します。
    その場所は通常 `/etc/docker/` にあります。
    存在しなければ、ここで生成します。
    macOS や Windows の場合、このファイルを直接編集はしません。
    そのかわり **Preferences** / **Daemon** / **Advanced** を実行してください。

{% comment %}
2.  If the file is empty, add the following:
{% endcomment %}
2.  ファイルが空であれば以下の記述を追加します。

    ```json
    {
      "debug": true
    }
    ```

    {% comment %}
    If the file already contains JSON, just add the key `"debug": true`, being
    careful to add a comma to the end of the line if it is not the last line
    before the closing bracket. Also verify that if the `log-level` key is set,
    it is set to either `info` or `debug`. `info` is the default, and possible
    values are `debug`, `info`, `warn`, `error`, `fatal`.
    {% endcomment %}
    このファイルにすでに JSON 記述が行われていた場合は `"debug": true` というキー項目のみを追加します。
    閉じブラケット直前の最終行として追加する場合には、行末にカンマは不要ですが、そうでなければカンマを忘れないでください。
    また `log-level` を設定する場合には、その値を `info` か `debug` にしてください。
    `info` がデフォルト値であり、設定可能な値は `debug`、`info`、`warn`、`error`、`fatal` のいずれかです。

{% comment %}
3.  Send a `HUP` signal to the daemon to cause it to reload its configuration.
    On Linux hosts, use the following command.
{% endcomment %}
3.  この設定を再読み込みさせるために、デーモンに対して `HUP` シグナルを送信します。
    Linux ホストの場合は以下のコマンドを実行します。

    ```bash
    $ sudo kill -SIGHUP $(pidof dockerd)
    ```

    {% comment %}
    On Windows hosts, restart Docker.
    {% endcomment %}
    Windows ホストの場合は Docker を再起動します。

{% comment %}
Instead of following this procedure, you can also stop the Docker daemon and
restart it manually with the debug flag `-D`. However, this may result in Docker
restarting with a different environment than the one the hosts' startup scripts
create, and this may make debugging more difficult.
{% endcomment %}
上のような方法ではなく、デバッグフラグ `-D` を手動で利用すれば Docker デーモンの停止と再起動を行うことができます。
ただしこれを行うと、ホストの初期起動スクリプトが生成するものとは異なる環境において Docker が再起動してしまうことがあります。
この場合、デバッグ作業は難しくなります。

{% comment %}
### Force a stack trace to be logged
{% endcomment %}
{: #force-a-stack-trace-to-be-logged }
### スタックトレースの強制ログ出力

{% comment %}
If the daemon is unresponsive, you can force a full stack trace to be logged
by sending a `SIGUSR1` signal to the daemon.
{% endcomment %}
デーモンが反応しなくなった場合には、フルスタックトレースをログ出力させるために、デーモンに対して `SIGUSR1` シグナルを送信します。

- **Linux の場合**

  ```bash
  $ sudo kill -SIGUSR1 $(pidof dockerd)
  ```

- **Windows Server の場合**

  {% comment %}
  Download [docker-signal](https://github.com/moby/docker-signal).
  {% endcomment %}
  [docker-signal](https://github.com/moby/docker-signal) をダウンロード。

  {% comment %}
  Get the process ID of dockerd `Get-Process dockerd`.
  {% endcomment %}
  `Get-Process dockerd` により dockerd のプロセス ID を取得。

  {% comment %}
  Run the executable with the flag `--pid=<PID of daemon>`.
  {% endcomment %}
  `--pid=<デーモンのプロセス ID>` フラグを使ってモジュールを実行。

{% comment %}
This forces a stack trace to be logged but does not stop the daemon.
Daemon logs show the stack trace or the path to a file containing the
stack trace if it was logged to a file.
{% endcomment %}
これによりスタックトレースがログに出力されますが、デーモンは停止されません。
デーモンログにスタックトレースが出力されているか、あるいはスタックトレースをファイル出力されていれば、そのファイルへのパスが示されています。

{% comment %}
The daemon continues operating after handling the `SIGUSR1` signal and
dumping the stack traces to the log. The stack traces can be used to determine
the state of all goroutines and threads within the daemon.
{% endcomment %}
デーモンが `SIGUSR1` シグナルを処理した後は、スタックトレースをログ出力した上で、動作を続行します。
スタックトレースからは、デーモン内部で処理されていた Go 言語のゴルーチン（goroutine）やスレッドの状態を確認することができます。

{% comment %}
### View stack traces
{% endcomment %}
{: #view-stack-traces }
### スタックトレースの確認

{% comment %}
The Docker daemon log can be viewed by using one of the following methods:
{% endcomment %}
Docker デーモンログは、以下に示すような方法により確認することができます。

{% comment %}
- By running `journalctl -u docker.service` on Linux systems using `systemctl`
- `/var/log/messages`, `/var/log/daemon.log`, or `/var/log/docker.log` on older
  Linux systems
{% endcomment %}
- `systemctl` を利用している Linux システム上にて `journalctl -u docker.service` を実行します。
- 古い Linux の場合は `/var/log/messages`、`/var/log/daemon.log`、`/var/log/docker.log` を確認します。

{% comment %}
> **Note**: It is not possible to manually generate a stack trace on Docker Desktop for
> Mac or Docker Desktop for Windows. However, you can click the Docker taskbar icon and
> choose **Diagnose and feedback** to send information to Docker if you run into
> issues.
{% endcomment %}
> **メモ**:
> Docker Desktop for Mac や Docker Desktop for Windows において、スタックトレースをコマンド操作によって取得することはできません。
> しかし問題が発生したときには、 Docker タスクバーアイコンをクリックし **Diagnose and feedback** を実行すれば、Docker に対して情報を送信することができます。

{% comment %}
Look in the Docker logs for a message like the following:
{% endcomment %}
Docker ログに出力された以下のようなメッセージを見てみます。

```none
...goroutine stacks written to /var/run/docker/goroutine-stacks-2017-06-02T193336z.log
...daemon datastructure dump written to /var/run/docker/daemon-data-2017-06-02T193336z.log
```

{% comment %}
The locations where Docker saves these stack traces and dumps depends on your
operating system and configuration. You can sometimes get useful diagnostic
information straight from the stack traces and dumps. Otherwise, you can provide
this information to Docker for help diagnosing the problem.
{% endcomment %}
Docker がスタックトレースやダンプを保存する場所は、オペレーティングシステムとその設定によって決まります。
スタックトレースやダンプからは、わかりやすい有用な診断情報が得られることがあります。
そうでないときには、Docker に対してこの情報を提供して、問題解決に役立てることができます。


{% comment %}
## Check whether Docker is running
{% endcomment %}
{: #check-whether-docker-is-running }
## Docker の起動確認

{% comment %}
The operating-system independent way to check whether Docker is running is to
ask Docker, using the `docker info` command.
{% endcomment %}
オペレーティングシステムを問わず Docker が動いているかどうかを確認するには `docker info` コマンドを実行します。

{% comment %}
You can also use operating system utilities, such as
`sudo systemctl is-active docker` or `sudo status docker` or
`sudo service docker status`, or checking the service status using Windows
utilities.
{% endcomment %}
あるいはオペレーティングシステムが提供するユーティリティーを用いることもできます。
たとえば `sudo systemctl is-active docker`、`sudo status docker`、`sudo service docker status` などです。
また Windows の場合は、サービス状態を確認するユーティリティーを利用します。

{% comment %}
Finally, you can check in the process list for the `dockerd` process, using
commands like `ps` or `top`.
{% endcomment %}
また `ps` や `top` コマンドを使えば、`dockerd` プロセス内のプロセス一覧を確認することができます。
