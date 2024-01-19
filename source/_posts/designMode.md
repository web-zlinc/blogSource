---
title: 前端设计模式总结
layout: page
date: 2023-08-05 11:18
comments: true
tags: 
	- javascript
key: "1"
---

### 前言

>   设计模式在我最初的认知中，总以为它是后端Java开发领域里的专有名词。在这样的认知下，对于作为一个前端开发者的我而言接触的并不多，愚昧的认为前端就不存在设计模式这一说法。然而在时间的推移下，工作之余通过对掘金平台相关技术文章的阅读及对vue源码调试与阅读，加上最近刚刚读完《JavaScript设计模式与开发实践》一书后。终于对设计模式有了系统的了解，也由此打破了之前对设计模式之存在与后端Java开发语言的局限认知，下面主要针对最近读完的相关设计模式一书后，梳理下个人对设计模式全新的认识。

<!--more-->

### 什么是设计模式

设计模式并非是软件开发中的专业术语。其实，“模式”最早诞生于建筑学，当时主要是从**研究解决同一个问题而设计出不同的建筑结构，并从中发现了一些高质量设计中的相似性**，并用“模式”来指代这种相似性。

> 设计模式的定义是：**在面向对象软件设计过程中针对特定问题的简洁而优雅的解决方案。** 比较通俗的说就是在某种场合下对某个问题的一种解决方案。

设计模式大体可分为：创建型， 行为型，结构型。

- 创建型：是为了封装创建对象的变化，如：**单例、抽象工厂、建造者、工厂、原型**
- 行为型：是为了封装对象相关行为的变化，如：**模板方法、命令模式、迭代器模式、观察者、中介者模式、备忘录模式、访问者模式**
- 结构型：是为了封装对象之间的组合关系，通过对结构的设计达到性能、可扩展性、业务目的的代码写法，如：**适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式**

可以看出细分后的设计模式达到了23种，下面让我们针对前端常用的设计模式做一个详细的了解，再了解前端设计模式之前需要掌握如下基本概念知识点：

> - 面向对象的三大特性：继承、多态、封装
> - JavaScript里的关键字this，以及常用方法call、apply
> - 闭包、高阶函数
> - 单一职责原则：一个对象（方法）只做一件事情
> - 最少知识原则：尽可能的减少对象之间的联系
> - 开放封闭原则：当需要改变一个程序的功能或者给这个程序增加新功能的时候，可以使用增加代码的方式，但是不允许改动程序的源代码。

**注意：** 就设计模式而言，因为JavaScript这门语言的自身特点，许多设计模式在JavaScript之中的实现跟在一些传统面向对象语言中的实现相差很大。在JavaScript中，很多设计模式都是通过闭包和高阶函数实现的。

### 单例模式

> **定义：** 保证一个类仅有一个实例，并提供一个访问它的全局访问点

单例模式相对于其他设计模式实现起来是比较简单的，但是实现方式却不少，而在Java中实现单例模式常常是基于类实现的，由于JavaScript是一门无类语言，下面我们主要针对JavaScript看看实现单例模式的方式。

#### 透明单例模式

```javascript
        var CreateDiv = (function(){

            var instance;

            var CreateDiv = function( html ){
              if ( instance ){
                  return instance;
              }
              this.html = html;
              this.init();
              return instance = this;
          };

          CreateDiv.prototype.init = function(){
              var div = document.createElement( 'div' );
              div.innerHTML = this.html;
              document.body.appendChild( div );
          };

          return CreateDiv;

        })();

        var a = new CreateDiv( 'sven1' );
        var b = new CreateDiv( 'sven2' );

        alert ( a === b );     // true
```

在这段代码中，CreateDiv的构造函数实际上负责了两件事情。第一是创建对象和执行初始化init方法，第二是保证只有一个对象，违反了单一职责理念。

#### 代理单例模式

针对以上代码缺陷进行优化

```javascript
        var CreateDiv = function( html ){
            this.html = html;
            this.init();
        };

        CreateDiv.prototype.init = function(){
            var div = document.createElement( 'div' );
            div.innerHTML = this.html;
            document.body.appendChild( div );
        };
```

引入代理类：

