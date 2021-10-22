## 什么是函数柯里化  
来自维基百科的解释：
> 在计算机科学中，柯里化（ Currying ）是把接受多个参数的函数变换成接受一个单一参数(最初函数的第一个参数)的函数，并且返回接受余下的参数且返回结果的新函数的技术。  
    
可以知道这是一种思想一种函数变形。并不会执行函数，实现不区分语言。以上说法已经很通俗了。简单来说当参数数量不够时，利用闭包把当前这个参数存起来，返回一个新的函数，等待返回的这个新函数继续被调用，继续获取参数，并把这个参数和闭包存的参数合并到一起，如果这个参数数量数量等于被柯里化函数形参数量，就开始执行，否则继续返回新函数，继续存参数，重复这个步骤。用 `JavaScript` 举个例子：   

```javascript  
add(1, 2, 3, 4); // 创建个简单函数，把所有参数累加并返回结果10 

// 把这个函数柯里化之后，就是把多个参数分解成改成单或多个组合传入，并返回相同结果。看起来像这样：
curryingAdd(1)(2)(3)(4) // 10
// 或者
curryingAdd(1, 2)(3)(4) // 10
// 或者
curryingAdd(1)(2, 3, 4) // 10
```

## 柯里化的好处  
看了以上的例子，看起来没什么用，反而实现起来更复杂。实际上这是一种高度抽象编程规范，属于函数式编程的思想，主要好处为：
   
- 参数复用
  
还有`提前确认`和`延迟执行`好处，其实都是一个意思，个人认为参数复用是最明显的好处，利用好可以写出更优雅的代码，让机器更像人类一步一步实现某件事，一段伪代码来举例：    

```javascript
function ajax(method, url){
// http请求的实现...
}  

function currying(fun){
// 柯里化实现...
}

把ajax方法柯里化
let curryingAjax = currying(ajax);

// 得到一个get请求，只执行一次
let getAjax = curryingAjax("get");

// 关联业务逻辑，获取用户信息接口
let getUserInfo = getAjax("http://api.test.com/getUserInfo");

// 关联业务逻辑，获取订单列表接口
let getOrderList = getAjax("http://api.test.com/getOrderList");
```  

可以看出`get`传入`curryingAjax`方法仅一次，之后可以调用多次`get`请求。先确定请求方法，在确定具体请求地址，这就实现了参数的复用。如果是`post`请求：  

```
let postAjax = curryingAjax("post");

// 之后根据业务逻辑多次调用
...
```  
## 柯里化完整代码  
 - 判断`currying`的参数第二位到最后一位（也就是`args`）的长度，如果长度等于`fn`形参长度，那么直接执行`fn`，并把`args`作为参数传入
 - 如果传入参数（`args`）不够时，利用递归和闭包，暂存传入的参数，并重新返回`currying`函数    

```javascript 
function currying(fn, ...args) {
  if (args.length >= fn.length) {
    return fn(...args);
  } else {
    return (...args2) => currying(fn, ...args, ...args2);
  }
}
```