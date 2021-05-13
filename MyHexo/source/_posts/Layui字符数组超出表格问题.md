title: Layui字符数组超出表格问题
author: tr
tags:
  - LayUi
categories:
  - LayUi
date: 2020-06-20 10:24:00
---
### 内容溢出问题

<!--more-->

```html
<style>
        .layui-table-cell {
            min-height: 30px;
            height: initial;
        }
        .layui-table-cell, .layui-table-tool-panel li {
            white-space: initial;
            /*为了让字母和数字也换行*/
            word-break: break-word;
        }
 </style>
```