---
title: Vue 与 jQuery Mobile 混用
date: 2018-01-25 11:23:06
categories: 网页开发
tags:
    - Vue
    - jQuery Mobile
---
由于项目需要，在手机端定下的框架是 jQuery Mobile。但由于应用比较大，没有 MVVM 支持会越来越困难。
正好想尝试 Vue ，于是就直接开始 Vue 和 jQuery Mobile 混用的挖坑之路。
<!-- more -->
# 表单元素

对于`input`类型是`text` `tel` `number`这些需要输入的表单元素，没有什么问题。
使用第三方插件提供支持的`date` `time` `datetime`类型元素，也没有问题。
但是对于`radio` `checkbox`这两个元素，实测 jQuery Mobile 和 Vue 无法自动结合。

例如

HTML
``` HTML
<fieldset id="fieldset" data-role="controlgroup">
    <legend>项目状态</legend>
    <label><input v-model="form.project_state" type="radio" name="project_state" v-model="form.project_state" value="在建"/>在建</label>
    <label><input v-model="form.project_state" type="radio" name="project_state" v-model="form.project_state" value="改扩建"/>改扩建</label>
    <label><input v-model="form.project_state" type="radio" name="project_state" v-model="form.project_state" value="生产"/>生产</label>
    <label><input v-model="form.project_state" type="radio" name="project_state" v-model="form.project_state" value="停产"/>停产</label>
</fieldset>
```

JavaScript
``` JavaScript
var content = new Vue({
    el: "fieldset",
    data: {
        form: {
            project_state: ""
        }
    }
})
```

这种情况下，进行单选操作，`content.form.project_state`的值是不更改的，