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
    <label><input v-model="form.project_state" type="radio" name="project_state" value="在建"/>在建</label>
    <label><input v-model="form.project_state" type="radio" name="project_state" value="改扩建"/>改扩建</label>
    <label><input v-model="form.project_state" type="radio" name="project_state" value="生产"/>生产</label>
    <label><input v-model="form.project_state" type="radio" name="project_state" value="停产"/>停产</label>
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
有可能是 jQuery Mobile 在实现的时候，阻断了事件的传播，Vue 无法获取到真正的值。
如果希望更改 Vue 对象中的值，那么需要给每个按钮元素加上`onclick`事件响应函数，替换`v-model`绑定。

HTML
``` HTML
<fieldset id="fieldset" data-role="controlgroup">
    <legend>项目状态</legend>
    <label><input v-model="form.project_state" type="radio" name="project_state" onclick="content.form.project_state= '在建'" value="在建"/>在建</label>
    <label><input v-model="form.project_state" type="radio" name="project_state" onclick="content.form.project_state= '改扩建'" value="改扩建"/>改扩建</label>
    <label><input v-model="form.project_state" type="radio" name="project_state" onclick="content.form.project_state= '生产'" value="生产"/>生产</label>
    <label><input v-model="form.project_state" type="radio" name="project_state" onclick="content.form.project_state= '停产'" value="停产"/>停产</label>
</fieldset>
```

现在这样就可以更改`content`中的值了。

但是有趣的是，如果你使用了`v-model`绑定元素到某个对象（例如这里的`project_state`），
那么当你在初始化 Vue 对象的时候，对该对象（`project_state`）赋值，
绑定到它的单选按钮组会自动根据该对象的值进行初始化。
比如你设置了`project_state`的初始值为“生产”，
那么“生产”对应的第三个单选按钮会在文档初始化的过程中被选中。

# 动态加载 DOM 元素

