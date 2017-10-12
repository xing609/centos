# Centos7.0 Logstash的logstash-input-jdbc，mysql数据实时同步ElasticSearch及词库 #


>**安装jdk1.8,采用yum安装方式，非常简单**


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


>##安装 elasticsearch 2.4.6

- 参考链接：[https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-centos-7](https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-centos-7)

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




>##安装ElasticSearch Head 插件

1.首先查找elasticsearch 安装目录

    find /* -name elasticsearch

于是我看到了/usr/share/elasticsearch/bin/这个目录，plugin就在里面了

2.进入该目录执行安装命令：

    /usr/share/elasticsearch/bin/plugin install mobz/elasticsearch-head

3.安装成功后打开浏览器验证：
    
    http://192.168.0.60:9200/_plugin/head/




>##安装ElasticSearch IK分词器

- 参考版本链接：[http://https://github.com/medcl/elasticsearch-analysis-ik](http://https://github.com/medcl/elasticsearch-analysis-ik)

1.进入elasticsearch/plugins插件目录：

    cd /usr/share/elasticsearch/plugins

2.创建ik文件夹

    mkdir ik

3.下载压缩包，ik 包版本要跟elasticsearch 版本对应：

    wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v1.9.1/elasticsearch-analysis-ik-1.9.1.zip
    
4.直接压缩到ik 文件目录下：
  
    uzip elasticsearch-analysis-ik-1.9.1.zip

5.进入elasticsearch.yml文件增加下面一行，然后重启下ES
    
    index.analysis.analyzer.ik.type : 'ik'

6.打开浏览器IK测试：

    http://192.168.0.60:9200/index/_analyze?analyzer=ik&text=中华人民共和国&pretty=true



>##安装Logstash 2.2

1.我们创建和编辑Logstash新百胜库文件：

   `sudo vi /etc/yum.repos.d/logstash.repo`

2.添加以下配置：

    [logstash-2.2]
    name=logstash repository for 2.2 packages
    baseurl=http://packages.elasticsearch.org/logstash/2.2/centos
    gpgcheck=1
    gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
    enabled=1

保存并退出。

3.用这个命令安装Logstash：
    
    sudo yum -y install logstash

4.进入logstash bin目录：

    `cd /opt/logstash/bin`

5.运行：

    ./logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'

6.启动后输入：xing test ,显示结果如下表示正常：

    OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
    xing test
    Settings: Default pipeline workers: 1
    Logstash startup completed
    {
       "message" => "xing test",
      "@version" => "1",
    "@timestamp" => "2017-10-12T09:46:05.884Z",
      "host" => "shanghai.ai01"
    }

>##安装连mysql java 驱动包

1.先进入logstash 根目录：

    cd /opt/logstash

2.下载mysql-connector-java-5.1.44压缩包：

    wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.44.tar.gz

3.解压：

    tar -zxvf mysql-connector-java-5.1.44.tar.gz

4.将解压的 mysql-connector-java-5.1.44-bin.jar 放到根目录下

    mv mysql-connector-java-5.1.44-bin.jar /opt/logstash/

5.在该目录下新建etc,并创建.cnf和.sql文件，名字自取：

      mkdir -p etc
      cd etc
      vi wp.cnf
      vi wp.sql

6.配置wp.cnf文件：

    input {
    stdin {
    }
    jdbc {
    # 数据库地址  端口  数据库名
      jdbc_connection_string => "jdbc:mysql://192.168.0.60:3306/wordpress"
    # 数据库用户名
      jdbc_user => "root"
    # 数据库密码
      jdbc_password => "1"
    # mysql java驱动地址
      jdbc_driver_library => "/opt/logstash/mysql-connector-java-5.1.44-bin.jar"
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      jdbc_paging_enabled => "true"
      jdbc_page_size => "50000"
    # sql 语句文件
      statement_filepath => "/opt/logstash/etc/wp.sql"
      schedule => "* * * * *"
      type => "jdbc"
    }
    }
    output {
    elasticsearch {
    hosts => ["192.168.0.61:9200"]
    index => "hangzhou"
    document_id => "%{id}"
    }
    stdout {
    codec => json_lines
    }
    }
保存并退出

7.启动logstash:

    bin/logstash -f etc/wp.conf

8.然后你就会看到查询的数据以json的格式显示出来，在head插件里有日志记录显示，表明配置成功！
