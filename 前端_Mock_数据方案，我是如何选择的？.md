# 前端 Mock 数据方案，我是如何选择的？

## 前言

`Mock` 可以翻译为“模拟”，在前端开发中通常指模拟后端接口和数据。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7c4275bcd184767a41a8349992f248e~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=733&h=296&s=86366&e=png&b=fefefe)

目前针对 mock 的方式可以说是五花八门，本文通过对比分析主流方案，帮助大家选择适合自己方案。

## 为什么用?

> **省流：我联调时间紧张，不想加班。**

我们在开发的时候，一般都是前后端同时开发，尤其是某些后台系统开发时，很多前端同学分分钟就可以做完静态页面的开发，然后就是摸鱼时间，等待后端的接口。如果你的**后端同事非常配合**，愿意在设计数据库的时候顺便定好返回的数据结构和字段，并且**联调时间很紧张**的时候，这时候你应该考虑着手去做一下前端 mock，为联调争取时间。

## 为什么不用?

> **省流：我联调时间充足，不如摸鱼。**

不论什么技术，都存在**反对**的声音：

- `不切实际`：Mock 的数据并不是真实的后端数据，后端有时都确定不了数据结构，可能会导致前端开发人员在开发过程中对接口的理解出现偏差，最终导致与后端接口不兼容的情况。
- `浪费时间`：编写模拟数据可能会浪费一些时间，特别是在接口比较复杂的情况下。

存在反对的声音是很正常的，可能是还没有真正遇到需要 mock 的场景，或者是因为了解到的 mock 方案并不适用。

写这篇文章，查看了许多看了文章，在评论区总结了很多人对 mock 这件事存在的`误区`：

**1. 后端接口每一个我都需要 mock。**

其实真的没必要，使用 mock 时，可以去模拟一些容易在联调时出问题的接口。这就类似烧水的问题，你在等待水烧开的时候，可以去拉屎，但没必要同时去吃口饭。

**2. 后端接口、字段或结构修改时，mock 接口也要同步修改。**

这个时候接口已经出了，你要做的是修改你的业务代码，而不是 mock 接口。这个时候 mock 已不再重要，你删除它反而更好，调用真实的接口去测试不是更好吗？

**3. 后端都确定不了数据结构，怎么 mock 都是无用的。**

连后端都无法确定时，那前端确实没必要 mock，等着就可以了。但大多数的时候，接口的结构都是类似的，用你多年的开发经验和团队配合，即使后端什么也没说，你也应该可以猜个大概，对于这样的接口我们确实可以 mock，而真实的接口拿到时，基本上也就是改改字段名称。或许可以再大胆一点，你用自己定义好的结构，分享给后端，没准也可以让后端按照你的希望结构返回给你。

> 当收益大于成本时，你才应该考虑去做。

如果你现在觉得 mock 也是可以做的，那么不妨看一下下文，我对主流 mock 方案的对比。

## 主流 Mock 方案对比

> **省流：推荐使用平台类或请求拦截的方案。**

本章节讲解六类**常见的 mock 方案**：

- 代码侵入
- 抓包工具 (Fiddler、Charles)
- 本地服务 (json-server、express)
- 浏览器插件 (Requestly)
- 平台 (Apifox、YApi)
- 请求拦截 (Mock.js)

通过 **5 个维度**去分析他们的**优点和缺点**：

- `简单性`：不需要复杂的配置和部署过程，可以快速地创建和管理模拟数据。
- `灵活性`：可以满足不同的开发需求，例如支持不同的请求类型、不同的请求方式等。
- `真实性`：能够生成更接近真实的数据，可以模拟不同的数据类型、格式和结构。
- `维护性`：可以方便地更新和修改模拟数据。
- `合作性`: 可以在团队内方便的共享模拟数据。

并且按照我个人的理解对其标记出**推荐指数**，5 分制。

### 1. 代码侵入

**推荐指数**：⭐

代码侵入可以说是开发中最常见的方式，可以说每个人都这么做过。这种方式虽然不好，但是极为方便，适合简单的逻辑开发，即用即删，否则前期一时爽，后期火葬场。

