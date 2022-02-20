# Seaweedfs

-   Object storage
-   Restful
   

## 启动方案、defaultReplication

+ 000 不备份， 只有一份数据
+ 001 在相同的rackj里备份一份数据
+ 010 在相同数据中心内不同的rack间备份一份数据
+ 100 在不同的数据中心备份一份数据
+ 200 在两个不同的数据中心各复制2次
+ 110 在不同的rack备份一份数据， 在不同的数据中心备份一次

 **如果数据备份类型是 xyz形式,各自的意义**

+ x 在别的数据中心备份的份数
+ y 不相同数据中心不同的racks备份的份数
+ z 在别的服务器相同的rack的备份份数

## 单机版

```

weed master -mdir="./master" -defaultReplication="000" -ip="localhost" -port=9334

weed volume -dir=./node1 -mserver="localhost:9334" -ip="localhost" -port=8081

weed volume -dir=./node2 -mserver="localhost:9334" -ip="localhost" -port=8082

weed volume -dir=./node3 -mserver="localhost:9334" -ip="localhost" -port=8083

```

## 最佳配置

```
./weed master -mdir="./master" -defaultReplication="000" -ip="localhost" -port=9334

weed -v 2 volume -index=leveldb -max=100 -dir=./node1 -mserver="localhost:9334" -ip="localhost" -port=8081

weed -v 2 volume -index=leveldb -max=100 -dir=./node2 -mserver="localhost:9334" -ip="localhost" -port=8082

weed -v 2 volume -index=leveldb -max=100 -dir=./node3 -mserver="localhost:9334" -ip="localhost" -port=8083

weed filer -collection 'test_ider' -master="localhost:9334"

```

-max 最大的同时写入卷 Increase concurrent writes

## 基本操作命令

```
curl [http://127.0.0.1:9333/dir/assign](http://127.0.0.1:9333/dir/assign)

{"fid":"7,01ad259c54","url":"localhost:8082","publicUrl":"localhost:8082","count":1}

curl -F "file=@./girl_1.jpg" localhost:8082/138,1e6816fc90

curl -F "file=@./1.docx" localhost:8888/path/to/sources/

curl -F "filename=@README.md" "[http://localhost:8888/path/to/sources/](http://localhost:8888/path/to/sources/)"
```

### 查询 volume 所在的 ip

`curl [http://localhost:9333/dir/lookup?volumeId=3](http://localhost:9333/dir/lookup?volumeId=3)`

### 增大同时写入命令

`curl [http://localhost:9333/vol/grow?count=100](http://localhost:9333/vol/grow?count=100)`

### 确认可用的 volume

`curl "[http://localhost:9333/dir/status?pretty=y](http://localhost:9333/dir/status?pretty=y)"`

### namespace

`curl [http://master:9333/dir/assign?collection=pictures](http://master:9333/dir/assign?collection=pictures)`

`curl [http://master:9333/dir/assign?collection=documents](http://master:9333/dir/assign?collection=documents)`

存储是 集合名作为前缀

### 直接从 主服务器上传文件

``curl -F file=@./1.txt [http://localhost:9333/submit`](http://localhost:9333/submit%60)`

### 删除文件

``curl -X DELETE [http://127.0.0.1:8080/3,01637037d6`](http://127.0.0.1:8080/3,01637037d6%60)`

### 遍历文件

`./weed export -dir ./node1 -volumeId 3 `

### 重写 mime

``curl -F "file=@myImage.png;type=image/png" [http://127.0.0.1:8081/5,2730a7f18b44`](http://127.0.0.1:8081/5,2730a7f18b44%60)`

### 日志

`weed -v=2 master`

`weed -logdir=. volume`

### web 监控台

`volume [http://127.0.0.1:8081/ui/index.html](http://127.0.0.1:8081/ui/index.html)`

###  白名单控制

`weed master -whiteList="::1,127.0.0.1"`

`weed volume -whiteList="::1,127.0.0.1"`

`# "::1" is for IP v6 localhost.`

###  _large_disk

取消了单卷最大 30GB 的限制

###  迁移

直接拷贝 volum 目录到另一个目录

###  内置权限控制

[jwt.signing.key](https://github.com/chrislusf/seaweedfs/wiki/Security-Overview)

### 服务权限控制

[a](https://github.com/chrislusf/seaweedfs/wiki/Security-Configuration)

### 增大并发，扩容volume

`curl "[http://localhost:9330/vol/grow?collection=lmd&count=8](http://localhost:9330/vol/grow?collection=lmd&count=8)"`

### 临时修改压缩剩余空间的比率 70%

`curl "[http://localhost:9333/vol/vacuum?garbageThreshold=0.7](http://localhost:9333/vol/vacuum?garbageThreshold=0.7)"`

### 增大系统打开文件数，当前环境生效

`ulimit -n 10240`

### 金融搜索 seaweedfs

```

master -metrics.address=221.122.67.246:9091 -volumeSizeLimitMB 5000 -garbageThreshold 0.99

volume -max=85 -index=leveldb -mserver="master:9333" -port=8080

filer -master="master:9333" -collection "finance"

```

Seaweed wikiedia history

```

./weed server -dir=/mnt/nvme0/weed_history_of_wikipedia -ip=229.lmd.wuzhen.ai -volume.max=5 -filer -filer.port=18888 -master.port=9333 -master.volumeSizeLimitMB=20000 -metrics.address=222.lmd.wuzhen.ai:9091 -s3 -s3.config=./s3config.json -s3.port=18333 -volume.port=8000 -volume.max=0

```

Mag-direct seaweed

```
ulimit -n 10240;./bin/weed volume -max=0 -fileSizeLimitMB=4096 -mserver="222.lmd.wuzhen.ai:19333" -dir="/zfs/seaweedfs_mag_direct" -port=17071 -ip 220.lmd.wuzhen.ai

```