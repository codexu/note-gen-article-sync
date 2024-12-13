# 零 rust 基础前端使直接上手 tauri 开发一个小工具

## 起因

有一天老爸找我，他们公司每年都要在线看视频学习，要花费很多时间，问我有没有办法可以自动学习。

在这之前，我还给我老婆写了个浏览器插件，解决了她的在线学习问题，她学习的是一个叫好医生的学习网站，我通过研究网站的接口和代码，帮她开发出了一键学习全部课程和自动考试的插件，原本需要十来天的学习时间，分分钟就解决了。

有兴趣的可以看一下，[好医生自动学习+考试插件源码](https://github.com/codexu/cmechina-chrome-plugin)。

正因为这次的经历，我直接接下了这个需求，毕竟可以在家人面前利用自己的能力去帮他们解决问题，是一件非常骄傲的事。

**事情并没有那么简单**

我回家一看，他们的学习平台是个桌面端的软件（毕竟是银行的平台，做的比那个好医生严谨的多），内嵌的浏览器，无法打开控制台，更没办法装插件，甚至视频学习调了什么接口，有什么漏洞都无法发现，我感觉有点无能为力。

但是牛逼吹出去了，也得想办法做。

## 技术选型

既然没办法找系统漏洞去快速学习，那只能按部就班的去听课了，我第一想到的方式是用按键精灵写个脚本，去自动点击就可以了。但是我爸又想给他的同事用，再教他们用按键精灵还是有点上手成本的，所以我打算自己开发一个小工具去实现。

由于我是个前端开发者，做桌面端首先想到的是 Electron，因为我有一些开发经验，所以并不难，但打包后的体积太大，本来一个小工具，做这么大，这不是显得我技术太烂嘛。

所以我选择了 tauri 去开发。

## 需求分析

首先我想到的方式就是：

1.  用鼠标框选一个区域，然后记录这个区域的颜色信息，记录区域坐标。
2.  不断循环识别这个区域，匹配颜色。
3.  如果匹配到颜色，则点击这个区域。

例如，本节课程学习后，会弹出提示框，进入下一节学习，那么可以识别这个按钮，如果屏幕出现这个按钮，则点击，从而实现自动学习的目的。

我还给它起了个很形象的名字，叫做打地鼠。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69bc07f4fda3476986d449aeef931857~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=720&h=1200&s=76565&e=png&a=1&b=fdfdfd)

由于要点击的不一定只有一个下一节，可能还有其他章节的可能要学习，所以还实现了多任务执行，这样可以识别多个位置。

