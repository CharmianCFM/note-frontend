[https://wangdoc.com/typescript/intro](https://wangdoc.com/typescript/intro)     阮一峰 
### 数组
TypeScript 数组有一个根本特征：所有成员的类型必须相同，但是成员数量是不确定的，可以是无限数量的成员，也可以是零成员。
数组的类型**有两种写法**：
```
let arr:number[] = [1, 2, 3];

let arr:(number|string)[];
```
数组类型的第二种写法是使用 TypeScript 内置的 Array 接口。
```
let arr:Array<number> = [1, 2, 3];

let arr:Array<number|string>;
```
TypeScript 允许声明**只读数组**，方法是在数组类型前面加上readonly关键字。
```
const arr:readonly number[] = [0, 1];

arr[1] = 2; // 报错
arr.push(3); // 报错
delete arr[0]; // 报错
```
上面示例中，arr是一个只读数组，删除、修改、新增数组成员都会报错。
TypeScript 使用T[][]的形式，表示二维数组，T是最底层数组成员的类型。
```
var multi:number[][] =
  [[1,2,3], [23,24,25]];
```
### 元组
元组（tuple）是 TypeScript 特有的数据类型，JavaScript 没有单独区分这种类型。它就像数组的各个成员的类型可以不同。TypeScript 的区分方法就是，成员类型写在方括号里面的就是元组，写在外面的就是数组。
由于成员的类型可以不一样，所以元组必须明确声明每个成员的类型。**由于需要声明每个成员的类型，所以大多数情况下，元组的成员数量是有限的**，从类型声明就可以明确知道，元组包含多少个成员，越界的成员会报错。
```
const s:[string, string, boolean]
  = ['a', 'b', true];
```
元组成员的类型可以添加问号后缀**（?），表示该成员是可选的。**所有可选成员必须在必选成员之后。
```
let a:[number, number?] = [1];
```
但是，使用扩展运算符**（...），可以表示不限成员数量的元组。**
```
type NamedNums = [
  string,
  ...number[]
];

const a:NamedNums = ['A', 1, 2];
const b:NamedNums = ['B', 1, 2, 3];
```
元组的成员可以添加成员名，这个成员名是说明性的，可以任意取名，没有实际作用。
```
type Color = [
  red: number,
  green: number,
  blue: number
];

const c:Color = [255, 255, 255];
```
元组也可以是**只读的**，不允许修改，有两种写法。
```
// 写法一
type t = readonly [number, string]

// 写法二
type t = Readonly<[number, string]>
```
### 函数
**函数的类型声明，需要在声明函数时，给出参数的类型和返回值的类型。返回值的类型通常可以不写**，因为 TypeScript 自己会推断出来。
```
function hello(
  txt:string
):void {
  console.log('hello ' + txt);
}
```
如果**变量被赋值为一个函数**，变量的类型有两种写法。
```
// 写法一
const hello = function (txt:string) {
  console.log('hello ' + txt);
}

// 写法二
const hello:
  (txt:string) => void
= function (txt) {
  console.log('hello ' + txt);
};
```
如果函数的类型定义很冗长，或者多个函数使用同一种类型，写法二用起来就很麻烦。因此，**往往用type命令为函数类型定义一个别名，便于指定给其他变量。**
```
type MyFunc = (txt:string) => void;

const hello:MyFunc = function (txt) {
  console.log('hello ' + txt);
};
```
TypeScript 允许函数传入的参数不足。**函数的实际参数个数，可以少于类型指定的参数个数，但是不能多于**。即 TypeScript 允许省略参数。
```
let myFunc:
  (a:number, b:number) => number;

myFunc = (a:number) => a; // 正确

myFunc = (
  a:number, b:number, c:number
) => a + b + c; // 报错
```
如果一个变量要套用另一个函数类型，有一个小技巧，就是使用typeof运算符。
```
function add(
  x:number,
  y:number
) {
  return x + y;
}

const myAdd:typeof add = function (x, y) {
  return x + y;
}
```
**函数类型还可以采用对象的写法。**
```
let add:{
  (x:number, y:number):number
};
 
add = function (x, y) {
  return x + y;
};
```
这种写法平时很少用，但是非常合适用在一个场合：函数本身存在属性。
```
function f(x:number) {
  console.log(x);
}

f.version = '1.0';
```
上面示例中，函数f()本身还有一个属性version。这时，f完全就是一个对象，类型就要使用对象的写法。
```
let foo: {
  (x:number): void;
  version: string
} = f;
```
**函数类型也可以使用 Interface 来声明，这种写法就是对象写法的翻版**。
```
interface myfn {
  (a:number, b:number): number;
}

var add:myfn = (a, b) => a + b;
```
### 对象
对象类型的最简单声明方法，就是**使用大括号表示对象，在大括号内部声明每个属性和方法的类型。属性的类型可以用分号结尾，也可以用逗号结尾。**
```
// 属性类型以分号结尾
type MyObj = {
  x:number;
  y:number;
};

// 属性类型以逗号结尾
type MyObj = {
  x:number,
  y:number,
};
```
最后一个属性后面，可以写分号或逗号，也可以不写。
**一旦声明了类型，对象赋值时，就不能缺少指定的属性，也不能有多余的属性。**读写不存在的属性也会报错。同样地，也不能删除类型声明中存在的属性，修改属性值是可以的。
```
type MyObj = {
  x:number;
  y:number;
};

const o1:MyObj = { x: 1 }; // 报错
const o2:MyObj = { x: 1, y: 1, z: 1 }; // 报错
o1.z = 1; // 报错
delete myUser.name // 报错
```
对象的**方法**（属性）使用**函数类型**描述。
```
const obj:{
  x: number;
  y: number;
  add(x:number, y:number): number;
  // 或者写成
  // add: (x:number, y:number) => number;
} = {
  x: 1,
  y: 1,
  add(x, y) {
    return x + y;
  }
};
```
除了type命令可以为对象类型声明一个别名，TypeScript 还提供了**interface命令**，可以把对象类型提炼为一个接口。
```
// 写法一
type MyObj = {
  x:number;
  y:number;
};

const obj:MyObj = { x: 1, y: 1 };

// 写法二
interface MyObj {
  x: number;
  y: number;
}

const obj:MyObj = { x: 1, y: 1 };
```
TypeScript 提供编译设置ExactOptionalPropertyTypes，只要同时打开这个设置和strictNullChecks，**可选属性**就不能设为undefined。
```
// 打开 ExactOptionsPropertyTypes 和 strictNullChecks
const obj: {
  x: number;
  y?: number;
} = { x: 1, y: undefined }; // 报错
```
上面示例中，打开了这两个设置以后，可选属性就不能设为undefined了。
注意，可选属性与允许设为undefined的必选属性是不等价的。
```
type A = { x:number, y?:number };
type B = { x:number, y:number|undefined };

const ObjA:A = { x: 1 }; // 正确
const ObjB:B = { x: 1 }; // 报错
```
上面示例中，属性y如果是一个可选属性，那就可以省略不写；如果是允许设为undefined的必选属性，一旦省略就会报错，必须显式写成{ x: 1, y: undefined }。
补充：lastName属性不确定存在的时候，要判断是否为undefined。建议使用下面的写法。
```
// 写法一
let firstName = (user.firstName === undefined)
  ? 'Foo' : user.firstName;
let lastName = (user.lastName === undefined)
  ? 'Bar' : user.lastName;

// 写法二
let firstName = user.firstName ?? 'Foo';
let lastName = user.lastName ?? 'Bar';
```
属性名前面加上readonly关键字，表示这个属性是**只读属性**，不能修改。
```
const person:{
  readonly age: number
} = { age: 20 };

person.age = 21; // 报错
```
注意，如果属性值是一个对象，readonly修饰符并不禁止修改该对象的属性，只是禁止完全替换掉该对象。
```
interface Home {
  readonly resident: {
    name: string;
    age: number
  };
}

const h:Home = {
  resident: {
    name: 'Vicky',
    age: 42
  }
};

h.resident.age = 32; // 正确
h.resident = {
  name: 'Kate',
  age: 23 
} // 报错
```
如果对象的属性非常多，一个个声明类型就很麻烦，而且有些时候，无法事前知道对象会有多少属性，比如外部 API 返回的对象。这时 TypeScript 允许采用属性名表达式的写法来描述类型**，称为“属性名的索引类型”**。
索引类型里面，最常见的就是属性名的字符串索引。
```
type MyObj = {
  [property: string]: string
};

const obj:MyObj = {
  foo: 'a',
  bar: 'b',
  baz: 'c',
};
```
上面示例中，类型MyObj的属性名类型就采用了表达式形式，写在方括号里面。[property: string]的property表示属性名，这个是可以随便起的，它的类型是string，即属性名类型为string。
JavaScript 对象的属性名的类型有三种可能，除了上例的string，还有number和symbol。
```
type T1 = {
  [property: number]: string
};

type T2 = {
  [property: symbol]: string
};
```
上面示例中，对象属性名的类型分别为number和symbol。
```
type MyArr = {
  [n:number]: number;
};

const arr:MyArr = [1, 2, 3];
// 或者
const arr:MyArr = {
  0: 1,
  1: 2,
  2: 3,
};
```
### interface
**interface 是对象的模板**，可以看作是**一种类型约定**，中文译为“接口”。使用了某个模板的对象，就拥有了指定的类型结构。
```
interface Person {
  firstName: string;
  readonly lastName: string; // 只读
  age?: number; // 可选
}

interface A {
  [prop: number]: string; //属性索引
}

// 字符串索引 > 数值索引
interface A {
  [prop: string]: number;
  [prop: number]: string; // 报错
}

interface B {
  [prop: string]: number;
  [prop: number]: number; // 正确
}
```
对象的**方法共有三种写法**。
```
// 写法一
interface A {
  f(x: boolean): string;
}

// 写法二
interface B {
  f: (x: boolean) => string;
}

// 写法三 函数的对象写法
interface C {
  f: { (x: boolean): string };
}
```
**interface 也可以用来声明独立的函数**。
```
interface Add {
  (x:number, y:number): number;
}

const myAdd:Add = (x,y) => x + y;
```
interface 内部可以**使用new关键字，表示构造函数**。
```
interface ErrorConstructor {
  new (message?: string): Error;
}
```
interface 可以使用extends关键字，**继承其他 interface**。但是继承的这些同名属性不能有类型冲突，否则会报错。
```
interface Style {
  color: string;
}

interface Shape {
  name: string;
}

interface Circle extends Shape {
  radius: number;
}

interface Circle extends Style, Shape {
  radius: number;
}
```
interface 可以**继承type**命令定义的对象类型。注意，如果type命令定义的类型不是对象，interface 就无法继承。
```
type Country = {
  name: string;
  capital: string;
}

interface CountryWithPop extends Country {
  population: number;
}
```
interface 还可以**继承 class**，即继承该类的所有成员。
```
class A {
  x:string = '';

  y():boolean {
    return true;
  }
}

interface B extends A {
  z: number
}

const b:B = {
  x: '',
  y: function(){ return true },
  z: 123
}
```
**同名接口会合并**，如果同名方法有不同的类型声明，那么会发生函数重载。而且，后面的定义比前面的定义具有更高的优先级。
```
interface Cloner {
  clone(animal: Animal): Animal;
}

interface Cloner {
  clone(animal: Sheep): Sheep;
}

interface Cloner {
  clone(animal: Dog): Dog;
  clone(animal: Cat): Cat;
}

// 等同于
interface Cloner {
  clone(animal: Dog): Dog;
  clone(animal: Cat): Cat;
  clone(animal: Sheep): Sheep;
  clone(animal: Animal): Animal;
}
```
### interface 与 type 的异同
interface命令与type命令作用类似，都可以表示对象类型。
它们的相似之处，首先表现在都能为对象类型起名。
interface 与 type 的区别有下面几点。
（1）type能够表示非对象类型，而interface只能表示对象类型（包括数组、函数等）。
（2）interface可以继承其他类型，type不支持继承。
继承的主要作用是添加属性，type定义的对象类型如果想要添加属性，只能使用&运算符，重新定义一个类型。
```
type Animal = {
  name: string
}

type Bear = Animal & {
  honey: boolean
}
```
（3）同名interface会自动合并，同名type则会报错。也就是说，TypeScript 不允许使用type多次定义同一个类型。
（4）interface不能包含属性映射（mapping），type可以，详见《映射》一章。
```
interface Point {
  x: number;
  y: number;
}

// 正确
type PointCopy1 = {
  [Key in keyof Point]: Point[Key];
};

// 报错
interface PointCopy2 {
  [Key in keyof Point]: Point[Key];
};
```
（7）interface无法表达某些复杂类型（比如交叉类型和联合类型），但是type可以。
（6）type 可以扩展原始数据类型，interface 不行。
```
// 正确
type MyStr = string & {
  type: 'new'
};

// 报错
interface MyStr extends string {
  type: 'new'
}
```
（5）this关键字只能用于interface。
```
// 正确
interface Foo {
  add(num:number): this;
};

// 报错
type Foo = {
  add(num:number): this;
};
```
上面示例中，type 命令声明的方法add()，返回this就报错了。interface 命令没有这个问题。
下面是返回this的实际对象的例子。
```
class Calculator implements Foo {
  result = 0;
  add(num:number) {
    this.result += num;
    return this;
  }
}
```
### 类
> 类（class）是面向对象编程的基本构件，封装了属性和方法，TypeScript 给予了全面支持。

readonly **属性的初始值，可以写在顶层属性，也可以写在构造方法里面**。
```
class A {
  readonly id:string;

  constructor() {
    this.id = 'bar'; // 正确
  }
}
```
**类的方法就是普通函数**，类型声明方式与函数一致。
```
class Point {
  x:number;
  y:number;

  constructor(x:number, y:number) {
    this.x = x;
    this.y = y;
  }

  add(point:Point) {
    return new Point(
      this.x + point.x,
      this.y + point.y
    );
  }
}
```
上面示例中，构造方法constructor()和普通方法add()都注明了参数类型，但是省略了返回值类型，因为 TypeScript 可以自己推断出来。
另外，**构造方法不能声明返回值类型，否则报错，因为它总是返回实例对象。**
```
class B {
  constructor():object { // 报错
    // ...
  }
}
```
**类允许定义属性索引。**
```
class MyClass {
  [s:string]: boolean |
    ((s:string) => boolean);

  get(s:string) {
    return this[s] as boolean;
  }
}
```
**TypeScript 的类本身就是一种类型，但是它代表该类的实例类型，而不是 class 的自身类型。**
```
class Color {
  name:string;

  constructor(name:string) {
    this.name = name;
  }
}

const green:Color = new Color('green');
```
上面示例中，定义了一个类Color。它的类名就代表一种类型，实例对象green就属于该类型。
### 泛型
有些时候，函数返回值的类型与参数类型是相关的。
```
function getFirst(arr) {
  return arr[0];
}
```
上面示例中，函数getFirst()总是返回参数数组的第一个成员。参数数组是什么类型，返回值就是什么类型。
为了解决这个问题，TypeScript 就引入了“泛型”（generics）。**泛型的特点就是带有“类型参数”**（type parameter）。
```
function getFirst<T>(arr:T[]):T {
  return arr[0];
}
```
上面示例中，函数getFirst()的函数名后面尖括号的部分**<T>，就是类型参数**，参数要放在一对尖括号（<>）里面。本例只有一个类型参数T，可以将其理解为类型声明需要的变量，需要在调用时传入具体的参数类型。
上例的函数getFirst()的参数类型是T[]，返回值类型是T，就清楚地表示了两者之间的关系。比如，输入的参数类型是number[]，那么 T 的值就是number，因此返回值类型也是number。
**函数调用时，需要提供类型参数。**
```
getFirst<number>([1, 2, 3])
```
**不过为了方便，函数调用时，往往省略不写类型参数的值**，让 TypeScript 自己推断。
```
getFirst([1, 2, 3])
```
上面示例中，TypeScript 会从实际参数[1, 2, 3]，推断出类型参数 T 的值为number。
**有些复杂的使用场景，TypeScript 可能推断不出类型参数的值，这时就必须显式给出了。**
```
function comb<T>(arr1:T[], arr2:T[]):T[] {
  return arr1.concat(arr2);
}
```
上面示例中，两个参数arr1、arr2和返回值都是同一个类型。如果不给出类型参数的值，下面的调用会报错。
```
comb([1, 2], ['a', 'b']) // 报错
```
上面示例会报错，TypeScript 认为两个参数不是同一个类型。但是，如果类型参数是一个联合类型，就不会报错。
```
comb<number|string>([1, 2], ['a', 'b']) // 正确
```
上面示例中，类型参数是一个联合类型，使得两个参数都符合类型参数，就不报错了。这种情况下，类型参数是不能省略不写的。
**类型参数的名字，可以随便取，但是必须为合法的标识符。**习惯上，类型参数的第一个字符往往采用大写字母。**一般会使用T（type 的第一个字母）作为类型参数的名字。如果有多个类型参数，则使用 T 后面的 U、V 等字母命名，各个参数之间使用逗号（“,”）分隔。**
下面是多个类型参数的例子。
```
function map<T, U>(
  arr:T[],
  f:(arg:T) => U
):U[] {
  return arr.map(f);
}

// 用法实例
map<string, number>(
  ['1', '2', '3'],
  (n) => parseInt(n)
); // 返回 [1, 2, 3]
```
上面示例将数组的实例方法map()改写成全局函数，它有两个类型参数T和U。含义是，原始数组的类型为T[]，对该数组的每个成员执行一个处理函数f，将类型T转成类型U，那么就会得到一个类型为U[]的数组。
总之，泛型可以理解成一段类型逻辑，需要类型参数来表达。有了类型参数以后，可以在输入类型与输出类型之间，建立一一对应关系。
function关键字定义的**泛型函数，类型参数放在尖括号中，写在函数名后面。**
```
function id<T>(arg:T):T {
  return arg;
}
```
那么对于**变量形式定义的函数**，泛型有下面两种写法。
```
// 写法一
let myId:<T>(arg:T) => T = id;

// 写法二
let myId:{ <T>(arg:T): T } = id;
```
interface 也可以采用泛型的写法。**泛型接口**
```
interface Box<Type> {
  contents: Type;
}

let box:Box<string>;
```
泛型类的类型参数写在类名后面。**泛型类**
```
class Pair<K, V> {
  key: K;
  value: V;
}
```
```
class C<NumType> {
  value!: NumType;
  add!: (x: NumType, y: NumType) => NumType;
}

let foo = new C<number>();

foo.value = 0;
foo.add = function (x, y) {
  return x + y;
};
```
上面示例中，先新建类C的实例foo，然后再定义实例的value属性和add()方法。类的定义中，属性和方法后面的感叹号是非空断言，告诉 TypeScript 它们都是非空的，后面会赋值。
**type 命令定义的类型别名，也可以使用泛型**。
```
type Nullable<T> = T | undefined | null;
```
**类型参数可以设置默认值。**使用时，如果没有给出类型参数的值，就会使用默认值。
```
function getFirst<T = string>(
  arr:T[]
):T {
  return arr[0];
}
```
泛型有一些使用注意点。
**（1）尽量少用泛型。**
泛型虽然灵活，但是会加大代码的复杂性，使其变得难读难写。一般来说，只要使用了泛型，类型声明通常都不太易读，容易写得很复杂。因此，可以不用泛型就不要用。
**（2）类型参数越少越好。**
多一个类型参数，多一道替换步骤，加大复杂性。因此，类型参数越少越好。
### Enum类型
TypeScript 就设计了 Enum 结构，用来将相关常量放在一个容器里面，方便使用。
```
enum Color {
  Red,     // 0
  Green,   // 1
  Blue     // 2
}
```
上面示例声明了一个 Enum 结构Color，里面包含三个成员Red、Green和Blue。第一个成员的值默认为整数0，第二个为1，第三个为2，以此类推。
使用时，**调用 Enum 的成员，与调用对象属性的写法一样**，可以使用点运算符，也可以使用方括号运算符。
```
let c = Color.Green; // 1
// 等同于
let c = Color['Green']; // 1
```
Enum 结构本身也是一种类型。比如，上例的变量c等于1，它的类型可以是 Color，也可以是number。
```
let c:Color = Color.Green; // 正确
let c:number = Color.Green; // 正确
```
上面示例中，变量c的类型写成Color或number都可以。但是，Color类型的语义更好。
Enum 结构的特别之处在于，它既是一种类型，也是一个值。绝大多数 TypeScript 语法都是类型语法，编译后会全部去除，但是** Enum 结构是一个值，编译后会变成 JavaScript 对象，留在代码中。**
```
// 编译前
enum Color {
  Red,     // 0
  Green,   // 1
  Blue     // 2
}

// 编译后
let Color = {
  Red: 0,
  Green: 1,
  Blue: 2
};
```
**由于 TypeScript 的定位是 JavaScript 语言的类型增强，所以官方建议谨慎使用 Enum 结构，因为它不仅仅是类型，还会为编译后的代码加入一个对象。**
**Enum 结构比较适合的场景是，成员的值不重要，名字更重要，从而增加代码的可读性和可维护性。**
```
enum Operator {
  ADD,
  DIV,
  MUL,
  SUB
}

function compute(
  op:Operator,
  a:number,
  b:number
) {
  switch (op) {
    case Operator.ADD:
      return a + b;
    case Operator.DIV:
      return a / b;
    case Operator.MUL:
      return a * b;
    case Operator.SUB:
      return a - b;
    default:
      throw new Error('wrong operator');
  }
}

compute(Operator.ADD, 1, 3) // 4
```
Enum 成员默认不必赋值，系统会从零开始逐一递增，按照顺序为每个成员赋值，比如0、1、2……
但是，也可以为 Enum 成员显式赋值。成员的值可以是任意数值，但不能是大整数（Bigint）。成员的值甚至可以相同。如果只设定第一个成员的值，后面成员的值就会从这个值开始递增。**Enum 成员值都是只读的，不能重新赋值。**
**为了让这一点更醒目，通常会在 enum 关键字前面加上const修饰，表示这是常量，不能再次赋值**。
```
const enum Color {
  Red,
  Green,
  Blue
}
```
加上const还有一个好处，就是编译为 JavaScript 代码后，代码中 Enum 成员会被替换成对应的值，这样能提高性能表现。
```
const enum Color {
  Red,
  Green,
  Blue
}

const x = Color.Red;
const y = Color.Green;
const z = Color.Blue;

// 编译后
const x = 0 /* Color.Red */;
const y = 1 /* Color.Green */;
const z = 2 /* Color.Blue */;
```
多个同名的 Enum 结构会自动合并。同名 Enum 合并时，不能有同名成员，否则报错。
**字符串 Enum 可以使用联合类型（union）代替。**
```
function move(
  where:'Up'|'Down'|'Left'|'Right'
) {
  // ...
 }
```
**keyof 运算符可以取出 Enum 结构的所有成员名，作为联合类型返回。**
```
enum MyEnum {
  A = 'a',
  B = 'b'
}

// 'A'|'B'
type Foo = keyof typeof MyEnum;
```
### 类型断言
对于没有类型声明的值，TypeScript 会进行类型推断，很多时候得到的结果，未必是开发者想要的。
TypeScript 提供了“类型断言”这样一种手段，**允许开发者在代码中“断言”某个值的类型，告诉编译器此处的值是什么类型。TypeScript 一旦发现存在类型断言，就不再对该值进行类型推断**，而是直接采用断言给出的类型。
```
type T = 'a'|'b'|'c';
let foo = 'a';

let bar:T = foo; // 报错
let bar:T = foo as T; // 正确
```
#### 语法
类型断言有两种语法。其中的语法一因为跟 JSX 语法冲突，现在一般都使用**语法二：值 as 类型**。
```
// 语法一：<类型>值
<Type>value

// 语法二：值 as 类型
value as Type
```
对象类型有严格字面量检查，如果存在额外的属性会报错。
```
// 报错
const p:{ x: number } = { x: 0, y: 0 };
```
上面示例中，等号右侧是一个对象字面量，多出了属性y，导致报错。解决方法就是使用类型断言，可以用两种不同的断言。
```
// 正确
const p0:{ x: number } =
  { x: 0, y: 0 } as { x: number };

// 正确
const p1:{ x: number } =
  { x: 0, y: 0 } as { x: number; y: number };
```
上面示例中，两种类型断言都是正确的。第一种断言将类型改成与等号左边一致，第二种断言使得等号右边的类型是左边类型的子类型，子类型可以赋值给父类型，同时因为存在类型断言，就没有严格字面量检查了，所以不报错。
#### 前提条件
**类型断言的使用前提是，值的实际类型与断言的类型必须满足一个条件。**
```
expr as T
```
上面代码中，expr是实际的值，T是类型断言，**它们必须满足下面的条件：expr是T的子类型，或者T是expr的子类型。**
```
const n = 1;
const m:string = n as string; // 报错
```
#### as const
如果没有声明变量类型，let 命令声明的变量，会被类型推断为 TypeScript 内置的基本类型之一；const 命令声明的变量，则被推断为值类型常量。
```
// 类型推断为基本类型 string
let s1 = 'JavaScript';

// 类型推断为字符串 “JavaScript”
const s2 = 'JavaScript';
```
有些时候，let 变量会出现一些意想不到的报错，变更成 const 变量就能消除报错。
```
let s = 'JavaScript';

type Lang =
  |'JavaScript'
  |'TypeScript'
  |'Python';

function setLang(language:Lang) {
  /* ... */
}

setLang(s); // 报错
```
一种解决方法就是把 let 命令改成 const 命令。
```
const s = 'JavaScript';
```
另一种解决方法是使用类型断言。TypeScript 提供了一种**特殊的类型断言as const，用于告诉编译器，推断类型时，可以将这个值推断为常量**
```
let s = 'JavaScript' as const;
setLang(s);  // 正确
```
**注意，as const断言只能用于字面量，不能用于变量。也不能用于表达式。**
as const断言可以用于整个对象，也可以用于对象的单个属性，这时它的类型缩小效果是不一样的。
```
const v1 = {
  x: 1,
  y: 2,
}; // 类型是 { x: number; y: number; }

const v2 = {
  x: 1 as const,
  y: 2,
}; // 类型是 { x: 1; y: number; }

const v3 = {
  x: 1,
  y: 2,
} as const; // 类型是 { readonly x: 1; readonly y: 2; }
```
```
// a1 的类型推断为 number[]
const a1 = [1, 2, 3];

// a2 的类型推断为 readonly [1, 2, 3]
const a2 = [1, 2, 3] as const;
```
上面示例中，**数组字面量使用as const断言后，类型推断就变成了只读元组。**
由于as const会将数组变成只读元组，所以很适合用于函数的 rest 参数。
```
function add(x:number, y:number) {
  return x + y;
}

const nums = [1, 2];
const total = add(...nums); // 报错
```
上面示例中，**变量nums的类型推断为number[]，**导致使用扩展运算符...传入函数add()会报错，因为add()只能接受两个参数，**而...nums并不能保证参数的个数**。
事实上，对于固定参数个数的函数，如果传入的参数包含扩展运算符，那么扩展运算符只能用于元组。只有当函数定义使用了 rest 参数，扩展运算符才能用于数组。
解决方法就是使用as const断言，将数组变成元组。
```
const nums = [1, 2] as const;
const total = add(...nums); // 正确
```
上面示例中，使用as const断言后，变量nums的类型会被推断为readonly [1, 2]，使用扩展运算符展开后，正好符合函数add()的参数类型。
**Enum 成员也可以使用as const断言。**
```
enum Foo {
  X,
  Y,
}
let e1 = Foo.X;            // Foo
let e2 = Foo.X as const;   // Foo.X
```
上面示例中，如果不使用as const断言，变量e1的类型被推断为整个 Enum 类型；**使用了as const断言以后，变量e2的类型被推断为 Enum 的某个成员，这意味着它不能变更为其他成员。**
#### 非空断言
对于那些可能为空的变量（即可能等于undefined或null），TypeScript 提供了非空断言，保证这些变量不会为空，写法是在变量名后面加上感叹号!。
```
function f(x?:number|null) {
  validateNumber(x); // 自定义函数，确保 x 是数值
  console.log(x!.toFixed());
}

function validateNumber(e?:number|null) {
  if (typeof e !== 'number')
    throw new Error('Not a number');
}
```
上面示例中，函数f()的参数x的类型是number|null，即可能为空。如果为空，就不存在x.toFixed()方法，这样写会报错。但是，开发者可以确认，经过validateNumber()的前置检验，变量x肯定不会为空，这时就可以使用非空断言，为函数体内部的变量x加上后缀!，x!.toFixed()编译就不会报错了。
非空断言在实际编程中很有用，有时可以省去一些额外的判断。
```
const root = document.getElementById('root');

// 报错
root.addEventListener('click', e => {
  /* ... */
});
```
上面示例中，getElementById()有可能返回空值null，即变量root可能为空，这时对它调用addEventListener()方法就会报错，通不过编译。但是，开发者如果可以确认root元素肯定会在网页中存在，这时就可以使用非空断言。
```
const root = document.getElementById('root')!;
```
上面示例中，getElementById()方法加上后缀!，表示这个方法肯定返回非空结果。
不过，非空断言会造成安全隐患，只有在确定一个表达式的值不为空时才能使用。比较保险的做法还是手动检查一下是否为空。
```
const root = document.getElementById('root');

if (root === null) {
  throw new Error('Unable to find DOM element #root');
}

root.addEventListener('click', e => {
  /* ... */
});
```
上面示例中，如果root为空会抛错，比非空断言更保险一点。
非空断言还可以用于赋值断言。TypeScript 有一个编译设置，要求类的属性必须初始化（即有初始值），如果不对属性赋值就会报错。
```
class Point {
  x:number; // 报错
  y:number; // 报错

  constructor(x:number, y:number) {
    // ...
  }
}
```
上面示例中，属性x和y会报错，因为 TypeScript 认为它们没有初始化。
这时就可以使用非空断言，表示这两个属性肯定会有值，这样就不会报错了。
```
class Point {
  x!:number; // 正确
  y!:number; // 正确

  constructor(x:number, y:number) {
    // ...
  }
}
```
另外，非空断言只有在打开编译选项strictNullChecks时才有意义。如果不打开这个选项，编译器就不会检查某个变量是否可能为undefined或null。
### ts模块
任何包含 import 或 export 语句的文件，就是一个模块（module）。相应地，如果文件不包含 export 语句，就是一个全局的脚本文件。
模块本身就是一个作用域，不属于全局作用域。模块内部的变量、函数、类只在内部可见，对于模块外部是不可见的。暴露给外部的接口，必须用 export 命令声明；如果其他文件要使用模块的接口，必须用 import 命令来输入。
如果一个文件不包含 export 语句，但是希望把它当作一个模块（即内部变量对外不可见），可以在脚本头部添加一行语句。
```
export {};
```
上面这行语句不产生任何实际作用，但会让当前文件被当作模块处理，所有它的代码都变成了内部代码。
TypeScript 模块除了支持所有 ES 模块的语法，特别之处在于允许输出和输入类型。
```
export type Bool = true | false;
```
上面示例中，当前脚本输出一个类型别名Bool。这行语句把类型定义和接口输出写在一行，也可以写成两行。
```
type Bool = true | false;

export { Bool };
```
假定上面的模块文件为a.ts，另一个文件b.ts就可以使用 import 语句，输入这个类型。
```
import { Bool } from './a';

let foo:Bool = true;
```
#### import type 语句
import 在一条语句中，可以同时输入类型和正常接口。
```
// a.ts
export interface A {
  foo: string;
}

export let a = 123;

// b.ts
import { A, a } from './a';
```
这样很不利于区分类型和正常接口，容易造成混淆。为了解决这个问题，TypeScript 引入了两个解决方法。
第一个方法是在 import 语句输入的类型前面加上type关键字。
```
import { type A, a } from './a';
```
上面示例中，import 语句输入的类型A前面有type关键字，表示这是一个类型。
第二个方法是使用 import type 语句，这个语句只用来输入类型，不用来输入正常接口。
```
// 正确
import type { A } from './a';
let b:A = 'hello';

// 报错
import type { a } from './a';
let b = a;
```
import type 语句也可以输入默认类型。
```
import type DefaultType from 'moduleA';
```
import type 在一个名称空间下，输入所有类型的写法如下。
```
import type * as TypeNS from 'moduleA';
```
同样的，export 语句也有两种方法，表示输出的是类型。
```
type A = 'a';
type B = 'b';

// 方法一
export {type A, type B};

// 方法二
export type {A, B};
```
#### commonJS 模块
TypeScript 使用import =语句输入 CommonJS 模块。
```
import fs = require('fs');
const code = fs.readFileSync('hello.ts', 'utf8');
```
上面示例中，使用import =语句和require()命令输入了一个 CommonJS 模块。模块本身的用法跟 Node.js 是一样的。
除了使用**import =语句**，TypeScript 还允许使用**import * as [接口名] from "模块文件"**输入 CommonJS 模块。
```
import * as fs from 'fs';
// 等同于
import fs = require('fs');
```

TypeScript 使用**export =**语句，输出 CommonJS 模块的对象，等同于 CommonJS 的module.exports对象。
```
let obj = { foo: 123 };

export = obj;
```
export =语句输出的对象，只能使用import =语句加载。
```
import obj = require('./a');

console.log(obj.foo); // 123
```
#### 模块定位
模块定位（module resolution）指的是一种算法，用来确定 import 语句和 export 语句里面的模块文件位置。
```
// 相对模块
import { TypeA } from './a';

// 非相对模块
import * as $ from "jquery";
```
相对模块指的是路径以/、./、../开头的模块。下面 import 语句加载的模块，都是相对模块。

- import Entry from "./components/Entry";
- import { DefaultHeaders } from "../constants/http";
- import "/mod";

相对模块的定位，是根据当前脚本的位置进行计算的，一般用于保存在当前项目目录结构中的模块脚本。
非相对模块指的是不带有路径信息的模块。下面 import 语句加载的模块，都是非相对模块。

- import * as $ from "jquery";
- import { Component } from "@angular/core";

非相对模块的定位，是由baseUrl属性或模块映射而确定的，通常用于加载外部模块。
#### 路径映射
TypeScript 允许开发者在tsconfig.json文件里面，手动指定脚本模块的路径。
（1）baseUrl
baseUrl字段可以手动指定脚本模块的基准目录。
```
{
  "compilerOptions": {
    "baseUrl": "."
  }
}
```
上面示例中，baseUrl是一个点，表示基准目录就是tsconfig.json所在的目录。
（2）paths
paths字段指定非相对路径的模块与实际脚本的映射。
```
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "jquery": ["node_modules/jquery/dist/jquery"]
    }
  }
}
```
上面示例中，加载模块jquery时，实际加载的脚本是node_modules/jquery/dist/jquery，它的位置要根据baseUrl字段计算得到。
注意，上例的jquery属性的值是一个数组，可以指定多个路径。如果第一个脚本路径不存在，那么就加载第二个路径，以此类推。
（3）rootDirs
rootDirs字段指定模块定位时必须查找的其他目录。
```
{
  "compilerOptions": {
    "rootDirs": ["src/zh", "src/de", "src/#{locale}"]
  }
}
```
上面示例中，rootDirs指定了模块定位时，需要查找的不同的国际化目录。
### 命名空间
namespace 是一种将相关代码组织在一起的方式，中文译为“命名空间”。
它出现在 ES 模块诞生之前，作为 TypeScript 自己的模块格式而发明的。但是，自从有了 ES 模块，官方已经不推荐使用 namespace 了。
namespace 用来建立一个容器，内部的所有变量和函数，都必须在这个容器里面使用。
```
namespace Utils {
  function isString(value:any) {
    return typeof value === 'string';
  }

  // 正确
  isString('yes');
}

Utils.isString('no'); // 报错
```
上面示例中，命名空间Utils里面定义了一个函数isString()，它只能在Utils里面使用，如果用于外部就会报错。
如果要在命名空间以外使用内部成员，就必须为该成员加上export前缀，表示对外输出该成员。
```
namespace Utility {
  export function log(msg:string) {
    console.log(msg);
  }
  export function error(msg:string) {
    console.error(msg);
  }
}

Utility.log('Call me');
Utility.error('maybe!');
```
上面示例中，只要加上export前缀，就可以在命名空间外部使用内部成员。
编译出来的 JavaScript 代码如下。
```
var Utility;

(function (Utility) {
  function log(msg) {
    console.log(msg);
  }
  Utility.log = log;
  function error(msg) {
    console.error(msg);
  }
  Utility.error = error;
})(Utility || (Utility = {}));
```
上面代码中，命名空间Utility变成了 JavaScript 的一个对象，凡是export的内部成员，都成了该对象的属性。
这就是说，namespace 会变成一个值，保留在编译后的代码中。这一点要小心，它不是纯的类型代码。
namespace 内部还可以使用import命令输入外部成员，相当于为外部成员起别名。当外部成员的名字比较长时，别名能够简化代码。
```
namespace Utils {
  export function isString(value:any) {
    return typeof value === 'string';
  }
}

namespace App {
  import isString = Utils.isString;

  isString('yes');
  // 等同于
  Utils.isString('yes');
}
```
上面示例中，import命令指定在命名空间App里面，外部成员Utils.isString的别名为isString。
import命令也可以在 namespace 外部，指定别名。
```
namespace Shapes {
  export namespace Polygons {
    export class Triangle {}
    export class Square {}
  }
}

import polygons = Shapes.Polygons;

// 等同于 new Shapes.Polygons.Square()
let sq = new polygons.Square();
```
上面示例中，import命令在命名空间Shapes的外部，指定Shapes.Polygons的别名为polygons。
namespace 可以嵌套。
```
namespace Utils {
  export namespace Messaging {
    export function log(msg:string) {
      console.log(msg);
    }
  }
}

Utils.Messaging.log('hello') // "hello"
```
上面示例中，命名空间Utils内部还有一个命名空间Messaging。注意，如果要在外部使用Messaging，必须在它前面加上export命令。
使用嵌套的命名空间，必须从最外层开始引用，比如Utils.Messaging.log()。
namespace 不仅可以包含实义代码，还可以包括类型代码。
```
namespace N {
  export interface MyInterface{}
  export class MyClass{}
}
```
namespace 与模块的作用是一致的，都是把相关代码组织在一起，对外输出接口。区别是一个文件只能有一个模块，但可以有多个 namespace。由于模块可以取代 namespace，而且是 JavaScript 的标准语法，还不需要编译转换，所以建议总是使用模块，替代 namespace。
如果 namespace 代码放在一个单独的文件里，那么引入这个文件需要使用三斜杠的语法。
```
/// <reference path = "SomeFileName.ts" />
```
namespace 本身也可以使用export命令输出，供其他文件使用。
```
// shapes.ts
export namespace Shapes {
  export class Triangle {
    // ...
  }
  export class Square {
    // ...
  }
}
```
上面示例是一个文件shapes.ts，里面使用export命令，输出了一个命名空间Shapes。
其他脚本文件使用import命令，加载这个命名空间。（和 import 模块语法相同）
```
// 写法一
import { Shapes } from './shapes';
let t = new Shapes.Triangle();

// 写法二
import * as shapes from "./shapes";
let t = new shapes.Shapes.Triangle();
```
不过，更好的方法还是建议使用模块，采用模块的输出和输入。
### 装饰器
装饰器（Decorator）是一种语法结构，用来在定义时修改类（class）的行为。
在语法上，装饰器有如下几个特征。
（1）第一个字符（或者说前缀）是@，后面是一个表达式。
（2）@后面的表达式，必须是一个函数（或者执行后可以得到一个函数）。
（3）这个函数接受所修饰对象的一些相关值作为参数。
（4）这个函数要么不返回值，要么返回一个新对象取代所修饰的目标对象。
举例来说，有一个函数Injectable()当作装饰器使用，那么需要写成@Injectable，然后放在某个类的前面。
```
@Injectable class A {
  // ...
}
```
上面示例中，由于有了装饰器@Injectable，类A的行为在运行时就会发生改变。
下面就是一个最简单的装饰器。
```
function simpleDecorator() {
  console.log('hi');
}

@simpleDecorator
class A {} // "hi"
```
上面示例中，函数simpleDecorator()用作装饰器，附加在类A之上，后者在代码解析时就会打印一行日志。
编译上面的代码会报错，提示没有用到装饰器的参数。现在就为装饰器加上参数，让它更像正式运行的代码。
```
function simpleDecorator(
  value:any,
  context:any
) {
  console.log(`hi, this is ${context.kind} ${context.name}`);
  return value;
}

@simpleDecorator
class A {} // "hi, this is class A"
```
上面的代码就可以顺利通过编译了，代码含义这里先不解释。大家只要理解，类A在执行前会先执行装饰器simpleDecorator()，并且会向装饰器自动传入参数就可以了。
装饰器有多种形式，基本上只要在@符号后面添加表达式都是可以的。下面都是合法的装饰器。
```
@myFunc
@myFuncFactory(arg1, arg2)

@libraryModule.prop
@someObj.method(123)

@(wrap(dict['prop']))
```
注意，@后面的表达式，最终执行后得到的应该是一个函数。
相比使用子类改变父类，装饰器更加简洁优雅，缺点是不那么直观，功能也受到一些限制。所以，装饰器一般只用来为类添加某种特定行为。
```
@frozen class Foo {

  @configurable(false)
  @enumerable(true)
  method() {}

  @throttle(500)
  expensiveMethod() {}
}
```
上面示例中，一共有四个装饰器，一个用在类本身（@frozen），另外三个用在类的方法（@configurable、@enumerable、@throttle）。它们不仅**增加了代码的可读性，清晰地表达了意图，而且提供一种方便的手段，增加或修改类的功能。**
#### 类装饰器
类装饰器的类型描述如下。
```
type ClassDecorator = (
  value: Function,
  context: {
    kind: 'class';
    name: string | undefined;
    addInitializer(initializer: () => void): void;
  }
) => Function | void;
```
类装饰器接受两个参数：value（当前类本身）和context（上下文对象）。其中，context对象的kind属性固定为字符串class。
类装饰器一般用来对类进行操作，可以不返回任何值，请看下面的例子。
```
function Greeter(value, context) {
  if (context.kind === 'class') {
    value.prototype.greet = function () {
      console.log('你好');
    };
  }
}

@Greeter
class User {}

let u = new User();
u.greet(); // "你好"
```
上面示例中，类装饰器@Greeter在类User的原型对象上，添加了一个greet()方法，实例就可以直接使用该方法。
类装饰器可以返回一个函数，替代当前类的构造方法。
类装饰器也可以返回一个新的类，替代原来所装饰的类。
```
function countInstances(value:any, context:any) {
  let instanceCount = 0;

  return class extends value {
    constructor(...args:any[]) {
      super(...args);
      instanceCount++;
      this.count = instanceCount;
    }
  };
}

@countInstances
class MyClass {}

const inst1 = new MyClass();
inst1 instanceof MyClass // true
inst1.count // 1
```
#### 方法装饰器
用来装饰类的方法（method）。它的类型描述如下。
```
type ClassMethodDecorator = (
  value: Function,
  context: {
    kind: 'method';
    name: string | symbol;
    static: boolean;
    private: boolean;
    access: { get: () => unknown };
    addInitializer(initializer: () => void): void;
  }
) => Function | void;
```
方法装饰器会改写类的原始方法，实质等同于下面的操作。
```
function trace(decoratedMethod) {
  // ...
}

class C {
  @trace
  toString() {
    return 'C';
  }
}

// `@trace` 等同于
// C.prototype.toString = trace(C.prototype.toString);
```
如果方法装饰器返回一个新的函数，就会替代所装饰的原始函数。
```
function replaceMethod() {
  return function () {
    return `How are you, ${this.name}?`;
  }
}

class Person {
  constructor(name) {
    this.name = name;
  }

  @replaceMethod
  hello() {
    return `Hi ${this.name}!`;
  }
}

const robin = new Person('Robin');

robin.hello() // 'How are you, Robin?'
```
#### 属性装饰器
用来装饰定义在类顶部的属性（field）
属性装饰器要么不返回值，要么返回一个函数，该函数会自动执行，用来对所装饰属性进行初始化。该函数的参数是所装饰属性的初始值，该函数的返回值是该属性的最终值。
```
function logged(value, context) {
  const { kind, name } = context;
  if (kind === 'field') {
    return function (initialValue) {
      console.log(`initializing ${name} with value ${initialValue}`);
      return initialValue;
    };
  }
}

class Color {
  @logged name = 'green';
}

const color = new Color();
// "initializing name with value green"
```
属性装饰器的返回值函数，可以用来更改属性的初始值。
```
function twice() {
  return initialValue => initialValue * 2;
}

class C {
  @twice
  field = 3;
}

const inst = new C();
inst.field // 6
```
#### getter 装饰器和 setter 装饰器
是分别针对类的取值器（getter）和存值器（setter）的装饰器。
这两个装饰器要么不返回值，要么返回一个函数，取代原来的取值器或存值器。
下面的例子是将取值器的结果，保存为一个属性，加快后面的读取。
```
class C {
  @lazy
  get value() {
    console.log('正在计算……');
    return '开销大的计算结果';
  }
}

function lazy(
  value:any,
  {kind, name}:any
) {
  if (kind === 'getter') {
    return function (this:any) {
      const result = value.call(this);
      Object.defineProperty(
        this, name,
        {
          value: result,
          writable: false,
        }
      );
      return result;
    };
  }
  return;
}

const inst = new C();
inst.value
// 正在计算……
// '开销大的计算结果'
inst.value
// '开销大的计算结果'
```
上面示例中，第一次读取inst.value，会进行计算，然后装饰器@lazy将结果存入只读属性value，后面再读取这个属性，就不会进行计算了。
#### 装饰器的执行顺序
装饰器的执行分为两个阶段。
（1）评估（evaluation）：计算@符号后面的表达式的值，得到的应该是函数。
（2）应用（application）：将评估装饰器后得到的函数，应用于所装饰对象。
也就是说，装饰器的执行顺序是，先评估所有装饰器表达式的值，再将其应用于当前类。
应用装饰器时，顺序依次为方法装饰器和属性装饰器，然后是类装饰器。
### declare关键字
**declare 关键字用来告诉编译器，某个类型是存在的**，可以在当前文件中使用。
它的**主要作用，就是让当前文件可以使用其他文件声明的类型。**举例来说，自己的脚本使用外部库定义的函数，编译器会因为不知道外部函数的类型定义而报错，这时就可以在自己的脚本里面使用**declare关键字，告诉编译器外部函数的类型。**这样的话，编译单个脚本就不会因为使用了外部类型而报错。
declare 关键字可以描述以下类型。

- 变量（const、let、var 命令声明）
- type 或者 interface 命令声明的类型
- class
- enum
- 函数（function）
- 模块（module）
- 命名空间（namespace）

declare 关键字的重要特点是，**它只是通知编译器某个类型是存在的，不用给出具体实现。比如，只描述函数的类型**，不给出函数的实现，如果不使用declare，这是做不到的。
**declare 只能用来描述已经存在的变量和数据结构**，不能用来声明新的变量和数据结构。另外，所有 declare 语句都不会出现在编译后的文件里面。
declare 关键字可以给出**外部变量**的类型描述。
```
declare let x:number;
```
上面示例中，变量x是其他脚本定义的，当前脚本不知道它的类型，编译器就会报错。
这时使用 declare 命令给出它的类型，就不会报错了。
下面的例子是脚本使用浏览器全局对象document。
```
declare var document;
document.title = 'Hello';
```
上面示例中，declare 告诉编译器，变量document的类型是外部定义的（具体定义在 TypeScript 内置文件lib.d.ts）。
declare 关键字可以给出**外部函数**的类型描述。
下面是一个例子。
```
declare function sayHello(
  name:string
):void;

sayHello('张三');
```
上面示例中，declare 命令给出了sayHello()的类型描述，因此可以直接使用它。
declare 给出 **class 类型描述**的写法如下。
```
declare class Animal {
  constructor(name:string);
  eat():void;
  sleep():void;
}
```
下面是一个复杂一点的例子。
```
declare class C {
  // 静态成员
  public static s0():string;
  private static s1:string;

  // 属性
  public a:number;
  private b:number;

  // 构造函数
  constructor(arg:number);

  // 方法
  m(x:number, y:number):number;

  // 存取器
  get c():number;
  set c(value:number);

  // 索引签名
  [index:string]:any;
}
```
**如果想把变量、函数、类组织在一起，可以将 declare 与 module 或 namespace 一起使用。**
```
declare namespace AnimalLib {
  class Animal {
    constructor(name:string);
    eat():void;
    sleep():void;
  }

  type Animals = 'Fish' | 'Dog';
}

// 或者
declare module AnimalLib {
  class Animal {
    constructor(name:string);
    eat(): void;
    sleep(): void;
  }

  type Animals = 'Fish' | 'Dog';
}
```
上面示例中，declare 关键字给出了 module 或 namespace 的类型描述。
declare module 和 declare namespace 里面，加不加 export 关键字都可以。
下面的例子是当前脚本使用了myLib这个外部库，它有方法makeGreeting()和属性numberOfGreetings。
```
let result = myLib.makeGreeting('你好');
console.log('欢迎词：' + result);

let count = myLib.numberOfGreetings;
```
myLib的类型描述就可以这样写。
```
declare namespace myLib {
  function makeGreeting(s:string): string;
  let numberOfGreetings: number;
}
```
**declare 关键字的另一个用途，是为外部模块添加属性和方法时，给出新增部分的类型描述。**
```
import { Foo as Bar } from 'moduleA';

declare module 'moduleA' {
  interface Foo {
    custom: {
      prop1: string;
    }
  }
}
```
上面示例中，从模块moduleA导入了类型Foo，它是一个接口（interface），并将其重命名为Bar，然后用 declare 关键字为Foo增加一个属性custom。这里需要注意的是，虽然接口Foo改名为Bar，但是扩充类型时，还是扩充原始的接口Foo，因为同名 interface 会自动合并类型声明。
如果要**为 JavaScript 引擎的原生对象添加属性和方法，可以使用declare global {}**语法。
```
export {};

declare global {
  interface String {
    toSmallString(): string;
  }
}

String.prototype.toSmallString = ():string => {
  // 具体实现
  return '';
};
```
上面示例中，为 JavaScript 原生的String对象添加了toSmallString()方法。declare global 给出这个新增方法的类型描述。
这个示例第一行的**空导出语句export {}，作用是强制编译器将这个脚本当作模块处理**。这是因为declare global必须用在模块里面。
下面的示例是为 window 对象（类型接口为Window）添加一个属性myAppConfig。
```
export {};

declare global {
  interface Window {
    myAppConfig:object;
  }
}

const config = window.myAppConfig;
```
declare global 只能扩充现有对象的类型描述，不能增加新的顶层类型。
declare 关键字给出** enum 类型描述**的例子如下，下面的写法都是允许的。
```
declare enum E1 {
  A,
  B,
}
```
**我们可以为每个模块脚本，定义一个.d.ts文件，把该脚本用到的类型定义都放在这个文件里面。**但是，更方便的做法是为整个项目，定义一个大的.d.ts文件，在这个文件里面使用declare module定义每个模块脚本的类型。
下面的示例是node.d.ts文件的一部分。
```
declare module "url" {
  export interface Url {
    protocol?: string;
    hostname?: string;
    pathname?: string;
  }

  export function parse(
    urlStr: string,
    parseQueryString?,
    slashesDenoteHost?
  ): Url;
}

declare module "path" {
  export function normalize(p: string): string;
  export function join(...paths: any[]): string;
  export var sep: string;
}
```
上面示例中，url和path都是单独的模块脚本，但是它们的类型都定义在node.d.ts这个文件里面。
使用时，自己的脚本使用三斜杠命令，加载这个类型声明文件。
```
/// <reference path="node.d.ts"/>
```
如果没有上面这一行命令，自己的脚本使用外部模块时，就需要在脚本里面使用 declare 命令单独给出外部模块的类型。
### d.ts 类型声明文件
**单独使用的模块，一般会同时提供一个单独的类型声明文件**（declaration file），把本模块的外部接口的所有类型都写在这个文件里面，便于模块使用者了解接口，也便于编译器检查使用者的用法是否正确。
类型声明文件里面只有类型代码，没有具体的代码实现。它的文件名一般为[模块名].d.ts的形式，其中的d表示 declaration（声明）。
举例来说，有一个模块的代码如下。
```
const maxInterval = 12;

function getArrayLength(arr) {
  return arr.length;
}

module.exports = {
  getArrayLength,
  maxInterval,
};
```
它的类型声明文件可以写成下面这样。
```
export function getArrayLength(arr: any[]): number;
export const maxInterval: 12;
```
除了使用export =，模块输出在类型声明文件中，也可以使用export default表示。
```
// 模块输出
module.exports = 3.142;

// 类型输出文件
// 写法一
declare const pi: number;
export default pi;

// 写法二
declare const pi: number;
export= pi;
```
#### 类型声明文件的使用
下面是一个如何使用类型声明文件的简单例子。有一个类型声明文件types.d.ts。
```
// types.d.ts
export interface Character {
  catchphrase?: string;
  name: string;
}
```
然后，就可以在 TypeScript 脚本里面导入该文件声明的类型。
```
// index.ts
import { Character } from "./types";

export const character:Character = {
  catchphrase: "Yee-haw!",
  name: "Sandy Cheeks",
};
```
类型声明文件也可以包括在项目的 tsconfig.json 文件里面，这样的话，编译器打包项目时，会自动将类型声明文件加入编译，而不必在每个脚本里面加载类型声明文件。比如，moment 模块的类型声明文件是moment.d.ts，使用 moment 模块的项目可以将其加入项目的 tsconfig.json 文件。
```
{
  "compilerOptions": {},
  "files": [
    "src/index.ts",
    "typings/moment.d.ts"
  ]
}
```
#### 类型声明文件的来源
类型声明文件主要有以下三种来源。

- TypeScript 编译器自动生成。
- TypeScript 内置类型文件。
- 外部模块的类型声明文件，需要自己安装。

只要使用编译选项declaration，编译器就会在编译时**自动生成**单独的类型声明文件。
下面是在tsconfig.json文件里面，打开这个选项。
```
{
  "compilerOptions": {
    "declaration": true
  }
}
```
安装 TypeScript 语言时，会同时安装一些**内置的类型声明文件**，主要是内置的全局对象（JavaScript 语言接口和运行环境 API）的类型声明。
这些内置声明文件位于 TypeScript 语言安装目录的lib文件夹内，数量大概有几十个，下面是其中一些主要文件。

- lib.d.ts
- lib.dom.d.ts
- lib.es2015.d.ts

如果开发者想了解全局对象的类型接口（比如 ES6 全局对象的类型），那么就可以去查看这些内置声明文件。
如果项目中使用了外部的某个第三方代码库，那么就需要这个库的类型声明文件。
这时又分成三种情况。
（1）这个库自带了类型声明文件。
一般来说，如果这个库的源码包含了[vendor].d.ts文件，那么就自带了类型声明文件。其中的vendor表示这个库的名字，比如moment这个库就自带moment.d.ts。使用这个库可能需要单独加载它的类型声明文件。
（2）这个库没有自带，但是可以找到社区制作的类型声明文件。
第三方库如果没有提供类型声明文件，社区往往会提供。TypeScript 社区主要使用 [DefinitelyTyped 仓库](https://github.com/DefinitelyTyped/DefinitelyTyped)，各种类型声明文件都会提交到那里，已经包含了几千个第三方库。
这些声明文件都会作为一个单独的库，发布到 npm 的@types名称空间之下。比如，jQuery 的类型声明文件就发布成@types/jquery这个库，使用时安装这个库就可以了。
```
$ npm install @types/jquery --save-dev
```
（3）找不到类型声明文件，需要自己写。
有时实在没有第三方库的类型声明文件，又很难完整给出该库的类型描述，这时你可以告诉 TypeScript 相关对象的类型是any。比如，使用 jQuery 的脚本可以写成下面这样。
```
declare var $:any

// 或者
declare type JQuery = any;
declare var $:JQuery;
```
上面代码表示，jQuery 的$对象是外部引入的，类型是any，也就是 TypeScript 不用对它进行类型检查。
也可以采用下面的写法，将整个外部模块的类型设为any。
```
declare module '模块名';
```
有了上面的命令，指定模块的所有接口都将视为any类型。
#### declare 关键字
类型声明文件里面，变量的类型描述必须使用declare命令，否则会报错，因为变量声明语句是值相关代码。
```
declare let foo:string;
```
interface 类型有没有declare都可以，因为 interface 是完全的类型代码。
```
interface Foo {} // 正确
declare interface Foo {} // 正确
```
类型声明文件里面，顶层可以使用export命令，也可以不用，除非使用者脚本会显式使用export命令输入类型。
```
export interface Data {
  version: string;
}
```
下面是类型声明文件的一些例子。先看 moment 模块的类型描述文件moment.d.ts。
```
declare module 'moment' {
  export interface Moment {
    format(format:string): string;

    add(
      amount: number,
      unit: 'days' | 'months' | 'years'
    ): Moment;

    subtract(
      amount:number,
      unit:'days' | 'months' | 'years'
    ): Moment;
  }

  function moment(
    input?: string | Date
  ): Moment;

  export default moment;
}
```
上面示例中，可以注意一下默认接口moment()的写法。
下面是 D3 库的类型声明文件D3.d.ts。
```
declare namespace D3 {
  export interface Selectors {
    select: {
      (selector: string): Selection;
      (element: EventTarget): Selection;
    };
  }

  export interface Event {
    x: number;
    y: number;
  }

  export interface Base extends Selectors {
    event: Event;
  }
}

declare var d3: D3.Base;
```
#### 模块发布
当前模块如果包含自己的类型声明文件，可以在 package.json 文件里面添加一个types字段或typings字段，指明类型声明文件的位置。
```
{
  "name": "awesome",
  "author": "Vandelay Industries",
  "version": "1.0.0",
  "main": "./lib/main.js",
  "types": "./lib/main.d.ts"
}
```
上面示例中，types字段给出了类型声明文件的位置。
注意，如果类型声明文件名为index.d.ts，且在项目的根目录中，那就不需要在package.json里面注明了。
有时，类型声明文件会单独发布成一个 npm 模块，这时用户就必须同时加载该模块。
```
{
  "name": "browserify-typescript-extension",
  "author": "Vandelay Industries",
  "version": "1.0.0",
  "main": "./lib/main.js",
  "types": "./lib/main.d.ts",
  "dependencies": {
    "browserify": "latest",
    "@types/browserify": "latest",
    "typescript": "next"
  }
}
```
上面示例是一个模块的 package.json 文件，该模块需要 browserify 模块。由于后者的类型声明文件是一个单独的模块@types/browserify，所以还需要加载那个模块。
#### 三斜杠命令
如果类型声明文件的内容非常多，可以拆分成多个文件，然后入口文件使用三斜杠命令，加载其他拆分后的文件。
举例来说，入口文件是main.d.ts，里面的接口定义在interfaces.d.ts，函数定义在functions.d.ts。那么，main.d.ts里面可以用三斜杠命令，加载后面两个文件。
```
/// <reference path="./interfaces.d.ts" />
/// <reference path="./functions.d.ts" />
```
三斜杠命令（///）是一个 TypeScript 编译器命令，用来指定编译器行为。它只能用在文件的头部，如果用在其他地方，会被当作普通的注释。另外，若一个文件中使用了三斜线命令，那么在三斜线命令之前只允许使用单行注释、多行注释和其他三斜线命令，否则三斜杠命令也会被当作普通的注释。
除了拆分类型声明文件，三斜杠命令也可以用于普通脚本加载类型声明文件。
三斜杠命令主要包含三个参数，代表三种不同的命令。

- path
- types
- lib

/// <reference path="" />是最常见的三斜杠命令，告诉编译器在编译时需要包括的文件，常用来声明当前脚本依赖的类型文件。
```
/// <reference path="./lib.ts" />

let count = add(1, 2);
```
上面示例表示，当前脚本依赖于./lib.ts，里面是add()的定义。编译当前脚本时，还会同时编译./lib.ts。编译产物会有两个 JS 文件，一个当前脚本，另一个就是./lib.js。
path参数指定了所引入文件的路径。
types 参数用来告诉编译器当前脚本依赖某个 DefinitelyTyped 类型库，通常安装在node_modules/@types目录。
types 参数的值是类型库的名称，也就是安装到node_modules/@types目录中的子目录的名字。
```
/// <reference types="node" />
```
上面示例中，这个三斜杠命令表示编译时添加 Node.js 的类型库，实际添加的脚本是node_modules目录里面的@types/node/index.d.ts。
可以看到，这个命令的作用类似于import命令。
注意，这个命令只在你自己手写类型声明文件（.d.ts文件）时，才有必要用到，也就是说，只应该用在.d.ts文件中，普通的.ts脚本文件不需要写这个命令。如果是普通的.ts脚本，可以使用tsconfig.json文件的types属性指定依赖的类型库。
/// <reference lib="..." />命令允许脚本文件显式包含内置 lib 库，等同于在tsconfig.json文件里面使用lib属性指定 lib 库。
前文说过，安装 TypeScript 软件包时，会同时安装一些内置的类型声明文件，即内置的 lib 库。这些库文件位于 TypeScript 安装目录的lib文件夹中，它们描述了 JavaScript 语言和引擎的标准 API。
库文件并不是固定的，会随着 TypeScript 版本的升级而更新。库文件统一使用“lib.[description].d.ts”的命名方式，而/// <reference lib="" />里面的lib属性的值就是库文件名的description部分，比如lib="es2015"就表示加载库文件lib.es2015.d.ts。
```
/// <reference lib="es2017.string" />
```
上面示例中，es2017.string对应的库文件就是lib.es2017.string.d.ts。
### 类型运算符
#### keyof运算符
keyof 是一个单目运算符，**接受一个对象类型作为参数，返回该对象的所有键名组成的联合类型**。
```
type MyObj = {
  foo: number,
  bar: string,
};

type Keys = keyof MyObj; // 'foo'|'bar'
```
```
interface T {
  0: boolean;
  a: string;
  b(): void;
}

type KeyT = keyof T; // 0 | 'a' | 'b'
```
由于 JavaScript 对象的键名只有三种类型，所以对于任意对象的键名的联合类型就是string|number|symbol。
```
// string | number | symbol
type KeyT = keyof any;
```
对于没有自定义键名的类型使用 keyof 运算符，返回never类型，表示不可能有这样类型的键名。
```
type KeyT = keyof object;  // never
```
上面示例中，由于object类型没有自身的属性，也就没有键名，所以keyof object返回never类型。
#### in运算符
JavaScript 语言中，in运算符用来确定对象是否包含某个属性名。
```
const obj = { a: 123 };

if ('a' in obj)
  console.log('found a');
```
上面示例中，in运算符用来判断对象obj是否包含属性a。
in运算符的左侧是一个字符串，表示属性名，右侧是一个对象。它的返回值是一个布尔值。
TypeScript 语言的类型运算中，**in运算符有不同的用法，用来取出（遍历）联合类型的每一个成员类型。**
```
type U = 'a'|'b'|'c';

type Foo = {
  [Prop in U]: number;
};
// 等同于
type Foo = {
  a: number,
  b: number,
  c: number
};
```
上面示例中，[Prop in U]表示依次取出联合类型U的每一个成员。
上一小节的例子也提到，[Prop in keyof Obj]表示取出对象Obj的每一个键名。
#### 方括号运算符
方括号运算符（[]）用于取出对象的键值类型，比如T[K]会返回对象T的属性K的类型。
```
type Person = {
  age: number;
  name: string;
  alive: boolean;
};

// Age 的类型是 number
type Age = Person['age'];
```
上面示例中，Person['age']返回属性age的类型，本例是number。
方括号的参数如果是联合类型，那么返回的也是联合类型。
```
type Person = {
  age: number;
  name: string;
  alive: boolean;
};

// number|string
type T = Person['age'|'name'];

// number|string|boolean
type A = Person[keyof Person];
```
方括号运算符的参数也可以是属性名的索引类型。
```
type Obj = {
  [key:string]: number,
};

// number
type T = Obj[string];
```
注意，方括号里面不能有值的运算。
```
// 示例一
const key = 'age';
type Age = Person[key]; // 报错

// 示例二
type Age = Person['a' + 'g' + 'e']; // 报错
```
#### extends...?: 条件运算符 
TypeScript 提供类似 JavaScript 的?:运算符这样的三元运算符，但多出了一个extends关键字。
条件运算符extends...?:可以根据当前类型是否符合某种条件，返回不同的类型。
```
T extends U ? X : Y
```
上面式子中的extends用来判断，类型T是否可以赋值给类型U，即T是否为U的子类型，这里的T和U可以是任意类型。
如果T能够赋值给类型U，表达式的结果为类型X，否则结果为类型Y。
```
// true
type T = 1 extends number ? true : false;
```
上面示例中，1是number的子类型，所以返回true。
下面是另外一个例子。
```
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}

// number
type T1 = Dog extends Animal ? number : string;

// string
type T2 = RegExp extends Animal ? number : string;
```
#### infer 关键字
infer关键字用来定义泛型里面推断出来的类型参数，而不是外部传入的类型参数。
它通常跟条件运算符一起使用，用在extends关键字后面的父类型之中。
```
type Flatten<Type> =
  Type extends Array<infer Item> ? Item : Type;
```
上面示例中，infer Item表示Item这个参数是 TypeScript 自己推断出来的，不用显式传入，而Flatten<Type>则表示Type这个类型参数是外部传入的。Type extends Array<infer Item>则表示，如果参数Type是一个数组，那么就将该数组的成员类型推断为Item，即Item是从Type推断出来的。
一旦使用Infer Item定义了Item，后面的代码就可以直接调用Item了。下面是上例的泛型Flatten<Type>的用法。
```
// string
type Str = Flatten<string[]>;

// number
type Num = Flatten<number>;
```
上面示例中，第一个例子Flatten<string[]>传入的类型参数是string[]，可以推断出Item的类型是string，所以返回的是string。第二个例子Flatten<number>传入的类型参数是number，它不是数组，所以直接返回自身。
如果不用infer定义类型参数，那么就要传入两个类型参数。
```
type Flatten<Type, Item> =
  Type extends Array<Item> ? Item : Type;
```
上面是不使用infer的写法，每次调用Flatten的时候，都要传入两个参数，就比较麻烦。
#### is 运算符
函数返回布尔值的时候，可以使用is运算符，限定返回值与参数之间的关系。
is运算符用来描述返回值属于true还是false。
```
function isFish(
  pet: Fish|Bird
):pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```
上面示例中，函数isFish()的返回值类型为pet is Fish，表示如果参数pet类型为Fish，则返回true，否则返回false。
is**运算符总是用于描述函数的返回值类型，**写法采用parameterName is Type的形式，即左侧为当前函数的参数名，右侧为某一种类型。它返回一个布尔值，表示左侧参数是否属于右侧的类型。
```
type A = { a: string };
type B = { b: string };

function isTypeA(x: A|B): x is A {
  if ('a' in x) return true;
  return false;
}
```
上面示例中，**返回值类型x is A可以准确描述函数体内部的运算逻辑。**
#### 模板字符串
TypeScript 允许使用模板字符串，构建类型。
模板字符串的最大特点，就是内部可以引用其他类型。
```
type World = "world";

// "hello world"
type Greeting = `hello ${World}`;
```
上面示例中，类型Greeting是一个模板字符串，里面引用了另一个字符串类型world，因此Greeting实际上是字符串hello world。
注意，**模板字符串可以引用的类型一共6种，分别是 string、number、bigint、boolean、null、undefined。引用这6种以外的类型会报错。**
```
type Num = 123;
type Obj = { n : 123 };

type T1 = `${Num} received`; // 正确
type T2 = `${Obj} received`; // 报错
```
模板字符串里面引用的类型，如果是一个联合类型，那么它返回的也是一个联合类型，即模板字符串可以展开联合类型。
```
type T = 'A'|'B';

// "A_id"|"B_id"
type U = `${T}_id`;
```
上面示例中，类型U是一个模板字符串，里面引用了一个联合类型T，导致最后得到的也是一个联合类型。
如果模板字符串引用两个联合类型，它会交叉展开这两个类型。
```
type T = 'A'|'B';

type U = '1'|'2';

// 'A1'|'A2'|'B1'|'B2'
type V = `${T}${U}`;
```
上面示例中，T和U都是联合类型，各自有两个成员，模板字符串里面引用了这两个类型，最后得到的就是一个4个成员的联合类型。
#### satisfies运算符
s**atisfies运算符用来检测某个值是否符合指定类型。**有时候，不方便将某个值指定为某种类型，但是希望这个值符合类型条件，这时候就可以用satisfies运算符对其进行检测
变量palette的类型被指定为Record<Colors, string|RGB>，这是一个类型工具，用来返回一个对象，详细介绍见《类型工具》一章。简单说，它的第一个类型参数指定对象的属性名，第二个类型参数指定对象的属性值。
可以使用satisfies运算符，对palette进行类型检测，但是不改变 TypeScript 对palette的类型推断。
```
type Colors = "red" | "green" | "blue";
type RGB = [number, number, number];

const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  bleu: [0, 0, 255] // 报错
} satisfies Record<Colors, string|RGB>;

const greenComponent = palette.green.substring(1); // 不报错
```
上面示例中，变量palette的值后面增加了satisfies Record<Colors, string|RGB>，表示该值必须满足Record<Colors, string|RGB>这个条件，所以能够检测出属性名bleu的拼写错误。同时，它不会改变palette的类型推断，所以，TypeScript 知道palette.green是一个字符串，对其调用substring()方法就不会报错。
satisfies也可以检测属性值。
```
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0] // 报错
} satisfies Record<Colors, string|RGB>;
```
上面示例中，属性blue的值只有两个成员，不符合元组RGB必须有三个成员的条件，从而报错了。
### 类型映射
映射（mapping）指的是，**将一种类型按照映射规则，转换成另一种类型，通常用于对象类型。**
举例来说，现有一个类型A和另一个类型B。
```
type A = {
  foo: number;
  bar: number;
};

type B = {
  foo: string;
  bar: string;
};
```
上面示例中，这两个类型的属性结构是一样的，但是属性的类型不一样。如果属性数量多的话，逐个写起来就很麻烦。
使用类型映射，就可以从类型A得到类型B。
```
type A = {
  foo: number;
  bar: number;
};

type B = {
  [prop in keyof A]: string;
};
```
上面示例中，类型B采用了属性名索引的写法，[prop in keyof A]表示依次得到类型A的所有属性名，然后将每个属性的类型改成string。
在语法上，[prop in keyof A]是一个属性名表达式，表示这里的属性名需要计算得到。具体的计算规则如下：

- prop：属性名变量，名字可以随便起。
- in：运算符，用来取出右侧的联合类型的每一个成员。
- keyof A：返回类型A的每一个属性名，组成一个联合类型。

下面是复制原始类型的例子。
```
type A = {
  foo: number;
  bar: string;
};

type B = {
  [prop in keyof A]: A[prop];
};
```
上面示例中，类型B原样复制了类型A。
为了增加代码复用性，可以把常用的映射写成泛型。
```
type ToBoolean<Type> = {
  [Property in keyof Type]: boolean;
};
```
上面示例中，定义了一个泛型，可以将其他对象的所有属性值都改成 boolean 类型。
TypeScript 4.1 引入了键名重映射（key remapping），允许改变键名。
```
type A = {
  foo: number;
  bar: number;
};

type B = {
  [p in keyof A as `${p}ID`]: number;
};

// 等同于
type B = {
  fooID: number;
  barID: number;
};
```
### TS 类型工具
TypeScript 提供了一些内置的类型工具，用来方便地处理各种类型，以及生成新的类型。
TypeScript 内置了17个类型工具，可以直接使用。
ConstructorParameters<Type>提取构造方法Type的参数类型，组成一个元组类型返回。
```
type T1 = ConstructorParameters<
  new (x: string, y: number) => object
>; // [x: string, y: number]

type T2 = ConstructorParameters<
  new (x?: string) => object
>; // [x?: string | undefined]
```
### TS 注释指令
// @ts-nocheck告诉编译器不对当前脚本进行类型检查，可以用于 TypeScript 脚本，也可以用于 JavaScript 脚本。
```
// @ts-nocheck

const element = document.getElementById(123);
```
上面示例中，document.getElementById(123)存在类型错误，但是编译器不对该脚本进行类型检查，所以不会报错。
如果一个 JavaScript 脚本顶部添加了// @ts-check，那么编译器将对该脚本进行类型检查，不论是否启用了checkJs编译选项。
```
// @ts-check
let isChecked = true;

console.log(isChceked); // 报错
```
// @ts-ignore告诉编译器不对下一行代码进行类型检查，可以用于 TypeScript 脚本，也可以用于 JavaScript 脚本。
```
let x:number;

x = 0;

// @ts-ignore
x = false; // 不报错
```
// @ts-expect-error主要用在测试用例，当下一行有类型错误时，它会压制 TypeScript 的报错信息（即不显示报错信息），把错误留给代码自己处理。
```
function doStuff(abc: string, xyz: string) {
  assert(typeof abc === "string");
  assert(typeof xyz === "string");
  // do some stuff
}

expect(() => {
  // @ts-expect-error
  doStuff(123, 456);
}).toThrow();
```
TypeScript 直接处理 JS 文件时，如果无法推断出类型，会使用 JS 脚本里面的** JSDoc 注释**。
使用 JSDoc 时，有两个基本要求。
（1）JSDoc 注释必须以/**开始，其中星号（*）的数量必须为两个。若使用其他形式的多行注释，则 JSDoc 会忽略该条注释。
（2）JSDoc 注释必须与它描述的代码处于相邻的位置，并且注释在上，代码在下。
下面是 JSDoc 的一个简单例子。
```
/**
 * @param {string} somebody
 */
function sayHello(somebody) {
  console.log('Hello ' + somebody);
}
```
上面示例中，注释里面的@param是一个 JSDoc 声明，表示下面的函数sayHello()的参数somebody类型为string。
TypeScript 编译器支持大部分的 JSDoc 声明
@type命令定义变量的类型。
```
/**
 * @type {string}
 */
let a;
```
@param命令用于定义函数参数的类型。
```
/**
 * @param {string}  x
 */
function foo(x) {}
```
如果是可选参数，需要将参数名放在方括号[]里面。
```
/**
 * @param {string}  [x]
 */
function foo(x) {}
```
方括号里面，还可以指定参数默认值。
```
/**
 * @param {string} [x="bar"]
 */
function foo(x) {}
```
上面示例中，参数x的默认值是字符串bar。
@return和@returns命令的作用相同，指定函数返回值的类型。
```
/**
 * @return {boolean}
 */
function foo() {
  return true;
}

/**
 * @returns {number}
 */
function bar() {
  return 0;
}
```
@extends命令用于定义继承的基类。
```
/**
 * @extends {Base}
 */
class Derived extends Base {
}
```
@public、@protected、@private分别指定类的公开成员、保护成员和私有成员。
@readonly指定只读成员。
```
class Base {
  /**
   * @public
   * @readonly
   */
  x = 0;

  /**
   *  @protected
   */
  y = 0;
}
```
###  tsconfig.json文件
tsconfig.json是 TypeScript 项目的配置文件，放在项目的根目录。反过来说，如果一个目录里面有tsconfig.json，TypeScript 就认为这是项目的根目录。
如果项目源码是 JavaScript，但是想用 TypeScript 处理，那么配置文件的名字是jsconfig.json，它跟tsconfig的写法是一样的。
tsconfig.json文件主要供tsc编译器使用，它的命令行参数--project或-p可以指定tsconfig.json的位置（目录或文件皆可）。
```
$ tsc -p ./dir
```
如果不指定配置文件的位置，tsc就会在当前目录下搜索tsconfig.json文件，如果不存在，就到上一级目录搜索，直到找到为止。
tsconfig.json文件的格式，是一个 JSON 对象，最简单的情况可以只放置一个空对象{}。下面是一个示例。
```
{
  "compilerOptions": {
    "outDir": "./built",
    "allowJs": true,
    "target": "es5"
  },
  "include": ["./src/**/*"]
}
```
本章后面会详细介绍tsconfig.json的各个属性，这里简单说一下，上面示例的四个属性的含义。

- include：指定哪些文件需要编译。
- allowJs：指定源目录的 JavaScript 文件是否原样拷贝到编译后的目录。
- outDir：指定编译产物存放的目录。
- target：指定编译产物的 JS 版本。

tsconfig.json文件可以不必手写，使用 tsc 命令的--init参数自动生成。

