# 2020-03-21_docker-mysql-8

- [概要](#概要)
- [セットアップ](#セットアップ)
- [調査ログ](#調査ログ)
- [参考](#参考)

## 概要

`mysql:8` コンテナでの永続化について  
マウントでは無く、Docker VMに持たせている方法  

## セットアップ

```bash
$ docker volume create --name=verification-db-data
verification-db-data

$ docker-compose up -d
Creating network "20200321dockermysql8_default" with the default driver
Creating verification-db

$ docker-compose ps
      Name                   Command             State                 Ports
------------------------------------------------------------------------------------------
verification-db   docker-entrypoint.sh mysqld   Up      0.0.0.0:3306->3306/tcp, 33060/tcp

$ docker exec -it verification-db cat hoge.csv
"id","title"
"1","hoge"
"2","fuga

```

## 調査ログ

調査した内容を書く

`docker-compose.yml` での記述

```yml
...
    volumes:
      - db-data:/var/lib/mysql
...
volumes:
  db-data:
    external:
      name: verification-db-data
```

`volume` を作成

```bash
$ docker volume ls | grep verification-db-data
local               verification-db-data
```

`volume` を確認

```bash
$ docker inspect verification-db-data
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/verification-db-data/_data",
        "Name": "verification-db-data",
        "Options": {},
        "Scope": "local"
    }
]
```

`Mountpoint` はホストには無い  
どこにあるのか  

```bash
$ ls -al /var/lib/docker/volumes/verification-db-data/_data
ls: /var/lib/docker/volumes/verification-db-data/_data: No such file or directory
```

DockerのVM側にある  
`screen` で `tty` を指定して入る  
入ったら `[Ctl] + [D]`

```bash
$ screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty 
```

`Mountpoint` で指定されたディレクトリにMySQLのデータがある

```bash
# ls /var/lib/docker/volumes/verification-db-data/_data
#innodb_temp        ib_buffer_pool      public_key.pem
auto.cnf            ib_logfile0         sample_database
binlog.000001       ib_logfile1         server-cert.pem
binlog.000002       ibdata1             server-key.pem
binlog.index        ibtmp1              sys
ca-key.pem          mysql               undo_001
ca.pem              mysql.ibd           undo_002
client-cert.pem     performance_schema
client-key.pem      private_key.pem

```

抜けるときは `[Ctr] + [A]` 後、 `[Ctr] + [K]` で `y` を指定する

## 参考

調査した際に見たサイトを書く

- [Dockerのデータを永続化！Data Volume（データボリューム）の理解から始める環境構築入門 | Enjoy IT Life](https://nishinatoshiharu.com/docker-volume-tutorial/ )
- [#docker の volume と mount の基本が分からない ( docker run であそぼ ) - Qiita](https://qiita.com/YumaInaura/items/53f0593234c396ce4bad )
- [Dockerのまとめ - コンテナとボリューム編 - Qiita](https://qiita.com/kompiro/items/7474b2ca6efeeb0df80f )
- [docker-compose MySQL8.0 のDBコンテナを作成する - Qiita](https://qiita.com/ucan-lab/items/b094dbfc12ac1cbee8cb )
- [Dockerにおけるボリュームのマウント - logicoffee プログラミング勉強日記](https://logicoffee.hatenablog.com/entry/2018/06/21/123025 )
- [docker-composeでno declaration was foundというエラーに遭遇 - Qiita](https://qiita.com/luccafort/items/ff6133bb8b50c0c31069 )
