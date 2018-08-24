# kylo VB 部署
## VirtualBox安装流程
* 安装VirtualBox软件
1. 安装4.x版本
2. 下载kylo镜像文件
3. 添加并设置网卡-桥接模式
4. 导入镜像并启动
5. root/kylo
6. 等待20分钟系统服务启动。

## kylo各主要文件位置。
1. nifi-log-/var/log/nifi
2. nifi-/opt/nifi/nifi-1.6.0
3. 保证这两个目录下的所有文件都属于nifi用户。

## 故障排除
1. service --status-all 查看系统服务状态。
2. 当有服务处于失败状态时，想办法启动，保证所有的服务状态为run
3. 启动相关服务的命令：service xxxx restart
4. 找运维开通mac 绑定

## 部署地址及密码
* 192.16.2.165 root/kylo
* http://192.16.2.165/kylo  dladmin/thinkbig
* http://192.16.2.165/nifi 
