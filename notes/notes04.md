## 21. 
部署web站点脚本：
```
# 前端项目构建完成后，查看当前项目目录有如下输出，其中 dist/ 为产物目录
$ ls -al 
  drwxr-xr-x    5 wen  staff     160  3 25 00:32 dist
  -rw-r--r--    1 wen  staff     126  3 21 19:55 README.md
  drwxr-xr-x  769 wen  staff   24608  3 25 23:03 node_modules
  -rw-r--r--    1 wen  staff     876  3 25 00:29 package.json
  drwxr-xr-x    4 wen  staff     128  3 25 00:32 src
  ...
  
# 将产物 dist/ 覆盖同步到远程生产机的相应nginx资源服务目录，其中 --delete 表示会先删除远程目标目录原有文件
$ rsync --delete -avz dist/ root@192.168.0.1:~/app/blog/frontend/dist/    # command1

# 注意上述脚本命令`command1`需要提前先配置构建机和生产机之间的免密登录，或者使用如下方式：
$ sudo apt-get -y install sshpass
$ sshpass -p "$PROD_PASSWORD" rsync --delete -avz dist/ root@192.168.0.1:~/app/blog/frontend/dist/    # command2

# 注意若上述脚本命令`command2`出现错误`Host key verification failed. rsync error: unexplained error...`则用如下方式：
$ rsync --rsh="sshpass -p $PROD_PASSWORD ssh -o StrictHostKeyChecking=no -l root" --delete -avz dist/ root@192.168.0.1:~/app/blog/frontend/dist/
```  
  
## 22. 
交换数组中的两项位置顺序
```
function getSwitchPositionArr(list: any[], sourceIndex: number, targetIndex: number) {
  const arr = [...list];
  arr[sourceIndex] = arr.splice(targetIndex, 1, arr[sourceIndex])[0];
  return arr;
}
```
