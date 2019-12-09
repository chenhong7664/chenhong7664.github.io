---
layout: post
title: "Express"
date: 2019-12-9 
description: "node.js-express"

tag: node.js
---   
**以下内容，基于 Express 4.x 版本**

# 修改代码自动重启

>有一个第三方命名工具：ｎｏｄｅｍｏｎ来解决频繁修改代码重启服务器的问题
>
>＊　nodemon是基于Node.js开发的第三方命令工具，使用时需独立安装

```shell
#在任意命令行执行都可以
#也就是说需要－－global来安转装的都可以在任意目录执行
npm install --global nodemon

```

​		安装完毕之后，使用：

```shell
npm app.js

#使用　nodemon
nodemon app.js
```



# Node.js 的 Express

*Express* 估计是那种你第一次接触，就会喜欢上用它的框架。因为它真的非常简单，直接。

在当前版本上，一共才这么几个文件：

```kotlin
lib/
├── application.js
├── express.js
├── middleware
│   ├── init.js
│   └── query.js
├── request.js
├── response.js
├── router
│   ├── index.js
│   ├── layer.js
│   └── route.js
├── utils.js
└── view.js
```

这种程度，说它是一个“框架”可能都有些过了，几乎都是工具性质的实现，只限于 Web 层。

当然，直接了当地实现了 Web 层的基本功能，是得益于 *Node.js* 本身的 API 中，就提供了 *net* 和 *http* 这两层， *Express* 对 *http* 的方法包装一下即可。

不过，本身功能简单的东西，在 `package.json` 中却有好长一串 *dependencies* 列表。

# Hello World

在跑 *Express* 前，你可能需要初始化一个 *npm* 项目，然后再使用 *npm* 安装 *Express*：

```bash
mkdir p
cd p
npm init
npm install express --save
```

新建一个 `app.js` ：

```tsx
const express = require('express');
const app = express();
app.all('/', (req, res) => res.send('hello') );
app.listen(8888);
```

调试信息是通过环境变量 *DEBUG* 控制的：

```jsx
const process = require('process');
process.env['DEBUG'] = 'express:*';
```

这样就可以在终端看到带颜色的输出了，嗯，是的，带颜色控制字符，vim 中直接跑就 SB 了。

# 应用 Application

*Application* 是一个上层统筹的概念，整合“请求-响应”流程。 `express()` 的调用会返回一个 *application* ，一个项目中，有多个 *app* 是没问题的：

```tsx
const express = require('express');

const app = express();
app.all('/', (req, res) => res.send('hello'));
app.listen(8888);

const app2 = express();
app2.all('/', (req, res) => res.send('hello2'));
app2.listen(8889);
```

多个 *app* 的另一个用法，是直接把某个 *path* 映射到整个 *app* ：

```tsx
const express = require('express');

const app = express();

app.all('/', (req, res) => {
    res.send('ok');
});

const app2 = express();
app2.get('/xx', (req, res, next) => res.send('in app2') )
app.use('/2', app2)

app.listen(8888);
```

这样，当访问 `/2/xx` 时，就会看到 `in app2` 的响应。

前面说了 *app* 实际上是一个上层调度的角色，在看后面的内容之前，先说一下 *Express* 的特点，整体上来说，它的结构基本上是“回调函数串行”，无论是 *app* ，或者 *route*， *handle*， *middleware*这些不同的概念，它们的形式，基本是一致的，就是 `(res, req, next) => {}` ，串行的流程依赖 `next()` 的显式调用。

我们把 *app* 的功能，分成五个部分来说。
## 基本路由
路由器
* 请求方法

* 请求路径

* 请求处理函数

  get:

  ```tsx
  // get:当你以get方法请求/的时候，执行对应的处理函数	
  app.get('/',function(req,res){
      res.send('hello world')
  })
  ```

  post:

  ```tsx
  //post：当你用post方法请求	/	的时候，指定对应的处理函数
  app.post('/',function(req,res){
      res.send("hello world")
  })
  ```

 ##  静态服务

  ```javascript
  //	 /public资源
  app.use(exporess.static('public'))
  // 	/files资源
  app.use(exporess.static('files'))
  //	 /public/XX
  app.use('/public',exporess.static('public'))
  // 	/static/XXX
  app.use('/static',exporess.static('public'))
  //　/static/
  app.use('/static',exporess.static(path.join(__dirname,'publci')))
  ```

  

  ##　在Experss中使用art-template模板引擎

