---
title: Github主页贪吃蛇贡献图
date: 2024-01-08 16:42:02
img: /images/snake.png
tags: [花里胡哨]
categories:
    - Github
summary: github个人主页首页是通过一个markdown文件配置的 这里可以配置一些图片可一些链接 这样的话 主页就可以搞一些花里胡哨的东西了
---

> github个人主页首页是通过一个markdown文件配置的 这里可以配置一些图片可一些链接 这样的话 主页就可以搞一些花里胡哨的东西了

### 1. 配置github workflows来定时或者在push的时候 更新贪吃蛇的svg图片
`README.md`是我们主页的markdown文件。
配置完成后的目录结构如下：
```
- .github
  - workflows
    - snake.yml
- snake
  - snake.svg
  - snake-dark.svg
- README.md
```

### 2. 配置workflow
```yml
name: Snake Contrib
permissions:
  contents: write
on:
  # 每天更新一次
  schedule:
    - cron: "0 0 * * *"

  # allows to manually run the job at any time
  workflow_dispatch:

  # 每次push main的时候更新一次
  push:
    branches:
      - main
jobs:
  generate:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
        # 生成svg图片 并且保存到dist/snake目录下
        # https://github.com/marketplace/actions/generate-snake-game-from-github-contribution-grid
      - name: generate github-contribution-grid-snake.svg
        uses: aelassas/snk/svg-only@main
        with:
          github_user_name: ${{ github.repository_owner }}
          outputs: |
            dist/snake/snake.svg
            dist/snake/snake-dark.svg?palette=github-dark

        # 生成的svg图片保存到dist目录下 把改动push到main分支
      - name: push github-contribution-grid-snake.svg to the output branch
        uses: crazy-max/ghaction-github-pages@v3.1.0
        with:
          target_branch: main
          build_dir: dist
          keep_history: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 3. 配置主页
在`README.md`中添加如下内容
```markdown
<!-- snake start -->
<!-- 这里的图片链接用cdn加速一下 具体用法 -> https://www.jsdelivr.com/?docs=gh -->
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://cdn.jsdelivr.net/gh/flowerbling/flowerbling/snake/snake.svg" />
  <source media="(prefers-color-scheme: light)" srcset="https://cdn.jsdelivr.net/gh/flowerbling/flowerbling/snake/snake.svg" />
  <img alt="github-snake" src="https://cdn.jsdelivr.net/gh/flowerbling/flowerbling/snake/snake-dark.svg" />
</picture>
<!-- snake end -->
```

### 4. 效果
![效果图](snake.png)