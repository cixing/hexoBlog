---
title: webpack可视化分析包大小
date: 2021-06-13 16:36:55
tags: webpack
---
## 使用webpack在线工具分析webpack包的大小
1. 生成统计文件 stats.json
使用 webpack 编译源代码时，用户可以生成一个包含有关模块统计信息的 JSON 文件。 这些统计信息可用于分析应用程序的依赖关系图以及优化编译速度。 该文件通常是按以下方式生成的：
在 webpack 启动时带入参数：--profile --json > stats.json ，例如：
``` bash
webpack --config webpack.config.prod.js --profile --json > stats.json
```
- --profile：记录下构建过程中的耗时信息
- --json：以 JSON 的格式输出构建结果，最后只输出一个 .json 文件，这个文件（stats.json）中包括所有构建相关的信息
- \>stats.json ：是 UNIX/Linux 系统中的管道命令、含义是把 webpack --profile --json 输出的内容通过管道输出到 stats.json 文件中

把state.json 上传到 [这里](http://chrisbateman.github.io/webpack-visualizer/)可以可视化查看分析当前webpack包的配置。

转载自：https://juejin.cn/post/6844904056985485320