**常见方案**

- 直接在代码中写死
- 导入本地 json 文件

**优点**

- 方便快捷。

**缺点**

- 切换到真实接口时比较麻烦，需要删除或注释 mock 代码，接口越多越容易出错。
- 一切需要侵入业务代码的都可以说是不好的。

### 2. 抓包工具

**推荐指数**：⭐⭐

使用抓包工具可以对网络请求进行拦截，将其替换为我们需要的 Mock 数据，这也不失为一种 Mock 方式。其优点主要是真实性强，但这种方式操作步骤比较繁琐，不方便统一配置，Mock 成本较高。

**常见方案**

- Fiddler
- Charles

移动端开发可以了解一下，Web 开发成本太高，这里不做分析。

### 3. 本地服务

**推荐指数**：⭐⭐⭐

本地服务这种方式比较自由，你可以用任何技术实现，甚至你可以用 express、koa、nest 这些方案，自己写一些接口，但是多少是有点毛病了，不如直接干全栈了。

当然也有一些非常不错的方案，可以简单的实现本地服务 + mock 数据。

**常见方案**

- [json-server](https://github.com/typicode/json-server)，可以通过一个 **JSON** 文件快速创建一个模拟的API服务器。
- 自建服务端，可以通过 proxy 切换接口。

### 4. 浏览器插件

**推荐指数**：⭐⭐⭐⭐

浏览器插件的方式与平台类相似，它运行于你的浏览器，避免了本地部署或者使用第三方的服务。但如果你想团队协作，还是绕不开三方服务。

**常见方案**

- [Requestly: Open Source HTTPs Debugging Proxy](https://requestly.io/) ([Chrome 应用商店](https://chrome.google.com/webstore/detail/requestly-open-source-htt/mdnleldcmiljblolnjhpnblkcekpdkpa?utm_source=ext_sidebar&hl=zh-CN))
- [Mock:Intercept and directly return data](https://chrome.google.com/webstore/detail/mockintercept-and-directl/kmphamhphplpcjcabjdjjklfjmgkmpba?utm_source=ext_sidebar&hl=zh-CN)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aef488bd878e49ecad71b1f77633ce74~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=640&h=400&s=167091&e=png&b=202020)

（上图为 Requestly）

**优点**

- 安装简单，只需要在插件商城安装即可。
- 学习成本极低，拥有可视化操作界面，几乎零成本上手。
- 使用便捷，在浏览器中调试时随时修改。

**缺点**

- 不适合团队协作，几乎涉及团队协作的时候都需要付费使用。
- 浏览器插件可能比较吃性能，特别是数据量比较大的时候。
- 安全第一，小心使用不良插件。

### 5. 平台类

**推荐指数**：⭐⭐⭐⭐⭐

大家多少都有接触过接口管理工具，常见的有 Swagger、Postman 等。接口管理工具 + Mock 数据组合成为了平台类的方案，这类方案一般提供了云端服务或者本地化部署（可能收费）。

老外也比较流行使用这种方式，看来这类真的是**有利可图**。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d645739b92c042fabd7f62ec607ed6c1~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2600&h=1646&s=834952&e=png&a=1&b=1f2025)

（上图为 Apifox）

其具备可视化界面也使得 mock 数据简单了很多。

**常见方案**

- [Apifox](https://apifox.com/)，相当于 Postman + Swagger + Mock + JMeter 组合。
- [YApi](https://hellosean1025.github.io/yapi/)，具备权限管理、Mock、数据导入、自动化测试等功能。
- [Moco](https://github.com/dreamhead/moco)，这是个 java 项目，对前端同学不太友好。

**优点**

- 配置强大，一般平台已经提供了非常丰富的封装，几乎适用于任何项目和场景。
- 接口管理与 Mock 一体，使用起来更方便。
- 通常提供了丰富的数据生成规则，拿来直用，生成的数据更接近真实。

**缺点**

- 依赖后端，与其让后端配合 mock，不如让他们加速提供接口。
- 不建议使用云端服务，毕竟公司项目和这些平台谁先凉凉谁也拿不准。
- 私有化部署依赖团队基建，需要运维人员配合搭建。

> 曾经有一个叫 [easy-mock](https://github.com/easy-mock/easy-mock) 的在线 mock 服务平台，早已不维护，我们要从中吸取教训。

### 6. 请求拦截

**推荐指数**：⭐⭐⭐⭐⭐

请求拦截是通过拦截 XMLHttpRequest 或 fetch 等网络请求，然后根据自定义规则返回 mock 数据。在国内，mock.js 无疑是最优秀的请求拦截方案。

**常见方案**

- [Mock.js](http://mockjs.com/)，几乎在国内是统治地位。
- [axios-mock-adapter](https://github.com/ctimmerm/axios-mock-adapter)，个人不推荐。

**优点**

- 不再依赖后端，前端自主实现。
- 不需要修改现有的逻辑代码，直接拦截 Ajax 请求，返回模拟的响应数据。
- Mock.js 还提供了数据模板定义功能，可以返回更加真实的数据。

**缺点**

- 如果非要挑一些缺点，那就是需要安装配置环境，有一定的学习成本。

## 我是如何选择的？

> **省流：以上的方案一个没有一个适合我的，我选择了 MSW + Faker。**

经过上文的分析，最后推荐了`平台类`和`请求拦截`的两种方式。

本章节开始，讲述一下我的选型方案、原理和实现方式。

### 技术选型

没有选择平台类的方案，原因除了上文提到的原因，最主要还是想 mock 在前端单独实现即可，不麻烦其他同事。

请求拦截的方式完全可以满足我的需求，但是我并没有选择使用 mock.js，以下是几点原因：

- 只支持 Ajax，不支持 Fetch。
- 只支持 Rest API，不支持 GraphQL API。
- 不支持 ts，这点在伪造数据时，还需要记住各种规则，没有代码提示。
- 可以拦截请求并返回模拟数据，但是无法模拟真实的网络请求（在 DevTools 中看不到）。
- 多年不维护，虽然功能已经很强大，但还是无法满足我的需求。

后来发现了一个更加强大的库，叫做 [Mock Service Worker](https://mswjs.io/)（下文简写为 MSW），它可以在浏览器中模拟真实的网络请求和响应，从而更好地测试和调试。

MSW 只解决了模拟接口请求，但是不支持模拟数据，数据需要你自己造，那么我想到了一个制造假数据（但很真实）的库，名字叫 [Faker.js](https://fakerjs.dev/)。

我最后的技术选型就是 `MSW` + `Faker`。

### 原理

既然比 mock.js 强大，那么实现原理肯定也不同，`MSW Browser` 使用 [Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) 来拦截实际请求，
Service Worker 是在浏览器后台运行的 JavaScript 脚本，它可以拦截和处理浏览器发出的网络请求，从而实现离线缓存、消息推送、网络代理等功能。

其实你并不需要理解关于 [Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) 的内容。

上文提到 `MSW Browser`，它是运行在浏览器端的，还有 `MSW Node`，可以与现有的测试框架（如 Jest、Mocha 等）集成。

## 实现方式

本章节通过 `vite` 创建一个 `vue3` + `ts` 项目，去集成 `MSW` + `Faker`，其他方案理论上类似，**不同点**我会在文中提到。

我已将 demo 代码上传至 [github](https://github.com/codexu/msw-mock-demo)。

### 初始化项目

```
npm create vite@latest
```

选择 vue，typescript（随意，这里主要是演示 MSW 和 Faker 对 ts 的支持）。

```
cd my-mock-app
npm install
```

这里不做多讲，大家轻车熟路。

### 安装依赖

```
npm install msw @faker-js/faker --save-dev
npm install axios --save
```

安装 axios 方便对比测试 Ajax 和 Fetch 的请求。

### 定义接口

在 `src/mocks` 目录下创建一个 `handlers.ts` 文件，它以后可以在浏览器或 Node 环境中复用。

```
mkdir src/mocks
touch src/mocks/handlers.ts
```

然后选择使用的请求类型，Rest API 或 GraphQL API。

> GraphQL 绝大多数团队都没有使用，这里以 Rest API 为主讲述。

我们先定义一个简单的 get 请求：

```ts
import { rest } from 'msw';

export const handlers = [
  rest.get('/api/user', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.delay(1000),
      ctx.json({ username: 'codexu' })
    );
  }),
];
```

上述代码将模拟请求一个**状态码**为 200 的请求，返回 **json** 数据，并且**延迟** 1000 ms。

### 集成到项目

集成也分为两种，就是前文提到的 `Browser` 和 `Node` 环境，这里我们先搞定浏览器。

在 `src/mocks` 中创建一个 `browser.js` 文件，在其中配置 Service Worker。

```
import { setupWorker } from 'msw';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);
```

随后需要在 `public` 文件夹中生成一个 Service Worker 脚本，然而我们不必自己编写任何 worker 的代码，直接使用 MSW CLI 提供的功能即可。

```
npx msw init <PUBLIC_DIR> --save
npx msw init public/ --save
```

⚠️ 注意这里的 `<PUBLIC_DIR>`，不同的环境下这个是有一些区别的。

可以参考以下下面的表格：

| 项目 | PUBLIC_DIR |
| -- | -- |
| [Create React App](https://create-react-app.dev/) | `./public` |
| [GatsbyJS](https://www.gatsbyjs.org/)             | `./static` |
| [NextJS](https://nextjs.org/)                     | `./public` |
| [VueJS](https://vuejs.org/)                       | `./public` |
| [Angular](https://angular.io/)                    | `./src` |
| [Preact](https://preactjs.com/)                   | `./src/static` |
| [Ember.js](https://emberjs.com/)                  | `./public` |
| [Svelte.js](https://svelte.dev/)                  | `./public` |
| [SvelteKit](https://kit.svelte.dev/)              | `./static` |
| [Vite](https://vitejs.dev/)                       | `./public` |

> PUBLIC_DIR 通常是浏览器访问页面时的根目录，即你打包时的 ./build、./public 或 ./dist 等。

在 main.ts 中加入：

```ts
if (import.meta.env.MODE === 'development') {
  const { worker } = await import("./mocks/browser");
  worker.start()
}
```

这段代码是在开发环境中动态加载。

这时运行 npm run dev，在页面中调用一下：

```ts
import axios from 'axios';
axios.get('/api/user');
```

可以看到，这个请求耗时1秒后，请求成功，并返回了预设的 username。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7713e45f4a854660af4add1f0e4b1b0a~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=815&h=273&s=24377&e=png&b=fefefe)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d85332f58f24aa5b9ba9f33a3a5db70~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=703&h=233&s=14825&e=png&b=fefefe)

当然使用 fetch 也是可以的：

```ts
fetch('/api/user')
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80f6307b87474d1a87ae0e036643d05f~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=864&h=82&s=8531&e=png&b=f0f4f7)

到这步已经完成了项目配置 mock 的环境，是不是很简单？

接下来我们让 mock 功能变得更加强大。

## 伪造“真实”数据

用简单的字符串或者随机数组创建的测试数据是无法满足测试的需求的，使用 Faker 可以生成看起来比较“真实”的测试数据。

使用 ts 开发，有非常方便的代码提示，几乎不用翻文档就可以找到需要伪造的数据。👇

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7253feb6d8944f89a68ad19d57e5a478~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=788&h=467&s=32601&e=png&b=262b32)

这里可以注意一下，faker 是支持多国语的，在 import 时可以选择你需要的语言，例如这里使用中文：

```ts
import { faker } from "@faker-js/faker/locale/zh_CN";
```

然后改造一下之前的接口，让其返回一个数组，里面丰富一些字段：

```ts
rest.get('/api/user', (req, res, ctx) => {
  return res(
    ctx.status(200), 
    ctx.delay(1000),
    ctx.json(
      Array.from({ length: 10 }).map(() => ({
        fullname: faker.person.fullName(),
        email: faker.internet.email(),
        avatar: faker.image.avatar(),
        address: faker.location.streetAddress(),
      }))
    )
  );
}),
```

保存之后，浏览器会自动刷新，再次请求了接口，这次返回的数据可以看到已经变化成👇。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1df12636fed64f07a366d211e8995879~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=772&h=316&s=44343&e=png&b=fefdfd)

可以注意的是，头像字段返回的链接是真实的图片哦。

## 深入学习

> **省流：前文已经可以满足 90% 的开发需求，本章节可以斟酌参考。**

### Methods

rest 支持所有的请求方式：

- `rest.get()`
- `rest.post()`
- `rest.put()`
- `rest.patch()`
- `rest.delete()`
- `rest.options()`

并且还提供了一个 `rest.all()` 方法用于拦截任何请求。

### Request

前文例子中，调用 rest 方法时，可以看到 Response resolver 中具有三个参数：

```ts
rest.get('/api/user', (req, res, ctx) => {});
```

如果了解过 express 的同学应该就很熟悉了，语法是一样的，它可以帮助我们获取到一些请求参数，这样模拟的请求就更加真实了。

值得一提的是 `req.passthrough()`，它可以使你的请求发送到服务器，加入判断即可实现**条件响应**：

```ts
import { rest } from 'msw'

export const handlers = [
  rest.post('/api/user', (req, res, ctx) => 
    if (req.url.searchParams.has('id')) {
      return req.passthrough()
    }
    return res(ctx.text('codexu'))
  })
]
```

当然，它还提供了很多其他功能，有兴趣可以[参考文档](https://mswjs.io/docs/api/request)。

### Response

[Response](https://mswjs.io/docs/api/response) 则是可以处理接口返回，例如延时、状态码和数据处理等。

另外值得一提的时 `res.once` 代表着只能请求一次。

```ts
rest.get('/api/user', (req, res, ctx) => {
  return res.once(ctx.json({ username: 'codexu' }))
})
```

第一次调用 `fetch('/api/user')`，可以正常返回请求内容。

第二次调用 `fetch('/api/user')`, 则会提示 `ERR: Network error! (no such endpoint)`。

### Context

[Context](https://mswjs.io/docs/api/context) 常用的就是：

- `status`: 用于处理状态码。
- `delay`: 用于延时，模拟真实的请求速度。
- `json`: 用于返回 json 数据。

除此之外，还提供了：

- [set](https://mswjs.io/docs/api/context/set)：用于设置 headers。
- [body](https://mswjs.io/docs/api/context/body)：在 body 返回，不参与 Content-Type 转换，例如二进制数据。
- [text](https://mswjs.io/docs/api/context/text)：Content-Type: text/plain，返回纯文本。
- [xml](https://mswjs.io/docs/api/context/xml)：Content-Type: text/xml，返回 XML 数据。
- [data](https://mswjs.io/docs/api/context/data)：针对 GraphQL 操作的响应。
- [errors](https://mswjs.io/docs/api/context/errors)：针对 GraphQL 的错误返回。
- [fetch](https://mswjs.io/docs/api/context/fetch)：可以理解为在拦截请求时，发送一个真实的请求，用于修正返回结果时使用。

### 用于测试

MSW 可以直接复用 handlers，用于测试。

在 `src/mocks/` 目录下创建 `server.js`：

```ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

由于各种测试框架用法不同，这里就不再过多描述，我想大家也很少会使用到测试的功能。

## 总结

文章前半段讲述了现在主流的 mock 方案，进行了优缺点的对比，并选出了两种最为实用的方案。我选择了请求拦截的方式，但没选择 mock.js 作为最终方案，而是选择了 MSW + Faker，因为这个组合方案更加强大。

**技术选型不分好坏，应根据不同的场景选择更合适方案。**

推荐大家尝试一下我的方案，也许它更适合你呢？

## 参考

- [Github Demo 源码](https://github.com/codexu/msw-mock-demo)
- [Mock Service Worker 官网](https://mswjs.io/)
- [Service Worker API 文档](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [Faker.js 官网](https://fakerjs.dev/)
- [Mock.js 文档](https://github.com/nuysoft/Mock/wiki/Getting-Started)