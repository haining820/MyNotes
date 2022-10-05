---
title: Vue
date: 2022-10-05
tags: [Vue]
toc: true
---

# 1、Vue

## 1.1、Vue 指令

- v-text：设置标签的文本值，`<h2v-text="message"></h2>`，简写：`<h2> {{message}} </h2>`
- v-html：后面可以添加 html 语句，内容中有 html 结构会被解析为标签
  - 对比 v-text：v-text 指令无论内容是什么，只会解析为文本；
  -  解析文本使用 v-text，需要解析 html 结构使用 v-html;

<!--more-->

- v-on：可简写为 `@`
  - 后面可接 `事件名+方法`，方法可以添加入参，在调用函数时传入参数；
  - 方法内部可以通过 this 关键字访问定义在 data 中的数据；

```bash
v-on:click="function"		# 鼠标单击		# 简写：@click="function"
v-on:dbclick="function"		# 鼠标双击
v-on:monseenter="function"	# 鼠标移入元素内（本身）		  # 对应mouseleave
v-on:mouseover="function"	# 鼠标移入元素内（本身及其子元素） # 对应mouseout
v-on:enter="function"		# 回车
```

- v-show：切换元素的显示状态，**原理是设置 display 为 true/false，对性能消耗小一些；**`<img src="link" v-show="true">`（后面可以跟true/false，data 中的数据，表达式，最终都会被解析为布尔值）

- v-if：切换元素的显示状态，**原理是直接移除 dom 元素，对性能消耗比较大；**

  v-show 和 v-if 的区别：看性能，频繁切换的元素用 v-show，反之用 v-if。

- v-bind：为元素绑定属性（src、title、class...）
  - 简写的话可以直接省略 v-bind，只保留前面的冒号

```bash
<img v-bind:src="imgSrc" >
<img :title="imgtitle+’!!’">
<img :class="isActive?'active':‘’”>	# 三元表达式
<img :class="{active:isActive}">	# 动态增删class建议使用对象的方式
```

- v-for：根据数据生成列表结构，data 中定义：
  - 普通数组：`arr:[1,2,3,4,5]`
  - 对象数组：`objArr:[{ name: "jack" },{ name: "rose" }]`

```bash
<ul>
    <li v-for="(item,index) in arr" :title="item">
    	{{ index }}{{ item }}
    </li>
    <li v-for="(item,index) in objArr">
    	{{ item.name }}
    </li>
</ul>
```

- v-model：获取和设置表单元素的值（双向数据绑定）
  - v-model 指令的作用是便捷的设置和获取表单元素的值；
  - 绑定的数据会和表单元素值相关联（一边变另一边也会同时变）

# 2、Vue 脚手架

> https://cli.vuejs.org/zh/guide/

Vue CLI 脚手架：基于 vue.js 进行快速开发的完整系统（cli：command-line interface，命令行界面）。

- v2 版本：vue-cli

- v3 版本：@vue/cli

![image-20221005173751565](https://haining820-bucket.oss-cn-beijing.aliyuncs.com/typora_img/vue%E8%84%9A%E6%89%8B%E6%9E%B6.png)

## 2.1、Vue CLI 安装

- 安装 node，node 中自带 npm；

- npm：node package manager，nodejs 包管理工具，前端主流技术都可以用 npm 进行管理（类比 maven）。

更换 npm 中心仓库：配置淘宝镜像

```bash
npm config set registry https://registry.npm.taobao.org
npm config get registry	# 验证
```

配置 npm 依赖的下载位置

```bash
npm config ls	# 查看默认位置， 验证
npm config set cache "D:\Environment\nodejs\node-repo\npm-cache"
npm config set prefix "D:\Environment\nodejs\node-repo\npm-global"
```



