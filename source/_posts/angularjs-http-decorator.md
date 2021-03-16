---
title: 装饰模式在AngularJS升级中的应用
categories:
  - web前端
date: 2021-03-16 13:49:12
tags:
  - AngularJS
---

# 装饰模式在AngularJS升级中的应用

最近接到一个任务，需要将原先项目中的AngularJS版本从1.3升级到1.6，升级过程基本挺顺利的，项目整体的修改并不多，只要参考官方给出的升级文档就可以了:
[https://code.angularjs.org/1.6.9/docs/guide/migration#migrating-from-1-3-to-1-4](https://code.angularjs.org/1.6.9/docs/guide/migration#migrating-from-1-3-to-1-4)

<!-- more -->

升级过程中发现$http服务不再支持success()和error()这两个callback，而是推荐使用promise的then(),catch()方法。 本来这个改动其实也只需要把原来的sucess和error的callback改成promise的方式就可以了，但是我搜了一下整个项目，发现有近百个文件需要调整，这。。。 改动太多了，万一没改好这锅就大了。

于是想了一下能不能有什么改动比较小的方法，正好最近刚看完设计模式，装饰模式不就可以完美解决这个问题吗？ 而且AngularJS也提供了decorator方法来装饰服务，果断搞起。

```
$provide.decorator('$http', function ($delegate) {
    var httpMethods = ['get', 'head', 'post', 'put', 'delete', 'patch'];
    var newHttp = null;
    var adapterFunction = function (originalMethod) {
        var args = [].slice.call(arguments, 1);
        var promise = originalMethod.apply(null, args);
        promise.success = function (fn) {
            promise._onSuccess = fn;
            return promise;
        };
        promise.error = function (fn) {
            promise._onError = fn;
            return promise;
        };

        promise.then(function (response) {
            var data = response.data;
            if (promise._onSuccess) {
                promise._onSuccess(data);
            }
        }, function (err) {
            if (promise._onError) {
                promise._onError(err);
            }
        });
        return promise;
    };
    newHttp = adapterFunction.bind(this, $delegate);
    httpMethods.forEach(function (httpMethod) {
        var originalMethod = $delegate[httpMethod];
        newHttp[httpMethod] = adapterFunction.bind(this, originalMethod);
    });
    return newHttp;
});
```
这里对常用的http方法进行了装饰，使其能够继续支持success和error的回调方法，在调用suceess和error时，将回调函数保存在promise的_onSuccess和_onError中，然后调用promise的then方法后，判断是否存在_onSuccess和_onError，如果存在就把response的data返回(保持和原来的success和error的返回一致)。