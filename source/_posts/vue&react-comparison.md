---
layout: post
title: "vue与react项目开发对比"
date: 2021-05-09 10:55
comments: true
tags: 
	 - vue 
	 - react
---

### 前言

从事前端行业以来，我所接触到的项目运用的前端框架有 Vue、React、Angular。没错，前端三大框架我都有过项目实战经验，但 Angular 的项目当时用的是 1.5.x 版本的（很老的版本了），因此对于 Angular 的新特性几乎没接触到，本文也就着重拿 Vue、React 两大热门框架就项目开发使用做一个对比分享。

<!--more-->

### 脚手架

vue-cli 3.\*

```js
npm install -g @vue/cli
vue create my-project  //node版本推荐8.11.0+

// 如果你还是习惯之前2.*版本的话，再安装一个工具即可
npm install -g @vue/cli-init
vue init webpack my-project
```

生成的项目结构大同小异，只不过 3.\*版本将原先直接暴露出来的 webpack 配置封装到了自己的 vue.config.js 配置文件中，包括常见 baseUrl、outputDir、devServer.proxy 等。

Vue-2:
![vue-2图](https://user-gold-cdn.xitu.io/2019/1/3/16813535d494f05c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Vue-3:
![vue-3图](https://user-gold-cdn.xitu.io/2019/1/3/16813535d522bcda?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

###### create-react-app

```js
npm install -g create-react-app
npx create-react-app my-project

```

![npm run eject前图](https://user-gold-cdn.xitu.io/2019/1/3/16813535e086004b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

###### UmiJS

步骤很简单：

- 先找个地方建个空目录，打开命令行工具，执行命令 `mkdir myapp && cd myapp`；
- 执行命令`npm create @umijs/umi-app`创建一个 React 工程；
- 执行命令`npm install`安装依赖；
- 依赖安装成功后，执行命令`npm run start`启动项目，在浏览器上打开 http://localhost:8000 访问项目。

目录结构如下所示：

```js
.
├── package.json
├── .umirc.ts
├── .env
├── dist
├── mock
├── public
└── src
    ├── .umi
    ├── layouts/index.tsx
    ├── pages
        ├── index.less
        └── index.tsx
    └── app.ts

```

### 开发组件 UI

Vue 和 React 中都主张的是组件化，其页面也是一个路由组件。在 Vue 中组件是定义在后缀为`.vue`的文件中，在 React 中组件是定义在后缀为`.js`的文件中，若使用 TypeScript 来开发 React，则其组件是定义在后缀为`.tsx`的文件中

- vue 推荐使用 template 模板

  ```vue
  // 模板渲染，可以直接使用data对象中的数据，利用指令来处理渲染逻辑
  <template>
    <div class="hello">
      <div v-for="(item, index) in list" :key="index">
        {{ item.title }}
      </div>
      // 处理表单，将用户输入的内容赋值到title上
      <input v-model="title" />
      // 事件监听，用户点击后触发methods中的方法
      <button @click="submit">提交</button>
    </div>
  </template>

  <script>
  export default {
    data() {
      return {
        list: [{ title: 'first' }, { title: 'second' }],
        title: '',
      }
    },
    methods: {
      submit() {
        this.list.push({ title: this.title })
        this.title = ''
      },
    },
  }
  </script>
  ```

- react 推荐使用 jsx 或者 js 文件来表示组件，react 支持 class 组件和 function 组件 2 种形式

  **类组件写法**

  ```jsx
  import React, { Component } from 'react'

  export default class HelloWorld extends Component {
    state = {
      list: [{ title: 'first' }, { title: 'second' }],
      title: '',
    }

    setTitle = (e) => {
      // 需要手动调用setState来进行重绘，否则input的value不会改变
      this.setState({ title: e.target.value })
    }

    submit = (e) => {
      const { title, list } = this.state
      list.push({ title })
      this.setState({ list, title: '' })
    }

    render() {
      const { title, list } = this.state
      return (
        <div className="App">
          // react会使用jsx的方式来进行模板的渲染，可混合使用js和html标签 // {}
          解析js，()解析html
          {list.map((item, index) => (
            <div key={index}>{item.title}</div>
          ))}
          // 事件监听 + 表单处理
          <input value={title} onChange={this.setTitle} />
          <button onClick={this.submit}>增加</button>
        </div>
      )
    }
  }
  ```

  **函数组件写法**

  ```react
  import React, { useState } from 'react'

  const HelloWorld = () => {
  	const [list, setList] = useState([{ title: 'first' }, { title: 'second' }]);
  	const [title, setTitle] = useState('');

  	const onSubmit = () => {
  		const _list = [...list];
  		_list.push(title);
  		setList([..._list]);
  	}

  	return (
  		 <div className="App">
          // react会使用jsx的方式来进行模板的渲染，可混合使用js和html标签
          // {}解析js，()解析html
          {
          	list.map((item, index) => (
                <div key={index}>{item.title}</div>
              ))
          }
          // 事件监听 + 表单处理
          <input value={title} onChange={e => setTitle(e.targt.value)} />
          <button onClick={onSubmit}>增加</button>
        </div>
  	)
  }

  export default HelloWorld;
  ```

### 数据管理（state&props）

> **注意**：vue 与 react 中的 props 都是单向数据流的，父级 prop 的更新会向下流动到子组件中，但是反过来则不行。prop 可以是数组或对象，用于接收来自父组件的数据

- vue 中的 props
  props 支持传递静态或动态 props，静态 props 一般传递字符串。

```
<blog-post title="My journey with Vue"></blog-post>
```

复制代码静态 prop 传递布尔值 true 可以这样写，传值 false 仍然需要使用动态 prop 传值。

```
<blog-post disabled></blog-post>
```

复制代码动态赋值使用 v-bind，可以简写为:

```
<blog-post v-bind:title="tile"></blog-post>
// 简写形式
<blog-post :title="tile"></blog-post>
```

复制代码动态 prop 常用来传递对象、数组、布尔值（false 值，true 值可以直接传属性）等。

```
<blog-post :title="post.title + ' by ' + post.author.name"></blog-post>
```

- vue 中的 state
  > **注意**：vue 中使用 data 来管理组件的数据，vue 将会递归将 data 的属性转换为 getter/setter，从而让 data 的属性能够响应数据变化。对象必须是纯粹的对象 (含有零个或多个的 key/value 对)。一旦观察过，不需要再次在数据对象上添加响应式属性。因此推荐在创建实例之前，就声明所有的根级响应式属性。
  > 当一个组件被定义，data 必须声明为返回一个初始数据对象的函数。

```
export default {
  name: 'NewComponent',
  data() {
    return {
      name: 'xxx',
      age: 12
    }
  }
}
```

当需要在组件内部修改数据时，可以直接通过 vue 实例修改：

```
  methods: {
    changeName() {
      this.name = 'new Name';
    }
  }
```

- react 中的 props

  > **注意**：与 vue 一样可以传递动态与静态 props,静态 props 一般传递字符串。函数组件和 class 组件都可以使用 props，函数组件使用 props 参数获取父组件传下来的 props

  函数组件获取 props

  ```
  function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
  }
  const element = <Welcome name="Sara" />;
  ```

  class 组件获取 props

  ```
  class Welcome extends React.Component {
    constructor(props) {
      super(props);
    }
    render() {
      const { name } = this.props;
      return <div>{name}</div>;
    }
  }
  ```

  动态 props

```
<Welcome name={name} />
```

- react 中的 state
  class 组件

  > class 组件在构造函数（constructor）中定义组件内数据（state），修改数据必须通过 setState 修改，不能直接修改 state，这点非常重要

  ```
    class Welcome extends React.Component {
      constructor(props) {
        super(props);
        this.state = {
          name: 'xx'
        };
        this.changeName = this.changeName.bind(this);
      }

      changeName() {
        this.setState({
          name: 'new name'
        });
      }

      render() {
        const { name } = this.state;
        return <div onClick={this.changeName}>{name}</div>;
      }
    }
  ```

  function 组件

  > react 16.0 之前函数组件只是纯的渲染组件，hooks 的出现赋予了函数组件管理 state 的能力。
  > useState 返回一个 state，以及更新 state 的函数。如果新的 state 需要通过使用先前的 state 计算得出，那么可以将函数传递给 setState。该函数将接收先前的 state，并返回一个更新后的值。

  ```
    import React, { useState } from 'react';

    function Counter({initialCount}) {
      const [count, setCount] = useState(initialCount);
      return (
        <>
          Count: {count}
          <button onClick={() => setCount(initialCount)}>Reset</button>
          <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
          <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
        </>
      );
    }

  ```

  > 关于 setState 有以下三点说明：
  >
  > - 与 class 组件中的 setState 方法不同，useState 不会自动合并更新对象。
  > - 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
  > - 只能在 React 的函数组件或自定义 hook 中调用 Hook。不要在其他 JavaScript 函数中调用。

### 组件数据交互

#### 父子组件数据交互

> 对于父子组件数据交互，vue 中使用 prop+自定义事件实现，react 通过 props+回调实现。

- vue

  > 父组件通过 props 传递数据给子组件，子组件使用$emit 触发自定义事件，父组件中监听子组件的自定义事件获取子组件传递来的数据。

  子组件使用$emit 传递自定义事件 myEvent:

  ```
    <template>
      <div @click="changeName">{{name}}</div>
    </template>

    <script>
    export default {
      name: 'NewComponent',
      data() {
        return {
          name: 'xxx',
          age: 12
        }
      },
      methods: {
        changeName() {
          this.name = 'new Name';
          this.$emit('myEvent', this.name);
        }
      }
    }
    </script>
  ```

  父组件使用@myEvent 监听自定义事件，回调函数参数是子组件传回的数据：

  ```
    <template>
      <div>
        <new-component @myEvent="getName"></new-component>
      </div>
    </template>

    <script>
    import NewComponent from './NewComponent';

    export default {
      components: {
        NewComponent
      },
      data() {
        return {}
      },
      methods: {
        getName(name) {
          console.log(name)
        }
      }
    }
    </script>
  ```

- react

  > 父组件使用 props 传递数据和回调函数给子组件，子组件通过 props 传下来的回调函数返回数据，父组件通过回调函数获取子组件传递上来的数据。

  子组件通过 props 接收父组件传下来的回调事件：

  ```
    import React, { useState } from 'react';

    function Children(props) {
      const { myEvent } = props;
      const [name, setName] = useState('xxx');

      const changeName = () => {
        setName('new name');
        myEvent('new name');
      };
      return <div onClick={changeName}>{name}</div>;
    }
  ```

  父组件通过回调事件获取子组件传递的参数：

  ```
    function Parent() {
      const changeName = name => {
        console.log(name);
      };
      return <Children myEvent={changeName}></Children>;
    }
  ```

#### 跨组件数据交互

> vue 和 react 都支持跨组件传递数据，vue 中主要通过 provide / inject 实现，react 中主要通过 Context 实现。

- vue

  祖先组件中定义 provide 选项，provide 选项应该是一个对象或返回一个对象的函数。

  ```
    <template>
      <div>
        <new-component @myEvent="getName"></new-component>
      </div>
    </template>

    <script>
    import NewComponent from './NewComponent';

    export default {
      provide: { // 定义provide选项
        message: 'This is a big news'
      },
      components: {
        NewComponent
      },
      data() {
        return {}
      },
      methods: {
        getName(name) {
          console.log(name)
        }
      }
    }
    </script>
  ```

  子组件通过 inject 选项获取祖先组件的 provide 选项值，inject 选项应该是一个字符串数组或者对象。

  ```
    <template>
      <div>{{message}}</div>
    </template>

    <script>
    export default {
      name: 'Children',
      inject: ['message'],
      data() {
        return {}
      }
    }
    </script>
  ```

- react

  > Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。

  在父组件创建一个 Context 对象，通过 Context.provider 的 value 属性向消费组件传值。

  ```
    import React, { useState } from 'react';

    // 创建Context对象
    const MyContext = React.createContext({ theme: 'black' });

    function Parent() {
      const changeName = name => {
        console.log(name);
      };
      // Context.provider向消费组件传值
      return (<MyContext.Provider value={{ theme: 'white' }}>
        <Children myEvent={changeName}></Children>;
      </MyContext.Provider>);

    }
  ```

  消费组件获取 Context 有 2 种方式：

  1. class 组件通过 contextType 获取最近 Context 上的那个值。

  ```
    class DeepChildren1 extends React.Component {
      constructor(props) {
        super(props);
      }

      static contextType = MyContext;

      render() {
        return <div>{this.context.theme}123</div>;
      }
    }
  ```

  2. 函数式组件通过 Context.Consumer 订阅到 Context 的变更。

  ```
    function DeepChildren(props) {
      return (<MyContext.Consumer>
        {
          (value) => (
            <div>{value.theme}</div>
          )
        }
      </MyContext.Consumer>);
    }
  ```

  > **注意**：当 Provider 的父组件进行重渲染时，consumers 组件会重新渲染，并且没有办法避免，应该尽量避免使用 Context。
