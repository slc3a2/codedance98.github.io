## index.html 
`index.html`:  

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div id='app'>
    <input type="text" v-model='v'>
    {{v}}
    <button @click='reset'>重置</button>
  </div>
  <script src='./index.js'></script>
  <script>
    const vm = new Mvvm({
      el: '#app',
      data: {
        v: 'hello world'
      },
      methods: {
        reset() {
          this.v = '';
        }
      }
    })
  </script>
</body>
</html>
```    

## index.js  

```javascript
class Mvvm {
  constructor(options) {
    const {el, data, methods} = options;
    this.methods = methods;
    this.target = null;
    // 监听this[data的key]所有属性，让变化可追踪
    this.observe(this, data);
    // dom解析：提取{{}}、v-model、事件重写：@click
    this.compile(document.querySelector(el))
  }
  
  // 数据变化监听
  observe(_this, data) {
    Object.keys(data).forEach((key)=> {
      let val = data[key];
      Object.keys(data).forEach((key)=> {
        // 如果val是数组或者对象，使用递归实现深层监听，直到val为简单数据类型。从而保证所有属性变化都被监听
        if(typeof val === 'object'){
          return this.observe(_this, val)
        }
        // dispatcher用来操作订阅者(watcher) add 或者 update。要配合Object.defineProperty的get和set来使用
        const dispatcher = new Dispatcher();
        Object.defineProperty(_this, key, {
          get: function() {
            console.log('get');
            // this.target会在compile方法中出现，把this.target(一个watcher)添加到dispatcher(将要更新的watcher的列表)中，用于未来更新这个watcher对应的dom
            dispatcher.add(this.target);
            return val
          },
          set: function(newV) {
            // 值无变化，不处理
            if(newV === val){return;}
            console.log(`set`);
            val = newV;
            // 因为set了，值发生变化了，所以要通知get中添加的所有订阅者(watcher)：你们要把对应的dom中使用的值更新成newV
            dispatcher.notify(newV);
          }
        })
      })
    })
  }

  // dom解析
  compile(dom) {
    const childs = dom.childNodes;
    for (const node of childs) {
      // nodeType 参考 https://www.w3school.com.cn/jsref/prop_node_nodetype.asp
      if(node.nodeType === 1) {
        const attrs = node.attributes;
        for(const attr of attrs) {
          if(attr.name === 'v-model') {
            const name = attr.value;
            // 放到订阅者列表中
            this.target = new Watcher(node, 'input');
            // this[name]是为了触发observe的get，才会被监听
            this[name];
            // 由于是demo，假设只有input一种情况，input就会有双向绑定。使用this[name], 并且赋值input的值，来触发observe的get。实现更新信息的发布
            node.addEventListener('input', (e) => {
              this[name] = e.target.value;
            });
          }
          // 使用bind传递this。并代理click事件函数到@click上。这里仅拿click事件实现，实际会有多种事件
          if(attr.name === '@click') {
            const name = attr.value;
            node.addEventListener('click', this.methods[name].bind(this))
          }
        }
      }
      // nodeType 参考 https://www.w3school.com.cn/jsref/prop_node_nodetype.asp
      if(node.nodeType === 3){
        // 正则匹配{{}}
        const reg = /\{\{(.*)\}\}/;
        const match = node.nodeValue.match(reg);
        if(match) {
          const name = match[1].trim();
          // 放到订阅者列表中
          this.target = new Watcher(node, 'text');
          // this[name]是为了触发observe的get，才会被监听
          this[name];
        }
      }
    }
  }
}


// 发布者
class Dispatcher {
  constructor() {
    this.watchers = [];
  }
  // 增加订阅者
  add(watcher) {
    this.watchers.push(watcher)
  }
  // 通知所有订阅者更新
  notify(value){
    this.watchers.forEach((item)=> {
      item.update(value)
    })
  }
}

// 订阅者
class Watcher {
  constructor(node, type) {
    this.node = node;
    this.type = type;
  }
  update(value) {
    // 区别dom类型来赋值
    if (this.type === 'input') {
      this.node.value = value;
    }
    if (this.type === 'text') {
      this.node.nodeValue = value;
    }
  }
}
```

vue在初始化后，执行`Observe`函数把data利用`Object.defineProperty`属性监听。同时也会使用`Compile`函数循环`dom`，提取vue相关的关键字，v-bind或者v-model，找到这些值，新建一个`Watcher`实例，然后手动get使这些watch放入dep列表中等待订阅。等待调用`Observer`的`set`(input事件，或者手动赋值)，然后通知dep中所有`Watcher`调用`update`方法。