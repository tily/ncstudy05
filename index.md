---
layout: page
title: ncstudy#05 ハンズオン資料
tagline: 2013/07/14
---
{% include JB/setup %}

講師：[@tily](https://github.com/tily)

Chef Apply と Chef Solo と serverspec とニフティクラウドとニフティクラウドオートメーションβが試せるお得なハンズオンです。

<hr />

## 0. 目次

<ul>
  <li>
    <a href="#1_">1. 準備</a>
    <ul>
      <li><a href="#11_ssh_">1.1. SSH でサーバーへログイン</a></li>
      <li><a href="#12_chef_solo_">1.2. Chef Solo のインストール</a></li>
    </ul>
  </li>
  <li>
    <a href="#2_chef_apply_">2. Chef Apply を試す</a>
    <ul>
      <li><a href="#21_chef_apply_">2.1. Chef Apply とは</a></li>
      <li><a href="#22_">2.2. レシピの作成と実行</a></li>
      <li><a href="#23_">2.3. べき等性の確認</a></li>
    </ul>
  </li>
  <li>
    <a href="#3_chef_solo__wordpress_">3. Chef Solo で WordPress レシピ開発</a>
    <ul>
      <li><a href="#31_chef_solo_">3.1. Chef Solo 用の設定ファイル配置</a></li>
      <li><a href="#32_wordpress_">3.2. WordPress レシピのダウンロード</a></li>
      <li><a href="#33_">3.3. レシピ実行</a></li>
    </ul>
  </li>
  <li>
    <a href="#4_">4. レシピのテストを書く</a>
    <ul>
      <li><a href="#41_serverspec_">4.1. serverspec のインストール</a></li>
      <li><a href="#42_httpd_">4.2. httpd のテストを修正</a></li>
      <li><a href="#43_mysqld_">4.3. mysql のテストを作成</a></li>
      <li><a href="#44_wordpress_">4.4. wordpress のテストを作成</a></li>
      <li><a href="#45_">4.5. 進んだ使い方</a></li>
    </ul>
  </li>
  <li>
    <a href="#5_cloudautomation__">5. CloudAutomation β で自動化！</a>
    <ul>
      <li><a href="#51_">5.1. レシピのアップロード</a></li>
      <li><a href="#52_json_">5.2. JSON ファイル作成</a></li>
      <li><a href="#53_">5.3. コントロールパネルから実行</a></li>
    </ul>
  </li>
</ul>

## 1. 準備

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 1.1. SSH でサーバーへログイン

#### Windows の場合

<ol>
  <li> TeraTerm を起動</li>
  <li> 「ホスト」に当日配布の IP アドレスを入力</li>
  <li> 「TCP ポート」に "22" を入力</li>
  <li> 「OK」ボタンをクリック (次の画面が表示されます)</li>
  <li> 「ユーザ名」に "root" を入力</li>
  <li> 「パスフレーズ」に当日配布のパスフレーズを入力</li>
  <li> 「RSA/DSA 鍵を使う」を選択し「秘密鍵」ボタンから当日配布する秘密鍵を指定</li>
  <li> 「OK」ボタンをクリック</li>
</ol>

下記のようなプロンプトが表示されればログイン成功です。

    [root@localhost ~]#

#### Mac OS X の場合

通常の SSH ログインコマンドでログイン可能です。

    chmod 600 ncstudy05_private.pem
    ssh -i /path/to/ncstudy05_private.pem root@[グローバルIPアドレス]


<div class="pull-right"><a href="#0_">目次へ</a></div>

### 1.2. Chef Solo のインストール

下記コマンドで一発でインストールすることができます。

    # curl -L https://www.opscode.com/chef/install.sh | bash

ちなみにバージョンを固定したい場合は -v で指定したり RPM から直接インストールすることも可能です。

    # bash install.sh -v 11.4.4.-2
    # rpm -ivh https://opscode-omnibus-packages.s3.amazonaws.com/el/6/x86_64/chef-11.4.4-2.el6.x86_64.rpm

正常にインストールが完了したかどうか chef-solo コマンドで確認してみましょう。

    # chef-solo
    [2013-07-09T17:15:39+09:00] WARN: *****************************************
    [2013-07-09T17:15:39+09:00] WARN: Did not find config file: /etc/chef/solo.rb, using command line options.
    [2013-07-09T17:15:39+09:00] WARN: *****************************************
    Starting Chef Client, version 11.4.0
    Compiling Cookbooks...
    Converging 0 resources
    Chef Client finished, 0 resources updated
    # which chef-solo
    /usr/bin/chef-solo

Chef 関連のモジュールは /opt/chef 配下にインストールされるので ls で閲覧してみてください。

    # ls /opt/chef/bin/
    chef-apply  chef-client  chef-shell  chef-solo  erubis  knife  ohai  restclient  shef

## 2. Chef Apply を試す

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 2.1. Chef Apply とは

`chef` gem をインストールすると、`chef-apply` というコマンドもインストールされます。
`chef-solo` よりも手軽に Chef の「べき等性」が試せるので、少し触ってみましょう。

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 2.2. レシピの作成と実行

まずはかんたんなレシピを作成します。
ファイルを作成して中に "hello world" と書き込むだけのものです。

    # vi recipe.rb
    ## 下記内容を書き込みます
    file '/var/tmp/test.txt' do
      content "hello, world\n"
    end

実行してみます。

    # chef-apply recipe.rb
    Recipe: (chef-apply cookbook)::(chef-apply recipe)
      * file[/var/tmp/test.txt] action create
        - create new file /var/tmp/test.txt with content checksum 853ff9
            --- /tmp/chef-tempfile20130709-856-1n2lyfc      2013-07-09 17:19:24.658237382 +0900
            +++ /tmp/chef-diff20130709-856-1qjl5ez  2013-07-09 17:19:24.658237382 +0900
            @@ -0,0 +1 @@
            +hello, world

ファイルが作成されたようです。実際に確認してみます。

    # ls /var/tmp/test.txt
    /var/tmp/test.txt
    # cat /var/tmp/test.txt
    hello, world

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 2.3. べき等性の確認

もう一度実行してみます。

    # chef-apply recipe.rb
    Recipe: (chef-apply cookbook)::(chef-apply recipe)
      * file[/var/tmp/test.txt] action create (up to date)

今度は up to date と表示され何も起きません。これはサーバーがレシピに書かれた内容通りの状態になっていることを Chef が認識し、ファイル作成をスキップしたためです。

このように「何度実行しても結果が同じになる」ような性質を Chef (や構成管理ツール) の世界では「<strong>idemponent (べき等)</strong>」と呼んでいます。

次にわざと作成されたファイルを書き換えた上で実行してみましょう。

    # echo hello nifty cloud > /var/tmp/test.txt
    # cat /var/tmp/test.txt
    hello nifty cloud
    # chef-apply recipe.rb
    Recipe: (chef-apply cookbook)::(chef-apply recipe)
      * file[/var/tmp/test.txt] action create
        - update content in file /var/tmp/test.txt from bac308 to 853ff9
            --- /var/tmp/test.txt   2013-07-09 17:28:37.003242663 +0900
            +++ /tmp/chef-diff20130709-2021-umd0np  2013-07-09 17:28:54.504237642 +0900
            @@ -1 +1 @@
            -hello nifty cloud
            +hello, world

ファイルが書き換わったことを認識して Chef がレシピ通りの状態に復元してくれたことが確認できたと思います。

今度は recipe.rb 自体のほうを書き換えた上で Chef 実行してみます。

    # vi recipe.rb
    ## world を chef に書き換えた
    file '/var/tmp/test.txt' do
      content "hello, chef\n"
    end
    # chef-apply recipe.rb
    Recipe: (chef-apply cookbook)::(chef-apply recipe)
      * file[/var/tmp/test.txt] action create
        - update content in file /var/tmp/test.txt from 853ff9 to 5b8079
            --- /var/tmp/test.txt   2013-07-09 17:28:54.557365556 +0900
            +++ /tmp/chef-diff20130709-2432-3waga3  2013-07-09 17:31:54.521237988 +0900
            @@ -1 +1 @@
            -hello, world
            +hello, chef

この場合にもサーバーの状態がレシピに記述された状態とは異なるので変更が適用されます。

このように、サーバーに変更が反映されるのは、下記の 2 つの場合のみです。

<ul>
  <li>サーバーの状態がレシピと異なってしまった場合</li>
  <li>レシピ自体が更新された場合</li>
</ul>

## 3. Chef Solo で WordPress レシピ開発

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 3.1. Chef Solo 用の設定ファイル配置

それでは簡易版ではなく、本格的な Cookbook をダウンロードして実行してみましょう。

まずは準備として Chef Solo が cookbooks 置き場として利用するディレクトリを作成しておきます。

    mkdir -p /var/chef/cookbooks

あと、少し煩雑ですがこの後の手順で knife cookbook site install コマンドを利用するために、/var/chef/cookbooks を git レポジトリにしておきます。

    yum install -y git
    cd /var/chef/cookbooks
    git init .
    touch README
    git add README
    git commit -m "add readme."
    
<div class="pull-right"><a href="#0_">目次へ</a></div>

### 3.2. WordPress レシピのダウンロード

それでは、Opscode のコミュニティサイトから knife コマンドを利用して wordpress cookbook をダウンロードしましょう。

下記のコマンドを実行します。

    # knife cookbook site install wordpress
    ## wordpress および wordpress が依存

/var/chef/cookbooks を見てみると、いくつかの cookbooks がインストールされているのが分かります。

    # cd /var/chef/cookbooks
    # ls
    README  apache2  build-essential  mysql  openssl  php  wordpress  xml

自動でブランチも切られているので、オリジナルの cookbook との差分を確認しながら cookbook を開発することができます。

    # git branch
      chef-vendor-apache2
      chef-vendor-build-essential
      chef-vendor-mysql
      chef-vendor-openssl
      chef-vendor-php
      chef-vendor-wordpress
      chef-vendor-xml

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 3.3. レシピ実行

cookbooks が正常にダウンロードできたので、実行してみます。

まずは設定ファイルを作成します。

    # vi dna.json
    ## 下記の内容を書き込む
    {
      "run_list": ["recipe[wordpress]"],
      "mysql": {
        "server_root_password": "password",
        "server_debian_password": "password",
        "server_repl_password": "password"
      }
    }

上記ファイルを指定して chef-solo を実行してみます。

    # chef-solo -j dna.json

ずらずらと Chef のログが流れはじめます。しばらくして下記のようなメッセージが表示されたら実行完了です。

    ## これより上のログは省略
    Recipe: apache2::default
      * service[apache2] action restart
        - restart service service[apache2]
    
    Recipe: wordpress::default
      * log[wordpress_install_message] action write
    
    Chef Client finished, 109 resources updated

当日配布するグローバル IP アドレスをブラウザに張り付けてみましょう。

WordPress のインストール画面が表示されたら成功です。

## 4. レシピのテストを書く

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 4.1. serverspec のインストール

せっかくなので、たった今行った動作確認の作業を今話題の serverspec で自動化してみましょう。

まず `gem` コマンドでインストールするため、Chef Solo をインストールしたときについてきた gem にパスを通しておきます。

    # export PATH=$PATH:/opt/chef/embedded/bin/
    # gem -v
    1.8.24

serverspec をインストールします。

    # gem install serverspec

`serverspec-init` というコマンドがインストールされます。

このコマンドでテストのひな形を作ります。
backend type (SSH 経由でテストするかローカルでテストするか) を聞かれるので、今回は「2) Exec (local) 」を選択しましょう。

