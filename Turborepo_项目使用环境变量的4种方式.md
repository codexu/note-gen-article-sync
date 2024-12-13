# Turborepo 项目使用环境变量的4种方式

最近在使用 turborepo 开发项目，但是发现它并没有对环境变量做到开箱即用，官方文档写的也是非常简短，只提出了一种方式，就是借助 dotenv 实现全局环境变量，示例代码中也没有提供可以参考的环境变量使用方式。

## 介绍

这里对几个名词作一下简单的介绍，如果对这些有经验可以直接省略本章节。

### 环境变量

环境变量是一种重要的配置工具，它可以在不同的部署环境中动态配置应用程序行为，而无需修改代码。环境变量通常用于存储诸如数据库连接字符串、API地址、端口号、密钥等敏感信息或其他配置参数。

### turborepo

[Turborepo](https://turbo.build/repo) 是一个用于管理具有多个相关项目的 monorepo 代码库，主要通过编排并行、顺序任务，配合缓存减少构建时间的工具。

### dotenv

[Dotenv](https://github.com/motdotla/dotenv) 通常用在 node 项目中，它将环境变量从 .env 文件加载到 process.env 中。

Process 是 node 中的线程容器，每一个 node 应用会有一个 process，可以通过 process 对象控制和获取当前 node 的进程，环境变量就是通过 process.env 获取。

## 实现方式

首先不论什么方式，我们可以先在根目录下创建三个环境变量文件，方便测试。

- .env 通用配置
- .env.development 开发环境
- .env.production 生产环境

当然你也可以创建一些其他环境的专用配置文件，或者创建 `.local` 仅本地使用的配置文件。

### 方式1，框架支持

如果你的应用只使用 `Next.js` 或者 `vite`，那么恭喜你，他们会自动从特定文件加载环境变量，你只需要在 `turbo.json` 中配置一下 `globalDotEnv` 和 `pipeline.**.dotenv` 即可。这里以 `vite` 举例：

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDotEnv": [".env"],
  "pipeline": {
    "build": {
      "dotEnv": [".env.production", ".env"]
    },
    "dev": {
      "dotEnv": [".env.development", ".env"]
    }
  }
}
```

只需要指定你的配置文件即可，其他框架官方并没有提，需要自行测试。

### 方式2，为每个 app 提供自己的 .env

有的项目并不需要全局的环境变量，而且各个 app 也不一定会复用环境变量。例如，一个后端应用加一个前端应用，后端的项目需要数据库账号密码等环境变量，而前端需要的是接口地址的环境变量，那么我们完全可以直接在对应的 app 下提供自己的 .env 文件即可。

这种使用方式就不多说了，大家基本对自己的项目都有一定的使用经验，接入方式大同小异。

### 方式3，创建一个 env 包

类似于全局 eslint、tsconfig 配置 package，我们可以在 packages/ 目录中创建一个 env 包，可以创建一个或多个文件用于存放不同的环境，可以使用 json 或者 js 模块：

```json
// development.json
{
  "BASE_API_URL": "http://localhost:3000"
}
```

在 app 中使用：

```javascript
const developmentEnv = require('env/development.json');
console.log(developmentEnv.BASE_API_URL);
```

这种方式我个人不是很推荐，这毕竟是真实的代码，会记录在 git 提交记录，如果有一些需要保密的安去信息，这样使用就很棘手了。

### 方式4，使用 dotenv 实现全局变量

首先先安装 dotenv-cli，在根目录的 package.json 中添加：

```json
{
  "devDependencies": {
    "dotenv-cli": "latest"
  }
}
```

运行 pnpm install。

在 scripts 中修改你的 npm 命令：

```json
{
  "scripts": {
    "build": "dotenv -c production -- turbo run build",
    "dev": "dotenv -c development -- turbo run dev",
  }
}
```

将 .env 文件添加到 turbo.json ：

```json
{
  "globalDotEnv": [".env"],
  "pipeline": {
    "dev": {
      "dependsOn": ["^build"]
    }
  }
}
```

-c 参数是 dotenv-cli 的一个选项，它允许你指定一个特定的环境。例如，如果你有一个名为 .env.production 的文件，你可以使用 dotenv -c production 来加载这个文件中的环境变量。

## 总结

以上这些方式可以帮助 Turborepo 项目更好地管理环境变量，确保环境变量的一致性和安全性。选择适合项目的方式，有助于简化环境变量的管理和使用。希望官方可以尽快出一个更好的使用方案，现在这些方案使用起来确实不太舒适。