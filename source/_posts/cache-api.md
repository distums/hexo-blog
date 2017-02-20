---
title: Cache API
date: 2017-02-20 21:39:11
tags:
  - 翻译
  - js
---

# Cache API

[原文地址](https://davidwalsh.name/cache)

ServiceWorker API意在让开发者能对缓存进行更多控制，包括缓存什么、不缓存什么以及如何缓存。当然我们也可以用ETags或者其同类来完成，但是使用JavaScript在代码级别做控制体验更好，更加可控。然而对于每个API，往缓存添加东西并不是娱乐或者游戏——你不得不亲自做清理工作，而这个“清理工作”我指的就是删除缓存。

让我们看看如果获取缓存，往缓存添加或者删除某个请求的缓存，以及将整个缓存删除。

# 检测`cache` API

可以通过检查`window`里面`caches`对象的存在性来检测浏览器是否支持Cache API：

```js
if('caches' in window) {
  // Has support!
}
```

# 创建一个缓存

创建一个缓存需要调用`cahces.open`函数同时传入缓存名字：

```js
caches.open('test-cache').then(function(cache) {
  // Cache is created and accessible
});
```

`caches.open`调用会返回一个[Promise](https://davidwalsh.name/promises)，而`cache`对象要么是新创建的，要么在`open`调用前就存在了。

# 往缓存添加内容

你可以将缓存看成一个`Request`对象的数组，而每个对象可以作为访问由浏览器保存起来的相应响应内容的键。简单的缓存添加由两个主要的函数完成：`add`和`addAll`。这些函数的参数可以是一个内容应被获取和缓存的URL字符串，或者是一个`Request`对象。你可以通过我的[fetch API](https://davidwalsh.name/fetch)这篇文章学习更多关于`Request`对象的知识。

可以使用`addAll`方法批量往缓存添加URL：

```js
caches.open('test-cache').then(function(cache) { 
  cache.addAll(['/', '/images/logo.png'])
    .then(function() { 
      // Cached!
    });
});
```

`addAll`方法接受一个URL的数组，其中每个URL的内容都应被缓存。`addAll`方法返回一个Promise。如果要添加一个URL，则可以使用`add`方法：

```js
caches.open('test-cache').then(function(cache) {
  cache.add('/page/1');  // "/page/1" URL will be fetched and cached!
});
```

`add`还接受一个自定义的`Request`对象：

```js
caches.open('test-cache').then(function(cache) {
  cache.add(new Request('/page/1', { /* request options */ }));
});
```

类似于`add`方法，还有一个`put`方法可以用于同时添加URL及其`Response`对象。

```js
fetch('/page/1').then(function(response) {
  return caches.open('test-cache').then(function(cache) {
    return cache.put('/page/1', response);
  });
});
```

# 探测缓存内容

要查看已经被缓存的请求，你可以使用每个缓存对象的`keys`方法来获取一个`Request`对象的数组：

```js
caches.open('test-cache').then(function(cache) { 
  cache.keys().then(function(cachedRequests) { 
    console.log(cachedRequests); // [Request, Request]
  });
});

/*
Request {
  bodyUsed: false
  credentials: "omit"
  headers: Headers
  integrity: ""
  method: "GET"
  mode: "no-cors"
  redirect: "follow"
  referrer: ""
  url: "https://fullhost.tld/images/logo.png"
}
*/
```

如果你想查看一个缓存了的`Request`对象的响应内容，你可以使用`cache.match`和`cache.matchAll`来完成：

```js
caches.open('test-cache').then(function(cache) {
  cache.match('/page/1').then(function(matchedResponse) {
    console.log(matchedResponse);
  });
});

/*
Response {
  body: (...),
  bodyUsed: false,
  headers: Headers,
  ok: true,
  status: 200,
  statusText: "OK",
  type: "basic",
  url: "https://davidwalsh.name/page/1"
}
*/
```

同样你可以通过我的[fetch API](https://davidwalsh.name/fetch)这篇文章学习更多关于`Response`对象的知识。

# 移除一个请求的缓存

使用缓存的`delete`方法来移除一个请求的缓存：

```js
caches.open('test-cache').then(function(cache) {
  cache.delete('/page/1');
});
```

缓存里面将不再包含`/page/1`的内容！

# 获取已存在缓存的名字

使用`caches.keys`获取已经存在了的缓存的名字：

```js
caches.keys().then(function(cacheKeys) { 
  console.log(cacheKeys); // ex: ["test-cache"]
});
```

同样的`window.caches.keys`返回一个[Promise](https://davidwalsh.name/tutorials/promises)。

# 删除一个缓存

删除一个已知名字的缓存是很简单的：

```js
caches.delete('test-cache').then(function() { 
  console.log('Cache successfully deleted!'); 
});
```

通常你删除一个缓存然后会用一个新的替代（为了触发一个新的service worker的re-installation）。

```js
// Assuming `CACHE_NAME` is the newest name
// Time to clean up the old!
var CACHE_NAME = 'version-8';

// ...

caches.keys().then(function(cacheNames) {
  return Promise.all(
    cacheNames.map(function(cacheName) {
      if(cacheName != CACHE_NAME) {
        return caches.delete(cacheName);
      }
    })
  );
});
```

当你想成为一名service worker专家时，你会乐意将这些代码段保存到你的工具箱中。Chrome和火狐已经支持service worker了，以此你将看到更多的网站/app（更加可靠）离线也能访问，而且加载速度更快。Enjoy!