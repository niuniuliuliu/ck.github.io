---
title: PWA(Progressive Web Apps)学习笔记
date: 2021-03-15 16:08:34
categories:
- web前端
tags:
- PWA
---
# PWA(Progressive Web Apps)学习笔记

## 什么是PWA
PWA = 普通的网站 + manifest + Service Workers

manifest文件包含网站相关的信息，包括图标，背景屏幕，颜色和默认方向。

Service Workers为网站提供了更好的体验（渐进增强），允许将网站添加到设备的主屏幕，离线缓存。

<!-- more -->

PWA应该具备的特性：  
* 响应式的 - 它适应较小的屏幕尺寸
* 连接无关 - 由于 Service Worker 缓存，它可以离线工作
* 应用式的交互 - 它使用应用外壳架构进行构建
* 始终保持最新 - 感谢 Service Worker 的更新过程
* 安全的 - 它通过 HTTPS 进行工作
* 可发现的 - 搜索引擎可以找到它
* 可安装的 - 使用清单文件
* 可链接的 - 可以简单的通过 URL 来共享

## 何为Service Workers
Service Workers由JavaScript编写，运行在浏览器后台，基于事件驱动。如果用户浏览器不支持Service Workers的话，并不会造成影响，网站还可以作为普通网站进行浏览，因此做到了“渐进增强”。

通过Service Workers，可以缓存 UI 外壳(用户界面所必需的最小化的 HTML、CSS 和 JavaScript)，动态内容在UI外壳加载后再加载，为用户提供类似原生app的体验。

* Service Workers运行在自己的全局脚本上下文中  
* 不绑定到具体的网页  
* 无法修改网页中的元素，因为它无法访问 DOM  
* 只能使用 HTTPS(localhost本地开发除外)  
* Service Workser运行在不同的线程中，不会被阻塞  



## Service Workers生命周期


![](life-cycle.jpg)

从生命周期图中可以看出，当第一次加载页面时，并不会有激活的 Service Worker 来控制页面。只有当 Service Worker 安装完成并且用户刷新了页面或跳转至网站的其他页面，Service Worker 才会激活并开始拦截请求。  
如果需要在第一次加载时，就希望Service Workers激活并开始拦截请求，可以通过如下方式立即激活Service Workers。
```
self.addEventListener('install', function(event) {
  //使 Service Worker 解雇当前活动的worker， 并且一旦进入等待阶段就会激活自身，触发activate事件
  event.waitUntil(self.skipWaiting());
});
```
结合self.clients.claim() 一起使用，以确保底层 Service Worker 的更新立即生效。
```
self.addEventListener('activate', function(event) {
  e.waitUntil(
        caches.keys().then(function(keyList) {
            return Promise.all(keyList.map(function(key) {
                if (key !== cacheName) {
                    console.log('[ServiceWorker] Removing old cache', key);
                    return caches.delete(key);
                }
            }));
        })
    );
    return self.clients.claim(); //确保底层 Service Worker 的更新立即生效
});
```


## Service Workers缓存
```
var cacheKey = "first-pwa";  //缓存的key，可以添加多个不同的缓存

var cacheList = [   //需要缓存的文件列表
    '/',
    'index.html',
    'icon.png',
    'main.css'
];

//在安装过程中缓存已知的资源
self.addEventListener('install', event => {  //监听install事件
    event.waitUntil(  //install完成后
        caches.open(cacheKey)  //打开cache
            .then(cache => cache.addAll(cacheList))  //将需要缓存的文件加入cache列表
            .then(() => self.skipWaiting())  //使 Service Worker 解雇当前活动的worker，
                                            // 并且一旦进入等待阶段就会激活自身，触发activate事件
                                            //无需等待用户跳转或刷新页面
    );
});


//拦截fetch请求
self.addEventListener('fetch', event => {
    event.respondWith(
        caches.match(event.request).then(response => { //如果请求的资源在缓存中
            if (response != null) return response;  //返回缓存资源

            //通过网络获取资源，并缓存
            var requestToCache = event.request.clone(); //克隆当前请求
            return fetch(requestToCache.url).then(response => {
                if (!response || response.status !== 200) {
                    return response;  //返回错误的响应
                }
                var responseToCache = response.clone(); //克隆响应
                caches.open(cacheKey)
                    .then(cache => {
                        cache.put(requestToCache, responseToCache);  //将响应添加到缓存中
                    });
                return response;  //返回响应
            });
        })
    );
});
```

