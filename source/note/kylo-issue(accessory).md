# kylo-issue(accessory)

## 问题一：elasticsearch 9300链接不上
*	解决：在/etc/elasticsearch/elasticsearch.yml中修改配置文件
		1：
		network.host: localhost
		transport.tcp.port: 9300

		或者2：
		network.host: 0.0.0.0
	
## 问题二：org.apache.hive.service.cli.HiveSQLException: Error while compiling statement: FAILED: SemanticException Cannot find class 'org.apache.hadoop.hive.druid.DruidStorageHandler'
*	解决：
		1.在ambari界面hive下的configs标签下把Interactive Query功能打开
		2.在ambari界面hive下的summary标签下复制HiveServer2 Interactive JDBC URL内容
		3.在/usr/hdp/2.6.5.0-292/hive2/bin下进入beeline交互行
		4.!connect jdbc:hive2://xbn-hdp-2:2181,xbn-hdp-1:2181,xbn-hdp-7:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2-hive2 scott tiger
		5.在连接后的提示符下运行hive/druid关联语句
		