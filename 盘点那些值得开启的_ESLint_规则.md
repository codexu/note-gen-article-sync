# 盘点那些值得开启的 ESLint 规则

ESLint 从 v8.53.0 起，将弃用代码风格相关规则，代码风格校验应该交由 `Prettier` 处理，如果你不喜欢 `Prettier`，那么要注意尽量使用 v8.50.0 之前的版本。

本文不谈代码风格的那些规则，只提及可以提高代码质量、减少运行时出错的的规则。

## 配置

ESLint 内置了大量规则，当然也可以通过插件添加更多规则，可以使用配置注释或配置文件来修改项目使用的规则。

### 配置文件

要在配置文件中配置规则，请使用rules的 Key(键值) 以及错误级别和您要使用的任何选项。

```json
{
    "rules": {
        "eqeqeq": "off",
        "curly": "error",
        "quotes": ["error", "double"]
    }
}
```

这里以 JSON 举例，当然，ESLint 还支持 `.js`、`.cjs`、`.yaml`、`.yml` 或直接在 `package.json` 内配置。

它的属性可以这样配置：

- `off` 或 `0` - 关闭规则
- `warn` 或 `1` - 打开规则作为警告（不影响退出代码）
- `error` 或 `2` - 将规则作为错误打开（触发时退出代码为 1）

### 配置注释

直接在文件内可以通过注释配置规则：

```js
/* eslint eqeqeq: "off", curly: "error" */
```

配置注释甚至可以包括说明为什么需要注释。-描述必须出现在配置之后，并且由两个或多个连续字符与配置分开。

### 禁用规则

要暂时禁用文件中的规则警告，请使用以下格式的块注释：

```js
/* eslint-disable */
/* eslint-enable */

/* eslint-disable no-alert, no-console */
/* eslint-enable no-alert, no-console */
```

## 推荐使用规则

### prefer-const

