---
title: Chrome扩展程序的开发及服务器部署实时更新
layout: page
date: 2024-03-22 19:05
comments: true
tags: 
  - html
  - css
  - javascript
  - 插件部署
key: "1"
---

### 扩展程序是什么
扩展程序主要是能够修改或增强 Chrome 浏览器功能的小程序。对前端开发人员非常友好，因为扩展程序是使用 Web 开发技术创建的：HTML、CSS 和 JavaScript。

<!--more-->

### 开发入门
chrome扩展程序主要由一下几部分组成
- **manifest.json：** 相当于插件的 meta 信息，包含插件的名称、版本号、图标、脚本文件名称等，这个文件是每个插件都必须提供的，其他几部分都是可选的。
- **background script：** 可以调用全部的 chrome 插件 API，实现跨域请求、网页截屏、弹出 chrome 通知消息等功能。相当于在一个隐藏的浏览器页面内默默运行。
- **页面功能：** 包括点击插件图标弹出的页面（简称 popup）、插件的配置页面（简称 options）。
- **content script：** 早期也被称为 injected script，是插件注入到页面的脚本，但是不会体现在页面 DOM 结构里。content script 可以操作 DOM，但是它和页面其他的脚本是隔离的，访问不到其他脚本定义的变量、函数等，相当于运行在单独的沙盒里。content script 可以调用有限的 chrome 插件 API，网络请求收到同源策略限制。

#### manifest.json字段说明
```json
  {
  // Required - 通俗易懂
  "manifest_version": 3,
  "name": "My Extension",
  "version": "versionString",

   // 『重点』action配置项主要用于点击图标弹出框，对于弹出框接受的是html文件
  "action": {
     "default_title": "Click to view a popup",
   	 "default_popup": "popup.html"
   }
    
  // 通俗易懂
  "default_locale": "en",
  "description": "A plain text description",
  "icons": {...},
  "author": ...,

  // 『重点』下面将出现的background.js 配置service work
  "background": {
    // Required
    "service_worker": "service-worker.js",
  },

    // 『重点』下面将出现content_script.js 应用于所有页面上下文的js
  "content_scripts": [
     {
       "matches": ["https://*.nytimes.com/*"],
       "css": ["my-styles.css"],
       "js": ["content-script.js"]
     }
   ],

    // 使用/添加devtools中的功能
  "devtools_page": "devtools.html",

    /**
    * 三个permission
    * host_permissions - 允许使用扩展的域名
    * permissions - 包含已知字符串列表中的项目 【只需一次弹框要求允许】
    * optional_permissions - 与常规类似permissions，但由扩展的用户在运行时授予，而不是提前授予【安全】
    * 列出常见选项
    * {
    *		activeTab: 当扩展卡选项被改变需要重新获取新的权限
    *		tabs: 操作选项卡api（改变位置等）
    *		downloads: 访问chrome.downloads API 的权限 便于下载但还是会受到跨域影响
    *		history: history api权限
    *		storage: 访问localstorage/sessionStorage权限
    * }
    */
  "host_permissions": ["http://*/*", "https://*/*"],
  "permissions": ["tabs"],
  "optional_permissions": ["downloads"],

    // 内部弹出可选页面 - 见fehelper操作页
  "options_page": "options.html",
  "options_ui": {
    "chrome_style": true,
    "page": "options.html"
  },
}
```
#### popup -- 弹窗页面
popup 页面也非常好理解，在 `manifest.json` 的定义里它是 `browser_action`， 就是我们扩展程序的界面（弹窗页）

#### content script
> content script 脚本是指能够在浏览器已经加载的页面内部运行的 javascript 脚本。可以理解为在页面加载玩完毕之后扩展程序会为页面注入一个或多个脚本，这些脚本可以获得浏览器所访问的 web 页面的详细信息。也就是我们可以利用这个脚本收集页面上各种我们需要的信息。

注入方式有两种：
- manifest.json文件声明式注入
  ```json
  {
      "content_scripts": [
        {
          "matches": ["http://*/*", "https://*/*"],
          "run_at": "document_idle",
          "js": ["content.js"]
        }
      ]
  }
  ```
  各个字段都比较直观。其中：
  1.  matches 表示页面 url 匹配时才加载
  2.  `run_at` 表示在什么时机加载，一般是 `document_idle`，避免 content_scripts 影响页面加载性能。

    需要注意的是，如果用户已经打开了 N 个页面，然后再安装插件，这 N 个页面除非重新刷新，否则是不会加载 manifest 声明的 content_scripts。安装插件之后新打开的页面是可以加载 content_scripts 的。
- 用程序动态注入脚本，代码如下：

    ```js
    chrome.tabs.executeScript({
      file: 'content.js'
    });

    ```
#### background -- 后台网页
chrome扩展程序将后台网页分为两种类型：
-   持续运行的后台网页
-   事件页面

**是否持久存在是事件页面与后台网页之间的根本区别**。
> 应用和扩展程序通常需要长时间运行的脚本来管理某些任务或状态，这就是后台页面的作用。事件页面只在需要时加载，当事件页面不活动时就会卸载，以便释放内存和其他系统资源，所以一般而言是推荐使用事件页面。

