---
layout: post
title: "Linux常用命令"
---

> 用户管理、磁盘、传输

# 1. Linux用户相关
```
参考: https://cloud.tencent.com/developer/article/1903675
sudo useradd -m <用户名>		#创建用户
sudo usermod -aG sudo/admin <用户名>    #修改用户sudo权限
或者修改/etc/sudoers文件修改用户sudo权限
id -u <用户名>			#查看指定用户uid
id -g <用户名>			#查看指定用户pid
echo "PID: $$"    #查看当前用户uid
echo "UID: $(id -u)"		#查看当前用户pid
sudo passwd <用户名>		#设置密码
sudo useradd -m -G users <用户名> 	#为<用户名>分配"users"用户组
userdel -r 用户名
```


# 2. 存储空间相关
```
df -h	#已占用磁盘
df -h .	#当前目录/挂载点下磁盘占用
du -h --max-depth=1 	#当前目录下每个目录的空间占用
```

# 3. 文件传输
```
# scp从远程主机拷贝文件或目录到本地
scp -r -i /home/jenkins/.ssh/id_rsa(远程私钥) -o StrictHostKeyChecking=no -P 22 shenxin@ip:/home/shenxin
# scp将文件从本地拷贝到远程
scp <file> root@10.227.193.55:/data/
# scp从容器拷贝目录到远程服务器
scp -r ./workcode root@ip:~/
# curl传输文件
curl --silent -v -u {nexus_usr}:{nexus_psw} --upload-file upload.py -k <url>/test/upload.py
curl -k -u {nexus_usr}:{nexus_psw} -X GET '<url>/service/rest/v1/search?repository=cireport&name={file_path}' | jq -r '.items[0].assets[0].checksum.md5
# 当前目录下文件查找
find . -type f -name "test.c"
# 特定目录下
find /path/to/directory -type f -name "filename.txt"
```
