前段时间用`Express.js`实现了一个`RESTful API`，今天做个粗略的总结，具体实现细节请查看[源码](https://github.com/wscj/restapi)。

## 明确需要实现哪些路由

借鉴`Rails`（一个典型的`RESTful API`后端框架），在`Rails`里，假如有一个标准的REST资源`articles`，那么这个资源就有下面这些路由：

```rb
    articles GET    /articles(.:format)          articles#index
             POST   /articles(.:format)          articles#create
 new_article GET    /articles/new(.:format)      articles#new
edit_article GET    /articles/:id/edit(.:format) articles#edit
     article GET    /articles/:id(.:format)      articles#show
             PATCH  /articles/:id(.:format)      articles#update
             PUT    /articles/:id(.:format)      articles#update
             DELETE /articles/:id(.:format)      articles#destroy
```

这里总共8个路由，但并不是每个都是`RESTful API`所必须的，第3个和第4个是由于`Rails`自身的需要而增加的。所以，我们只需要实现另外的6个，如果以`books`作为标准的REST资源，那么我们需要为这个资源实现下面这6个路由：

```
  POST /books
   GET /books
   GET /books/:id
   PUT /books/:id
 PATCH /books/:id
DELETE /books/:id
```

## 配置路由

以`books`为REST资源写一个具体的路由

```js
router.get('/api/books', (req, res) => {  //获取全部书本信息
  model.getAll('books', req.query, (arg) => {
    send(res, arg)
  })
})
.get('/api/books/:id', (req, res) => {    //获取单一本书信息
  model.getOne('books', req.params.id, (arg) => {
    send(res, arg)
  })
})
.post('/api/books', (req, res) => {       //创建一本书
  model.create('books', req.body, (arg) => {
    send(res, arg)
  })
})
.put('/api/books/:id', (req, res) => {    //修改一本书，理论上需要提供全部字段，但本项目PUT的实现与PATCH一致，只需提供修改的字段
  model.update('books', req.params.id, req.body, (arg) => {
    send(res, arg)
  })
})
.patch('/api/books/:id', (req, res) => {  //修改一本书，仅需提供修改字段
  model.update('books', req.params.id, req.body, (arg) => {
    send(res, arg)
  })
})
.delete('/api/books/:id', (req, res) => { //删除一本书
  model.delete('books', req.params.id, (arg) => {
    send(res, arg)
  })
})
```

解释下上面的代码：

* 这里6个方法对应6个需要实现的路由，也是**动作**与**URI**的6种**非重复**组合，URI有`/api/books`与`/api/books/:id`两种，动作有`GET`、`POST`、`PUT`、`PATCH`、`DELETE`五种
* 所有URI都加上了`/api`前缀作为一个特殊标识，但这不是必需的
* 前端发出符合REST标准的HTTP请求，到达后端会根据URI与动作进入上述6个方法中对应的一个
* 路由的每个函数都做相同的3件事
    * 获取请求的参数
    * 调用`model`相应的方法处理相关业务
    * 写好对结果的处理作为回调函数，响应前端请求
    
## Model

在`Rails`里，每个资源都对应一个`model`，每个`model`又对应一张数据库表，对数据库的操作全都封装在`model`里。`Rails`作为一个优秀的框架，其封装级别很高，使用也很方便，比如有一个REST资源`articles`，只需要在路由文件`routes.rb`添加下面这一句代码，即自动生成所有需要的REST接口：

```rb
resources :articles
```

本文的目标是实现`RESTful`接口，所以尽可能采用简单的方式，没有给每个REST资源配备一个`model`，而是使用同一个`model`，用**资源名**作为参数来区分资源。正因为`model`是公用的，所以不在`model`里处理业务，而是在路由里处理（这样做并不是很好）。

`model`需要的实现功能比较简单，来看看代码

```javascript
const model = {}

//这里可以是其他的数据库
const db = require('./sqlite')

model.getAll = (table, filter, callback) => {
  db.getAll(table, filter, callback)
}

model.getOne = (table, id, callback) => {
  db.getOne(table, id, callback)
}

model.create = (table, fields, callback) => {
  db.create(table, fields, callback)
}

model.update = (table, id, fields, callback) => {
  db.update(table, id, fields, callback)
}

model.delete = (table, id, callback) => {
  db.delete(table, id, callback)
}

model.getSubResource = (table, foreignKey, callback) => {
  db.getSubResource(table, foreignKey, callback)
}

module.exports = model
```

`model`的核心是6个函数，前面5个是对应`GET`、`POST`、`PUT`、`PATCH`、`DELETE` 这5个动作，最后一个用于获取父子资源。每个函数的功能都比较简单，只要调用数据库模块提供的对应的函数，并把路由传进来的参数传进去即可。对数据的处理在数据库模块完成，对最终结果的处理已在路由以回调函数的形式传递进来，所以`model`的任务很轻松。

## 数据库

数据库模块的核心也是6个函数，与`model`的6个核心函数相对应。这6个函数的任务是根据参数对数据做处理，最后把结果作为参数执行回调函数，完成一个响应请求的最后一步。

数据库有多种，使用不同的数据库需要编写不同的模块，但不论是哪种数据库模块，都需要具备上述的6个函数，并且函数的处理逻辑也要基本相同，只是实现的方式有所差异。以`sqlite`数据库举例看一个函数的实现逻辑：

```js
Sqlite.update = (resource, id, fields, callback) => {

  //第一步
  let sql = `update ${resource} set `
  Object.keys(fields).forEach((key) => {
    sql += `${key}='${fields[key]}',`
  })
  sql = sql.substr(0, sql.length - 1) + ` where rowid=${id}`
  
  //第二步
  db.run(sql, function(err) {
    //第三步
    if (err) {
      console.error(err)
    }
    else if (this.changes === 0) {
      callback({ error: 404 })
    }
    else {
      Sqlite.getOne(resource, id, callback, 201)
    }
  })

}
```

实现逻辑：

1. 根据函数的功能与参数拼凑好待执行的SQL代码字符串（第一步）
2. 调用数据库接口执行上述的SQL代码（第二步）
3. 根据执行结果以确定下一步（第三步）

## 最后

`RESTful API`很实用，免去接口取名的麻烦，也减少前、后端程序员对接的工作量，以后的工作尽量采用这种方式。
