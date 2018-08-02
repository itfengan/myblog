---
title: ReactNative碎片整理之网络请求
date: 2018-07-30 18:49:21
tags:
- RN
categories: RN
password:
---

React Native 中虽然也内置了XMLHttpRequest 网络请求Api（也就是ajax），但XMLHttpRequest 是一个设计粗糙的API，不符合职责分离的原则，配置和调用方式比较混乱，而且基于事件的异步模型不如现代的Promise友好。所以，React Native官方推荐使用Fetch Api

<!--more-->

### Fetch基本使用

#### 最简单的使用

> 如果只是请求一下服务器，不需要任何操作

```javascript
fetch('http://chenxiaoping.com/demo')添加请求头
```

>  添加请求头和请求参数（服务端支持json参数）

```javascript
fetch('https://mywebsite.com/endpoint/', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    firstParam: 'yourValue',
    secondParam: 'yourOtherValue',
  })
})
```

>  添加请求头和请求参数（服务端不支持json参数）

```javascript
fetch('https://mywebsite.com/endpoint/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
  },
  body: 'key1=value1&key2=value2'
})
```

### Response对象参数解析

**Response对象中包含了多种属性：**

- status (number) ： HTTP请求的响应状态行。
- statusText (String) ： 服务器返回的状态报告。
- ok (boolean) ：如果返回200表示请求成功，则为true。
- headers (Headers) ： 返回头部信息。
- url (String) ：请求的地址。

**Response对象还提供了多种方法：**

- formData()：返回一个带有FormData的Promise。
- json() ：返回一个带有JSON对象的Promise。
- text()：返回一个带有文本的Promise。
- clone() ：复制一份response。
- error()：返回一个与网络相关的错误。
- redirect()：返回了一个可以重定向至某URL的response。
- arrayBuffer()：返回一个带有ArrayBuffer的Promise。
- blob() ： 返回一个带有Blob的Promise。

> 解析json和修改json数据

```javascript
loadData() {
        return fetch('https://facebook.github.io/react-native/movies.json',)
            .then((response) => response.json())
            .then((json) => {
                json['description']='hello'
                Alert.alert(json.description)
            })
            .catch((error) => {
                this.setState(() => {
                    return {
                        buttonText: 'doLoad',
                    }
                });
                console.error(error);
            })
    }
```

response：

```json
{
  "title": "The Basics - Networking",
  "description": "Your app fetched this from a remote endpoint!",
  "movies": [
    { "id": "1", "title": "Star Wars", "releaseYear": "1977" },
    { "id": "2", "title": "Back to the Future", "releaseYear": "1985" },
    { "id": "3", "title": "The Matrix", "releaseYear": "1999" },
    { "id": "4", "title": "Inception", "releaseYear": "2010" },
    { "id": "5", "title": "Interstellar", "releaseYear": "2014" }
  ]
}
```

可以看到弹窗为‘hello’

### 超时

```javascript
fetch(input, init).then(function(response) { ... });
```

### 参数

- input

定义要获取的资源。这可能是： 一个 USVString 字符串，包含要获取资源的 URL。 一个 Request 对象。

- init 可选 一个配置项对象，包括所有对请求的设置。可选的参数有：

1. method: 请求使用的方法，如 GET、POST。
2. headers: 请求的头信息，形式为 Headers 对象或 ByteString。 body: 请求的 body 信息：可能是一个 Blob、BufferSource、FormData、URLSearchParams 或者 USVString 对象。注意 GET 或 HEAD 方法的请求不能包含 body 信息。
3. mode: 请求的模式，如 cors、 no-cors 或者 same-origin。
4. credentials: 请求的 credentials，如 omit、same-origin 或者 include。
5. cache: 请求的 cache 模式: default, no-store, reload, no-cache, force-cache, or only-if-cached.

根本找不到 timeout 配置

```javascript
		// 超时版的fetch
        _fetch(fetch, timeout) {
            return Promise.race([
                fetch,
                new Promise(function (resolve, reject) {
                    setTimeout(() => reject(new Error('request timeout')), timeout);
                })
            ]);
        }
        // 使用
        _fetch(fetch('url'), 1000).then((info)=> {
            return info.text();
        }).then((info)=> {
            console.log(info);
        }).catch((err)=> {
            throw new Error(err);    
        });
```