[prefer-const](https://eslint.org/docs/latest/rules/prefer-const)，如果变量永远不会被重新分配资源，那么应该使用 `const` 声明。

例如我们定义了许多变量，但其中有部分变量定义后不会再次修改，那么我们应该使用 `const` 定义，可以减少阅读者的认知负担，提高代码维护性。

```js
let a = 3; ❌ // 'a' is never reassigned. Use 'const' instead.
```

### no-const-assign

[no-const-assign](https://eslint.org/docs/latest/rules/no-const-assign)，`const` 声明的变量是无法修改的，如果没有 ESLint 提示，它将引发运行时报错，我们应在编码时直接排除掉这个错误。

```js
const a = 3;
a += 1; ❌ // 'a' is assigned a value but never used.
```

### no-var

[no-var](https://eslint.org/docs/latest/rules/no-var)，`let` 和 `const` 的声明都是块级作用域，`var` 则是函数作用域，直接避免使用 `var`，减少心智负担，相信用它的人已经很少了吧。

### no-prototype-builtins

[no-prototype-builtins](https://eslint.org/docs/latest/rules/no-prototype-builtins)，不要直接在对象上调用某些 `Object.prototype` 方法。

有点难以理解，这里指的的是某些方法，不是全部都禁止调用。以 `Object.prototype.hasOwnProperty` 为例，如果被恶意接收到 `{"hasOwnProperty": 1}` JSON 值，在直接调用 `hasOwnProperty` 方法时会导致报错。你可以按照下面的方式去调用：

```js
object.hasOwnProperty(key); ❌

Object.prototype.hasOwnProperty.call(object, key); ✅

const has = Object.prototype.hasOwnProperty; // 缓存一次
has.call(object, key); ✅✅

console.log(Object.hasOwn(object, key)); ✅✅✅ // ES2022 支持
```

### prefer-object-spread

[prefer-object-spread](https://eslint.org/docs/latest/rules/prefer-object-spread)，推荐使用对象扩展运算符 `...` 而不是使用 [Object.assign()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 进行对象的浅拷贝。

对象扩展运算符是 `ES2018` 中引入的对象扩展是一种声明式替代方案，其性比更动态、命令式的 `Object.assign` 更好，并且更直观，更易理解。

```js
const original = { a: 1, b: 2 };
const copy = Object.assign(original, { c: 3 }); ❌

const original = { a: 1, b: 2 };
const copy = { ...original, c: 3 }; ✅
```

### array-callback-return

[array-callback-return](https://eslint.org/docs/latest/rules/array-callback-return)，数组提供了多个过滤、映射等方法，如果你忘记加入 return，那么大概率会出现错误，如果你不想 return 结果，应该考虑使用 forEach。

```js
[[0, 1], [2, 3], [4, 5]].reduce((acc, item, index) => {
  const flatten = acc.concat(item); ❌
});

// good
[[0, 1], [2, 3], [4, 5]].reduce((acc, item, index) => {
  const flatten = acc.concat(item);
  return flatten; ✅
});
```

解构是 JavaScript ES6 中加入的一种新语法，用于从数组索引或对象属性创建变量，称为解构。

### prefer-destructuring

[prefer-destructuring](https://eslint.org/docs/latest/rules/prefer-destructuring)，访问和使用对象的多个属性时强制使用对象解构而不是通过成员表达式访问属性。

解构使你无需为这些属性创建临时引用，也无需重复访问该对象。重复对象访问会产生更多重复代码，需要更多阅读，并产生更多出错的情况，解构对象还提供了块中使用的对象结构的单一定义，而不需要读取整个块来确定使用的内容。

```js
function getFullName(user) {
  const firstName = user.firstName; ❌
  const lastName = user.lastName; ❌

  return `${firstName} ${lastName}`;
}

function getFullName({ firstName, lastName }) { ✅
  return `${firstName} ${lastName}`;
}

const arr = [1, 2, 3, 4];

const first = arr[0]; ❌
const second = arr[1]; ❌

const [first, second] = arr; ✅
```

当你的函数有多个返回值的时候，建议使用对象解构而不是数组解构：

```js
function processInput(input) {
  return [left, right, top, bottom]; ❌
  return { left, right, top, bottom }; ✅
}

// 对比
const [left, __, top] = processInput(input);
const { left, top } = processInput(input);
```

使用数组解构调用者需要考虑返回数据的顺序，不要讲 `react useState` 的返回值为什么是数组，因为它的特殊性，并且只有两个返回值。

### prefer-rest-params

[prefer-rest-params](https://eslint.org/docs/latest/rules/prefer-rest-params)，禁止使用 `arguments` 变量，使用剩余参数。

`arguments` 是一个类数组对象，它代表传递给一个函数的参数列表，可以直接访问，例如我们定义一个函数，未定义需要传递的参数，而调用函数时传递了参数，那么在函数内部，可以通过 `arguments` 获取到所有传递的参数。由于 `arguments` 没有 `Array.prototype` 提供的方法，所以有点不方便，并且 ES6 提供了可选参数的功能，我们可以将该功能用于可变参数函数。

```js
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments); ❌
}

function concatenateAll(...args) {} ✅
```

### default-param-last

[default-param-last](https://eslint.org/docs/latest/rules/default-param-last)，强制默认参数位于最后，这使得调用时，如果要传递于默认参数一致的值，或者不想传值时，传递的参数中无需加入那些无意义的 undefined。

```js
function createUser(isAdmin = false, id) {} ❌
createUser(undefined, "codexu")

function createUser(id, isAdmin = false) {} ✅
createUser("codexu") 
```

### no-new-func

[no-new-func](https://eslint.org/docs/latest/rules/no-new-func)，禁止使用 Function 构造函数来创建新函数，以这种方式创建函数会类似于 `eval()` 的字符串，会带来安全问题。

```js
const add = new Function('a', 'b', 'return a + b'); ❌
const subtract = Function('a', 'b', 'return a - b'); ❌
```

### no-param-reassign

[no-param-reassign](https://eslint.org/docs/latest/rules/no-param-reassign)，禁止修改参数。如果你操作了作为参数传入的对象，这将会产生副作用。在 JavaScript 中，对象是通过引用传递的，所以如果你修改了一个对象参数，这个修改会影响到函数外部的那个对象。重新赋值函数参数可能会使代码更难理解。当你看到一个参数被修改时，你需要跟踪它的变化，这可能会使代码阅读和理解变得更困难。函数的参数通常被视为不可变的，当你修改参数时，这可能会违反其他开发者的预期。

我认为这是一个原则，遵守它对你和你的伙伴都好。

### no-plusplus

[no-plusplus](https://eslint.org/docs/latest/rules/no-plusplus)，禁止使用一元运算符 `++` 和 `--`。因为如果你失误在变量与运算符中间增加了空格，那么相当于增加了一个分号，直接改变了代码的语义导致错误。

使用 `+= 1` 而不是 `++` 之类的语句来改变你的值也更具表现力。禁止一元递增和递减语句还可以防止你无意中预先递增/预先递减值，这也可能导致程序中出现意外行为。

```js
num++; ❌
--num; ❌

num += 1; ✅
num -= 1; ✅
```

### eqeqeq

[eqeqeq](https://eslint.org/docs/latest/rules/eqeqeq)，使用 `===` 和 `!==` 而不是`==`和 `!=`。

`==` 和 `!=` 在比较两个值时，会进行类型转换。这可能会导致一些意想不到的结果。例如，'5' == 5 会返回 `true`，因为字符串 '5' 被转换为了数字 5。而 `===` 和 `!==` 不会进行类型转换，它们只有在两个操作数的类型和值都相同（或不同）时，才会返回 true（或 false）。还可以避免一些常见的错误，例如 `null` 和 `undefined` 是不同的，但是 `null == undefined` 会返回 `true`。

### no-case-declarations

[no-case-declarations](https://eslint.org/docs/latest/rules/no-case-declarations)，在 `switch` 语句中，禁止在 `case` 或 `default` 子句中进行声明（如 let、const、function 和 class）。原因是 `JavaScript` 的 `switch` 语句的设计中，整个 `switch` 块有一个共享的词法环境，所以在一个 `case` 子句中声明的变量在整个 `switch` 块中都是可见的，如果这样做的话话出现意想不到的情况。

```js
switch (foo) {
  case 1:
    let x = 1;
    break;
  case 2:
    let x = 2; ❌ // SyntaxError: Identifier 'x' has already been declared
    break;
}
```

如果你想在 `switch` 语句中声明变量，那你应该使用块级作用域去包含你的声明：

```js
switch (foo) {
  case 1: {
    let x = 1;
    break;
  }
  case 2: {
    let x = 2; ✅
    break;
  }
}
```

### no-unneeded-ternary

[no-unneeded-ternary](https://eslint.org/docs/latest/rules/no-unneeded-ternary)，禁止不必要的三元运算语句。

三元运算本来就是为了简洁条件运算结果，既然这样，那就应该做到最简洁，下面这些情况尽量用其他运算符代替：

```js
const foo = a ? a : b; ❌
const bar = c ? true : false; ❌
const baz = c ? false : true; ❌
const quux = a != null ? a : b; ❌

const foo = a || b; ✅
const bar = !!c; ✅
const baz = !c; ✅
const quux = a ?? b; ✅
```

### no-restricted-globals

[no-restricted-globals](https://eslint.org/docs/latest/rules/no-restricted-globals)，禁用指定的全局变量。

这里需要配置禁用两个方法：`isNaN` 和 `isFinite`，因为他们可能会导致与你的预期结果不一致。

使用 `Number.isNaN` 而不是全局 `isNaN`，因为全局 `isNaN` 将非数字强制转换为数字，对于强制转换为 `NaN` 的任何内容都返回 `true`。

```js
isNaN('1.2'); ❌ // false
isNaN('1.2.3'); ❌ // true

Number.isNaN('1.2.3'); ✅ // false
Number.isNaN(Number('1.2.3')); ✅ // true
```

使用 `Number.isFinite` 代替全局 `isFinite`，因为全局 `isFinite` 将非数字强制转换为数字，对于强制转换为有限数字的任何内容返回 `true`。

```js
isFinite('2e3'); ❌ // true

Number.isFinite('2e3'); ✅ // false
Number.isFinite(parseInt('2e3', 10)); ✅ // true
```

## 插件

[eslint-plugin-import](https://github.com/import-js/eslint-plugin-import) 支持 ES6+ 的导入/导出语法校验，它默认已经开启了一些实用的规则，也提供了许多需要手动开启或者实验性的规则。

你可以通过 npm 来单独安装它：

```sh
npm install eslint-plugin-import --save-dev
```

在配置文件中加入：

```json
{
  "extends": [
    "plugin:import/recommended",
    "plugin:import/errors",
    "plugin:import/warnings"
  ],
  "plugins": [
    "import"
  ]
}
```

这里推荐开启以下几个规则。

### import/no-mutable-exports

[import/no-mutable-exports](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-mutable-exports.md),不要导出可变的变量。虽然某些特殊情况可能需要此技术，但一般来说，只应导出常量引用。

```js
let foo = 3;
export { foo }; ❌

const foo = 3;
export { foo }; ✅
```

### import/prefer-default-export

[import/prefer-default-export](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/prefer-default-export.md)，在具有单个导出的模块中，优先选择默认导出而不是命名导出，这更有利于可读性和可维护性。

```js
export function foo() {} ❌

export default function foo() {} ✅
```

### import/first

[import/first](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/first.md)，将所有 import 放在非导入语句上方，由于 import 会被提升，因此将它们全部放在顶部可以防止出现意外的行为。

```js
import foo from 'foo';
foo.init(); ❌
import bar from 'bar';

import foo from 'foo';
import bar from 'bar';
foo.init(); ✅
```

### import/no-webpack-loader-syntax

[import/no-webpack-loader-syntax](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-webpack-loader-syntax.md)，禁用模块导入时使用 `Webpack` 加载器语法。`webpack-loader` 语句是非标准的的语法，所以更推荐在 `webpack.config.js `中使用加载器语法。

```js
import fooSass from 'css!sass!foo.scss'; ❌
import fooSass from 'foo.scss'; ✅
```

## 总结

ESLint 提供了大量的规则，配合插件机制，可以辅助我们写出更优秀的 JavaScript 代码，极大地提高开发效率和代码质量。对于想编写高质量代码的开发人员来说，ESLint是非常有用的工具，毫不夸张的说，大量的 bug 都可以在使用 ESLint 时被避免。

当然 ESLint 对于很多人来说像噩梦一样，这种工具更适合在团队中使用，或者对自己要求较高的个人开发者使用，有的人认为 ESLint 这种东西就是反人类，而我认为，使用它正是帮你写出不反人类的代码。

建议大家使用一些常见的规范，例如 airbnb、standard，他们都拥有大量的拥护者，正是因为 JS 放飞自我的设计，才使得 ESLint 这种工具成为必备。

最后希望这些规则可以帮到各位，谢谢。