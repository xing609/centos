# Centos7.0 Logstash的logstash-input-jdbc，mysql数据实时同步ElasticSearch及词库 #


----------


> **安装jdk1.8,采用yum安装方式，非常简单**


1、查看yum库中jdk的版本

    [root@localhost ~]# yum search java|grep jdk

2、选择java-1.8.0安装

    [root@localhost ~]# yum install java-1.8.0-openjdk*

3、配置环境变量

    [root@localhost ~]# vi /etc/profile
这里jdk1.8.0的文件夹路径是:
    /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.144-0.b01.el7_4.x86_64

添加如下内容，
    
    #set java environment
    JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.144-0.b01.el7_4.x86_64
    JRE_HOME=$JAVA_HOME/jre
    CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
    PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
    export JAVA_HOME JRE_HOME CLASS_PATH PATH
4、环境变量生效

    [root@localhost ~]# source /etc/profile
5、查看jdk是否安装成功

    [root@localhost ~]# java -version



----------


>**安装 elasticsearch 2.4.6**

Elasticsearch可以用包管理器通过添加弹性的包库进行安装。

1.运行以下命令来导入Elasticsearch公共GPG钥匙插入转：

 `sudo rpm --import http://packages.elastic.co/GPG-KEY-elasticsearch`

2.创建Elasticsearch一个新的yum库文件。请注意，这是一个命令：

    echo '[elasticsearch-2.x]
    name=Elasticsearch repository for 2.x packages
    baseurl=http://packages.elastic.co/elasticsearch/2.x/centos
    gpgcheck=1
    gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    ' | sudo tee /etc/yum.repos.d/elasticsearch.repo

3.用这个命令安装Elasticsearch：

  `sudo yum -y install elasticsearch`

4.Elasticsearch现已安装完毕。让我们编辑配置：
    
    sudo vi /etc/elasticsearch/elasticsearch.yml
    
    修改 Cluster名字自取：`cluster.name: my-shanhai-cluster`
    
    修改 Node节点名字自取： `node.name: node-shanhai`
    
    修改 Network访问IP为本机：`network.host: 192.168.0.60`

 （先按ESC,再按`:wq`）保存并退出elasticsearch.yml。

 （设置显示行：：`set nu`）
    
5.现在启动Elasticsearch：
    
    sudo systemctl start elasticsearch

（注意：elasticsearch 启动过程有点慢，大概30s左右，耐心等待！）

6.然后运行以下命令将在系统启动后自动启动Elasticsearch：

    sudo systemctl enable elasticsearch

7.打开浏览器访问本机IP测试http://192.168.0.60:9200/

    {
    "name": "node-shanhai",
    "cluster_name": "my-shanhai-cluster",
    "cluster_uuid": "9kGKZvlqQimXxaRf3uoGCQ",
    "version": {
    "number": "2.4.6",
    "build_hash": "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp": "2017-07-18T12:17:44Z",
    "build_snapshot": false,
    "lucene_version": "5.5.4"
    },
    "tagline": "You Know, for Search"
    }




>**安装ElasticSearch Head 插件**

1.首先查找elasticsearch 安装目录

    find /* -name elasticsearch

于是我看到了/usr/share/elasticsearch/bin/这个目录，plugin就在里面了

2.进入该目录执行安装命令：

    /usr/share/elasticsearch/bin/plugin install mobz/elasticsearch-head

3.安装成功后打开浏览器验证：
    
    http://192.168.0.60:9200/_plugin/head/


>**安装Logstash 2.2**

　　
