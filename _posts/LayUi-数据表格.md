title: LayUi 数据表格
author: tr
tags:
  - LayUI
categories:
  - LayUi
date: 2020-04-21 17:50:00
---
### LayUi 数据表格学习

数据绑定，动态数据绑定，监听

<!--more-->


引入官方css和js


#### 数据绑定

**的layui官方，文档给的例子居然不能用。

1. 前端页面中写出table即可

        <table id="pic-mod-table"></table>
        
2. 后端初始化数据

```javascript
 //图模信息展示
    table.render({
        elem: '#pic-mod-table'
        ,height: 312
        ,url: applicationUtil.gatewayURL+'/pssc-graph/graph/graph-name' //数据接口
        ,response: {
            statusCode: 200 //规定成功的状态码，默认：0
            ,countName: 'totalCount' //规定数据总数的字段名称，默认：count
            ,dataName: 'list' //规定数据列表的字段名称，默认：data
        }
        ,limit: 20
        ,parseData: function(res){ //res 即为原始返回的数据
            console.log(res.data.list);
            return {
                "code": res.code, //解析接口状态
                "msg": res.msg, //解析提示文本
                "totalCount": res.data.totalCount, //解析数据长度
                // "count": 19, //解析数据长度
                "list": res.data.list //解析数据列表
            };
        }
        ,totalRow:true
        ,page: true //开启分页
        ,cols: [[ //表头
            {field: 'objId', title: '', fixed: 'left',type: 'numbers'}
            ,{field: 'graphId', title: '图模标识'}
            ,{field: 'graphName', title: '图模名称'}
            ,{field: 'graphType', title: '图模类型'}
            ,{field: 'createTime', title: '创建时间', sort: true}
            ,{field: 'graphVersion', title: '图模版本', sort: true}
        ]]
    });
```

如果不用数据接口：

```javascript
layui.use('table', function(){
    var table = layui.table;

    //图模信息展示
    table.render({
        elem: '#pic-mod-table'
        ,height: 312
        ,url: applicationUtil.gatewayURL+'/pssc-graph/graph/graph-name' //数据接口
        // ,parseData: function(res){ //res 即为原始返回的数据
        //     console.log(res.data.list);
        //     return {
        //         "code": res.code, //解析接口状态
        //         "msg": res.msg, //解析提示文本
        //         "count": res.data.totalCount, //解析数据长度
        //         // "count": 19, //解析数据长度
        //         "data": res.data.list //解析数据列表
        //     };
        // }
        ,page: true //开启分页
        ,cols: [[ //表头
            {field: 'objId', title: '', fixed: 'left',type: 'numbers'}
            ,{field: 'graphId', title: '图模标识'}
            ,{field: 'graphName', title: '图模名称'}
            ,{field: 'graphType', title: '图模类型'}
            ,{field: 'createTime', title: '创建时间', sort: true}
            ,{field: 'graphVersion', title: '图模版本', sort: true}
        ]]
        ,data: [
            {
                "objId": "AC8B1950-A1AE-4455-9165-5C1B53BBB04C",
                "graphId": "灵武变525绒化线单线图.sln",
                "graphName": "灵武变525绒化线单线图.sln",
                "graphType": "黑图",
                "createTime": "2020-04-20 14:05:48",
                "graphVersion": 4,
                "graphCount": 4
            },
            {
                "objId": "36096914-967B-4ADD-BAB4-FF8A87FE69AF",
                "graphId": "上前城变524上丽二回线单线图.sln",
                "graphName": "上前城变524上丽二回线单线图.sln",
                "graphType": "黑图",
                "createTime": "2020-04-15 17:47:28",
                "graphVersion": 46,
                "graphCount": 42
            },
            {
                "objId": "685B5AF2-603A-4823-AAC0-94574159DFD2",
                "graphId": "金沙渠变515金林一回线单线图.sln",
                "graphName": "金沙渠变515金林一回线单线图.sln",
                "graphType": "黑图",
                "createTime": "2020-04-15 17:45:47",
                "graphVersion": 3,
                "graphCount": 3
            }
        ]

    });
 }
```

数据接口json格式：

