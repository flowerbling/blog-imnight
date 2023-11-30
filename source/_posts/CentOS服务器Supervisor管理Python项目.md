---
title: CentOS服务器Supervisor管理Python项目
date: 2023-11-30 11:56:32
tags: [服务器, 编程]
categories:
    - Python
---

## CentOS上安装Supervisor来管理Python项目
1. 安装supervisor
```bash 安装supervisor
sudo yum install supervisor
# sudu pip install supervisor
```
2. 创建一个Supervisor配置文件来管理你的Python项目
```bash 编辑配置
sudo vi /etc/supervisord.d/app-for-us.conf

# 文件配置内容
[program:app-for-us]
command=/root/anaconda3/envs/app-for-us/bin/uvicorn main:app --host 0.0.0.0 --port 8000
directory=/root/night/app-for-us-api/backend
user=root
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/root/night/logs/app-for-us_stdout.log
stderr_logfile=/root/night/logs/app-for-us_stderr.log
```

3. 修改supervisor.conf 让配置生效
```bash
code /etc/supervisord.conf
# 修改配置
[include]
files = supervisord.d/*.ini -> files = supervisord.d/*.conf
```

4. 启动服务
```bash
sudo service supervisord start
```

5. 查看状态
```bash
supervisorctl status
# app-for-us                       RUNNING   pid 121459, uptime 0:04:06
```