---
title: 断点续传
layout: page
date: 2022-07-05 11:18
comments: true
tags: 
	- http
  - react
key: "1"
---

### 前言
> 相信大家都做过上传文件的需求，不可避免遇到上传大文件时，由于要上传大量数据，导致上传的整个过程较慢。由于前后端交互不可能无限制时间，这就导致了大文件上传总是超时而失败。面对这个较为常见的需求务必需要开发者给出对应的解决方案。

### 原理
既然大文件不支持一次性上传，那我们就将大文件按一定规则分割成多个小文件，然后客户端每次只上传一个小文件（切片），后端接收到上传过来的全部文件后根据一定的规则来组合这些小文件。如果在上传过程中出现网络中断等意外情况，下次再次上传时可以从已经上传的部分继续上传，而不是重新上传。

### 详解
断点续传技术利用http1.1协议在Header里添加两个参数来实现，两个参数分别是客户端请求时发送的Range和服务端返回信息时返回的Content-Range，Range用于指定第一个字节和最后一个字节的位置，格式如下：
```javascript
  Range:(unit=first byte pos)-[last byte pos]
```
Range常用的格式有一下情况：
- Range:bytes=0-1024 ，表示传输的是从开头到第1024字节的内容；
- Range:bytes=1025-2048 ，表示传输的是从第1025到2048字节范围的内容；
- Range:bytes=-2000 ，表示传输的是最后2000字节的内容；
- Range:bytes=1024- ，表示传输的是从第1024字节开始到文件结束部分的内容；
- Range:bytes=0-0,-1 表示传输的是第一个和最后一个字节 ；
- Range:bytes=1024-2048,2049-3096,3097-4096 ，表示传输的是多个字节范围。
Content-Range用于响应带有Range的请求。服务器会将Content-Range添加到响应的头部，格式如下：
```javascript
  Content-Range:bytes(unit first byte pos)-[last byte pos]/[entity length]
```
场景格式内容如下：
```javascript
  Content-Range: bytes 2048-4096/10240
```
**注意：这里的2048-4096表示当前发送的数据范围，10240表示文件总大小。**
光有Range和Content-Range还是不够的，还要知道服务端是否支持断点续传，需要从如下两方面判断即可：

- 判断服务端是否是 HTTP/1.1 及以上版本，如果是则支持断点续传，如果不是则不支持。
- 服务端返回响应的头部是否包含 Access-Ranges ，且参数内容是 bytes
符合以上两个条件即可判定位支持断点续传。**注意：**If-Range 必须与 Range 配套使用。缺少其中任意一个另一个都会被忽略。

### 校验
当服务器文件发生改变后，客户端继续向服务端发送断点续传时数据肯定出错。这时候需要Last-Modified来标识服务端文件是否发生变化，客服端使用if-modified-since将前面服务端发送给客服端的Last-Modified发送给服务器，服务器进行最后修改时间验证后，来告知客户端发是否需要重新从服务器获取内容。客户端判断是否需要更新，只需判断服务器返回的状态码即可：
- 206 代表不需要重新获取接着下载就行
- 200代表需要重新获取。 
但是 Last-Modified 和 if-Modified-Since 存在一些问题：

> 某些文件只是修改了时间内容却没变，这时我们并不希望客户端重新缓存这些文件；某些文件修改频繁，有时一秒要修改十几次，但是 if-Modified-Since 是秒级的，无法判断比秒更小的级别； 部分服务器无法获得精确的修改时间。 
要解决上述问题需要用到 Etag ，只需将相关标记（例如文件版本号等）放在引号内即可。当使用校验的时候我们不需要手动实现验证，只需要利用 if-Range 结合 Last-Modified 或者 Etage 来判断是否发生改变，如果没有发生改变服务器将向客户端发送剩余的部分，否则发送全部。