# Webpack Series

## 🧩 一、基础与原理篇

### Webpack 的核心概念有哪些？

**标准回答：**
Webpack 的核心概念包括：
- **Entry**：入口文件，告诉 webpack 从哪里开始解析打包。  
- **Output**：告诉 webpack 打包结果输出到哪里、叫什么名字。。  
- **Loader**：用于转换非 JS 文件（如 CSS、图片、TS、Vue 等）。  
- **Plugin**：扩展 webpack 功能的插件机制，能处理打包优化、静态资源处理、环境变量注入等。
  - 这里说的"静态资源处理"包括以下一些动作：
    - **静态资源的处理和优化**：`MiniCssExtractPlugin`（提取CSS到单独文件）、`CssMinimizerWebpackPlugin`（CSS压缩，webpack5推荐）、`ImageMinimizerWebpackPlugin`（图片压缩）
    - **文件的复制、移动、重命名**：`CopyWebpackPlugin`（复制静态文件到输出目录）、`FileManagerWebpackPlugin`（文件操作管理）
    - **资源的压缩和优化**：`TerserPlugin`（JS压缩，webpack5内置）、`CompressionWebpackPlugin`（Gzip压缩）、`BundleAnalyzerPlugin`（包分析优化）
    - **资源的缓存策略**：webpack5内置持久化缓存（`cache.type: 'filesystem'`）、通过`output.filename`配置contenthash实现长期缓存
    - **清理构建目录等**：webpack5内置清理功能（`output.clean: true`）、`HtmlWebpackPlugin`（生成HTML并自动引入资源）
- **Mode**：`development`、`production`、`none` 三种模式，分别对应不同的优化策略。
- **Module**：webpack 一切皆模块，JS、CSS、图片都会被视为模块依赖。
- **Chunk**：代码块，webpack打包过程中的中间产物，由一个或多个模块组合成的代码块，最终输出为一个或多个文件。
  - **Entry Chunk**：入口文件对应的代码块
  - **Normal Chunk**：通过`import()`动态导入产生的代码块  
  - **Runtime Chunk**：webpack运行时代码单独提取的代码块

---

### Loader 和 Plugin 的区别是什么？

**标准回答：**
- **Loader**：Loader 是文件级别的转换器，比如把 TypeScript 转成 JS、把 CSS 转成 JS 模块。
  - **静态资源处理**：对于图片、字体等静态资源，webpack通过不同的loader将它们转换为JS可识别的模块：
    - `file-loader`：将文件复制到输出目录，返回文件路径（webpack5中被Asset Modules替代）
    - `url-loader`：小文件转base64内联，大文件fallback到file-loader（webpack5中被Asset Modules替代）
    - **webpack5 Asset Modules**：内置资源模块，无需额外loader
      - `asset/resource`：类似file-loader，输出文件并返回URL
      - `asset/inline`：类似url-loader，将文件转为base64 DataURL
      - `asset`：自动选择inline或resource（默认8kb阈值）
      - `asset/source`：类似raw-loader，返回文件源码  
```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        type: 'asset', // 自动选择inline或resource
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024 // 8kb阈值
          }
        }
      }
    ]
  }
};
```

- **Plugin**：Plugin 是 webpack 的生命周期钩子扩展机制，可以在构建的不同阶段执行更复杂的任务，比如压缩、打包分析、注入环境变量等。  
- loader 解决“文件转换”，Plugin 解决“构建过程控制”。

---

### Webpack 打包的基本流程？

**标准回答：**
1. **初始化**：读取配置文件和 CLI 参数进行初始化。  
2. **解析入口**：从 Entry 开始递归解析依赖，这里面会通过 Loader 处理各类资源，生成 AST，然后通过 AST 来分析模块间依赖关系，并生成 dependency graph 。
  - 基于 entry 生成 entryDependency
  - 基于 NormalModuleFactory 通过 dependency 生成对应的 Module 对象实例，并调用 build 方法
  - 根据 module 类型解析 loaders ，调用 LoaderRunner 进行模块转译
  - 模块转译后的结果，会通过**Acorn**解析器转换成AST语法树（webpack始终使用Acorn解析器，loader内部可能使用其他解析器作为中间转换层）
  - 分析AST语法树，提取出模块的依赖
  - 递归执行从 “NormalModuleFactory” 开始的流程，直到所有依赖都被解析
