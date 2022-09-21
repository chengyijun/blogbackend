---
title: webpack学习笔记
date: 2022-08-31 09:03:33
tags:
---

```bash
# 初始化项目 生成 package.json 文件
npm init -y

# 安装依赖
npm i webpack webpack-cli -D

# npx 会将node_modules 中的库 临时添加到环境变量
# 命令行打包
npx webpack ./src/main.js --mode=development
```

