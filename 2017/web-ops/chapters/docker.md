# Dockerについて

[採用技術と研修での使い方](tools.md)にも書きましたが、Dockerはコンテナ型仮想化エンジンです。Dockerを用いると、隔離された環境でプロセスを動かすことができます。
「軽量VMとしての側面も持つ」と書きましたが、一般的に仮想マシンと呼ばれるものでは、ホストOSの上で仮想マシンを動かすためのハイパーバイザと呼ばれるソフトウェアが動き、その上でゲストOSが動き、更にその上でアプリケーションが動きます。一方でコンテナ型の仮想化エンジンであるDockerは、Dockerエンジンの上に、1つのプロセスが走るための隔離された空間を切ります。この空間をコンテナと呼びます。ホストOSと各コンテナではカーネルが共有されるため、ゲストOSのセットアップなどが必要ありません。そのため、仮想マシンに比べて高速に起動できます。アーキテクチャについてより詳しく知りたい場合は、[公式の解説](http://www.docker.com/what-docker)や各種資料を参照してください。

<img src="../assets/vm_and_docker.png" width="500" alt="vm_and_docker">

## Docker for Macのインストール

DockerはLinuxコンテナという技術をベースとしています。Macでは直接Linuxコンテナを使えないため、Dockerコンテナを立ち上げることができませんが、[Docker for Mac](https://docs.docker.com/docker-for-mac/)をインストールすることでこれが可能となります。Docker for Macを導入すると、[Docker Engine](https://docs.docker.com/engine/userguide/intro/)、[Docker Compose](https://docs.docker.com/compose/overview/)、[Docker Machine](https://docs.docker.com/machine/overview/)と、それらを扱うためのCLIがインストールされます。
Docker for Macが登場する前は、boot2dockerというツールや、Docker MachineでVirtualBoxの上にDockerホストを用意して、その上にコンテナをつくる方法が採られていました。2017年7月現在では、Docker for Macがデファクトスタンダードになっています。

## Dockerコンテナ

適当な作業用ディレクトリを切り、`Dockerfile`という名前のファイルを作ります。
`Dockerfile`には、Dockerイメージを構築する手順を記述します。

```dockerfile
# Dockerfile

FROM asuforce/puppet

RUN apt-get update -qq
```

最初に、`FROM`でベースとするイメージを指定する必要があります。今回は、簡単にPuppetを試すため[Dockerhub](https://hub.docker.com)に[asuforce/puppet](https://hub.docker.com/r/asuforce/puppet/)イメージを用意しましたので、これを使います。Dockerhubは、DockerイメージのGitHubのようなもので、多くのイメージが公開されています。asuforce/puppetのDockerfileもまた公開されており、上の内容と以下はほぼ等価です。

```dockerfile
FROM ubuntu:16.04

RUN apt-get update -qq \
  && apt-get install -qq wget
RUN wget https://apt.puppetlabs.com/puppet5-release-xenial.deb && dpkg -i puppet5-release-xenial.deb
RUN apt-get update -qq \
  && apt-get install -qq --no-install-recommends puppet-agent=5.0.0-1xenial ruby=1:2.3.0+1 \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*
RUN gem install bundler --no-document

ENV PATH /opt/puppetlabs/bin:$PATH
```

Dockerfileができたら、Dockerイメージを作ります。できたイメージは`docker images`コマンドで確認できます。

```console
$ docker build -t puppet:standalone .
```

イメージが作られていることが確認できたら、イメージIDを指定してコンテナを起動します。`-it`オプションは、今はコンテナ内で操作を行うためのオプションと覚えておいてください。

```console
$ docker run -it puppet:standalone
root@a671ad70ca3c:/#
```

## Docker Compose

Docker Composeは、複数のコンテナからなる構成の定義や起動を行うためのツールです。依存関係にあるコンテナを順番に起動したり、まとめてコンテナを止めることなどができます。
Docker Composeでは`docker-compose.yml`というファイルに定義を記述します。以下の内容ならば、`web001`、`bastion`という名前のコンテナが立ち上がります。コンテナの中ではinitプロセスが起動し、ホストであるMacの8080番ポートが`web001`の80番に、2222番が`bastion`の22番にそれぞれマッピングされます。

```yml
# docker-compose.yml

version: '3'
services:
  web001:
    build: .
    command: /sbin/init
    privileged: true
    hostname: web001
    ports:
      - '8080:80'
  bastion:
    build: .
    command: /sbin/init
    privileged: true
    hostname: bastion
    ports:
      - '2222:22'
```

```console
$ docker-compose up -d

... 中略

$ docker-compose ps
 Name      Command     State          Ports
---------------------------------------------------
bastion   /sbin/init   Up      0.0.0.0:2222->22/tcp
web001    /sbin/init   Up      0.0.0.0:8080->80/tcp
```

立ち上がったコンテナには、コンテナ名を指定して、以下のコマンドで入ることができます。

```console
$ docker-compose exec web001 bash
```