3. **生成 Chunk**：根据依赖关系将模块划分为不同的 Chunk，并生成 Chunk Graph。
4. **输出文件**：通过 Plugin 优化、压缩等，并输出到指定目录。

---

## ⚙️ 二、常见配置篇

### 如何处理 CSS 文件？

```js
{
  test: /\.css$/,
  use: ['style-loader', 'css-loader']
}
```

**标准回答：**
- **css-loader** 解析 @import 与 url()。
- **style-loader** 把样式注入到 <style> 标签中。
- **MiniCssExtractPlugin** 生产环境推荐使用，分离 CSS 文件，方便缓存。

---

### 如何支持 TypeScript 或 ES6？

```js
{
  test: /\.tsx?$/,
  use: 'ts-loader',
  exclude: /node_modules/
}
```

推荐的 Babel-loader 规则（webpack5）：
```js
{
  test: /\.(js|jsx|ts|tsx)$/,
  exclude: /node_modules/,
  use: {
    loader: 'babel-loader',
    options: {
      cacheDirectory: true // 提升二次构建速度
    }
  }
}
```

.babelrc（或 babel.config.js）示例：
```json
{
  "presets": [
    ["@babel/preset-env", { "useBuiltIns": "usage", "corejs": 3 }],
    "@babel/preset-typescript",
    ["@babel/preset-react", { "runtime": "automatic" }]
  ],
  "plugins": [
    ["@babel/plugin-transform-runtime", { "corejs": false }]
  ]
}
```

说明：
- `@babel/preset-env` 负责按目标环境转换语法，并按需注入 polyfill（需要 `core-js@3`）。
- `@babel/preset-typescript` 用 Babel 转 TS（类型检查交给 `tsc` 或 `fork-ts-checker-webpack-plugin`）。
- `@babel/preset-react` 可选，React 项目建议开启并使用自动 JSX 运行时。
- `@babel/plugin-transform-runtime` 避免重复注入 helper，减少包体积。

**标准回答：**
- **ts-loader** 或 **babel-loader** 用于转译 TypeScript 或 ES6 语法。
- 并通过 **tsconfig.json** 或 **.babelrc** 控制转译规则。

---

### 如何处理图片资源？

```js
{
  test: /\.(png|jpg|gif|svg)$/,
  type: 'asset',
  parser: {
    dataUrlCondition: { maxSize: 8 * 1024 } // 小于 8KB 转 base64
  }
}
```

**标准回答：**
- Webpack 5 内置了 Asset Modules，取代了旧的 file-loader、url-loader，更简洁。

---

## 🚀 三、性能优化篇

### 如何加快构建速度？

**标准回答：**

#### 1. 持久化缓存 (cache)
```javascript
// webpack 4
module.exports = {
  cache: true, // 内存缓存，webpack5中通用适用，在webpack5中对应 { cache: { type: 'memory' } }
};

// webpack 5 (推荐)
module.exports = {
  cache: {
    type: 'filesystem', // 磁盘缓存
    buildDependencies: {
      config: [__filename], // 配置文件变化时清除缓存
    },
  },
};
```
**原理说明：** 
- webpack 4：只支持内存缓存，重启后失效
- webpack 5：支持磁盘缓存，重启后仍然有效
- 可以将构建时间从30秒减少到3-5秒（提升80-90%）

#### 2. 多线程编译 (thread-loader)
```javascript
// webpack 4
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          'thread-loader',
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true, // webpack 4 需要手动启用babel缓存
            },
          },
        ],
      },
    ],
  },
};

// webpack 5 (推荐)
module.exports = {
  cache: { type: 'filesystem' }, // 全局磁盘缓存
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          'thread-loader',
          'babel-loader', // 无需 cacheDirectory，webpack5 全局缓存已覆盖
        ],
      },
    ],
  },
};
```
**性能对比：**
- 单线程编译：100个文件需要20秒
- 4线程编译：100个文件需要6-8秒（提升60-70%）
- 注意：小项目可能因为线程开销反而变慢

#### 3. 限制loader作用范围 (exclude/include)
```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        include: path.resolve(__dirname, 'src'), // 只处理src目录
        exclude: /node_modules/, // 排除node_modules
        use: 'babel-loader',
      },
    ],
  },
};
```
**为什么可以排除 node_modules？**
1. **已经编译过**：npm包通常已经编译为ES5，无需再次babel转译
2. **兼容性考虑**：大部分npm包会发布兼容版本，支持主流浏览器
3. **性能优化**：node_modules文件数量庞大，跳过可大幅提升速度

