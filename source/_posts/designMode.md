---
title: 前端设计模式总结
layout: post
date: 2023-08-05 11:18
comments: true
tags: 
	- javascript
key: "1"
---

### 前言

>   设计模式在我最初的认知中，总以为它是后端Java开发领域里的专有名词。在这样的认知下，对于作为一个前端开发者的我而言接触的并不多，愚昧的认为前端就不存在设计模式这一说法。然而在时间的推移下，工作之余通过对掘金平台相关技术文章的阅读及对vue源码调试与阅读，加上最近刚刚读完《JavaScript设计模式与开发实践》一书后。终于对设计模式有了系统的了解，也由此打破了之前对设计模式之存在与后端Java开发语言的局限认知，下面主要针对最近读完的相关设计模式一书后，梳理下个人设计模式全新的认识。

<!--more-->

<img src="/Users/zhanglinchong/Downloads/weixinread.jpg" alt="weixinread" style="height:700px;width:60%" />

### 什么是设计模式

设计模式并非是软件开发中的专业术语。其实，“模式”最早诞生于建筑学，当时主要是从**研究解决同一个问题而设计出不同的建筑结构，并从中发现了一些高质量设计中的相似性**，并用“模式”来指代这种相似性。

> 设计模式的定义是：**在面向对象软件设计过程中针对特定问题的简洁而优雅的解决方案。**比较通俗的说就是在某种场合下对某个问题的一种解决方案。

设计模式大体可分为：创建型， 行为型，结构型。

- 创建型：是为了封装创建对象的变化，如：**单例、抽象工厂、建造者、工厂、原型**
- 行为型：是为了封装对象相关行为的变化，如：**模板方法、命令模式、迭代器模式、观察者、中介者模式、备忘录模式、访问者模式**
- 结构型：是为了封装对象之间的组合关系，通过对结构的设计达到性能、可扩展性、业务目的的代码写法，如：**适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式**

可以看出细分后的设计模式达到了23种，下面让我们针对前端常用的设计模式做一个详细的了解，再了解前端设计模式之前需要掌握如下基本概念知识点：

- 面向对象的三大特性：继承、多态、封装
- JavaScript里的关键字this，以及常用方法call、apply
- 闭包、高阶函数

**注意：**就设计模式而言，因为JavaScript这门语言的自身特点，许多设计模式在JavaScript之中的实现跟在一些传统面向对象语言中的实现相差很大。在JavaScript中，很多设计模式都是通过闭包和高阶函数实现的。

### 单例模式

> 定义：保证一个类仅有一个实例，并提供一个访问它的全局访问点

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

前端领域常见单例模式有全局缓存、浏览器window对象、vue里全局store

### 策略模式

> 定义：定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。策略模式的实现并不复杂，关键是如何从策略模式的实现背后，找到封装变化、委托和多态性这些思想的价值

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

策略模式的一些优点。

- 策略模式利用组合、委托和多态等技术和思想，可以有效地避免多重条件选择语句。

- 策略模式提供了对开放—封闭原则的完美支持，将算法封装在独立的strategy中，使得它们易于切换，易于理解，易于扩展。

- 策略模式中的算法也可以复用在系统的其他地方，从而避免许多重复的复制粘贴工作。

- 在策略模式中利用组合和委托来让Context拥有执行算法的能力，这也是继承的一种更轻便的替代方案。

### 代理模式

> 定义：是为一个对象提供一个代用品或占位符，以便控制对它的访问

图片预加载功能

~~~javascript
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

### 迭代器模式

> 定义：提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。迭代器模式可以把迭代的过程从业务逻辑中分离出来，在使用迭代器模式之后，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素。

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

### 发布—订阅模式

> 定义：发布—订阅模式又叫观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。在JavaScript开发中，我们一般用事件模型来替代传统的发布—订阅模式

~~~javascript
		//小明找中介购房例子
				var event = {
            clientList: [],
            listen: function( key, fn ){
              if ( ! this.clientList[ key ] ){
                  this.clientList[ key ] = [];
              }
              this.clientList[ key ].push( fn );    // 订阅的消息添加进缓存列表
            },
            trigger: function(){
              var key = Array.prototype.shift.call( arguments ),    // (1);
                  fns = this.clientList[ key ];

              if ( ! fns || fns.length === 0 ){    // 如果没有绑定对应的消息
                  return false;
              }

              for( var i = 0, fn; fn = fns[ i++ ]; ){
                  fn.apply( this, arguments );    // (2) // arguments是trigger时带上的参数
              }
            }
        };
				//定义installEvent函数可以给所有的对象都动态安装发布—订阅功能
			  var installEvent = function( obj ){
            for ( var i in event ){
              obj[ i ] = event[ i ];
            }
        };

			  var salesOffices = {};
        installEvent( salesOffices );

        salesOffices.listen( 'squareMeter88', function( price ){    // 小明订阅消息
            console.log( ’价格= ' + price );
        });

        salesOffices.listen( 'squareMeter100', function( price ){     // 小红订阅消息
            console.log( ’价格= ' + price );
        });

        salesOffices.trigger( 'squareMeter88', 2000000 );    // 输出：2000000
        salesOffices.trigger( 'squareMeter100', 3000000 );    // 输出：3000000
       //取消订略事件
        event.remove = function( key, fn ){
            var fns = this.clientList[ key ];

            if ( ! fns ){    // 如果key对应的消息没有被人订阅，则直接返回
              return false;
            }
            if ( ! fn ){    // 如果没有传入具体的回调函数，表示需要取消key对应消息的所有订阅
              fns && ( fns.length = 0 );
            }else{
              for ( var l = fns.length -1; l >=0; l-- ){    // 反向遍历订阅的回调函数列表
                  var _fn = fns[ l ];
                  if ( _fn === fn ){
                      fns.splice( l, 1 );    // 删除订阅者的回调函数
                  }
              }
            }
        };

        var salesOffices = {};
        var installEvent = function( obj ){
            for ( var i in event ){
              obj[ i ] = event[ i ];
            }
        }

        installEvent( salesOffices );

        salesOffices.listen( 'squareMeter88', fn1 = function( price ){    // 小明订阅消息
            console.log( ’价格= ' + price );
        });

        salesOffices.listen( 'squareMeter88', fn2 = function( price ){    // 小红订阅消息
            console.log( ’价格= ' + price );
        });

        salesOffices.remove( 'squareMeter88', fn1 );    // 删除小明的订阅
        salesOffices.trigger( 'squareMeter88', 2000000 );     // 输出：2000000
          
        
          
~~~

优点：

- 为时间上的解耦
- 为对象之间的解耦
- 应用非常广泛，既可以用在异步编程中，也可以帮助我们完成更松耦合的代码编写

### 装饰者模式

> 定义：可以动态地给某个对象添加一些额外的职责，而不会影响从这个类中派生的其他对象。

使用AOP函数实现数据上报功能

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

