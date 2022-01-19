title: Layui事件监听
author: tr
tags:
  - LayUi
categories: []
date: 2020-06-13 15:57:00
---
### Layui事件监听

<!--more-->

1. 官方提供了elemen.on和filter操作，但是公司统一监听为如下形式。


```html
 <div class="layui-form-item">
            <button class="layui-btn" data-method="functionConfig" title="功能配置" style="margin-left: 10px;">功能配置</button>
 </div>
```

```js
layui.use(['layer', 'form', 'admin'], function () {
..................
 $(document).on('click','.btn,.layui-btn',function () {
            var othis = $(this), method = othis.data('method');
            active[method] ? active[method].call(this, othis) : '';
 });
}

var active={
     functionConfig: function () {
     }
};
```