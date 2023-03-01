---
layout: post
title: "vue源码分析心得"
date: 2021-12-08 22:39
comments: true
tags: 
	 - vue
   - javaScript
---
## 前言
> 前面一直想要探索一下vue源码的奥秘，刚好最近工作任务不是很重没有加班，就利用工作之余的时间研究一下vue源码，下面是自己在此过程中对部分知识点的梳理，不是很全面也还在学习中，会不断完善更新。
### 观察者模式

类似于node的的Event模块，使用一个容器对象来存储注册的事件，提供on方法注册事件，off方法移除事件，emit方法发射事件。如下：
<!--more-->
```js
class Event {
    constructor() {
        this.eventInfo = {}
    }
  	/** 注册事件, 可以连续注册, 可以注册多个事件 */
    on(type, handle){
        (this.eventInfo[type] || (this.eventInfo[type] = [])).push(handle);
    }
  	/** 移除事件, 
    	* - 如果没有参数, 移除所有事件, 
      * - 如果只带有 事件名 参数, 就移除这个事件名下的所有事件,
      * - 如果带有 两个 参数, 那么就是表示移除某一个事件的具体处理函数
    * */
    off(type, handle){
        if(!arguments.length) {
            this.eventInfo = {}
        }else if (arguments.length === 1) {
            this.eventInfo[type] = []
        }else if (arguments.length === 2) {
            const len = this.eventInfo[type].length;
          	// 倒着循环 数组的 序号不会受到影响
            for (let i = len - 1; i >= 0; i--) {
                if (this.eventInfo[type][i] === handle) {// 函数比较为引用类型比较的是引用地址
                    this.eventInfo[type].splice(i, 1);
                }
            }
        }
    }
   /** 
   		* 发射事件, 触发事件, 包装参数 传递给事件处理函数
   */
    emit(type) {
        const len = this.eventInfo[type].length;
        let args = Array.prototype.slice.call( arguments, 1 )
        for (let i = 0; i < len; i++) {
           this.eventInfo[type][i].apply( null, args );
        }
    }
}

const event = new Event()
```

### 插值语法解析实现

```js
function getValueByPath(obj, path) {
  const paths = path.split('.'); // [xxx, yyy, zzz];
  let res = obj;
  let prop;
  while (prop = paths.shift()) {
    res = res[prop];
  }
  return res;
}

//解析：{{ data.userInfo.userName }}
const data = { userInfo: {userName: 'zhangsan' }} // 类似于vue里定义到data里的数据
const reg = /\{\{(.+?)\}\}/g
const str = '{{ userInfo.userName }}'; // 类似于vue模板里的用法
const value = str.replace(reg, (_, g) => {
  // _: {{ data.userInfo.userName }}, g:  data.userInfo.userName
  return getValueByPath(data, g.trim());
})
```

### 属性代理

在vue里`const app = new Vue(options)`通过optinos传递的data属性，在vue底层initData的时候将options上的很多属性都绑定到了vue实例上，因此我们可以通过app.xxx访问到options.data.xxx上的对应的属性值，修改app.xxx =xxx 同样会同步修改到data.xxx。

实现原理大致如下：

```js
const options = {
  data: {
    name: 'zhangsan'
  }
}
/** 将某一个对象的属性 访问 映射到 对象的某一个属性成员上 */
const proxy = (target, prop, key) => {
  Object.defineProperty(target, key, {
    enumerable: true,
    configurable: true,
    get () {
      return target[prop][key];
    },
    set(newVal) {
      target[prop][key] = newVal;
    }
  })
}

class Vue {
  constructor(options) {
   	this._data = options.data;
    this.initData();
  }
  initData(){
    const keys = Object.keys(this._data);
    for (let i = 0; i<keys.length; i++) {
      proxy(this, '_data', keys[i])
    }
  }
}

const app = new Vue(options)
```

### 数据响应式处理

