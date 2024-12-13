# 生成 PDF 用 Turborepo 管理，简单的事情复杂化？

最近在做一个生成报告的项目，稍微了解过这方面知识的同学大概都可以想到直接 HTML 写模板，利用 html2canvas + jspdf 两个库就可以实现，非常简单。但是为什么我采用 `Turborepo` 来管理这个项目呢？

> 有兴趣的同学在看本文前应先了解一下 `Monorepo` 的相关知识再来阅读本文。

本文主要讲解为什么采用 `Turborepo` 来管理这个项目，以及技术实现的细节、避坑指南。

## 需求分析

回到项目，我想很多公司在做一个项目时，都是拍脑袋决定的，基本上就会给你一个非常简单的需求让你去实现。例如我们这个报告项目，当时给的需求就是产品规划了一个报告模板，给到我，是否可以前端实现模板，数据后端接口获取，然后生成 `PDF`，对于这种简单的需求当然可以了，我想在座的各位也是手到擒来，直接创建仓库一把梭了。然而我没着急做，我先去了解一下这个项目未来的规划，了解到这应该是未来一个比较主要的业务，所以需求绝对不会像现在简单的一句需求就可以实现，围绕着这个模板，肯定会衍生出其他附属的项目。所以我打算采用单仓库的形式，采用 `Turborepo` 来管理这个项目。

事实证明，我这么做是非常明智的，经过几个月的迭代，目前除了模板这个项目又衍生出了 3 个需求：

1.  **自动生成报告**。目前模板是在浏览器中实现，需要手动操作，并不能实现自动化生成报告的需求。所以这里我采用了 `Nest` 写了个接口，提供给后端去调用，使用 `puppeteer` 去自动化处理和生成报告，然后再将报告通过 `minio` 存储起来即可。
2.  **后台管理系统**。增加各种设备管理等操作、报告数据生成、报告编辑等等各种功能。模板项目已经存在三套模板，未来会对某些企业会专门定制，所以我打算后台管理系统创建一个新的项目去做，模板项目专注各类模板的开发。
3.  **桌面端应用**。这个项目部署在内网中，公网无法访问，销售团队在对外时，采用 VPN 链接内网访问对他们来说比较麻烦，本地部署成本又高，所以决定开发一个桌面端应用供销售团队展示使用，只需要提供模板编辑和导出 PDF 模板即可，也就是把模板项目打包成桌面端应用，这里为了方便我没有采用熟悉的 `Electron` 和 `Tauri`，而是利用 `cordova` 帮助我打包，实际上他也是打包成 Electron 应用，只不过节约了 Electron 接入的成本。

未来我觉得还可能会衍生出的项目还有小程序或者 APP，这是目前的一个项目目录：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b60763f08aec4aabb55749e2b007ff91~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=482\&h=806\&s=70786\&e=png\&b=23262b)

可见我把这几个项目统统放在一个仓库的 `apps/` 目录下，顺便提到的是 `packages/` 目录下统一了 `ESLint`、`TS`、`tailwind` 的配置，全局的环境变量，通用的工具库和组件。

## Turborepo 优势

把这一堆项目塞到一个仓库中，我想对于很多不了解 Monorepo 的同学来说是无法接受的。那么采用 **Turborepo** 的好处究竟有哪些呢？

### 开发体验

前文都了解到了，所有的衍生项目都是围绕着 `template` 这个项目开发的，可以说是依赖于它，那么不论我开发哪个项目时，我都要启动 template 这个项目。如果是多仓库的模式，我需要命令行进入 template 目录，然后运行，再打开新的命令行标签页，找到开发的项目，再运行...

开发过程中，几乎每天都会在这几个项目中各种穿梭，如果启动多个命令行标签页，看控制台输出非常麻烦，启动程序也是非常的繁琐。那么在 Turborepo 下，只需要执行一次 npm run dev 即可开启丝滑的开发体验。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b115ab55c0f450e876d2c6223684cdb~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2048\&h=1536\&s=654517\&e=png\&b=3f4352)

