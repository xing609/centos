# Centos7.0 Logstash的logstash-input-jdbc插件mysql数据同步ElasticSearch及词库 #

----------


> # 安装jdk1.8,采用yum安装方式，非常简单 #

----------

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

># 安装 ElasticSearch 2.4.6 #

----------





　　
