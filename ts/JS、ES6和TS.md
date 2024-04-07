:::info
💡  根据 [遗忘曲线](https://baike.baidu.com/item/%E9%81%97%E5%BF%98%E6%9B%B2%E7%BA%BF/7278665?fr=aladdin)：如果没有记录和回顾，6天后便会忘记75%的内容
      读书笔记正是帮助你记录和回顾的工具，不必拘泥于形式，其核心是：记录、翻看、思考
:::
## JavaScript
> 脚本语言（英语：Scripting language）是为了缩短传统的“编写、编译、链接、运行”（edit-compile-link-run）过程而创建的计算机编程语言。一个脚本通常是解释运行而非编译。
> 脚本语言主要包括：1、Python；2、JavaScript；3、Ruby；4、PHP；5、Shell；6、Lua。 Python是一种多用途、易读的编程语言，适用于数据分析、机器学习等。 JavaScript是网页开发中不可或缺的客户端脚本语言。
> ts是需要编译的，编译成js才能在浏览器运行

![](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1705329735542-f9b81da2-b4e0-433d-92a7-9001eb7c1257.jpeg)

## ES6
JavaScript有一个基于原型的、面向对象的编程模型。
它使用其他对象作为原型创建对象，并实现继承，它操作所谓的原型链,其实在javascript没有类的概念，在es6中才开始引入关键字class，当然在es6中也是用构造函数和原型链，只是在语法上更清晰，本质上还是和es5相同

![es6-tutorial.jpeg](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1705329761793-1be45985-908f-4691-8edf-454856d77f7b.jpeg#averageHue=%23fefdfb&clientId=u202f7329-5f24-4&from=drop&id=u7ee25196&originHeight=4344&originWidth=914&originalType=binary&ratio=2&rotation=0&showTitle=false&size=836460&status=done&style=none&taskId=u626c36ab-c38c-4c41-8ae5-2f7ec270f52&title=)
## TypeScript
TypeScript支持ES6类语法，但也添加了一些其他特性，比如访问修饰符和接口，因此TypeScript不是纯ES6
TypeScript 通过类型注解提供编译时的静态类型检查。
（TypeScript 就是为 JS 语言添加静态类型系统）

![](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1705377970122-34b6785b-cac3-4f8b-9b99-e04b1f83bc7a.jpeg)

## 强类型语言 弱类型语言 静态类型 动态类型语言
![编程语言.jpeg](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1705380502506-24f2ddb7-4432-4ccf-99d3-8bd19aa2a528.jpeg#averageHue=%23cdd1ce&clientId=u57cbda6f-f934-4&from=drop&id=uaeb51633&originHeight=1479&originWidth=1825&originalType=binary&ratio=2&rotation=0&showTitle=false&size=460277&status=done&style=none&taskId=uc372b10c-d77b-4c73-8a0f-5b012f8e287&title=)
## TS声明文件
当使用第三方库时，我们需要引用它的声明文件，才能获得对应的代码补全、接口提示等功能。

1. 通常我们会把声明语句放到一个单独的文件（jQuery.d.ts）中，这就是声明文件
```
// src/jQuery.d.ts

declare var jQuery: (selector: string) => any;
declare const jQuery: (selector: string) => any;
declare let jQuery: (selector: string) => any;
declare function jQuery(selector: string): any;

declare var 声明全局变量
declare function 声明全局方法
declare class 声明全局类
declare enum 声明全局枚举类型
declare namespace 声明（含有子属性的）全局对象
interface 和 type 声明全局类型
export 导出变量
export namespace 导出（含有子属性的）对象
export default ES6 默认导出
export = commonjs 导出模块
export as namespace UMD 库声明全局变量
declare global 扩展全局变量
declare module 扩展模块
```

2. 第三方声明文件

当然，jQuery 的声明文件不需要我们定义了，社区已经帮我们定义好了：[jQuery in DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/jquery/index.d.ts)。我们可以直接下载下来使用。
但是更推荐的是使用 @types 统一管理第三方库的声明文件。@types 的使用方式很简单，直接用 npm 安装对应的声明模块即可，以 jQuery 举例：
```bash
npm install @types/jquery --save-dev


console.log(1);
setTimeout(() => {
  console.log(2);
  Promise.resolve().then(data => {
    console.log(3);
  });
});
new Promise((resolve) => {
  resolve()
  console.log(4)
}).then(() => {
  console.log(5);
  setTimeout(() => {
    console.log(6);
  });
}).then(() => console.log(7))
console.log(8);

148 57 236
```
## TSC编译器
TypeScript 官方提供的编译器叫做 tsc，可以将 TypeScript 脚本编译成 JavaScript 脚本。本机想要编译 TypeScript 代码，必须安装 tsc。
根据约定，TypeScript 脚本文件使用.ts后缀名，JavaScript 脚本文件使用.js后缀名。tsc 的作用就是把.ts脚本转变成.js脚本。

tsc 是一个 npm 模块， 安装:
```
$ npm i typescript
```
tsc app.ts 命令会在当前目录下，生成一个app.js脚本文件
```
$ tsc app.ts
```
TypeScript 允许将tsc的编译参数，写在配置文件tsconfig.json。只要当前目录有这个文件，tsc就会自动读取，所以运行时可以不写参数。
```
$ tsc file1.ts file2.ts --outFile dist/app.js
```
上面这个命令写成tsconfig.json，就是下面这样。
```
{
  "files": ["file1.ts", "file2.ts"],
  "compilerOptions": {
    "outFile": "dist/app.js"
  }
}
```
有了这个配置文件，编译时直接调用tsc命令就可以了。
```
$ tsc
```
[ts-node](https://github.com/TypeStrong/ts-node) 是一个非官方的 npm 模块，可以直接运行 TypeScript 代码。
```
$ npm install -g ts-node

$ ts-node script.ts
```
如果只是想简单运行 TypeScript 代码看看结果，ts-node 不失为一个便捷的方法

## 面试题
### 说说你对ts的理解，与js的区别
> 1. 是什么
>    1. TypeScript 是 JavaScript 的一个超集，不仅支持 ECMAScript 6 标准，还扩展了js的语法，添加了一些其他特性，比如访问修饰符。
>    2. 它的设计目标是开发大型应用，为js添加了一个静态类型系统，通过类型声明提供编译时的静态类型检查。它支持面向对象编程的概念，如类、接口、继承、泛型等。
>    3. ts需要编译成JavaScript来运行，
>       1. TypeScript 官方没有做运行环境，只提供编译器(tsc)。需要编译成js后使用JavaScript 的运行环境（浏览器和 Node.js）运行。
>       2. 现有的js程序也可以直接在ts下工作（但可能会报错）
> 2. 特性
>    1. 类型声明和编译时类型检查
>    2. 类型推断
>       1. 没有声明变量类型的会自动推断变量类型  let a = 123; a = 'name';//报错
>    3. 类型擦除
>       1. 在编译过程中会利用工具擦除类型声明和类型相关的代码
>    4. 接口
>       1. 定义对象的类型
>    5. 枚举
>    6. Mixin 
>    7. 泛型编程
>       1. 写代码时使用一些 以后才会指定的类型
>    8. 命名空间
>       1. 名字只在该区域内有效
>    9. 元组
>       1. 相当于一个可以装不同类型数据的数组
>    10. ts的类、模块、箭头函数、可选参数和默认参数 是从ES6反向移植而来
> 3. 区别
>    1. ~~ts是js的超集，扩展了js的语法~~
>    2. ~~ts可处理已有的js代码，并只对其中的ts代码进行编译~~
>    3. ~~ts文件需要编译成js，才能执行（ts文件在编写是就会自动编译成js）~~
>    1. js是解释型脚本语言，ts是面向对象的编程语言（需要编译）
>    2. ts是强类型、静态类型语言，在编译时进行类型检查，在开发期间就能突出显示错误。 而js是弱类型动态类型的语言，在运行时才会报错
>    3. ts支持es6，支持模块，命名空间，可选参数默认参数；
>    4. 支持面向对象编程如类、继承、接口和泛型，
>    5. ts侧重于客户端，js可用与客户端和服务端。
> 
![IMG_6375.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1705378620408-de5cfc36-3ecd-4b1a-bf97-2111b82bfa5d.jpeg#averageHue=%23c0d4ad&clientId=u57cbda6f-f934-4&from=drop&id=HxGn4&originHeight=765&originWidth=1170&originalType=binary&ratio=2&rotation=0&showTitle=false&size=431564&status=done&style=none&taskId=ud7e4a35f-386f-4120-b8e7-cb524989988&title=)
> 4. ts的优点（静态类型的优点）
>    1. 提供可选的强静态类型
>    2. 更早的发现bug：能够在编译时进行类型检查，在开发期间就能突出显示错误。
>    3. 代码可预测：**声明的变量一旦指定类型，它的类型就再也不能修改。**这样变量就具有可预测性。
>       1. JavaScript 的变量可以赋予任何类型的值。有时候，我们会看到一个变量在执行的过程中变成各种各样的类型，一会是字符串，一会是对象，非常不好预测，尤其是有复杂条件判断的时候。这其实是并不是好的开发习惯，但在 JavaScript 它就是可以这么干！
>    4. 丰富的ide支持：**智能提示、自动补全、代码导航等**
>       1. 因为使用了类型，所以检测某个变量是什么类型、可以使用哪些方法就变得容易，在开发体验上就可以进行改善了。
>       2. 目前在绝大多数 IDE（集成开发环境）中已经支持 TypeScript 的 **智能提示、自动补全、代码导航** 等功能，并能在编写时实时反馈类型错误并提供准确的建议，比如可以指出传入函数的对象缺了哪些属性。
>    5. 方便重构
>       1. 修改他人的 JavaScript 代码，往往非常痛苦，项目越大越痛苦，因为不确定修改后是否会影响到其他部分的代码。
>       2. 类型信息大大减轻了重构的成本。一般来说，只要函数或对象的参数和返回值保持类型不变，就能基本确定，重构后的代码也能正常运行。如果还有配套的单元测试，就完全可以放心重构。越是大型的、多人合作的项目，类型信息能够提供的帮助越大。
>    6. 支持面向对象的写法
>       1. TypeScript 支持接口、抽象类、枚举等面向对象语言的特性，支持你更好地实现一些设计模式。
> 5. ts缺点（静态类型的缺点）
>    1. 需要编译成js才能运行
>       1. 浏览器和 Nodejs 并不支持 TypeScript，所以多了一步编译操作。对于普通项目来说通常不长，其实还好。但如果你用来写脚本的话，就需要多安装 tsc 编译工具，还要配置好 tsconfig.json 文件，还是有点麻烦。
>       2. 原生的 JavaScript 代码，可以直接在 JavaScript 引擎运行。添加类型系统以后，就多出了一个单独的编译步骤，检查类型是否正确，并将 TypeScript 代码转成 JavaScript 代码，这样才能运行。
>    2. 不是真正的静态类型，因为ts的类型是可选的
>    3. 增加了编程工作量  需要写更多的代码
>    4. 有一定的学习成本
>    5. 兼容性问题。
>       1. 当使用第三方库时，我们需要引用它的声明文件，才能获得对应的代码补全、接口提示等功能。（声明文件的质量无法保证）
>       2. TypeScript 依赖 JavaScript 生态，需要用到很多外部模块。但是，过去大部分 JavaScript 项目都没有做 TypeScript 适配，虽然可以自己动手做适配，不过使用时难免还是会有一些兼容性问题。

> 6. js的缺点
>    1. JavaScript支持动态类型，这可能会导致大量的运行时错误。
>    2. JavaScript中的错误会在运行时突出显示。

### 
