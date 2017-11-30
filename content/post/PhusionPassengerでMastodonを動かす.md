---
title: "PhusionPassengerでMastodonを動かす"
date: 2017-11-30T22:07:56+09:00
draft: true
slug: "passenger-mstdn"
---
# PhusionPassengerでMastodonを動かす
## 手順
### 1. PhusionPassengerのインストールとNginxのビルド
※Rubyは直コンパイルを想定しています。
※centos7 fedora26で動作確認済み
※nginxインストール済みの場合はnginxを一回消して下さい

```
gem install passenger
cd /usr/local/src
curl -L -O https://nginx.org/download/nginx-1.13.3.tar.gz
tar xofz nginx-1.13.3.tar.gz
cd nginx-1.13.3
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --add-module=/usr/local/lib/ruby/gems/2.4.0/gems/passenger-enterprise-server-5.1.8/src/nginx_module
make
make test
make install
```

これでnginxが/etc/nginx以下にインストールされます。
systemdは以下のように設定すると幸せになれます

```
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=false

[Install]
WantedBy=multi-user.target
```
### 2.Mastodon側の変更
mastodon-webはそのままでも動きますがStreamingはbabelでいい感じにしないと動きません。
mastodonのpackage.jsonのscriptsセクションに

```
"build:passenger": "rimraf ./tmp/streaming && babel ./streaming/index.js --out-dir ./tmp",
```

を設定し、npmでbabelをインストールした後に`npm run build:passenger`を実行しましょう。
live/tmp/streaming/index.jsが出力されます。このままではうまく設定ファイルを読み込まないのでindex.jsの一部を変更します。

```
dotenv.config({
  path: env === 'production' ? '.env.production' : '.env'
});
```

ココを

```
dotenv.config({
  path: env === 'production' ? '/home/mastodon/live/.env.production' : '.env'
});
```

に変更しましょう。

コレでMastodon側の設定は終了です

### 3.最後の設定
最後にnginxの設定をしましょう

nignx.confのhttpディレクティブを

```
http {
  passenger_root /usr/local/lib/ruby/gems/2.4.0/gems/passenger-5.1.8;
  passenger_ruby /usr/local/bin/ruby;
  passenger_log_file /var/log/nginx/passenger.log;
  passenger_max_pool_size 4;
  passenger_max_instances_per_app 3;
}
```

のように設定します。passenger_max_pool_sizeとpassenger_max_instances_per_appはマシンのスッペクによって調整して下さい。
次にmastodon用のnginxの設定ファイルを以下のような感じにします。

```
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}

server {
  listen 80;
  listen [::]:80;
  server_name example.com;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name example.com;
  passenger_enabled on;
  passenger_app_group_name web;
  root /home/mastodon/live/public;
  ssl_protocols TLSv1.2;
  ssl_ciphers HIGH:!MEDIUM:!LOW:!aNULL:!NULL:!SHA;
  ssl_prefer_server_ciphers on;
  ssl_session_cache shared:SSL:10m;

  ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

  keepalive_timeout    300;
  sendfile             on;
  client_max_body_size 0;

  gzip on;
  gzip_disable "msie6";
  gzip_vary on;
  gzip_proxied any;
  gzip_comp_level 6;
  gzip_buffers 16 8k;
  gzip_http_version 1.1;
  gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

  add_header Strict-Transport-Security "max-age=31536000";
location /api/v1/streaming {
  root /home/mastodon/live;
  passenger_enabled on;
  passenger_app_group_name streaming;
  passenger_app_type node;
  passenger_startup_file /home/mastodon/live/streaming/index.js;
  tcp_nodelay on;
}
  error_page 500 501 502 503 504 /500.html;
}
```

保存したら`systemctl stop mastodon-{web,streaming}`をし`systemctl start nginx`してみて下さい。
何もエラーが出ない場合は`passenger-status`を実行し以下のような表示になるか確認して下さい。
※数値が違ってもApplication groupsにstreamingとwebが動いてるのがわかれば大丈夫です。

```
Version : 5.1.8
Date    : 2017-10-04 23:02:37 +0900
Instance: Itug6cq4 (nginx/1.13.3 Phusion_Passenger/5.1.8)

----------- General information -----------
Max pool size : 4
App groups    : 2
Processes     : 2
Requests in top-level queue : 0

----------- Application groups -----------
streaming:
  App root: /home/mastodon
  Requests in queue: 0
  * PID: 963     Sessions: 11      Processed: 2132    Uptime: 23h 37m 41s
    CPU: 0%      Memory  : 37M     Last used: 3s ago

web:
  App root: /home/mastodon/live
  Requests in queue: 0
  * PID: 926     Sessions: 0       Processed: 23280   Uptime: 23h 37m 45s
    CPU: 1%      Memory  : 192M    Last used: 2s ago
```

このように表示されれば完成です。よいMastodonライフを