(もちろんニフティクラウドのサーバーは SSH 経由でもテスト可能です。)

    # serverspec-init
    Select a backend type:
    
      1) SSH
      2) Exec (local)
    
    Select number: 2     ## 2 を入力
    
     + spec/
     + spec/localhost/
     + spec/localhost/httpd_spec.rb
     + spec/spec_helper.rb
     + Rakefile

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 4.2. httpd のテストを修正

この状態で `rake spec` コマンドを実行すると、デフォルトで作成された httpd のテストが失敗します。

    # rake spec
    /opt/chef/embedded/bin/ruby -S rspec spec/localhost/httpd_spec.rb
    .....F
    
    Failures:
    
      1) File "/etc/httpd/conf/httpd.conf"
         Failure/Error: it { should contain "ServerName localhost" }
           grep -q -- ServerName\ localhost /etc/httpd/conf/httpd.conf
           expected File "/etc/httpd/conf/httpd.conf" to contain "ServerName localhost"
         # ./spec/localhost/httpd_spec.rb:18:in `block (2 levels) in <top (required)>'
    
    Finished in 0.06117 seconds
    6 examples, 1 failure
    
    Failed examples:
    
    rspec ./spec/localhost/httpd_spec.rb:18 # File "/etc/httpd/conf/httpd.conf"
    rake aborted!
    /opt/chef/embedded/bin/ruby -S rspec spec/localhost/httpd_spec.rb failed
    
    Tasks: TOP => spec
    (See full trace by running task with --trace)


