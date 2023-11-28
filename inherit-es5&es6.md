es5实现继承有原型链继承、构造函数继承、组合继承、原型式继承、寄生继承和寄生组合继承方式
es6主要是通过class类的实现继承，而class本质上也是由寄生组合继承而来的

# 原型链继承

```js
function Parent() {
    this.name = 'parent1'
    this.play = [1, 2, 3]
}

function Child2() {
    this.type = 'child2'
}

Child2.prototype = new Parent()

let s1 = new Child2()
let s2 = new Child2()
s1.play.push(4)
console.log(s1.play, s2.play)
```

输出结果

```bash
[ 1, 2, 3, 4 ] [ 1, 2, 3, 4 ]
```

改变 s1 的 play 属性，会发现 s2 也跟着发生变化了，这是因为两个实例使用的是同一个原型对象，内存空间是共享的

# 构造函数继承

```js
// <!-- 构造函数继承 -->

function Parent() {
    this.name = 'parent'
    this.play = [1, 2, 3, 4]
    this.say = function () {
        console.log('hello world')
    }
}

Parent.prototype.getName = function () {
    return this.name
}

function Child() {
    Parent.call(this)
    this.type = 'child'
}

let child1 = new Child()
let child2 = new Child()
child1.play.push(5)

child1.say() // 输出hello world ，说明子类可以继承父类自己定义的方法

console.log(child1.play, child2.play) //输出 [ 1, 2, 3, 4, 5 ] [ 1, 2, 3, 4 ]，说明子类之间不共享引用属性

console.log(child1.getName()) //报错，子类不能继承父类原型属性或者方法
```

相比原型链继承方式，父类的引用属性不会被共享，优化了第一种继承方式的弊端，但是只能继承父类的实例属性和方法，不能继承父类原型上的属性或者方法

# 组合继承

```js
// <!-- 构造函数继承 -->
function Parent() {
    this.name = 'parent'
    this.play = [1, 2, 3, 4]
}

Parent.prototype.getName = function () {
    return this.name
}

function Child() {
    Parent.call(this)
    this.type = 'child'
}

Child.prototype = new Parent()
// 不加这一句的话Child.prototype.constructor属性是指向 Parent,要把constructor属性给指回来
Child.prototype.constructor = Child

let child1 = new Child()
let child2 = new Child()

child1.play.push(5)
console.log(child1.play, child2.play) // 不互相影响
console.log(child1.getName()) // 正常输出'parent'
console.log(child2.getName()) // 正常输出'parent'
```

寄生组合方式解决了原型链继承和构造函数继承的问题，但是它执行了两次`Parent`,造成多构造一次的性能开销

# 原型式继承

```js
let parent = {
    name: 'parent',
    frinds: [1, 2, 3, 4],
    getName: function () {
        return this.name
    },
}

let child1 = Object.create(parent)
child1.name = 'child1'
child1.frinds.push('5')

let child2 = Object.create(parent)
child2.name = 'child2'
child2.frinds.push('6')

console.log(child1.name) //child1
console.log(child1.name === child1.getName()) //true
console.log(child2.name) //child2

console.log(child1.frinds) //[ 1, 2, 3, 4, '5', '6' ]
console.log(child2.frinds) //[ 1, 2, 3, 4, '5', '6' ]
```

原型式继承主要是通过`Object.creat()`来实现的，因为 Object.create 方法实现的是浅拷贝，多个实例的引用类型属性指向相同的内存，存在篡改的可能

# 寄生式继承

```js
let parent = {
    name: 'parent5',
    friends: [1, 2, 3, 4],
    getName: function () {
        return this.name
    },
}

function clone(originalObj) {
    let clone = Object.create(originalObj)
    clone.getFriends = function () {
        return this.friends
    }
    return clone
}

let child1 = clone(parent)
console.log(child1.getName()) //parent5
console.log(child1.getFriends()) //[ 1, 2, 3, 4 ]

let child2 = clone(parent)
child2.friends.push(5)
console.log(child1.friends) //[ 1, 2, 3, 4, 5 ]
console.log(child2.friends) //[ 1, 2, 3, 4, 5 ]
```

从上面的例子可以看到，寄生式继承只不过是在原型式继承的基础上添加了一些方法，浅拷贝的问题依然存在

# 寄生组合式继承

```js
function clone(parent, child) {
    // 这里改用 Object.create 就可以减少组合继承中多进行一次构造的过程
    child.prototype = Object.create(parent.prototype)
    child.prototype.constructor = child
}

function Parent() {
    this.name = 'parent'
    this.play = [1, 2, 3, 4]
}
Parent.prototype.getName = function () {
    return this.name
}

function Child() {
    Parent.call(this)
    this.friends = 'child'
}

clone(Parent, Child)
Child.prototype.getFriends = function () {
    return this.friends
}

const child = new Child()
console.log(child)
```

# es6 类继承

```js
 class Parent {
    constructor(color, speed) {
        this.color = color
        this.speed = speed
    }
}

class Child extends Parent {
    construct(color, speed) {
        super(color, speed)
        this.container = true
    }
}
```

利用babel工具进行转换，我们会发现extends实际采用的也是寄生组合继承方式，因此也证明了这种方式是较优的解决继承的方式

# 总结

- 原型链继承是通过将子类的原型指向父类实现的，缺点是实例上继承而来的引用类型属性内存空间是共享的，对引用类型的数据进项更改会影响所有的实例
- 构造函数是通过在通过在子类中调用父类的构造函数并将其this指向子类, 其缺点是子类无法继承父类原型上定义的属性和方法
- 组合继承结合了原型链继承和构造函数继承，但缺点是父类被执行了两次
- 原型式继承是通过`Object.create()`来实现的，因为`Object.create()`是浅拷贝，所以当子类之间改变引用类型的数据时会互相影响。
- 寄生继承是在原型式继承的基础上加了方法的，缺点是相同的
- 寄生组合式继承集成了以上问题的优点，es6也是在寄生组合的基础上实现了类继承。



