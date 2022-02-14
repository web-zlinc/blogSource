---
layout: post
title: "winged 框架学习"
date: 2021-09-16 16:11
comments: true
tags: 
	 - winged
   - winged-cli
---

## 前言

> Winged 是一个跨平台、现代化、ts-first 的泛前端框架，目前支持微信小程序、网页开发。

<!--more-->

## 主要功能

- es6 class 语法描述视图
- 包含类 html 模版语言。支持双向数据绑定、事件绑定、逻辑表述
- 支持数据变更检测（vWatch）和计算属性（vComputed）_eventData_: this['SE']['changetodoitem']
- 提供组件化能力
- 基于代码生成 t s 类型约束
- 提供命令行接口（winged cli）

  - 创建项目

  - 提供开发环境，包括编译功能，本地预览服务器，图片自动上传，关联重命名，过时文件清理等

  - 代码生成功能，能够快速生成页面（Page）或组件（Component）所需的各个文件

- 提供 tslint 和 tsconfig 最佳实践

## 项目结构

```typescript
.
  |-src/                          # 源代码目录
  |---html/                       # 存放web使用的html入口文件，通常只会有index.html（小程序中也会有此文件夹，以后会用到，但是现在不用管）
  |---imgs/                       # 存放图片资源的文件夹，cli会自动上传图片到cdn
  |---less/                       # 存放less文件
  |-----base/                     # 存放只被require而不需要生成css/wxss的文件
  |-------variables.less
  |-------mixins.less
  |-----components/               # 存放组件使用的less文件
  |-----pages/                    # 存放页面使用的less文件。在小程序中只有这里的的文件会生成对应的wxss
  |-------home-page.less
  |---ts/                         # 存放ts文件
  |-----components/               # 存放组件使用的ts文件
  |-----pages/                    # 存放组件使用的ts文件，在小程序中只有这里的文件会生成对应的wxml
  |-------HomePage.ts
  |-----v/                        # 存放由视图结构文件生成的v文件，文件以.v.ts结尾
  |-------components/             # 存放由组件类视图结构生成的v文件
  |---------InputGroup.v.ts
  |-------pages/                  # 存放由页面类视图结构生成的v文件
  |---------HomePage.v.ts
  |-----app.ts                    # 小程序入口文件
  |-----index.ts                  # 网页入口文件，包含网页路由
  |---viewTpls/                   # 存放视图结构文件，文件为html格式，包含winged独有的逻辑标签和绑定点
  |-----components/               # 存放组件类视图结构
  |-------input-group.html        # 文件名无要求
  |-----pages/                    # 存放页面类视图结构
  |-------home-page.html          # html文件名必须以 -page 结尾
  |-web/                          # 网页输出目录，编译打包后的网页资源会放入此文件夹。此文件夹应该被 search.exclude
  |-wxapp/                        # 小程序输出目录，编译打包后的小程序资源会放入此文件夹。此文件夹应该作为微信开发者工具的根目录
  |---app.json
  |---libs/                       # 库文件代码目录，目前会存放编译好的winged.js文件
  |-----winged.js
  |---components/                 # 组件逻辑代码目录，只有js
  |-----InputGroup.js             # 由 InputGroup.ts 编译生成
  |---pages/                      # 页面资源目录，包括页面逻辑对应的js，和页面的其他资源文件（wxss，wxml）
  |-----home.wxml                 # 由 home-page.html "推导"生成
  |-----home.js                   # 由 HomePage.ts "推导"生成，仅会引用 HomePage.js 并构建页面。页面逻辑在 HomePage.js 中
  |-----home.json                 # 由 HomePage.ts "推导"生成
  |-----home.wxss                 # 由 home-page.less 编译生成
  |-----HomePage.js               # 由 HomePage.ts 编译生成
  |---v/                          # 存放编译好的v文件，供其他文件require
  |-----...                       # 内容略
  |- winged.json                  # winged项目配置文件，此文件存在时才会被识别为合法的winged项目
  |- devup.sh                     # 开发环境快速启动脚本，执行此文件可以快速运行开发环境
```

