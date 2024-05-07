### 1 call 和 apply 的区别是什么，哪个性能好一些
```
fn.call(obj, 10, 20, 30)
fn.apply(obj, [10, 20, 30])
```

- call和apply都是Function 原型上的方法，每一个函数作为 Function的实例都可以调用这两个方法，他们的目的都是用来改变函数中的 this 指向的，让函数执行并改变函数中 this 的指向。
- 两者唯一的区别就是，call 的入参是要把传给 fn的参数一个个传递，而apply则把要把传给 fn的参数以数组的形式传递，apply 的入参只能是两个：对象和参数数组。
- bind函数也是用来改变 this 指向的，只不过它不会立即执行，只是预先把函数中的 this 进行处理。

（有大神测试过）call的性能比 apply 稍微好一些（尤其是 传递给函数的参数 超过三个的时候，三个以下差别不大）所以后期开发的时候可以使用 call 多一些：fn.call(obj, ...[10, 20, 30])
自己实现性能测试（结果只能作为参考：因为任何的代码性能测试都是和测试环境有关系的，例如 CPU、内存、GPU 等电脑当前性能不会有相同的情况，不同浏览器也会导致性能上的不同）
console.time可以测试出一段程序的执行时间
```javascript
console.time('A')
for(let i = 0; i <= 1000000; i++){}
console.timeEnd('A') // ‘A’:6.984ms
```
### 2 实现 (5).add(3).minus(2) 结果为 6
>  arr.push(10)  arr是 Array 类的实例，可以调用 Array.prototype上的方法，而 push就是其中一个
>  1 扩展Number类的原型方法: 实现实例调用方法，把该方法加到实例的原型上
> 2 链式写法，每个方法都需返回这个类的实例
> 注意： eval(item);  // 把字符串当做脚本代码执行 并且是在全局作用域执行

