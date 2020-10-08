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