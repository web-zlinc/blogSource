---
title: 搭建项目
layout: page
date: 2022-05-11 15:21
comments: true
tags: 
	- umi
  - react
key: "1"
---

### 前言

> 作为一个前端开发者，相信随着自己经验的不断积累，对于项目搭建或多或少都有所了解。笔者梳理出的大致工作包含以下步骤：
>
> - 了解项目背景及业务
> - 根据项目需求做技术选型
> - 项目大致框架搭建
> - 编码规范约束
> - 基本的通用工具类、组件封装、通用样式梳理
> - 后期的代码维护与优化工作

<!--more-->

### 项目背景及业务

可能有人会说前端不像后端那样必须要对项目的业务流程有很深层次的了解，因为前端大部分工作都是建立页面展示以及与后端交互上，而后端交互只需要在特定功能模块开发时和对应后端人员沟通即可。然而这可能更多是建立在具体的某个功能点开发时的一个工作模式，但是如果需要从零到一的去搭建前端项目话，肯定离不开对项目背景及业务有一定了解，因为只有建立在这个基础之下，才能做到以下工作：

- 确定项目适合用什么技术栈（vue/react）开发
- 主要的技术栈确定后，在能够根据项目各个功能点选择适合的辅助工具
- 粗略确定项目通用的一些配置、工具类、组件、样式
- 后续的功能模块开发以及与后端沟通的效率提升（提升工作效率）

### 技术选型

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47fb2992b134471f809996645117591f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

技术选型主要体现在前端框架的选择上，需要考虑的因素有：

- 框架是否能满足大部分应用需求？权衡多个框架对比
- 框架是否有丰富的组件库，以及社区的支持情况如何？
- 对团队成员学习成本如何？
- 后续的维护成本与难度如何？
- 是否能够以最小代价迁移现有引用？
- 浏览器的支持情况，自身的扩展性如何？

目前前端领域主流的框架是 vue、react、angular 这三个，每个框架都有自身的优劣，笔者简单梳理一下各自的使用场景。

#### Vue

- Vue 借鉴了 Angular 和 React 的一些思想，在其基础上开发了一套更易上手的框架。它既不像 Angular 需要理解大量的基础知识，也不像 React 在使用 VirtualDOM 的同时需要学习 JSX 及其相关的语法。
- Vue 的开发者尤雨溪是中国人，框架本身提供了大量丰富的中文文档，这也为 Vue 的发展和使用带来巨大的优势。Vue 框架适合于需要快速上手、上线的应用，还适用于迁移传统的多单面应用。Vue 框架，它可以满足快速上线的需求，同时在后期也可以演进成单页面应用。

#### React

- React 是第一个采用 VirtualDOM 的、流行的前端框架。传统的 DOM 操作是直接在 DOM 上操作的，当需要修改一系列元素中的值时，就会直接对 DOM 进行操作。而采用 VirtualDOM 则会对需要修改的 DOM 进行比较（DIFF），从而只选择需要修改的部分。

- React 只是一个 View 层，它是为了优化 DOM 的操作而诞生的。为了完成一个完整的应用，我们还需要路由库、执行单向流库、webAPI 调用库、测试库、依赖管理库等，这 简直是一场噩梦。因此为了搭建出一个完整的 React 工程，我们还需要做大量的额外工作。

#### Angular

- Angular 是一个大而全的框架，它提供了开发一个完整应用所需的所有要素，其总体的架构思想依赖注入、强类型等，更适合那些有后端经验的开发者，而对于一个没有经验的新手，直接使用 Angular 框架，则需要较长的学习时间。

笔者最近在公司里搭建的项目选择的技术栈是 React，原因主要有：

- 公司项目业务方向主要是做水利方面的，并且以往的项目使用也都是 react 技术栈
- 目前新项目前端救我一个人，笔者使用 react 开发项目经验相对丰富些
- React 与 TS 有着更全面的支持

### 框架搭建

在做好技术选型后，项目主要的开发技术栈就确定了，整个项目的框架搭建根据主要的技术栈选择社区现有比较成熟的脚手架皆可。

笔者使用的脚手架是蚂蚁集团的 Umi，借用官网一张技术收敛图：

![img](https://img.alicdn.com/tfs/TB1hE8ywrr1gK0jSZFDXXb9yVXa-1227-620.png)

主要集成了路由、状态管理、webpack。更好的搭配 AntdPro、Antd、TS，上手成本低、可扩展、完备路由。

如果技术栈是 vue 的话，脚手架常见的有：vue-cli、vite 可供选择，当然也可以完全不依赖脚手架使用 webpack 自定义可扩展项目构建工具。

### 编码规范约束

在公司里基本都是团队开发，便会涉及多人开发同一个项目，而作为前期项目框架搭建者肯定需要考虑到不同人的不同编码规范的统一。不然的话每个人都按自己的风格编写代码，后续对于项目维护者而言就是一件头疼的事了，为了达到事半功倍的效果，务必做好前期的编码规范约束。

笔者之前的团队规模相对比较大，在这一块的做法是召集大家开一个会议，在会议上商讨并最终确定一套规范。主要确定的有：

- 组件、方法、变量命名规则
- 样式类名风格统一
- 必要的代码注释

当然除了人为的约定外，还有代码编辑器的编码规范，主要包含：

- 统一使用一个编辑器，并设置通用的代码风格配置文件
- 项目里使用统一 ESlint、Prettierr、stylelint 工具配置

### 通用工具类、组件、样式封装

在前期熟悉了项目业务后，能够大体总结出使用场景比较多的通用组件、再结合过往的项目经验整理前期通用的项目配置。笔者在这个环节做的工作有：

- 项目大体的 layout 组件封装
- 结和业务抽取出通用的功能组件，如：个性化表格及弹窗、权限路由拦截、常用图表、图谱、页头等组件
- 全局格式化样式，项目常用主色调变量定义、button 尺寸及圆角大小
- 接口请求响应拦截处理
- 通用工具类方法

当然这只是笔者自己根据所负责的项目梳理的在该环节下所作的工作，这边做的项目业务方向主要是水利方向，会涉及到的点有：地图\图谱（区域内水库，闸门，阀门分布情况、以及拓扑关系展示）、图表（展示各时间段水流失、排放情况）、列表数据展示。更具项目类型的不同可能所做的工作会有差异，但还是有相似性的工作的。

### 后期的代码维护与优化工作

- 代码维护

  定期组织团队代码评审会议，主要针对以往开发的功能模块代码的强壮性、可行性做评审，集体讨论对应代码优化方案并做好记录，会议后有对应开发人员落实改进方案。

- 优化工作主要分为：
  1. 项目构建提速优化
     - 缩小 loader 配置检查文件路径
     - 开启插件配置缓存功能为二次构建提速
  2. 项目打包体积优化
     - 对项目代码 js、css 代码压缩
     - 开启 treeshaking 功能
  3. 页面访问提速
     - 路由懒加载
     - 异步加载模块
     - 预加载

### 总结

以上主要是针对个人就最近从零搭建项目整理的心得，输出该文主要是想提炼出自己在这个过程中学到的东西，欢迎读者批评指正，只要是对自己有提升的欣然接受并改正。