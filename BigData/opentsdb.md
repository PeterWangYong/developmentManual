### 安装HBase

```bash
wget https://mirror.bit.edu.cn/apache/hbase/stable/hbase-2.2.4-bin.tar.gz
tar xf hbase-2.2.4-bin.tar.gz
echo $JAVA_HOME
cd hbase-2.2.4
./bin/start-hbase.sh 
jps (HMaster)

# 访问：http://localhost:16010/
./bin/stop-hbase.sh
```

### 访问HBase

```bash
./bin/hbase shell
hbase(main):001:0> help
hbase(main):004:0> create 'test', 'cf'
hbase(main):005:0> list 'test'
hbase(main):006:0> describe 'test'
hbase(main):007:0> put 'test', 'row1', 'cf:a', 'value1'
hbase(main):008:0> put 'test', 'row2', 'cf:b', 'value2'
hbase(main):009:0> put 'test', 'row3', 'cf:c', 'value3'
hbase(main):010:0> scan 'test'
hbase(main):011:0> get 'test', 'row1'
hbase(main):012:0> disable 'test'
hbase(main):013:0> enable 'test'
hbase(main):014:0> disable 'test'
hbase(main):015:0> drop 'test'
hbase(main):016:0> quit
```

### 安装Zookeeper

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.6.1/apache-zookeeper-3.6.1-bin.tar.gz
tar xf apache-zookeeper-3.6.1-bin.tar.gz
cd apache-zookeeper-3.6.1-bin
vim conf/zoo.cfg
	tickTime=2000
	dataDir=/var/lib/zookeeper
	clientPort=2181
bin/zkServer.sh start
telnet localhost 2181
```

### 安装OpenTSDB

```bash
wget https://github.com/OpenTSDB/opentsdb/releases/download/v2.4.0/opentsdb-2.4.0.noarch.rpm
yum install opentsdb-2.4.0.noarch.rpm -y

# HBase建表
env COMPRESSION=NONE HBASE_HOME=./hbase-2.2.4 /usr/share/opentsdb/tools/create_table.sh 

# 配置文件
/etc/opentsdb/opentsdb.conf

# tsd.http.cachedir - Path to write temporary files to
# tsd.http.staticroot - Path to the static GUI files found in ./build/staticroot
# tsd.storage.hbase.zk_quorum - If HBase and Zookeeper are not running on the same machine, specify the host and port here.

# 启动
tsdb tsd

# 访问GUI
http://127.0.0.1:4242

# 清理临时文件（tsd.http.cachedir）
tools/clean_cache.sh.
```

### 向TSD写数据

#### name schema  命名格式

- metric name  

  指标名 如 sys.cpu.user

- tag

  标签 如 host=webserver01,cpu=0 用于定位具体的time series

  every time series in opentsdb must have at least one tag

time series由 metric name 和 tag 组合定义，查询格式如下：

```bash
# ${aggregation}:${metric name}{$tag1, $tag2}
sum:sys.cpu.user{host=webserver01,cpu=42}
sum:sys.cpu.user
start=1d-ago&m=avg:sys.cpu.user{host=webserver01}
```

#### Limited Unique IDs(UIDs)  UID限制

opentsdb会为每一个metric name,tag name(tagk),tag value(tagv)分配一个ID好，默认有1600万个可分配的ID，如果用完了数据就无法写入了。

当创建一个新的TSDB时我们可以设置UID的宽度：

```bash
tsd.storage.uid.width.metric
tsd.storage.uid.width.tagk
tsd.storage.uid.width.tagv
```

#### Data Specification 数据字段

```
metric, timestamp, value, tags
```

#### 添加自动创建metric,tag配置

```bash
tsd.core.auto_create_metrics = true

# 下面两个默认为true
tsd.core.auto_create_tagks = true
tsd.core.auto_create_tagvs = true
```

#### Telnet

```bash
put sys.cpu.user 1356998400 42.5 host=webserver01 cpu=0
```

#### HTTP API

```http
POST http://192.168.4.128:4242/api/put?summary

[
	{
		"metric": "sys.cpu.user",
		"timestamp": 1591268344069,
		"value": 42.5,
		"tags": {
			"host": "webserver01",
			"cpu": "0"
		}
	}
]
```

#### tcollector 工具

### tsdb 命令行工具







