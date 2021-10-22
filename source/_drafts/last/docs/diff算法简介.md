## diff算法和virtual dom

![WechatIMG3177.png](./img/diff-arithmetic/HrGjhDp8ebSt62R.png)  

在现代`mvvm`框架`react`和`vue`中，因为直接操作dom相对是要耗费更多性能的，所以`react`和`vue`都使用了`virtual dom`来替代处理dom，大致的过程是：按照dom结构生成一份`virtual dom`为`js`对象树状格式。如果数据有更新，需要表现在dom上，那么先把变化表现在`virtual dom`，再通过`diff`算法比对数据更新前后两次`virtual dom`找到差别。最后把仅仅改变的这部分更新到dom上，不用重新渲染整个页面。  

总结：`diff`算法就是找到两个`virtual dom`树结构的不同  

## diff原理 
首先传入新旧`vnode`到`patch`函数开始比较:  

```  
function patch (oldVnode, vnode) {
    if (sameVnode(oldVnode, vnode)) {
        patchVnode(oldVnode, vnode) // 如果两个vnode值得比较，调用patchVnode
    } else {
        const oEl = oldVnode.el // oldVnode.el为对应的真实dom元素
        let parentEle = api.parentNode(oEl)
        createEle(vnode) // 按照vnode结构创建真实dom插入父元素，直接替换oldVnode
        if (parentEle !== null) {
            api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl))
            api.removeChild(parentEle, oldVnode.el)
            oldVnode = null
        }
    }
    return vnode
}
```
首先使用`sameVnode`方法判断两个`vnode`是否值得比较，代码如下
  
```javascript  
function sameVnode(oldVnode, vnode){
	//  key代表当前节点数据的key属性，大概的意思就是div和span为不同标签。p.class1和p.class2为不同标签
	return vnode.key === oldVnode.key && vnode.sel === oldVnode.sel
}
```    
一致就按照上步骤，继续往下处理。执行`patchVnode`方法，如不一致，直接用新元素**直接替换**到旧元素。  

如果两个`vnode`值得比较，调用`patchVnode`:

```javascript
patchVnode (oldVnode, vnode) {
    const el = vnode.el = oldVnode.el
    let i, oldCh = oldVnode.children, ch = vnode.children
    if (oldVnode === vnode) return // 引用相同，就说明相同，直接return
    if (oldVnode.text !== null && vnode.text !== null && oldVnode.text !== vnode.text) {
    	 // 如果有文本内容，比较，如果不同直接设置，用新的替换旧的文本
        api.setTextContent(el, vnode.text)
    }else {
        updateEle(el, vnode, oldVnode)
        if (oldCh && ch && oldCh !== ch) { // 如果都有子节点，且不同，开始比较子节点
            updateChildren(el, oldCh, ch)
        }else if (ch){
        	  // 如果只有新vnode上有子节点，直接新建
            createEle(vnode)
        }else if (oldCh){
        	  // 如果新vnode没子子节，而老vnode有，则删除老节点的子节点
            api.removeChildren(el)
        }
    }
}
```  
 如果都有子节点，且不同，开始比较子节点，调用`updateChildren`函数开始比较子节点:

```javascript
updateChildren (parentElm, oldCh, newCh) { // parentElm：真是dom节点。oldCh：老vnode的所有子节点。newCh：新vnode的所有子节点。
	 // 通过下标，设置新旧vnode的前后两个元素，
    let oldStartIdx = 0, newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
   	 // 设置一些比较过程需要的变量
    let oldKeyToIdx
    let idxInOld
    let elmToMove
    let before
    // 两对节点，分别从头和尾向中间循环靠拢依次比对，如果起始下标大于结尾下标，证明比对结束，作为循环结束条件
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
            if (oldStartVnode == null) {   //对于vnode.key的比较，会把oldVnode = null
                oldStartVnode = oldCh[++oldStartIdx] 
            }else if (oldEndVnode == null) {
                oldEndVnode = oldCh[--oldEndIdx]
            }else if (newStartVnode == null) {
                newStartVnode = newCh[++newStartIdx]
            }else if (newEndVnode == null) {
                newEndVnode = newCh[--newEndIdx]
            }else if (sameVnode(oldStartVnode, newStartVnode)) {
                patchVnode(oldStartVnode, newStartVnode)
                oldStartVnode = oldCh[++oldStartIdx]
                newStartVnode = newCh[++newStartIdx]
            }else if (sameVnode(oldEndVnode, newEndVnode)) {
                patchVnode(oldEndVnode, newEndVnode)
                oldEndVnode = oldCh[--oldEndIdx]
                newEndVnode = newCh[--newEndIdx]
            }else if (sameVnode(oldStartVnode, newEndVnode)) {
                patchVnode(oldStartVnode, newEndVnode)
                api.insertBefore(parentElm, oldStartVnode.el, api.nextSibling(oldEndVnode.el))
                oldStartVnode = oldCh[++oldStartIdx]
                newEndVnode = newCh[--newEndIdx]
            }else if (sameVnode(oldEndVnode, newStartVnode)) {
                patchVnode(oldEndVnode, newStartVnode)
                api.insertBefore(parentElm, oldEndVnode.el, oldStartVnode.el)
                oldEndVnode = oldCh[--oldEndIdx]
                newStartVnode = newCh[++newStartIdx]
            }else {
               // 使用key时的比较
                if (oldKeyToIdx === undefined) {
                    oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx) // 有key生成index表
                }
                idxInOld = oldKeyToIdx[newStartVnode.key]
                if (!idxInOld) {
                    api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                    newStartVnode = newCh[++newStartIdx]
                }
                else {
                    elmToMove = oldCh[idxInOld]
                    if (elmToMove.sel !== newStartVnode.sel) {
                        api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                    }else {
                        patchVnode(elmToMove, newStartVnode)
                        oldCh[idxInOld] = null
                        api.insertBefore(parentElm, elmToMove.el, oldStartVnode.el)
                    }
                    newStartVnode = newCh[++newStartIdx]
                }
            }
        }
        if (oldStartIdx > oldEndIdx) {
            before = newCh[newEndIdx + 1] == null ? null : newCh[newEndIdx + 1].el
            addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx)
        }else if (newStartIdx > newEndIdx) {
            removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
        }
}
```  


总结：    

 - **调用patch**：传入`新旧vnode`，调用`sameVnode`，返回`true`说明值得比较，则调用`patchVnode`，否则直接使用新节点替换旧节点  

 - **调用patchVnode**：更详细的对比新旧vnode，分几种情况：  
 	- 1: 对于新旧vnode的`.el`真实dom属性，并对比，如果===那就证明是同一个，直接return
 	- 2: 如果存在文本节点，直接用新vnode内文本替换旧vnode文本
 	- 3: 判断子节点，如果新vnode有，旧vnode没有，则直接把新vnode的子节点加入到真实dom，相反就删除掉真实dom子节点
 	- 4. 如果都有子节点，且不同，调用updateChildren，详细对比
 - **调用updateChildren**： 从两边向中间收拢循环两个新旧vnode的每个子节点，以此对比，头头，尾尾，头尾，尾头。如果这四种比对调用sameVnode，得到true，证明值得对比，那么就调用第二步骤的`patchVnode`递归继续比，除了这四种情况。如果有key，则会对比key。否则直接替换新元素到真实dom。