# Contents(公开类)
* `class pyspark.SparkConf(loadDefaults=True, _jvm=None, _jconf=None)`

        Spark应用程序的配置。用于将各种Spark参数设置为键值对。
    
        大多数情况下，您将使用SparkConf()创建一个SparkConf对象，该对象还将从spark.*Java系统属性加载值。在这种情况下，直接在SparkConf对象上设置的任何参数优先于系统属性。
        对于单元测试，您还可以调用SparkConf(false)跳过加载外部设置并获得相同的配置，而不管系统属性是什么。
    
        这个类中的所有setter方法都支持链接。例如，你可以写conf.setMaster('local').setAppName('My AppName').
        请注意，一旦SparkConf对象被传递给Spark，它就被克隆，用户不能再对其进行修改。
* `class pyspark.SparkContext(master=None, appName=None, sparkHome=None, pyFiles=None, environment=None, batchSize=0, serializer=PickleSerializer(), conf=None, gateway=None, jsc=None, profiler_cls=<class 'pyspark.profiler.BasicProfiler'>)`

        Spark功能的主要入口点。SparkContext表示与Spark集群的连接，可用于在该集群上创建RDD和广播变量。

# spark初始化
## 基础配置
```python
from pyspark import SparkConf, SparkContext
from pyspark.sql import SparkSession
conf=SparkConf().setAll([
                         ('spark.yarn.queue','root.q_dm.q_dm_up.q_dm_up_online'),
                         ('spark.master','yarn'),
                         ('spark.app.name','value_level'),
                         ('spark.executor.memory','10g'),
                         ('spark.executor.cores','4'),
                         ('spark.sql.shuffle.partitions','3000'),
                         ('spark.cores.max','8'),
                         ('spark.executor.instances','300'),
                         ('spark.default.parallelism','3000')
                        ])
spark=SparkSession.builder.config(conf=conf).enableHiveSupport().getOrCreate()
sc=spark.sparkContext.addPyFile(path)   #可向每个executor添加python包
```
## submit命令
```
${SPARK_PATH}/bin/spark-submit \
 --master yarn \    #master 的地址，提交任务到哪里执行，例如 spark://host:port,  yarn,  local
 --queue ${YARN_QUEUE} \    #队列名
 --deploy-mode ${DEPLOY_MODE} \ #在本地 (client) 启动 driver 或在 cluster 上启动，默认是 client
 --class \   #应用程序的主类，仅针对 java 或 scala 应用
 --name \   #应用程序的名称
 --driver-memory 6g \   # Driver内存，默认 1G
 --driver-java-options \    #传给 driver 的额外的 Java 选项
 --driver-library-path \    #传给 driver 的额外的库路径
 --driver-class-path \  #传给 driver 的额外的类路径
 --driver-cores 4 \     #Driver 的核数，默认是1。在 yarn 或者 standalone 下使用
 --executor-memory 12g \    #每个 executor 的内存，默认是1G
 --executor-cores 15 \  #每个 executor 的核数。在yarn或者standalone下使用
 --num-executors 10 \   #启动的 executor 数量。默认为2。在 yarn 下使用
 --archives ./source/py27.zip#python_env \  #上传python环境到每个executor
 --py-files ${ModelPath}/sparkxgb.zip aaa.py\   #XXXX是你将你所有需要用到的python文件打包成一个zip文件,aaa是你的python文件的main函数所在的py文件。
 --jars ${ModelPath}/xgboost4j-spark-0.90.jar,${ModelPath}/xgboost4j-0.90.jar \    #用逗号分隔的本地 jar 包，设置后，这些 jar 将包含在 driver 和 executor 的 classpath 下
 --packages \    #包含在driver 和executor 的 classpath 中的 jar 的 maven 坐标
 --exclude-packages	\   #为了避免冲突 而指定不包含的 package
 --repositories	\   #远程 repository
 --properties-file \    #加载的配置文件，默认为 conf/spark-defaults.conf
 --conf spark.default.parallelism=150 \
 --conf spark.executor.memoryOverhead=4g \
 --conf spark.driver.memoryOverhead=2g \
 --conf spark.yarn.maxAppAttempts=3 \
 --conf spark.yarn.submit.waitAppCompletion=true \
 --conf spark.pyspark.driver.python=./source/py27/bin/python2 \ ##指定driver上python路径
 --conf spark.yarn.appMasterEnv.PYSPARK_PYTHON=./python_env/py27/bin/python2 \  #指定executor上python路径
 --conf spark.pyspark.python=./python_env/py27/bin/python2 \
 #或者先上传至hdfs
--conf spark.yarn.dist.archives=hdfs://user/huangxiaojuan/py27.zip#python_env\
 ${ModelPath}/xgboostDemo.py $input_path_train $input_path_test $output_path

```