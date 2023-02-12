# 期末实验

## 基本命令

```bash
# 登录
ssh root@rcx-2019211279-0001
ssh root@rcx-2019211279-0002
ssh root@rcx-2019211279-0003 
ssh root@rcx-2019211279-0004 

cd /root/lastLab/resource/
cd /root/lastLab/resource/data
cd /root/lastLab/resource/code
```

## 环境配置部分

### 一 安装Kafka

#### 1.1将本地相关资源压缩上传服务器并解压

```bash
# 本地压缩
# 上传服务器 使用winSCP
# 解压文件
tar -xvf resource.tar
```

#### 1.2 解压Kafka

```bash
# 解压kafka
tar -zxvf kafka_2.11-0.10.2.2.tgz
# 将解压得到的文件夹移到/home/modules目录下
mv kafka_2.11-0.10.2.2 /home/modules/
```

#### 1.3 编辑config/server.properties文件，修改delete.topic.enable和zookeeper.connect

#### 1.4 将kafka文件夹通过scp发送到其余结点对应目录下

```bash
scp -r /home/modules/kafka_2.11-0.10.2.2 root@rcx-2019211279-0002:/home/modules/
scp -r /home/modules/kafka_2.11-0.10.2.2 root@rcx-2019211279-0003:/home/modules/
scp -r /home/modules/kafka_2.11-0.10.2.2 root@rcx-2019211279-0004:/home/modules/

```

#### 1.5 编辑config/server.properties文件，修改broker.id分别为1、2、3、4

```bash
vim /home/modules/kafka_2.11-0.10.2.2/config/server.properties
```

#### 1.6 各节点启动zookeeper

```bash
/usr/local/zookeeper/bin/zkServer.sh start
```

#### 1.7 各节点启动kafka

```bash
/home/modules/kafka_2.11-0.10.2.2/bin/kafka-server-start.sh -daemon /home/modules/kafka_2.11-0.10.2.2/config/server.properties
```

#### 1.8 jps确认kafka启动成功

```bash
jps
```

### 二、安装Redis（单机部署）

#### 2.1 升级gcc

```bash
yum -y install centos-release-scl

yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils devtoolset-9-libatomic-devel

scl enable devtoolset-9 bash
```

#### 2.2 解压redis

```bash
tar -zxvf redis-6.0.6.tar.gz
```

#### 2.3进入解压得到的文件夹，编译，安装

```bash
cd redis-6.0.6
make
make install
```

#### 2.4 修改redis.conf，设置redis为守护进程，并允许远程连接，关闭保护模式

```bash
cd redis-6.0.6
grep -n daemon < redis.conf
grep -n protected-mode < redis.conf
grep -n requirepass < redis.conf
grep -n bind < redis.conf

vim /root/redis-6.0.6/redis.conf
vim redis.conf 
```

#### 2.5启动redis

```bash
redis-server redis-6.0.6/redis.conf
```

#### 2.6ps确认redis运行成功

```bash
ps -ef | grep redis
```

### 三、安装python3

```bash
yum  install python3
# 安装需要的库 happybase pandas redis kafka
# happybase
pip3 install happybase
# python-dev
yum search python | grep python-devel
yum install python3-devel
# pandas
python3 -m pip install --upgrade --force pip
pip3 install setuptools==33.1.1
pip3 install pandas
# redis
pip3 install redis
# kafka
pip3 install kafka
```

修改python代码

加密码
rcx@200012291918

```python
# 字符集问题
# 注明utf-8
# coding=utf-8

# 在redis_connnect（）中
grep -n redis.Con < generatorRecord.py
grep -n redis.Con < load_movie_redis.py 
grep -n redis.Con < load_train_ratings_hbase.py
vim generatorRecord.py 

pool(...password = 'rcx@200012291918'...)


grep -n redis.Con < recommend_client.py 
grep -n redis.Con < recommend_server.py
vim recommend_server.py 
vim recommend_server.py 

```

## 运行

### 1 启动HDFS

```bash
start-all.sh
```

### 2 启动zookeeper(所有节点)

```bash
zkServer.sh start

/usr/local/zookeeper/bin/zkServer.sh start
```

### 3 启动HBase

```bash
start-hbase.sh

/usr/local/hbase/bin/start-hbase.sh
```

### 4配置HBase Thrift连接，以便python中的happybase库能够连接Hbase