可以看到左侧展示不同的项目，他们的所有控制台输出都在这一个命令行标签页中展示了。

### 构建体验

Turborepo 除了编排任务的能力出色，还有就是构建的缓存功能，它可以帮你节省大量的构建时间：

*   当你第一次构建时，它会同时构建你的所有项目（除非你自己定义编排任务），以最大并行数量来进行构建。
*   都构建过一次之后，会将构建内容缓存在 node\_modules/.cache/turbo/ 目录中。
*   如果此时你再执行一次构建你会发现瞬间就构建完成了，这是因为检测到你的代码没有变动，直接提取缓存。
*   如果此时你修改了某个项目的代码后再构建，你会发现，其他的项目使用缓存，而修改的项目重新编译了。

这样的话节省了大量的时间，也无需关心某个项目是否编译过了。

曾经的一个项目我就是自己编排的构建方式，需要手动去选择要编译的项目，有兴趣的可以了解一下 [《微前端与Monorepo的架构设计》](https://juejin.cn/post/7225800207329230905)，文章内有提到这块的实现，使用 Turborepo 之后，极大的提升了构建体验。

### 代码共享

多仓库的代码共享是很难实现的，有的人说，可以发布 npm 包去实现代码的共享，这里我提出几点不太合适的理由：

1.  不方便维护，过多的包会创建多个仓库，版本管理也是难点。
2.  开发过程蹩脚，需要发布，本地再更新。
3.  业务代码一般属于商业机密，不适合发到公共场合，搭建私服或者购买企业账号都浪费成本。
4.  如果你的代码更新了，但是你的依赖包没有更新，容易出 bug。

在公司项目中，存在多个可以共享的包：

*   **eslint-config-custom**: ESLint 配置。
*   **tailwind-config**: Tailwind 配置。
*   **tsconfig**: TS 配置。
*   **utils**: 通用工具库。
*   **ui**: 通用组件库。
*   **env**: 全局环境变量，经常变得可能就是环境变量，如果每个项目都要配置一遍，那就太麻烦了。

## 项目基建

从本章节开始，我将介绍这个模板项目的技术实现的细节，包括前端、后端、桌面端应用的实现，以及一些技术细节的讲解和避坑指南。

### 初始化项目

建议直接使用 pnpm 创建项目，因为 Turborepo 可以使用 pnpm 来进行保管理，所以我们直接使用 pnpm 来创建项目：

```bash
pnpm dlx create-turbo@latest
```

随后就是输入项目名称、选择包管理工具，选择 pnpm workspaces 即可。如果你习惯用 npm 或者 yarn 也是可以的。

> npm 采用的是 `lerna` 来管理，yarn 采用的是 `yarn workspace` 来管理，pnpm 采用的是 `pnpm workspaces` 来管理。

### 创建业务项目

Turborepo 的业务项目建议采用 `apps/` 目录来存放，这里你可以选择任意脚手架去初始化的你的项目。我使用的 vite 创建的 vue-ts 项目。

进入 `apps/` 目录，执行：

```bash
pnpm create vite template --template vue-ts
```

其他的项目也是类似的，这里不再赘述。

### Packages

在 `packages/` 目录下存放一些通用的配置，例如 `ESLint`、`TS`、`tailwind` 的配置，通用的工具库和组件。

#### ESLint

在 `packages/` 目录下创建一个 `eslint-config-custom` 目录，创建一个 vue 的配置文件：

```js
const { resolve } = require("node:path");

const project = resolve(process.cwd(), "tsconfig.json");

module.exports = {
  extends: [
    "@vercel/style-guide/eslint/browser",
    "@vue/eslint-config-typescript",
  ].map(require.resolve),
  parserOptions: {
    ecmaVersion: "latest",
  },
  settings: {
    "import/resolver": {
      typescript: {
        project,
      },
    },
  },
  ignorePatterns: ["node_modules/", "dist/", ".eslintrc.js"],
  rules: {
    // ...
  },
};
```

package.json 中修改 "name": "eslint-config-custom"。

在业务项目 template 中 package.json 中 devDependencies 添加：

```json
"eslint-config-custom": "workspace:*",
```

`workspace:*` 是 pnpm 的语法，指向的是 `packages/` 目录下的所有包。

在 .eslintrc.js 中添加：

```js
require("@rushstack/eslint-patch/modern-module-resolution");

module.exports = {
  root: true,
  extends: ["plugin:vue/vue3-essential", "custom/vue"],
};
```

`custom/vue` 是指向 `eslint-config-custom` 的 vue.js 配置，这块不理解的去阅读一下 ESLint 的文档。

#### Tailwind

在 `packages/` 目录下创建一个 `tailwind-config` 目录，创建一个 tailwind.config.ts 文件：

```ts
import type { Config } from "tailwindcss";
const path = require("path");

const config: Config = {
  mode: "jit",
  content: [
    "./src/**/*.{vue,js,ts,jsx,tsx,mdx}",
    path.join(path.dirname(require.resolve('../ui')), '/src/**/*.{vue,js,ts,jsx,tsx,mdx}'),
  ],
};

export default config;
```

这里注意 content 的配置，增加了 ui 的路径，这样可以让 ui 的组件也能被 tailwind 扫描到，不然你的 ui 组件会没有样式。

`package.json` "name": "tailwind-config"。

在项目中安装与 ESLint 类似的方式，自行配置。

#### TS

TS 在 vue 项目和 nest 项目中都有使用到。

在 `packages/` 目录下创建一个 `tsconfig` 目录，创建一个 base.json:

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "display": "Default",
  "compilerOptions": {
    "composite": false,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "inlineSources": false,
    "isolatedModules": true,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "preserveWatchOutput": true,
    "skipLibCheck": true,
    "strict": true,
    "strictNullChecks": true
  },
  "exclude": ["node_modules"]
}
```

这个文件是基本的配置，vue 项目和 nest 项目都会继承这个配置。

配置 vue 的话就是创建 vue.json：

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "display": "Vue",
  "extends": ["./base.json", "@vue/tsconfig/tsconfig.dom.json"]
}
```

