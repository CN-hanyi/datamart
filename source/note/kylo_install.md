# kylo install
* 准备HDP环境，并准备一台边缘结点作为kylo的运行主机（边缘结点的解释：1.在局域网中能访问hdp环境，2. 将该主机最为所有hdp的client结点）
* 登陆Ambari,点击hosts->Actions->add New hosts .出现如下页面。
  ![add new hosts](http://p8pdrbs0b.bkt.clouddn.com/d1c34bf68c971211afb10e0fb1c319be.jpg)
  此处根据hdp安装文档，安装Client到该边缘结点。
* Change Hive to run both Hive and HDFS as the end user.

  1. Login to Ambari.
  2. Go to Hive -→ Config.
  3. Change “Run as end user instead of Hive user” to true.
  4. Restart required applications.

* Create an Ambari user for Kylo to query the status REST API’s. kylo:kylo

  1. Login to Ambari.
  2. Got to “Manage Ambari” → Users.
  3. Click on “Create Local User”.
  4. Create a user called “kylo” and save the password for later.
  5. Go to the “Roles” screen.
  6. Assign the “kylo” user to the “Cluster User” role.


* 添加用户和组，nifi kylo activemq,在所有结点上。包括kylo和hdp.
    ```
    $ useradd -r -m -s /bin/bash nifi
    $ useradd -r -m -s /bin/bash kylo
    ```
* 安装mysql，在kylo结点上。或者使用已经存在的mysql服务。
  a. Create a MySQL admin user or use root user to grant “create schema” access from the Kylo edge node.

     This is required to install the “kylo” schema during Kylo installation.

     Example:

     ```
     GRANT ALL PRIVILEGES ON *.* TO 'root'@'KYLO_EDGE_NODE_HOSTNAME' IDENTIFIED BY 'abc123' WITH GRANT OPTION; FLUSH PRIVILEGES;
     ```
  b.Create the “kylo” MySQL user.  kylo/root:KYLO*1,存储过程也需要赋权限！！！
     ```
     CREATE USER 'kylo'@'<KYLO_EDGE_NODE>' IDENTIFIED BY 'abc123';
     grant create, select, insert, update, delete, execute ON kylo.* to kylo'@'KYLO_EDGE_NODE_HOSTNAME';
     FLUSH PRIVILEGES;
     ```

  c. 授权kylo用户对于hive mysql metadata 的访问权限。

    ```
    GRANT select ON hive.SDS TO 'kylo'@'KYLO_EDGE_NODE_HOSTNAME';
    GRANT select ON hive.TBLS TO 'kylo'@'KYLO_EDGE_NODE_HOSTNAME';
    GRANT select ON hive.DBS TO 'kylo'@'KYLO_EDGE_NODE_HOSTNAME';
    GRANT select ON hive.COLUMNS_V2 TO 'kylo'@'KYLO_EDGE_NODE_HOSTNAME';
    ``` 

* 安装kylo，rpm -ivh kylo-<version>.rpm ，拷贝离线文件到setup文件夹。


* Create Kylo Database and User，kylo:root:KYLO*1
  3. 由于系统时区的问题需要修改sql语句，修改liquibase.enabled to false in /opt/kylo/kylo-services/conf/application.properties，
  4. 运行kylo-service,生成默认的mysql表。
  5. 确定show variables like '%func%';show variables like 'log_bin';及bin_log关闭，或者验证开启。参考：http://www.cnblogs.com/kerrycode/p/7641835.html
  6. 运行/opt/kylo/setup/sql/generate-update-script.sh，并修改1970 +8小时。运行kylo-db-update-script.sql脚本。
  7. 脚本执行的3个问题
     [ERROR in query 237] Can't DROP 'MIN_EVENT_TIME_MILLIS'; check that column/key exists
     [ERROR in query 255] Can't DROP 'PRIMARY'; check that column/key exists
     [ERROR in query 263] Multiple primary key defined·
  8. 如果nifi和kylo不在同一个结点上，需要授权nifi所在结点使用kylo用户访问数据库。

* Create the “nifi” home folders in HDFS.

  ```
  [root]$ su - hdfs
  [hdfs]$ hdfs dfs -mkdir /user/nifi
  [hdfs]$ hdfs dfs -chown nifi:nifi /user/nifi
  [hdfs]$ hdfs dfs -ls /user
  ```

  
* /opt/kylo/setup/setup-wizard.sh -o 注意全路径
* 修改/opt/kylo/kylo-service/conf/spark.properties 文件
    ```
    spark.shell.master = yarn
    spark.shell.deployMode = cluster
    spark.shell.files = /usr/hdp/current/spark-client/conf/hive-site.xml,/opt/kylo/kylo-services/conf/log4j.properties,/opt/kylo/kylo-services/conf/spark.properties
    spark.shell.jars = /usr/hdp/current/spark-client/lib/datanucleus-api-jdo-3.2.6.jar,/usr/hdp/current/spark-client/lib/datanucleus-core-3.2.10.jar,/usr/hdp/current/spark-client/lib/datanucleus-rdbms-3.2.9.jar,/opt/kylo/kylo-services/lib/mariadb-java-client-1.5.7.jar
    spark.shell.sparkArgs = --driver-memory 512m --executor-memory 512m --driver-class-path /opt/kylo/kylo-services/conf:/opt/kylo/kylo-services/lib/mariadb-java-client-1.5.7.jar --driver-java-options -Dlog4j.configuration=log4j-spark.properties
    ```

* 修改/opt/kylo/kylo-service/conf/application.properties文件。
* Create Folders for NiFi standard-ingest Feed
  a. Create the dropzone directory on the NiFi edge node.
     ```
     $ mkdir -p /var/dropzone
     $ chown nifi /var/dropzone
     ```
  b. Create the HDFS root folders.
  ```
  [root]# su - hdfs
  [hdfs ~]$ hdfs dfs -mkdir /etl
  [hdfs ~]$ hdfs dfs -chown nifi:nifi /etl
  [hdfs ~]$ hdfs dfs -mkdir /model.db
  [hdfs ~]$ hdfs dfs -chown nifi:nifi /model.db
  [hdfs ~]$ hdfs dfs -mkdir /archive
  [hdfs ~]$ hdfs dfs -chown nifi:nifi /archive
  [hdfs ~]$ hdfs dfs -mkdir -p /app/warehouse
  [hdfs ~]$ hdfs dfs -chown nifi:nifi /app/warehouse
  [hdfs ~]$ hdfs dfs -ls /

  ```
* 启动nifi es AtiveMQ . service start ${}
* 启动kylo服务 kylo-service start







