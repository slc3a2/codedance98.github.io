---
title: 使用css实现圆环加载
date: 2020/10/04 09:44:03
tags: [css]
---

### html  

```html
<div class="round"></div>
```
### css
使用`background-image`渐变属性和动画状态的`伪元素遮盖`来实现：    

```css
.round {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background-color: #eee;
  background-image: linear-gradient(to right, transparent 50%, blue 0);
  position: relative;
}
.round::before {
  width: 50%;
  height: 100%;
  content: '';
  position: absolute;
  top: 0; 
  left: 50%;
  border-radius: 0 100% 100% 0 / 50%;
  background-color: inherit;
  transform-origin: left;
  animation: spin 5s linear infinite, dye 10s step-end infinite;
}
@keyframes spin {
  to { transform: rotate(.5turn); }
}
@keyframes dye {
  50% { background: blue; }
}
```
#### 效果

![demo.gif](./使用css实现环状进度/E8CM6WZ1hRv7piy.gif)