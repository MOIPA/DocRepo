title: layui自适应
author: tr
tags:
  - LayUi
categories:
  - LayUi
date: 2020-05-13 16:07:00
---
### layui自适应

layui页面的自适应问题

<!--more-->

#### 自适应页面布局

官方提供了`layui-body`但是使用后会自动向右便宜，这是官方为了适配本身的侧边栏页面，需要手动将left改为0px。

`layui-card`会让页面底部变为白色，区别于body的灰色。

##### 宽度自适应:设定全局布局`layui-fluid`,内部使用`layui-row`，`layui-row`内部使用`<div class="layui-col-md8">` 8表示12个中占8位长度。


页面布局如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>图模导入</title>
    <meta charset="utf-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <link rel="stylesheet" href="../../assets/libs/layui/css/layui.css"/>
    <link rel="stylesheet" href="../../assets/module/admin.css"/>
    <link rel="stylesheet" href="../../assets/css/theme.css"/>
</head>
<body class="layui-body" style="left: 0px;">
<div class="layui-card" style="margin:10px;margin-top:0px;">
    <div class="layui-card-body" style="padding-left:10px;padding-top:5px;padding-bottom: 0px;padding-right: 10px;">
        <div class="layui-fluid" style="padding:0px;margin: 0px;">
            <div class="layui-row">
                <div class="layui-col-md8">
                    <div class="layui-row">
                        <form class="layui-form" style="margin-top: 2px;" action="">
                            <div class="layui-form-item">
                                <div class="layui-inline">
                                    <div class="layui-inline">
                                        <input id="search-text" type="text" name="search" placeholder="图像名称"
                                               autocomplete="off" class="layui-input" style="margin-top: 5px;">
                                    </div>
                                    <button class="layui-btn" lay-submit lay-filter="search-graph">查询
                                    </button>
                                    <button type="reset" class="layui-btn layui-btn-primary">重置
                                    </button>
                                </div>
                            </div>
                        </form>
                    </div>
                    <div class="layui-row" style="margin-top: -14px;">
                        <table id="pic-mod-table" lay-filter="pic-mod-table"></table>
                    </div>
                    <div class="layui-row"style="margin-top: 11px;">
                        <table id="his-pic-mod-table" lay-filter="his-pic-mod-table"></table>
                    </div>
                </div>
                <div class="layui-col-md4">
                    <div class="layui-form-item">
                        <!--                <label class="layui-form-label">添加附件：</label>-->
                        <input type="hidden" id="filePath" name="filePath"/>
                        <div class="layui-input-inline shangc_txt" style="margin-top: 5px;padding-left: 10px;width: 100%">
                            <div class="layui-upload">
                                <button type="button" class="layui-btn" id="upload-svg-btn">
                                    <i class="layui-icon">&#xe67c;</i>选择文件
                                </button>
                                <div class="layui-upload-list" style="border: 1px solid #D9D9D9;width: 98%">
                                    <table class="layui-table" style="padding:0px;margin: 0px;overflow-y: auto;margin-top: 2px;">
                                        <thead>
                                        <tr>
                                            <th>文件名</th>
                                            <th>文件大小</th>
                                            <th>文件状态</th>
                                            <th>文件操作</th>
                                        </tr>
                                        </thead>
                                        <tbody id="upload-list"></tbody>
                                    </table>
                                </div>
                                <button type="button" class="layui-btn layui-btn-disabled" id="upload">上传</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>


<!-- js部分 -->
<script type="text/javascript" src="../../assets/libs/layui/layui.js"></script>
<script type="text/javascript" src="../../assets/libs/jquery/jquery-3.2.1.min.js"></script>
<script type="text/javascript" src="../../assets/js/common.js"></script>
<script type="text/javascript" src="../../assets/js/applicationUtil.js"></script>

<script type="text/javascript" src="js/graph.js"></script>
</body>
</html>
```

##### 高度自适应

###### 数据表格

如下代码，可以直接在方法渲染中，官方做法：数据表格参数：`height:'full-20'`，20表示上下预留的像素，这个方法只能适用于页面单行数据表格的情况，如果两行同样数据表格，下面的表格会被顶出。

```javascript
  table.render({
        elem: '#pic-mod-table'
        , id: 'pic-mod-table'
        , url: applicationUtil.gatewayURL + '/pssc-graph/graph/graph-name' //数据接口
        , response: {
            statusCode: 200 //规定成功的状态码，默认：0
            , countName: 'totalCount' //规定数据总数的字段名称，默认：count
            , dataName: 'list' //规定数据列表的字段名称，默认：data
        }
        , limit: 20
        , parseData: function (res) { //res 即为原始返回的数据
            // console.log(res.data.list);
            return {
                "code": res.code, //解析接口状态
                "msg": res.msg, //解析提示文本
                "totalCount": res.data.totalCount, //解析数据长度
                // "count": 19, //解析数据长度
                "list": res.data.list //解析数据列表
            };
        }
        , height: 'full-20'
        , totalRow: false
        , page: true //开启分页
        , cols: [[ //表头
            {field: 'objId', title: '', fixed: 'left', type: 'numbers', align: 'center'}
            , {field: 'graphId', title: '图模标识', width: '28%', align: 'center'}
            , {field: 'graphName', title: '图模名称', width: '28%', align: 'center'}
            , {field: 'graphType', title: '图模类型', width: '11%', align: 'center'}
            , {field: 'createTime', title: '创建时间', width: '17%', sort: true, align: 'center'}
            , {field: 'graphVersion', title: '图模版本', width: '14%', sort: true, align: 'center'}
        ]]
    });

```
###### js监听resize事件

手动获取页面高度，并根据比例判断页面占比和高度

```javascript

    var form = layui.form;
    var allHeight = $('.layui-body').height() - 15;
    var offsetPercent = (allHeight - 70) / allHeight
    allHeight *= offsetPercent;
    var halfHeight = allHeight / 2;
    var lay_off_size = halfHeight
    $('.layui-upload-list').css('height', allHeight * 0.994 - $("#upload").height());

    // 自适应问题
    window.onresize = function () {
        allHeight = $('.layui-body').height() - 15;
        offsetPercent = (allHeight - 70) / allHeight
        allHeight *= offsetPercent;
        halfHeight = allHeight / 2;
        $('.layui-upload-list').css('height', allHeight * 0.994 - $("#upload").height());

        // resize all table
        table.reload('pic-mod-table', {
            height: halfHeight
        });
        table.reload('task-pic-mod-table', {
            height: halfHeight
        });
        table.reload('his-pic-mod-table', {
            height: halfHeight
        });
    }

```
