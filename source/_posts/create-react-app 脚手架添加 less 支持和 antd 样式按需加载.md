---
title: 十条有用的 GO 技术
date: 2018-02-03 17:42:23
tags: React
---
# create-react-app 脚手架添加 less 支持和 antd 样式按需加载

## 1. 创建项目

```shell
npm install -g create-react-app       /* 安装create-react-app，建议使用cnpm */

create-react-app react-test           /* 使用命令创建应用，myapp为项目名称 */

cd react-test                         /* 进入目录，然后启动 */

npm start
```

## 2.create-react-app 把 webpack 配置文件暴露出来

`create-react-app` 生成的项目文，看不到webpack相关的配置文件，需要先暴露出来，使用如下命令即可：

```Shell
npm run eject
```

## 3. 添加 babel-plugin-import

**babel-plugin-import** 是一个用于 **按需加载** **组件代码和样式** 的 **babel** 插件（原理）。

```shell
npm install babel-plugin-import --save-dev
```

## 4.添加 less 、less-loader

```Shell
npm install less less-loader --save-dev
```

## 5.修改 webpack 配置文件

修改 `webpack.config.dev.js` 和 `webpack.config-prod.js` 配置文件

- `test: /\.css$/` 改为 `/\.(css|less)$/`
- `test: /\.css$/` 的 `use` 数组配置增加 `less-loader`

```js
{
  test: /\.(css|less)$/,
  use: [
    require.resolve('style-loader'),
    {
      loader: require.resolve('css-loader'),
      options: {
        importLoaders: 1,
      },
    },
    {
      loader: require.resolve('postcss-loader'),
      options: {
        // Necessary for external CSS imports to work
        // https://github.com/facebookincubator/create-react-app/issues/2677
        ident: 'postcss',
        plugins: () => [
          require('postcss-flexbugs-fixes'),
          autoprefixer({
            browsers: [
              '>1%',
              'last 4 versions',
              'Firefox ESR',
              'not ie < 9', // React doesn't support IE8 anyway
            ],
            flexbox: 'no-2009',
          }),
        ],
      },
    },
    {
      loader: require.resolve('less-loader') // compiles Less to CSS
    }
  ],
},
```

## 6.修改 package.json 文件，添加 .babelrc 文件

package.json

```Json
"babel": {
    "presets": [
      "react-app"
    ],
    "plugins": [
      [
        "import",
        {
          "libraryName": "antd",
          "style": true
        }
      ]
    ]
  },
```

.babelrc

```json
{
   "presets": [
      "react-app"
    ],
    "plugins": [
      [
        "import",
        {
          "libraryName": "antd",
          "style": true
        }
      ]
    ]
}
```

然后就可以了，哈哈哈！