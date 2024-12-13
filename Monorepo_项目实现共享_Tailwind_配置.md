# Monorepo 项目实现共享 Tailwind 配置

12345

> 本文仅适用于有一定 monorepo 使用经验和原子化 CSS 爱好者。

在这篇文章中，我将介绍如何在 monorepo 项目中实现多个 package 之间共享 tailwind.css 配置，从而避免每次创建一个新包时要复制配置文件、统一配置项需要重复修改多次的问题。

需要注意的是，本文基于 turborepo 和 pnpm 工作区实现，如果你使用 Lerna、NX、Yarn 或者其他可能存在一些差异。

## 目录结构

用过 turborepo 的同学应该可以了解，它默认模板比其他 monorepo 方案多出了 apps 目录。

```sh
- apps
  - docs
  - web
- packages
  - ui
  - tsconfig
  - eslint-config-custom
  - tailwind-config # 这里就是我们要做的通用配置
- package.json
```

可见，把通用配置、UI 组件这类可以复用的包放在 packages/ 中，而业务相关的项目放在 apps/ 中，这样的目录方案使得项目变得更加清晰。

目前我们有 6 个包，他们分别为：

- 2 个业务包，docs 和 web，他们通常是可以运行的独立项目。
- 4 个通用包，ui 是组件库，其他是通用的配置。

可能需要用到 tailwind 通用配置的一般是业务包和 UI 组件包。

## 创建配置

将 tailwind 配置整合为一个通用的包，可以方便的在其他包里快速引用。

### 创建 package.json

首先，在 `packages/` 目录下创建 `tailwind-config` 目录，生成一个 `package.json`：

```json
{
  "name": "tailwind-config",
  "version": "0.0.0",
  "private": true,
  "main": "tailwind.config.ts",
  "types": "tailwind.config.ts",
}
```

### 安装依赖

这里我使用的是 pnpm：

```sh
pnpm add -F packages/tailwind-config tailwindcss
```

`-F` 是 `--filter` 的缩写，它是 pnpm 提供的参数，用于过滤你想指定的包，它提供了丰富的语法，有兴趣可以阅读[参考文档](https://pnpm.io/zh/filtering)。

```sh
pnpm --filter <package_selector> <command>
```

如果你使用其他包管理器，请自行参考，或者你可以 `cd packages/tailwind-config` 进行安装即可。

> 这个包并不需要 autoprefixer 和 postcss，因为它只提供配置文件，不需要参与编译工作。

### 创建配置文件

创建配置文件，`tailwind.config.ts`，当然你也可以用 js：

```ts
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./src/**/*.{vue,js,ts,jsx,tsx,mdx}",
    "node_modules/ui/**/*.{vue,jsx,tsx}",
  ],
  // ... 其他配置
};

export default config;
```

这里重点说一下 content 配置，它会影响我们的编译工作。

通常情况下，我们将 content 设置为要读取的模版文件，一般为 html、vue、jsx 等，主要是看你需要 tailwind 去识别哪里的 class。

`/**/*.{html,js}` 表示这个目录和这个目录下的所有子目录，匹配 html 和 js 文件。

**重点**

事例代码上还增加了 `node_modules/ui/**/*.{vue,jsx,tsx}`，这是因为我们在引入 packages/ui 这个包时，这里的组件 tailwind 默认不会去识别，导致未变异，样式丢失。

## 引用配置

这里分为两种引入方式，无需编译和需要编译的情况。

### 无需编译

这里通常是 `packages/ui` 这类通用组件库。

与配置包一样，如果你的组件库不需要编译，比如你不打算使用 vue-demi 将你的组件编译成 vue2、vue3 两个兼容版本，那么你只需要在 `package.json` 中的 `devDependencies` 添加：

```json
"devDependencies": {
  "tailwind-config": "workspace:*",
}
```

> 如果你使用其他包管理器，这里写法不同，请注意。

在 `packages/ui/` 目录下创建 `tailwind.config.ts`：

```ts
import type { Config } from 'tailwindcss';
import sharedConfig from 'tailwind-config';

const config: Pick<Config, 'presets'> = {
  presets: [sharedConfig],
};

export default config;
```

[presets](https://tailwindcss.com/docs/presets) 参数的作用是重复使用自己的预设配置。

### 需要编译

这种通常是 `apps/web` 这类业务包，它一般都编译于 webpack、vite 这类工具，所以我们除了上文需要做的那些，还有一些特殊配置。

除了 tailwindcss 还需要安装 PostCSS 和 autoprefixer。

```sh
pnpm add -F apps/web tailwindcss postcss autoprefixer
```

- 为什么要安装 postcss? 
  - 因为 tailwind 生成 css 需要通过 PostCSS 工具进行处理。
- 为什么要安装 autoprefixer
  - autoprefixer 是一个 PostCSS 插件，它确保生成的 CSS 在所有支持的浏览器中都能正常工作。
  
`tailwind.config.ts` 的配置方式与上文一致。

再增加一个 `postcss.config.js` 文件，配置一下 postcss：

```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

在源码的 src 目录下或其他位置创建一个 tailwind.css 文件，用于引入 tailwind 样式，并在你的入口文件引入这个 css。

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

如果你使用了其他的 tailwind 插件，也在这里引入。

## 总结

统一的 tailwind 配置，对于管理同一品牌下的多个 tailwind 项目来说非常有用，实现了颜色、字体和其他常见自定义的配置的唯一来源，使得项目越来越复杂的情况下，也可以轻松实现配置修改。