~~~javascript
        var ProxySingletonCreateDiv = (function(){

            var instance;
            return function( html ){
              if ( !instance ){
                  instance = new CreateDiv( html );
              }

              return instance;
            }

        })();

        var a = new ProxySingletonCreateDiv( 'sven1' );
        var b = new ProxySingletonCreateDiv( 'sven2' );

        alert ( a === b );
~~~
#### 惰性单例
惰性单例指的是在需要的时候才创建对象实例
~~~javascript
        <html>
            <body>
              <button id="loginBtn">登录</button>
            </body>

        <script>
         var createLoginLayer = function(){
            var div = document.createElement( 'div' );
            div.innerHTML = ’我是登录浮窗’;
            div.style.display = 'none';
            document.body.appendChild( div );
            return div;
        };

        var createSingleLoginLayer = getSingle( createLoginLayer );

        document.getElementById( 'loginBtn' ).onclick = function(){
            var loginLayer = createSingleLoginLayer();
            loginLayer.style.display = 'block';
        };
        </script>
        </html>
~~~
#### 适用场景
- 频繁实例化然后又销毁的对象
- 经常使用的对象，但实例化时耗费时间或者资源多
- 线程池之类的控制资源时，资源之间的通信

### 策略模式

> **定义：** 定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。策略模式的实现并不复杂，关键是如何从策略模式的实现背后，找到封装变化、委托和多态性这些思想的价值

~~~javascript
      //很多公司的年终奖是根据员工的工资基数和年底绩效情况来发放的。例如，绩效为S的人年终奖有4倍工资，绩效为A的人年终奖有3倍工资，而绩效为B的人年终奖是2倍工资。假设财务部要求我们提供一段代码，来方便他们计算员工的年终奖。 
			// 面向对象实现
        var performanceS = function(){};

        performanceS.prototype.calculate = function( salary ){
            return salary * 4;
        };

        var performanceA = function(){};

        performanceA.prototype.calculate = function( salary ){
            return salary * 3;
        };

        var performanceB = function(){};

        performanceB.prototype.calculate = function( salary ){
            return salary * 2;
        };
~~~

~~~javascript
        var Bonus = function(){
            this.salary = null;      // 原始工资
            this.strategy = null;    // 绩效等级对应的策略对象
        };

        Bonus.prototype.setSalary = function( salary ){
            this.salary = salary;    // 设置员工的原始工资
        };

        Bonus.prototype.setStrategy = function( strategy ){
            this.strategy = strategy;    // 设置员工绩效等级对应的策略对象
        };

        Bonus.prototype.getBonus = function(){    // 取得奖金数额
            return this.strategy.calculate( this.salary );    // 把计算奖金的操作委托给对应的策略对象
        };
~~~

~~~javascript
        var bonus = new Bonus();

        bonus.setSalary( 10000 );
        bonus.setStrategy( new performanceS() );  // 设置策略对象

        console.log( bonus.getBonus() );    // 输出：40000

        bonus.setStrategy( new performanceA() );  // 设置策略对象
        console.log( bonus.getBonus() );    // 输出：30000
~~~

JavaScript函数实现

~~~javascript
        var strategies = {
            "S": function( salary ){
              return salary * 4;
            },
            "A": function( salary ){
              return salary * 3;
            },
            "B": function( salary ){
              return salary * 2;
            }
        };

        var calculateBonus = function( level, salary ){
            return strategies[ level ]( salary );
        };

        console.log( calculateBonus( 'S', 20000 ) );     // 输出：80000
        console.log( calculateBonus( 'A', 10000 ) );     // 输出：30000

				//方式二：
		    var S = function( salary ){
            return salary * 4;
        };

        var A = function( salary ){
            return salary * 3;
        };

        var B = function( salary ){
            return salary * 2;
        };

        var calculateBonus = function( func, salary ){
            return func( salary );
        };

        calculateBonus( S, 10000  );    // 输出：40000

~~~

通过对比实现，我们可以发现策略模式带来的优点有：

> - 利用组合、委托和多态等技术和思想，可以有效地避免多重条件选择语句。
> - 提供了对开放—封闭原则的完美支持，将算法封装在独立的strategy中，使得它们易于切换，易于理解，易于扩展。
> - 算法也可以复用在系统的其他地方，从而避免许多重复的复制粘贴工作。
> - 利用组合和委托来让Context拥有执行算法的能力，是继承的一种更轻便的替代方案。
#### 适用场景
> 当你负责的模块，基本满足以下情况时
> - 各判断条件下的策略相互独立且可复用
> - 策略内部逻辑相对复杂
> - 策略需要灵活组合

