使用JS开发的过程中，经常会用到赋值，简单的使用等于号只能对部分对象进行赋值，如Number、String、Boolean等，但对于Object、Array、Date等对象等于号只是传递对象的地址，并没有在内存上分配新的空间来存放复制的内容，所以在复制这些对象是需要使用深度复制，即克隆。下面写一个克隆的方法，可作为一个工具类的常用方法，注意下面两个知识点。

* DOM节点对象自带domNode.cloneNode(argument)方法，argument为true时将domNode本身以及其子节点作为副本返回，argument为false时仅将其本身作为副本返回，这里argument固定为true。
* object的constructor可区分对象是json对象还是function的实例，json对象只需要遍历赋值，function的实例需要通过构造函数new一个对象，再遍历复制成员变量或函数。

```javascript
function clone (item) {
  if (item === null || item === undefined) {
    return item
  }

  // Dom node
  if (item.nodeType && item.cloneNode) {
    return item.cloneNode(true)
  }

  var i, obj, key
  var type = toString.call(item)

  if (type === '[object Date]') { // Date
    return new Date(item.getTime())
  } else if (type === '[object Array]') { // Array
    obj = []
    i = item.length

    while (i--) {
      obj[i] = clone(item[i])
    }
  } else if (type === '[object Object]' && item.constructor === Object) { // Object
    obj = {}

    for (key in item) {
      obj[key] = clone(item[key])
    }
  } else if (type === '[object Object]') { // Instance of function
    obj = new item.constructor()

    for (var attr in item) {
      if (item.hasOwnProperty(attr)) obj[attr] = clone(item[attr])
    }
  }

  return obj || item
}
```
