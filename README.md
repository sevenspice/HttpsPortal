# 概要

* [https-portal](https://github.com/SteveLTN/https-portal)をカスタマイズしたコンテナを生成する。
    * 日本時間でログを記録するようパッケージを追加している。

# 必要条件

| OS      | バージョン |
| :------ | :-------- |
| Ubuntu  | `16.04`   |
| Ubuntu  | `18.04`   |

| アプリケーション | バージョン               |
| :--------------- | :----------------------- |
| docker           | `>=18.05.0-ce`            |
| docker-compose   | `>=1.21.2`                |

# スタートガイド

リポジトリをクローンする。この手順ではホームディレクトリにクローンする。
```
git clone https://github.com/sevenspice/HttpsPortal.git ~/HttpsPortal
```

ディレクトリを移動する。
```
cd ~/HttpsPortal
```

`docker-compose`の設定ファイルを作成する。
```
cp docker-compose.yml.origin docker-compose.yml
```

`docker-compose.yml`を環境に合わせて適宜編集する。
```
vi docker-compose.yml
```
* 詳しくは[docker-compose.ymlの編集](#docker-compose.ymlの編集)に記載している。
* 参考 : [compose-file-v3](https://docs.docker.com/compose/compose-file/)
* 参考 : [https-portal](https://github.com/SteveLTN/https-portal) 

nginx のドメインごとの設定ファイルを作成する。
```
cp https/nginx/conf.d/localhost.conf.erb https/nginx/conf.d/証明書を発行するドメイン名.conf.erb
cp https/nginx/conf.d/localhost.ssl.conf.erb https/nginx/conf.d/証明書を発行するドメイン名.ssl.conf.erb
```
例
```
cp https/nginx/conf.d/localhost.conf.erb https/nginx/conf.d/example.com.conf.erb
cp https/nginx/conf.d/localhost.ssl.conf.erb https/nginx/conf.d/example.com.ssl.conf.erb
```

コンテナをビルドし立ち上げる。
```
docker-compose up -d --build
```
        
# 再起動手順

停止。
```
docker-compose down
```
* `-v` のオプションを付けると証明書の再発行が行われるため注意すること！

起動。
```
docker-compose up -d --build
```

# docker-compose.ymlの編集

``` yml
version: '3.3'
volumes:
  https:
    driver: local
services:
  https:
    build: './https'
    image: 'https'
    container_name: 'https'
    ports:
      - '80:80'
      - '443:443'
    logging:
      driver: 'json-file'
      options:
        max-size: '10m'
        max-file: '1'
    volumes:
      - 'https:/var/log/nginx'
      - 'https:/var/lib/https-portal'
    environment:
      - PROXY_BUFFERS="8 64m"
      - PROXY_BUFFER_SIZE="64m"
    tty: true
    restart: 'always'
    environment:
      DOMAINS: 'example.com -> http://dockerhost:8000'
      STAGE:   'staging'
      # STAGE: 'production'
```
* 公開するドメイン名と表示するコンテンツへのリダイレクト先を設定する。
* 稼働環境を指定する。本番ならば`production`を指定すること。
    - `Let's encrypt`は一定時間内に繰り返し発行すると一時的に証明書が発行不可になるため注意すること。
    - `staging`だと`Let's encrypt`から検証用の証明書が発行される。
        - 問題がなければ`production`で立ち上げなおすことで本番に切り替わる。
    - テストや開発等でコンテナの再構築や再起動を繰り返す場合は`local`にすることをお勧めする。