```json

{
    "code": 200,
    "success": true,
    "data": {
        "totalCount": 40,
        "pageSize": 20,
        "totalPage": 2,
        "currPage": 2,
        "list": [
            {
                "objId": "7CD70357-39B2-4241-BB1B-FA0F8D05D58D",
                "graphId": "10kV燕鸽变528大新线单线图.sln",
                "graphName": "10kV燕鸽变528大新线单线图.sln",
                "graphType": "黑图",
                "createTime": "2020-04-14 16:10:49",
                "graphVersion": 1,
                "graphCount": 1
            },
            {
                "objId": "1128F8B4-947D-4111-9AE0-1B77E05DB0C0",
                "graphId": "10kV兴泾变523芦植线单线图.sln",
                "graphName": "10kV兴泾变523芦植线单线图.sln",
                "graphType": "黑图",
                "createTime": "2020-04-14 16:10:38",
                "graphVersion": 1,
                "graphCount": 1
            }
        ]
    },
    "msg": "操作成功"
}
```

#### 参数说明

`table.render(ele....,,);`

`elem："#id"`: 对应html的标签

`height`: 自设定高度

`url`: 后端数据接口

`method`: 可选post或者get

`where`: post负载的数据

`dataType:"json"`: 传入的数据类型

`page:true`: 开启分页

`limit:20`: 分页数量

` done:function (res,curr,count) {}`: 渲染完成执行的回调函数


#### 表格自适应问题

+ `height` 参数如若填写为 `'full-20'` 可以自动变为自适应，这是官方自带的参数，表示除了表格外留出20像素，`'full-xx'`根据自己页面调节。然而这种方式存在一个缺点，当页面存在两个数据表格上下布局时只能存在一个。

+ `height` ：推荐的自适应。将`height`写为一个页面大小的比例值，然后监听页面拉伸事件，代码如下：

```javascript

layui.use(['form', 'table', 'element', 'upload','restClient'], function () {

    var form = layui.form;
    var allHeight = $('.layui-body').height()-15;
    var offsetPercent = (allHeight-70)/allHeight
    allHeight *= offsetPercent;
    var halfHeight = allHeight/2;
    var lay_off_size = halfHeight
    $('.layui-upload-list').css('height', allHeight*0.994-$("#upload").height());
    
    
    // 自适应问题
    window.onresize = function(){
        allHeight =$('.layui-body').height()-15;
        offsetPercent = (allHeight-70)/allHeight
        allHeight *= offsetPercent;
        halfHeight = allHeight/2;
        $('.layui-upload-list').css('height', allHeight*0.994-$("#upload").height());

        // resize all table
        table.reload('pic-mod-table', {
           height:halfHeight
        });
        table.reload('task-pic-mod-table', {
            height:halfHeight
        });
        table.reload('his-pic-mod-table', {
            height:halfHeight
        });
    }
    
     //监听行单击图模事件
    table.on('row(pic-mod-table)', function (obj) {
        // (obj.tr) //得到当前行元素对象
        // layer.msg(JSON.stringify(obj.data));
        // 渲染历史表格
        his_table_render(obj.data,halfHeight);
    });
    
    // 图模历史版本信息展示
    var his_table_render = function (data,height) {
        // layer.msg(data);
        table.render({
            elem: '#his-pic-mod-table'
            // , height: 312
            , url: applicationUtil.gatewayURL + '/pssc-graph/graph/show-history' //数据接口
            , where: data   // 对象参数
            , page: true //开启分页
            , response: {
                statusCode: 200 //规定成功的状态码，默认：0
                , countName: 'totalCount' //规定数据总数的字段名称，默认：count
                , dataName: 'list' //规定数据列表的字段名称，默认：data
            }
            , height: height
            , contentType: 'application/json'
            , method: 'post'
            , limit: 20
            , align: 'center'
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
            , cols: [[ //表头
                {field: 'id', title: '', fixed: 'left', type: 'numbers',width:'5%',align: 'center'}
                , {field: 'objId', title: '图模ID',width:'20%',align: 'center'}
                , {field: 'graphId', title: '图模标识',width:'20%',align: 'center'}
                , {
                    field: 'graphName', title: '图模名称',width:'20%',align: 'center' ,templet: function (d) {
                        return '<span style="color: #339A96;">' + d.graphName + '</span>'
                        // return '<button class="layui-btn" lay-submit lay-filter="search-graph" onclick="showGraph()">' + d.graphName + '</button>'
                    }
                }
                , {field: 'graphType', title: '图模类型',width:'8%',align: 'center'}
                , {field: 'createTime', title: '创建时间',width:'15%', sort: true,align: 'center'}
                , {field: 'graphVersion', title: '图模版本', width:'13%',sort: true,align: 'center'}
            ]]
        });
    }

}
```

前端layui界面

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