安装　：

```shell
npm 	install --save art-template
npm 	install --save express -art-template
```

配置：

```javascript
app.engine('art',require('express-art-template'))
```

使用：

```javascript
app.get('/',function(req,res){
    //express默认会去项目中的views目录找index.html
    res.rendera('index.html',{
        title:'hello world'
    })
})
```

如果希望修该views视图的渲染目录可以：

```javascript
//第一个文件名字不要写错
app.set('views',目录路径)
```



## 路由 - Handler 映射

```tsx
app.all('/', (req, res, next) => {});
app.get('/', (req, res, next) => {});
app.post('/', (req, res, next) => {});
app.put('/', (req, res, next) => {});
app.delete('/', (req, res, next) => {});
```

上面的代码就是基本的几个方法，路由的匹配是串行的，可以通过 `next()` 控制：

```tsx
const express = require('express');

const app = express();

app.all('/', (req, res, next) => {
    res.send('1 ');
    console.log('here');
    next();
});

app.get('/', (req, res, next) => {
    res.send('2 ');
    console.log('get');
    next();
});

app.listen(8888);
```

对于上面的代码，因为重复调用 `send()` 会报错。

同样的功能，也可以使用 `app.route()` 来实现：

```jsx
const express = require('express');

const app = express();

app.route('/').all( (req, res, next) => {
    console.log('all');
    next();
}).get( (req, res, next) => {
    res.send('get');
    next();
}).all( (req, res, next) => {
    console.log('tail');
    next();
});

app.listen(8888);
```

`app.route()` 也是一种抽象通用逻辑的形式。

还有一个方法是 `app.params` ，它把“命名参数”的处理单独拆出来了（我个人不理解这玩意儿有什么用）：

```tsx
const express = require('express');

const app = express();

app.route('/:id').all( (req, res, next) => {
    console.log('all');
    next();
}).get( (req, res, next) => {
    res.send('get');
    next()
}).all( (req, res, next) => {
    console.log('tail');
});

app.route('/').all( (req, res) => {res.send('ok')});

app.param('id', (req, res, next, value) => {
    console.log('param', value);
    next();
});

app.listen(8888);
```

`app.params` 中的对应函数会先行执行，并且，记得显式调用 `next()` 。

## Middleware

其实前面讲了一些方法，要实现 *Middleware* 功能，只需要 `app.all(/.*/, () => {})` 就可以了， *Express* 还专门提供了 `app.use()` 做通用逻辑的定义：

```tsx
const express = require('express');

const app = express();

app.all(/.*/, (req, res, next) => {
    console.log('reg');
    next();
});

app.all('/', (req, res, next) => {
    console.log('pre');
    next();
});

app.use((req, res, next) => {
    console.log('use');
    next();
});

app.all('/', (req, res, next) => {
    console.log('all');
    res.send('/ here');
    next();
});

app.use((req, res, next) => {
    console.log('use2');
    next();
});

app.listen(8888);
```

注意 `next()` 的显式调用，同时，注意定义的顺序， `use()` 和 `all()` 顺序上是平等的。

*Middleware* 本身也是 `(req, res, next) => {}` 这种形式，自然也可以和 *app* 有对等的机制——接受路由过滤， *Express* 提供了 *Router* ，可以单独定义一组逻辑，然后这组逻辑可以跟 *Middleware*一样使用。

```tsx
const express = require('express');
const app = express();
const router = express.Router();

app.all('/', (req, res) => {
    res.send({a: '123'});
});

router.all('/a', (req, res) => {
    res.send('hello');
});

app.use('/route', router);

app.listen(8888);
```

## 功能开关，变量容器

`app.set()` 和 `app.get()` 可以用来保存 *app* 级别的变量（对， `app.get()` 还和 *GET* 方法的实现名字上还冲突了）：

```tsx
const express = require('express');

const app = express();

app.all('/', (req, res) => {
    app.set('title', '标题123');
    res.send('ok');
});

app.all('/t', (req, res) => {
    res.send(app.get('title'));
});

app.listen(8888);
```

上面的代码，启动之后直接访问 `/t` 是没有内容的，先访问 `/` 再访问 `/t` 才可以看到内容。

对于变量名， *Express* 预置了一些，这些变量的值，可以叫 *settings* ，它们同时也影响整个应用的行为：