### 代理模式

> **定义：** 是为一个对象提供一个代用品或占位符，以便控制对它的访问

~~~javascript
        //例子： 图片预加载功能
        var myImage = (function(){
            var imgNode = document.createElement( 'img' );
            document.body.appendChild( imgNode );

            return function( src ){
              imgNode.src = src;
            }
        })();

        var proxyImage = (function(){
            var img = new Image;

            img.onload = function(){
              myImage( this.src );
            }

            return function( src ){
              myImage( 'file:// /C:/Users/svenzeng/Desktop/loading.gif' );
              img.src = src;
            }
        })();

        proxyImage( 'http://imgcache.qq.com/music// N/k/000GGDys0yA0Nk.jpg' );
~~~

缓存代理

~~~javascript
        // 计算乘积
        var mult = function(){
            console.log( ’开始计算乘积’ );
            var a = 1;
            for ( var i = 0, l = arguments.length; i < l; i++ ){
              a = a * arguments[i];
            }
            return a;
        };

        mult( 2, 3 );    // 输出：6
        mult( 2, 3, 4 );    // 输出：24

				// 加入缓存代理
				var proxyMult = (function(){
            var cache = {};
            return function(){
              var args = Array.prototype.join.call( arguments, ', ' );
              if ( args in cache ){
                  return cache[ args ];
              }
              return cache[ args ] = mult.apply( this, arguments );
            }
        })();

         proxyMult( 1, 2, 3, 4 );    // 输出：24
         proxyMult( 1, 2, 3, 4 );    // 输出：24
~~~

第二次调用proxyMult( 1, 2, 3, 4 )的时候，本体mult函数并没有被计算，proxyMult直接返回了之前缓存好的计算结果。通过增加缓存代理的方式，mult函数可以继续专注于自身的职责——计算乘积，缓存的功能是由代理对象实现的。

**用高级函数动态创建代理**

通过传入高阶函数这种更加灵活的方式，可以为各种计算方法创建缓存代理

~~~javascript
        /**************** 计算乘积 *****************/
        var mult = function(){
            var a = 1;
            for ( var i = 0, l = arguments.length; i < l; i++ ){
              a = a * arguments[i];
            }
            return a;
        };

        /**************** 计算加和 *****************/
        var plus = function(){
            var a = 0;
            for ( var i = 0, l = arguments.length; i < l; i++ ){
              a = a + arguments[i];
            }
            return a;
        };

        /**************** 创建缓存代理的工厂 *****************/
        var createProxyFactory = function( fn ){
            var cache = {};
            return function(){
              var args = Array.prototype.join.call( arguments, ', ' );
              if ( args in cache ){
                  return cache[ args ];
              }
              return  cache[ args ] = fn.apply( this, arguments );
            }
        };

        var proxyMult = createProxyFactory( mult ),
        proxyPlus = createProxyFactory( plus );

        alert ( proxyMult( 1, 2, 3, 4 ) );    // 输出：24
        alert ( proxyMult( 1, 2, 3, 4 ) );    // 输出：24
        alert ( proxyPlus( 1, 2, 3, 4 ) );    // 输出：10
        alert ( proxyPlus( 1, 2, 3, 4 ) );    // 输出：10
~~~
#### 适用场景
> 当你负责的模块，基本满足以下情况时
> - 模块职责单一且可复用
> - 两个模块间的交互需要一定限制关系

### 迭代器模式

> **定义：** 提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。迭代器模式可以把迭代的过程从业务逻辑中分离出来，在使用迭代器模式之后，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素。

#### 内部迭代器

函数内部定义迭代规则，完全接手整个迭代过程，外部只需要一次初始调用，不用关心迭代器内部的实现。

~~~javascript
        var each = function( ary, callback ){
            for ( var i = 0, l = ary.length; i < l; i++ ){
              callback.call( ary[i], i, ary[ i ] );  // 把下标和元素当作参数传给callback函数
            }
        };

        each( [ 1, 2, 3 ], function( i, n ){
            alert ( [ i, n ] );
        });
