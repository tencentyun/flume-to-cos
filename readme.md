# flume to cos 使用指南

以下分为flume独立部署与依赖hadoop环境部署介绍：

## 独立使用flume作agent时

1. 首先将对应hadoop依赖版本的[hadoop-cos-{hadoop.version}-{cosn.version}.jar](https://github.com/tencentyun/hadoop-cos/releases)和[cos_api-bundle-5.x.x.jar](https://github.com/tencentyun/hadoop-cos/releases)以及定制的flume-hdfs-sink-1.8.0和fastjson的jar包放到flume的lib目录下，保证hadoop-cos能够正确的加载到FLUME_CLASSPATH中；

2. 将hadoop相关的依赖放到flume的lib目录下，保证flume能够正确加载到hadoop的相关依赖；

3. 由于flume sink cos所依赖的COSN文件系统为Hadoop兼容的文件系统，因此可以通过定义HdfsSink来将管道流Sink到COSN中，这里只需要修改hdfs的sink选项即可：

    ```conf
    ...

    # 在${agentName}.sinks.${sink}.hdfs.haconfigs选项中
    ${agentName}.sinks.${sink}.hdfs.haconfigs={"fs.cosn.impl":"org.apache.hadoop.fs.CosFileSystem","fs.AbstractFileSystem.cosn.impl":"org.apache.hadoop.fs.CosN","fs.cosn.userinfo.secretId":"AKIDxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx","fs.cosn.userinfo.              secretKey":"xxxxxxxxxxxxxxxxxxxx","fs.cosn.bucket.region":"ap-shanghai"}
    ${agentName}.sinks.${sink}.hdfs.path="cosn://${your bucket}/path"
    ...

    ```

4. 启动/重启flume服务即可：

  ```shell
  ./bin/flume-ng agent -c conf -f conf/flume-conf.properties -n $agentName

  ```

## 环境中使用hadoop时

假设环境中的已有Hadoop运行环境，hadoop的根目录为${HADOOP_HOME}，则按如下步骤进行配置

1.首先，将对应hadoop版本的hadoop-cos的jar包([hadoop-cos-{hadoop.version}-{cosn.version}.jar](https://github.com/tencentyun/hadoop-cos/releases)和[cos_api-bundle-5.x.x.jar](https://github.com/tencentyun/hadoop-cos/releases))放置到${HADOOP_HOME}/share/hadoop/tools/lib目录下，同时在${HADOOP_HOME}/hadoop-2.8.5/etc/hadoop/hadoop-env.sh末尾添加以下内容：

```shell
for f in $HADOOP_HOME/share/hadoop/tools/lib/*.jar; do
  if [ "$HADOOP_CLASSPATH" ]; then
    export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$f
  else
    export HADOOP_CLASSPATH=$f
  fi
done

```

2.编辑hadoop的${HADOOP_HOME}/etc/hadoop/core-site.xml，添加hadoop-cos的相关配置：

```xml
<property>
    <property>
        <name>fs.cosn.impl</name>
        <value>org.apache.hadoop.fs.CosFileSystem</value>
    </property>

    <property>
        <name>fs.AbstractFileSystem.cosn.impl</name>
        <value>org.apache.hadoop.fs.CosN</value>
    </property>

    <property>
        <name>fs.cosn.userinfo.secretId</name>
        <value>xxxxxxxxxxxxxxxxxxxxxxxxx</value>
    </property>

    <property>
        <name>fs.cosn.userinfo.secretKey</name>
        <value>xxxxxxxxxxxxxxxxxxxxxxxx</value>
    </property>

    <property>
        <name>fs.cosn.bucket.region</name>
        <value>ap-xxx</value>
    </property>

</property>

```

3.编辑${FLUME_HOME}/conf/flume-env.sh文件，将Hadoop的类路径添加到FLUME_CLASS中：

```shell
export $HADOOP_HOME/etc/hadoop:$HADOOP_HOME/share/hadoop/common:$HADOOP_HOME/share/hadoop/hdfs:$HADOOP_HOME/share/hadoop/tools/lib/*

```

4.修改flume-conf.properties中的hdfs路径为COS的路径即可：

```conf
  ${agentName}.sinks.${sink}.hdfs.path="cosn://${your bucket}/path"

```

5.启动/重启flume即可：

  ```shell
  ./bin/flume-ng agent -c conf -f conf/flume-conf.properties -n $agentName

  ```
