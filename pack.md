# 前端打包工具简单介绍

## 一、Webpack 架构与插件机制

### 1. Webpack 架构核心组成

* **Entry（入口）**
指定应用的起点文件，比如 src/index.js。

* **Module（模块）**
Webpack 把项目当作模块图，模块可以是 JS、CSS、图片等。

* **Loader（加载器）**
负责模块的预处理，比如 Babel 转译、CSS 预处理，Webpack 自身只识别 JS 和 JSON，其他类型靠 Loader 转换。

* **Plugin（插件）**
插件用来扩展 Webpack 功能，比如打包优化、资源管理等。插件通过生命周期钩子影响构建过程。

* **Chunk（块）**
由模块组成的代码块，Webpack 通过代码分割生成多个 Chunk。

* **Bundle（包）**
最终生成的一个或多个文件，供浏览器加载。

### 2. Webpack 工作流程

* 从 Entry 开始，构建依赖图。

* 递归解析模块依赖。

* Loader 转换文件内容。

* Plugin 钩子运行，注入功能。

* Chunk 生成。

* Bundle 输出

### 3、Webpack 基本配置示例

```js
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js', // 入口
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'), // 输出目录
    clean: true,
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader', // 转译ES6+
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'], // 处理CSS
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html', // 模板文件
    }),
  ],
  mode: 'development',
  devtool: 'source-map', // Source Map 支持
  devServer: {
    static: './dist',
    hot: true, // HMR支持
  },
};
```

## 二、Vite 原理与内部实现（包含 esbuild 与 Rollup）

### 1. Vite 设计理念

* 开发时：利用浏览器原生 ES 模块，启动快，修改文件只热更新。

* 构建时：用 Rollup 或 Rspack 做打包，兼顾速度与功能

### 2. 开发模式

* 使用 esbuild 预构建依赖（node_modules），极大提升启动速度。

* 源码不打包，按需请求原始模块，浏览器负责解析。

* 通过 WebSocket 实现 HMR（热模块替换）

### 3. 构建模式

* 使用 Rollup 进行生产构建，保证代码优化、Tree-shaking 和代码分割。

* 未来支持用 Rspack 或 Rolldown 替代 Rollup 提升性能

### 4. esbuild 的作用

* 基于 Go 语言实现的超快编译器。

* Vite 用它做依赖预构建和某些转换。

* 速度快，但功能和插件生态比 Rollup 差

### 5. Rollup 基本配置示例

```js
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';

export default {
  input: 'src/index.js',
  output: {
    file: 'dist/bundle.js',
    format: 'esm', // 输出ES模块
    sourcemap: true,
  },
  plugins: [
    resolve(), // 处理node_modules
    commonjs(), // 处理CommonJS模块
    babel({ babelHelpers: 'bundled', exclude: 'node_modules/**' }),
  ],
};
```

#### 参数详解

* input
入口文件，告诉 Rollup 从哪里开始打包。
* output.file
打包后的文件路径。
* output.format
指定输出格式，如 'esm'（ES模块）、'cjs'（CommonJS）、'iife'（立即执行函数）等。
* output.sourcemap
是否生成 Source Map。
* plugins
插件数组，用来扩展 Rollup 的功能，类似 Webpack 的 loader + plugin。

### 6. esbuild 基本配置示例

```js
// build.js
const esbuild = require('esbuild');

esbuild.build({
  entryPoints: ['src/index.js'],
  bundle: true,
  outfile: 'dist/bundle.js',
  minify: false,
  sourcemap: true,
  target: ['es2020'],
  loader: {
    '.js': 'jsx',
    '.css': 'css',
  },
}).catch(() => process.exit(1));
```

#### 以上参数详解

* entryPoints
指定入口文件。
* bundle
是否将所有依赖打包到一个文件。
* outfile
输出文件路径。
* minify
是否压缩输出文件。
* sourcemap
是否生成调试用的 source map。
* target
输出的 JS 代码目标版本。
* loader
自定义不同后缀文件如何加载，esbuild 自带对多种格式的支持。

