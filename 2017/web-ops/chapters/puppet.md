# Puppetについて

サーバがどのような状態にあるべきかを記述したものを、Puppetではマニフェストと呼びます。Puppetにおいて、マニフェストを適用する方法は2つあります。localhostに置いてあるマニフェストをlocalhostに適用するやり方、つまり1台で完結する方法をStandaloneモードと呼び、各サーバがマニフェストを持った1台からカタログ（マニフェストをコンパイルしたもの）を取得して適用する方法をAgent/Masterモード（または Client/Serverモード）と呼びます。ここではStandaloneモードから始めて、その後Agent/Masterモードを試します。
各モードの詳細は、[Puppetのドキュメント](https://docs.puppet.com/puppet/latest/reference/architecture.html)を参照してください。

## Standalone

`manifests`ディレクトリを切り、最初のマニフェスト`site.pp`をつくります。

```puppet
# manifests/site.pp

notify { 'hello':
  message => 'hello, puppet!'
}
```

[notifyリソース](https://docs.puppet.com/puppet/latest/reference/type.html#notify)を使い、メッセージを出力するだけのマニフェストです。Dockerコンテナの中に設置する必要があるので、Dockerfileを以下のように編集します。

```dockerfile
# Dockerfile

FROM asuforce/puppet

RUN apt-get update -qq

ADD . /etc/puppet
```

この`ADD`によって、このディレクトリの内容がコンテナの`/etc/puppet`以下にコピーされます。コピーする必要がないものは`.dockerignore`というファイル内で指定します。

```dockerignore
# .dockerignore

docker-compose.yml
Dockerfile
/vendor
.DS_Store
.git
.librarian
.bundle
```

今起動しているコンテナは古いDockerfileから作られたものなので、コンテナを立て直します。その後、`/etc/puppet`以下にマニフェストが配置されていることを確認します。

```console
$ docker-compose up -d --build
$ docker-compose exec web001 cat /etc/puppet/manifests/site.pp
notify { 'hello':
  message => 'hello, puppet!'
}
```

マニフェストを適用してみます。以下の様な出力が得られれば成功です。

```console
$ docker-compose exec web001 puppet apply /etc/puppet/manifests/site.pp
Notice: Compiled catalog for localhost.local in environment production in 0.01 seconds
Notice: hello, puppet!
Notice: /Stage[main]/Main/Notify[hello]/message: defined 'message' as 'hello, puppet!'
Notice: Finished catalog run in 0.02 seconds
```

## Puppet Module

マニフェストをStandaloneモードで適用できましたが、単にhelloと出力するだけでは実用性がありませんので、次はNginxをインストールします。Puppetではマニフェストを再利用可能にするためにモジュールという仕組みがあり、Nginxのインストールや設定を行うモジュールがつくられているので、それを利用します。モジュールは[Puppet Forge](https://forge.puppet.com)から取得することができますが、一般的に[librarian-puppet](https://github.com/voxpupuli/librarian-puppet)を使って管理します。Rubyで例えると、モジュールはgem、Puppet ForgeはRubygems、librarian-puppetはbundlerに相当します。

librarian-puppetはRubyのgemとして入手できるので、まずはGemfileを作ります。

```ruby
# Gemfile

source 'https://rubygems.org'

gem 'puppet'
gem 'librarian-puppet'
```

`bundle install --path vendor/bundle`を実行し、Gemfile.lockを作成します。
その後、librarian-puppetで管理するモジュールをPuppetfileというファイルに記述します。今回はnginxのインストールに[puppet-nginx](https://forge.puppet.com/puppet/nginx)を使います。

```ruby
# Puppetfile

forge 'https://forgeapi.puppetlabs.com'

mod 'puppet-nginx'
```

`bundle exec librarian-puppet install --path vendor/modules`を実行し、Puppetfile.lockを作成してください。これはGemfile.lockのように、Puppet Moduleのバージョン管理を行い、開発環境と運用環境で同じPuppet Moduleを使うことを保証します。

Dockerコンテナ内で`bundle install`等を行いたいので、Dockerfileに追記します。

```dockerfile
# Dockerfile

FROM asuforce/puppet

RUN apt-get update -qq \
  && apt-get install -qq git

ADD . /etc/puppet

WORKDIR /etc/puppet
RUN bundle install --path vendor/bundle
RUN bundle exec librarian-puppet install
```

ここまでで、puppet-nginxモジュールを使う準備ができたので、どのように使用するかを見ていきます。
モジュールには一般的に1つ以上のクラスが含まれます。Puppetにおけるクラスは、プログラミングにおける関数のような存在です。引数を取り、ある1つのことを行います。puppet-nginxモジュールは`nginx`クラスを提供しており、以下のように使うことでNginxのインストールと起動を行うことができます。

```puppet
# 引数を取る場合の書き方
class { 'nginx':
  package_source => 'nginx-stable'
}

# 引数を取らない場合の書き方
# 引数の一部はデフォルト値が設定されている
class { 'nginx': }

# 引数を取らない場合は以下の書き方でもよい
# :: によってトップレベルスコープのnginxクラスを指定している
include ::nginx
```

`nginx`クラスは単にNginxのインストールと起動をするだけなので、追加の設定はjfryman-nginxモジュールが定義した[リソース](https://docs.puppet.com/puppet/latest/reference/lang_defined_types.html)を使用して行います。ここでは`nginx::resource::vhost`を使ってドキュメントルートを設定します。同時に、[fileリソース](https://docs.puppet.com/puppet/latest/reference/types/file.html)でそこに置くHTMLファイルを作ります。

```puppet
# manifests/site.pp

# jfryman-nginxが用意したクラスを利用する
include ::nginx

# /var/www/htmlをドキュメントルートに設定する
# これの前に必要な物をrequireで指定する
nginx::resource::server { 'set www root':
  www_root => '/var/www/html',
  require  => [Package['nginx'], Class['Nginx::Config']],
}

# /var/www/html/index.htmlを配置するためにディレクトリを用意する
# $dirs のように変数を使用できる
$dirs = ['/var/www', '/var/www/html']
file { $dirs:
  ensure => directory,
}

# /var/www/html/index.htmlを配置する
file {'/var/www/html/index.html':
  content => '<h1>Hello from Docker Container</h1>',
}
```

クラスとリソースは、どちらも幾つかの引数をとり、特定の処理を行うものです。最も大きな違いは、クラスは複数回使うことができないという点です。使い分けの目安は、Nginxをインストールする等のある程度まとまった処理はクラスで書き、`file`や`user`のように何度か使用したいものや、リソースとしてモデリングしたほうが自然である場合はリソースとして定義するのがよいでしょう。

ではNginxをインストールするため、コンテナを立てなおして、マニフェストを適用します。

```console
$ docker-compose up -d --build
$ docker-compose exec web001 puppet apply --modulepath modules /etc/puppet/manifests/site.pp
```

`web001`の80番ポートはホストの8080番からアクセスできるように設定してあるので、[http://localhost:8080](http://localhost:8080)をブラウザで開いて、"Hello from Docker Container"が表示されればNginxが正常に動いています。

## Agent/Master

これまでの Standalone モードは、サーバを増やすと全てのサーバに対して、マニュフェストを置く必要がありました。それを解決する Agent/Master モードについて触ります。
この研修では Puppetserver を作成し、他のサーバにはマニュフェストを管理させないようにします。
Puppetserver を起動するサーバを master とし、その他のサーバを agent と呼びます。

### コンテナの準備

AgentとMasterでセットアップの方法が異なるため、Dockerfileも2つ用意します。Agentにはマニフェストを置かないので、Dockerfileはほぼ空です。Masterには[asuforce/puppetserver](https://hub.docker.com/r/asuforce/puppetserver/)というイメージを使用します。

このイメージは[asuforce/puppet](https://hub.docker.com/r/asuforce/puppet/)をベースイメージとしpuppetserverをインストール、起動を行います。`Dockerfile-master`ではそれに加えて、マニフェストの配置とPuppetモジュールのインストールを行います。

```dockerfile
# Dockerfile-agent
FROM asuforce/puppet

RUN apt-get update -qq
```

```dockerfile
# Dockerfile-master

FROM asuforce/puppetserver

RUN apt-get update -qq \
  && apt-get install -qq git

WORKDIR /etc/puppetlabs/code/environments
RUN cp -r production development
ADD . development

WORKDIR /etc/puppetlabs/code/environments/development
RUN bundle install --path vendor/bundle
RUN bundle exec librarian-puppet install
```

docker-compose.ymlは以下のようになります。

```yaml
version: '3'
services:
  web001:
    build:
      context: .
      dockerfile: Dockerfile-agent
    command: /sbin/init
    privileged: true
    hostname: web001.local
    ports:
      - '8080:80'
    links:
      - pmaster:pmaster.local
  pmaster:
    build:
      context: .
      dockerfile: Dockerfile-master
    command: /sbin/init
    privileged: true
    hostname: pmaster.local
```

適用するマニフェストは、これまでと同じNginxのインストールを行うものです。ただし、どのホストにどのマニフェストが適用されるかを、`node`を使って指定します。

```puppet
# manifests/site.pp

node 'web001.local' {
  include ::nginx

  nginx::resource::server { 'set www root':
    www_root => '/var/www/html',
    require  => [Package['nginx'], Class['Nginx::Config']],
  }

  $dirs = ['/var/www', '/var/www/html']
  file { $dirs:
    ensure => directory,
  }

  file { '/var/www/html/index.html':
    content => '<h1>Hello from Docker Container</h1>',
  }
}
```

これらを記述できたら以下を実行してコンテナを立ち上げます。

```console
$ docker-compose up -d --build
```

### Puppetserverの起動

以下のコマンドを実行し、[systemd](https://www.freedesktop.org/wiki/Software/systemd/)経由でPuppetserverを起動します。

```console
$ docker-compose exec pmaster systemctl start puppetserver
```

### マニフェストの適用

ここまで準備ができたら、Masterからカタログを取得するためAgentで以下のように`puppet agent`コマンドを実行します。`--server`オプションでMasterを指定します。`--test`は、1度だけマニフェストの適用を試すときに用いられます。実体は複数のオプションをまとめて指定するエイリアスです。`--noop`は実際の適用はされず、適用される内容を一度確認する時に使います。このことをDry Runと呼びます。

```console
# Dry Run で適用される内容を確認
$ docker-compose exec web001 puppet agent --test --server pmaster.local --environment development --noop

# 適用
$ docker-compose exec web001 puppet agent --test --server pmaster.local --environment development
```

[http://localhost:8080](http://localhost:8080)で前と同じHTMLが見えれば成功です。`Skipping run of Puppet configuration client`といったメッセージが出た場合は、`puppet agent --enable`を行ってから再度カタログ要求を行ってください。

### autosignについて


実際にVMを使って、Puppetserverを作成し、AgentノードでPuppetを適用すると初回は証明書が無いというエラーが出て失敗します。しかし、皆さんは特に失敗することなくHTMLを見ることができたと思います。

これは[asuforce/puppetserver](https://hub.docker.com/r/asuforce/puppetserver/)上で[autosign](https://docs.puppet.com/puppet/latest/reference/ssl_autosign.html)という仕組みを使っているからです。

PuppetのAgentとMasterの間の通信は全てSSLで暗号化されて行われます。このときに使用するサーバ証明書をつくるため、AgentからMasterへ署名要求（CSR）が送られています。`puppet cert`コマンドでMasterによる署名を行ったあと、再度Agentからカタログを要求します。以下のコマンドは手動の場合の手順になります。詳細なステップは[公式のドキュメント](https://docs.puppet.com/puppet/latest/man/cert.html)を参照してください。

この署名を自動で行う仕組みがautosignになります。使う場合、信頼できるホストのみが署名要求を行えるように、ネットワークを隔離するなどの工夫が必要です。

```console
$ docker-compose exec pmaster puppet cert --list
$ docker-compose exec pmaster puppet cert sign web001.local
$ docker-compose exec web001 puppet agent --test --server pmaster.local
```

## Puppetの文法

ここまでNginxをインストールするマニフェストを書いてきましたが、`->`、`~>`、`<| |>`など、カバーできていないPuppetの文法がいくつも存在します。言語としての概要を把握するには、公式の[Visual Index](https://docs.puppet.com/puppet/latest/reference/lang_visual_index.html)や[The Puppet Language Style Guide](https://docs.puppet.com/puppet/latest/style_guide.html)というページが役に立ちます。前者はコンパクトにまとまっているので、目を通すだけならば時間もかからないと思います。後者については文量はありますが、活用するとより綺麗に Puppet を書けるようになります。また、より理解を深めたい場合は@antipopの[入門Puppet](https://github.com/kentaro/puppet-book)に挑戦することをおすすめします。
