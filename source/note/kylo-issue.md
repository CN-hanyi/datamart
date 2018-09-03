# kylo issue

## 安装完成，启动kylo 创建kylo表，初始化数据过程中报错。
 * 问题描述

     ```
        Reason: liquibase.exception.DatabaseException: (conn:36) Invalid default value for 'modified_time'
        Query is : CREATE TABLE kylo.FEED (id BINARY(16) NOT NULL, name VARCHAR(100) NOT NULL, description VARCHAR(255) NULL, FEED_TYPE VARCHAR(45) NULL, created_time timestamp DEFAULT NOW() NOT NULL, modified_time timestamp DEFAULT '1970-01-01 00:00:01' NOT NULL) [Failed SQL:
        CREATE TABLE kylo.FEED (id BINARY(16) NOT NULL, name VARCHAR(100) NOT NULL, description VARCHAR(255) NULL, FEED_TYPE VARCHAR(45) NULL, created_time timestamp DEFAULT NOW() NOT NULL, modified_time timestamp DEFAULT '1970-01-01 00:00:01' NOT NULL)]
     ```

 * . 修改liquibase.enabled to false in /opt/kylo/kylo-services/conf/application.properties后继续报错无法建表 Caused by: org.hibernate.exception.SQLGrammarException: could not extract ResultSet
  * 问题解决
   1. 由于系统时区的问题需要修改sql语句，修改liquibase.enabled to false in /opt/kylo/kylo-services/conf/application.properties，
   2. 改回true,并创建基础表后，执行/opt/kylo/setup/sql/generate-update-sql.sh,生成kylo-db-update-script.sql，并在myql中执行。
   3. 再次改回false，service-kylo restart
##  执行自定义feed
  * 问题描述

    ```
        2018-08-23 21:13:32,451 ERROR [Timer-Driven Process Thread-1] c.t.nifi.v2.init.InitializeFeed InitializeFeed[id=6fbe2407-7799-3908-0310-a611ce25f5f4] InitializeFeed[id=6fbe2407-7799-3908-0310-a611ce25f5f4] failed to process session due to com.google.common.util.concurre
        nt.UncheckedExecutionException: org.springframework.web.client.HttpClientErrorException: 401 Unauthorized; Processor Administratively Yielded for 1 sec: com.google.common.util.concurrent.UncheckedExecutionException: org.springframework.web.client.HttpClientErrorExceptio
        n: 401 Unauthorized
    ```
  * 问题解决
  kylo用户名密码错误，修改nifi中service密码为正确密码。 
## 执行spark job 报错 
   * 问题描述

    ```
       WARN Hive: Failed to access metastore. This class should not accessed in runtime.
       org.apache.hadoop.hive.ql.metadata.HiveException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
       java.lang.ClassNotFoundException: org.datanucleus.api.jdo.JDOPersistenceManagerFactory
    ```

   * 问题解决
   https://community.hortonworks.com/questions/41271/unable-to-run-spark-job-in-clutser-mode.html

   ```
       Check the hive-site.xml contents. Should be like as below for spark.
       Add hive-site.xml to the driver-classpath so that spark can read hive configuration. Make sure —files must come before you .jar file
       Add the datanucleus jars using --jars option when you submit
       Check the contents of hive-site.xml
   ```

## 执行transform表数据预览时spark-shell报错。
   * 问题描述

    ```
    Caused by: java.lang.ClassNotFoundException: org.apache.tez.dag.api.SessionNotRunning
        at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:331)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
    ```

   * 问题解决
   修改spark.properties文件，

   ```
   spark.shell.files = /usr/hdp/current/spark-client/conf/hive-site.xml,/opt/kylo/kylo-services/conf/log4j.properties,/opt/kylo/kylo-services/conf/spark.properties 注意只有一个hive-site.xml的配置。
   spark.shell.jars = /usr/hdp/current/spark-client/lib/datanucleus-api-jdo-3.2.6.jar,/usr/hdp/current/spark-client/lib/datanucleus-core-3.2.10.jar,/usr/hdp/current/spark-client/lib/datanucleus-rdbms-3.2.9.jar,/opt/kylo/kylo-services/lib/mariadb-java-client-1.5.7.jar 注意添加所有到jar。
   ```

  * 参考地址 :https://stackoverflow.com/questions/43757969/hive-on-tez-doesnt-work-in-spark-2

