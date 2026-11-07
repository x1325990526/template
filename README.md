# Webpack 前端起步模板

最小化的 webpack 5 原生前端项目模板（HTML / CSS / JS，无框架）。包含开发服务器、生产构建和一键部署到 GitHub Pages 三套流程。

## 环境要求

- Node.js **≥ 22.15**（webpack-dev-server 6 的最低要求）
- npm

## 快速开始

```bash
npm install
npm run dev
```

## 命令

| 命令 | 作用 |
| --- | --- |
| `npm install` | 安装依赖 |
| `npm run dev` | 启动开发服务器，自动打开浏览器，改代码实时刷新 |
| `npm run build` | 生产构建，产物输出到 `dist/` |
| `npm run deploy` | 先构建，再把 `dist/` 推送到 origin 的 `gh-pages` 分支 |
| `npm run test` | 跑一遍全部测试 |
| `npm run test:watch` | watch 模式，改动文件后自动重跑 |

## 测试

用 Jest 跑单元测试。项目是 ESM（`package.json` 里的 `"type": "module"`），所以由 Babel 先转译再执行，转译规则写在 `babel.config.cjs`，不需要改 webpack 配置。

```bash
npm run test              # 跑一遍
npm run test:watch        # 持续监听，改完自动重跑
npm run test -- math      # 只跑文件名匹配 math 的测试文件
npm run test -- -t "adds" # 只跑测试名匹配 adds 的用例
```

约定：

- 测试文件命名 `*.test.js` 或 `*.spec.js`，放在被测模块旁边（`src/math.js` ↔ `src/math.test.js`），放在 `__tests__/` 目录里也能被识别。
- Jest 默认运行环境是 `node`。要测 DOM（`document`、`window`），先 `npm i -D jest-environment-jsdom`，再在 `package.json` 里加 `"jest": { "testEnvironment": "jsdom" }`。
- 模板默认不带测试文件。一个都没有时 `npm test` 会以非零码退出（Jest 找不到测试就报错），写了第一个测试文件才会变绿。

## 目录结构

```text
.
├── src/
│   ├── index.js          # JS 入口（webpack entry）
│   ├── template.html     # HTML 模板，HtmlWebpackPlugin 会用它生成 index.html
│   └── styles.css        # 样式，由 index.js import
├── babel.config.cjs      # Babel 配置，Jest 用它转译 ESM 源码
├── webpack.common.js     # 共享配置：entry / output / loader / plugin
├── webpack.dev.js        # 开发配置：development 模式 + devServer
├── webpack.prod.js       # 生产配置：production 模式
├── package.json          # 依赖与脚本（dev / build / deploy / test）
└── dist/                 # 构建产物（已 gitignore，不需要提交）
```

## 从哪里开始改

- **HTML 骨架** → `src/template.html`
- **JS 逻辑** → `src/index.js`
- **样式** → `src/styles.css`
- **图片、其他模块** → 放进 `src/` 即可
- **测试** → 和被测模块放在一起，命名 `src/xxx.test.js`

只需要记住两条约定：

1. **CSS 不要用 `<link>` 引入**，在 `index.js` 里 `import "./styles.css"` 就够了，style-loader 会把样式注入页面。
2. **图片要在 JS 里 import 才能被 webpack 处理**：

   ```js
   import logo from "./logo.png";

   const img = document.createElement("img");
   img.src = logo; // 构建后是带 hash 的 URL
   ```

   直接写在 `template.html` 里的 `<img src="./logo.png">` 不会被处理（没有配 html-loader）。

## 部署到 GitHub Pages

```bash
npm run deploy
```

这条命令会先执行构建，然后把 `dist/` 的内容推送到 origin 远程的 `gh-pages` 分支。

前提与说明：

1. 仓库必须已经配置好 origin 远程 —— `gh-pages` 只认 git remote，**不读 package.json 里的任何地址**，所以不需要 `homepage` 字段。
2. 首次部署后，到 GitHub 仓库的 Settings → Pages，选择 Deploy from a branch → `gh-pages` → `/ (root)`。
3. 以后每次 `npm run deploy` 都会重新构建并覆盖 `gh-pages` 分支，访问地址保持 `https://<用户名>.github.io/<仓库名>/`。

## 配置说明

- `entry` 的键名 `app` 决定产物文件名 `dist/app.bundle.js`
- `HtmlWebpackPlugin` 用 `src/template.html` 生成 `dist/index.html`，并自动插入打包后的 `<script>` 标签
- `.css` 由 `style-loader` + `css-loader` 处理
- `png / svg / jpg / jpeg / gif` 由 asset modules 处理
- 开发服务器不写磁盘，产物在内存里；`devServer.watchFiles` 让改动 `template.html` 也能触发刷新

需要新资源类型（如字体、音频）时，在 `webpack.common.js` 的 `rules` 里加一条：

```js
{
  test: /\.(woff2?|ttf|mp3|mp4)$/i,
  type: 'asset/resource',
}
```

## 作为模板使用时

派生新项目后**不需要修改任何配置**：`package.json` 的 `name` 只是占位符，构建流程不读它；`entry`、模板路径、输出目录都写在 webpack 配置文件里。
