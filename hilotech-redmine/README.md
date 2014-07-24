Redmine /w Apache
=================

CentOS 6.5上でRedmineを動作させるDockerfileです。 

* WebサーバはApache
* Redmineとの結合にはPassengerを使用
* データベースは外部のMySQLを参照
* あらかじめテーマやプラグインがいくつも放り込んである

となっています。

欠点：

* めんどくさいので適当につくったらイメージが1.4Gになった…


使用方法
--------

* Dockerfileの先頭にある変数を適当に設定してください
    * _MY_FQDN        Redmineを動作させるドメイン名（FQDN）
    * _DATABASE_HOST  MySQLが動作しているホスト
    * _DATABASE_NAME  MySQLデーターベース名（ex: redmine）
    * _DATABASE_USER  MySQLユーザー名（ex: redmine）
        * ユーザーの接続元ホストに  
          grant all privileges on redmine.* to redmine@'172.%' identified by 'パスワード';  
          としてDockerコンテナを追加しておいてください
    * _DATABASE_PASS  MySQLユーザーパスワード
    * _SMTP_RELAY     メール送信のためにリレーしてもらうSMTPサーバー
        * リレーするSMTPサーバー（Dockerホスト推奨）には、Postfixなら  
          /etc/postfix/main.cf に  
          mynetworks = 172.0.0.0/8, 127.0.0.0  
          smtpd_recipient_restrictions = permit_mynetworks, reject  
          を追加してください
* Dockerfileを置いたディレクトリで  
  # docker build --no-cache --rm -t hilotech/redmine ./  
  などとしてイメージをビルド
* 起動は  
  # docker run -p 8000:80 -d hilotech/redmine  
  など
    * サービスを80番ポートで公開しています。
      ホスト側の空きポート（上の例では8000）とペアリングして使ってください
* 起動後は、速攻で  
  http://動作ドメイン名/
  に ID:admin、パスワード:password でログインし、adminのパスワードを変更すること


参考
----

* ファイル保存領域はコンテナの外に追い出したほうがよいと思います
    * /opt/redmine/files がファイル保存領域です。  
      -v オプションでホストの特定ディレクトリにマウントしてください
* 起動デーモン類は /etc/services.sh に記述してあります


バージョン
----------

2014/07/24 現時点での主要コンポーネントのバージョンは、

* Redmine 2.5.2
* Ruby 2.1
* Apache 2.2.15
* Passenger 4.0.46
* Subversion 1.7.4

です。