属于Service Workers作用域范围内的所有http请求都将触发fetch事件，包括html、css、js、图片等。


### 拦截包含save-data的http请求头部的实例
如果用户在浏览器中启用了节省数据的功能，浏览器在每个http请求头部中会加入save-data请求头。
```
this.addEventListener('fetch', function (event) {
 
  if(event.request.headers.get('save-data')){
    // 我们想要节省数据，所以限制了图标和字体
    if (event.request.url.includes('fonts.googleapis.com')) {
        // 不返回任何内容
        event.respondWith(new Promise(resolve => resolve(new Response('', {
            status: 417,
            statusText: 'Ignore fonts to save data.'
            })))
        );
    }
  }
});
```





## 如何保证Service Workers能获取到最新的文件
* 更新存储缓存的名称。
* 缓存破坏，每次发布时更新文件的名称，如增加一个版本号等。

## Web应用清单(mainifest.json)
mainifest.json需要在网页head标签中引用
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Lato">
    <link rel="stylesheet" href="main.css">
    <link rel="manifest" href="manifest.json"/>
    <title>PWA</title>
</head>
<body>
<h1>Hello PWA!</h1>
<script type="text/javascript">
    if (navigator.serviceWorker != null) {
        navigator.serviceWorker.register('sw.js').then(registration => {
            console.log('ServiceWorker registration successful with scope: ', registration.scope);
        }).catch(function (err) {
            console.log('ServiceWorker registration failed: ', err);
        });
    } else {
        //serviceWorker is not supported
    }
</script>
</body>
</html>
```

manifest.json中包含的字段主要包括：
```
{
  "name": "First PWA",
  "short_name": "pwa",
  "display": "standalone",
  "start_url": "/index.html",
  "theme_color": "#FFDF00",
  "background_color": "#FFDF00",
  "orientation": "landscape",
  "scope": "/",
  "icons": [
    {
      "src": "icon.png",
      "sizes": "144x144",
      "type": "image/png"
    }
  ]
}
```
* name：当用户被提示安装应用时出现的文本。
* short_name：当应用安装后出现在用户主屏幕上的文本。
* display：显示模式，默认为browser。包括fullscreen、standalone、minimal-ui 或 browser 。
    * fullscreen：应用占用整个可用的显示区域。
    * standalone：应用以看起来像一个独立的原生应用。此模式下，用户代理将排除诸如 URL 栏等标准浏览器 UI 元素，但可以包括诸如状态栏和系统返回按钮的其他系统 UI 元素。
    * minimal-ui：此模式类似于 fullscreen，但为终端用户提供了可访问的最小 UI 元素集合，例如，后退按钮、前进按钮、重载按钮以及查看网页地址的一些方式。
    * browser：使用操作系统内置的标准浏览器来打开 Web 应用。
* start_url：应用启动时的第一个页面。
* theme_color：可以对浏览器的地址栏进行着色，以符合网站的主色调。
* background_color：启动时的背景色。
* orientation： 屏幕方向。
* icons：当应用被添加到设备主屏幕时所显示的图标。

参考 [https://developer.mozilla.org/en-US/docs/Web/Manifest](https://developer.mozilla.org/en-US/docs/Web/Manifest)
### 监听添加到主屏幕事件
```
    //监听添加到主屏幕事件
    window.addEventListener('beforeinstallprompt', function (event) {
        // //取消添加
        // e.preventDefault();
        // return false;

        event.userChoice.then(function (result) {
            console.log(result.outcome);
            if (result.outcome == 'dismissed') {

            } else {

            }
        });
    });
```

## 推送通知
目前FireFox、Chrome、Edge 已经支持 Push API。推送的过程主要分为三个步骤：
* 客户端订阅
* 发送需要推送的消息到push service
* push service推送到客户端

### 客户端订阅
![](https://developers.google.com/web/fundamentals/push-notifications/images/svgs/browser-to-server.svg)

在订阅前，需要先生成VAPID， VAPID是“自主应用服务器标识” ( Voluntary Application Server Identification ) 的简称。它是一个规范，定义了应用服务器和推送服务之间的握手。

1.客户端订阅消息，此时浏览器会询问用户是否允许消息推送通知。  
2.从浏览器获取PushSubscription对象，其中包含了客户端的信息，可以理解为标示设备的id。  
```
var vapidPublicKey = 'BF0eSi4ANvVKr017Gr_Xzb-bN9l8-c3qRUHqVU6C-vFy_i3xgrKDY-13BPF5BVx93IVObJwnwrt5vjX-ltM6Uuo';