今回はドメイン取得は行わないので、ServerName のテストは削除しておきます。

削除した上、rake spec で `F` が表示されなくなったら、次へ進みましょう。

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 4.3. mysqld のテストを作成

`httpd_spec.rb` を参考に、mysqld のテストを書いてみます。

下記が確認できるようにしましょう。

<ul>
  <li>mysql-server パッケージがインストールされていること</li>
  <li>mysqld デーモンが有効化されていること (chkconfig mysqld on されていること)</li>
  <li>mysqld デーモンが起動していること</li>
  <li>3306 ポートが LISTEN していること</li>
</ul>

    # vi ./spec/localhost/mysqld_spec.rb

(回答例はこちら：[https://gist.github.com/tily/5990140](https://gist.github.com/tily/5990140))

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 4.4. wordpress のテストを作成

次に、[serverspec のドキュメント](http://serverspec.org/resource_types.html) を参照しながら、
下記を確認できるようにしてみます。

<ul>
  <li>http://localhost/wp-admin/install.php にアクセスすると "Welcome to the famous five minute WordPress installation process!" という文字列が表示されること</li>
</ul>

    # vi ./spec/localhost/wordpress_spec.rb

(回答例はこちら：[https://gist.github.com/tily/5990140](https://gist.github.com/tily/5990140))

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 4.5 進んだ使い方

<strong>HTML 出力</strong>

    # rake spec SPEC_OPTS="--format html"

<strong>JUnit 形式の XML (Jenkins で利用可能) へ変換</strong>

    # gem install ci_reporter

    # vi Rakefile
    ## 下記行を追加
    require 'ci/reporter/rake/rspec'

    # rake ci:setup:rspec spec
    ## spec/reports 配下に XML ファイルが生成される

このような仕組みにより、こんな使い方をすることが可能。

<ul>
  <li>既存のサーバーで定期的に serverspec を実行し HTML 出力結果をメールで通知</li>
  <li>Jenkins で定期的にニフティクラウドのサーバーを立ち上げ serverspec のテスト結果を出力</li>
</ul>

参考：

* [Vagrant + Chef Solo + serverspec + Jenkins でサーバー構築を CI - naoyaのはてなダイアリー](http://d.hatena.ne.jp/naoya/20130520/1369054828)
* [Vagrantからニフティクラウド上のサーバのprovisioningが実行できるvagrant-niftycloudを作った | Oreradio.memo](http://www.oreradio.com/2013/07/22.php)
* [Chef のレシピから serverspec のテストを自動生成する chef-serverspec-handler という gem を作ってみた - DevOps について書くブログ](http://tily.hatenablog.com/entry/2013/07/21/150404)

## 5. CloudAutomation β で自動化！

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 5.1. レシピのアップロード

(ここからは API キーが必要な関係で講師のデモになります。)

これまで作った cookbooks をニフティクラウドにアップロードします。

例：[https://some.ncss.nifty.com/cookbooks.tgz](https://some.ncss.nifty.com/cookbooks.tgz)

<div class="pull-right"><a href="#0_">目次へ</a></div>

### 5.2. JSON ファイル作成

今回のハンズオンで使った dna.json をそのまま JSON テンプレートの中に指定することができます。

    {
      "run_list": [
          "recipe[nc::dependency_resolver]",
          "recipe[nc::manager_store]",
          "recipe[nc::manager]"
      ],
      "nc": {
        "resource": {
          "security_group": [
            {"name": "wordpressfw"}
          ],
          "security_group_ingress": [
            {"name": "wordpressfw", "from_port": 80, "cidr_ip": "0.0.0.0/0"},
            {"name": "wordpressfw", "from_port": 22, "cidr_ip": "0.0.0.0/0"}
          ],
          "instance": [
            {
              "instance_id"   : "wordpress",
              "image_id"      : 26,
              "instance_type" : "mini",
              "security_group": "wordpressfw",
              "key_name"      : "yoursshkey",
              "chef_solo": {
                "json_attributes": {
                  "run_list":["recipe[wordpress]"],
                  "mysql": {
                    "server_root_password"  : "password",
                    "server_debian_password": "password",
                    "server_repl_password"  : "password"
                  }
                },
                "recipe_url": "ここに cookbooks の URL を書く"
              }
            }
          ]
        }
      }
    }


<div class="pull-right"><a href="#0_">目次へ</a></div>

### 5.3. コントロールパネルから実行

あとはコントロールパネルの「CloudAutomation β」機能から上記 JSON を実行するだけです。

<hr />

以上、ご参加ありがとうございました。
