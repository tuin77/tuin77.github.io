---
title: 采用antfu模板 GitHub Pages搭建个人博客
date: 2024-03-26
lang: en
duration: 15min
description: 使用GitHub Pages搭建个人博客
---

[[toc]]


## 部署
本文记录使用 <a href="https://github.com/antfu/antfu.me" target="_blank">antfu开源模板</a> 搭建个人博客 
 <a href="https://tuin77.github.io/" target='_blank'>点击预览</a> 


本地环境 Mac node:v18.18.2 npm:10.2.5 pnpm:8.15.3

在GitHub平台克隆antfu开源模板或者直接fork[本项目](https://github.com/tuin77/tuin77.github.io)；修改仓库名称 用户名称+.github.io 

运行项目：克隆代码至本地运行 安装依赖pnpm install（需node版本>18;pnpm>8）；运行项目pnpm dev

项目部署：在GitHub平台 对该仓库进行设置自行打包 Settings> Pages> Build and deployment> Source 选择Github Actions（每月300min免费时长）；

部署脚本：在项目根目录 创建文件夹github/workflows/；复制文件 [部署脚本](https://github.com/tuin77/tuin77.github.io/blob/main/.github/workflows/github-pages.yml)
GitHub Actions官方[教程](https://docs.github.com/zh/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

```yml
name: 部署文档

on:
  push:
    branches:
      # 确保这是你正在使用的分支名称
      - main

permissions:
  contents: write

jobs:
  deploy-gh-pages:
    runs-on: ubuntu-latest

    # Grant GITHUB_TOKEN the permissions required to make a Pages deployment
    permissions:
      pages: write    # to deploy to Pages
      id-token: write # to verify the deployment originates from an appropriate source
    
    # Deploy to the github-pages environment
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
          # 如果你文档需要 Git 子模块，取消注释下一行
          # submodules: true

      - name: 安装 pnpm
        uses: pnpm/action-setup@v2
        with:
          run_install: true
          version: 8

      - name: 设置 Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20
          cache: pnpm

      - name: 构建文档
        env:
          NODE_OPTIONS: --max_old_space_size=8192
        run: |-
          pnpm run build

      - name: 上传构建产物
        uses: actions/upload-pages-artifact@v1
        with:
          name: github-pages
          path: dist

      
      # deploy
      - name: Deploy Page To Release
        id: deployment
        uses: actions/deploy-pages@v1
```

在posts新增md文章，本地预览确定没有问题后提交推送至远程 点击仓库Actiosns 查看打包进度 完成后 查看用户名称+.github.io预览网站。



## 可能遇到的问题 

本地安装依赖pnpm install：

> Something went wrong installing the "sharp" module

```js
npm rebuild --verbose sharp
pnpm config set sharp_binary_host=https://npm.taobao.org/mirrors/sharp
pnpm config set sharp_libvips_binary_host=https://npm.taobao.org/mirrors/sharp-libvips
pnpm install
```



 GitHub Actions部署问题
> Error: Artifact could not be deployed. Please ensure the content does not contain any hard links, symlinks and total size is less than 10GB.

> Error: No artifacts named "github-pages" were found for this workflow run. 

实际打包测试官方推荐的upload-artifact@v4会出现上述问题[issues](https://github.com/actions/deploy-pages/issues/305) 改用actions/upload-pages-artifact@v1 具体代码见[仓库](https://github.com/tuin77/tuin77.github.io)

