#### new
<hr>

```!
new 运算符创建一个用户定义的对象类型的实例或者具有构造函数的内置对象类型之一
```
也许有点难懂，我们在模拟new 之前，先看看new实现了哪些功能。

举个例子:

```js
function Homer (name,age) {
    this.name = name;
    this.age = age;
    this.habit = 'Games';
}
// 因为缺乏锻炼的缘故，身体强度让人担忧
Homer.prototype.strength = 60;
Homer.prototype.sayYourName = function () {
    console.log('I am ' + this.name);
}
var person = new Homer('Kevin', '18');
console.log(person.name); // Kevin
console.log(person.habit); // Games
console.log(person.strength); // 60
person.sayYourName(); // I am Kevin;
```
从这个例子中，我们可以看到，实例 person 可以：

1. 访问到 Homer 构造函数里的属性
2. 访问到 Homer.prototype 中的属性

接下来，我们可以尝试着模拟一下了

因为 `new` 是关键字,所以无法像bind函数一样直接覆盖，所以我们写一个函数，命名为 objectFactory,来模拟 new 的效果。用的时候是这样的：

```js
function Homer () {
    ……
}

// 使用 new
var person = new Homer(……);
// 使用 objectFactory
var person = objectFactory(Homer, ……)
```
#### 初步实现
<hr>
分析：
因为new 的结果是一个新对象，所以在模拟实现的时候，我们也要建立一个新对象，假设这个对象叫obj,因为obj会具有Homer构造函数里的属性，想想经典继承的例子，我们可以使用Homer.apply(obj,arguments)来给obj添加新的属性。

在 JavaScript 深入系列第一篇中，我们便讲了原型与原型链，我们知道实例的 __proto__ 属性会指向构造函数的 prototype，也正是因为建立起这样的关系，实例可以访问原型上的属性。

现在，我们可以尝试着写第一版了：

```js
function objectFactory () {
    var obj =  new Object(),
    Contructor = [].shift.call(arguments);
    obj.__proto__ = Contructor.prototype;
    Constructor.apply(obj,arguments);
}
```
在这一版中，我们：

1. 用 new Object() 的方式新建了一个对象Obj
2. 取出第一个参数，就是我们要传入的构造函数。此外因为shift会修改原数组，所以 arguments 会被去除第一个参数。
3. 将 obj 的原型指向构造函数，这样obj就可以反问到构造函数原型中的属性
4. 使用apply, 改变构造函数 this的指向到新建的对象，这样obj就可以访问到构造函数中的属性。
5. 返回obj

复制以下的代码，到浏览器中，我们可以做一下测试

```js
function Otaku (name, age) {
    this.name = name;
    this.age = age;

    this.habit = 'Games';
}

Otaku.prototype.strength = 60;

Otaku.prototype.sayYourName = function () {
    console.log('I am ' + this.name);
}

function objectFactory() {
    var obj = new Object(),
    Constructor = [].shift.call(arguments);
    obj.__proto__ = Constructor.prototype;
    Constructor.apply(obj, arguments);
    return obj;
};

var person = objectFactory(Otaku, 'Kevin', '18')

console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // 60

person.sayYourName(); // I am Kevin
```
#### 返回值效果实现
<hr>

```js
function Otaku (name, age) {
    this.strength = 60;
    this.age = age;

    return {
        name: name,
        habit: 'Games'
    }
}

var person = new Otaku('Kevin', '18');

console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // undefined
console.log(person.age) // undefined
```
在这个例子中，构造函数返回了一个对象，在实例person中只能访问返回对象中的属性。

而且还要注意一点，在这里我们是返回了一个对象，假如我们只是返回一个基本类型的值呢？
再举个例子：

```js
function Otaku (name, age) {
    this.strength = 60;
    this.age = age;

    return 'handsome boy';
}

var person = new Otaku('Kevin', '18');

console.log(person.name) // undefined
console.log(person.habit) // undefined
console.log(person.strength) // 60
console.log(person.age) // 18
```
结果完全颠倒过来，这次尽管有返回值，但是相当于没有返回值进行处理。

所以我们还需要判断返回的值是不是一个对象，如果是一个对象，我们就返回这个对象，如果没有，我们该返回什么就返回什么。

再来看第二版的代码，也是最后一版的代码：

```js
// 第二版的代码
function objectFactory() {
    var obj = new Object(),//从Object.prototype上克隆一个对象

    Constructor = [].shift.call(arguments);//取得外部传入的构造器

    var F=function(){};
    F.prototype= Constructor.prototype;
    obj=new F();//指向正确的原型

    var ret = Constructor.apply(obj, arguments);//借用外部传入的构造器给obj设置属性

    return typeof ret === 'object' ? ret : obj;//确保构造器总是返回一个对象

};

```
#### new 的实现
```js
// yck code
新生成了一个对象
链接到原型
绑定 this
返回新对象
在调用 new 的过程中会发生以上四件事情，我们也可以试着来自己实现一个 new

function create() {
    // 创建一个空的对象
    let obj = new Object()
    // 获得构造函数
    let Con = [].shift.call(arguments)
    // 链接到原型
    obj.__proto__ = Con.prototype
    // 绑定 this，执行构造函数
    let result = Con.apply(obj, arguments)
    // 确保 new 出来的是个对象
    return typeof result === 'object' ? result : obj
}
```
#### instanceof 的实现

```js
// a instanceof Object
function myInstanceof(left,right){
    var proto = left.__proto__;
    var protoType = right.prototype;
    while(true){
        if(proto === null){
            return false
        }
        if(proto == protoType){
            return true
        }
        proto = proto.__proto__
    }
}
```