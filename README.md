# 2020-03-21_docker-mysql-8

## 概要

`mysql:8` コンテナでの永続化について  
マウントでは無く、Docker VMに持たせている方法  

## セットアップ

```bash
$ docker volume create --name=vertification-db-data
vertification-db-data

$ docker-compose up -d
Creating network "20200321dockermysql8_default" with the default driver
Creating vertification-db

$ docker-compose ps
      Name                   Command             State                 Ports
------------------------------------------------------------------------------------------
vertification-db   docker-entrypoint.sh mysqld   Up      0.0.0.0:3306->3306/tcp, 33060/tcp

$ docker exec -it vertification-db cat hoge.csv
"id","title"
"1","hoge"
"2","fuga

```

## 調査ログ

`docker-compose.yml` での記述

```yml
...
    volumes:
      - db-data:/var/lib/mysql
...
volumes:
  db-data:
    external:
      name: vertification-db-data
```

`volume` を作成

```
$ docker volume ls | grep vertification-db-data
local               vertification-db-data
```

`volume` を確認

```
$ docker inspect vertification-db-data
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/vertification-db-data/_data",
        "Name": "vertification-db-data",
        "Options": {},
        "Scope": "local"
    }
]
```

`Mountpoint` はホストには無い  
どこにあるのか  

```
$ ls -al /var/lib/docker/volumes/vertification-db-data/_data
ls: /var/lib/docker/volumes/vertification-db-data/_data: No such file or directory
```

DockerのVM側にある  
`screen` で `tty` を指定して入る  
入ったら `[Ctl] + [D]`

```
$ screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty 
```

`Mountpoint` で指定されたディレクトリにMySQLのデータがある

```
# ls /var/lib/docker/volumes/vertification-db-data/_data
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