~~~

#### 外部迭代器

必须显式地请求迭代下一个元素，增加了一些调用的复杂度，但相对也增强了迭代器的灵活性，我们可以手工控制迭代的过程或者顺序

```javascript
        var Iterator = function( obj ){
            var current = 0;

            var next = function(){
              current += 1;
            };

            var isDone = function(){
              return current >= obj.length;
            };

            var getCurrItem = function(){
              return obj[ current ];
            };

            return {
              next: next,
              isDone: isDone,
              getCurrItem: getCurrItem
              length: obj.length
            }
        };

				var compare = function( iterator1, iterator2 ){
            if(iterator1.length ! == iterator2.length){
              alert('iterator1和iterator2不相等’);
            }
            while( ! iterator1.isDone() && ! iterator2.isDone() ){
              if ( iterator1.getCurrItem() ! == iterator2.getCurrItem() ){
                    throw new Error ( 'iterator1和iterator2不相等’ );
              }
              iterator1.next();
              iterator2.next();
            }

            alert ( 'iterator1和iterator2相等’ );
        }
```

外部迭代器虽然调用方式相对复杂，但它的适用面更广，也能满足更多变的需求
#### 优缺点
> **优点 :**  分离 了 集合对象 的 遍历行为 ; 抽象出了 迭代器 负责 集合对象的遍历 , 可以让外部的代码 透明的 访问集合内部的数据 ;
> **缺点 :**  类的个数成对增加 ; 迭代器模式 , 将 存储数据 , 遍历数据 两个职责拆分 ; 如果新添加一个 集合类 , 需要增加该 集合类 对应的 迭代器类 , 类的个数成对增加 , 在一定程度上 , 增加了系统复杂性 ;
#### 适用场景
- **内容保密 :**  访问 集合对象 的内容 , 无需暴露内部表示 ;
- **统一接口 :**  为遍历 不同的 集合结构 , 提供统一接口 ;

### 发布—订阅模式

> **定义：** 又叫观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。在JavaScript开发中，我们一般用事件模型来替代传统的发布—订阅模式

