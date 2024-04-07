### any类型
1 基本含义 
any 类型表示没有任何限制，该类型的变量可以赋予任意类型的值。变量类型一旦设为any，TypeScript 实际上会关闭这个变量的类型检查。即使有明显的类型错误，只要句法正确，都不会报错。
2 类型推断问题
对于开发者没有指定类型、TypeScript 必须自己推断类型的那些变量，如果无法推断出类型，TypeScript 就会认为该变量的类型是any。
3 污染问题
**any类型除了关闭类型检查，还有一个很大的问题，就是它会“污染”其他变量**。它可以赋值给其他任何类型的变量（因为没有类型检查），**导致其他变量出错（也变成any了）**。
```
// X Y ADD 都会被推断为any

function add(x, y) {
  return x + y;
}
add(1, [1, 2, 3]) // 不报错


// 污染问题

let x:any = 'hello';
let y:number;
y = x; // 不报错
y * 123 // 不报错
y.toFixed() // 不报错
```
从集合论的角度看，any类型可以看成是所有其他类型的全集，包含了一切可能的类型。TypeScript 将这种类型称为“顶层类型”（top type），意为涵盖了所有下层。
### unkown类型
> 为了解决any类型“污染”其他变量的问题引入的
> 跟any的相似之处，所有类型的值都可以分配给unknown类型

```
let x:unknown;

x = true; // 正确
x = 42; // 正确
x = 'Hello World'; // 正确
```
跟any类型的不同之处在于，它不能直接使用。主要有以下几个限制。
```
// 不能直接赋值给其他类型的变量（除了any类型和unknown类型）。
let v:unknown = 123;

let v1:boolean = v; // 报错
let v2:number = v; // 报错


// 不能直接调用unknown类型变量的方法和属性。或者直接当作函数执行
let v1:unknown = { foo: 123 };
v1.foo  // 报错

let v2:unknown = 'hello';
v2.trim() // 报错

let v3:unknown = (n = 0) => n + 1;
v3() // 报错
```
unknown类型变量能够进行的运算是有限的，**只能进行比较运算（运算符==、===、!=、!==、||、&&、?）、取反运算（运算符!）、typeof运算符和instanceof运算符**这几种，其他运算都会报错。
那么，怎么才能使用unknown类型变量呢？
答案是只有经过**“类型缩小”**，unknown类型变量才可以使用。所谓“类型缩小”，就是缩小unknown变量的类型范围，确保不会出错。
```
let a:unknown = 1;

if (typeof a === 'number') {
  let r = a + 10; // 正确
}
```
上面示例中，unknown类型的变量a经过typeof运算以后，能够确定实际类型是number，就能用于加法运算了。这就是“类型缩小”，即将一个不确定的类型缩小为更明确的类型。
下面是另一个例子。
```
let s:unknown = 'hello';

if (typeof s === 'string') {
  s.length; // 正确
}
```
上面示例中，确定变量s的类型为字符串以后，才能调用它的length属性。
总之，**unknown可以看作是更安全的any**。一般来说，凡是需要设为any类型的地方，通常都应该优先考虑设为unknown类型。
在集合论上，unknown也可以视为所有其他类型（除了any）的全集，所以它和any一样，也属于 TypeScript 的顶层类型。
### never类型
> TypeScript 还引入了“空类型”的概念，即该类型为空，不包含任何值。

变量x的类型是never，就不可能赋给它任何值，否则都会报错。因为不存在空类型
never类型可以赋值给任意其他类型。空集是任何集合的子集。never类型是任何其他类型所共有的，TypeScript 把这种情况称为“底层类型”（bottom type）。
never类型的使用场景，主要是在一些类型运算之中，保证类型运算的完整性
```
// 上面示例中，函数f()会抛出错误，所以返回值类型可以写成never，即不可能返回任何值。
// 各种其他类型的变量都可以赋值为f()的运行结果（never类型）。

function f():never {
  throw new Error('Error');
}

let v1:number = f(); // 不报错
let v2:string = f(); // 不报错
let v3:boolean = f(); // 不报错
```
总之，TypeScript 有两个“顶层类型”（any和unknown），但是“底层类型”只有never唯一一个。
### 基本类型
> js基本类型 8种 = 5+2+1
> 5种基本类型：布尔型boolean、 数值型number、 字符串string、 undefined、 null
> 2种新增基本类型： bigint、symbol
> 1种引用类型：object
> 对象是最复杂的数据类型，又可以分成三个子类型。
>    - 狭义的对象（object）
>    - 数组（array）
>    - 函数（function）
> 
注意，上面所有类型的名称都是小写字母，首字母大写的Number、String、Boolean等在 JavaScript 语言中都是内置对象，而不是类型名称。
> 要精确表示比253还大的整数，可以使用内置的BigInt类型，它的表示方法是在整数后加一个n，例如9223372036854775808n，
> undefined 和 null 是两种独立类型，它们各自都只有一个值。undefined 类型只包含一个值undefined，null 类型也只包含一个值null，表示为空（即此处没有值）。。

