---
title: jest框架接入常见问题汇总
date: 2021-06-13 16:15:09
tags: jest
---

* import/require引入模块时找不到对应文件模块，报错信息通常为：Cannot find module XXX from XXX

  * 首先确认模块是否真正存在，如果真正存在，检查引入模块路径
  * 上述步骤检查没问题，查看Jest配置文件中的modulePaths或moduleNameMapper是否配置正确

   参考示例：

  ```javascript
  // 模块别名设置，解析模块时要搜索的其他位置的绝对路径
  modulePaths: ['<rootDir>/src'],
  // 模块别名映射，用于解析时文件查找
  moduleNameMapper: {
    '^react-native$': 'react-native-web',
    '^.+\\.module\\.(css|sass|scss)$': 'identity-obj-proxy',
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  ```

* 遇到import语法解析错误，报错信息通常为：SyntaxError: Cannot use import statement outside a module

  * Jest在解析JS文件时通常依赖babel-jest解析库，这个库依赖项目的Babel配置文件
  * 需要增加polyfill来解析import语法， 以Babel 7版本为例，通常引入@babel/preset-env来处理
  * Jest 24版本放弃了对Babel 6版本的支持，如果想使用Babel 6版本，babel-jest和Jest须使用23版本
  * Babel 6版本需要根据文档对应的规则来配置polyfill

  参考示例：

  ```javascript
  module.exports = {
    return {
      presets: ['@babel/preset-env', '@babel/preset-react'],
    };
  };
  ```

* react项目中我们在实现class时，通常会使用到箭头函数来定义需要被绑定的事件，但是Jest编译时会遇到classProperties解析错误，报错信息通常为：    SyntaxError: XXX: Support for the experimental syntax 'classProperties' isn't currently enabled 

  * @babel/preset-env不提供proposal阶段新特性的转码，需要单独配置plugin
  * 需要使用@babel/plugin-proposal-class-properties来单独处理class中的箭头函数

  参考示例：

  ```javascript
  module.exports = {
    return {
      presets: ['@babel/preset-env', '@babel/preset-react'],
      plugins: ['@babel/plugin-proposal-class-properties'],
    };
  };
  ```

* 解析ES7或ES8语法报错

  * 需要配置@babel/preset-env的polyfill版本，安装core-js 3版本
  * 也可使用@babel/plugin-transform-runtime也可以解决

  参考示例：

  ```javascript
  module.exports = {
    return {
      presets: [
        [
          '@babel/preset-env', {
            corejs: 3,
            useBuiltIns: 'usage',
          },
        ],
        '@babel/preset-react'],
    };
  };
  ```

* 遇到装饰器解析报错时，通常报错信息为：SyntaxError: XXX Support for the experimental syntax 'decorators-legacy' isn't currently enabled

  * 需要配置@babel/plugin-proposal-decorators来解析装饰器语法

  参考示例：

  ```javascript
  module.exports = {
    return {
      presets: ['@babel/preset-env', '@babel/preset-react'],
      plugins: [['@babel/plugin-proposal-decorators', { legacy: true }]],
    };
  };
  ```

* 在使用import动态引入时，会遇到import动态引入报错

  * 需要使用Babel插件dynamic-import-node来处理

* Jest会自动将process.env.NODE_ENV设置为'test'，我们可以根据NODE_ENV来动态配置Babel，这样可以隔离Jest的Babel配置

  参考示例：

  ```javascript
  module.exports = (api) => {
    const isTest = api.env('test');
  
    // 根据jest设置的环境变量来返回对应配置项
    if (isTest) {
      return {
        presets: ['@babel/preset-env', '@babel/preset-react'],
        plugins: [
          '@babel/plugin-transform-runtime',
          ['@babel/plugin-proposal-decorators', { legacy: true }],
          '@babel/plugin-proposal-class-properties',
        ],
      };
    }
    return {
      presets: ['es2015'],
    };
  };
  ```

* 遇到版本编译错误，通常报错信息为：TypeError: require(...).install is not a function jest 

  * 可能是缓存问题，需要使用bin命令参数--clearCache来清除Jest编译缓存
  * 如果不是缓存问题，可能是node版本问题，Jest 26版本不支持node 8，根据需求看是否需要Jest版本降级或node升级
  * Jest 23版本可以支持node 8

* node_modules中有些第三方库需要借助Jest进行编译

  * 设置Jest配置文件中的transformIgnorePatterns，来配置需要编译的文件

  参考示例：

  ```javascript
  // 配置忽略文件的规则（@tencent/leah-ui|antd|rc-pagination/es|rc-calendar/es|rc-util/es|css-animation|rc-tooltip/es）这些目录是需要编译的
  transformIgnorePatterns: [
    'node_modules/(?!(@tencent/leah-ui|antd|rc-pagination/es|rc-calendar/es|rc-util/es|css-animation|rc-tooltip/es)/)',
  ],
  ```

* 遇到解析CSS或图片模块报错

  * 通常需要配置Jest配置文件的transform属性，来匹配对应文件编译器

  参考示例：

  ```javascript
  // 配置对应文件规则的文件编译器
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': '<rootDir>/node_modules/babel-jest',
    '^.+\\.(css|styl|less|sass|scss)$':
      '<rootDir>/node_modules/jest-css-modules-transform',
    '\\.(jpg|jpeg|png|gif|webp|svg)$': 'jest-transform-file',
  },
  ```

* VS Code 编辑器没有Jest语法提示

  * 安装@types/jest之后，VS Code会有语法提示`tnpm i -D @types/jest`

* Eslint检查Jest测试文件语法报错

  * Eslint配置文件中env设置 jest:true 取消jest语法报错

  参考示例：

  ```javascript
  env: {
    jest: true,
  },
  ```

* 接入同源测试平台，需要依赖jest-html-reporters这个解析库生成同源测试平台的解析文件