它存在的目的在于，在扩展的整个生命周期内需要长时间管理一些任务或状态。它的主要功能及适用场景，大致如下：
-   事件页面监听的某个事件触发
-   应用或扩展程序第一次安装或者更新到新版本（为了注册事件）
-   内容脚本或其他扩展程序发送消息
-   扩展程序中的其他视图调用了 runtime.getBackgroundPage

#### Content Script向background通信
- 注入页面脚本content.js
  ```js
  // 发送消息
    chrome.runtime.sendMessage(
    {
        msg: '从 Content Script 向 事件页面 传递消息',
        result: 1
    },
    function(response) {
        if (response && response.msg) {
            console.log(response.msg);
        }
    }
    );
  ```
- background.js 后台页面
  ```js
     // 接收消息
    chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
      if (request.result) {
          sendResponse({
              farewell: "ok"
          });
      }
    });
  ```
在发送端，可以使用 `runtime.sendMessage` 或 `tabs.sendMessage` 方法。这些方法分别允许您从内容脚本向扩展程序或者反过来发送可通过 JSON 序列化的消息，可选的 callback 参数允许您在需要的时候从另一边处理回应。

而在接收端，我们需要设置一个 `runtime.onMessage` 事件监听器来处理消息。
#### popup 弹窗向ContentScript通信
- popup弹窗页面
  ```js
  let obj = {
      msg: '从 popup 弹窗页面  向 Content Script 传递消息',
      result: 0
    };

    // 发送消息
    chrome.tabs.query({ active: true, currentWindow: true }, function(tabs) {
      chrome.tabs.sendMessage(tabs[0].id, obj, function(response) {
          console.log("Send Success");
      });
    });
  ```
- Content Script
  ```js
    // 接收消息
    chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
      console.log(sender.tab ? "来自内容脚本：" + sender.tab.url : "来自扩展程序");

      if (request && !request.result) {
          console.log(result.msg);
      }
    });
  ```
需要注意，从 popup 弹窗页面 向 Content Script 传递消息时，由于浏览器可能同时打开多个 tab 页，所以需要指定一下传递的页面，指定发送至哪一个标签页。

使用 `chrome.tabs.query({ active: true, currentWindow: true }, function(tabs) {})` 则能正确选中当前打开的标签页。
### 服务器部署
1. 修改扩展程序源码根目录下manifest.json配置文件version字段值, 同时添加update_url字段，该字段值为服务器提供的获取更新插件的update.xml文件的接口
```json
{
  "name": "Getting Started Example",
  "description": "Build an Extension!",
  "version": "1.0",
  "manifest_version": 3,
   "update_url": "http://xxx.xx.xxx:xxxx/update.xml"
}
```
2. 将开发好的扩展程序打包成crx文件，打开chrome浏览器开发者工具选择扩展程序并开启开发者模式

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6be124e9a0a546e287616ccc06cbb0ed~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=961&h=678&s=87759&e=png&b=fdfdfd)
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1ff4e2b93b243eeb1ca0f53297b1f3e~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=730&h=606&s=59468&e=png&b=666666)
3. 将打包好的crx文件拷贝到服务器对应目录下，并在同级目录创建update.xml文件

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46c1b161e10f4ac292b75d54a40d922f~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1587&h=833&s=509100&e=png&b=fcfbfb)
update.xml文件内容如下：
```xml
<?xml version='1.0' encoding='UTF-8'?>

<gupdate xmlns='http://www.google.com/update2/response' protocol='2.0'>

  <app appid='xxxxxxxxxxxxxxxxxxxxxxxxxxx'>

    <updatecheck codebase='http://xxx.xx.xx:xxx/home/updata/busnissHelper.crx' version='1.1.5' />

  </app>

</gupdate>
```
>**xml文件需要注意**
> - appid：扩展程序源码打包成crx文件后拖拽到扩展程序里显示多的id
> - codebase：对应后端提供的获取crx文件的接口路径
> - version：与扩展程序源码根目录下manifest.json配置文件下的version字段值一致

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abace0043e3c4e9d9eace68b153baa3f~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1105&h=687&s=51879&e=png&b=fefefe)
4. 服务器相关文件更新替换好后，利用docker可视化管理工具重新部署

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb2b315b65d1480fb13ce0fd79fcfc68~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1414&h=634&s=99359&e=png&b=fafafa)
5. 查看扩展程序更新后的效果

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4af71381c9045e79494b5e67026c035~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1076&h=900&s=94273&e=png&b=fefefe)
### 总结
以上主要是对Manifest V3进行的讲解的。V2与V3相关配置有所更新，可参考[迁移清单](https://doc.yilijishu.info/chrome/mv3-migration-checklist.html)。
另外我在开发过程中遇到过一下问题：
- 扩展程序如要使用第三方库（如：JQuery）需要将压缩文件下载到本地源码目录不能通过cdn的形式引入使用
- 使用V3版本设计到异步请求的可以直接使用内置fetch与服务端通信

总体来讲，chrome扩展程序开发对前端开发是比较友好且易上手的，需要着重关注的有contentScript、backgroundScript、Manifest.json这几个模块，涉及到chrome提供的api可参考[Google Chrome Extension官方文档](https://developer.chrome.com/docs/extensions/reference/scripting/#type-CSSInjection "Google Chrome Extension官方文档")。