- `case sensitive routing`
- `env`
- `etag`
- `jsonp callback name`
- `json escape`
- `json replacer`
- `json spaces`
- `query parser`
- `strict routing`
- `subdomain offset`
- `trust proxy`
- `views`
- `view cache`
- `view engine`
- `x-powered-by`

具体的作用，可以参考 [https://expressjs.com/en/4x/api.html#app.set](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fexpressjs.com%2Fen%2F4x%2Fapi.html%23app.set) 。

（上面这些值中，干嘛不放一个最基本的 *debug* 呢……）

除了基本的 `set() / get()` ，还有一组 `enable() / disable() / enabled() / disabled()` 的包装方法，其实就是 `set(name, false)` 这种。 `set(name)` 这种只传一个参数，也可以获取到值，等于 `get(name)` 。

## 模板引擎

*Express* 没有自带模板，所以模板引擎这块就被设计成一个基础的配置机制了。

```tsx
const process = require('process');
const express = require('express');
const app = express();

app.set('views', process.cwd() + '/template');

app.engine('t2t', (path, options, callback) => {
    console.log(path, options);
    callback(false, '123');
});

app.all('/', (req, res) => {
    res.render('demo.t2t', {title: "标题"}, (err, html) => {
        res.send(html)
    });
});

app.listen(8888);
```

`app.set('views', ...)` 是配置模板在文件系统上的路径， `app.engine()` 是扩展名为标识，注册对应的处理函数，然后， `res.render()` 就可以渲染指定的模板了。 `res.render('demo')` 这样不写扩展名也可以，通过 `app.set('view engine', 't2t')` 可以配置默认的扩展名。

这里，注意一下 `callback()` 的形式，是 `callback(err, html)` 。

## 端口监听

*app* 功能的最后一部分， `app.listen()` ，它完成的形式是：

```css
app.listen([port[, host[, backlog]]][, callback])
```

注意， `host` 是第二个参数。

`backlog` 是一个数字，配置可等待的最大连接数。这个值同时受操作系统的配置影响。默认是 512 。

# 请求 Request

这一块倒没有太多可以说的，一个请求你想知道的信息，都被包装到 `req` 的属性中的。除了，头。头的信息，需要使用 `req.get(name)` 来获取。

## GET 参数

使用 `req.query` 可以获取 GET 参数：

```tsx
const express = require('express');
const app = express();

app.all('/', (req, res) => {
    console.log(req.query);
    res.send('ok');
});

app.listen(8888);
```

请求：

```csharp
# -*- coding: utf-8 -*-
import requests
requests.get('http://localhost:8888', params={"a": '中文'.encode('utf8')})
```

## POST 参数

POST 参数的获取，使用 `req.body` ，但是，在此之前，需要专门挂一个 Middleware ， `req.body`才有值：

```tsx
const express = require('express');
const app = express();

app.use(express.urlencoded({ extended: true }));
app.all('/', (req, res) => {
    console.log(req.body);
    res.send('ok');
});

app.listen(8888);
# -*- coding: utf-8 -*-

import requests

requests.post('http://localhost:8888', data={"a": '中文'})
```

如果你是整块扔的 json 的话：

```kotlin
# -*- coding: utf-8 -*-

import requests
import json

requests.post('http://localhost:8888', data=json.dumps({"a": '中文'}),
              headers={'Content-Type': 'application/json'})
```

*Express* 中也有对应的 `express.json()` 来处理：

```tsx
const express = require('express');
const app = express();

app.use(express.json());
app.all('/', (req, res) => {
    console.log(req.body);
    res.send('ok');
});

app.listen(8888);
```

*Express* 中处理 `body` 部分的逻辑，是单独放在 `body-parser` 这个 npm 模块中的。 *Express* 也没有提供方法，方便地获取原始 raw 的内容。另外，对于 POST 提交的编码数据， *Express* 只支持 UTF-8 编码。

如果你要处理文件上传，嗯， *Express* 没有现成的 Middleware ，额外的实现在 [https://github.com/expressjs/multer](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fgithub.com%2Fexpressjs%2Fmulter) 。（ Node.js 天然没有“字节”类型，所以在字节级别的处理上，就会感觉很不顺啊）

## Cookie

Cookie 的获取，也跟 POST 参数一样，需要外挂一个 `cookie-parser` 模块才行：

```php
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(cookieParser())
app.all('/', (req, res) => {
    console.log(req.cookies);
    res.send('ok');
});

app.listen(8888);
```

请求：

```kotlin
# -*- coding: utf-8 -*-

import requests
import json

requests.post('http://localhost:8888', data={'a': '中文'},
              headers={'Cookie': 'a=1'})
```

如果 Cookie 在响应时，是配置 *res* 做了签名的，则在 *req* 中可以通过 `req.signedCookies` 处理签名，并获取结果。

## 来源 IP

*Express* 对 `X-Forwarded-For` 头，做了特殊处理，你可以通过 `req.ips` 获取这个头的解析后的值，这个功能需要配置 `trust proxy` 这个 *settings* 来使用：

```php
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(cookieParser())
app.set('trust proxy', true);
app.all('/', (req, res) => {
    console.log(req.ips);
    console.log(req.ip);
    res.send('ok');
});

app.listen(8888);
```

请求：

```kotlin
# -*- coding: utf-8 -*-

import requests
import json

#requests.get('http://localhost:8888', params={"a": '中文'.encode('utf8')})
requests.post('http://localhost:8888', data={'a': '中文'},
              headers={'X-Forwarded-For': 'a, b, c'})
```

如果 `trust proxy` 不是 `true` ，则 `req.ip` 会是一个 ipv4 或者 ipv6 的值。

# 响应 Response

*Express* 的响应，针对不同类型，本身就提供了几种包装了。

## 普通响应

使用 `res.send` 处理确定性的内容响应：

```ruby
res.send({ some: 'json' });
res.send('<p>some html</p>');
res.status(404); res.end();
res.status(500); res.end();
```

`res.send()` 会自动 `res.end()` ，但是，如果只使用 `res.status()` 的话，记得加上 `res.end()` 。

## 模板渲染

模板需要预先配置，在 *Request* 那节已经介绍过了。

```php
const process = require('process');
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(cookieParser())

app.set('trust proxy', false);
app.set('views', process.cwd() + '/template');
app.set('view engine', 'html');
app.engine('html', (path, options, callback) => {
    callback(false, '<h1>Hello</h1>');
});

app.all('/', (req, res) => {
    res.render('index', {}, (err, html) => {
        res.send(html);
    });
});

app.listen(8888);
```

这里有一个坑点，就是必须在对应的目录下，有对应的文件存在，比如上面例子的 `template/index.html` ，那么 `app.engine()` 中的回调函数才会执行。都自定义回调函数了，这个限制没有任何意义， `path, options` 传入就好了，至于是不是要通过文件系统读取内容，怎么读取，又有什么关系呢。

## Cookie

`res.cookie` 来处理 *Cookie* 头：

```tsx
const process = require('process');
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(cookieParser("key"))

app.set('trust proxy', false);
app.set('views', process.cwd() + '/template');
app.set('view engine', 'html');
app.engine('html', (path, options, callback) => {
    callback(false, '<h1>Hello</h1>');
});
app.all('/', (req, res) => {
    res.render('index', {}, (err, html) => {
        console.log('cookie', req.signedCookies.a);
        res.cookie('a', '123', {signed: true});
        res.cookie('b', '123', {signed: true});
        res.clearCookie('b');
        res.send(html);
    });
});

app.listen(8888);
```

请求：

```python
# -*- coding: utf-8 -*-

import requests
import json

res = requests.post('http://localhost:8888', data={'a': '中文'},
                    headers={'X-Forwarded-For': 'a, b, c',
                             'Cookie': 'a=s%3A123.p%2Fdzmx3FtOkisSJsn8vcg0mN7jdTgsruCP1SoT63z%2BI'})
print(res, res.text, res.headers)
```

注意三点：

-  `app.use(cookieParser("key"))` 这里必须要有一个字符串做 *key* ，才可以正确使用签名的 cookie 。
-  `clearCookie()` 仍然是用“设置过期”的方式来达到删除目的，`cookie()` 和 `clearCookie()` 并不会整合，会写两组 `b=xx` 进头。
-  `res.send()` 会在连接上完成一个响应，所以，与头相关的操作，都必须放在 `res.send()` 前面。

## 头和其它

`res.set()` 可以设置指定的响应头， `res.rediect(301, 'http://www.zouyesheng.com')` 处理重定向， `res.status(404); res.end()` 处理非 20 响应。

```php
const process = require('process');
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(cookieParser("key"))

app.set('trust proxy', false);
app.set('views', process.cwd() + '/template');
app.set('view engine', 'html');
app.engine('html', (path, options, callback) => {
    callback(false, '<h1>Hello</h1>');
});

app.all('/', (req, res) => {
    res.render('index', {}, (err, html) => {
        res.set('X-ME', 'zys');
        //res.redirect('back');
        //res.redirect('http://www.zouyesheng.com');
        res.status(404);
        res.end();
    });
});

app.listen(8888);
```

`res.redirect('back')` 会自动获取 `referer` 头作为 `Location` 的值，使用这个时，注意 `referer`为空的情况，会造成循环重复重定向的后果。

## Chunk 响应

*Chunk* 方式的响应，指连接建立之后，服务端的响应内容是不定长的，会加个头： `Transfer-Encoding: chunked` ，这种状态下，服务端可以不定时往连接中写入内容（不排除服务端的实现会有缓冲区机制，不过我看 *Express* 没有）。

```tsx
const process = require('process');
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(cookieParser("key"))

app.set('trust proxy', false);
app.set('views', process.cwd() + '/template');
app.set('view engine', 'html');
app.engine('html', (path, options, callback) => {
    callback(false, '<h1>Hello</h1>');
});

app.all('/', (req, res) => {
    const f = () => {
        const t = new Date().getTime() + '\n';
        res.write(t);
        console.log(t);
        setTimeout(f, 1000);
    }

    setTimeout(f, 1000);
});

app.listen(8888);
```

上面的代码，访问之后，每过一秒，都会收到新的内容。

大概是 `res` 本身是 Node.js 中的 *stream* 类似对象，所以，它有一个 `write()` 方法。

要测试这个效果，比较方便的是直接 telet：

```ruby
zys@zys-alibaba:/home/zys/temp >>> telnet localhost 8888
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
GET / HTTP/1.1
Host: localhost

HTTP/1.1 200 OK
X-Powered-By: Express
Date: Thu, 20 Jun 2019 08:11:40 GMT
Connection: keep-alive
Transfer-Encoding: chunked

e
1561018300451

e
1561018301454

e
1561018302456

e
1561018303457

e
1561018304458

e
1561018305460

e
1561018306460
```

每行前面的一个字节的 `e` ，为 16 进制的 14 这个数字，也就是后面紧跟着的内容的长度，是 *Chunk* 格式的要求。具体可以参考 HTTP 的 RFC ， [https://tools.ietf.org/html/rfc2616#page-2](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc2616%23page-2) 。

Tornado 中的类似实现是：

```python
# -*- coding: utf-8 -*-

import tornado.ioloop
import tornado.web
import tornado.gen
import time

class MainHandler(tornado.web.RequestHandler):
    @tornado.gen.coroutine
    def get(self):
        while True:
            yield tornado.gen.sleep(1)
            s = time.time()
            self.write(str(s))
            print(s)
            yield self.flush()

def make_app():
    return tornado.web.Application([
        (r"/", MainHandler),
    ])

if __name__ == "__main__":
    app = make_app()
    app.listen(8888)
    tornado.ioloop.IOLoop.current().start()
```

*Express* 中的实现，有个大坑，就是：

```tsx
app.all('/', (req, res) => {
    const f = () => {
        const t = new Date().getTime() + '\n';
        res.write(t);
        console.log(t);
        setTimeout(f, 1000);
    }

    setTimeout(f, 1000);
});
```

这段逻辑，在连接已经断了的情况下，并不会停止，还是会永远执行下去。所以，你得自己处理好：

```tsx
const process = require('process');
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(cookieParser("key"))

app.set('trust proxy', false);
app.set('views', process.cwd() + '/template');
app.set('view engine', 'html');
app.engine('html', (path, options, callback) => {
    callback(false, '<h1>Hello</h1>');
});

app.all('/', (req, res) => {
    let close = false;
    const f = () => {
        const t = new Date().getTime() + '\n';
        res.write(t);
        console.log(t);
        if(!close){
            setTimeout(f, 1000);
        }
    }

    req.on('close', () => {
        close = true;
    });

    setTimeout(f, 1000);
});

app.listen(8888);
```

`req` 挂了一些事件的，可以通过 `close` 事件来得到当前连接是否已经关闭了。

`req` 上直接挂连接事件，从 `net` `http` `Express` 这个层次结构上来说，也很，尴尬了。 Web 层不应该关心到网络连接这么底层的东西的。

我还是习惯这样：

```tsx
app.all('/', (req, res) => {
    res.write('<h1>123</h1>');
    res.end();
});
```

不过 `res.write()` 是不能直接处理 json 对象的，还是老老实实 `res.send()` 吧。

# 我会怎么用 Express

先说一下，我自己，目前在 *Express* 运用方面，并没有太多的时间和复杂场景的积累。

即使这样，作为技术上相对传统的人，我会以我以往的 web 开发的套路，来使用 *Express* 。

我不喜欢日常用 `app.all(path, callback)` 这种形式去组织代码。

首先，这会使 `path` 定义散落在各处，方便了开发，麻烦了维护。

其次，把 `path` 和具体实现逻辑 `callback` 绑在一起，我觉得也是反思维的。至少，对于我个人来说，开发的过程，先是想如何实现一个 *handler* ，最后，再是考虑要把这个 *handle* 与哪些 `path` 绑定。

再次，单纯的 `callback` 缺乏层次感，用 `app.use(path, callback)` 这种来处理共用逻辑的方式，我觉得完全是扯谈。共用逻辑是代码之间本身实现上的关系，硬生生跟网络应用层 HTTP 协议的 `path` 概念抽上关系，何必呢。当然，对于 `callback` 的组织，用纯函数来串是可以的，不过我在这方面并没有太多经验，所以，我还是选择用类继承的方式来作层次化的实现。

我自己要用 *Express* ，大概会这样组件项目代码（不包括关系数据库的 *Model* 抽象如何组织这部分）：

```csharp
./
├── config.conf
├── config.js
├── handler
│   ├── base.js
│   └── index.js
├── middleware.js
├── server.js
└── url.js
```

-  `config.conf` 是 ini 格式的项目配置。
-  `config.js` 处理配置，包括日志，数据库连接等。
-  `middleware.js` 是针对整体流程的扩展机制，比如，给每个请求加一个 UUID ，每个请求都记录一条日志，日志内容有请求的细节及本次请求的处理时间。
-  `server.js` 是主要的服务启动逻辑，整合各种资源，命令行参数 *port* 控制监听哪个端口。不需要考虑多进程问题，（正式部署时 *nginx* 反向代理到多个应用实例，多个实例及其它资源统一用 *supervisor* 管理）。
-  `url.js` 定义路径与 *handler* 的映射关系。
-  `handler` ，具体逻辑实现的地方，所有 `handler` 都从 `BaseHandler` 继承。

`BaseHandler` 的实现：

```kotlin
class BaseHandler {
    constructor(req, res, next){
        this.req = req;
        this.res = res;
        this._next = next;
        this._finised = false;
    }

    run(){
        this.prepare();
        if(!this._finised){
            if(this.req.method === 'GET'){
                this.get();
                return;
            }
            if(this.req.method === 'POST'){
                this.post();
                return;
            }
            throw Error(this.req.method + ' this method had not been implemented');
        }
    }

    prepare(){}
    get(){
        throw Error('this method had not been implemented');
    }
    post(){
        throw Error('this method had not been implemented');
    }

    render(template, values){
        this.res.render(template, values, (err, html) => {
            this.finish(html);
        });
    }

    write(content){
        if(Object.prototype.toString.call(content) === '[object Object]'){
            this.res.write(JSON.stringify(content));
        } else {
            this.res.write(content);
        }
    }

    finish(content){
        if(this._finised){
            throw Error('this handle was finished');
        }
        this.res.send(content);
        this._finised = true;
        if(this._next){ this._next() }
    }

}

module.exports = {BaseHandler};

if(module === require.main){
    const express = require('express');
    const app = express();
    app.all('/', (req, res, next) => new BaseHandler(req, res, next).run() );
    app.listen(8888);
}
```

要用的话，比如 `index.js` ：

```jsx
const BaseHandler = require('./base').BaseHandler;

class IndexHandler extends BaseHandler {
    get(){
        this.finish({a: 'hello'});
    }
}

module.exports = {IndexHandler};
```

`url.js` 中的样子：

```java
const IndexHandler = require('./handler/index').IndexHandler;

const Handlers = [];

Handlers.push(['/', IndexHandler]);

module.exports = {Handlers};
```

# 日志

后面这几部分，都不属于 *Express* 本身的内容了，只是我个人，随便想到的一些东西。

找一个日志模块的实现，功能上，就看这么几点：

- 标准的级别： DEBUG，INFO，WARN, ERROR 这些。
- 层级的多个 *logger* 。
- 可注册式的多种 *Handler* 实现，比如文件系统，操作系统的 *rsyslog* ，标准输出，等。
- 格式定义，一般都带上时间和代码位置。

Node.js 中，大概就是 *log4js* 了， [https://github.com/log4js-node/log4js-node](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fgithub.com%2Flog4js-node%2Flog4js-node) 。

```tsx
const log4js = require('log4js');

const layout = {
    type: 'pattern',
    pattern: '- * %p * %x{time} * %c * %f * %l * %m',
    tokens: {
        time: logEvent => {
            return new Date().toISOString().replace('T', ' ').split('.')[0];
        }
    }
};
log4js.configure({
  appenders: {
        file: { type: 'dateFile', layout: layout, filename: 'app.log', keepFileExt: true },
        stream: { type: 'stdout', layout: layout }
  },
  categories: {
      default: { appenders: [ 'stream' ], level: 'info', enableCallStack: false },
      app: { appenders: [ 'stream', 'file' ], level: 'info', enableCallStack: true }
  }
});

const logger = log4js.getLogger('app');
logger.error('xxx');

const l2 = log4js.getLogger('app.good');
l2.error('ii');
```

总的来说，还是很好用的，但是官网的文档不太好读，有些细节的东西没讲，好在源码还是比较简单。

说几点：

-  `getLogger(name)` 需要给一个名字，否则 `default` 的规则都匹配不到。
-  `getLogger('parent.child')` 中的名字，规则匹配上，可以通过 `.` 作父子继承的。
-  `enableCallStack: true` 加上，才能拿到文件名和行号。

# ini 格式配置

json 作配置文件，功能上没问题，但是对人为修改是不友好的。所以，个人还是喜欢用 ini 格式作项目的环境配置文件。

Node.js 中，可以使用 *ini* 模块作解析：

```php
const s = `
[database]
host = 127.0.0.1
port = 5432
user = dbuser
password = dbpassword
database = use_this_database

[paths.default]
datadir = /var/lib/data
array[] = first value
array[] = second value
array[] = third value
`

const fs = require('fs');
const ini = require('ini');

const config = ini.parse(s);
console.log(config);
```

它扩展了 `array[]` 这种格式，但没有对类型作处理（除了 `true` `false`），比如，获取 `port` ，结果是 `"5432"` 。简单够用了。

# WebSocket

Node.js 中的 WebSocket 实现，可以使用 *ws* 模块， [https://github.com/websockets/ws](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fgithub.com%2Fwebsockets%2Fws) 。

要把 *ws* 的 WebSocket Server 和 *Express* 的 *app* 整合，需要在 *Express* 的 *Server* 层面动手，实际上这里说的 *Server* 就是 Node.js 的 *http* 模块中的 `http.createServer()` 。

```tsx
const express = require('express');
const ws = require('ws');

const app = express();

app.all('/', (req, res) => {
    console.log('/');
    res.send('hello');
});

const server = app.listen(8888);

const wss = new ws.Server({server, path: '/ws'});
wss.on('connection', conn => {
    conn.on('message', msg => {
        console.log(msg);
        conn.send(new Date().toISOString());
    });
});
```

对应的一个客户端实现，来自： [https://github.com/ilkerkesen/tornado-websocket-client-example/blob/master/client.py](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fgithub.com%2Filkerkesen%2Ftornado-websocket-client-example%2Fblob%2Fmaster%2Fclient.py)

```python
# -*- coding: utf-8 -*-

import time
from tornado.ioloop import IOLoop, PeriodicCallback
from tornado import gen
from tornado.websocket import websocket_connect

class Client(object):
    def __init__(self, url, timeout):
        self.url = url
        self.timeout = timeout
        self.ioloop = IOLoop.instance()
        self.ws = None
        self.connect()
        PeriodicCallback(self.keep_alive, 2000).start()
        self.ioloop.start()

    @gen.coroutine
    def connect(self):
        print("trying to connect")
        try:
            self.ws = yield websocket_connect(self.url)
        except Exception:
            print("connection error")
        else:
            print("connected")
            self.run()

    @gen.coroutine
    def run(self):
        while True:
            msg = yield self.ws.read_message()
            print('read', msg)
            if msg is None:
                print("connection closed")
                self.ws = None
                break

    def keep_alive(self):
        if self.ws is None:
            self.connect()
        else:
            self.ws.write_message(str(time.time()))

if __name__ == "__main__":
    client = Client("ws://localhost:8888/ws", 5)
```