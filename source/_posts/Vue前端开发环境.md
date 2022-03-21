---
title: Vue前端开发环境
date: 2019-07-20 18:23:09
tags: 
	- 前端
---

| 技术       | 版本        | 说明                |
| :--------- | ----------- | ------------------- |
| node       | v10.16.0    | node编译环境        |
| npm        | 6.9.0       | npm包管理工具       |
| Vue        | 2.9.6       | 前端框架            |
| Vue-router | 3.0.2       | 前端路由框架        |
| Vuex       | 3.1.0       | vue状态管理组件     |
| Vue-cli    | ————        | Vue脚手架           |
| Element-ui | 2.7.0       | 前端UI框架          |
| Echarts    | 4.2.1       | 数据可视化框架      |
| Uni-app    | ————        | 跨平台前端框架      |
| Mockjs     | 1.0.1-beta3 | 模拟后端数据        |
| Axios      | 0.18.0      | 基于Promise的Http库 |
| Js-cookie  | 2.2.0       | Cookie组件          |
| Jsonlint   | 1.6.3       | Json解析组件        |
| screenfull | 4.2.0       | 全屏组件            |
| Xlsx       | 0.14.1      | Excel表导出组件     |
| Webpack    | ————        | 模板打包器          |

<!--more-->

```shell
 # 安装vue
 $ npm install vue@2.1.6
 # 全局安装 vue-cli
 $ npm install --global vue-cli
 # 创建一个基于 webpack 模板的新项目my-project
 $ vue init webpack my-project
 # 进入项目目录
 $ cd my-project
 # 安装依赖
 $ npm install
 # 运行项目
 $ npm run dev
 # 卸载旧版本，在安装新版本。
 npm uninstall vue-cli -g 
 npm install --global vue-cli
```
