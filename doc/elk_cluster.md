### elasticsearch 集群部署

http://blog.csdn.net/napoay/article/details/52202877
http://blog.csdn.net/ljhabc1982/article/details/53994562

### 1.0 bug
http://zhwen.org/?p=970

##### 1.ERROR: bootstrap checks failed max virtual memory areas vm.max_map_count [65536] is too low, increase to at least [262144]

```bash
在elasticsearch.yml中配置bootstrap.system_call_filter为false，注意要在Memory下面:
bootstrap.memory_lock: false
bootstrap.system_call_filter: false

vi /etc/security/limits.conf
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```
##### 2.max number of threads [1024] for user [lish] likely too low, increase to at least [2048]

```bash
vi /etc/security/limits.d/90-nproc.conf
修改如下内容：
* soft nproc 1024
#修改为
* soft nproc 2048
```
##### 3.max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]

```bash
vi /etc/sysctl.conf
添加下面配置：
vm.max_map_count=655360

并执行命令：
sysctl -p
```


### 2.0 master-node
elasticsearch.yml 配置如下

```bash
#同集群名字必须一致，否则节点无法加入集群
cluster.name: my-application
node.name: node-9.46

# es启动后绑定(network.host:9200/9300)
network.host: 192.168.9.46
http.port: 9200

# 开启跨域请求. 主要给插件使用, 如elasticsearch-head
http.cors.enabled: true
http.cors.allow-origin: "*"

#配置为主节点
node.master: true
node.data: true

discovery.zen.ping.unicast.hosts: ["192.168.9.46"]

```

### 2.1 data-node
elasticsearch.yml 配置如下

```bash
[root@test2 elk_test]# cat elasticsearch/config/elasticsearch.yml
cluster.name: my-application
node.name: node-6.1

network.host: 192.168.6.1
http.port: 9200

http.cors.enabled: true
http.cors.allow-origin: "*"

#date nodes
node.master: false
node.data: true
#node.ingest: false

discovery.zen.ping.unicast.hosts: ["192.168.9.46"]
#discovery.zen.minimum_master_nodes: 1
xpack.security.enabled: false
```

### 2.2 ingest-node
elasticsearch.yml 配置如下
```bash
cluster.name: my-application
node.name: node-9.70

network.host: 192.168.9.70
http.port: 9200

http.cors.enabled: true
http.cors.allow-origin: "*"

#ingest
node.master: false
node.data: false
node.ingest: true


discovery.zen.ping.unicast.hosts: ["192.168.9.46"]
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
#discovery.zen.minimum_master_nodes: 1
xpack.security.enabled: false
```

### 2.3 用docker 方式启动:
```bash
多台机器，--network=host模式，注意配置network.host
单台机器，应当使用桥接网络，结合docker-compose管理.
docker run .... -u elasticsearch  ... 指定用户启动
whoami 测试用户是否正确
```
##### 2.3.1 docker 测试:
```bash
docker run --rm -it  --network=host -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e xpack.security.enabled=false -v `pwd`/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml docker.elastic.co/elasticsearch/elasticsearch:5.5.1
```

##### 2.3.2 docker 启动:
```bash
docker run --name elk_es1 -d  --network=host -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e xpack.security.enabled=false -v `pwd`/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml docker.elastic.co/elasticsearch/elasticsearch:5.5.1
```

### 3.0 es docker 编译过程分析.