有兴趣可以看一下[源码](https://github.com/codexu/whack-moles)。

## 零基础入门 rust

Tauri 已经提供了很多可以在前端调用的接口去实现很多桌面端的功能，但也不能完全能满足我本次开发的需求，所以还是要学习一点 rust 的语法。

这里简单说一下我学到的一些简单语法，方便大家快速入门。由于功能简单，我们并不需要了解 rust 那些高深的内容，了解基础语法即可，不然想学会 rust 我觉得真心很难。我们完全可以先入门，再深入。

### 适合人群

有一定其他编程语言(C/Java/Go/Python/JavaScript/Typescript/Dart等)基础。你至少得会写点代码是吧。

### 环境安装

推荐使用 rustup 安装 rust，rustup 是官方提供的的安装工具。

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

安装后，检查版本，这类似安装 node 后查看版本去验证是否成功安装。

```sh
rustc --version

>>> rustc 1.73.0 (cc66ad468 2023-10-03)
```

cargo 是 rust 官方的包管理工具，类似 npm，这里也校验一下是否成功安装。

```sh
cargo --version
```

> 如果提示不存在指令，重新打开终端再尝试。

### 编辑器

官方推荐 [Clion](https://www.jetbrains.com/zh-cn/clion/)，是开发 rust 首选开发工具。

不过作为前端，我们依然希望可以使用 vscode 去开发，当然，这也是没有问题的。

vscode 需要搭配 rust-analyzer 一起使用。

除了上面提到的两个命令，还有 `rustup` 命令也可以直接使用了：

```sh
rustup component add rust-analyzer
```

执行后就都配置好了，可以进行语法的学习了。

## 变量与常量的声明

定义变量和常量的声明 javascript 和 rust 是一样的，都是通过 let 和 const，但是在定义变量时还是有一些区别的：

- 默认情况下，变量是不可变的。（这点对于前端同学来说是不是很奇怪？）
- 如果你想定义一个可变的变量，需要在变量名前面加上 `mut`。

```rs
let x = 1;
x = 2; ❌
```

```rs
let mut x = 1;
x = 2; ✅
```

如果你不想用 mut，你也可以使用相同的名称声明新的变量：

```rs
let x = 1;
let x = x + 1;
```

Rust 里常量的命名规范是使用全大写字母，每个单词之间使用下划线分开，虽然 JS 没有强制的规范，但是我们也是这么做的。

### 数据类型

对于只了解 javascript 的同学，这个是非常重要的一环，因为 rust 需要在定义变量时做出类型的定义。即使是有过 typescript 开发经验的同学，这里也有着非常大的区别。这里只说一些与 js 区别较大的地方。

#### 数字

首先 ts 对于数字的类型都是统一的 number，但是 rust 区别就比较大了，分为有符号整型，无符号整型，浮点型。

有 i8、i16、i32、i64、i128、u8、u16、u32、u64、isize、usize、f32、f64。

虽然上面看起来有这么多种类型去定义一个数字类型，实际上它们只是去定义了这个值所占用的空间，新手其实不用太过于纠结这里。如果你不知道应该选择哪种类型，直接使用默认的`i32`即可，速度也很快。有符号就是分正负(+,-)，无符号只有正数。浮点型在现代计算机里上 f64 和 f32 运行速度差不多，f64 更加精确，所以不用太纠结。

#### 数组

数组定义也有很大区别，你需要一开始就定义好数组的长度：

```rs
let a: [i32; 5] = [0; 5];
```

这表示定义一个包含 5 个元素的数组，所有元素都初始化为 0。一旦定义，数组的大小就不能改变了。

这是不是让前端同学很难理解，那么如何定义一个可变的数组呢？这好像更符合前端的思维。

在 Rust 中，Vec 是一个动态数组，也就是说，它可以在运行时增加或减少元素。

```rs
let v: Vec<i32> = Vec::new();
v.push(4);
```

这是不是更符合前端的直觉？毕竟后面我们要使用鼠标框选一个范围的颜色，这个颜色数组是不固定的，所以要用到 `Vec`。

数据类型就说到这，其他的有兴趣自行了解即可。

### 引用包

rust 同 javascript 一样，也可以引入其他包，但语法上就不太一样了，例如：

```rs
use autopilot::{geometry::Point, screen, mouse};
```

强行翻译成 es module 引入：

```js
import { Point, screen, mouse } from 'autopilot';
```

看到 `::` 是不是有点懵逼，javascript 可没有这样的东西，你可以直觉的把它和 `.` 想象成一样就行。

`::` 主要用于访问模块（module）或类型（type）的成员。例如，你可以使用 :: 来访问模块中的函数或常量，或者访问枚举的成员。

`.` 用于访问结构体（struct）、枚举（enum）或者 trait 对象的实例成员，包括字段（field）和方法（method）。

### 其他语法

循环：

```rs
for i in 0..colors.len() {}
```

条件判断：

```rs
if colors[i] != screen_colors[i] {

}
```

他们就是少了括号，还有一些高级的语法是 ES 没有的，这都很好理解。

那么我说这样就算入门了，不算过分吧？如果你要学一个语言，千万别因为它难而不敢上手，你直接上手去做，遇坑就填，你会进步很快。

如果你觉得这样很难写代码，那么我建议你买个 copilot 或者平替通义灵码，你上手写点小东西应该就不成问题了，毕竟我就这样就开始做了。

## 软件开发

[Tauri 官网](https://tauri.app/zh-cn/)翻译还不全，读起来可能有点吃力，借助翻译工具将就着看吧，我有心帮大家翻译，但是提了 pr，好几天也没人审核。

你可以把 tauri 当作前端和后端不分离的项目，webview 就是前端，rust 写后端。

### 创建项目

tauri 提供了很多方式去帮你创建一个新的项目：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cccf9e5c3a1f49f0878a5017f4be9630~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1968&h=728&s=143006&e=png&b=ecf5e8)

这里初始化一个 vite + vue + ts 的项目：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9e28418ed5e406b9859471d93235382~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1736&h=578&s=131254&e=png&b=353744)

最后的目录结构可以看一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbe8e4c5f24d488282e99349ac5ff180~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=902&h=666&s=68079&e=png&b=22252a)

