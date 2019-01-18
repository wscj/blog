伴随着Node.js的出现，前端程序员也可以写后台服务器，也可以写数据库。近来开发一个单机版的APP，数据均保存在本地，由于该APP支持多用户，需要做用户权限控制，所以选用关系型数据库来管理数据比较适合，数据量并不大，所以在关系型数据库中选用轻量级的sqlite最适合不过了。在我们前端小组，也就只有我会数据库，所以数据库的设计，数据库接口的编写都让我包了。这次主要记录下在编写接口中学到的一些东西，所写内容均基于用Node.js操作sqlite的一个包node-sqlite3。

**总结要点如下：**
1. 有专门的接口文档，我这里使用jsdoc将注释自动生成接口文档
1. 写清楚参数的数据类型与参数的意义
1. 如果有返回结果，写清楚返回结果（如果是异步函数，则结果作为回调函数的参数返回）
    * 返回结果中，有专门的字段说明运行是否成功，如下例1中error字段，0表示成功，比0大的整数可以表示不同的错误
    * 返回结果中，有专门的字段包含其他需要返回的信息，如下例1中的result字段，可以是运行错误的错误信息，也可以是运行成功后需要返回的数据
1. 对参数做校验，如参数不合法，在返回结果中写明错误信息
1. 由于对数据库的操作是异步执行的，并且数据库同时运行多个指令会出错，所以必须先完成一个指令才能开始执行下一个指令。如下例2，采用递归，在每次执行完成后的回调函数里开始执行下一个sql命令
1. 尽量减少对数据库的操作次数，如下例3，原本插入n条数据应该执行n次insert命令，但可以使用`select xxx union select xxx`构造成一条sql命令，一次性完成

**参考例子1**

```javascript
/**
 * sqlite数据库模块
 * @module Sqlite
 * @example
 * const Database = require("/path/to/xxx");
 */
(function(Sqlite) {

  /**
   * 数据库对象
   * @author 陈景
   * @member db
   * @type {object}
   * @private
   */
  const db = new require('sqlite3').verbose().Database('/path/to/xx.db')

  /**
   * 获取某些数据
   * @author 陈景
   * @method getSomething
   * @param {object} arg
   * @param {number} arg.id 对应数据库表t1的主键
   * @param {function} arg.callback 回调函数，该函数的参数有以下情况
   *
   * * { error: 0, result: rows } 查询成功，result为查询结果
   * * { error: 1, result: err  } 查询失败，result为错误信息
   */
  Sqlite.getSomething = function (arg) {
    //参数arg校验
    if (toString.call(arg) !== '[object Object]') {
      console.error('Argument is not an object')
      return
    }

    //ID必须为数字
    if (isNaN(arg.id)) {
      console.error('arg.id is not a number')
      arg.callback && arg.callback({ error: 1, result: 'arg.id is not a number' })
      return
    }
    //ID必须为整数
    else if (String(arg.id).indexOf('.') > -1) {
      console.error('arg.id is not an integer')
      arg.callback && arg.callback({ error: 1, result: 'arg.id is not an integer' })
      return
    }
    //ID必须不小于1
    else if (arg.id < 1) {
      console.error('arg.id is less than 1')
      arg.callback && arg.callback({ error: 1, result: 'arg.id is less than 1' })
      return
    }

    let sql = `select rowid, * from t1 where rowid = ${arg.id}`

    db.all(sql, (err, rows) => {
      if (err) {
        console.error(err)
        arg.callback && arg.callback({ error: 1, result: err })
      } else {
        arg.callback && arg.callback({ error: 0, result: rows })
      }
    })
  }
}(module.exports))
```

**参考例子2**

```javascript
/**
 * 初始化数据库，创建表、触发器等
 * @method init
 * @author 陈景
 */
Sqlite.init = function() {
  let sqls = []

  sqls.push(`Create Table t1 (name NVarchar(100) Default '', sex Integer)`)
  sqls.push(`Create Table t2 (name NVarchar(100) Default '')`)
  sqls.push(`Create Table t3 (name NVarchar(100) Default '')`)

  //采用递归逐条运行sql命令，避免同时运行多条sql命令而导致错误
  (function exec(sqls) {
    if (sqls.length) {
      db.run(sqls[0], (err) => {
        if (err) {
          console.error(err)
        } else {
          sqls.shift()
          exec(sqls)
        }
      })
    }
  }(sqls))
}
```

**参考例子3**

```javascript
/**
 * 添加数据，可一次性添加多条数据
 * @method addSomething
 * @author 陈景
 * @param {object}  arg
 * @param {Array} arg.list 需要被添加的数据信息
 * @param {Function} arg.callback 回调函数，该函数的参数用以下情况
 *
 * * { error: 0 } 添加成功
 * * { error: 1 } 添加失败
 */
Sqlite.addSomething = function(arg) {

  // ... 此处省略对参数的校验

  if (arg.list.length) {

    let sql = `Insert Into t1 (name, sex) `

    //把多个用户的数据构造成一条sql语句，
    for (let i = 0 i < arg.list.length i++) {
      if (i !== 0) {
        sql += ' union '
      }
      sql += `Select '${arg.list[i].name}', ${arg.list[i].sex}`
    }

    db.run(sql, (err) => {
      if (err) {
        console.error(err)
        arg.callback && arg.callback({ error: 1 })
      } else {
        arg.callback && arg.callback({ error: 0 })
      }
    })
  } else {
    arg.callback && arg.callback({ error: 0 })
  }
}
```