```js

 // 针对数组类型方法响应式处理
let ARRAY_METHOD = [
  'push',
  'pop',
  'shift',
  'unshift',
  'reverse',
  'sort',
  'splice',
];
let array_methods = Object.create( Array.prototype );
ARRAY_METHOD.forEach( method => {
  array_methods[ method ] = function () {
    // 调用原来的方法
    console.log( '调用的是拦截的 ' + method + ' 方法' );

    // 将数据进行响应式化
    for( let i = 0; i < arguments.length; i++ ) {
      observe( arguments[ i ] );
    } 

    let res = Array.prototype[ method ].apply( this, arguments );
    return res;
  }
} );


const defineReactive = ( target, key, value, enumerable ) => {
 // 函数内部就是一个局部作用域, 这个 value 就只在函数内使用的变量 ( 闭包 )
  if ( typeof value === 'object' && value != null ) {
    // 是非数组的引用类型
    observe(value); // 递归
  }

  Object.defineProperty(target, key, {
    configurable: true,
    enumerable: !!enumerable,
    get() {
      return value;
    },
    set(newVal) {
      if ( value === newVal ) return;
      // 目的
      // 将重新赋值的数据变成响应式的, 因此如果传入的是对象类型, 那么就需要使用 observe 将其转换为响应式
      if (typeof newVal === 'object' && newVal != null) {
        observe(newVal);
      } 
      value = newVal;
}

// 对象或者数组类型需要递归处理对应的属性成响应式
const observe = ( obj ) => {
  // 之前没有对 obj 本身进行操作, 这一次就直接对 obj 进行判断
  if ( Array.isArray( obj ) ) {
    // 对其每一个元素处理
    obj.__proto__ = array_methods;
    for ( let i = 0; i < obj.length; i++ ) {
      observe( obj[ i ] ); // 递归处理每一个数组元素 
    }
  } else {
    // 对其成员进行处理
    let keys = Object.keys( obj );
    for ( let i = 0; i < keys.length; i++ ) {
      let prop = keys[ i ]; // 属性名
      defineReactive( obj, prop, obj[ prop ], true );
    }
  }
}
```

**注意：**针对数组类型的响应式操作任然存在缺陷，如直接对array.length进行操作

然后在上面`属性代理`那里定义的initData方法里调用`observe(this._data)`实现对_data数据的响应式处理

```js
initData(){
  	observe(this._data);// 响应式化
  
    const keys = Object.keys(this._data);
   //属性代理
    for (let i = 0; i<keys.length; i++) {
      // 将 this._data[ keys[ i ] ] 映射到 this[ keys[ i ] ] 上
      // 就是要 让 this 提供 keys[ i ] 这个属性
      // 在访问这个属性的时候 相当于在 访文this._data 的这个属性
      proxy(this, '_data', keys[i])// 这里封装成proxy是在vue中不单单只处理了_data，像computed,watch等等也是做了同样的的处理
    }
  }
```

### 发布订阅模式

形式： 不局限于函数，形式可以是对象等

- 中间的**全局的容器 **，用来**存储**可以被触发的东西（函数，对象）；

- 需要一个方法，可以往容器中**传入**东西（函数，对象）

- 需要一个方法，可以将容器中的东西取出来**使用**（函数调用，对象的方法调用）

在vue页面中变更（diff）是以组件为单位，如果页面中只有一个组件（Vue实例），不会有性能损失，但如果有多个组件（多个watcher的一种情况），第一次会有多个组件watcher存入到全局watcher中。假如修改了局部数据（某个组件的数据），表示只会对该组件进行diff算法，也就是说只会重新生成该组件的抽象语法树，只会访问该组件的watcher表示再次往全局存储的只有改组件的watcher，页面更新的时候也就只需要更新一部分

> **虚拟dom：**就对应就是一个页面组成的最小单位 **组件**
>
> **data:** 对应与我们在组件里定义的数据
>
> **watcher：** 与组件存在一一对应的关系，它主要是监听到组件数据修改就会派发更新，内存中会全局存储watcher（可以有多个watcher，并且至少会有一个watcher）

#### 什么时候会将组件对应的wather存入全局watcher

> 在模板渲染与虚拟Dom生成的时候会读取数据，此时会调用depend方法，将对应的watcher存入全局watcher中。

#### 什么时候调用watcher做派发更新

> 当修改数据的时候，也就是设置组件里定义的data属性的时候，会调用notify方法，将全局的所有watcher一一触发

下面让我们来看看代码层面的实现：

watcher观察者，用于发射更新行为，会提供以下方法：

- `get()`用来进行**计算**或者**执行**处理函数

- `update()`公共的外部方法，该方法会触发内部的run方法

- `run()`运行，用来判断内部是使用异步还是同步运行等，这个方法最终会调用内部的get方法

