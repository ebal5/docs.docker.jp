---
title: Docker App
description: Docker App について
keywords: Docker App, applications, compose, orchestration
---

{% comment %}
>This is an experimental feature.
>
>{% include experimental.md %}
{% endcomment %}
>これは試験的な（experimental）機能です。
>
>{% include experimental.md %}

{% comment %}
## Overview
{% endcomment %}
{: #overview }
## 概要

{% comment %}
Docker App is a CLI plug-in that introduces a top-level `docker app` command to bring
the _container experience_ to applications. The following table compares Docker containers with Docker applications.
{% endcomment %}
Docker App is a CLI plug-in that introduces a top-level `docker app` command to bring
the _container experience_ to applications. The following table compares Docker containers with Docker applications.


{% comment %}
| Object        | Config file   | Build with         | Execute with          | Share with        |
| ------------- |---------------| -------------------|-----------------------|-------------------|
| Container     | Dockerfile    | docker image build | docker container run  | docker image push |
| App           | App Package   | docker app bundle  | docker app install    | docker app push   |
{% endcomment %}
| オブジェクト  | 設定ファイル     | ビルドコマンド     | 実行コマンド          | プッシュコマンド  |
| ------------- |------------------| -------------------|-----------------------|-------------------|
| コンテナー    | Dockerfile       | docker image build | docker container run  | docker image push |
| アプリ        | アプリパッケージ | docker app bundle  | docker app install    | docker app push   |


{% comment %}
With Docker App, entire applications can now be managed as easily as images and containers. For example,
Docker App lets you  _build_, _validate_ and _deploy_ applications with the `docker app` command. You can
even leverage secure supply-chain features such as signed `push` and `pull` operations.
{% endcomment %}
With Docker App, entire applications can now be managed as easily as images and containers. For example,
Docker App lets you  _build_, _validate_ and _deploy_ applications with the `docker app` command. You can
even leverage secure supply-chain features such as signed `push` and `pull` operations.

{% comment %}
> **NOTE**: `docker app` works with `Engine - Community 19.03` or higher and `Engine - Enterprise 19.03` or higher.
{% endcomment %}
> **NOTE**: `docker app` works with `Engine - Community 19.03` or higher and `Engine - Enterprise 19.03` or higher.

{% comment %}
This guide walks you through two scenarios:
{% endcomment %}
This guide walks you through two scenarios:

{% comment %}
1. Initialize and deploy a new Docker App project from scratch.
1. Convert an existing Compose app into a Docker App project (added later in the beta process).
{% endcomment %}
1. Initialize and deploy a new Docker App project from scratch.
1. Convert an existing Compose app into a Docker App project (added later in the beta process).

{% comment %}
The first scenario describes basic components of a Docker App with tools and workflow.
{% endcomment %}
The first scenario describes basic components of a Docker App with tools and workflow.

{% comment %}
## Initialize and deploy a new Docker App project from scratch
{% endcomment %}
## Initialize and deploy a new Docker App project from scratch

{% comment %}
This section describes the steps for creating a new Docker App project to familiarize you with the workflow and most important commands.
{% endcomment %}
This section describes the steps for creating a new Docker App project to familiarize you with the workflow and most important commands.

{% comment %}
1. Prerequisites
1. Initialize an empty new project
1. Populate the project
1. Validate the app
1. Deploy the app
1. Push the app to Docker Hub or Docker Trusted Registry
1. Install the app directly from Docker Hub
{% endcomment %}
1. Prerequisites
1. Initialize an empty new project
1. Populate the project
1. Validate the app
1. Deploy the app
1. Push the app to Docker Hub or Docker Trusted Registry
1. Install the app directly from Docker Hub

{% comment %}
### Prerequisites
{% endcomment %}
### Prerequisites

{% comment %}
You need at least one Docker node operating in Swarm mode. You also need the latest build of the Docker CLI
with the App CLI plugin included.
{% endcomment %}
You need at least one Docker node operating in Swarm mode. You also need the latest build of the Docker CLI
with the App CLI plugin included.

{% comment %}
Depending on your Linux distribution and your security context, you might need to prepend commands with `sudo`.
{% endcomment %}
Depending on your Linux distribution and your security context, you might need to prepend commands with `sudo`.

{% comment %}
### Initialize a new empty project
{% endcomment %}
### Initialize a new empty project

{% comment %}
The `docker app init` command is used to initialize a new Docker application project. If you run it on
its own, it initializes a new empty project. If you point it to an existing `docker-compose.yml` file,
it initializes a new project based on the Compose file.
{% endcomment %}
The `docker app init` command is used to initialize a new Docker application project. If you run it on
its own, it initializes a new empty project. If you point it to an existing `docker-compose.yml` file,
it initializes a new project based on the Compose file.

{% comment %}
Use the following command to initialize a new empty project called "hello-world".
{% endcomment %}
Use the following command to initialize a new empty project called "hello-world".

```
$ docker app init --single-file hello-world
Created "hello-world.dockerapp"
```

{% comment %}
The command produces a single file in your current directory called `hello-world.dockerapp`.
The format of the file name is <project-name> appended with `.dockerapp`.
{% endcomment %}
The command produces a single file in your current directory called `hello-world.dockerapp`.
The format of the file name is <project-name> appended with `.dockerapp`.

```
$ ls
hello-world.dockerapp
```

{% comment %}
If you run `docker app init` without the `--single-file` flag, you get a new directory containing three YAML files.
The name of the directory is the name of the project with `.dockerapp` appended, and the three YAML files are:
{% endcomment %}
If you run `docker app init` without the `--single-file` flag, you get a new directory containing three YAML files.
The name of the directory is the name of the project with `.dockerapp` appended, and the three YAML files are:

{% comment %}
- `docker-compose.yml`
- `metadata.yml`
- `parameters.yml`
{% endcomment %}
- `docker-compose.yml`
- `metadata.yml`
- `parameters.yml`

{% comment %}
{% endcomment %}
However, the `--single-file` option merges the three YAML files into a single YAML file with three sections.
Each of these sections relates to one of the three YAML files mentioned previously: `docker-compose.yml`,
`metadata.yml`, and `parameters.yml`. Using the `--single-file` option enables you to share your application
using a single configuration file.

{% comment %}
{% endcomment %}
Inspect the YAML with the following command.

```
$ cat hello-world.dockerapp
# Application metadata - equivalent to metadata.yml.
version: 0.1.0
name: hello-world
description:
---
# Application services - equivalent to docker-compose.yml.
version: "3.6"
services: {}
---
# Default application parameters - equivalent to parameters.yml.
```

{% comment %}
{% endcomment %}
Your file might be more verbose.

{% comment %}
{% endcomment %}
Notice that each of the three sections is separated by a set of three dashes ("---"). Let's quickly describe each section.

{% comment %}
{% endcomment %}
The first section of the file specifies identification metadata such as name, version,
description and maintainers. It accepts key-value pairs. This part of the file can be a separate file called `metadata.yml`

{% comment %}
{% endcomment %}
The second section of the file describes the application. It can be a separate file called `docker-compose.yml`.

{% comment %}
{% endcomment %}
The final section specifies default values for application parameters. It can be a separate file called `parameters.yml`

{% comment %}
{% endcomment %}
### Populate the project

{% comment %}
{% endcomment %}
This section describes editing the project YAML file so that it runs a simple web app.

{% comment %}
{% endcomment %}
Use your preferred editor to edit the `hello-world.dockerapp` YAML file and update the application section with
the following information:

```
version: "3.6"
services:
  hello:
    image: hashicorp/http-echo
    command: ["-text", "${hello.text}"]
    ports:
      - ${hello.port}:5678
```

{% comment %}
{% endcomment %}
Update the `Parameters` section to the following:

```
hello:
  port: 8080
  text: Hello world!
```

{% comment %}
{% endcomment %}
The sections of the YAML file are currently order-based. This means it's important they remain in the order we've explained, with the _metadata_ section being first, the _app_ section being second, and the _parameters_ section being last. This may change to name-based sections in future releases.

{% comment %}
{% endcomment %}
Save the changes.

{% comment %}
{% endcomment %}
The application is updated to run a single-container application based on the `hashicorp/http-echo` web server image.
This image has it execute a single command that displays some text and exposes itself on a network port.

{% comment %}
{% endcomment %}
Following best practices, the configuration of the application is decoupled from the application itself using variables.
In this case, the text displayed by the app and the port on which it will be published are controlled by two variables defined in the `Parameters` section of the file.

{% comment %}
{% endcomment %}
Docker App provides the `inspect` subcommand to provide a prettified summary of the application configuration.
It is a quick way to check how to configure the application before deployment, without having to read
the `Compose file`. It's important to note that the application is not running at this point, and that
the `inspect` operation inspects the configuration file(s).

```
$ docker app inspect hello-world.dockerapp
hello-world 0.1.0

Service (1) Replicas Ports Image
----------- -------- ----- -----
hello       1        8080  hashicorp/http-echo

Parameters (2) Value
-------------- -----
hello.port     8080
hello.text     Hello world!
```

{% comment %}
{% endcomment %}
`docker app inspect` operations fail if the `Parameters` section doesn't specify a default value for
every parameter expressed in the app section.

{% comment %}
{% endcomment %}
The application is ready to be validated and rendered.

{% comment %}
{% endcomment %}
### Validate the app

{% comment %}
{% endcomment %}
Docker App provides the `validate` subcommand to check syntax and other aspects of the configuration.
If the app passes validation, the command returns no arguments.

```
$ docker app validate hello-world.dockerapp
Validated "hello-world.dockerapp"
```

{% comment %}
{% endcomment %}
`docker app validate` operations fail if the `Parameters` section doesn't specify a default value for
every parameter expressed in the app section.

{% comment %}
{% endcomment %}
As the `validate` operation has returned no problems, the app is ready to be deployed.

{% comment %}
{% endcomment %}
### Deploy the app

{% comment %}
{% endcomment %}
There are several options for deploying a Docker App project.

{% comment %}
{% endcomment %}
- Deploy as a native Docker App application
- Deploy as a Compose app application
- Deploy as a Docker Stack application

{% comment %}
{% endcomment %}
All three options are discussed, starting with deploying as a native Docker App application.

{% comment %}
{% endcomment %}
#### Deploy as a native Docker App

{% comment %}
{% endcomment %}
The process for deploying as a native Docker app is as follows:

{% comment %}
{% endcomment %}
Use `docker app install` to deploy the application.

{% comment %}
{% endcomment %}
Use the following command to deploy (install) the application.

```
$ docker app install hello-world.dockerapp --name my-app
Creating network my-app_default
Creating service my-app_hello
Application "my-app" installed on context "default"
```

{% comment %}
{% endcomment %}
By default, `docker app` uses the [current context]({{site.baseurl}}/engine/context/working-with-contexts) to run the
installation container and as a target context to deploy the application. You can override the second context
using the flag `--target-context` or by using the environment variable `DOCKER_TARGET_CONTEXT`. This flag is also
available for the commands `status`, `upgrade`, and `uninstall`.

```
$ docker app install hello-world.dockerapp --name my-app --target-context=my-big-production-cluster
Creating network my-app_default
Creating service my-app_hello
Application "my-app" installed on context "my-big-production-cluster"
```

{% comment %}
{% endcomment %}
> **Note**: Two applications deployed on the same target context cannot share the same name, but this is
valid if they are deployed on different target contexts.

{% comment %}
{% endcomment %}
You can check the status of the app with the `docker app status <app-name>` command.

```
$ docker app status my-app
INSTALLATION
------------
Name:         my-app
Created:      35 seconds
Modified:     31 seconds
Revision:     01DCMY7MWW67AY03B029QATXFF
Last Action:  install
Result:       SUCCESS
Orchestrator: swarm

APPLICATION
-----------
Name:      hello-world
Version:   0.1.0
Reference:

PARAMETERS
----------
hello.port: 8080
hello.text: Hello, World!

STATUS
------
ID              NAME            MODE          REPLICAS    IMAGE             PORTS
miqdk1v7j3zk    my-app_hello    replicated    1/1         hashicorp/http-echo:latest   *:8080->5678/tcp
```

{% comment %}
{% endcomment %}
The app is deployed using the stack orchestrator. This means you can also inspect it using the regular `docker stack` commands.

```
$ docker stack ls
NAME                SERVICES            ORCHESTRATOR
my-app              1                   Swarm
```

{% comment %}
{% endcomment %}
Now that the app is running, you can point a web browser at the DNS name or public IP of the Docker node on
port 8080 and see the app. You must ensure traffic to port 8080 is allowed on
the connection form your browser to your Docker host.

{% comment %}
{% endcomment %}
Now change the port of the application using `docker app upgrade <app-name>` command.
```
$ docker app upgrade my-app --set hello.port=8181
Upgrading service my-app_hello
Application "my-app" upgraded on context "default"
```

{% comment %}
{% endcomment %}
You can uninstall the app with `docker app uninstall my-app`.

{% comment %}
{% endcomment %}
#### Deploy as a Docker Compose app

{% comment %}
{% endcomment %}
The process for deploying as a Compose app comprises two major steps:

{% comment %}
{% endcomment %}
1. Render the Docker app project as a `docker-compose.yml` file.
2. Deploy the app using `docker-compose up`.

{% comment %}
{% endcomment %}
You need a recent version of Docker Compose to complete these steps.

{% comment %}
{% endcomment %}
Rendering is the process of reading the entire application configuration and outputting it as a single `docker-compose.yml` file. This creates a Compose file with hard-coded values wherever a parameter was specified as a variable.

{% comment %}
{% endcomment %}
Use the following command to render the app to a Compose file called `docker-compose.yml` in the current directory.

```
$ docker app render --output docker-compose.yml hello-world.dockerapp
```

{% comment %}
{% endcomment %}
Check the contents of the resulting `docker-compose.yml` file.

```
$ cat docker-compose.yml
version: "3.6"
services:
  hello:
    command:
    - -text
    - Hello world!
    image: hashicorp/http-echo
    ports:
    - mode: ingress
      target: 5678
      published: 8080
      protocol: tcp
```

{% comment %}
{% endcomment %}
Notice that the file contains hard-coded values that were expanded based on the contents of the `Parameters`
section of the project's YAML file. For example, `${hello.text}` has been expanded to "Hello world!".

{% comment %}
{% endcomment %}
> **Note**: Almost all the `docker app` commands propose the `--set key=value` flag to override a default parameter.

{% comment %}
{% endcomment %}
Try to render the application with a different text:

```
$ docker app render hello-world.dockerapp --set hello.text="Hello whales!"
version: "3.6"
services:
  hello:
    command:
    - -text
    - Hello whales!
    image: hashicorp/http-echo
    ports:
    - mode: ingress
      target: 5678
      published: 8080
      protocol: tcp
```

{% comment %}
{% endcomment %}
Use `docker-compose up` to deploy the app.

```
$ docker-compose up --detach
WARNING: The Docker Engine you're using is running in swarm mode.
<Snip>
```

{% comment %}
{% endcomment %}
The application is now running as a Docker Compose app and should be reachable on port `8080` on your Docker host.
You must ensure traffic to port `8080` is allowed on the connection form your browser to your Docker host.

{% comment %}
{% endcomment %}
You can use `docker-compose down` to stop and remove the application.

{% comment %}
{% endcomment %}
#### Deploy as a Docker Stack

{% comment %}
{% endcomment %}
Deploying the app as a Docker stack is a two-step process very similar to deploying it as a Docker Compose app.

{% comment %}
{% endcomment %}
1. Render the Docker app project as a `docker-compose.yml` file.
2. Deploy the app using `docker stack deploy`.

{% comment %}
{% endcomment %}
Complete the steps in the previous section to render the Docker app project as a Compose file and make sure
you're ready to deploy it as a Docker Stack. Your Docker host must be in Swarm mode.

```
$ docker stack deploy hello-world-app -c docker-compose.yml
Creating network hello-world-app_default
Creating service hello-world-app_hello
```

{% comment %}
{% endcomment %}
The app is now deployed as a Docker stack and can be reached on port `8080` on your Docker host.

{% comment %}
{% endcomment %}
Use the `docker stack rm hello-world-app` command to stop and remove the stack. You must ensure traffic to
port `8080` is allowed on the connection form your browser to your Docker host.

{% comment %}
{% endcomment %}
### Push the app to Docker Hub

{% comment %}
{% endcomment %}
As mentioned in the introduction, `docker app` lets you manage entire
applications the same way that you currently manage container images. For
example, you can push and pull entire applications from registries like Docker
Hub with `docker app push` and `docker app pull`. Other `docker app` commands,
such as `install`, `upgrade`, `inspect`, and `render` can be performed directly
on applications while they are stored in a registry.

{% comment %}
{% endcomment %}
Push the application to Docker Hub. To complete this step, you need a valid
Docker ID and you must be logged in to the registry to which you are pushing
the app.

{% comment %}
{% endcomment %}
By default, all platform architectures are pushed to the registry. If you are
pushing an official Docker image as part of your app, you may find your app
bundle becomes large with all image architectures embedded. To just push the
architecture required, you can add the `--platform` flag.

```bash
$ docker login

$ docker app push my-app --platform="linux/amd64" --tag <hub-id>/<repo>:0.1.0
```

{% comment %}
{% endcomment %}
### Push the app to DTR

{% comment %}
{% endcomment %}
Pushing an app to Docker Trusted Registry (DTR) involves the same procedure as
[pushing an app to Docker Hub](#push-the-app-to-docker-hub) except that you
need your DTR user credentials and [your DTR repository
information](/ee/dtr/user/manage-images/review-repository-info/). To use client
certificates for DTR authentication, see [Enable Client Certificate
Authentication](/ee/enable-client-certificate-authentication/).

{% comment %}
{% endcomment %}
By default, all platform architectures are pushed to DTR. If you are pushing an
official Docker image as part of your app, you may find your app bundle
becomes large with all image architectures embedded. To just push the
architecture required, you can add the `--platform` flag.

```bash
$ docker login dtr.example.com

$ docker app push my-app --platform="linux/amd64" --tag dtr.example.com/<user>/<repo>:0.1.0
```

{% comment %}
{% endcomment %}
### Install the app directly from Docker Hub or DTR

{% comment %}
{% endcomment %}
Now that the app is pushed to the registry, try an `inspect` and `install` command against it.
The location of your app is different from the one provided in the examples.

```
$ docker app inspect myuser/hello-world:0.1.0
hello-world 0.1.0

Service (1) Replicas Ports Image
----------- -------- ----- -----
hello       1        8080  myuser/hello-world@sha256:ba27d460cd1f22a1a4331bdf74f4fccbc025552357e8a3249c40ae216275de96

Parameters (2) Value
-------------- -----
hello.port     8080
hello.text     Hello world!
```

{% comment %}
{% endcomment %}
This action was performed directly against the app in the registry. Note that for DTR, the application will be prefixed with the Fully Qualified Domain Name (FQDN) of your trusted registry.

{% comment %}
{% endcomment %}
Now install it as a native Docker App by referencing the app in the registry, with a different port.

```
$ docker app install myuser/hello-world:0.1.0 --set hello.port=8181
Creating network hello-world_default
Creating service hello-world_hello
Application "hello-world" installed on context "default"
```

{% comment %}
{% endcomment %}
Test that the app is working.

{% comment %}
{% endcomment %}
The app used in these examples is a simple web server that displays the text "Hello world!" on port 8181,
your app might be different.

```
$ curl http://localhost:8181
Hello world!
```

{% comment %}
{% endcomment %}
Uninstall the app.

```
$ docker app uninstall hello-world
Removing service hello-world_hello
Removing network hello-world_default
Application "hello-world" uninstalled on context "default"
```

{% comment %}
{% endcomment %}
You can see the name of your Docker App with the `docker stack ls` command.