### 7. Vite 基本配置示例

```js
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  root: 'src', // 项目根目录
  build: {
    outDir: '../dist', // 输出目录
    sourcemap: true,
  },
  server: {
    port: 3000,
    open: true,
  },
});
```

## 三、现代构建工具性能对比（含 Rolldown、Rspack）

| 工具       | 实现语言 | 启动速度 | 构建速度 | 插件生态      | 适用场景              |
| -------- | ---- | ---- | ---- | --------- | ----------------- |
| Webpack  | JS   | 较慢   | 慢    | 丰富        | 复杂大型项目            |
| Rollup   | JS   | 中    | 较快   | 丰富        | 库打包               |
| esbuild  | Go   | 非常快  | 非常快  | 少         | 快速原型，依赖预构建        |
| Rspack   | Rust | 快    | 快    | 正在完善      | Webpack 替代        |
| Rolldown | Rust | 预期快  | 预期快  | 兼容 Rollup | Rollup 的 Rust 重写版 |

### 四、现代打包工具常用特性与概念介绍

#### 1. 代码分割 (Code Splitting)

* 作用：将打包后的代码拆分成多个小块（chunk），按需加载，提高页面首屏加载速度。
* Webpack 和 Rollup 都支持动态导入 import()，实现按需加载。
* Vite 和 esbuild 也支持基于 Rollup 的代码分割。
* Rspack 和 Rolldown 也具备类似特性

#### 2. Tree Shaking

* 作用：自动移除代码中未被使用的导出，减小打包体积。
* Rollup 是 Tree Shaking 的先驱，支持 ES6 模块静态分析。
* Webpack 也支持，前提是使用 ES6 模块语法。
* esbuild、Vite 和其他现代工具均支持 Tree Shaking

#### 3. 热模块替换（HMR）

* 作用：在开发过程中，修改代码后，自动替换页面中的模块，而不是刷新整个页面。
* Webpack 的 webpack-dev-server 提供 HMR。
* Vite 内置极速的 HMR，性能很高。
* Rspack 也支持 HMR，且兼容 Webpack 配置

#### 4. Loader 与 Plugin

* Loader（只存在于 Webpack 和 Rspack）：负责预处理文件（如转译、打包 CSS、图片等）。
* Plugin：扩展构建流程功能，所有工具都支持插件机制，但实现形式略有不同。
* Rollup、Vite、Rolldown 插件通常基于 Rollup 插件生态

#### 5. 静态资源处理

* 现代打包工具支持对图片、字体、音视频等资源进行处理：

  - 小文件可以内联为 base64。

   - 大文件自动拷贝到输出目录并生成带哈希的文件名，支持缓存。

* Webpack 通过 asset 模块类型处理。

* Vite 内置支持静态资源导入。

* esbuild 也支持静态资源加载

#### 6. 环境变量和模式

* 通过环境变量注入不同配置，如 development 和 production 模式。
* Webpack 通过 DefinePlugin 注入。
* Vite 支持 .env 文件和内置的 import.meta.env。
* Rollup 和 esbuild 也可以通过插件或脚本实现

#### 7. 多入口和多页面支持

* 适合复杂项目、多页面应用（MPA）。
* Webpack 支持多入口配置，结合 HtmlWebpackPlugin 可以输出多页面。
* Rollup 和其他工具更倾向于库打包，MPA 需额外配置。
* Vite 支持通过 build.rollupOptions.input 配置多入口

#### 8. 构建缓存与增量构建

* 通过缓存机制提升构建速度。
* Webpack 和 Rspack 支持持久化缓存。
* esbuild 本身极快，增量构建表现优异。
* Vite 利用 esbuild 和 Rollup，速度也很快

#### 9. Source Map

* 帮助调试打包后的代码，映射回源码。
* 生产环境和开发环境可配置是否生成，避免暴露源码。
* 常用类型如 source-map、eval-source-map、cheap-module-source-map 等