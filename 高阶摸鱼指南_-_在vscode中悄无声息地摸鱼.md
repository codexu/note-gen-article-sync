# 高阶摸鱼指南 - 在vscode中悄无声息地摸鱼

## 想法

作为前端开发者，大多数人都使用 VSCode，并且可能会找一些在 VSCode 中可以摸鱼的插件。我也尝试了一些：

- [Zhihu On VSCode](https://marketplace.visualstudio.com/items?itemName=niudai.vscode-zhihu)，知乎摸鱼。
- [daily anime](https://marketplace.visualstudio.com/items?itemName=deepred.daily-anime)，追番插件。
- [韭菜盒子](https://marketplace.visualstudio.com/items?itemName=giscafer.leek-fund)，看股票、基金、期货实时数据。
- [电影集](https://marketplace.visualstudio.com/items?itemName=axetroy.vscode-movie)，看热映电影。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8cd0b44759db434e94c9eefc46e3b2c6~tplv-k3u1fbpfcp-watermark.image?)

除此之外，还有许多其他的插件，比如听音乐、看小说、刷力扣、浏览掘金等等。甚至有人开发了一个类似小霸王游戏机的插件。

![未标题-1.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca04cfd3022c469389beb7f77d2d14ac~tplv-k3u1fbpfcp-watermark.image?)

VSCode本身是基于 **Node.js** 和**浏览器**的应用程序，因此，熟悉前端开发的同学可以想到，在 VSCode 中可以实现几乎所有 Node.js 和浏览器能做的事情。上面提到的摸鱼插件有着很好的想法，功能和代码也做得不错，但它们忽略了一个重要的点：

> **摸鱼需要保持隐蔽性**

这些摸鱼插件占用了大量的屏幕空间，有些甚至将整个屏幕占满。既然如此，为什么不直接打开浏览器，直接摸鱼呢？

## 最好的摸鱼插件是什么？

我认为扩展商店中的这些摸鱼插件，整活已经大于摸鱼，我更需要的是真正的安全的摸鱼方案，那么一个完美的摸鱼插件究竟需要哪些特点呢？

- **绝对的隐蔽性**，不能让同事和老板一眼就察觉到你在摸鱼。
- **专属的阅读内容**，每个人喜欢的平台和内容都不同。例如，我喜欢浏览头条和知乎，而其他人可能更喜欢浏览掘金或小红书，只有喜欢的内容才能让摸鱼更加舒适。
- **丝滑的操控**，在保持隐蔽的同时，能够在别人专注查看你屏幕内容的时候，瞬间隐藏摸鱼的内容。这绝对不能只依赖于快捷键，因为如果你突然按下键盘，就会引起巨大的摸鱼嫌疑。

然而，是否存在这样一款插件能够满足上述3个要求呢？

答案是：**并没有！**

那么我们该怎么办呢？那就是自己开发插件。

## 我的专用摸鱼插件

我个人非常喜欢浏览头条，无论是在马桶上、吃饭时、躺在床上甚至是在梦中，我都会刷头条。只有在写代码的时候无法刷到。因此，为了满足我的一边摸鱼一边写代码的需求，我开发了一款专用的摸鱼插件。在这里，我将介绍如何使用这款插件，并分享开发思路。我并不是在推销我的插件，而是希望给大家提供一些开发思路。如果你感兴趣，也可以尝试自己开发一款适合自己的摸鱼插件，因为最适合自己的才是最好的。

![动画.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6476d6ba303f4c87b7930646911753d1~tplv-k3u1fbpfcp-watermark.image?)

### 前置知识

- `VSCode` 扩展相关接口，开发相关界面和操作。
- `puppeteer` 用于爬取头条内容。

### 使用方式

平时还是以工作为主，插件基于 `puppeteer`，比较**吃内存**，所以启动插件的方式不应该是默认启动，我采用的方式是：通过 `ctrl+shift+p` 打开命令面板，输入开始摸鱼，用于激活插件，此时右下角状态栏([Status Bar](https://code.visualstudio.com/api/references/vscode-api#StatusBarItem))激活，并爬取了一些内容，通过左右切换按钮切换内容，最后一个按钮点击，弹出一个新窗口([Webview](https://code.visualstudio.com/api/extension-guides/webview))，可以扫码登录，这样爬取的内容就是平台推荐我的内容。

![F2EE2CA0-5EB0-4b89-8817-E3957A118C40.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3eab82796924222834789e73fce5199~tplv-k3u1fbpfcp-watermark.image?)

如果环境比较安全，可以点击状态栏标题，可弹出快速选择框，展示内容列表（[QuickPick](https://code.visualstudio.com/api/references/vscode-api#QuickPick)），可以快速找到想看的内容：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58d410cdaf1d459698ece6ea0c5cfae4~tplv-k3u1fbpfcp-watermark.image?)

内容选择后，如何才能悄无声息的看到内容和评论呢？

我这里采用了一种极为隐蔽并且操作方便的方式，那就是将鼠标移动到某个单词上，然后弹出展示框([Document selectors](https://code.visualstudio.com/api/references/document-selector#document-selectors))：

- 文章内容

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed5a7ad2042347e58f65d05fca87c9c4~tplv-k3u1fbpfcp-watermark.image?)

- 评论内容

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d381ea39a1d54829a2ee32799d105919~tplv-k3u1fbpfcp-watermark.image?)

情况危险的时候只要将鼠标轻轻挪开，就可以隐藏掉，这样的操作方式比老板键隐蔽的多。

## 入门 VSCode 插件开发

上面提到了很多内容，主要是为了激发大家的学习兴趣。在学习新知识时，很多同学不知道如何将所学知识应用到实际项目中。但是，如果我们能以兴趣作为学习的动力，并设定一个项目目标，就能够坚持下去并持续学习。

这样的学习方式更加有效和有趣，下面开始进入正题：

### 了解 VSCode 都可以做哪些功能

在学习一项新的技能时，首先要做的就是以下几点：

- 它能做什么？
- 我能用它做什么？
- 值不值得去学习？

接下来让我们带着问题去了解 VSCode 插件开发。

#### 它能做什么？

VSCode 几乎支持在它内部任何位置的扩展，这里先看一下基础界面上可操作的区域：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6c7cc4896ca4328bf895c7373a40d91~tplv-k3u1fbpfcp-watermark.image?)

- [Web View](https://code.visualstudio.com/api/extension-guides/webview)，相当浏览器打开一个网页，这块我们后续可以用于做扫码登录。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f273697f83a943c5b9681f6227a2aeb5~tplv-k3u1fbpfcp-watermark.image?)

- [Status Bar](https://code.visualstudio.com/api/ux-guidelines/status-bar) 状态栏位于底部，可以做为显示信息和操作，分为主要（左）和次要（右）。

还有一些是通过事件触发才可以展示到界面上的 UI，例如：

- [Quick Pick](https://code.visualstudio.com/api/references/vscode-api#QuickPick)，让用户从列表中选择其中一个。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/368d579d09fd451f854ba10124cac529~tplv-k3u1fbpfcp-watermark.image?)

- [Display Notifications](https://code.visualstudio.com/api/references/vscode-api#window.showInformationMessage) 提供了三个 API 用于显示不同严重程度的通知消息。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b76dd3a5d4464beda1aef4329b8cc57c~tplv-k3u1fbpfcp-watermark.image?)

- [Show Hovers](https://code.visualstudio.com/api/language-extensions/programmatic-language-features#show-hovers) 鼠标悬停到匹配的信息后，展示内容。

![hovers.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d033abf4fab49b480b507efd9b086fc~tplv-k3u1fbpfcp-watermark.image?)

VSCode 还提供了超多的接口，这里就不一一举例，以上的内容已经足够强大，完全可以支撑摸鱼插件的开发。更多的扩展接口请查看官方文档，写的很详细。

#### 我能用它做什么？

上文介绍的摸鱼插件就是首当其冲的可以做的，虽然摸鱼这种事情看起来很 low，但是觉得是一个好的入门方式，毕竟兴趣是最好的老师，我想没人不爱摸鱼吧。

如果觉得摸鱼插件很 low，你可以发动你的小脑筋做点有用的插件。比如我英文不好，我自己做了一款翻译插件，我经常在变量命名时使用，它的基本使用方式是，写一个中文名，然后选择，翻译，凭借3年级的英文水平，选一个大概合适的单词，然后在对其做格式转换（大驼峰、小驼峰、下划线等等）。这里我也不贴出插件名了，毕竟我是自用的。

#### 值不值得去学习？

总的来说，我认为学习 VSCode 插件开发是非常值得的。首先，学习成本并不高，因为官方文档写得非常详细。其次，由于 VSCode 的使用占比极高，因此如果您能够开发出一款优秀的插件，那么一定会有很多用户使用它，这会给您带来很大的成就感。

### 创建插件开发项目

最好的学习方式就是看[官方文档](https://code.visualstudio.com/api)，本文简单讲解，带大家快速入门。

官方提供了基于 yo 的脚手架，帮助我们快速生成插件开发的基础环境，安装 yo 和 generator-code：

```
npm install -g yo generator-code
```

然后执行

```
yo code
```

第一个问题

> What type of extension do you want to create?

选择 `New Extension`，ts 或 js 都可以，其他选项按照问题随意填写。

创建好项目后，可以按 `F5` 调试插件，按 `Ctrl+Shift+P` 输入 **Hello World**，如果看到出现 `Hello World from HelloWorld!` 通知，则插件运行成功。

### 目录结构

```
.
├── .vscode
│   ├── launch.json
│   └── tasks.json
├── .gitignore
├── README.md
├── src
│   └── extension.ts    // 插件的入口文件
├── package.json        // 插件的配置文件
├── tsconfig.json
├── webpack.config.js
├── ...
```

创建的目录结构非常简单，我们只需要关注 `src/extension.ts` 和 `package.json` 即可。

extension.ts 文件中 export 两个方法： `activate` 和 `deactivate`，表示激活和销毁时执行的方法，通常我们将核心代码都写在 activate 函数中。

```
import * as vscode from 'vscode';
export function activate(context: vscode.ExtensionContext) {
}
export function deactivate() {}
```

package.json 中有几个特殊的字段需要注意：

- engines: 最低支持的vscode版本
- activationEvents: 用来定义插件在什么时候被激活
- contributes: 最重要的配置，几乎插件所有的配置都在这里，内容较多，请查看[文档](https://code.visualstudio.com/api/references/contribution-points)

看到这里基本就可以说是入门了，剩下的内容直接从实战开始，去了解更多的 API 使用方式。

## 实战

本章节通过对常用 API 的核心属性方法讲解，结合实际操作带大家了解如何开发插件。

### 配置

首先打开 package.json，在 `contributes` 属性中添加 `commands` 数组，参数中包含：

- command: 指令，可以在代码中去调用。
- title：名称，例如在 ctrl+shift+p 输入标题搜索，然后确定可执行这条指令。

这里注册一条初始化摸鱼插件和结束摸鱼的指令：

```json
"contributes": {
  "commands": [
    {
      "command": "FishX.init",
      "title": "开始摸鱼"
    },
    {
      "command": "FishX.dispose",
      "title": "结束摸鱼"
    }
  ]
},
```

然后注册两个快捷键指令， 用于切换内容，与 `commands` 一样，需要在 `contributes` 属性中添加 `keybindings` 数组，它同样具有 command 属性，不同的是：

- key: 快捷键，组合快捷键用 `+` 连接。

```json
"keybindings":[
  {
    "command": "FishX.next",
    "key": "alt+d"
  },
  {
    "command": "FishX.prev",
    "key": "alt+a"
  }
]
```

其他还可以配置一些插件的基础信息：

- displayName: 插件名称，这个是在扩展搜索时，展示的名称。
- description: 插件简介，搜索时同样展示。
- publisher: 发布者，这块与你后续发布时有关。
- icon: 图标，直接扔到根目录即可。

### 爬虫

Puppeteer 在这里作为爬虫使用，用于爬取摸鱼的内容，这是个很有趣的库，这块不做多讲，大家可以自行了解。

参考文档：http://www.puppeteerjs.com/

或者有兴趣可以看一下我的另一篇文章[《使用前端技术破解掘金滑块验证码》快速上手 puppeteer 章节](https://juejin.cn/post/7257386139849801789#heading-1)，也可以帮你快速入门。

### 注册指令

通过 [vscode.commands.registerCommand](https://code.visualstudio.com/api/references/vscode-api#commands.registerCommand) 可以将命令 ID 绑定到执行函数。

前文在 package.json 中注册了 FishX.init 命令，这里我们使用 registerCommand 函数将此命令绑定相应的执行函数：

```ts
import * as vscode from 'vscode';

export async function activate(context: vscode.ExtensionContext) {

    const initDisposable = vscode.commands.registerCommand(
      "FishX.init",
      async () => {
          // 需要执行的代码
      }
    )

    context.subscriptions.push(
        initDisposable,
    );
}
```

registerCommand 执行后返回一个 [Disposable](https://code.visualstudio.com/api/references/vscode-api#Disposable) 对象，使用 context.subscriptions.push 将它添加到当前插件的上下文中，以便在插件被注销时自动释放这些资源。

> context.subscriptions.push() 可以将一个或多个 `Disposable` 对象添加到当前插件的上下文中。

### 状态栏

VSCode 状态栏是一个非常重要的区域，大多数插件和一些常用配置在这里进行操作。

它具备操作便捷和屏占比低，并且不处于屏幕的核心区域，如果别人在看你的屏幕时，这个位置大概率不会被专注去看，这也是我把它作为插件操作的入口处的原因。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/259bf9a1134e44e4a638c49912eded66~tplv-k3u1fbpfcp-watermark.image?)

官方示例：https://github.com/microsoft/vscode-extension-samples/tree/main/statusbar-sample

#### 功能

状态栏提供了一些简单的功能，空间比较紧张，所以要谨慎使用：

- 支持字符串，由于空间问题，尽量使用短文本。
- 支持图标，尽量在必要的情况下使用，例如制作切换内容和登录按钮。
- 具备点击事件的功能。
- 三种状态：进度、失败、警告。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59cbc566d12a4bd8a77d5fd60e6fe976~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/124ec848b2764cca95b0320566743d61~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5900f7057a104a71a327cb19a4957f94~tplv-k3u1fbpfcp-watermark.image?)

> 提示：VSCode 提供了内置图标，状态尽量不使用自定义颜色。

#### API 详解

StatusBar 是 VSCode 提供的整个状态栏，我们要做的是将功能写在 [StatusBarItem](https://code.visualstudio.com/api/references/vscode-api#StatusBarItem) 上，你可以在 StatusBar 添加多个 StatusBarItem。

使用 createStatusBarItem 函数可以创一个 StatusBarItem：

```ts
createStatusBarItem(alignment?: StatusBarAlignment, priority?: number): StatusBarItem
```

参数：

- alignment: 对其方式，参考 [StatusBarAlignment](https://code.visualstudio.com/api/references/vscode-api#StatusBarAlignment)。
- priority：优先级，值越高显示在更靠左侧的位置。

属性：

- text: 展示文字或[图标](https://code.visualstudio.com/api/references/icons-in-labels)，图标使用 `$(***)` 的方式。
- command：点击时触发的指令。
- tooltip：提示文字。
- 更多请参考 https://code.visualstudio.com/api/references/vscode-api#StatusBarItem

方法：

- dispose()：销毁。
- show()：展示，注意创建后调用才会展示。
- hide()：隐藏。

#### 代码

这里创建了 4 个 StatusBarItem，分别对应内容展示、左右切换按钮、登录按钮。

```ts
const statusBarContent = vscode.window.createStatusBarItem(
  vscode.StatusBarAlignment.Right,
  100
);
statusBarContent.text = "$(loading~spin) 鱼塘建造中...";
statusBarContent.command = "FishX.quickPick";
statusBarContent.show();
// 其余照葫芦画瓢...
```

> $(loading~spin) 表示加载中的图标，查看[更多图标](https://code.visualstudio.com/api/references/icons-in-labels)。

### 快速选择

光靠左右切换，很难快速定位到想看的内容，这时候通过上文触发的 `FishX.quickPick` 指令，来弹出内容列表：

![动画.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8309b2d21d3046378e01057af023c589~tplv-k3u1fbpfcp-watermark.image?)

#### 功能

快速选择提供的功能比较简单，就是一个列表，然后可以选择。

#### API 详解

使用 `vscode.window.createQuickPick()` 即可创建一个快速选择功能，调用后返回一个 [QuickPick](https://code.visualstudio.com/api/references/vscode-api#QuickPick) 对象，通过它可以为选择窗口添加内容和绑定事件。

属性：

- items: 可以给他传递任何类型的数组，这是最核心的属性。
- 其他属性请参考文档。

方法：

- onDidChangeSelection：绑定选择事件，其参数就是 items 中选择的那一项。
- show()：展示快速选择。
- hide()：隐藏快速选择。
- 其他方法请参考文档。

#### 代码

```ts
const quickPickDisposable = vscode.commands.registerCommand(
  "FishX.quickPick",
  async () => {
    if (quickPick === undefined) {
      quickPick = vscode.window.createQuickPick();
    }
    quickPick.items = [...];
    quickPick.onDidChangeSelection(async (selection) => {
      // selection 获取到选择的选项
      // 选择后隐藏
      quickPick.hide();
    });
    quickPick.show();
  }
);
```

### 悬停信息

利用鼠标悬浮，可以做到快速展示和隐藏摸鱼内容。

![动画.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55a762db1e6b420798ebb8cd696a3d4a~tplv-k3u1fbpfcp-watermark.image?)

#### 功能

VSCode 提供了 `vscode.languages.registerHoverProvider` 方法，用于为指定的语言注册一个 hover 提示效果，当用户将鼠标悬停在代码中的某个位置时，会触发 hover provider 并显示相关信息。

#### API 详解：

使用 `vscode.languages.registerHoverProvider` 方法可以注册一个 hover provider。

需要传入两个参数：

- selector：一个 string 数组，用于指定要注册的语言。
- provider：方法，用于实现 hover 逻辑。

`provider` 方法需要传入三个参数：

- document：表示当前代码文档。
- position： 表示鼠标所在的位置，配合 document 可以获取到选择的文本。
- token：表示取消操作的 token。

#### 代码

```ts
vscode.languages.registerHoverProvider(
  ["typescript", "javascript", "vue"],
  {
    async provideHover(document, position, token) {
      const range = document.getWordRangeAtPosition(position);
      const word = document.getText(range);
      if (word === "const") {
        return new vscode.Hover('文章...', range);
      }
      if (word === "export") {
        return new vscode.Hover('评论...', range);
      }
    },
  }
);
```

## 总结

实战没有贴出完整代码，用少量的代码带大家快速了解如何实现一个插件的基本功能，如果需要深入了解可以参考：

- 完整代码：[https://github.com/codexu/FishX](https://github.com/codexu/FishX)
- 官方文档：[https://code.visualstudio.com/docs](https://code.visualstudio.com/docs)

另外还有调试和发布本文就不细讲了。

大家如果有什么摸鱼心得和优秀的插件，希望多多在评论区分享。