# kylo install
* 添加用户和组，nifi kylo activemq
* 安装kylo，rpm -ivh kylo-<version>.rpm ，拷贝离线文件到setup文件夹
* Create Kylo Database and User，
  1. CREATE DATABASE kylo;
  2. Grant Kylo user access to DB，存储过程也需要赋权限！！！
  3. 由于系统时区的问题需要修改sql语句，修改liquibase.enabled to false in /opt/kylo/kylo-services/conf/application.properties，
  4. 运行kylo-service,生成默认的mysql表。
  5. 确定show variables like '%func%';show variables like 'log_bin';及bin_log关闭，或者验证开启。参考：http://www.cnblogs.com/kerrycode/p/7641835.html
  6. 运行/opt/kylo/setup/sql/generate-update-script.sh，并修改1970 +8小时。运行kylo-db-update-script.sql脚本。
  7. 脚本执行的3个问题
     [ERROR in query 237] Can't DROP 'MIN_EVENT_TIME_MILLIS'; check that column/key exists
     [ERROR in query 255] Can't DROP 'PRIMARY'; check that column/key exists
     [ERROR in query 263] Multiple primary key defined·
* /opt/kylo/setup/setup-wizard.sh -o 注意全路径
* 安装spark
* 添加spark-env.sh文件并添加文件制定目录的Hadoop配置。
* 修改/opt/kylo/kylo-service/conf/spark.properties 文件
  ·![spark.properties配置文件](http://p8pdrbs0b.bkt.clouddn.com/9bf92df941bdd6c79f91266e724a589a.jpg)
* 修改/opt/kylo/kylo-service/conf/application.properties文件。
* 遇到的问题
 ![](http://p8pdrbs0b.bkt.clouddn.com/ca3a4765587f71e8b7e122147e4d8fca.jpg)