nest 与之类似。

package.json "name": "tsconfig"。

在业务项目中，安装与 ESLint 类似的方式，配置时，在 tsconfig.json 中增加:

```json
{
  "extends": "tsconfig/vue.json",
}
```

#### UI 与 Utils

组件与通用的工具库，都可以放在 `packages/` 目录下，安装方式也与上述的方式一致。

### 环境变量

环境变量不存在于 packages 中，而是在根目录创建一个 .env 文件，区分环境变量的话，可以创建 .env.development、.env.production 等文件。常见的环境变量配置有接口的地址、端口号、是否开启 mock 等等。

#### Vite 获取环境变量

在 vite.config.ts 中加入 envDir 配置：

```ts
import { fileURLToPath, URL } from 'node:url'

export default defineConfig({
  envDir: fileURLToPath(new URL('../../', import.meta.url)),
})
```

#### Nest 获取环境变量

可以在 app.module.ts 中引入 `dotenv`：

```ts
// 获取根目录下的 .env 文件
const envFilePath = [
  `${resolve(process.cwd(), '../../')}/.env`,
  `${resolve(process.cwd(), '../../')}/.env.development`,
  `${resolve(process.cwd(), '../../')}/.env.production`,
];

@Module({
  imports: [
    ConfigModule.forRoot({
      envFilePath,
    }),
  ]
})
export class AppModule {}
```

在 service 中使用：

```ts
this.configService.get<string>('...');
```

其他的项目请自行查阅文档。

## 模板开发

模板开发是非常简单的，这里我采用了 `vite` + `vue3` 来开发。

### 生成 PDF

生成 DPF 采用了 `html2canvas` + `jspdf` 来实现。我们先从开发一个生成 PDF 的函数开始：