JavaScript 语言将值分成8种类型。
TypeScript 继承了 JavaScript 的类型设计，以上8种类型可以看作 TypeScript 的基本类型。
另外，undefined 和 null 既可以作为值，也可以作为类型，取决于在哪里使用它们。
这8种基本类型是 TypeScript 类型系统的基础，复杂类型由它们组合而成。
### 包装对象类型
> 注意，String()只有当作构造函数使用时（即带有new命令调用），才会返回包装对象。如果当作普通函数使用（不带有new命令），返回就是一个普通字符串。其他两个构造函数Number()和Boolean()也是如此。

JavaScript 的8种类型之中，undefined和null其实是两个特殊值，object属于复合类型，剩下的五种属于原始类型（primitive value），代表最基本的、不可再分的值。

- boolean
- string
- number
- bigint
- symbol

上面这五种原始类型的值，都有对应的包装对象（wrapper object）首字母大写。
五种包装对象之中，symbol 类型和 bigint 类型无法直接获取它们的包装对象（即Symbol()和BigInt()**不能作为构造函数使用**），但是剩下三种可以。

- Boolean()
- String()
- Number()

以上三个构造函数，执行后可以直接获取某个原始类型值的包装对象。
```
const s = new String('hello');
typeof s // 'object'
s.charAt(1) // 'e'
```
由于包装对象的存在，导致每一个原始类型的值都有包装对象和字面量两种情况。
```
'hello' // 字面量
new String('hello') // 包装对象
```
为了区分这两种情况，TypeScript 对五种原始类型分别提供了大写和小写两种类型。

- Boolean 和 boolean
- String 和 string
- Number 和 number
- BigInt 和 bigint
- Symbol 和 symbol

大写类型同时包含包装对象和字面量两种情况，小写类型只包含字面量，不包含包装对象。
```
const s1:String = 'hello'; // 正确
const s2:String = new String('hello'); // 正确

const s3:string = 'hello'; // 正确
const s4:string = new String('hello'); // 报错
```
注意，目前在 TypeScript 里面，symbol和Symbol两种写法没有差异，bigint和BigInt也是如此，建议始终使用小写的symbol和bigint
### Object类型和object类型
> 无所不包的Object类型既不符合直觉，也不方便使用。

大写的Object类型代表 JavaScript 语言里面的广义对象。除了undefined和null这两个值不能转为对象，其他任何值都可以赋值给Object类型。空对象{}是Object类型的简写形式，所以使用Object时常常用空对象代替。
```
let obj:{};
// let obj:Object;
 
obj = true;
obj = 'hi';
obj = 1;
obj = { foo: 123 };
obj = [1, 2];
obj = (a:number) => a + 1;
```
小写的object类型代表 JavaScript 里面的狭义对象，只包含对象、数组和函数，不包括原始类型的值。
```
let obj:object;
 
obj = { foo: 123 };
obj = [1, 2];
obj = (a:number) => a + 1;
obj = true; // 报错
obj = 'hi'; // 报错
obj = 1; // 报错
```
大多数时候，我们使用对象类型，只希望包含真正的对象，不希望包含原始类型。所以，建议总是使用小写类型object，不使用大写类型Object。
### undefined和null
> 为了兼容js, 任何其他类型的变量都可以赋值为undefined或null。undefined和null这两种值能互相赋值。这并不是开发者想要的行为，也不利于发挥类型系统的优势。