## 基本概念

> Winged 和其他框架主要的不同，在于会通过 html 生成对应的 `v.ts` 文件；`v.ts`会暴露一个抽象类（typescript abstract class），使用者通过在自己的 ts 文件编写一个类(class)并继承它，来使用`Winged`提供的功能。

### 文件概念

- [视图结构文件](https://www.yuque.com/n_tech/winged/zppy00)（view struct file）**:** 用于生成`v.ts`文件的 html，也叫或**视图模板**（view tpl, tpl=template）

- **抽象视图文件**（abs view file, abs=abstract）**:**由视图结构文件生成的`v.ts`文件，也叫**v 文件**(v file)

- **实现视图文件**(impl view file, impl=implementation)**:**将继承并实现这个 v 文件的 ts 文件，也称为**视图类文件**(view class file)

### 视图结构文件（viewTpl file）

在`src/viewTpls/pages/`路径下编写 html 结构页面

> *注意：*视图结构文件的名称必须采用中划线命名法，采用 html 后缀（小程序中也需要采用 html 后缀）。页面需要以 `-page` 结尾，但是组件则 **不** 需要以 `-component` 结尾。

```html
<section id="book-page">
  <p>Please input book name:</p>
  <input type="text" @value="bookName" />
  <If cond="tooLong">
    // IF为逻辑节点
    <p>Name too long</p>
    <Else />
    <p>Hello, {{bookName}}</p>
  </If>
  <button @bind:click="handleSubmit">Submit</button> //@bind:click为虚拟属性
</section>
```

#### 数据绑定点

出现位置如下：

- 任意一般标签和逻辑节点的内容部分，如: `<view>{{a}}</view>`

- 任意一般标签（不包含逻辑节点）的非虚拟属性部分：如: `<image class="icon {{iconClass}}"src="{{host}}/a.png"></image>`

**注意：**1、不可出现在逻辑节点的属性中。2、不可出现在一般标签的虚拟属性中

##### 支持语法形式如下

- 字符串输出: `{{name}}` 或者 `{{user.name}}`。

  - 如果为多级路径，前置路径为空时不会报错。

  - 目前**不**支持数组下标，比如`{{list[1].name}}`是不支持的

- 或操作： `{{a||'a not exists'}}`

- 三元表达式： `{{a == 'yes' ? 'yes':'no'}}` 或者 `{{a ? a:'a not exists'}}`

- **注意：**目前在数据点中，值支持变量引用或者字符串，因此写数字或布尔值都会导致错误。比如可以写`{{a == 'a'?'yes'}}`，但是不能写`{{a == 1?'yes'}}` 或者 `a=='a'?true` 请注意

#### 逻辑节点

与普通节点相同，但是首字母要大写

- 逻辑节点可以是双标签也可以是单标签，但标签必须显示闭合
- 逻辑节点是一个独立的逻辑体，都有自己与许的属性，除此之外其他属性皆不允许
- 可能含有必填属性
- **逻辑节点中所有属性中皆不可包含 DataPoint 的双花括号，但有些属性可能会共用 DataPoint 的解析规则（If 的 cond 属性）**

##### `<If></If>, <Elif/> 和 <Else/>`

```html
<!-- 页面视图文件 url : src/viewTpls/pages/home-page.html -->
<section>
  <If cond="!userName">
    <h1>No name</h1>
    <Elif cond="tooLong" />
    <h1>Name too long</h1>
    <Else />
    <h1>Hello, {{userName}}</h1>
  </If>
</section>
```

##### `<For></For>`

- 通过 `list` 属性绑定循环列表 （必须指定此属性）。
- 通过 `item` 属性拿到每次循环的值 (必须指定此属性)。

- 通过 `index` 属性拿到每次循环的索引。

```html
<!-- 页面视图文件 url : src/viewTpls/pages/home-page.html -->
<section>
  <ul>
    <For list="someList" item="li" index="idx">
      <!-- 基于list渲染 -->
      <li>
        <span>{{idx}}:</span>
        <span>name - {{li.name}}</span>
        <span>- age - {{li.age}}</span>
      </li>
      <!-- 嵌套流程控制节点 -->
      <If cond="li.name=='Yao'">
        <h1>bingo~</h1>
        <Elif cond="idx==0" />
        <h1>bingo!</h1>
      </If>
      <!-- 支持多层级 For循环 -->
      <ul>
        <For list="li.chlidren" item="li">
          <li>层级2: {{li}}</li>
        </For>
      </ul>
    </For>
  </ul>
</section>
```

##### `<Slot/> 和 <Plugin/>`

```html
<!-- src/viewTpls/components/example-item.html -->
<section id="example-item">
  left:
  <slot name="left"></slot>
  right:
  <slot name="right"></slot>
  默认值:
  <slot name="else">
    <h1>我是Slot的默认值</h1>
  </slot>
</section>
```

```html
<!-- src/viewTpls/pages/home-page.html -->

<section>
  <!-- 基础使用 -->
  <ExampleItem>我是插槽,默认匹配第一个Slot节点</ExampleItem>

  <!-- 具名插槽 (利用 Plugin 的 slot字段 匹配 Slot 的name 字段)-->
  <ExampleItem>
    <Plugin slot="left">
      <h1>我是左插槽</h1>
    </Plugin>
    <Plugin slot="right">
      <h1>我是右插槽</h1>
    </Plugin>
  </ExampleItem>

  <!-- 列表中使用 -->
  <For list="someList" item="li" index="idx">
    <ExampleItem>
      <Plugin slot="left">
        <h1>{{li.left}}</h1>
      </Plugin>
      <Plugin slot="right">
        <h1>{{li.right}}</h1>
      </Plugin>
    </ExampleItem>
  </For>
</section>
```

#### 虚拟属性

##### 双向绑定

```html
<input @value="myValue" />
```

##### 视图逻辑中使用

```typescript
export class BookPage extends BookPageV<BookPage> {
  public onLoad(params: { [key: string]: string }): void {
    // 设置初始值
    this.myValue = '初始值'
  }

  // 监测值的改变
  @vWatch('value')
  protected handleValueChange() {
    console.log(this.myValue)
    const strValue = this.myValue as string
    if (strValue && strValue.length > 5) {
      // 直接修改属性可以造成输入框中中值改变
      this.myValue = strValue.slice(0, 5)
    }
  }
}
```

##### 事件绑定 @bind:[eventName] 和 @catch:[eventName]

```html
<!-- 无参数 -->
<button @bind:click="handleBtnClick">Btn</button>
<!-- 有参数 -->
<button @bind:click="handleBtn2Click({isBtn2:true, name})">Btn2</button>
```

> ```
> @bind` 和 `@catch` 的区别是 `@catch` 会阻止事件冒泡。在小程序中会编译成内建的catch监听，web上会自动调用`stopPropagation
>
> eventName可以填写任意的合法事件，其中一些事件会进行隐式转换，提提升兼容性。 目前会转换的事件有:
> :click 事件在小程序中会被转换成 :tap
> ```

##### 布尔属性 @attr:[name]

```html
<!-- 存在属性 -->
<input type="text" disabled />
<!-- 不存在属性 -->
<input type="text" />

<!-- 逻辑判断-->
<input type="" @attr:disabled="!inputDisabled" />
<input type="" @attr:disabled="activeInput!='myInput'" />

<!-- 错误的形式 -->
<input type="text" disabled="false" />
```

##### 元素引用 @ref

**注意**：在小程序中不可使用 `@ref`

```html
<!-- 视图结构文件 -->
<div class="some-element">
  <button @ref="btn">按钮1</button>
  <If cond="someCond">
    <p @ref="p">这个段落只有在someCond为true时才会存在</p>
  </If>
</div>
```

```typescript
// 对应视图实现文件
class SomeComponent extends SomeComponentV<SomeComponent> {
  protected onMount() {
    // 例1:
    // this.refs.btn 可以获取 @ref="btn" 对应元素，其类型为 HTMLButtonElement
    // 注意，由于生命周期和视图结构的影响，this.refs.btn 有可能未空
    // 在 Component 中至少要等到 onMount 调用后才会有值
    const btnElement = this.refs.btn

    // 例2-1:
    // 由于对 someCond 的改变并不会立即渲染
    // 所以下面的 pElement 会为空，要注意判断
    this.someCond = true
    const pElement = this.refs.p

    // 例2-2:
    // 如果要避免例2-1中的问题，可以改成如下形式
    this.setState({ someCond: true }, () => {
      const pElement = this.refs.p
    })
  }
}
```

### 抽象视图文件（v file）

### 视图实现文件（view file）

#### 组件实现文件

> **组件实现文件是一个组件逻辑部分的主干，所有组件相关的控制代码都在这里实现。组件实现文件会暴露一个主实现类，他继承自组件抽象文件(V 文件)的主类**

```html
<!-- src/viewTpls/components/example-item.html -->
<section>
  <h1>接受父组件的参数 => {{info}}</h1>
</section>

<!-- src/viewTpls/pages/home-page.html -->
<section>
  <ExampleItem data="meow~~~"></ExampleItem>
</section>
```

```typescript
// src/ts/compoents/ExampleItem.ts
import { ExampleItemV } from '../v/components/ExampleItem.v'

export class ExampleItem extends ExampleItemV<ExampleItem> {
  /** 纯类型属性，定义了组件能够接收的属性，不需要赋值 */
  public propsType!: {
    data: string
  }
  public eventsType!: {}

  /**
   * 生命周期方法,可以拿到props参数,类型为this['propsType']
   * 将拿到的data 赋值给 数据绑定点 info
   * 详细的生命周期,会在后面逐一介绍.
   */
  public onPropsChange(props: this['propsType']): void {
    this.info = props.data
  }
}

// src/ts/pages/HomePage.ts
import { PageParams } from 'winged'
import { HomePageV } from '../v/pages/HomePage.v'

export class HomePage extends HomePageV<HomePage> {
  public onLoad(params: PageParams): void {}
}
```

##### eventsType

用来定义组件产生的事件类型,常用于子向父传参

```html
<!-- 视图结构 -->
<!-- src/viewTpls/components/example-item.html -->
<section>
  <button @bind:click="handleClick">与父组件通信</button>
</section>

<!-- src/viewTpls/pages/home-page.html -->
<section>
  <ExampleItem :getInfo="handleGetInfo"></ExampleItem>
  <h1>{{info}}</h1>
</section>
```

```typescript
// 视图实现
// src/ts/compoents/ExampleItem.ts
import { ExampleItemV } from '../v/components/ExampleItem.v'

export class ExampleItem extends ExampleItemV<ExampleItem> {
  public propsType!: {}
  /** 纯类型属性,定义了组件能够产生的事件,约束了emitEvent的参数类型,用来与父组件通信,不需要赋值 */
  public eventsType!: {
    /** 格式为 事件名 getInfo , 参数类型 {info :string} */
    getInfo: {
      info: string
    }
  }
  public onPropsChange(props: this['propsType']): void {}

  /**
   * 视图事件处理，对应结构文件中的 @bind:click="handleClick"
   * 函数体中 this.emitEvent 方法的参数 由 上面定义的 eventsType 约束
   */
  protected handleClick() {
    this.emitEvent('getInfo', { info: '向父亲传参' })
  }
}

// src/ts/pages/HomePage.ts
import { PageParams } from 'winged'
import { HomePageV } from '../v/pages/HomePage.v'

export class HomePage extends HomePageV<HomePage> {
  public onLoad(params: PageParams): void {
    this.info = ''
  }
  /**
   * 响应子组件发射的事件
   * 接受的参数类型 eventData 通过this['SE']['函数名']获取
   */
  public handleGetInfo(eventData: this['SE']['handleGetInfo']) {
    this.info = eventData.info
  }
}
```

##### component 生命周期

- **onMount()：**会在组件被创建时调用，在组件被销毁前只会触发一次。
- **onPropsChange()：**会在组件的 props 变化时调用。在组件被销毁前可能会多次触发。
- **onBeforeDestory()：**在组件销毁前调用。

#### 页面实现文件

> 是一个页面逻辑部分的主干，所有页面相关的控制代码都在这里实现。页面实现文件会暴露一个实现类，继承自页面抽象文件（V 文件）的主类。

**步骤：**

- 创建视图文件

  ```html
  <!-- 页面视图文件 url : src/viewTpls/pages/home-page.html -->
  <section>Hello,World</section>
  ```

- 编写视图实现文件，用来继承自内部自动生成的 V 文件主类

  创建完视图文件，并且编译好 V 文件后，在控制台输入一下命令

  ```bash
  winged impl
  ```

  ```typescript
  import { PageParams } from 'winged'
  import { HomePageV } from '../v/pages/HomePage.v'

  export class HomePage extends HomePageV<HomePage> {
    public onLoad(params: PageParams): void {
      // TODO:
    }
  }
  ```

##### 实现事件绑定

```html
<!-- 页面视图文件 url : src/viewTpls/pages/home-page.html -->
<section>
  <For list="list" item="item" index="index">
    <button @bind:click="handleClick({index})">{{item}}</button>
  </For>
</section>
```

```typescript
// 视图实现文件
import { PageParams } from 'winged'
import { HomePageV } from '../v/pages/HomePage.v'

export class HomePage extends HomePageV<HomePage> {
  // tslint:disable-next-line: no-empty
  public onLoad(params: PageParams): void {
    this.list = ['a', 'b', 'c']
  }
  protected handleClick(event: MouseEvent, eventData: this['E']['handleClick']): void {
    console.log(eventData.index)
  }

```

- 第一个参数接受是事件对象 类型为 MouseEvent, 这是 TS 内部支持的.
- 第二个参数接受的是 eventData ,也就是 `@bind:click="handleClick({index})"` 传递的 list 遍历的 index ,他的类型 存储在 V 文件中 `public E!: {handleClick: { index: number }}` 所以需要 通过`this['E']['handleClick']` 的方式来获取。

##### page 生命周期

- onLoad(params: PageParams)整个生命周期内只会触发一次
- onShow()整个生命周期内只会触发一次
- onReady()整个生命周期内只会触发一次

执行顺序 : onLoad()=> onShow() => onReady()

### 父子组件通信

> Winged 中，**数据由父向子传递，事件由子向父传递**

```html
<!-- User 页面 (src/viewTpls/pages/user-page.html) -->
<section>
  <For list="users" item="user" index="index">
    <UserInfo
      user="{{user}}"
      :click="handleUserClick({user, index})"
    ></UserInfo>
  </For>
</section>

<!-- UserInfo 组件 (src/viewTpls/components/user-info.html) -->
<div class="user" @bind:click="handleSelfClick">
  <img src="{{user.avatar}}" />
  <span>{{user.nickname}}</span>
</div>
```

User 页面中通过[组件别名](https://www.yuque.com/n_tech/winged/logical-node#oix7ft)引用了 UserInfo 组件，并写有 `user={{user}}` 和 `:click="handleUserClick({user, index})"` ，其中：

- `user={{user}}` 是一个 **属性(property)传递**。一个组件可以接收的属性需要被定义在视图实现类的 `propsType` 属性中
- `:click="handleUserClick({user, index})"` 是一个 **事件(event)响应**一个组件可以产生的事件种类需要被定义在视图实现类的 `eventsType` 属性中

**注意：**目前组件属性声明仅支持 TypeScript 类型语法（基础类型，简单的 interface 和 type），如果使用了泛型或者函数类型，编译将会报错

### @vWatch 和@vComputed 装饰器

**注意：**由于 ts 的限制，被@vWatch/@vComputed 监听的属性须是 public 属性

```typescript
// 0.7.0 以前，可以写
class SomeView extends SomeViewV<SomeView> {
  public a: string
  public b: { c: string }

  @vComputed('a', 'b.c') public get someKey() {}
  @vWatch('a', 'b.c') public handleKeyChange() {}
}

// 0.7.0 以后，'b.c' 引用将必须使用 deep 版本，且非 deep 版本需要传入实现类泛型
class SomeView extends SomeViewV<SomeView> {
  public a: string
  public b: { c: string }
  d
  @vComputed<SomeView>('a') public get someKey1() {}
  @vWatch<SomeView>('a') public handleKeyChange1() {}

  @vComputedDeep('b.c') public get someKey2() {}
  @vWatchDeep('b.c') public handleKeyChange2() {}
}
```

#### @vWatch

@vWatch 装饰器可以用于创建一个视图数据（state）监听器，在视图数据变化时此监听器将被调用

```typescript
import { vWatched } from 'winged'
import { TestPageV } from '../v/pages/TestPage.v'

export class TestPage extends TestPageV<TestPage> {
  public someValue = 'a'

  public onLoad(): void {
    setTimeout(() => {
      // 当 this.someValue 改变时，handleSomeValueChange 会被调用
      this.someValue = 'b'
    }, 1000)
  }

  @vWatch('someValue')
  protected handleSomeValueChange() {
    console.log('some value 改变：', this.someValue)
  }
}
```

#### @vComputed

```typescript
import { utils, vComputed } from 'winged'
import { TestPageV } from '../v/pages/TestPage.v'

export class TestPage extends TestPageV<TestPage> {
  public createdAt: number = 0
  public dateFormat = 'Y/M/D'

  // 这个属性会在 createdAt 或 dateFormat 变化时自动重计算
  // 视图中可以直接使用这个属性
  @vComputed('createdAt', 'dateFormat')
  public get displayDate() {
    return utils.parseTime(this.createdAt, this.dateFormat)
  }

  public userList: Array<{ name: string }> = []

  // 这个属性会在 userList 中的成员产生变化时被重新计算
  // 视图中可以直接使用这个属性
  // NOTE: 注意，如果 userList 可能为空，记得做空处理，否则执行会报错
  @vComputed('userList')
  public get userCount() {
    return this.userList.length
  }

  // 这个属性会在 userList 中的成员产生变化时被重新计算
  // 视图中可以直接使用这个属性
  // NOTE: 注意，如果 userList 可能为空，记得做空处理，否则执行会报错
  @vComputed('userList')
  public get isEmpty() {
    return this.userList.length === 0 ? true : false
  }

  public onLoad(): void {
    setTimeout(() => {
      // 通过修改原始属性来触发计算属性更新
      this.createdAt = Date.now()
      this.userList.push({ name: 'Alice' })
    }, 1000)
  }
}
```

### 陷阱和 Tips

- 不要忽略 cli 中的错误提示和警告
- 不要在 view 中使用非纯数据对象
- 如果要使用 for...in 循环，一定要进行`hasOwnProperty`检查（推荐使用`for..of` + `Object.keys`）
- 使用正确的方式来操作数组数据，遵循以下规则

  - 增建数组成员务必使用 Array API
  - 保证每个数组成员时纯数据，且实现同一个 interface
  - 不要直接对着之前没有的值下标赋值
  - 不要在任何时候给数字成员赋值 null
  - 不要修改数组的 length 属性

- 合理的设计组件的 propsType，不支持类型有

  - Class 类型
  - Function 或者函数字面量

- HTML 里不能使用`===`
