---
title: Python项目打包Docker镜像流程
date: 2023-12-20 15:28:57
tags: Docker
categories:
    - Docker
---
## 项目新建Dockerfile文件

> 把之前写的一个项目后端打包成镜像

```bash
cd 项目目录
touch Dockerfile
```

## 编写Dockerfile

用python3.10.13作为项目基础镜像。项目开发的时候基于python3.10.13

```bash
FROM python:3.10.13

WORKDIR /app

ENV ENV="prod"
ENV TZ=Asia/Shanghai

COPY . /app

RUN pip install poetry
RUN poetry install

CMD ["poetry", "run", "uvicorn", "backend.main:app", "--log-level", "error", "--host", "0.0.0.0", "--port", "8002"]
```



## 打包镜像

```bash
docker build -t app:0.0.1 .
```

```bash
 [+] Building 109.2s (10/10) FINISHED
 => [internal] load build definition from Dockerfile                                                0.0s
 => => transferring dockerfile: 287B                                                                0.0s
 => [internal] load .dockerignore                                                                   0.0s
 => => transferring context: 2B                                                                     0.0s
 => [internal] load metadata for docker.io/library/python:3.10.13                                  17.2s
 => [1/5] FROM docker.io/library/python:3.10.13@sha256:ba7e6f1feea05621dec8a6525e1bb9edf30a3897bc6  0.0s
 => [internal] load build context                                                                   1.4s
 => => transferring context: 2.13MB                                                                 1.3s
 => CACHED [2/5] WORKDIR /app                                                                       0.0s
 => [3/5] COPY . /app                                                                               2.4s
 => [4/5] RUN pip install poetry                                                                   25.7s
 => [5/5] RUN poetry install                                                                       58.7s
 => exporting to image                                                                              3.7s
 => => exporting layers                                                                             3.7s
 => => writing image sha256:dd19b00b3a3e6674fa022ab1d7abfcc07b166e6384880b522593feb8835d7d89        0.0s
 => => naming to docker.io/library/app:0.0.1                                                        0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
```

## 测试运行

```bash
 docker run -d -p 8002:8002 app:0.0.1
```

```bash
curl http://0.0.0.0:8002/utils/anon/ping

# {"ping":"pong"}%
```



## 打包好的镜像推到docker hub

```bash
# docker login
# 将本地的镜像重新标记为远程仓库地址和镜像名称
docker tag app:0.0.1 zhangwanheng/myapp:0.0.1

docker push zhangwanheng/myapp:0.0.1

The push refers to repository [docker.io/zhangwanheng/myapp]
afe179cfbb78: Layer already exists
f579eb81c7d5: Layer already exists
411eb48f9ce3: Layer already exists
a989dbee4530: Pushed
c5b80a51412c: Layer already exists
9162bff0c68b: Layer already exists
93cb76a64c1d: Layer already exists
a0814d1f5387: Layer already exists
ac7146fb6cf5: Layer already exists
209de9f22f2f: Layer already exists
777ac9f3cbb2: Layer already exists
ae134c61b154: Layer already exists
0.0.1: digest: sha256:e858748c5d351f024261ac11df2b64f3f6f2453595c6d3876fe8b01f7ff09520 size: 2851
```