```
// tsc --strictNullChecks app.ts

let age:number = 24;

age = null;      // 报错
age = undefined; // 报错

let x:undefined = null; // 报错
let y:null = undefined; // 报错

let x:any     = undefined;
let y:unknown = null;

let x:undefined = undefined; 
let y:null = null; 
```
打开strictNullChecks以后，undefined和null只能赋值给自身，或者any类型和unknown类型的变量
这个选项在配置文件tsconfig.json的写法如下。
```
{
  "compilerOptions": {
    "strictNullChecks": true
    // ...
  }
}
```
### 值类型
TypeScript 规定，**单个值**也是一种类型，称为“值类型”，如'hello'。
值类型就意味着不能赋为其他值。
```
let x:'hello';

x = 'hello'; // 正确
x = 'world'; // 报错
```
变量x是const命令声明的，TypeScript 就会推断它的类型是值https，而不是string类型。这样推断是合理的，因为const命令声明的变量，一旦声明就不能改变，相当于常量。
```
// ts类型推断  x 的类型是 "https"
const x = 'https';

// 类型推断是string
let a = 'https';

// 类型推断是any
let b;

// y 的类型是 string
const y:string = 'https';
```
```
// 类型推断 x 的类型是 { foo: number } 不是值类型
 const x = { foo: 1 };
```
值类型可能会出现一些很奇怪的报错。
```
const x:5 = 4 + 1; // 报错
```
上面示例中，等号左侧的类型是数值5，等号右侧4 + 1的类型，TypeScript 推测为number。由于5是number的子类型，number是5的父类型，父类型不能赋值给子类型，所以报错了。
但是，反过来是可以的，子类型可以赋值给父类型。因为子类型继承了父类型拥有父类所有特征。
```
let x:5 = 5;
let y:number = 4 + 1;

x = y; // 报错
y = x; // 正确
```
如果一定要让子类型可以赋值为父类型的值，就要用到类型断言。
```
const x:5 = (4 + 1) as 5; // 正确
```
### 联合类型
> 只包含单个值的值类型，用处不大。实际开发中，往往将多个值结合，作为联合类型使用。

联合类型（union types）指的是**多个类型组成的一个新类型**，使用符号|表示。
```
let setting:true|false;

let gender:'male'|'female';

let rainbowColor:'赤'|'橙'|'黄'|'绿'|'青'|'蓝'|'紫';


let x:string|number;
x = 123; // 正确
x = 'abc'; // 正确

let name:string|null;
name = 'John';
name = null;

// 联合类型的第一个成员前面，也可以加上竖杠|，这样便于多行书写。
let x:
  | 'one'
  | 'two'
  | 'three'
  | 'four';
```
如果一个变量有多种类型，读取该变量时，往往需要进行“**类型缩小**”（type narrowing），区分该值到底属于哪一种类型，然后再进一步处理。
```
function printId(
  id:number|string
) {
  if (typeof id === 'string') {
    console.log(id.toUpperCase());
  } else {
    console.log(id);
  }
}
```
### 交叉类型
交叉类型（intersection types）指的多个类型组成的一个新类型，使用符号&表示。
交叉类型A&B表示，即交叉类型同时满足A和B的特征。
```
let x:number&string;
```
上面示例中，变量x同时是数值和字符串，这当然是不可能的，所以 TypeScript 会认为x的类型实际是never。
交叉类型的主要用途是表示对象的合成。
```
let obj:
  { foo: string } &
  { bar: string };

obj = {
  foo: 'hello',
  bar: 'world'
};
```
交叉类型常常用来为对象类型添加新属性。
```
type A = { foo: number };

type B = A & { bar: number };
```
### type命令
> type命令用来**定义一个类型的别名**。

```
type Age = number;

let age:Age = 55;
```
上面示例中，type命令为number类型定义了一个别名Age。这样就能像使用number一样，使用Age作为类型。别名不允许重名。
别名的作用域是块级作用域。这意味着，代码块内部定义的别名，影响不到外部。
```
type Color = 'red';

if (Math.random() < 0.5) {
  type Color = 'blue';
}
```
别名支持使用表达式，也可以在定义一个别名时，使用另一个别名，即别名允许嵌套。
```
type World = "world";
type Greeting = `hello ${World}`;
```
上面示例中，别名Greeting使用了模板字符串，读取另一个别名World。
type命令属于类型相关的代码，编译成 JavaScript 的时候，会被全部删除。
### typeof运算符
> JavaScript 里面，typeof 运算符是一个一元运算符，，返回一个字符串。typeof运算符只可能返回**八种**结果。