~~~javascript
		//小明找中介购房例子
     var Event = (function(){

            var clientList = {},
              listen,
              trigger,
              remove;

            listen = function( key, fn ){
              if ( ! clientList[ key ] ){
                  clientList[ key ] = [];
              }
              clientList[ key ].push( fn );
            };

            trigger = function(){
              var key = Array.prototype.shift.call( arguments ),
                  fns = clientList[ key ];
                  if ( ! fns || fns.length === 0 ){
                      return false;
                  }
                  for( var i = 0, fn; fn = fns[ i++ ]; ){
                      fn.apply( this, arguments );
                  }

            };

            remove = function( key, fn ){
              var fns = clientList[ key ];
              if ( ! fns ){
                  return false;
              }
              if ( ! fn ){
                  fns && ( fns.length = 0 );
              }else{
                  for ( var l = fns.length -1; l >=0; l-- ){
                    var _fn = fns[ l ];
                    if ( _fn === fn ){
                        fns.splice( l, 1 );
                    }
                  }
              }
          };

          return {
              listen: listen,
              trigger: trigger,
              remove: remove
          }

        })();

        Event.listen( 'squareMeter88', function( price ){     // 小红订阅消息
          console.log( ’价格= ' + price );       // 输出：’价格=2000000'
        });

        Event.trigger( 'squareMeter88', 2000000 );    // 售楼处发布消息 
~~~
通过上面基于全局Event对象的实现，就能实现在模块在完全不知道对方存在的情况下进行通信。
#### 优缺点
> **优点：** 时间上或对象之间的解藕、用在异步编程中；可以帮助我们完成更松耦合的代码编写
> **缺点：** 创建订阅者本身要消耗一定时间和内存，当订阅一个消息后，或许消息最后都没发生，但订阅者始终存在与内存中。另外过度使用的话，对象和对象之间的必然联系也就被深埋在背后，导致程序难以跟踪维护和理解，定位bug难。
#### 适用场景
> - 完全解藕的对象之间的通信
> - 事件驱动系统
> - 新闻网站推送系统


### 装饰者模式

> **定义：** 可以动态地给某个对象添加一些额外的职责，而不会影响从这个类中派生的其他对象。

使用AOP函数实现数据上报功能

~~~javascript
        <html>
            <button tag="login" id="button">点击打开登录浮层</button>
            <script>

            var showLogin = function(){
              console.log( ’打开登录浮层’ );
              log( this.getAttribute( 'tag' ) );
            }

            var log = function( tag ){
              console.log( ’上报标签为： ' + tag );
              // (new Image).src = 'http://xxx.com/report? tag=' + tag;    // 真正的上报代码略
            }

            document.getElementById( 'button' ).onclick = showLogin;

            </script>
        </html>
~~~
我们看到在showLogin函数里，既要负责打开登录浮层，又要负责数据上报，这是两个层面的功能，在此处却被耦合在一个函数里。使用AOP分离之后，代码如下：
~~~javascript
        <html>
            <button tag="login" id="button">点击打开登录浮层</button>
            <script>

            Function.prototype.after = function( afterfn ){
              var __self = this;
              return function(){
                  var ret = __self.apply( this, arguments );
                  afterfn.apply( this, arguments );
                  return ret;
              }
            };

            var showLogin = function(){
              console.log( ’打开登录浮层’ );
            }

            var log = function(){
              console.log( ’上报标签为： ' + this.getAttribute( 'tag' ) );
            }

            showLogin = showLogin.after( log );    // 打开登录浮层之后上报数据

            document.getElementById( 'button' ).onclick = showLogin;
            </script>
        </html>
~~~
装饰者模式和代理模式的结构看起来非常相像，两种模式都描述了怎样为对象提供一定程度上的间接引用，它们的实现部分都保留了对另外一个对象的引用，并且向那个对象发送请求，而二者的区别在于它们的意图和设计目的。
>- **代理模式目的：** 当直接访问本体不方便或者不符合需要时，为这个本体提供一个替代者。本体定义了关键功能，而代理提供或拒绝对它的访问，或者在访问本体之前做一些额外的事情。
>- **装饰者模式目的：** 一开始不能确定对象的全部功能时，就为对象动态加入行为。经常会形成一条长长的装饰链。

### 责任链模式
> **定义：** 使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系，将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。
~~~javascript
  //例子：一个售卖手机的电商网站，公司针对支付过定金的用户有一定的优惠政策。在正式购买后，已经支付过500元定金的用户会收到100元的商城优惠券，200元定金的用户可以收到50元的优惠券，而之前没有支付定金的用户只能进入普通购买模式，也就是没有优惠券，且在库存有限的情况下不一定保证能买到。

  // 原始实现：
          var order = function( orderType, pay, stock ){
            if ( orderType === 1 ){        // 500元定金购买模式
              if ( pay === true ){    // 已支付定金
                  console.log( '500元定金预购，得到100优惠券’ );
              }else{    // 未支付定金，降级到普通购买模式
                  if ( stock > 0 ){    // 用于普通购买的手机还有库存
                      console.log( ’普通购买，无优惠券’ );
                  }else{
                      console.log( '手机库存不足' );
                  }
              }
            }

            else if ( orderType === 2 ){     // 200元定金购买模式
              if ( pay === true ){
                  console.log( '200元定金预购, 得到50优惠券' );
              }else{
                  if ( stock > 0 ){
                      console.log( '普通购买, 无优惠券' );
                  }else{
                      console.log( '手机库存不足' );
                  }
              }
            }

            else if ( orderType === 3 ){
              if ( stock > 0 ){
                  console.log( '普通购买, 无优惠券' );
              }else{
                  console.log( '手机库存不足' );
              }
            }
        };

        order( 1 , true, 500);  // 输出： 500元定金预购, 得到100优惠券
~~~
虽然我们得到了意料中的运行结果，但这远远算不上一段值得夸奖的代码。order函数不仅巨大到难以阅读，而且需要经常进行修改。虽然目前项目能正常运行，但接下来的维护工作无疑是个梦魇。恐怕只有最“新手”的程序员才会写出这样的代码。
#### 用职责链模式重构代码
~~~javascript
        var order500 = function( orderType, pay, stock ){
            if ( orderType === 1 && pay === true ){
              console.log( '500元定金预购，得到100优惠券’ );
            }else{
              return 'nextSuccessor';    // 我不知道下一个节点是谁，反正把请求往后面传递
            }
        };

        var order200 = function( orderType, pay, stock ){
            if ( orderType === 2 && pay === true ){
              console.log( '200元定金预购，得到50优惠券’ );
            }else{
              return 'nextSuccessor';    // 我不知道下一个节点是谁，反正把请求往后面传递
            }
        };

        var orderNormal = function( orderType, pay, stock ){
            if ( stock > 0 ){
              console.log( ’普通购买，无优惠券’ );
            }else{
              console.log( ’手机库存不足’ );
            }
        };

        //定义一个构造函数Chain，在new Chain的时候传递的参数即为需要被包装的函数，同时它还拥有一个实例属性this.successor，表示在链中的下一个节点
        // Chain.prototype.setNextSuccessor  指定在链中的下一个节点
        // Chain.prototype.passRequest  传递请求给某个节点

        var Chain = function( fn ){
            this.fn = fn;
            this.successor = null;
        };

        Chain.prototype.setNextSuccessor = function( successor ){
            return this.successor = successor;
        };

        Chain.prototype.passRequest = function(){
            var ret = this.fn.apply( this, arguments );

            if ( ret === 'nextSuccessor' ){
                return this.successor && this.successor.passRequest.apply( this.successor, arguments );
            }

            return ret;
        };
        //把3个订单函数分别包装成职责链的节点
        var chainOrder500 = new Chain( order500 );
        var chainOrder200 = new Chain( order200 );
        var chainOrderNormal = new Chain( orderNormal );
        //指定节点在职责链中的顺序：
        chainOrder500.setNextSuccessor( chainOrder200 );
        chainOrder200.setNextSuccessor( chainOrderNormal );
        //把3个订单函数分别包装成职责链的节点
        chainOrder500.passRequest( 1, true, 500 );    // 输出：500元定金预购，得到100优惠券
        chainOrder500.passRequest( 2, true, 500 );    // 输出：200元定金预购，得到50优惠券
        chainOrder500.passRequest( 3, true, 500 );    // 输出：普通购买，无优惠券
        chainOrder500.passRequest( 1, false, 0 );     // 输出：手机库存不足
~~~
通过改进，我们可以自由灵活地增加、移除和修改链中的节点顺序，假如某天网站运营人员又想出了支持300元定金购买，那我们就在该链中增加一个节点即可:
~~~javascript
        var order300 = function(){
            // 具体实现略
        };

        chainOrder300= new Chain( order300 );
        chainOrder500.setNextSuccessor( chainOrder300);
        chainOrder300.setNextSuccessor( chainOrder200);
~~~
#### 优缺点
- 优点：解耦了请求发送者和N个接收者之间的复杂关系，由于不知道链中的哪个节点可以处理你发出的请求，所以你只需把请求传递给第一个节点即可
- 缺点：程序中多了一些节点对象，可能在某一次的请求传递过程中，大部分节点并没有起到实质性的作用，它们的作用仅仅是让请求传递下去，带来的性能损耗。

#### 适用场景
> 当你负责的模块，基本满足以下情况时
> - 你负责的是一个完整流程，或你只负责流程中的某个环节
> - 各环节有一定的执行顺序
> - 各环节可重组
> - 各环节可复用

### 总结
本文主要是在自己花34小时读完《JavaScript设计模式与开发实践》一书后，针对前端较常用的：单例模式、策略模式、代理模式、迭代器模式、发布订略模式、装饰者模式、责任链模式七种设计模式，通过具体的例子（案例均出自《JavaScript设计模式与开发实践》）从不使用设计模式到使用设计模式的代码实现对比做的粗略梳理，凸显出使用设计模式从可读性、可维护性、可扩展性多个维度带来的便利。也算是个人对设计模式粗略认知理解后的总结，希望能给大家带来收获。**文章有不足的地方，欢迎大家提出宝贵意见**。

### 参考文献
- 《JavaScript设计模式与开发实践》
- [前端渣渣唠嗑一下前端中的设计模式](https://juejin.cn/post/6844904138707337229?searchId=2023082316451460F5837957FC2630B492)
