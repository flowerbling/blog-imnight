---
layout: serverless
title: ServerlessDevs的使用和更新
date: 2023-11-28 13:58:51
tags: [Serverless]
categories:
    - Serverless
---

# Serverless Devs 项目

**Serverless Devs** 是一个开源开放的 Serverless 开发者平台，致力于为开发者提供强大的工具链体系。通过该平台，开发者不仅可以一键体验多云 Serverless 产品，极速部署 Serverless 项目，还可以在 Serverless 应用全生命周期进行项目的管理，并且非常简单快速的将 Serverless Devs 与其他工具/平台进行结合，进一步提升研发、运维效能。



Serverless 升级到v3版本

- 查看当前版本 当前2.1.15

    ```bash
    s -v
    # @serverless-devs/s: 2.1.15, core: 0.1.66, s-home: /root/.s, linux-x64, node-v16.15.0
    ```

- 查看文档 了解版本更新内容

    Serverless Devs 开发者工具会不定期的进行更新升级。开发者在使用 Serverless Devs 开发者工具时，可以根据系统提醒进行进行最新版本的感知。

    当客户端感知到系统升级之后，开发者可以通过命令`npm i -g @serverless-devs/s3`进行更新操作，也可以通过 [Release](https://github.com/Serverless-Devs/Serverless-Devs/releases)信息查看升级的具体内容，以决定是否进行本次升级。



- 更新本地 **Serverless** 版本

    ```bash
     npm i -g @serverless-devs/s3 --force
     s -v
     # @serverless-devs/s3: 0.1.0, s-home: /root/.s, linux-x64, node-v16.15.0
    ```

    ![image-20231128102822159](/Users/bzy/Library/Application Support/typora-user-images/image-20231128102822159.png)



- 尝试更新 s.yml文件

    ```bash
    s cli fc3 s2tos3 --source s.yml --target s3.yml

    #####
    # [2023-11-28 10:36:53][INFO][fc3] transform /root/llmbot/s.yml  =====>  /root/llmbot/s3.yaml
    # Error Message:
    # Cannot read properties of undefined (reading 'length')
    #
    # Env:       @serverless-devs/s3: 0.1.0, linux-x64 node-v16.15.0
    # Logs:      /root/.s/logs/1128103652
    # Get Help:  DingTalk: 33947367
    # Feedback:  https://github.com/Serverless-Devs/Serverless-Devs/issues
    ```

- 查看错误信息

    Error.json

    ```json
    [
      {
        "message": "Cannot read properties of undefined (reading 'length')",
        "exitCode": 101
      }
    ]
    ```



    ```yaml

    [31mError Message:[39m
    [31mCannot read properties of undefined (reading 'length')[39m
    [2023-11-28 10:36:53][DEBUG] TypeError: Cannot read properties of undefined (reading 'length')
        at SYaml2To3._handle_fc_service (/root/.s/components/devsapp.cn/v3/fc3/dist/index.js:379:96182)
        at SYaml2To3._handle_fc (/root/.s/components/devsapp.cn/v3/fc3/dist/index.js:379:97546)
        at SYaml2To3._transform (/root/.s/components/devsapp.cn/v3/fc3/dist/index.js:379:95887)
        at SYaml2To3.<anonymous> (/root/.s/components/devsapp.cn/v3/fc3/dist/index.js:379:95802)
        at Generator.next (<anonymous>)
        at /root/.s/components/devsapp.cn/v3/fc3/dist/index.js:379:93645
        at new Promise (<anonymous>)
        at M (/root/.s/components/devsapp.cn/v3/fc3/dist/index.js:379:93392)
        at SYaml2To3.run (/root/.s/components/devsapp.cn/v3/fc3/dist/index.js:379:95534)
        at Fc.<anonymous> (/root/.s/components/devsapp.cn/v3/fc3/dist/index.js:379:41869)

    Env:       @serverless-devs/s3: 0.1.0, linux-x64 node-v16.15.0
    Logs:      [4m/root/.s/logs/1128103652[24m
    Get Help:  DingTalk: 33947367
    Feedback:  [36m[4mhttps://github.com/Serverless-Devs/Serverless-Devs/issues[24m[39m

    [2023-11-28 10:36:53][DEBUG] Run traceid: 1128103652

    ```

- 找到报错位置 s.yml 两处配置 这里的配置字段名和原先不通 执行的时候 s2tos3工具报错 暂时手动更新一下

```yaml
 vpcConfig: ${vars.service.vpcConfig}
 nasConfig: ${vars.service.nasConfig}

 这里移除 生成新的配置后 在手动添加
```



### 注意

**3.0 相对 2.0 不支持的功能如下：**

- 非 custom/custom-container runtime，不支持单实例多并发， 即 instanceConcurrency 永远为 1
- custom container 中的 No webServerMode 模式不再支持
- 异步调用配置中 StatefulInvocation 不再支持，可以通过调用函数的时候，采用 header 指定这次调用为异步有状态调用

**因此对于 2.0 转成生成的 FC 3.0 版本的 s.yaml， 尽量保证 s deploy 行为的兼容:**

1. 对于非 custom/custom-container runtime，instanceConcurrency 字段不再有效，您即使修改 instanceConcurrency 的值， 重新 `s deploy -t s3.yaml`， instanceConcurrency 也不会生效。
2. 对于 2.0 yaml 中的 custom container 中的 webServerMode 字段会忽略， 执行`s deploy -t s3.yaml` ， 但不会影响原来的 webServerMode 值
3. 对于 2.0 yaml 异步调用配置中 statefulInvocation 字段会忽略， 执行`s deploy -t s3.yaml`， 但不会影响原来的 statefulInvocation 值
4. 对于 actions 或者 plugin 中有一些自定义行为可能需要进行一些调试调整



- 手动部署生成的 s.yml 文件

    ```bash
    fc_env=staging s deploy --use-local -t s3.yml
    ```

- 调用函数 测试