`src` 就是前端的目录。

`src-tauri` 就是后端的目录。

### 前端

前端是老本行，不想说太多的东西，大家都很熟悉，把页面写出来就可以了。

值得一提的就是 tauri 提供的一些接口，这些接口可以让我们实现一些浏览器上无法实现的功能。

#### 与后端通讯

```ts
import { invoke } from "@tauri-apps/api";

invoke('event_name', payload)
```

通过 `invoke` 可以调用 rust 方法，并通过 payload 去传递参数。

#### 窗口间传递信息

这里的窗口指的是软件的窗口，不是浏览器的标签页。由于我们要框选一块显示器上的区域，所以要创建一个新的窗口去实现，而选择后要将数据传递给主窗口。

```ts
import { listen } from '@tauri-apps/api/event';

listen<{ index: number}>("location", async (event) => {
  const index = event.payload.index;
  // ...
})
```

#### 获取窗口实例

例如隐藏当前窗口的操作：

```ts
import { getCurrent } from '@tauri-apps/api/window';

const win = getCurrent()
win.hide() // 显示窗口即 win.show()
```

与之相似的还有：

- `appWindow` 获取主窗口实例。
- `getAll` 获取所有窗口实例，可以通过 `label` 来区分窗口。

最主要的是 `WebviewWindow`，可以通过他去创建一个新的窗口。

```ts
const screenshot = new WebviewWindow("screenshot", {
  title: "screenshot",
  decorations: false,
  // 对应 views/screenshot.vue
  url: `/#/screenshot?index=${props.index}`,
  alwaysOnTop: true,
  transparent: true,
  hiddenTitle: true,
  maximized: true,
  visible: false,
  resizable: false,
  skipTaskbar: false,
})
```

这里我们创建了一个最大化、透明的窗口，且它位于屏幕最上方，页面指向就是 vue-router 的路由，index 是因为我们不确定要创建多少个窗口，用于区分。

可以通过创建这样的透明窗口，然后实现一个框选区域的功能，这对于前端来说，并不难。

例如鼠标点击左键，滑动鼠标，再松开左键，绘制这个矩形，再加一个按钮。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92ed5fce2ea1403ba07f6b1898959985~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=348&h=270&s=17337&e=png&b=f7f7f7)

随后将位置信息传递给主窗口，并关闭这个透明窗口。

### 后端

首先，`src-tauri/src/main.rs` 是已经创建好的入口文件，里面已有一些内容，不用都了解。

#### 暴露给前端的方法

```rs
tauri::Builder::default().invoke_handler(tauri::generate_handler![scan_once, ...])
```

通过 `invoke_handler` 可以暴露给前端 `invoke` 调用的方法。

> `!` 在 rust 中是指宏调用，主要是方便，并不是 javascript 里的非的含义，这里注意下。

#### 获取屏幕颜色

这里为了性能，我只获取了 x 起始位置到 x 结束位置，y 轴取中间一行的颜色。

```rs
use autopilot::{geometry::Point, screen};

pub fn scan_colors(start_x: f64, end_x: f64, y: f64) -> Vec<[u8; 3]> {
    // 双重循环，根据 start_x, end_x, y 定义坐标数组
    let mut points: Vec<Point> = Vec::new();
    let mut x = start_x;
    while x < end_x {
        points.push(Point::new(x, y));
        x += 1.0;
    }
    // 循环获取坐标数组的颜色
    let mut colors: Vec<[u8; 3]> = Vec::new();
    for point in points {
        let pixel = screen::get_color(point).unwrap();
        colors.push([pixel[0], pixel[1], pixel[2]]);
    }
    return colors;
}
```

这样就获取到一组颜色数组，包含了 RGB 信息。

这里安装了一个叫 `autopilot` 的包，可以通过 `cargo add autopilot` 安装，他可以获取屏幕的颜色，也可以操作鼠标。

#### 鼠标操作

使用 autopilot::mouse 可以进行鼠标操作，移动至 x、y 坐标、病点击鼠标左键。

```rs
use autopilot::{geometry::Point, mouse};

