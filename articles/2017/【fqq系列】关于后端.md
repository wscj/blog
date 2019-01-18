后端使用了基于`Node.js`的框架`Express.js`，这是前端程序员最容易接受的后端框架了。后端的功能主要是提供接口响应前端的请求，期间很可能需要与数据库交互，所以这里还引入了`Sqlite`数据库，有了后台框架与数据库，后台基本的架构就完整了。

下文分为`Express.js`与`Sqlite`两部分

## 后端框架部分

在`Vue-cli`生成的代码中已经包含了`Express.js`框架，在开发环境会使用到它，生产环境是不需要的，所以理论上我应该另外新建一个`Express.js`的项目作为后台服务，中间用代理把前端的`http`请求转发给这个服务。但是把前、后端的代码都写在同个项目很方便管理，权衡之后决定不另外增加后台项目，直接利用已有的`Express.js`编写后端的业务代码。当然，最后在打包生成静态文件用于部署生产环境时，我把该项目里的后端部分抽出去作为一个[独立项目](https://github.com/tadashi-chen/fqq-backend)，用于测试生产环境，以保证该项目的完整性。

后端主要是为前端提供接口，所以这里用到了`Express.js`的路由。新建一个文件`fqq/build/router.js`，这里便是路由的全部内容，包含了与前端交互的所有接口。后端的程序的入口文件是`fqq/build/dev-server.js`，在里面把路由引入进来：
```js
var router = require('./router')
app.use(router)
```
路由里有一个拦截器，其它的都是方法（接口），先看拦截器的代码：
```js
//处理身份验证
router.use(function (req, res, next) {

  function unvalid (res) {
    res.append('Verify', 'fail')
    res.sendFile(path.join(__dirname, '../index.html'))
  }

  //登录的请求不需要token
  if (req.url.substr(0, 6) !== '/login') {
    const token = req.headers['authorization']
    if (!token) {
      unvalid(res)
    }
    else {
      try {
        const jwt = require('jsonwebtoken')
        const decoded = jwt.verify(token, 'mySecret')
        req.account = decoded.account
        req.uid = decoded.uid
        next()
      } catch (err) {
        unvalid(res)
      }
    }
  } else {
    next()
  }
})
```
前台发送过来的每个请求都先经过拦截器，所以在拦截器里验证`token`信息最合适，如果`token`合法则从中取出用户的ID放入请求头部，并把请求转交到下一步，如果不合法则直接给前台返回验证失败信息，让前台跳转到登录界面。

再看一个接口的代码：
```js
router.get('/getFriendList', (req, res) => {
  Sqlite.getFriendList({
    uid: req.uid,
    callback: function (arg) {
      (arg.error === 0) && res.send({ list: arg.list })
    }
  })
})
```
接口的代码基本都是这种格式，不同接口根据各自的需要，调用相应的数据库接口去获取对应的数据，然后把数据返回给前台。因为对`Sqlite`数据库的操作都是采用异步的方式，所以响应前端请求的逻辑都在回调函数里实现。

## 数据库部分

现在流行的数据库有很多种，与`Node.js`搭配最多的是`mongodb`，但我觉得在本项目使用`Sqlite`更方便，因为不需要另外安装数据库软件，`Node.js`有一个[现成的包](https://github.com/mapbox/node-sqlite3)可以直接操作`Sqlite`，所以`npm install`的时候就把数据库的环境一块给配置好了。也不需要另外开启数据库服务，随时都可以读取，如果不存在数据库文件也可以直接创建，很方便。另外我个人也比较喜欢写`sql`，而`mongodb`是非关系型数据库。对数据库的操作基本都是增删改查，本项目采用的方式是拼凑`sql`语句，再调用相应的接口来执行它们，这里可以查看[接口文档](https://github.com/mapbox/node-sqlite3/wiki/API)。

对数据库的操作我也单独写成一个模块，新建文件`fqq/build/sqlite.js`，然后在需要用到的地方引入即可，本项目只有`fqq/build/router.js`用到了。
```js
//引用数据库模块
const Sqlite = require('./sqlite')

//调用数据库模块的一个接口
Sqlite.getFriendList({
  uid: req.uid,
  callback: function (arg) {
    (arg.error === 0) && res.send({ list: arg.list })
  }
})
```

`Node.js`本身是单线程的，操作数据库的最佳方式是异步而非同步，所以本项目一律采用异步的方式。异步的写法会增加编写业务代码的难度，比如我先调用一个接口创建一张数据库表，然后再调用另一个接口对刚创建的表插入数据，但由于这两个接口的运行是异步的，不能保证哪个先被运行完成，一旦先执行插入数据的操作，那必定报数据库无此表的错误，所以这样是存在隐患的。这里我用了一个小技巧，就是使用递归的方式：
> 把所有数据库操作的指令存放到一个数组，用递归的方式逐个执行这些指令，在每轮递归成功的回调函数里去开启下一轮的递归，由于回调函数是在数据库操作完成之后才会触发，所以这样能保证执行的顺序，如下示例：
```js
//待执行的sql命令
const sqls = ['create table ...', 'create ...', 'insert into ...']
(function exec (sqls) {
  if (sqls.length) {
    db.run(sqls.shift(), (err) => {
      err ? console.error(err) : exec(sqls) //回调函数中继续下一轮的递归
    })
  }
  else {
    //结束递归
  }
}(sqls))
```