因为生成 PDF 的功能可以说是个通用的功能，所以我把这个函数放在了 `packages/utils` 中，这样其他的项目未来的业务需求中也有可能会用到，可以直接引入使用。

首先我们要先安装 `html2canvas` 和 `jspdf`：

```bash
pnpm add -F utils html2canvas jspdf
```

> \-F utils 是指安装到 packages/utils 包中，其他包的安装方式也是类似的。

安装成功后打开 `packages/utils/packages.json`，可以看到这两个包已经安装成功了。

```json
{
  "dependencies": {
    "html2canvas": "^1.4.1",
    "jspdf": "^2.5.1"
  }
}
```

在 `packages/utils/src/` 中创建一个 `exportPdf.ts` 文件，先引入 `html2canvas` 和 `jspdf`：

```ts
import html2canvas from 'html2canvas'
import Jspdf from 'jspdf'
```

然后写几个函数，先写一个将 html 转换为 canvas 的函数：

```ts
function getPageImage(element: HTMLElement) {
  return new Promise<HTMLCanvasElement>((resolve, reject) => {
    html2canvas(element, {
      scale: 2,
    })
      .then((canvas) => {
        resolve(canvas)
      })
      .catch(() => {
        reject(new Error('HTML转换图片失败'))
      })
  })
}
```

这里注意 scale 的配置，这个是用来设置清晰度的，这里设置为 2，可以根据实际情况调整，这个值越大，生成的图片越清晰，但是也会增加生成的图片的大小。

然后写一个将 canvas 转换为 PDF ，并且下载的函数：

```ts
async function downloadPdf(
  canvas: HTMLCanvasElement | HTMLCanvasElement[],
  fileName: string,
  isSave: boolean
) {
  let width = 0
  let height = 0
  if (isCanvasElementArray) {
    width = canvas[0].width
    height = canvas[0].height
  } else {
    width = canvas.width
    height = canvas.height
  }
  const doc = new Jspdf('p', 'px', [width, width * Math.sqrt(2)])
  const isCanvasElementArray = Array.isArray(canvas)
  if (isCanvasElementArray) {
    canvas.forEach((item, index) => {
      if (index !== 0) doc.addPage()
      doc.addImage(
        item.toDataURL('image/jpeg', 1.0),
        'JPEG',
        0,
        0,
        item.width,
        item.height
      )
    })
  } else {
    doc.addImage(
      canvas.toDataURL('image/jpeg', 1.0),
      'JPEG',
      0,
      0,
      width,
      height
    )
  }
  if (isSave) {
    const pdf = await doc.save(`${fileName}.pdf`, {
      returnPromise: true,
    })
    return pdf
  } 
  return doc
}
```

这里说一下传入的三个参数：

1.  **canvas**: HTMLCanvasElement | HTMLCanvasElement\[]，这里是 HTML 转换后的 canvas，可以是一个数组，也可以是一个，因为有可能是多页的 PDF。
2.  **fileName**: string，生成的 PDF 的文件名。
3.  **isSave**: boolean，是否保存，如果是保存的话，会直接下载，如果不保存，会返回一个 Jspdf 实例。这个是预留如果不是本地生成 PDF 的话，可以直接返回 Jspdf 实例，然后在后端生成 PDF，例如调用接口将 PDF 的 File 传给后端。

生成 PDF 调用了 jspdf，new Jspdf 的参数分别是：

1.  **'p'**，表示纵向，'l' 表示横向。
2.  **'px'**，表示单位是 px。
3.  **\[width, width \* Math.sqrt(2)]**，表示宽高比为 1:√2，这是 PDF 的标准比例。

调用后返回 doc，用于操作文稿的内容。

doc.addImage() 是将 canvas 添加到 PDF 中，参数分别是：jsPDF.addImage(imageData: string | HTMLCanvasElement | HTMLImageElement | Uint8Array | RGBAData, format: string, x: number, y: number, w: number, h: number, alias?: string | undefined, compression?: ImageCompression | undefined, rotation?: number | undefined): Jspdf (+2 overloads)

