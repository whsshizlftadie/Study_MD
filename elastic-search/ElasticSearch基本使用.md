# ElasticSearch基本使用



## 1-准备以及安装

### 1.1 安装docker

```bash
#设置系统内核参数，否则会因为内存不足无法启动
# 查看内核max_map_count参数值，默认为65530
cat /proc/sys/vm/max_map_count
 
# 重新设置max_map_count的值
sysctl -w vm.max_map_count=262144
# 立即生效
sysctl -p
```

### 1.2 修改配置文件

```bash
mkdir -p /mydata/elasticsearch/config
mkdir -p /mydata/elasticsearch/data/
mkdir -p /mydata/elasticsearch/plugins
echo "http.host: 0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml
chmod -R 777 /mydata/elasticsearch/
```

### 1.3 运行

```bash
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 
-e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" 
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml 
-v /mydata/elasticsearch/data/:/usr/share/elasticsearch/data 
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins 
-d elasticsearch:7.7.0

# 复制下面这段可直接运行，由于格式原因复制上面的可能运行会被断开导致命令不完整
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /mydata/elasticsearch/data/:/usr/share/elasticsearch/data -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.7.0

#参数说明
--name表示镜像启动后的容器名称  

-d: 后台运行容器，并返回容器ID；

-e: 指定容器内的环境变量

-p: 指定端口映射，格式为：主机(宿主)端口:容器端口

discovery.type=single-node：单机运行

如果启动不了，可以加大内存设置：-e ES_JAVA_OPTS="-Xms512m -Xmx512m"
```

### 1.4 处理跨域

```bash
#进入容器
docker exec -it elasticsearch(或者容器id) /bin/bash

#找到配置文件
vim config/elasticsearch.yml

#把这两行写进配置文件中(注意是yml配置文件)：
http.cors.enabled: true 
http.cors.allow-origin: "*"

#配置修改完后需执行命令exit退出容器，接着执行docker restart 容器ID重启容器即可。
```

### 1.5 安装IK分词器

下载ik分词器压缩包：[ik分词器:7.7.0](https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.7.0/elasticsearch-analysis-ik-7.7.0.zip)

```bash
--第一种方法
#将压缩包移动到容器中
docker cp /tmp/elasticsearch-analysis-ik-7.7.0.zip elasticsearch:/usr/share/elasticsearch/plugins
#如果有xftp工具直接下载后，连接并移到远程服务器上

#进入容器
docker exec -it elasticsearch /bin/bash  

#创建目录
mkdir /usr/share/elasticsearch/plugins/ik

#将文件压缩包移动到ik中
mv /usr/share/elasticsearch/plugins/elasticsearch-analysis-ik-7.7.0.zip /usr/share/elasticsearch/plugins/ik

#进入目录
cd /usr/share/elasticsearch/plugins/ik

#解压
unzip elasticsearch-analysis-ik-7.7.0.zip

#删除压缩包
rm -rf elasticsearch-analysis-ik-7.7.0.zip

--第二种远程下载（不稳定）
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.7.0/elasticsearch-analysis-ik-7.7.0.zip

```

使用第二种方式

执行cd /usr/share/elasticsearch/plugins来到插件目录创建一个IK目录。

将压缩包移动到IK目录中，执行解压指令elasticsearch-analysis-ik-7.7.0.zip

接着删除压缩包即可，此时你可以看到一个config包和几个jar包

最后退出容器，重启重启容器即可。



## 2-基本使用

#### 2.0 配置文件

```xml

```



#### 2.1 接口继承ElasticsearchRepository<pojo,key_type>

#### 2.2 使用RestTemplate