**特殊情况需要处理 node_modules：**
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        include: path.resolve(__dirname, 'src'),
        exclude: /node_modules\/(?!(some-es6-package|another-package))/,
        use: 'babel-loader',
      },
    ],
  },
};
```

**优化效果：**
- 避免对node_modules中的文件进行不必要的转译
- 大型项目可减少50-70%的处理文件数量
- 构建时间可减少30-50%

#### 4. 路径别名优化 (resolve.alias)
```javascript
// webpack.config.js
module.exports = {
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      'components': path.resolve(__dirname, 'src/components'),
      'utils': path.resolve(__dirname, 'src/utils'),
    },
    extensions: ['.js', '.jsx', '.ts', '.tsx'], // 减少扩展名查找
  },
};
```
**查找对比：**
```javascript
// 优化前：webpack需要逐层查找
import Button from '../../../components/Button/index.js'

// 优化后：直接定位到目标目录
import Button from 'components/Button'
```
**性能提升：** 减少文件系统查找时间，大项目可提升10-20%构建速度

#### 5. 热更新 (HMR)
```javascript
// webpack 4 & 5 通用配置
module.exports = {
  devServer: {
    hot: true, // 启用HMR
    liveReload: false, // 禁用整页刷新
  },
  plugins: [
    // webpack 4 需要手动添加
    new webpack.HotModuleReplacementPlugin(),
    // webpack 5 在 devServer.hot: true 时自动添加
  ],
};
```
**开发体验对比：**
- 无HMR：修改代码 → 重新编译整个应用 → 刷新页面 → 丢失状态（3-10秒）
- 有HMR：修改代码 → 只编译变更模块 → 热替换 → 保持状态（0.1-1秒）
- 开发效率提升：90%以上的时间节省

---

### 如何优化打包体积？

**标准回答：**
- 使用 splitChunks 提取公共代码。
- 使用 TerserPlugin 压缩 JS，CssMinimizerPlugin 压缩 CSS。
- 删除无用代码（Tree Shaking）。
- 按需加载（懒加载、动态 import）。
- 开启 production 模式自动启用优化。

---

### Tree Shaking 是怎么实现的？

**标准回答：**
Tree Shaking 利用 ES Module 的 静态依赖分析特性（import/export）：
- webpack 在构建时标记未使用的导出；
- 在压缩阶段由 TerserPlugin 移除无用代码；
- 要求：
    - 使用 ES Module（不能是 require）；
    - package.json 中设置 "sideEffects": false。（也可以通过配置 optimization.sideEffects 开启）

补充：sideEffects 是 package.json 中的一个字段，用于告诉 webpack 是否可以安全地删除未使用的导出。
- 如果设置为 false，webpack 会假设模块没有副作用，因此可以安全地删除未使用的导出。
- 如果设置为 true，webpack 会假设模块有副作用，因此不能删除未使用的导出。

什么是副作用？
- 副作用是指模块在被导入时，除了导出值之外，还会执行其他操作影响外部环境。
- 例如，修改全局变量、改变 DOM 结构、发送网络请求等。

## 💡 四、实战与经验篇

### 你在项目中遇到过哪些 webpack 问题？怎么解决的？

**标准回答示例：**
比如我们项目打包时间太长，我排查发现：
- Babel 转译 node_modules 太慢，于是用 exclude: /node_modules/；
- 使用了持久化缓存 cache.type = 'filesystem'；
- 用 webpack-bundle-analyzer 分析依赖，拆分 vendor；
- 最后构建时间从 50 秒降到 10 秒左右。

### 如果打包出来体积异常大，你怎么排查？

**标准回答：**
- 使用 webpack-bundle-analyzer 查看依赖体积分布；
- 检查是否打包了重复依赖（React 多版本问题）：
  - 使用 webpack-bundle-analyzer（npm install --save-dev webpack-bundle-analyzer）发现重复依赖
  - 查看依赖树找出重复版本：npm ls react
  ```bash
  # 输出可能显示：
  # ├── react@17.0.2
  # └── some-ui-lib@1.0.0
  #     └── react@16.14.0  # 发现重复！
  ```
  - 解决方案：使用 resolve.alias 强制统一版本
  ```javascript
  // webpack.config.js
  module.exports = {
    resolve: {
      alias: {
        'react': path.resolve('./node_modules/react'),
        'react-dom': path.resolve('./node_modules/react-dom')
      }
    }
  };
  ```
  - 假如某个冲突的依赖是自己维护的一个包里面引入的，那我们也可以通过peerDependencies来解决。
- 其实通过 bundle analyzer 工具，我们也可能发现一些依赖中有很多冗余代码，这些代码对应的能力我们并没有使用到，这时候我们可以通过 webpack5 内置的 ignorePlugin 来忽略这些代码。 
  - 虽然提供了webpack5有tree shaking的能力，但是两者的作用机制是不同的：
    - Tree Shaking 是通过 ES Module 静态语法分析，在编译阶段确定哪些导出（export）没有被引用（import），从而在打包结果中删除无用代码。
    - IgnorePlugin 是在模块解析阶段，直接阻止某些模块被解析或打包。也就是说，它不是“删除没用的代码”，而是让某些模块根本不被引入。
```js
new webpack.IgnorePlugin({
  resourceRegExp: /^\.\/locale$/,
  contextRegExp: /moment$/
});
```
➡️ 含义是：在打包 moment 时，忽略掉它内部 require('./locale') 这种调用。
moment 就是一个典型例子：
它内部会 require 所有语言包，虽然你只用到中文，但 Tree Shaking 分析不出哪些语言包没用，因为这是动态的 require.context() 调用。

Tree Shaking 看不懂这个：
```js
require('./locale/' + name);
```

所以它没法删除无用 locale 文件。
但 IgnorePlugin 可以在解析阶段直接说：“凡是匹配到 moment 的 ./locale 路径，不要打包进去。”
于是打包体积瞬间从几百 KB → 几十 KB。

- 检查动态引入是否生效；
- 查看 mode 是否是 production；
- 查看是否没有设置 sideEffects: false 或使用了大文件未懒加载。

### 如果要自己写一个 webpack 插件，你会怎么做？

**标准回答：**
- 插件本质是一个有 apply(compiler) 方法的类：

```js
class MyPlugin {
  apply(compiler) {
    compiler.hooks.emit.tap('MyPlugin', (compilation) ={
      console.log('自定义插件执行');
    });
  }
}
module.exports = MyPlugin;
```
- 所以初始化插件其实是很简单的，难的是插件的功能实现。可以从以下几块来逐步实现：
  - 确认自己想要的能力
  - 查看 webpack 钩子文档，确认自己需要介入的阶段
  - 实现自己的功能逻辑
  
利用 webpack 的 Tapable 钩子机制 可以在生命周期任意阶段介入逻辑。

### 🎯 **真实插件案例：自动生成版本信息插件**

```javascript
// version-info-plugin.js
class VersionInfoPlugin {
  constructor(options = {}) {
    this.options = {
      filename: 'version.json',
      includeHash: true,
      ...options
    };
  }