```javascript
~function () {

  function check(n) {
    n = Number(n); //数字或者NaN
    return isNaN(n) ? 0 : n;
  }
  
  function add(n) {
    n = check(n);
    // this+=n 报错 因为this不能直接被赋值 只能用 call 和 apply 改变指向
  	return this + n;
  }

  function minus(n) {
    n = check(n);
  	return this - n;
  }

  Number.prototype.add = add;
  Number.prototype.minus = minus;

  // ['add', 'minus'].forEach(item => {
  //   Number.prototype[item] = eval(item);
  // });
}();

console.log((5).add(3).minus(2)); // 6
```
### 3 箭头函数和普通函数的区别
1 箭头函数语法上更简洁。
2 箭头函数**没有自己的 this**，它的 this是继承函数所处上下文中的 this，使用 call/apply 等任何方式都无法改变 this 的指向。
3 箭头函数中**没有 arguments 类数组**，只能基于...arg（更好用）获取传递的参数集合（数组）(...arg) => {arg为数组}
4 箭头函数不能new,因为箭头函数没有this 也**没有prototype**（没有构造函数）
```javascript
// 更简洁
let fn = x => y => x+y

function fn (x) {
  return function (y) {
    return x + y
  }
}

fn(1)(2) // 3
```
没有自己的 this
```javascript
function fn1 {
  // window
  console.log(this); 
}

let obj = {
  name: 'BOB',
}

fn1.call(obj); // obj

let fn2 = () => {
  // window
  console.log(this)
}

fn2.call(obj) // window

document.body.onclick = () => {
  // this =》 window 不是当前的 body了
}

document.body.onclick = function () {
  // this => body
  arr.sort(function(a, b) {
    // 回调函数中的 this 一般都是 widow
    return a - b
  })

  arr.sort((a, b) => {
    // this => 上下文的 this 也就是 body
    return a - b 
  })
}
```
可以中断的foreach
```javascript
// 回调函数 函数在执行的时候，可以把传递进来的回调函数去执行（执行多次，传参，更改 this）
function each(arr, callBack) {
	for(let i = 0; i < arr.length; i++) {
    // callBack(arr[i], i);
    let flag = callBack.call(arr, arr[i], i);
    if(!flag) {
      break; // 结束循环 forEach不支持根据返回值结束循环 可以自己重新写一个方法扩展 array 的原型方法
    }
  }
}

each([10,20,30,40], function (item, index) { // 回调换成箭头函数 上面的函数就不需要调用call了
  // this => Window
  // this => 原始操作数组
  if (index > 1) {
    return false;
  }
})
```
没有 prototype，不能 new
```javascript
function Fn() {
  this.x = 100;
}
Fn.prototype.getX = function() {}
let f = new Fn();

// let Fn = () => {
//    this.x = 200;
// }
// let f = new Fn(); // TypeError: Fn is not a constructor
```
![截屏2024-01-22 下午1.23.57.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1705901041353-7cb5c0bd-17a5-4816-b878-9692e07c0afd.png#averageHue=%23f9f9f9&clientId=u2b8c688e-dbc7-4&from=drop&height=163&id=u0a733a3e&originHeight=248&originWidth=484&originalType=binary&ratio=2&rotation=0&showTitle=false&size=30330&status=done&style=none&taskId=ufa2d1fed-931c-427c-bda2-d3d1fac2094&title=&width=319)
#### 思考题 1 实现可中断的each，并返回新数组
```javascript
// 实现
(function(){
  function each (callBack, obj= window) {
    const arr = this, newArr = [];
    for(let i = 0; i < arr.length; i++) {
        const result = callBack.call(obj, arr[i], i);
        if(!result) break;
        newArr.push(result);
    }
    return newArr;
  }
  Array.prototype.each = each;
})()

// 调用
let arr = [10, 20, 30, 'AA', 40],
  obj = {};
const arrNew = arr.each(function (item, index) {
  // each 第二个参数不传，this 是 window 即可。
  if(isNaN(item)) {return false} // return false则结束循环
  return item * 10;
}, obj) 
// 最终结果 arrNew = [100, 200, 300]
```
#### 思考题 2 重写replace(reg, callback)，实现和内置一样的效果
```javascript
let str = 'zhufeng2019zhufeng2029';
str = str.replace(/zhufeng/g, function(...arg) {
  // arg中存储了每一次大正则匹配的信息和小分组匹配的信息
  return '@'; // 返回啥就替换成啥
});
// ['zhufeng', 0, 'zhufeng2019zhufeng2029']
// ['zhufeng', 11, 'zhufeng2019zhufeng2029']

(function (){
    // 将str中的v1替换成v2
    function swap(str,v1,v2) {
        let index = str.indexOf(v1);
        return str.substring(0,index) + v2 + str.substring(index + v1.length);
    }
    function replace(reg,callback) {
        let isGlobal = reg.global,
            _this = this.substring(0),
            arr = reg.exec(this); // ['1234','1234','4']
        while(arr) {
            if (typeof callback === "function") {
                let result = callback.apply(window, arr);
                _this = swap(_this, arr[0], result);
            }
            arr = reg.exec(this); // ['1233','1233','15']
            if (!isGlobal) { // 没有可以匹配的时候，reg的 global属性会变为 false
                break;
            }
        }
        return _this;
    }
    String.prototype.myreplace = replace;
})();

let msg = "yi1234threeyour1233";
msg = msg.replace(/((\d){4})/g,function(context, group1){
    return "这里有四个数字";
});
console.log(msg);
//yi这里有四个数字threeyour这里有四个数字

```
### 4 把一个字符串的大小写取反，Abc变aBC
```javascript
let str = 'hkjsHSNaj哈哈哈123Ak啊'
str = str.replace(/[a-zA-Z]/g, content => {
  // content 每一次正则匹配的结果 h(第1次) k(第2次)
  return content.toUpperCase() === content ? content.toLowerCase() : content.toUpperCase();
})
// 验证是否大写
// 方法一： 转化为大写后看是否和原本的值一致
// 方法二： ASCII 表中找到大写字符的取值范围 用charCodeAt()进行判断（65-90） 
        // content.charCodeAt() >= 65 && content.charCodeAt() <= 90
                    
```
### 5 实现一个字符串匹配算法 从字符串 S 中查找是否存在字符串 T，存在返回位置，否则-1.不可用 indexOf/includes实现
```javascript
~function() {
	// 方法一 循环
  function myIndexOf(T) {
    // this => S
    let res = -1;
    for(let i = 0; i <= this.length - T.length; i++) {
      if(this.substr(i, T.length) === T)  // 从i开始向后截取n个 区分substring
        res = i;
        break;
      }
    }
    return res;
  }
  
  // 方法二 正则处理
  function myIndexOf(T) {
    let reg = new RegExp(T),
      res = reg.exec(this); // res是数组
    return res === null ? -1 : res.index;
  }
  
  String.prototype.myIndexOf = myIndexOf;
}()
```
![截屏2024-01-22 下午8.32.47.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1705926771659-90d04db7-d51b-4eb4-86b9-52561f5184d6.png#averageHue=%23fcfafa&clientId=ub2cf3ac2-654c-4&from=drop&id=ued7c5d9f&originHeight=326&originWidth=1048&originalType=binary&ratio=2&rotation=0&showTitle=false&size=56435&status=done&style=none&taskId=ub699a7f9-2cac-46a2-acf4-47a202e7e21&title=)
### 6 对象的属性赋值练习题
```javascript
// 对象的数字属性会被自动转换为字符串
var a = {}, b = 123, c = '123';
a[b] = 'b'
a[c] = 'c'
console.log(a[b]) // c

// 对象的Symbol属性唯一
var a = {}, b = Symbol('123'), c = Symbol('123');
a[b] = 'b'
a[c] = 'c'
console.log(a[b]) // b

// object
var a = {}, b = {key: '123'}, c = {key: '456'};
a[b] = 'b' // obj["[object Object]"] = 'b'
a[c] = 'c'  // => obj: {'[object Object]': 'c'}
console.log(a[b]) // c
```
1 对象的属性如果是 number会被自动转换为字符串
2 对象的属性名不能是一个对象，如果遇到对象，则会被转化为字符串（属性名遇到对象会默认转化为字符串toString）
```javascript
obj = {}
arr = [22,34]
obj[arr] = 'name'
// => obj: {'22,34': 'name'}
```
注意：普通对象 toString()调用的是 Object.prototype上的方法。返回值是数据类型 '[object Object]'
Object.prototype.toString.call() 是用来判断数据类型的
###  7 判断字符串是一个正确的网址
```javascript
let str = "http://www.zhufengpeixun.cn/index.html?x=1&from=wx#video";
```
url格式：

1. 协议 http/https/ftp
2. 域名（必需） www.bai.qq.com.cn
3. 请求路径 /  /index.html  /stu/
4. 问号传参   ?x=1&from=wx
5. 哈希值   #video"
```javascript
((http|https|ftp):\/\/)?     // 协议 ://
(([\w-]+\.)+[a-z0-9]+)       //域名 ～.~  [\w-]字母数字下划线和-号
((\/[^/]*)+)?                // 请求路径 斜杠加非斜杠 *表示0个或多个
(\?[^#]+)?									// 问号传参
(#.+)?											// 哈希值

/^ $/i                        // i 表示不区分大小写 

所以
let str = "http://www.zhufengpeixun.cn/index.html?x=1&from=wx#video";
let reg = /^((http|https|ftp):\/\/)?(([\w-]+\.)+[a-z0-9]+)((\/[^/]*)+)?(\?[^#]+)?(#.+)?$/i;
console.log(reg.test(str))
  
```
从左往右数，第几个括号就是第几个分组，对应下面数组的下标
![截屏2024-01-22 下午10.24.56.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1705933500822-c998064d-3349-4c06-87bb-8caefb8c9368.png#averageHue=%23fbf6f6&clientId=ub2cf3ac2-654c-4&from=drop&id=u3b1c78a9&originHeight=570&originWidth=1018&originalType=binary&ratio=2&rotation=0&showTitle=false&size=124867&status=done&style=none&taskId=u055ef57c-efd9-43c0-99a8-4641bf6cfe0&title=)
```javascript
// (?: 表示只匹配不捕获 即不进行分组
reg = /^(?:(http|https|ftp):\/\/)?(?:([\w-]+\.)+[a-z0-9]+)((\/[^/]*)+)?(\?[^#]+)?(#.+)?$/i;
reg.exec(str)
```
![截屏2024-01-22 下午10.29.27.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1705933771130-ce9408cc-1560-4153-addb-a86ff77079b7.png#averageHue=%23fbf6f6&clientId=ub2cf3ac2-654c-4&from=drop&id=u5cba5590&originHeight=534&originWidth=1274&originalType=binary&ratio=2&rotation=0&showTitle=false&size=138389&status=done&style=none&taskId=ube8a17bb-587b-4c67-9452-8da4a336283&title=)
```javascript
// ((?:\/[^/?#]*)+)? 非/非？非# 里面的括号也去掉不要了
reg = /^(?:(http|https|ftp):\/\/)?(?:([\w-]+\.)+[a-z0-9]+)((?:\/[^/?#]*)+)?(\?[^#]+)?(#.+)?$/i;
reg.exec(str)
```
![截屏2024-01-22 下午10.34.00.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1705934044579-7c40f341-1880-4d9b-8263-7bbccb3d78ee.png#averageHue=%23fcf7f7&clientId=ub2cf3ac2-654c-4&from=drop&id=u0d92994b&originHeight=508&originWidth=1338&originalType=binary&ratio=2&rotation=0&showTitle=false&size=125411&status=done&style=none&taskId=ub07afac3-573e-4a9b-98e0-36c68226d84&title=)
```javascript
str = "www.zhufengpeixun.cn/?x=1&from=wx#video"
reg = /^(?:(http|https|ftp):\/\/)?(?:([\w-]+\.)+[a-z0-9]+)((?:\/[^/?#]*)+)?(\?[^#]+)?(#.+)?$/i;
reg.exec(str)
```
![截屏2024-01-22 下午10.36.19.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1705934183675-dca71a62-068a-4305-baba-439982249404.png#averageHue=%23fbf7f7&clientId=ub2cf3ac2-654c-4&from=drop&id=u6bfee724&originHeight=558&originWidth=1362&originalType=binary&ratio=2&rotation=0&showTitle=false&size=122715&status=done&style=none&taskId=u8b1d8c80-5bbd-4241-908b-3169337f405&title=)
```javascript
// 最终版正则表达式如下
reg = /^(?:(http|https|ftp):\/\/)?(?:([\w-]+\.)+[a-z0-9]+)((?:\/[^/?#]*)+)?(\?[^#]+)?(#.+)?$/i;
reg.exec(str)
```
### 继承相关 填空题
```javascript
function Foo() {
  Foo.a = function () {
    console.log(1);
  }
  this.a = function () {
    console.log(2);
  }
}
// Foo类，在原型上设置实例的公有的属性方法 =》实例.a()
Foo.prototype.a = function () {
    console.log(3);
}
// =》把 Foo当做普通对象，设置私有属性a =》 Foo.a()
Foo.a = function () {
    console.log(4);
}

Foo.a(); // 4
let obj = new Foo(); // new的时候执行函数 Foo,为this也就是obj设置了私有属性a； 并且修改了Foo对象的私有属性 a
obj.a(); // 2  直接在私有属性上找到了 a，就不会去原型上找了
Foo.a(); // 1  new的时候执行函数了一遍构造函数
```
### 图片懒加载
完全滚动到视口中的时候再加载图片
```javascript
// imgBox 用来占位 img src没有加载的时候，展示 background
// <body>
  // <div class="imgBox">
  //   <img src="" alt="" data-img="http://a.com/img/a.jpg">
  // </div>

//   <!-- import js -->
//   <script src="node_modules/jquery/dist/jquery.min.js"></script>
//   <script src="delayImg.js"></script>
// </body>

// jQuery 单张图片懒加载
let $imgBox = $('.imgBox'),
  $img = $imgBox.children('img'),
  $window = $(window);

// jQuery中的事件支持 多事件绑定
$window.on('load scroll', function() {
  if($img.attr('isLoad') === 'true') return; // 加载过的无需再次加载
  
  let $A = $imgBox.outerHeight() + $imgBox.offset().top, // 图片底边距离页面顶部的距离 outerHeight自身高度
    $B = $window.outerHeight() + $window.scrollTop(); // 浏览器窗口底边距离页面顶部的距离
  
  if($A < $B) {
    // => 滚动到视口中了 加载真实图片 真实的src url写在'data-img'属性上了
    $img.attr('src', $img.attr('data-img'));
    $img.on('load', function () {
      // => 监听$img图片有没有加载成功
      // $img.css('display', 'block');
      $img.stop().fadeIn(); //fadeIn 增加逐渐出现动画
    });
    $img.attr('isLoad', 'true'); // 增加自定义属性 标识已经加载过了 无需再次加载
  }
});

```
多张图片懒加载
```javascript

// jQuery 多张图片懒加载 imgBox都放在 container 里面
let $container = $('.container'),
  $imgBoxs = null,
  $window = $(window);

let str = ''; 
// array中存储真实的 src 此处用 null mock
new Array(20).fill(null).forEach(item => {
  str+= `  <div class="imgBox">
    <img src="" alt="" data-img="http://a.com/img/a.jpg">
  </div>`;
});
$container.html(str); // 给 container 增加元素imgBox div
$imgBoxs = $container.children('.imgBox');

$window.on('load scroll', function() {
  // 浏览器底边框距离 body 的距离
  $B = $window.outerHeight() + $window.scrollTop();
  // 循环判断每一个图片，计算出是否需要加载
  $imgBoxs.each((index, item) => {
    let $item = $(item),
      $itemA = $item.outerHeight() + $item.offset().top,
      isLoad = $item.attr('isLoad');
    if($itemA <= $B && isLoad !== 'true') {
      // 当前区域中的图片需要加载
       $img = $imgBox.children('img'),
       $img.attr('src', $img.attr('data-img'));
       $img.on('load', () => $img.stop().fadeIn()); // 没有用到 this就可以用箭头函数
       $img.attr('isLoad', 'true'); 
    }
  })
});
```
### 8 密码合格校验：6～16位的字符串，必需同时包含大写小写字母和数字
正向预查 (?=pattern)  必需符合 pattern 这个条件 条件不参与捕获
```javascript
reg = /cainiao(?=8)/  
reg.exec('cainiao8')  // 匹配到cainiao  因为条件 8不参与捕获
reg.exec('cainiao9')  // null
```
负向预查 (?!pattern)   必需不符合 pattern 这个条件 条件不参与捕获
```javascript
reg = /cainiao(?！8)/  
reg.exec('cainiao8')  // null
reg.exec('cainiao9')  // 匹配到cainiao  因为条件 8不参与捕获
```
```javascript
var reg1 = /(?=^)\d{2}(?=$)/;  左边要满足是开头，右边要满足是结尾 只是为了说明，实际会用 reg2
var reg2 = /^\d{2}$/; 
// 两者的意思是相同的 描述2位数字的字符串
```
所以：
```javascript
let reg = /(^?![a-z]+$)(^?![A-Z]+$)(^?![0-9]+$)^[a-zA-Z0-9]{6,16}$/;  
(^?![a-z]+$) 不能全是小写
(^?![0-9]+$) 不能全是数字

reg = /(^?![a-zA-Z]+$)(^?![0-9]+$)^[a-zA-Z0-9]{6,16}$/; 
(^?![a-zA-Z]+$) 纯大小写字母不行 
(^?![0-9]+$)    纯数字不行

最终版本：利用排除法 负向预查条件
reg = /(^?![a-zA-Z]+$)(^?![0-9]+$)(^?![a-z0-9]+$)(^?![A-Z0-9]+$)^[a-zA-Z0-9]{6,16}$/; 
(^?![a-z0-9]+$)  小写+数字 不行
(^?![A-Z0-9]+$)  大写+数字 不行

```
扩展：必须是1-10 位数字字母下划线，且需有下划线
```javascript
reg = /(?!^[a-zA-Z0-9]+$)^\w{1,10}$/
```
扩展：包含 \w (数字或字母或下划线)，但是必需有下划线
```javascript
reg = /(?=_)\w+/
```
### 9 实现$attr(name,  value)遍历： 属性为name值为value的元素集合
```javascript
function $attr(property, value) {
  // 页面中所有的标签
  let elements = document.getElementsByTagName('*'),
    arr = [];
  // [].forEach.call(elements, item => {})
  elements = Array.from(elements);
  elements.forEach(item => {
    // 当前元素的property 对应的属性值
    let itemValue = item.getAttribute(property);
    if(property === 'class') {
      // class属性特殊处理 \b 单词边界
      new RegExp("\\b" + value + "\\b").test(itemValue) ? arr.push(item) : null;
      return;
    }
    if(itemValue === value) {
      arr.push(item);
    }
  })
  return arr;
}

let ary = $attr('class', 'box')
```
### 10 英文字母汉字组成的字符串 给英文单词前后加空格
```javascript
let str = "jk健康jds你好kl啊啊j呀",
  reg = /\b[a-z]+\b/ig;
str = str.replace(reg, value => {
  return " " + value + " ";
}).trim();
console.log(str);
```
### 11 数组扁平化 并去除其中重复部分数据 最终得到一个升序且不重复的数组
```javascript
let arr = [[1,2,2], [3,4,5,5], [6,7,8,9,[11,12,[12,13,[14]]]], 10];
```
方法1： ES6 新增数组扁平化 flat 方法  Array.prototype.flat 参数是 depth
```javascript
// 扁平化
arr = arr.flat(Infinity);
// 去重 new Set(arr) 是个对象 [...new Set(arr)]  Array.from(new Set(arr))
// 排序  
arr = [...new Set(arr)].sort((a, b) => a - b)

//总结 一行代码
arr = Array.from(new Set(arr.flat(Infinity))).sort((a,b) => a-b)
```
##### 补充：数组去重
>    1. 数组作为对象的属性
>    2. for循环+include
>    3. new Set（）
>    4. 利用 reduce+includes
> 
function unique(arr){
    return arr.reduce((prev,cur) => prev.includes(cur) ? prev : [...prev,cur],[]);
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr));
// [1, "true", true, 15, false, undefined, null, NaN, "NaN", 0, "a", {…}, {…}]

> 1 数组作为对象的属性
> function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return
    }
    var arrry= [];
    var  obj = {};
    for (var i = 0; i < arr.length; i++) {
        if (!obj[arr[i]]) {
            arrry.push(arr[i])
            obj[arr[i]] = 1
        } else {
            obj[arr[i]]++
        }
    }
    return arrry;
}

var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
//[1, "true", 15, false, undefined, null, NaN, 0, "a", {…}]    //两个true直接去掉了，NaN和{}去重

> 4 暴力解法：两重 for循环
> function unique(arr){            
        for(var i=0; i<arr.length; i++){
            for(var j=i+1; j<arr.length; j++){
                if(arr[i]==arr[j]){         //第一个等同于第二个，splice方法删除第二个
                    arr.splice(j,1);
                    j--;
                }
            }
        }
return arr;
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
//[1, "true", 15, false, undefined, NaN, NaN, "NaN", "a", {…}, {…}]     //NaN和{}没有去重，两个null直接消失了

方法2：把数组直接 toString 变成字符串
```javascript
let arr = [[1,2,2], [3,4,5,5], [6,7,8,9,[11,12,[12,13,[14]]]], 10];
arr = arr.toString().split(',').map(item => Number(item))

arr.join().split(',') // 成功

// join 也可以 split的时候把,和｜都进行拆分
arr.join('|').split(/(?:,|\|)/g)
```
![截屏2024-01-23 下午6.15.47.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1706004951674-220b80d1-923b-49a0-b256-1634b89c1343.png#averageHue=%23faf7f7&clientId=ub2cf3ac2-654c-4&from=drop&id=u6cae9ae6&originHeight=540&originWidth=1394&originalType=binary&ratio=2&rotation=0&showTitle=false&size=103603&status=done&style=none&taskId=uaf3c1817-975f-4388-8ff7-2b04d823552&title=)

方法3：JSON.stringify(arr)       JSON.parse()
```javascript
arr = JSON.stringify(arr).replace(/(\[|\])/g, '').split(',').map(item => Number(item))
```
![截屏2024-01-23 下午6.52.39.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1706007164590-3ed10d5f-612d-42a0-aa4b-d52356ee29d6.png#averageHue=%23f7f2f2&clientId=ub2cf3ac2-654c-4&from=drop&height=103&id=u76eb798e&originHeight=176&originWidth=812&originalType=binary&ratio=2&rotation=0&showTitle=false&size=35808&status=done&style=none&taskId=ue4846dd5-7fb3-4b9b-9a3b-d50f3298acd&title=&width=476)

方法4： ...展开运算符 + concat
```javascript
while(arr.some(item => Array.isArray(item))) {
  arr = [].concat(...arr);
}
```

方法5： 递归 （面试官更喜欢问递归）
```javascript
~function () {
  function myFlat() {
    let result = [],
      _this = this;
    // 循环数组的每一项 把不是数组的 push到结果数组中
    let fn = (arr) => {
      for(let i = 0; i < arr.length; i++) {
        let item = arr[i];
        if(Array.isArray(item)) {
          fn(item);
          continue;
        }
        result.push(item)
      }
    };
    fn(_this);
    return result;
  }
  Array.prototype.myFlat = myFlat;
}();
arr = arr.myFlat();
```
### 继承和原型链
当谈到继承时，JavaScript 只有一种结构：对象。每个对象（object）都有一个私有属性原型[[prototype]]指向另一个名为**原型**（prototype）的对象。原型对象也有一个自己的原型，层层向上直到一个对象的原型为 null。根据定义，null 没有原型，并作为这个**原型链**（prototype chain）中的最后一个环节。
当试图**访问一个对象的属性**时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。
> **备注：** 遵循 ECMAScript 标准，符号 someObject.[[Prototype]] 用于标识 someObject 的原型。内部插槽 [[Prototype]] 可以通过 [Object.getPrototypeOf()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf) 和 [Object.setPrototypeOf()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) 函数来访问。这个等同于 JavaScript 的非标准但被许多 JavaScript 引擎实现的属性 [__proto__](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/proto) 访问器。为在保持简洁的同时避免混淆，在我们的符号中会避免使用 obj.__proto__，而是使用 obj.[[Prototype]] 作为代替。其对应于 Object.getPrototypeOf(obj)。
> **警告：** Object.prototype.__proto__ 访问器是**非标准**的，且已被弃用。你几乎总是应该使用 Object.setPrototypeOf 来代替。
> 它不应与函数的 func.prototype 属性混淆，后者**指定在给定函数被用作构造函数**时**分配给所有对象**_**实例 **_**的 ****[[Prototype]]**。我们将在[后面的小节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain#%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)中讨论构造函数的原型属性。
> 对象字面量初始化器中的 __proto__ 是标准化，被优化的。甚至可以比 [Object.create](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create) 更高效。在创建对象时声明额外的自有属性比 [Object.create](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create) 更符合习惯。

值得注意的是，**{ __proto__: ... } 语法**与 obj.__proto__ 访问器不同：前者是标准且未被弃用的。
```javascript
const o = {
  a: 1,
  b: 2,
  // __proto__ 设置了 [[Prototype]]。它在这里被指定为另一个对象字面量。
  __proto__: {
    b: 3,
    c: 4,
  },
};

// o.[[Prototype]] 具有属性 b 和 c。
// o.[[Prototype]].[[Prototype]] 是 Object.prototype（我们会在下文解释其含义）。
// 最后，o.[[Prototype]].[[Prototype]].[[Prototype]] 是 null。
// 这是原型链的末尾，值为 null，
// 根据定义，其没有 [[Prototype]]。
// 因此，完整的原型链看起来像这样：
// { a: 1, b: 2 } ---> { b: 3, c: 4 } ---> Object.prototype ---> null

console.log(o.a); // 1
// o 上有自有属性“a”吗？有，且其值为 1。

console.log(o.b); // 2
// o 上有自有属性“b”吗？有，且其值为 2。
// 原型也有“b”属性，但其没有被访问。
// 这被称为属性遮蔽（Property Shadowing）

console.log(o.c); // 4
// o 上有自有属性“c”吗？没有，检查其原型。
// o.[[Prototype]] 上有自有属性“c”吗？有，其值为 4。

console.log(o.d); // undefined
// o 上有自有属性“d”吗？没有，检查其原型。
// o.[[Prototype]] 上有自有属性“d”吗？没有，检查其原型。
// o.[[Prototype]].[[Prototype]] 是 Object.prototype 且
// 其默认没有“d”属性，检查其原型。
// o.[[Prototype]].[[Prototype]].[[Prototype]] 为 null，停止搜索，
// 未找到该属性，返回 undefined。

```
继承‘方法’：当继承的函数被调用时，[this](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this) 值指向的是当前继承该对象的对象，而不是拥有该函数属性的原型对象。
```javascript
const parent = {
  value: 2,
  method() {
    return this.value + 1;
  },
};

console.log(parent.method()); // 3
// 当调用 parent.method 时，“this”指向了 parent

// child 是一个继承了 parent 的对象
const child = {
  __proto__: parent,
};
console.log(child.method()); // 3
// 调用 child.method 时，“this”指向了 child。
// 又因为 child 继承的是 parent 的方法，
// 首先在 child 上寻找“value”属性。但由于 child 本身
// 没有名为“value”的自有属性，该属性会在
// [[Prototype]] 上被找到，即 parent.value。

child.value = 4; // 在 child，将“value”属性赋值为 4。
// 这会遮蔽 parent 上的“value”属性。
// child 对象现在看起来是这样的：
// { value: 4, __proto__: { value: 2, method: [Function] } }
console.log(child.method()); // 5
// 因为 child 现在拥有“value”属性，“this.value”现在表示
// child.value
```
构造函数
```javascript
const boxes = [
  { value: 1, getValue() { return this.value; } },
  { value: 2, getValue() { return this.value; } },
  { value: 3, getValue() { return this.value; } },
];
//这是不够好的，因为每一个实例都有自己的，做相同事情的函数属性，这是冗余且不必要的。
//相反，我们可以将 getValue 移动到所有盒子的 [[Prototype]] 上：
const boxPrototype = {
  getValue() {
    return this.value;
  },
};

const boxes = [
  { value: 1, __proto__: boxPrototype },
  { value: 2, __proto__: boxPrototype },
  { value: 3, __proto__: boxPrototype },
];

```
这样，所有盒子的 getValue 方法都会引用相同的函数，降低了内存使用率。但是，手动绑定每个对象创建的 __proto__ 仍旧非常不方便。这时，我们就可以使用_构造函数_，它会自动为每个构造的对象设置 [[Prototype]]。构造函数是使用 [new](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new) 调用的函数。
```javascript
// 一个构造函数
function Box(value) {
  this.value = value;
}

// 使用 Box() 构造函数创建的所有盒子都将具有的属性
Box.prototype.getValue = function () {
  return this.value;
};

const boxes = [new Box(1), new Box(2), new Box(3)];
```
我们说 new Box(1) 是通过 Box 构造函数创建的一个_实例_。Box.prototype 与我们之前创建的 boxPrototype 并无太大区别——它只是一个普通的对象。通过构造函数创建的每一个实例都会自动将构造函数的 [prototype](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype) 属性作为其 [[Prototype]]。
即，Object.getPrototypeOf(new Box()) === Box.prototype。Constructor.prototype 默认具有一个自有属性：[constructor](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor)，它引用了构造函数本身。即，Box.prototype.constructor === Box。这允许我们在任何实例中访问原始构造函数。
上面的构造函数可以重写为[类](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)：
```javascript
class Box {
  constructor(value) {
    this.value = value;
  }

  // 在 Box.prototype 上创建方法
  getValue() {
    return this.value;
  }
}
```
类是构造函数的语法糖，这意味着你仍然可以修改 Box.prototype 来改变所有实例的行为。
因为 Box.prototype 引用了（作为所有实例的 [[Prototype]] 的）相同的对象，所以我们可以通过改变 Box.prototype 来改变所有实例的行为
```javascript
function Box(value) {
  this.value = value;
}
Box.prototype.getValue = function () {
  return this.value;
};
const box = new Box(1);

// 在创建实例后修改 Box.prototype
Box.prototype.getValue = function () {
  return this.value + 1;
};
box.getValue(); // 2
```
**Constructor.prototype **仅在构造实例时有用。它与** Constructor.[[Prototype]] **无关，后者是构造函数的_自有 _原型，即 Function.prototype。也就是说，Object.getPrototypeOf(Constructor) === Function.prototype。
[创建对象的几种方法：](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain#%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%90%8C%E7%9A%84%E6%96%B9%E6%B3%95%E6%9D%A5%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1%E5%92%8C%E6%94%B9%E5%8F%98%E5%8E%9F%E5%9E%8B%E9%93%BE)
1 使用语法结构创建对象
2 使用构造函数创建对象
3 调用 [Object.create()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create) 来创建一个新对象。该对象的 [[Prototype]] 是该函数的第一个参数
4 使用类来创建对象
```javascript
// 1 语法结构
const o = { a: 1 };
// 新创建的对象 o 以 Object.prototype 作为它的 [[Prototype]]
// Object.prototype 的原型为 null。
// o ---> Object.prototype ---> null

const b = ["yo", "whadup", "?"];
// 数组继承了 Array.prototype（具有 indexOf、forEach 等方法）
// 其原型链如下所示：
// b ---> Array.prototype ---> Object.prototype ---> null

function f() {
  return 2;
}
// 函数继承了 Function.prototype（具有 call、bind 等方法）
// f ---> Function.prototype ---> Object.prototype ---> null

const p = { b: 2, __proto__: o };
// 可以通过 __proto__ 字面量属性将新创建对象的
// [[Prototype]] 指向另一个对象。
// （不要与 Object.prototype.__proto__ 访问器混淆）
// p ---> o ---> Object.prototype ---> null

//对象字面量初始化器中的 __proto__ 是标准化，被优化的。甚至可以比 Object.create 更高效



// 2 构造函数
function Graph() {
  this.vertices = [];
  this.edges = [];
}

Graph.prototype.addVertex = function (v) {
  this.vertices.push(v);
};

const g = new Graph();
// g 是一个带有自有属性“vertices”和“edges”的对象。
// 在执行 new Graph() 时，g.[[Prototype]] 是 Graph.prototype 的值。



// 3 Object.create() 
const a = { a: 1 };
// a ---> Object.prototype ---> null

const b = Object.create(a);
// b ---> a ---> Object.prototype ---> null
console.log(b.a); // 1 (inherited)

const c = Object.create(b);
// c ---> b ---> a ---> Object.prototype ---> null

const d = Object.create(null);
// d ---> null（d 是一个直接以 null 为原型的对象）
console.log(d.hasOwnProperty);
// undefined，因为 d 没有继承 Object.prototype



// 4 类
class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}

class Square extends Polygon {
  constructor(sideLength) {
    super(sideLength, sideLength);
  }

  get area() {
    return this.height * this.width;
  }

  set sideLength(newLength) {
    this.height = newLength;
    this.width = newLength;
  }
}

const square = new Square(2);
// square ---> Square.prototype ---> Polygon.prototype ---> Object.prototype ---> null

```
**修改对象的 ****[[Prototype]]**** 内部属性：**
虽然上面的所有方法都会在对象创建时设置原型链，但是 [Object.setPrototypeOf()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) 允许修改现有对象的 [[Prototype]] 内部属性。
```javascript
const obj = { a: 1 };
const anotherObj = { b: 2 };
Object.setPrototypeOf(obj, anotherObj);
// obj ---> anotherObj ---> Object.prototype ---> null
```
此方法性能不佳。如果可以在创建对象时设置原型，则应避免此方法。
### 12 手写 new   3句话
```javascript
// 最清晰版本
function myNew(Con, ...args) {
  // // 1 创建一个新的空对象 Con即 constructor 类
  // let obj = {};
  // // 2 将这个空对象的__proto__（原型）指向构造函数的原型
  // obj.__proto__ = Con.prototype;
  // // Object.setPrototypeOf(obj, Con.prototype); 
  // // 但是setPrototypeOf性能不好 更推荐 create
  
  // 以上两行等价于
  let obj = Object.create(Con.prototype);
  // 3 将this指向空对象 执行构造函数
  let res = Con.apply(obj, args);  // Con.call(obj, ...args);
  // 对构造函数返回值做判断，然后返回对应的值（构造函数返回的对象 或 新建的对象）
  return res instanceof Object ? res : obj;
}
```
```javascript
// 版本2 学习细节即可
function createNew(Fn,...args) {
	// 创建空对象
	let newObj = null
	// 获取Fn构造函数
	// 这里由于argumens虽然是类数组类型，但并没有数组中的方法，因此使用call将数组的shift()方法绑定给arguments
	// 这里constructor为Fn，arguments为...args,即需传入构造函数中的参数。
	let constructor = Array.prototype.shift.call(arguments)
	let result = null
	// 判断构造函数的函数类型
	if(typeof constructor !== 'function')  {
		console.log('type error')
		return
	}
	// 原型链链接
	newObj = Object.create(constructor.prototype)
	// 绑定this，执行构造函数，接收构造函数返回值
	result = constructor.apply(newObj,args)
	// 若result为引用类型，返回；否则返回newObj;
	return result instanceof Object ? result : newObj;
}

```
补充：
instanceof 是根据原型__proto__判断的
对于数组类型校验，Array.isArray()是最权威的
![截屏2024-01-23 下午8.05.49.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1706011559601-6b725167-10c6-4ff8-9755-a77c81e5b34c.png#averageHue=%23fbfbfb&clientId=ub2cf3ac2-654c-4&from=drop&height=285&id=udd557b36&originHeight=612&originWidth=1010&originalType=binary&ratio=2&rotation=0&showTitle=false&size=74320&status=done&style=none&taskId=u68d6a748-09ad-4b87-b91a-607c835d48b&title=&width=470)

### 13 数组合并
1 localeCompare 排序
方法返回一个数字，表示参考字符串在排序顺序中是在给定字符串之前、之后还是与之相同
a.localeCompare(b) a在b之前为负数，之后为正数，相同为 0
```javascript
let arr1 = ['A1','A2','B1','B2','C1','C2'];
let arr2 = ['A', 'B', 'C'];
// 合并后的数组为：['A1','A2','A','B1','B2','B','C1','C2','C']

arr2 = arr2.map(item => item + '珠峰');
let arr = arr1.concat(arr2);
arr = arr.sort((a,b) => a.localeCompare(b)).map(item => {
  return item.replace('珠峰', '')
});
console.log(arr);

```
2 保持原数组的顺序 2重循环 
arr.splice(n, 0, 'S', 'b'); 删除的时候是从 n开始包括 n, 增加的时候是增加到n的前面
```javascript
let arr1 = ['C1','C2','A1','A2','B1','B2'];
let arr2 = [ 'B', 'C', 'A'];
// 合并后的数组为：['C1','C2','C','A1','A2','A','B1','B2','B']
let n = 0;
for(let i = 0; i < arr2.length; i++) {
  let value = arr2[i];
  for(let j = 0; j < arr1.length; j++) {
    if(arr1[j].includes(value)) {
      // n记录最后一个包含的项的 index
      n = j;
    }
  }
  // 把 value 插入到n的后面 也就是 n+1 的前面
  arr1.splice(n + 1, 0, value);
}
console.log(arr1)
```
### 14  for循环中用 setTimeout 打印 i
定时器是异步编程，每一轮循环设置定时器，无需等待，继续下一轮执行。（定时器触发的时候，循环已经结束了）
```javascript
for(var i = 0; i < 10; i++) {
 setTimeout(() => {
   console.log(i)
 },1000)
}
// 10次 10
// 执行回调函数的时候 i在当前函数中作用域没有声明 则去上一级作用域找 找到的i为10 此时的i在window上

// 1 let
for(let i = 0; i < 10; i++) {
  // let存在块级作用域，每一次循环都会在当前块作用域中形成一个私有变量 i 存储 0-9
  // 当定时器执行的时候，所使用的i就是所处块作用域中的i
   setTimeout(() => {
     console.log(i)
   },1000)
}
// 0 1 2 3 4 5 6 7 8 9

// 2 使用闭包形成块级作用域
for(var i = 0; i < 10; i++) {
  ~ function(i) {
    setTimeout(() => {
       console.log(i)
     },1000)
  }(i);

  // 闭包方式 写法2 包在函数上
  // setTimeout((i => () => console.log(i))(i), 1000)
}

// 3 基于 bind 的预处理机制：在循环的时候就把每次执行函数需要输出的结果，预先传给函数即可
var fn = function (i) {
  console.log(i)
}
for(var i = 0; i < 10; i++) {
  setTimeout(fn.bind(null, i), 1000)
}

```
#### 补充： 声明提升
**函数声明**和**变量声明（不是初始化）**总是会被解释器悄悄地被**"提升"到方法体的最顶部**。
（let声明没有提升， let 声明的变量只能在执行到声明所在的位置之后才能被访问（暂时性死区））
```javascript
x = 5; // 变量 x 设置为 5

elem = document.getElementById("demo"); // 查找元素 
elem.innerHTML = x;                     
// 在元素中显示 x

var x; // 声明 x
```
JavaScript 中，函数及变量的声明都将被提升到函数的最顶部。
JavaScript 中，变量可以在使用后声明，也就是变量可以先使用再声明。
```javascript
var x = 5; // 初始化 x

elem = document.getElementById("demo"); // 查找元素 
elem.innerHTML = x + " " + y;           // 显示 x 和 y

var y = 7; // 初始化 y


// ------------以上等价于------------------------------------------------
var x = 5; // 初始化 x
var y;     // 声明 y

elem = document.getElementById("demo"); // 查找元素
elem.innerHTML = x + " " + y;           // 显示 x 和 y   --y 输出了 undefined 此时未赋值

y = 7;    // 设置 y 为 7

```
注意： JavaScript 只有声明的变量会提升，初始化的不会提升。（var y = 7；var y；声明会提升， y=7不会提升）
#### 补充：js作用域
**作用域**是可访问**变量的集合**。
**在 JavaScript 中, 作用域为可访问变量，对象，函数的集合。**
js中[作用域](https://so.csdn.net/so/search?q=%E4%BD%9C%E7%94%A8%E5%9F%9F&spm=1001.2101.3001.7020)只有函数（局部）作用域和全局作用域，ES6 新增块级作用域
JavaScript 函数作用域: 作用域在函数内修改。
**局部作用域**：变量在函数内声明，变量为局部变量，具有局部作用域。
var声明的变量有可能是全局变量有可能是局部变量，取决于声明的位置。
**局部变量**：只能在函数内部访问。局部变量在函数开始执行时创建，函数执行完后局部变量会自动销毁。
```javascript
// 此处不能调用 carName 变量
function myFunction() {
    var carName = "Volvo";
    // 函数内可调用 carName 变量
}
```
**全局变量**：变量在函数外定义，即为全局变量。全局变量有 **全局作用域**: 网页中所有脚本和函数均可使用。 
```javascript
var carName = " Volvo";
 
// 此处可调用 carName 变量
function myFunction() {
    // 函数内可调用 carName 变量
}
```
**注意**：如果变量在函数内没有声明（没有使用 var 关键字），该变量为全局变量。
以下实例中 carName 在函数内，但是为全局变量。
在 HTML 中, 全局变量是 window 对象，所以window 对象可以调用函数内的局部变量。
```javascript
// 此处可调用 carName 变量
myFunction(); // 函数得先执行
console.log(carName)

function myFunction() {
    carName = "Volvo";
    // 此处可调用 carName 变量
}
```
JavaScript **变量生命周期**在它声明时初始化。
局部变量在函数执行完毕后销毁。
全局变量在页面关闭后销毁。
函数参数只在函数内起作用，是局部变量。
回顾：for循环并不是一个函数体，for循环中定义的变量q和i的作用域是for循环所在的函数体，和a同级
```javascript
var a=[];  
for(var i = 0;i<10;i++){  
   var q = i;  
   a[i]=function(){console.log(q)}  
}  
a[0]()  
      
// 其中，由于for循环并不是一个函数体，所以for循环中定义的变量q和i是作用域for循环所在的函数体，和a同级，  
// i++ 和  q=i 并不是重新定义变量，只是重复赋值，最终循环结束，i = 10,q=9;    
// 由于function(){console.log(q)} 并不是立即执行，所以这里的q一直是存储的内存引用，最终所有的a[i]()都是输出 9  

// var a = 1;
// var a = 2;
// 在 js不报错，等价于
// var a;
// a = 1
// a = 2
// let不行 重名报错 let声明没有提升 
// let 声明的变量只能在执行到声明所在的位置之后才能被访问（暂时性死区）

// 不过，在es6中新增了let命令声明变量，let所声明的变量，只在let命令所在的代码块有效果，for循环的计数器中就很适合let命令  
var a=[];  
for(var i = 0;i<10;i++){  
  let q = i;  
  a[i]=function(){console.log(q)}  
}  
a[6]()    //这里会输出   6  
// let声明的变量仅在块级作用域有效，所以这里的q只在本轮循环有效果

var a=[];  
for(let i = 0;i<10;i++){  
  a[i]=function(){console.log(i)}  
}  
a[6]()  //这里会输出   6
// 每次循环的i其实都是一个新的变量  

```
### 15  匿名函数起名字 填空
（比较另类的存在，主要是了解知识点）
1 本应匿名的函数，设置了函数名，在外面是不能用的。在函数里面是可读的。
2 类似于创建常量(const)一样，这个名字存储的值不能再被修改。修改时严格模式下会直接报错。非严格模式下不报错但是不会有任何效果。
```javascript
let fn = function AAA(){
  console.log(AAA); // 输出当前函数。此时的 AAA 就像定义常量一样，不可修改
  AAA = 1000; // 严格模式 报错 assignment to const variable，
  // 非严格模式下不报错但是修改不成功
};
AAA(); // AAA not define
fn(); // 对的
```
 题目：（访问变量的时候：首先看是不是自己的私有变量：有没有声明、是不是形参、是不是自己的函数名字）
```javascript
var b = 10;
(function b() {
  b = 20;
  console.log(b);
  // 首先看是不是自己的私有变量：有没有声明、是不是形参、是不是自己的函数名字
  // b是自己的函数名，但是不可修改，非严格模式 打印的b是函数

  //要想让b输出20, 把b变成私有的不能是全局的， 1 增加声明 var b = 20; 2 改为形参 (function b(b) 
})();
console.log(b) // 10

// 去掉匿名函数的名字b,则都变成全局的了，20 20
```
### 16 == 比较运算符   toString()
#### 运算规则
== 进行比较的时候会先转换成相同的数据类型
```javascript
1. {} == {} 不相等 两个对象比较，比较的是堆内存的地址
2. null == undefined 相等   null === undefined  不等
3. NaN == NaN 不相等  NaN和谁都不相等
4. [12] == '12';  '12' == [12] 相等 对象和字符串比较，就是对象.toString()转为字符串之后再进行比较
5. 剩余所有情况，都转换为数字再比较（前提数据类型不一致）
  	对象转数字：先转换为字符串，在转换为数字
  	字符串转数字：只要出现一个非数字即为 NaN
  	布尔转数字： true 1 false 0
  	null转数字： 0
  	undefined转数字： NaN

    [12] == true  false  Number([12].toString()) == 12
  	[] == false		true		0(即Number('')) == 0
    [] == 1				false
  	"1" == 1			true
  	true == 2 		fasle

```
#### 题目
```javascript
var a; // 求 a
if(a == 1 && a == 2 && a == 3) {
  console.log(1)
}

// 肯定不是 Number、String、Boolean、null、undefined
// 对象和数字比较：obj.toString(）成字符串，再转换为数字

// 方法一
var a = {
  n: 0,
  toString: function() {
    return ++this.n;
  }
}
if(a == 1 && a == 2 && a == 3) {
  console.log(1)
}
// a.toString()此时调取的是自己的私有方法toString，不再是Object.prototype.toString了

// 方法二 
// shift 删除第一项 返回第一项 改变原数组
let a = [1, 2, 3];
a.toString = a.shift;
if(a == 1 && a == 2 && a == 3) {
  console.log(1)
}

// 方法三 ES6 新增
// String.fromCharCode(122) <=> "z".charCodeAt()
// Array.from()
// Array.isArray()
// Object.create(obj)
// Object.defineProperty() 详见MDN 在对象上定义一个新属性，或者修改对象的属性，返回对象
    let obj = {
      name: 'jaja'
    };
    Object.defineProperty(obj, 'name', { // 最大的优势在于能够监听它的get和set，这就是vue双向绑定的原理Object.defineProperty
      get: function() {
        return 'haha';
      },
      set: function() {
        this.value = 'hehehe'
      }
    })
    obj.name; //'haha'
    obj.name = 3;  //obj.value = 'hehehe'

// 所以答案是
Object.defineProperty(window, 'a', {
  get: function () {
    this.value ? this.value += 1 : this.value = 1;  // this就是a，value表示a的值
    return this.value;
  }
})
if(a == 1 && a == 2 && a === 3) { // a没写var默认为它是 window 的变量
  console.log(1)
}
```
### 17 Array.prototype.push实现 填空
```javascript
let obj = {
  2: 3,
  3: 4,
  length: 2,
  push: Array.prototype.push
}
obj.push(1)
obj.push(2)
console.log(obj)

// 没有思路时候，就手写 push 实现一下 看看原理
// Array.prototype.push = function (val) {
//   this[this.length] = val;
//   // 浏览器自动给this.length +1
//   return this.length；
// }
// obj.push(1)
// obj[obj.length]=1  obj[2]=1  obj.length=3
// obj.push(2)
// obj[obj.length]=1  obj[3]=2  obj.length=4
// => {
//   2: 1,
//   3: 2,
//   length: 4,
//   push: Array.prototype.push
// }
```
### 18 冒泡排序 Bubble  大的向上走
```javascript
// bubble: 实现冒泡排序
// @params 
//   arr 要排序的数组
// @return 
// 	[array] 排序后的数组
function bubble(arr) {
  let temp = null;
  // 外层循环控制比较的轮数
  for(let j = 0; j < arr.length - 1; j++) {
    // 内层循环控制每一轮比较的次数
    for(let i = 0; i < arr.length - 1 - j; i++) {
       if(arr[i] > arr[i+1]) {
          temp = arr[i];
          arr[i] = arr[i+1];
          arr[i+1] = temp;
        }
    }
  }
  return arr;
}

let arr = [12, 8, 24, 16, 1];
arr = bubble(arr);
console.log(arr);
```
### 19  插入排序 Insert  起牌
```javascript
function insert(arr) {
  // 先抓一张牌进来
  let resultArr = [];
  resultArr.push(arr[0]);
  // 从第二项开始抓牌
  for(let i = 1; i < arr.length; i++) {
     // 拿到一张牌 arr[i], 和手里的牌从后往前比, 大的方后面
     for(let j = resultArr.length-1; j >= 0; j--) {
       if(arr[i] >= resultArr[j]) {
         resultArr.splice(j+1, 0, arr[i]);
         break;
       }
       if(j === 0) {
         resultArr.unshift(arr[i])
       }
     }
  }
  return resultArr;
}

let arr = [12, 8, 24, 16, 1];
arr = insert(arr);
console.log(arr);
```
### 20  快速排序  Quick Sort 递归
找到中间项并从原数组中移除，让拿出来的每一项和中间项比较，小的放左大的放右，
对左边和右边分别重复此操作。
```javascript
// 递归说明
function fn() {
  fn();
}
fn()
// 会导致死循环 栈溢出

function fn() {
  console.log(1)
  setTimeout(fn, 0);
}
fn()
// 不会导致栈溢出 也会一直执行

// 从 1 加到 10
// let total = null;
// for(let i = 1; i<=10; i++) {
//   total += i;
// }

// 递归实现
function sum(n) {
  if(n>10){return 0}
  return n + sum(n+1);
  // return 1+sum(2)
  // return 1+2+sum(3)
  //...
  // return 1+2+3+4+5+6+7+8+9+10+sum(11)即 0
}
console.log(sum(1))

```
```javascript
function quick(arr) {
  // 4 递归结束条件 数组中小于等于一项
  if(arr.length <= 1) {
    return arr;
  }
  
  // 1 找到中间项并从原数组中移除
  let middleIndex = Math.floor(arr.length/2);
  let middleValue = arr.splice(middleIndex, 1)[0]; //返回的是删掉的数据形成的数组
  
  // 2 准备左右两个数组，循环数组中剩下的每一项 小的放左大的放右
  let leftArr = [], 
    rightArr = [];
  for(let i = 0; i <arr.length; i++) {
    let item = arr[i];
    item < middleValue ? leftArr.push(item) : rightArr.push(item)
  }
  
  // 3 递归左右数组 最后拼接左—中-右
  return quick(leftArr).concat(middleValue, quick(rightArr));
}

let arr = [12, 8, 24, 16, 1];
arr = quick(arr);
console.log(arr);
```
### 21 数组数据处理 填空
```javascript
let obj = {
  1: 222,
  2: 213,
  5: 53
};
// 请把数据处理为如下结构：[222, 213, null, null, 53, null, null, null, null, null, null, null]

// Array.from(obj) // [] 因为没有 length 属性 不能转数组

// 方法 1
let arr = new Array(12).fill(null).map((item, index) => {
  return obj[index+1] || null;
})

// 方法 2 obj自行设置length
// obj.length = 12;
// let arr = Array.from(obj);  
// [undefined, 222, 213, undefined, undefined, 53, undefined, undefined, undefined, undefined, undefined, undefined]
obj.length = 13;
let arr = Array.from(obj).slice(1).map(item => 
  typeof item === 'undefined' ? null : item;
);  
console.log(arr);

// 方法 3 Object.keys
let arr = new Array(12).fill(null);
Object.keys(obj).forEach((item) => {
  arr[item-1] = obj[item];
});
console.log(arr);

```
#### Arrray.prototype常见方法
push/pop
shift/unshift
splice/slice
concat
join
toString
reverse
sort
indexOf/lastIndexOf
includes
forEach/map/some/find
flat
fill
### 22  求两个数组的交集
思考题：交差并补
```javascript
let nums1 = [1, 2, 2, 1];
let nums2 = [2, 2];
// 结果 [2, 2]

// 方法 1
let arr = [];
for (let i = 0; i < nums1.length; i++) {
  let item1 = nums1[i];
	for (let j = 0; j < nums2.length; j++) {
  	let item2 = nums2[j];
    if(item1 === item2) {
      arr.push(item1);
      break;
    }
  }
}
console.log(arr)

// 方法 2 使用数组自带方法替换
let nums1 = [1, 2, 2, 1];
let nums2 = [2, 2];
let arr = [];
//  nums1.forEach(item => nums2.includes(item) ? arr.push(item) : null);  // 不行
nums1.forEach((item, index) => {
  let n = nums2.indexOf(item);
  if(n >= 0) {
    arr.push(item);
    nums2.splice(n, 1);
  }
});

```
### 23  旋转数组 填空
```javascript
let nums1 = [1, 2, 2, 1], k=2;
nums1.rotate(k) 
// [2, 1, 1, 2]

function rotate(k) {
  if(k<0 || k === this.length || k===0) { return this};
  if(k>this.length) { k = k % this.length};
  //旋转数组 后面的 K 个拿出来放在前面
  // 思路一
  // return this.slice(-k).concat(this.slice(0, this.length - k))
  // return [...this.splice(this.length - k), ...this];

  // 思路二
  // for(let i = 0; i < k; i++) {
  //   this.unshift(this.pop());
  // }
  // return this;
  // 等价于
  new Array(k).fill('').forEach(() => this.unshift(this.pop()));
  return this;
}
Array.prototype.rotate = rotate;

```
### 24  函数柯理化 有难度
```javascript
// 请实现一个 add方法 层级不固定
add(1) // 1
add(1)(2) // 3
add(1)(2)(3) // 6
add(1)(2,3) // 6
add(2,3)(1) // 6
add(1,2,3)  // 6
```
#### 手写bind
执行bind方法，会返回一个匿名函数，当事件触发，匿名函数执行
```javascript
let obj = {name: 1}
function fn(...arg) {
  console.log(this, arg)
}
document.body.onclick = fn.myBind(obj, 100, 200);

// 点击的时候执行函数，此时FN中的this=>obj, arg=> [100, 200, 事件对象]
// document.body.onclick = fn.bind(obj, 100, 200);
// 等价于=> document.body.onclick = function(e) {
//	fn.call(obj, 100, 200, e)
// }

(function () {
  // this 指的是函数fn
  // context 指的是 obj 要改变的 this的指向
  // outerArg其余需要传递给函数 fn的参数
  function myBind(context = window, ...outerArg) {
     let _this = this; // 函数 fn
     return function anonymous(...innerArg) { // 这个函数直接使用this的话，this指的是 下文document.body.onclick中的 this 即Body
       _this.call(context, ...outerArg.concat(innerArg)); // 立即执行
       // _this.apply(context, outerArg.concat(innerArg)); 
     }
  }
  Function.prototype.myBind = myBind;
})()

```
**函数执行形成了一个闭包（bind），把一些信息预先存储起来，当以后需要用到的时候（bind返回的函数执行的时候），直接从闭包（函数）里面拿来用。**
闭包的二大作用：保存，保护。
bind就是一个柯理化函数。
#### 函数柯理化
**预先处理的思想（利用闭包(保存)的机制）**
```javascript
function fn(x) {
  // 预先在闭包中把x存储起来了
  return function(y) {
    return x + y;
  }
}
fn(100)(200)
// 第一次fn执行，返回一个函数，作用域没有销毁，形成一个闭包，把传进来的 100 保存了
// 第二次执行 遇到x 当前函数中没有 再直接去上一个闭包里面去找就行了
```
俩字描述柯理化就是闭包。
利用闭包存储数据，供以后闭包里面的子集作用域使用。
```javascript
function currying(fn, length) {
  // length 表示最终有几个数相加 不关心调用几次
  length = length || fn.length;
  return function(...args) {
    if(args.length >= length) {
      return fn(...args);
    }
    // 闭包(返回函数)里面套闭包（bind）
    return currying(fn.bind(null, ...args), length - args.length);
  }
}

function $add(n1, n2, n3, n4) {
  return n1 + n2 + n3 + n4;
}

let add = currying($add, 4);
console.log(add(1,2,3,4));
console.log(add(1)(2)(3)(4));
console.log(add(1,2)(3,4));
```
add(1,2)(3,4)执行过程图解：
![百度网盘.jpeg](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1706264890010-10520eea-ee59-42b1-a8a3-b7dc2572b214.jpeg#averageHue=%23e7e1d9&clientId=u41a690a9-c599-4&from=drop&id=u80baa858&originHeight=822&originWidth=2160&originalType=binary&ratio=2&rotation=0&showTitle=false&size=707893&status=done&style=none&taskId=u67b03581-0905-4935-ad21-eec1fed1a55&title=)
```javascript
// ts eval当做语句执行
let add1 = currying((...arg) => eval(arg.join('+')), 5)
console.log(add1(1,2)(3,4,5));

// 简单版 固定层数
function add(...A) {
  return function (...B) {
    return function (...C) {
      return eval([...A, ...B, ...C].join('+'));
    }
  }
}
add(1)(2)(3);
```











