---
title: learning-status
date: 2021-12-16 16:04:00
tags:
  - learn
categories:
  - [统计]
---

# JavaScript

<!--more-->

## Vue

- Vue3+ElementPlus+Koa2 全栈开发后台系统 -> 7-9
- Vue2.5 实战微信读书 媲美原生 App 的企业级 web 书城
- 全网稀缺 Vue 2.0 高级实战 独立开发专属音乐 WebAPP

## 开发一款 npm

1. commander

```javascript
const { program } = require("commander");
program.version("0.0.1");

program.option("-n ", "输出名称");
program.option("-t --type <type> ", "输出名称");

program.parse(process.argv);

const options = program.opts();
console.log("opts=>", options);

// figlet // console 大文字
// chalk // console 彩色文字
// inquirer //console 选择交互
// shelljs //
// ora //
// download-git-repo //
```

---

## TypeScript

- 系统入门到项目实战 趁早学习提高职场竞争力 -> 4-8

---

## React

> 生态：redux dva MobX redux-thunk umi

```javascript
// redux-thunk 可以返回一个fn
// redux-saga  拦截同名的action
```

- React 全家桶+AntD 单车后台管理系统开发
- ReactHooks 重构去哪网购票
- 用 React.js+Egg.js 造轮子 全栈开发旅游电商应用
- React16+React-Router4 从零打造企业级电商后台管理系统
- React16.8+Next.js+Koa2 开发 Github 全栈项目
- Electron+React+七牛云 实战跨平台桌面应用

---

## React-Native

- React Native+Redux 打造高质量上线 App

```javascript
/**
 * react-navigation 6.x
 * 按照官网安装三个插件，ios  npx pod-install ios
 * android 修改 MainActivity.java
 * 此过程中 @react-navigation 缺失的依赖 需要重新手动安装
 * */
// ctrl + M 调出菜单
```

- ReactNative + TypeScript 仿喜马拉雅开发 App -> 粗略接触
- 新版 React Native+Redux 打造高质量上线 App -> 重点在调用 native modules、code push、屏幕适配

---

# Flutter

- flutter 移动电商实战
- Flutter 从入门到进阶实战携程网 App ->

> 笔记

1. 安卓的环境变量

```properties
# local.properties
org.gradle.java.home=C:\\Program Files\\Java\\jdk1.8.0_291
sdk.dir=C:\\Users\\Administrator\\AppData\\Local\\Android\\Sdk
ndk.dir=C:\\Users\\Administrator\\AppData\\Local\\Android\\Sdk\\ndk-bundle
```

2. JAVA_HOME 环境变量指向 `bin` 上一级
3. DIO 访问 地址使用 ip/域名，不可使用 localhost

# 微前端

- 从零打造微前端框架：实战“汽车资讯平台”项目

---

# 区块链

> 对称加密

- 收发用同一个私钥

> 非对称加密

1. 通过私钥获取公钥
2. 通过私钥加密 -> 生成签名
3. 信息、公钥、签名 -> 验证

`vorpal` 命令行工具  
`cli-table` 命令行输出表格美化  
`dgram` 处理 udp

## `truffle` sol 合约开发工具

`truffle compile` 编译  
`truffle migrate` 部署  
`truffle migrate --reset --all` 重新部署
`truffle test` 测试

进度：04-13

> 已学完

- 数据可视化
- Vue 全家桶+SSR+Koa2 全栈开发美团网
- React 16+Redux+React Router 4 Node.Js 全栈开发招聘 App 项目实战视频
- Taro 小程序

# 感兴趣

- 量化交易
- three.js
