---
title: webpack to vite踩坑记录
date: 2022-06-10 14:09:26
tags: vite
---
# webpack to vite踩坑记录

## 为什么换vite
1. vite开发HRM非常快，毫秒级时延体验丝滑
2. 学习一下vite

## vite和webpack是对立的吗
1. 不完全，vite不是为了取代webpack的。
[vite多久后能干掉webpack？](https://www.zhihu.com/question/477139054/answer/2156019180) - 尤雨溪的回答 - 知乎

2. vite主要利用浏览器自带的ES模块化功能，用上浏览器真正支持的模块化。而之前传统浏览器没有模块化方案，为了实现模块化的效果，webpack把依赖的模板用IIFE把一个个模块打包进一个js里，用IIFE的方式实现模块代码隔离。

3. 现有场景下vite对react的支持，在插件上还有欠缺，许多好用的插件还没有官方化，用于生产环境还需要谨慎，现有大多是用vite开发库类包。现在大多数人的做法还是vite用于开发环境，webpack用于生产环境来走的。如果vite对react的支持插件丰富的话，可以考虑都用一整套。

## 踩坑记录开始

### 使用插件转化现有项目
  先说结论: **不了解**vite，想一键转换是**费力不讨好**。

  我一开始的想法，是搜有没有现成的插件支持一键转换。毕竟现有项目使用webpack的比较多，相信很多人有这方便的需求。确实找到了两个插件[wp2vite](https://github.com/tnfe/wp2vite) 和 [webpack-to-vite](https://github.com/originjs/webpack-to-vite)。

  但是这两个插件简直一言难尽。本来是一个不熟悉vite的小白想通过一键转换快速上手，后来还是被逼得研究文档自己配。

  webpack-to-vite 文档说明丰富 readme文档不全，遇上莫名报错无法解决后放弃。

  ```
  progress [████████████████████████████████████████] 99% | Transforming: resolve.alias options | 2205/2222
TypeError [ERR_INVALID_ARG_TYPE]: The "path" argument must be of type string. Received undefined
    at validateString (internal/validators.js:124:11)
    at Object.resolve (path.js:980:7)
    at relativePathFormat (/Users/minhuazhang/.config/yarn/global/node_modules/@originjs/webpack-to-vite/dist/src/utils/file.js:49:71)
    at WebpackTransformer.transformDevServer (/Users/minhuazhang/.config/yarn/global/node_modules/@originjs/webpack-to-vite/dist/src/transform/transformWebpack.js:256:60)
    at WebpackTransformer.transform (/Users/minhuazhang/.config/yarn/global/node_modules/@originjs/webpack-to-vite/dist/src/transform/transformWebpack.js:136:34)
    at processTicksAndRejections (internal/process/task_queues.js:93:5)
    at async geneViteConfig (/Users/minhuazhang/.config/yarn/global/node_modules/@originjs/webpack-to-vite/dist/src/generate/geneViteConfig.js:13:24)
    at async start (/Users/minhuazhang/.config/yarn/global/node_modules/@originjs/webpack-to-vite/dist/src/cli/cli.js:82:9) {
  code: 'ERR_INVALID_ARG_TYPE'
}
Conversion failed.
```

  wp2vite，采用这个插件后，转化的时候报错。
  ```
  [wp2vite-env]:开始分析项目属性
  [wp2vite-common]:目前仅支持 React/Vue 项目
  ```  
  提示: 该项目不是react或vue项目。 我明明用的react为什么无法识别呢？原来是由于我采用的monorepo结构，使用Yarn workspace方案，子项目公共依赖的react被我提到了根目录，在子目录下调用这个插件，插件只是识别当前目录的package.json。没有找到react和vue就报错了。

  解决办法: 把跟项目的react依赖先copy到子项目的package.json。然后再运行
  ```
   wp2vite --config=./webpack.config.js
  ```
  就开始转化了。
  ![image.png](https://s2.loli.net/2022/06/10/T1QmjLN5SfeC2kh.png)

  转化后的只能看一个config的结构，vite dev都跑不起来 vite build也不行。

  ### 放弃插件，自己动手

  #### 最简单的先跑起来
  从官网下载react的vite demo，先让 vite能跑起来。
```
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: 
  [
    react(),
  ]
});
```
最简单的先跑起来, vite dev vite build都没有问题。
#### 区分dev prod环境
1. 第一步
```
export default defineConfig(({ command, mode }) => {
  if (mode === 'serve') {
    return {XXX}
  else {
    return {
      XXX // build 独有配置
      }
  }
}
```
2. 第二步
在script中 vite cli命令添加上mode参数
```
	"dev": "vite --mode serve",
```

#### 引入html模板

vite默认从main.ts 或者main.js构建，我们项目中通常配置了index.html作为html的模板。在vite中找类似HtmlWebpackPlugin的插件。

使用[vite-plugin-html](https://github.com/vbenjs/vite-plugin-html)插件
配置如下：
```
plugins:
      [
        react(),
        createHtmlPlugin({
          entry: '/src/main.tsx',
          template: 'index.html',
          inject: {
            data: {
              injectScript: `<script src="./src/main.tsx"></script>`,
            },
            tags: [
              {
                injectTo: 'body',
                tag: 'div',
                attrs: {
                  id: 'root',
                },
              },
            ],
          },
        }),
      ]
```

#### vite dev频繁刷新

运行vite dev的时候，发现页面会频繁刷新，发现需要配置hmr的host
```
  server: {
        hmr: {
          protocol: 'ws',
          host: '127.0.0.1'
        }
      },
```
这个解决了频繁刷新问题

#### 配置路径
  webpack之前配置了src路径，在使用import Fund from 'pages/Fund';类路径的时候，pages路径会自动在src目录下查找。

  vite中需要解决路径问题，可以使用如下配置
```
  resolve: {
        alias: {
            'api': path.resolve(__dirname, './src//api'),
            'common': path.resolve(__dirname, './src//common'),
            'doc': path.resolve(__dirname, './src//doc'),
        },
      },
```
path是引入的path模块 import * as path from 'path';

#### 固定包使用CDN引入
  react、react-dom一般使用cdn引入可以减少打包体积，vite中也是一样的。

  webpack中我们通常是这样配置的
  ```
  externals: {
  'react': 'React',
  'react-dom': 'ReactDom',
},
  ```
  然后在index.html通过script标签插入cdn js。

 vite引入CDN有两种方式。
 第一种
 