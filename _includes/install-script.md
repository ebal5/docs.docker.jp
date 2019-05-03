<!-- This file is included in Docker CE or EE installation docs for Linux. -->

<!--
### Install using the convenience script
-->
### 便利スクリプトを使ったインストール
{: #install-using-the-convenience-script }

<!--
Docker provides convenience scripts at [get.docker.com](https://get.docker.com/)
and [test.docker.com](https://test.docker.com/) for installing edge and
testing versions of Docker CE into development environments quickly and
non-interactively. The source code for the scripts is in the
[`docker-install` repository](https://github.com/docker/docker-install).
**Using these scripts is not recommended for production
environments**, and you should understand the potential risks before you use
them:
-->
Docker では [get.docker.com](https://get.docker.com/) と [test.docker.com](https://test.docker.com/) において便利なスクリプトを提供しています。
これは Docker CE の安定版あるいはテスト版を、開発機にすばやく対話形式をとらずにインストールするものです。
このスクリプトのソースコードは [`docker-install` リポジトリ](https://github.com/docker/docker-install)にあります。
このスクリプトを本番環境において利用することはお勧めしません。
またこのスクリプトの潜在的リスクについては、十分理解した上で利用してください。

<!--
- The scripts require `root` or `sudo` privileges to run. Therefore,
  you should carefully examine and audit the scripts before running them.
- The scripts attempt to detect your Linux distribution and version and
  configure your package management system for you. In addition, the scripts do
  not allow you to customize any installation parameters. This may lead to an
  unsupported configuration, either from Docker's point of view or from your own
  organization's guidelines and standards.
- The scripts install all dependencies and recommendations of the package
  manager without asking for confirmation. This may install a large number of
  packages, depending on the current configuration of your host machine.
- The script does not provide options to specify which version of Docker to install,
  and installs the latest version that is released in the "edge" channel.
- Do not use the convenience script if Docker has already been installed on the
  host machine using another mechanism.
-->
- スクリプトを実行するには ``root`` 権限か ``sudo`` が必要です。
  したがって十分に内容を確認してからスクリプトを実行するようにしてください。
- スクリプトは自動的に情報取得を行い、利用している Linux ディストリビューション、そのバージョン、そしてパッケージ管理システムの設定を行います。
  なおこのスクリプトは、インストール時にパラメーターを受け渡すような設定はできないものになっています。
  このことから時には、不適切な設定となる場合があります。
  それは Docker の観点の場合もあれば、開発現場のガイドラインや標準に対しての場合もあります。
- スクリプトはパッケージマネージャーによって、依存パッケージや推奨パッケージをすべてインストールします。
  その際にはインストールして良いかどうかを問いません。
  したがって相当数のパッケージがインストールされることもあります。
  これはホストマシンのその時点での設定によります。
- スクリプトには、インストールする Docker のバージョンを指定するようなオプションはありません。
  "エッジ" チャネルにてリリースされている最新版がインストールされます。
- ホストマシンに別の方法ですでに Docker をインストールしているのであれば、この便利スクリプトは使わないでください。

<!--
This example uses the script at [get.docker.com](https://get.docker.com/) to
install the latest release of Docker CE on Linux. To install the latest
testing version, use [test.docker.com](https://test.docker.com/) instead. In
each of the commands below, replace each occurrence of `get` with `test`.
-->
次の例は Linux に Docker CE の最新安定版リリースのインストールに [get.docker.com](https://get.docker.com/) のスクリプトを使います。
最新テスト版を使いたい場合は、代わりに [test.docker.com](https://test.docker.com/) を指定します。
その場合はコマンド中の `get` を `test` に置き換えて実行します。

> **警告**:
>
> インターネットからスクリプトをダウンロードしたら、まず内容を十分確認してから実行してください。
{:.warning}

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh

<output truncated>
```

<!--
If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:
-->
非 root ユーザーから Docker を利用したい場合、"docker" グループに属するユーザーを追加することを考えてください。
たとえば以下のとおりです。

```bash
  sudo usermod -aG docker your-user
```

<!--
Remember to log out and back in for this to take effect!
-->
いったんログアウトしてこのユーザーでのログインを行い、正しくログインできることを確認してください。

> **警告**:
>
> "docker" グループにユーザーを追加すれば、そのユーザーはコンテナーを実行できるようになります。
> このユーザーは Docker ホスト上では root 権限を得るために利用されます。
> 詳細については
> [Docker Daemon Attack Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface)
> を参照してください。
{:.warning}

<!--
Docker CE is installed. It starts automatically on `DEB`-based distributions. On
`RPM`-based distributions, you need to start it manually using the appropriate
`systemctl` or `service` command. As the message indicates, non-root users can't
run Docker commands by default.
-->
Docker CE がインストールされました。
`DEB` ベースのディストリビューションでは Docker が自動的に開始されます。
`RPM` ベースの場合は手動での実行が必要となるため、 `systemctl` か `service` のいずれか適当なものを実行します。
上の出力メッセージに示されているように、デフォルトで非 root ユーザーは Docker コマンドを実行できません。

<!--
#### Upgrade Docker after using the convenience script
-->
#### 便利スクリプトを使った後の Docker のアップグレード

<!--
If you installed Docker using the convenience script, you should upgrade Docker
using your package manager directly. There is no advantage to re-running the
convenience script, and it can cause issues if it attempts to re-add
repositories which have already been added to the host machine.
-->
便利スクリプトを使って Docker をインストールした場合、Docker のアップグレードはパッケージマネージャーを直接使って行ってください。
便利スクリプトは再実行する意味はありません。
ホストマシンにリポジトリが追加されているところに、このスクリプトを再実行したとすると、そのリポジトリを再度追加してしまうため、問題になることがあります。