```
typeof undefined; // "undefined"
typeof true; // "boolean"
typeof 1337; // "number"
typeof "foo"; // "string"
typeof Symbol(); // "symbol"
typeof 127n // "bigint"
typeof {}; // "object"
typeof parseInt; // "function"
```
TypeScript 将typeof运算符移植到了类型运算，它的操作数依然是一个值，但是返回的不是字符串，而是该值的 TypeScript 类型。所以**只能用在类型运算之中**（即跟类型相关的代码之中）**，不能用在值运算**。
```
const a = { x: 0 };

type T0 = typeof a;   // { x: number }
type T1 = typeof a.x; // number
```
也就是说，同一段代码可能存在两种typeof运算符，一种用在值相关的 JavaScript 代码部分，另一种用在类型相关的 TypeScript 代码部分。
```
let a = 1;
let b:typeof a;

if (typeof a === 'number') {
  b = a;
}
```
上面示例中，用到了两个typeof，第一个是类型运算，第二个是值运算。它们是不一样的，不要混淆。
JavaScript 的 typeof 遵守 JavaScript 规则，TypeScript 的 typeof 遵守 TypeScript 规则。它们的一个重要区别在于，编译后，前者会保留，后者会被全部删除。
由于编译时不会进行 JavaScript 的值运算，所以TypeScript 规定，typeof 的参数只能是标识符，不能是需要运算的表达式。
```
type T = typeof Date(); // 报错
```
上面示例会报错，原因是 typeof 的参数不能是一个值的运算式，而Date()需要运算才知道结果。
另外，typeof命令的参数不能是类型。
```
type Age = number;
type MyAge = typeof Age; // 报错
```
上面示例中，Age是一个类型别名，用作typeof命令的参数就会报错。
typeof 是一个很重要的 TypeScript 运算符，有些场合不知道某个变量foo的类型，这时使用typeof foo就可以获得它的类型。
### 块级类型声明
TypeScript 支持块级类型声明，即类型可以声明在代码块（用大括号表示）里面，并且只在当前代码块有效。
```
if (true) {
  type T = number;
  let v:T = 5;
} else {
  type T = string;
  let v:T = 'hello';
}
```
上面示例中，存在两个代码块，其中分别有一个类型T的声明。这两个声明都只在自己的代码块内部有效，在代码块外部无效。
### 类型的兼容
TypeScript 的类型存在兼容关系，某些类型可以兼容其他类型。
```
type T = number|string;

let a:number = 1;
let b:T = a;
```
上面示例中，变量a和b的类型是不一样的，但是变量a赋值给变量b并**不会报错**。这时，我们就认为，b的类型兼容a的类型。
TypeScript 为这种情况定义了一个专门术语。如果类型A的值可以赋值给类型B，那么类型A就称为类型B的子类型（subtype）。在上例中，类型number就是类型number|string的**子类型**。
TypeScript 的一个规则是，**凡是可以使用父类型的地方，都可以使用子类型，但是反过来不行。**
```
let a:'hi' = 'hi';
let b:string = 'hello';

b = a; // 正确
a = b; // 报错
```
因为子类型继承了父类型的所有特征，所以可以用在父类型的场合。但是，子类型还可能有一些父类型没有的特征，所以父类型不能用在子类型的场合。

### Symbol类型
Symbol 是 ES2015 新引入的一种原始类型的值。它类似于字符串，但是每一个 Symbol 值都是独一无二的，与其他任何值都不相等。
> ES6 引入了一种新的原始数据类型 Symbol ，表示独一无二的值，最大的用法是用来定义对象的唯一属性名。可以接受一个字符串作为参数，为新创建的 Symbol 提供描述，用来显示在控制台或者作为字符串的时候使用，便于区分。
> let sy = Symbol("KK"); 
> console.log(sy); // Symbol(KK)
> typeof(sy); // "symbol"
> // 相同参数 Symbol() 返回的值不相等
> let sy1 = Symbol("kk"); 
> sy === sy1; // false

Symbol 值通过Symbol()函数生成。在 TypeScript 里面，Symbol 的类型使用symbol表示。
```
let x:symbol = Symbol();
let y:symbol = Symbol();

x === y // false
```
上面示例中，变量x和y的类型都是symbol，且都用Symbol()生成，但是它们是不相等的。
unique symbol 类型的一个作用，就是用作属性名，这可以保证不会跟其他属性名冲突。如果要把某一个特定的 Symbol 值当作属性名，那么它的类型只能是 unique symbol，不能是 symbol。
```
const x:unique symbol = Symbol();
const y:symbol = Symbol();

interface Foo {
  [x]: string; // 正确
  [y]: string; // 报错
}
```
上面示例中，变量y当作属性名，但是y的类型是 symbol，不是固定不变的值，导致报错。
