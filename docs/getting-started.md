---
description: 从本地 wxa-runner 源码在浏览器里运行微信小程序：首次构建、启动原生小程序和 uni-app、Taro 编译产物，以及怎么用 Chrome DevTools 打断点、看请求、看页面结构。
---

# 快速开始

wxa-runner 在 Chrome 里用微信官方基础库运行小程序。需要 Node.js ≥ 18，基础库和编译器不用另外装。

## 启动

### npx 启动

> 暂时可用但是已被腾讯封禁，后续不维护。

原生小程序在 `project.config.json` 所在目录运行：

```bash
npx wxa-runner dev . --open
```

uni-app、Taro 等项目传入编译产物目录，例如：

```bash
npx wxa-runner dev ./dist/dev/mp-weixin --open
```

### 从本地源码启动

推荐直接使用克隆到本机的源码。下面的 `/path/to/...` 都需要替换成实际绝对路径。

首次使用时，先进入 wxa-runner 源码目录安装依赖并构建：

```bash
cd /path/to/wxa-runner
pnpm install --frozen-lockfile
pnpm build
```

构建会生成本地 `core`、`compiler`、`extractor` 等包需要的文件，并让 workspace 内的依赖指向这些本地包。通常只需要在首次使用和更新源码后执行。

然后进入小程序项目继续启动。原生小程序的目录就是 `project.config.json` 所在目录：

```bash
cd /path/to/miniprogram
node /path/to/wxa-runner/packages/cli/bin/wxa-runner.js dev . --open
```

uni-app、Taro 等项目改为传入编译产物目录：

```bash
cd /path/to/miniprogram
node /path/to/wxa-runner/packages/cli/bin/wxa-runner.js dev ./dist/dev/mp-weixin --open
```

如果经常在多个项目里使用，可以在 `~/.zshrc` 或 `~/.bashrc` 里封装成命令，之后用 `wxa-local` 代替 `wxa-runner`：

```bash
wxa-local() {
  node /path/to/wxa-runner/packages/cli/bin/wxa-runner.js "$@"
}
```

重新加载 shell 配置后，原来的启动命令可以写成：

```bash
wxa-local dev . --open
```

更新 wxa-runner 源码后，需要回到源码目录重新构建：

```bash
cd /path/to/wxa-runner
git pull
pnpm install --frozen-lockfile
pnpm build
```

### 原生小程序

在 `project.config.json` 所在目录运行：

```bash
wxa-local dev . --open
```

### uni-app

装上 Vite 插件，`npm run dev:mp-weixin` 时自动启动 wxa-runner：

```bash
pnpm add -D @open-wxa/unplugin-wxa-runner @open-wxa/wxa-runner-core
```

```ts
// vite.config.ts
import uni from '@dcloudio/vite-plugin-uni';
import wxaRunner from '@open-wxa/unplugin-wxa-runner/vite';

export default defineConfig({
  plugins: [uni(), wxaRunner()]
});
```

首轮编译完成后终端打印 wxa-runner 的地址，改了源码页面会自动[热更新](./hot-reload.md)。
插件要求 Node.js 20.19 / 22.12 及以上，端口、setup 等选项见[插件文档](https://github.com/open-wxa/wxa-extensions/tree/main/packages/unplugin#readme)。

<details class="details custom-block">
<summary>不用插件：两个终端分别启动</summary>

```bash
npm run dev:mp-weixin                         # 终端 1
wxa-local dev ./dist/dev/mp-weixin --open     # 终端 2
```

HBuilderX 创建的项目，产物在 `unpackage/dist/dev/mp-weixin`。

</details>

Taro 等其他框架：先跑框架的监听构建，再用 `wxa-local dev <编译产物目录>` 启动。

---

浏览器里会出现一台模拟手机，可以像模拟器一样点击、滚动。接下来常用的：

- 直接打开某一页：`http://localhost:3000/?path=pages/detail/detail&id=1`
- 跳过登录：见[启动前准备（setup）](./setup.md)
- 每次都调用完整路径：把上面的 `wxa-local` 换成 `node /path/to/wxa-runner/packages/cli/bin/wxa-runner.js`

## 调试

打开 Chrome DevTools（macOS `Cmd+Option+I`，Windows `F12`）：

| 想做什么 | 怎么做 |
| --- | --- |
| 打断点 | Sources 面板按 `Cmd+P`（Windows `Ctrl+P`），输入文件名，如 `pages/index/index` |
| 在控制台调用 `getCurrentPages()`、`wx.xxx` | 先把控制台左上角的上下文从 `top` 切到 `wxa-appservice` |
| 看 `wx.request` | Network 面板，地址形如 `/__wxa__/proxy/https/api.example.com/user/info` |
| 看页面结构 | Elements 面板，每个页面是一个 iframe |

报错和单步调试停在业务代码上，不会跳进基础库。

<details class="details custom-block">
<summary>被 setup 拦下的请求在 Network 里是什么样</summary>

显示为 `/__wxa__/mock/https/…`，控制台同时打印 `[wxa-runner] 拦截请求`。
它们靠 Service Worker 显示：第一次打开页面时，它装好之前的请求不显示；强制刷新（`Shift+Reload`）后、用局域网 IP 访问时不显示。请求本身不受影响。

</details>

## 下一步

- [启动前准备（setup）](./setup.md)：跳过登录、mock 原生 API、拦截 `wx.request`
- [在 Playwright 中测试](./testing.md)：写自动化测试
- [功能与限制](./features.md)：支持哪些组件和 API，和开发者工具的区别