  apply(compiler) {
    // 1. 确认能力：在构建完成后生成版本信息文件
    
    // 2. 选择钩子：emit 阶段（文件输出前）
    compiler.hooks.emit.tapAsync('VersionInfoPlugin', (compilation, callback) => {
      
      // 3. 实现功能逻辑
      const versionInfo = {
        buildTime: new Date().toISOString(),
        version: process.env.npm_package_version || '1.0.0',
        gitCommit: process.env.GIT_COMMIT || 'unknown',
        ...(this.options.includeHash && {
          buildHash: compilation.hash
        })
      };

      const content = JSON.stringify(versionInfo, null, 2);
      
      // 添加到输出资源中
      compilation.assets[this.options.filename] = {
        source: () => content,
        size: () => content.length
      };

      callback();
    });
  }
}

module.exports = VersionInfoPlugin;
```

**使用方式：**
```javascript
// webpack.config.js
const VersionInfoPlugin = require('./version-info-plugin');

module.exports = {
  plugins: [
    new VersionInfoPlugin({
      filename: 'build-info.json',
      includeHash: true
    })
  ]
};
```

**输出结果：**
```json
// dist/build-info.json
{
  "buildTime": "2024-01-15T10:30:00.000Z",
  "version": "1.2.3",
  "gitCommit": "abc123def",
  "buildHash": "a1b2c3d4e5f6"
}
```

**实际应用场景：**
- 🔍 **问题排查**：快速确认线上版本
- 📊 **监控告警**：版本发布追踪
- 🚀 **灰度发布**：版本控制和回滚

## 🌟 五、加分题（进阶）

### webpack 和 Vite 的区别是什么？

**详细对比（基于 webpack 5 vs Vite 5+）：**

#### 🏗️ **1. 构建理念差异**
| 对比维度 | webpack 5 | Vite 5+ |
|---------|-----------|---------|
| **开发模式** | 预打包模式：启动时分析依赖图并打包 | 按需编译：启动即服务，请求时编译 |
| **启动流程** | 分析 → 打包 → 启动服务器 | 启动服务器 → 按需处理请求 |
| **冷启动时间** | 大型项目：30-60秒 | 大型项目：1-3秒 |

#### ⚙️ **2. 底层技术架构**
**webpack 5：**
- **模块系统**：自定义模块加载器 + 依赖图分析
- **编译工具**：Babel/TypeScript Compiler（JavaScript 实现）
- **打包策略**：基于 Entry → Chunk → Bundle 的打包流程
- **缓存机制**：持久化缓存（Persistent Caching）

**Vite 5+：**
- **开发模式**：原生 ES Module + esbuild（Go 语言，速度快 10-100 倍）
- **生产模式**：Rollup 打包（Tree Shaking 更优秀）
- **预构建**：依赖预构建（Pre-bundling）优化 node_modules
- **缓存策略**：HTTP 缓存 + 文件系统缓存

#### 🚀 **3. 开发体验对比**
**启动速度：**
```bash
# 大型 React 项目（1000+ 组件）
webpack 5: npm run dev  # 45秒启动
Vite 5:    npm run dev  # 2秒启动
```

**热更新（HMR）速度：**
```bash
# 修改单个组件文件
webpack 5: 500ms - 2s   # 需要重新打包相关模块
Vite 5:    50ms - 200ms # 只编译当前文件
```

**内存占用：**
- **webpack 5**：开发时需要在内存中保存完整的打包结果
- **Vite 5**：按需编译，内存占用更低

#### 📦 **4. 生产构建对比**
**打包工具：**
- **webpack 5**：使用自身打包，支持 Code Splitting、Tree Shaking
- **Vite 5**：使用 Rollup，Tree Shaking 效果更好，产物更小

**构建速度：**
```bash
# 中大型项目生产构建
webpack 5: 2-5分钟
Vite 5:    1-3分钟（esbuild 预处理 + Rollup 打包）
```

**产物优化：**
- **webpack 5**：Bundle 分析成熟，插件丰富
- **Vite 5**：天然的 ES Module 输出，更好的 Tree Shaking

#### 🔧 **5. 生态系统成熟度**
**webpack 5：**
- ✅ 插件生态极其丰富（2000+ 插件）
- ✅ 社区方案成熟，问题解决方案多
- ✅ 企业级项目验证充分
- ❌ 配置复杂，学习成本高

**Vite 5：**
- ✅ 开箱即用，配置简单
- ✅ 现代化工具链，开发体验好
- ✅ 框架支持广泛（Vue、React、Svelte 等）
- ⚠️ 插件生态相对较新，复杂场景可能需要自定义

#### 🎯 **6. 适用场景建议**
**选择 webpack 5：**
- 大型企业项目，需要复杂的构建配置
- 需要特定的 loader/plugin 支持
- 团队对 webpack 生态熟悉
- 需要兼容老旧浏览器

**选择 Vite 5：**
- 新项目，追求开发体验
- 现代浏览器环境
- 中小型到中大型项目
- 团队希望减少构建配置复杂度

#### 💻 **7. 配置代码对比**
**webpack 5 配置示例：**
```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    clean: true,
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env', '@babel/preset-react', '@babel/preset-typescript']
          }
        }
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({ template: './public/index.html' }),
    new MiniCssExtractPlugin({ filename: '[name].[contenthash].css' })
  ],
  optimization: {
    splitChunks: { chunks: 'all' }
  }
};
```

**Vite 5 配置示例：**
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash', 'axios']
        }
      }
    }
  },
  server: {
    port: 3000,
    open: true
  }
});
```