mouse::move_to(Point::new(x, y));
mouse::click(mouse::Button::Left, None);
```

#### 配置权限

在 `src-tauri/tauri.conf.json` 中配置 allowlist，如果不想了解都有哪些权限，直接 `all: true`，全部配上，以后再慢慢了解。

```json
"tauri": {
    "macOSPrivateApi": true,
    "allowlist": {
      "all": true,
    },
}
```

注意 mac 上如果使用透明窗口，还需要配置 macOSPrivateApi。

整体流程就是这样的，其他都是细节处理，有兴趣可以看下源码。

## 构建

我爸的电脑是 windows，而我的是 mac，所以需要构建一个 windows 安装包，但是 tauri 依赖本机库和开链，所以想跨平台编译是不可能的，最好的方法就是托管在 `GitHub Actions` 这种 CI/CD 平台去做。

在项目下创建 `.github/workflows/release.yml`，它将会在你发布 `tag` 时触发构建。

```yml
name: Release

on:
  push:
    tags:
      - 'v*'
  workflow_dispatch:

concurrency:
  group: release-${{ github.ref }}
  cancel-in-progress: true

jobs:
  publish:
    strategy:
      fail-fast: false
      matrix:
        platform: [macos-latest, windows-latest]

    runs-on: ${{ matrix.platform }}
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: 8
          run_install: true

      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - name: install Rust stable
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable

      - name: Build Vite + Tauri
        run: pnpm build

      - name: Create release
        uses: tauri-apps/tauri-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tagName: v__VERSION__ # the action automatically replaces \_\_VERSION\_\_ with the app version
          releaseName: 'v__VERSION__'
          releaseBody: 'See the assets to download and install this version.'
          releaseDraft: true
          prerelease: false
```

这里提供一个实例，具体情况具体修改。

`secrets.GITHUB_TOKEN` 并不需要你配置，他是自动获取的，主要是获得权限去操作你的仓库。因为构建完成会自动创建 release，并上传安装包。

你还需要修改一下仓库的配置：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcb2542238bc42059894741a05fa8872~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=692&h=1084&s=79716&e=png&b=24282e)

选中 Read and write permissions，勾选 
Allow GitHub Actions to create and approve pull requests。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b3d9168652c4e9ca497867da8346789~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1588&h=636&s=155276&e=png&b=24282e)

当你发布 tag 后，会触发 action 执行。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9dedfc1bdae845308ce59ed549719b91~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2054&h=870&s=135586&e=png&b=23282e)

可见，打包速度真的很慢。

Actions 执行完毕后，进入 Releases 页面，可以看到安装包已经发布。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46e41fe418944f118526d9ee0b5a980d~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2302&h=1006&s=169934&e=png&b=23272d)

## 总结

- 关于 `tauri` 和 `electron，甚至是` `flutter`、`qt` 这种技术方向没必要讨论谁好谁坏，主要还是考虑项目的痛点，去选择适合自己的方式，没必要捧高踩低。
- `Rust` 真的很难学，我上文草草几句入门，其实并没有那么简单，刚上手会踩很多坑，甚至无从下手不会写代码。我主要的目的是希望大家有想法就要着手去做，毕竟站在岸上学不会游泳。`Flutter` 使用 `dart`，我曾经写写过两个 app，相比于 `rust`，`dart` 对于前端同学来说可以更轻松的学习。
- `Tauri` 我目前还是比较看好，也很看好 `rust`，大家有时间的话还是值得学习一下，尤其是 2.0 版本还支持了移动端。
- 看到很多同学，在学习一门语言或技术时，总是不知道做什么，不只是工作，其实我们身边有很多事情都可以去做，可能只是你想不到。我平时真的是喜欢利用代码去搞一些奇奇怪怪的事，例如我写过 `vscode` 摸鱼插件、自动学习视频的 `chrome` 插件、互赞平台、小电影爬虫等等，这些都是用 javascript 就实现的。你可以做的很多，给自己提一个需求，然后不要怕踩坑，踩坑的过程是你进步最快的过程，享受它。