1.  **imageData**，使用 item.toDataURL('image/jpeg', 1.0)，将 canvas 转换为图片。
2.  **format**，'JPEG'，表示图片格式。
3.  **x**，0，表示 x 轴的位置。
4.  **y**，0，表示 y 轴的位置。
5.  **w**，item.width，表示图片的宽度。
6.  **h**，item.height，表示图片的高度。

其他可选参数可以查看 jspdf 的文档。

doc.save() 是保存 PDF，参数分别是：jsPDF.save(filename: string, options?: { returnPromise: boolean })

1.  **filename**，`${fileName}.pdf`，表示文件名。
2.  **options**，{ returnPromise: true }，表示返回一个 Promise。

然后就是对外导出这个函数，提供给其他项目使用：

```ts
export default async function exportPdf(
  element: HTMLElement | HTMLElement[],
  fileName: string,
  isSave = true
) {
  return new Promise((resolve) => {
    const isHTMLElementArray = Array.isArray(element)
    if (isHTMLElementArray) {
      const getPageImages: Promise<HTMLCanvasElement>[] = []
      element.forEach((e) => {
        getPageImages.push(getPageImage(e))
      })
      Promise.all(getPageImages).then(async (allCanvas) => {
        const doc = await downloadPdf(allCanvas, fileName, isSave)
        resolve(doc as Jspdf)
      })
    } else {
      getPageImage(element).then(async (canvas) => {
        const doc = await downloadPdf(canvas, fileName, isSave)
        resolve(doc as Jspdf)
      })
    }
  })
}
```

### 避坑 tailwind

在使用 html2canvas + tailwind 的组合时，可能会产生一些问题，导致不能按照预期转换样式，导致内容错乱，这里总结一下可能会出现的问题：