- `cleanupDep()` 简单理解为清除队列

```js
let watcherid = 0;
class Watcher {
  /**
   * 
   * @param {Object} vm JGVue 实例
   * @param {String|Function} expOrfn 如果是渲染 watcher, 传入的就是渲染函数, 如果是 计算 watcher 传入的就是路径表达式, 暂			时只考虑 expOrFn 为函数的情况.
   */
  consructor(vm, expOrFn){
    this.vm = vm;
    this.getter = expOrFn;
    this.id = watcherid ++;
    this.deps = []; // 依赖项
    this.get();
  }
  // 计算触发getter
  get(){
    pushTarget(this);
    this.getter.cal(this.vm, this.vm);
    popTarget();
  }
  /**
   * 执行, 并判断是懒加载, 还是同步执行, 还是异步执行: 
   * 我们现在只考虑 异步执行 ( 简化的是 同步执行 )
   */
  run(){
    this.get();
  }
  // 对外公开的方法，用于在属性发生变化的时候触发的接口
  update(){
    this.run()
  }
   /** 清空依赖队列 */
  cleanupDep() {

  }
  addDep(dep){
    this.deps.push(dep)
  }
}
```

Dep对象，该对象提供依赖收集（depend）的功能，和派发更新（notify）的功能，在notify中去调用watcher的update方法

#### watcher与Dep的关系

所谓依赖收集 **实际上就是告诉当前的watcher什么属性被访问了**，那么在这个watcher计算的时候或者渲染的时候就会将这些收集到的属性进行更新

#### 如何将属性与当前的watcher关联起来

> 在全局准备一个targetStack（watcher栈， 简单的理解为 watcher ”数组“，把一个操作中需要使用的watcher都存储起来），在watcher调用get方法的时候，将当前watcher放到全局，在ge t之前结束的时候（之后），将这个全局的watcher移除，提供：push Target、popTarget。在每个属性中都有一个Dep对象

在访问对象属性的时候（ge t），我们的渲染watcher就在全局中将属性与watcher关联，其实就是将当前渲染的watcher存储到属性相关的dep中。同事讲de p也存储到全局的watcher中（互相引用的关系）

- 属性引用了当前的渲染 watcher, **属性知道谁渲染它**
- 当前渲染 watcher 引用了 访问的属性 ( Dep ), **当前的 Watcher 知道渲染了什么属性**

```js
let depid = 0;
class Dep {
  constructor(){
    this.id = depid++;
    this.subs = []; // 存储的是与当前Dep关联的watcher
  }
  addSub(sub){// 添加一个watcher
    this.subs.push(sub);
  }
  removeSub(sub){// 移除指定watcher
    for(let i = this.subs.length - 1; i>= 0; i--){
      if(sub===this.subs[i]){
        this.subs.splice(i, 1);
      }
    }
  }
  // 将当前Dep与当前watcher关联
  depend(){
    // 就是将当前的dep与当前的watcher关联
    if(Dep.target){
      this.addSub(Dep.target);//将当前的watcher关联到的当前dep上
      Dep.target.addDep(this);// 将当前的dep与当前渲染watcher关联上
    }
  }
  //触发与之关联的watcher的update方法，起到更新的作用
  notify(){
    // 此时, deps 中已经关联到 我们需要使用的 那个 watcher 了
    let deps = this.subs.slice();
    deps.forEach(watcher => {
      watcher.update();
    })
  }
}
// 全局的容器存储渲染 Watcher
Dep.target = null;
let targetStack = [];

/** 将当前操作的 watcher 存储到 全局 watcher 中, 参数 target 就是当前 watcher */
const pushTarget(target){
  targetStack.unshift(Dep.target);
  Dep.target = target;
}
/** 将 当前 watcher 踢出 */
const popTarget() {
  Dep.target = targetStack.shift();// 踢到最后就是 undefined
}

```
### 数组去重
```js
  const arr = [1, 2, 3, 3, 5, 7, 2, 8, 1, 7, 5, 8];
  const obj = {};
  const setArr = [];
  arr.forEach(o => obj[o] || (obj[o] = true, setArr.push(o)));
  console.log(setArr); // [1, 2, 3, 5, 7, 8]
  console.log(obj); // {1: true, 2: true, 3: true, 5: true, 7: true, 8: true}
```