function urlBase64ToUint8Array(base64String) {
        const padding = '='.repeat((4 - base64String.length % 4) % 4);
        const base64 = (base64String + padding)
            .replace(/\-/g, '+')
            .replace(/_/g, '/');
        const rawData = window.atob(base64);
        const outputArray = new Uint8Array(rawData.length);
        for (let i = 0; i < rawData.length; ++i) {
            outputArray[i] = rawData.charCodeAt(i);
        }
        return outputArray;
    }

    function subscribeForPushNotification(registration) {
        return registration.pushManager.getSubscription()
            .then(function (subscription) {
                if (subscription) {
                    return;
                }
                return registration.pushManager.subscribe({
                    userVisibleOnly: true,
                    applicationServerKey: urlBase64ToUint8Array(vapidPublicKey)
                })
                    .then(function (subscription) {
                        var rawKey = subscription.getKey ? subscription.getKey('p256dh') : '';
                        var key = rawKey ? btoa(String.fromCharCode.apply(null, new Uint8Array(rawKey))) : '';
                        var rawAuthSecret = subscription.getKey ? subscription.getKey('auth') : '';
                        var authSecret = rawAuthSecret ?
                            btoa(String.fromCharCode.apply(null, new Uint8Array(rawAuthSecret))) : '';
                        var endpoint = subscription.endpoint;
                        return fetch('http://localhost:3001/api/register', {
                            method: 'post',
                            headers: new Headers({
                                'content-type': 'application/json'
                            }),
                            body: JSON.stringify({
                                endpoint: subscription.endpoint,
                                key: key,
                                authSecret: authSecret,
                            }),
                        });
                    });
            });
    }
```
3.将PushSubscription发送到服务端保存。

服务端示例：
```
this.post('/register', 'register', async (req, res, next) => {
            try {
                let {endpoint, authSecret, key} = req.body;
                let subscriber = {
                    endpoint,
                    keys: {
                        auth: authSecret,
                        p256dh: key
                    }
                };
                subscribers.push(subscriber);
                res.apiSuccess({});
            }
            catch (err) {
                next(err);
            }
        });
```


### 发送消息到push service
通过Web Push协议将需要推送的消息发送到push service。
![](https://developers.google.com/web/fundamentals/push-notifications/images/svgs/server-to-push-service.svg)
使用web-push的发送示例：
```
this.post('/send', 'send', async (req, res, next) => {
            try {
                let message = req.body.message;
                for (let subscriber of subscribers) {
                    webpush.sendNotification(
                        subscriber,
                        JSON.stringify({
                            msg:message,
                            url:'http://localhost:3001',
                            icon:'https://github.com/SangKa/PWA-Book-CN/blob/master/assets/figure1.4.png?raw=true'
                        })
                    );
                }
                res.apiSuccess({});
            }
            catch (err) {
                next(err);
            }
        });
```
### push service推送到客户端
当push service收到消息后，会将消息保存起来，直到目标设备上线后将消息推送到客户端，或者消息超时不再发送。
![](https://developers.google.com/web/fundamentals/push-notifications/images/svgs/push-service-to-sw-event.svg)



## Servicer Worker toolbox
[sw-toolbox](https://github.com/GoogleChromeLabs/sw-toolbox)

## 一些工具

* [webpagetest](https://www.webpagetest.org)，可以使用来自世界各地的真实设备对你的网站进行测试
* [http-server](https://www.npmjs.com/package/http-server) 测试、开发用的http服务
* [内网穿透 ngrok](https://ngrok.com)


## 参考
[https://github.com/SangKa/PWA-Book-CN](https://github.com/SangKa/PWA-Book-CN)

 [https://developers.google.com/web/fundamentals/push-notifications/how-push-works](https://developers.google.com/web/fundamentals/push-notifications/how-push-works)  

 [https://codelabs.developers.google.com/codelabs/your-first-pwapp/#0](https://codelabs.developers.google.com/codelabs/your-first-pwapp/#0)