```bash
/usr/local/hbase/bin/hbase-daemon.sh start thrift

```

### 5 在HBase中创建对应的表

需要进入到hbase.bash

```bash
hbase shell

create 'movie_records','details'

scan 'movie_records'
```

### 6.启动load_train_ratings_hbase.py（需要运行完）

```bash
python3 load_train_ratings_hbase.py rcx-2019211279-0001 9090 "movie_records" "../../data/json_train_ratings.json"

```

### 7 启动redis

```bash
redis-server redis-6.0.6/redis.conf

redis-cli -h 127.0.0.1 -p 6379
auth rcx@200012291918
# 查看redis中的key
keys *
# 查看redis中的表
lrange * 0 -1
```

### 8 启动load_movie_redis.py（需要运行完）

```bash
python3 load_movie_redis.py rcx-2019211279-0001 6379 "../../data/movies.csv"
```

### 10 启动Kafka (所有节点) 并创建 Kafka Topic

```bash
/home/modules/kafka_2.11-0.10.2.2/bin/kafka-server-start.sh /home/modules/kafka_2.11-0.10.2.2/config/server.properties

-daemon
/home/modules/kafka_2.11-0.10.2.2/bin/kafka-server-start.sh -daemon /home/modules/kafka_2.11-0.10.2.2/config/server.properties

# 创建topic
/home/modules/kafka_2.11-0.10.2.2/bin/kafka-topics.sh --zookeeper rcx-2019211279-0001:2181 --create --topic movie_rating_records --partitions 1 --replication-factor 1

```

```bash
#删除kafka topic
/home/modules/kafka_2.11-0.10.2.2/bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic movie_rating_records

#查看kafka topic
/home/modules/kafka_2.11-0.10.2.2/bin/kafka-topics.sh --list --zookeeper localhost:2181

#查看kafka topic消息数量
/home/modules/kafka_2.11-0.10.2.2/bin/kafka-run-class.sh  kafka.tools.GetOffsetShell --broker-list rcx-2019211279-0001:9092 --topic movie_rating_records --time -1

# 关闭kafka
cd /home/modules/kafka_2.11-0.10.2.2/bin
. kafka-server-stop.sh
```

### 10 启动 generatorRecord.py

```bash
python3 generatorRecord.py -h rcx-2019211279-0001:9092  -f "../../data/json_test_ratings.json"

```

### 11 启动hbase2spark、kafkaStreaming、recommend

```bash
#spark提交hbase2spark任务
/root/spark-2.1.1-bin-hadoop2.7/bin/spark-submit --class hbase2spark --master yarn --num-executors 3 --driver-memory 512m --executor-memory 512m --executor-cores 1 rcxtest.jar
#spark提交kafkaStreaming任务
/root/spark-2.1.1-bin-hadoop2.7/bin/spark-submit --class kafkaStreaming --master yarn --num-executors 3 --driver-memory 512m --executor-memory 512m --executor-cores 1 rcxtest.jar
#spark提交recommend任务
/root/spark-2.1.1-bin-hadoop2.7/bin/spark-submit --class recommend --master yarn --num-executors 3 --driver-memory 512m --executor-memory 512m --executor-cores 1 rcxtest.jar

```

### 12启动recommend_server.py

```bash
#启动推荐系统server（可以在本地Windows/macOS运行）
python3 recommend_server.py rcx-2019211279-0001 6379 23456
```

### 13 启动recommend_client.py

```bash
#启动推荐系统server（可以在本地Windows/macOS运行）
python3 recommend_client.py 127.0.0.1 23456
```

## 退出

```bash
# 关闭C&S
在recommend_client和recommend_server终端中直接ctrl+C
# 关闭步骤11的三个任务终端
直接ctrl+C
# 关闭generatorRecord
该终端中直接直接ctrl+C
# 关闭kafka （4台）
cd /home/modules/kafka_2.11-0.10.2.2/bin
. kafka-server-stop.sh
# 关闭redis
ps -ef | grep redis
kill -9 pid
# 关闭thrift
/usr/local/hbase/bin/hbase-daemon.sh stop thrift
# 关闭hbase
/usr/local/hbase/bin/stop-hbase.sh
# 关闭zookeeper (4台)
/usr/local/zookeeper/bin/zkServer.sh stop
# 关闭hdfs
/home/modules/hadoop-2.7.7/sbin/stop-all.sh
```
