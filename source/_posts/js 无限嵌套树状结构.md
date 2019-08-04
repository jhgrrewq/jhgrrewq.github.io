---
title: js 无限嵌套树状结构
tag: 
	- js
---

当后端返回节点数据不是树状结构时，需要前端进行转化 (其实借助了对象的引用，最终处理后只需要返回顶层节点)

注意 **正式使用需要注意对数据进行深拷贝，避免对传入的数据影响**

```js
let data = [{
  id: '1', pid: ''
},{
  id: '2', pid: ''
},{
  id: '3', pid: '2'
},{
  id: '3.1', pid: '3'
},{
  id: '3.2', pid: '3'
},{
  id: '3.1.1', pid: '3.1'
},{
  id: '4', pid: ''
}]

function arrToTree(data, idKey = 'id', pidKey = 'pid') {
  let r = []
  let tmp = {}

  for (let i = 0; i < data.length; i++) {
    // 以每条数据的id作为key值，数据作为value值存入到一个临时对象里面
    tmp[data[i][idKey]] = data[i]
  }

  for (let j = 0; j < data.length; j++) {
    const key = tmp[data[j][pidKey]]
    // 循环每一条数据的pid，假如这个临时对象有这个key值，就代表这个key对应的数据有children，需要Push进去
    if (key) {
      if (!key['children']) {
        key['children'] = [data[j]]
      } else {
        key['children'].push(data[j])
      }
    } else {
      // 如果没有这个Key值，那就代表没有父级,直接放在最外层
      r.push(data[j])
    }
  }
  return r
}

arrToTree(data)
```