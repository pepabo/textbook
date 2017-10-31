# 採用技術と研修での使い方

## Docker

[Docker](https://www.docker.com/)はオープンソースのコンテナ型仮想化エンジンです。

Dockerではアプリケーションとその実行環境を「コンテナ」と呼ばれる単位でパッケージングでき、そこから作ったイメージ（コンテナの元となるもの）をそのまま別のマシンへ移して動かすことが可能です。
チーム内に配布すれば全員が同じ環境が使用できますし、イメージを本番環境へデプロイして動かす事例も発表されています。
Web開発研修で使用したherokuも、[Dockerイメージをそのままデプロイできる仕組みを持っています](https://devcenter.heroku.com/changelog-items/926)。

ペパボでは、社内CIでアプリケーションのテストを行う際や、SUZURIの画像合成サーバ、minneのアプリケーションの開発環境にDockerが利用されています。

また、詳しくは[Dockerについて](docker.md)で述べますが、Dockerコンテナはその仕組み上、[VirtualBox](https://www.virtualbox.org/)などによる仮想マシンよりも起動が高速で、軽量VMとしての側面も持ち合わせています。

実際に社内でコンテナクラスタ管理システムの[Kubernetes](https://kubernetes.io/)を構築できるような仕組みが検証されています。
先の話なので断言はできませんが、Dockerベースのデプロイが安定して稼働すれば、手元のDockerコンテナからイメージをつくり、それをそのままNyahの上にデプロイすればサービスが動く未来が来るかもしれません。

以上の実績と優位性、将来性から、Dockerを使用します。

### 参考

- [Docker基礎](http://www.slideshare.net/mainya/dockerdocker09-010)

## Puppetによる構成管理

インフラの構成管理とは、サービスを正常に提供することができるようにサーバの状態を管理することを指します。全てをシェルスクリプトで管理するのも構成管理の1つのやり方ですが、近年は構成管理ツールと呼ばれるもの使って、サーバがあるべき状態をコードで管理することが一般的です。
サーバがあるべき状態をロールと呼び、多くの場合ロールは複数存在します。インフラ構築時に構成管理ツールが担うのは、サーバを立ててOSをインストールした後の、ミドルウェアのインストールや設定変更をしてサービスを提供できるようにする過程です。継続的に構成管理ツールを適用して、あるべき状態と実際の状態の乖離を常に防ぐ運用が行われる場合もあります。

構成管理ツールとして、ペパボでは[Puppet](https://puppet.com/product/how-puppet-works)が主に使われており、この研修でもこれを使います。研修の目的としては、ツールの使い方を学ぶことよりも、自動化や構成管理を実際に行い、その利点を学んでもらうことが重要ではありますが、実際のサービスに用いられているPuppetマニフェストなどを参考にしながら、Puppet自体についても学んで欲しいと思います。[Puppet](puppet.md)でも触れていますが、この研修ではAgent/Masterと呼ばれる方式で使っていきます。

なお、他にも[Chef](https://www.chef.io/chef/), [Itamae](http://itamae.kitchen/), [Ansible](https://www.ansible.com/)などが代表的な構成管理ツールとして知られています。興味のある人は調べてみてください。

## Serverspecによるテスト

Web開発研修でテストを書いたように、Webオペレーション研修でもテストを書いていきます。使用するツールは[Serverspec](https://github.com/mizzy/serverspec)を使用します。
Serverspecは、内部の仕組みがわかっている前提で、設定ファイルが配置されているか、その内容が適切か、といった項目をテストしていきます。

また[infrataster](https://github.com/ryotarai/infrataster)というツールもあり、外から見たときの動作をテストします。
こちらの使用は必須では無いので、計画に余裕がある場合は調査し導入を検討してみましょう。

## Mackerel について

今回の研修では利用しませんが、ペパボでは Mackerel というツールを用いて、サーバの監視を行なっています。
アカウントを作成し、実際のサービスメトリックを見てみましょう。

### 監視とは

サーバに障害が起きたことや、負荷が増加していることを検知して対策するために、サーバの状態を監視します。サーバ監視ツールとして、[Zabbix](http://www.zabbix.com/)、[Nagios](https://www.nagios.org/)、[Munin](http://munin-monitoring.org/)などがあり、モダンなものとして[Datadog](https://www.datadoghq.com/)や[Prometheus](https://prometheus.io/)があります。先程も書いたようにペパボでは、はてなが運営する[Mackerel](https://mackerel.io)という監視サービスを主に利用しています。