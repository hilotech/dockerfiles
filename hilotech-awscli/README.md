AWS-CLI
=======

CentOS 6.5にAWS-CLIを詰め合わせたDockerfileです。


使用方法
--------

* Dockerfileの先頭にある変数を適当に設定してください
    * _SMTP_RELAY     メール送信のためにリレーしてもらうSMTPサーバー
        * リレーするSMTPサーバー（Dockerホスト推奨）には、Postfixなら  
          /etc/postfix/main.cf に  
          mynetworks = 172.0.0.0/8, 127.0.0.0  
          smtpd_recipient_restrictions = permit_mynetworks, reject  
          を追加してください
* Dockerfileを置いたディレクトリで  
  # docker build --no-cache --rm -t hilotech/awscli ./  
  などとしてイメージをビルド
* 起動は  
  # docker run -d hilotech/awscli  
  など
* SSHでiamユーザーがpasswordパスワードでアクセスできるようになって
  いるので（日本語になってない…）、docker ps でターゲットのコンテナ
  IDをチェックしてから、  
  # ssh iam@`docker inspect --format="{{ .NetworkSettings.IPAddress }}" コンテナID`  
  とかするとログインできます