**代码中用Promise.race()将fetch和一个新的Promise包装在了一起,新的Promise和fetch谁率先返回就把该Promise实例返回值传递给下面的.then()或者是.catch()**

[让fetch也可以timeout](http://www.open-open.com/lib/view/open1472717659359.html)

[[可以设置超时版的的fetch](https://www.cnblogs.com/sorrowx/p/7096316.html)](https://www.cnblogs.com/sorrowx/p/7096316.html)

[fetch添加超时时间只需几行代码--fetch timeout](https://blog.csdn.net/qq_33323251/article/details/79832689)

### 简单封装

```javascript
export default class FetchUtil {
    init() {
        this.url = '';
        this.method = 'GET';
        this.headers = {};
        this.body_type = 'form';
        this.bodys = {};
        this.credentials = 'omit';
      //默认返回json对象
        this.return_type = 'json';
        this.overtime = 0;
        this.firstThen = undefined;

        return this;
    }
    setUrl(url) {
        this.url = url;
        return this;
    }
    setMethod(val) {
        this.method = val;
        return this;
    }
    setBodyType(val) {
        this.body_type = val;
        return this;
    }

    setReturnType(val) {
        this.return_type = val;
        return this;
    }

    setOvertime(val) {
        this.overtime = val;
        return this;
    }

    setHeader(name, val = null) {
        if (typeof name == 'string') {
            this.headers[name] = val;
        } else if (typeof name == 'object') {
            Object.keys(name).map((index) => {
                this.headers[index] = name[index];
            });
        }

        return this;
    }

    setBody(name, val = null) {
        if (typeof name == 'string') {
            this.bodys[name] = val;
        } else if (typeof name == 'object') {
            Object.keys(name).map((index) => {
                this.bodys[index] = name[index];
            });
        }
        return this;
    }

    setCookieOrigin() {
        this.credentials = 'same-origin';
        return this;
    }

    setCookieCors() {
        this.credentials = 'include';
        return this;
    }

    thenStart(then) {
        this.firstThen = then;
        return this;
    }

    dofetch() {
        let options = {};
        options.method = this.method;
        options.credentials = this.credentials;

        options.headers = this.headers;

        if ({} != this.bodys && this.method != 'GET') {
            if ('form' == this.body_type) {
                this.setHeader("Content-Type", "application/x-www-form-urlencoded;charset=UTF-8");
                let data = '';
                Object.keys(this.bodys).map((index) => {
                    let param = encodeURI(this.bodys[index]);
                    data += `${index}=${param}&`;
                });
                options.body = data;
            } else if ('file' == this.body_type) {
                let data = new FormData();
                Object.keys(this.bodys).map((index) => {
                    data.append(index, this.bodys[index]);
                });
                options.body = data;
            } else if ('json' == this.body_type) {
                options.body = JSON.stringify(this.bodys);
            }
        }

        return Promise.race([
            fetch(this.url, options),
            new Promise((resolve, reject) => {
                setTimeout(() => reject(new Error('request timeout')), this.overtime ? this.overtime : 30 * 1000);
            })
        ]).then(
            (response) => {
                if (this.firstThen) {
                    let tempResponse = this.firstThen(response);
                    if (tempResponse) {
                        return tempResponse;
                    }
                }
                return response;
            }
        ).then(
            (response) => {
                if ('json' == this.return_type) {
                    return response.json();
                } else if ('text' == this.return_type) {
                    return response.text();
                } else if ('blob' == this.return_type) {
                    return response.blob();
                } else if ('formData' == this.return_type) {
                    return response.formData();
                } else if ('arrayBuffer' == this.return_type) {
                    return response.arrayBuffer();
                }
            }
        );
    }
}

//use
let fetchUtil = new FetchUtil();
        fetchUtil.init()
            .setUrl('https://www.google.com/')
            .setMethod('GET')
            .setOvertime(15 * 1000)
            .setHeader({
                'Accept': 'application/json',
                'Content-Type': 'application/json'
            })
            .dofetch()
            .then((response) => {
          		//这里的response已经被转成object了
                console.warn( typeof response.title);
            })
            .then((data) => {

                Alert.alert(data);
            })
            .catch((error) => {
                console.warn( '=> catch: ', error);
                // console.log('=> catch: ', error);
            });
```

