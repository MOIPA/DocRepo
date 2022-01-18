title: Layui时间判断和前端转义
author: tr
tags:
  - LayUi
categories:
  - LayUi
date: 2020-06-09 11:23:00
---
#### Layui 前端防止sql注入

<!--more-->

```js
 a.cmsId1 = a.cmsId1.replace(/null|select|update|delete|exec|count|'|"|=|;|>|<|%|-/g,function(d){console.log(d);return '\\\\'+d});

```

#### 时间判断

```html
 <label class="layui-form-mid">现场送电时间</label>
                                            <div class="layui-input-inline">
                                                <input class="Wdate layui-input planOffStartTime-style pssc-color"
                                                       id="sceneStartDate"
                                                       type="text" onclick="WdatePicker({dateFmt:'yyyy-MM-dd HH:mm:ss',startDate:'%y-%M-%d 00:00:00',
                                                   isShowToday:false,onpicked: active.getSendStartTime})" data-form="formed" autocomplete="off">
                                            </div>
```

```js
getTOutageStartTime() {
        if ($("#stopEndDate").val() == "") {
            return;
        }
        if ($("#stopEndDate").val() != "") {
            if (new Date($("#outageStartTime").val()) >= new Date($("#stopEndDate").val())) {
                layer.msg('停电结束时间应大于停电开始时间！', {
                    icon: 2,
                    time: 2000
                }, function () {
                    $("#outageStartTime").val("");
                });
                return;
            }
        }
    },
    getTOutageEndTime() {
        if ($("#outageStartTime").val() == "") {
            return;
        }
        if ($("#outageStartTime").val() != "") {
            if (new Date($("#outageStartTime").val()) >= new Date($("#stopEndDate").val())) {
                layer.msg('停结束时间应大于停电开始时间！', {
                    icon: 2,
                    time: 2000
                }, function () {
                    $("#stopEndDate").val("");
                });
                return;
            }
        }
    },
```