#### 📝 **总结**
1. **构建理念**：针对开发环境，webpack 是"先打包再启动"，Vite 是"启动即服务，按需编译"，生产环境，vite 会使用 rollup 打包。
2. **开发体验**：Vite 冷启动快（几秒，webpack可能要几十秒），热更新快（vite是毫秒级，webpack是秒级）
3. **技术栈**：webpack 用自定义模块系统 + 可配置编译器（Babel/TypeScript/SWC），Vite 开发用原生 ES Module + esbuild，生产用 Rollup 打包
4. **适用场景**：webpack 适合大型企业项目和复杂配置，Vite 适合新项目和追求开发体验

**一句话总结：** Vite 通过现代浏览器的原生 ES Module 实现了极致的开发体验，而 webpack 通过成熟的生态和灵活配置适合复杂的企业级项目。

### 解释一下 source map 的作用与配置。

**标准回答：**
- Source Map 用于把编译后的代码映射回源代码，方便调试。

常用配置：
- devtool: 'eval-source-map'（开发速度快）
- devtool: 'source-map'（生产调试用）
- devtool: 'cheap-module-source-map' 忽略列信息，生成更快。

生产环境一般使用 hidden-source-map 上传到错误监控平台。

### splitChunks 的优化过程中，你是如何确定哪些代码需要被拆分？有没有存在拆分不当的情况？