1.  有兴趣可以阅读一下 [html2canvas 支持的 CSS 属性](https://html2canvas.hertzen.com/features)，在写模板时尽量避免不支持的 CSS 属性，以免导出不符合预期的样式。
2.  tailwind 的样式是通过类名来控制的，而 html2canvas 是通过解析 HTML 来生成 canvas 的，所以可能会出现样式不符合预期的情况。
3.  字体下沉，应该是 tailwind 预检查的问题，可以在 tailwind.config.ts 中配置：

```css
@layer base {
	img {
		@apply inline-block;
	}
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f32055929f224fc7a647132687f201dc~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=751\&h=183\&s=30713\&e=png\&b=fdf6f6)

4.  图片必须是同源的，否则会导致图片无法加载，这个是浏览器的安全策略，可以在后端将图片转换为 base64，然后再传给前端，或者做代理。
5.  字体同样可能会出现图片的问题。
6.  避免使用高级 CSS 特性，html2canvas 大概率都不会支持。

总之，建议直接用 CSS 去写模板样式，避免使用 tailwind，因为太不可控，反而浪费时间。

### 读取 CSV 数据

在某些时候，可能数据不走接口，而是通过导入 CSV 文件来获取，这里可以使用 `papaparse` 来读取 CSV 文件，这是一个非常好用的库，可以直接读取 CSV 文件，转换为 JSON 格式，同样也可以将 JSON 转换为 CSV 文件。

具体实现非常简单，可以参考一下官方文档：[papaparse](https://www.papaparse.com/docs)。

这类的方法我都会放在 `packages/utils` 中，方便其他项目调用。

## 自动化生成报告

自动化生成报告是一个非常重要的功能，这里我采用了 `Nest` + `puppeteer` 来实现。虽然目前只写了一个接口，看起来用 nest 比较大材小用，但是未来这个项目会有很多的业务逻辑，所以我选择了 nest。

Puppeteer 是一个无头浏览器，可以模拟浏览器的行为，例如点击、输入、滚动等等，这里我们只需要用到他的截图功能，将页面截图，然后生成 PDF 即可。它本身就是就支持生成 PDF 的功能，但这里因为已经在模板项目中使用 html2canvas + jspdf 生成 PDF 了，所以这里就不再使用 puppeteer 生成 PDF 了。理论上讲 puppeteer 生成 PDF 的效果会更好，因为它是直接生成 PDF，而不是转换为图片再生成 PDF，可以支持全部的 CSS 属性。

如果你没有了解过 puppeteer，可以先看一下官方文档：[puppeteer](https://pptr.dev/)，或者阅读我曾经写过的一篇文章：[《使用前端技术破解掘金滑块验证码》](https://juejin.cn/post/7257386139849801789)，里面有简单的入门教程。本文不做详细的介绍，只讲解 puppeteer 使用上的一些细节。

### 修改下载路径

模板项目生成 PDF，最终是以下载的形式保存到本地的，这里我们需要修改 puppeteer 的下载路径，将 PDF 保存到指定的路径。

```ts
const downloadPath = path.resolve(__dirname, '../download'); // 自定义下载路径
const client = await page.createCDPSession();
await client.send('Page.setDownloadBehavior', {
  behavior: 'allow',
  downloadPath,
});
```

`page.createCDPSession()` 是创建一个 CDPSession，CDPSession 是 Chrome DevTools Protocol 的一个会话，可以通过这个会话来发送命令给浏览器，这里我们发送了一个 Page.setDownloadBehavior 命令，设置下载路径。

### 判定文件是否下载完成

生成文件并下载后，我们是无法监听到文件是否下载完成的，这里我们可以通过反复检查文件是否存在来判断文件是否下载完成。

```ts
await new Promise<void>((resolve) => {
  const timer = setInterval(() => {
    if (existsSync(filePath)) {
      clearInterval(timer);
      resolve();
    }
  }, 100);
});
```

existsSync 是 node.js 的一个方法，用来判断文件是否存在，这里我们反复检查文件是否存在，如果存在就清除定时器，结束 Promise。

### 性能优化

Puppeteer 是一个无头浏览器，所以性能是非常重要的，这里我们可以通过一些方式来优化性能：

1.  启动浏览器会占用大量的内存，我觉得至少保留 2G 内存比较保险。
2.  如果需要连续产生多个 PDF，可以考虑使用 `browser.newPage()` 来创建新的页面，而不是新增加浏览器实例，这样可以减少内存的占用。
3.  如果短期内要产生大量 PDF，应考虑队列的方式(@nestjs/bull)，或者简单采用 for + await 的方式，避免同时打开大量页面，导致内存占用过高。

## 后台管理系统

后台管理大家都很熟悉，这里没什么好说的，它与模板项目是相互独立的，在需要手动编辑报告时会跳转到模板项目，然后保存后再跳转回来。

## 桌面端应用

桌面端是个临时的需求，想给客户展示时用，所以我采用了 [cordova](https://cordova.apache.org/) 来打包。

Cordova 是你想快速将 web 页面生成为桌面端应用、移动端应用的一个工具，只需要简单的几个命令即可。

### 安装 Cordova

首先你需要安装 Cordova，这里我采用全局安装：

```bash
npm install -g cordova
```

### 创建项目

使用命令行工具创建一个空白的 Cordova 项目。进入 apps/ `cordova create <path>` :

```bash
cd apps/
cordova create desktop
```

### 添加平台

创建好项目后，进入项目目录，添加平台，这里我们只需要桌面端，所以增加 electron，可见 cordova 桌面平台是基于 electron 实现的。

```bash
cd desktop
cordova platform add electron
```

### 打包

在打包前应该先将模板项目打包，然后将打包文件放在 apps/desktop/www 中，然后再打包桌面端应用。

打包非常简单，只需要执行：

```bash
cordova build --release
```

### 自动化打包

但是每次打包都需要手动将 apps/template/dist 目录下的文件都复制到 apps/desktop/www 中，这样非常麻烦，所以我们要做成自动化的方式，因为 template 使用 vite 作为构建工具，所以可以写一个 vite 插件来完成构建后复制文件的操作，在 apps/template/ 中创建 copyFilesPlugin.ts

```ts
import fse from 'fs-extra'

function copyFilesPlugin() {
  return {
    name: 'copy-files-plugin',
    closeBundle() {
      fse.emptyDirSync('../desktop/www')
      fse.copySync('dist', '../desktop/www')
    },
  }
}

export default copyFilesPlugin
```

这里使用了 fs-extra 来操作文件，这是 node.js 的一个文件操作库，比原生的 fs 更好用。

在 vite.config.ts 中引入这个插件：

```ts
import copyFilesPlugin from './copyFilesPlugin'

export default defineConfig({
  // ...
  plugins: [vue(), copyFilesPlugin()],
})
```

这样 template 打包后会复制到 apps/desktop/www 中。

## package.json

常规开发时，基本不需要 cordova 项目也启动，所以在 package.json 中配置 scripts 时，不定义 build 或者 dev 这种指令，可以修改为 `desktop:dev`、`desktop:build` 这种指令，这样在根目录执行 npm run dev 时不会启动它。

desktop 项目依赖于 template 项目，所以需要先执行 build 后再执行 desktop:build，所以我们可以在 package.json 中配置：

```json
{
  "scripts": {
    "dev": "cross-env NODE_ENV=development turbo run dev",
    "build": "cross-env NODE_ENV=production turbo run build",
    "desktop:build": "npm run build && turbo run desktop:build",
  },
}
```

这样就可以按照正确的顺序打包了。

## turbo.json

如果你想在根目录下执行 `desktop:build`，你需要在 turbo.json 中 `pipeline` 增加这个指令：

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".output/**", "dist/**"]
    },
    "lint": {},
    "dev": {
      "cache": false,
      "persistent": true
    },
    "desktop:build": {
      "dependsOn": ["^build"],
      "outputs": ["desktop/paltforms/electron/build/**"]
    },
  }
}
```

outputs 是指定输出的文件，这里是指定 electron 的打包文件，turbo 才会知道缓存哪些文件，重复打包时会直接提取缓存，节省时间。

## 总结

这篇文章主要讲解了 Turborepo 的使用，以及如何使用 Turborepo 来管理多个项目，以及如何优化 Turborepo 的开发体验，以及如何在 Turborepo 中实现一些功能，例如自动化生成报告、桌面端应用等等。

使用 Turborepo 可以极大的提升开发体验，避免了多仓库切换开发的繁琐，提高了代码的复用性，极大的提高了构建的效率。

我非常建议大家在这种类似的场景下使用 Turborepo，并不是要开发 vue 那样的框架才需要 monorepo，其实只要是业务相互存在关联关系的项目，都可以使用 monorepo 的方式来管理。

如果你不喜欢 turborepo，其实还有很多种方式可以选择：

*   **Lerna**：它是一个老牌的 Monorepo 工具了，现在不建议使用。
*   **Yarn、Pnpm**：他们都提供了 workspace 功能，Turborepo 也是基于他们去实现的。
*   **Rush**：Rush 是微软开发的，它的配置要比 lerna 和 turborepo 都要复杂得多。
*   **Nx**：Nx 也是非常优秀的选择方式，它区分了 Nx 和 Nx Cloud，Nx Cloud 是收费的，但是提供了很多的功能，例如自动化 PR、自动化部署等等。

## 参考

*   [Turborepo](https://turbo.build/repo/docs)
*   [Vite](https://vitejs.dev/)
*   [Nest](https://nestjs.com/)
*   [Puppeteer](https://pptr.dev/)
*   [Cordova](https://cordova.apache.org/)
*   [fs-extra](https://github.com/jprichardson/node-fs-extra)
*   [papaparse](https://www.papaparse.com/docs)
*   [html2canvas](https://html2canvas.hertzen.com/)
*   [jspdf](https://github.com/parallax/jsPDF)