我在做打包优化的时候，首先会通过 webpack-bundle-analyzer 分析打包结果，看看哪些模块体积比较大、被复用得多或者更新频率比较低。

一般我会重点关注两类代码：一类是第三方依赖，比如 vue、lodash、axios 这类库，它们体积大、更新少，适合单独拆成一个 vendors 包；另一类是业务内被多次复用的公共模块，比如工具函数或组件库，这部分我会放在一个 common chunk 里。

在 Webpack 5 里我通常会配置：

```js
optimization: {
  splitChunks: {
    chunks: 'all',
    cacheGroups: {
      vendors: {
        test: /[\\/]node_modules[\\/]/,
        name: 'vendors',
        priority: -10
      },
      common: {
        minChunks: 2,
        name: 'common',
        priority: -20
      }
    }
  }
}
```


这样可以把公共依赖拆出来，提高缓存命中率，减小首屏包体积。

当然也遇到过拆分不当的情况，比如一开始拆得太细，导致网络请求数暴增、加载反而变慢。后来我通过调高 minSize，并适当合并一些 chunk，让请求数量和加载速度达到平衡。

另外 Webpack 5 的 deterministic chunkIds 和缓存机制也很有帮助，它能保证 chunk 的哈希值稳定，让缓存更容易命中。

### 有没有尝试过一些插件或者 loader ？

有的。我在优化构建性能和开发体验的时候，尝试过不少 Webpack 5 的插件和 loader。

比如在性能优化方面，我用过：

CssMinimizerPlugin 压缩样式；
- CompressionWebpackPlugin 生成 gzip 文件；
- 还有 Webpack 5 自带的 TerserPlugin 处理 JS 压缩。
- 构建体验方面，我开启了 Webpack 5 的 持久化缓存：
```js
cache: {
  type: 'filesystem'
}
```
这样二次构建速度能快很多。

资源层面，我现在基本都用 Webpack 5 自带的 asset module 来处理图片和字体，这样就不再需要 url-loader 或 file-loader 了。
此外我也经常配合 babel-loader、vue-loader、postcss-loader 这一套来保证兼容性和样式自动前缀。
最后，如果要分析包体积，我会用 WebpackBundleAnalyzer，它能很直观地看到每个模块的大小和依赖关系，对后续的拆包调整特别有帮助。

### webpack dev server 底层是如何存储和提供资源的

首先要了解一个点，资源文件在开发环境都是存储在内存当中的。这里，就有一个问题，如何存储到内存中？

其实很简单，通过map等数据结构来存储和读取，模拟一个文件操作系统，比如key就是一个path，对应的值就是文件的内容。

知道了这个，就很好理解了，dev server 提供一个代理服务，浏览器通过请求来触发文件系统的读取操作，并进行渲染。

### webpack 可以在 dev server 中做到类似 vite 的按需编译吗？

答案是可以的。

可以基于 webpack5 的实验属性：

```js
experiments: {
  lazyCompilation: {
    imports: true,
    entries: false
  }
}
```

搭配 cache.type 设置 fileSystem ，整个开发环境编译耗时会有比较大的提升。


