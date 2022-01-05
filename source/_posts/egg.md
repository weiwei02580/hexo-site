---
title: egg
date: 2021-12-22 16:47:14
tags:
---

# Node.js 中进程

1. child_process 子进程
1. cluster 模块
1. master 进程与 cluster 进程通信

## child_process

```js
// child_process.js
const { exec, spawn } = require("child_process");
// exec 创建子进程，且缓存执行结果并返回
exec("cat a.js", (error, stdout, stderr) => {
  if (error) {
    console.log("报错了", error);
    return;
  }
  console.log(`stderr: ${stderr}`);
  console.log(`stdout: ${stdout}`);
});
// spawn 返回stream流
const ls = spawn("ls", ["-a"], { encoding: "utf8" });
ls.stdout.on("data", (data) => {
  console.log(`stdout:${data}`);
});
ls.stderr.on("data", (data) => {
  console.log(`stderr:${data}`);
});
ls.on("close", (code) => {
  console.log(`close:${code}`);
});
```

## cluster

```js
const cluster = require("cluster");
const http = require("http");
const os = require("os");
const cpus = os.cpus().length;
// console.log(cpus);

if (cluster.isMaster) {
  console.log(`主进程 ${process.pid} 正在进行`);

  // 衍生工作进程
  for (let index = 0; index < cpus; index++) {
    cluster.fork();
  }
} else {
  // 工作进程可以共享任何tcp链接
  // 这里共享http服务器
  http
    .createServer((req, res) => {
      res.writeHead(200, { "Content-type": "text/html;charset=utf-8" });
      res.write("你好");
      res.end();
    })
    .listen(8000);

  console.log(`工作进程 ${process.pid} 已经启动`);
}
```

## 进程通信

```js
// child.js 子进程
console.log(`子进程${process.pid}`);
process.on("message", (msg) => {
  console.log(`来自master：${msg}`);
});
process.send("这是子进程");
// master.js 主进程
const { fork } = require("child_process");
const child = fork("./child.js");
child.on("message", (msg) => {
  console.log(`来自子进程:${msg}`);
});

child.send("来自master");
```

# Egg

## 插件

`egg-validate` ctx.request.body 数据校验

```js
// {app_root}/config/plugin.js
exports.validate = {
  enable: true,
  package: "egg-validate",
};
// 使用
const rlue = {
  name: { type: "string" },
  age: { type: "number" },
};
ctx.validate(rlue);
```

`egg-view-ejs` 模版引擎

```js
// {app_root}/config/plugin.js
exports.ejs = {
  enable: true,
  package: "egg-view-ejs",
};
// {app_root}/config/config.default.js
const path = require("path");
config.view = {
  mapping: {
    ".html": "ejs",
  },
  root: [path.join(appInfo.baseDir, "app/html")].join(","),
};
config.ejs = {
  delimiter: "$", // 全局修改分隔符
};
// 使用
// view/*.html
await ctx.render(
  "user.html",
  {
    id: 100,
    name: "admin",
  },
  {
    delimiter: "$", // 单个修改ejs引擎
  }
);
```

```html
<!-- user.html -->
<!-- 引入html片段 -->
<% include user-header.html %>
<div>
  id: <%= id %>
  <br />
  注释：<%# 注释 %>
  <br />
  name:<%= name%>
</div>
<ul>
  <% for(var i=0;i < lists.lemgth; i++){ %>
  <li><%= lists[i] %></li>
  <% } %>
</ul>
```

`egg-static` 静态文件

```js
// {app_root}/config/config.default.js
config.static = {
  perfix: "/assets/",
};
```

`cookies`

```js
// 1. 设置中文cookie
ctx.cookies.set("cookie-name", "cookie-value", {
  maxAge: 1000 * 60 * 10,
  httpOnly: true, // 仅服务端操作
  encrypt: true, // 加密可保存中文的value
});
ctx.cookies.set("cookie-name", null);
ctx.cookies.get("cookie-name", {
  encrypt: true,
});
// 2. base64转换一下
encode(str){
  return new Buffer(str).toString('base64')
}
decode(str){
  return new Buffer(str,'base64').toString()
}
```

`session` 基于 cookie

```js
// {app_root}/config/config.default.js
config.session = {
  key: "CUSTOM_SESS",
  httpOnly: true,
  maxAge: 1000 * 50,
  renew: true, // 自动刷新
};
// 赋值
ctx.session.user = "value";
// 读取
ctx.session.user;
// 删除
ctx.session.user = null;

// app.js app拓展保存到内存中
module.exports = (app) => {
  const store = {};
  app.sessionStore = {
    async get(key) {
      console.log("--store--", store);
      return store[key];
    },
    async set(key, value) {
      store[key] = value;
    },
    async destroy(key) {
      store[key] = null;
    },
  };
};
```

`httpClient` 调用其他接口

```js
ctx.curl("http://localhost:3000/index", { dataType: "json" });
```

## 中间件

```js
// {app_root}/middleware/m1.js
module.exports = (options) => {
  return async (ctx, next) => {
    console.log("m1 start");
    await next();
    console.log("m1 end");
  };
};
// {app_root}/config/config.default.js
config.middleware = ["m1"];
```

## 扩展方式

| 扩展点      | 说明         | this 指向         | 使用方式          |
| ----------- | ------------ | ----------------- | ----------------- |
| application | 全局应用对象 | app 对象          | this.app          |
| context     | 上下文       | ctx 对象          | this.ctx          |
| request     | 请求         | ctx.request 对象  | this.ctx.request  |
| response    | 响应         | ctx.response 对象 | this.ctx.response |
| helper      | 帮助函数     | ctx.helper 对象   | this.ctx.helper   |

```js
// {app_root}/extend/application.js
const path = require("path");
module.exports = {
  // 方法扩展
  package(key) {
    const pack = getPack();
    return key ? pack[key] : pack;
  },
  // 属性扩展
  get allPackage() {
    return getPack();
  },
};
function getPack() {
  const filePath = path.join(process.cwd(), "package.json");
  const pack = require(filePath);
}
// 使用
const { ctx, app } = this;
app.package();
app.allPackage;

// {app_root}/extend/context.js
module.exports = {
  params(key) {
    const method = this.request.method;
    if (method === "GET") {
      return key ? this.query[key] : this.query;
    } else {
      return key ? this.request.body[key] : this.request.body;
    }
  },
};
// 使用
const { ctx } = this;
ctx.params();

// {app_root}/extend/request.js
module.exports = {
  get token() {
    console.log("header", this.header);
    return this.get("token");
  },
};
// 使用
const { ctx } = this;
ctx.request.token;

// {app_root}/extend/response.js
module.exports = {
  set token(token) {
    this.set("token", token);
  },
};
// 使用
const { ctx } = this;
ctx.response.token = "abc123";

// {app_root}/extend/helper.js
module.exports = {
  base64Encode(str = "") {
    return new Buffer(str).toString("base64");
  },
};
const { ctx } = this;
ctx.helper.base64Encode("string");
```

## Egg.js 中的插件

存放位置`{app_root}/lib/plugin`

```js
// egg-auth/app
// package.json
// {
//   "name": "egg-auth",
//   "eggPlugin": {
//     "name": "auth"
//   }
// }
// /middleware/auth.js
module.exports = (options) => {
  return async (ctx, next) => {
    const url = ctx.request.url;
    console.log("url", url);
    const user = ctx.session.user;
    if (!user && !options.exclude.includes(url.split("?")[0])) {
      ctx.body = {
        status: 1001,
        msg: "用户未登录",
      };
    } else {
      await next();
    }
  };
};
// {app_root}/config/plugin.js
const path = require("path");
exports.auth = {
  enable: true,
  path: path.join(__dirname, "../lib/plugin/egg-auth"),
};
// {app_root}/app.js
app.config.coreMiddleware.push("auth");
// {app_root}/config/config.default.js
config.auth = {
  exclude: ["/home", "/user", "/login", "/logout"],
};
```

## 定时器

```js
// {app_root}/schedule/get_info.js
const Subscription = require("egg").Subscription;
class getInfo extends Subscription {
  static get schedule() {
    return {
      interval: 1000 * 3,
      cron: "*/3 * * * * *", // 秒 分钟 小时 月中天 月份 周中天
      // 每隔3秒执行
      type: "worker", // 'all'
    };
  }

  async subscribe() {
    const info = this.ctx.info;
    console.log(Date.now(), info);
  }
}
module.exports = getInfo;
```

## egg-mysql

`npm i egg-mysql` 简单查询实际上作用不大

```js
// {app_root}/config/plugin.js
exports.mysql = {
  enable: true,
  package: "egg-mysql",
};
```

```js
// {app_root}/config/config.default.js
config.mysql = {
  app: true, // 挂载在app上
  agent: false,
  client: {
    host: "",
    port: "",
    user: "",
    password: "",
    database: "",
  },
};
```

```js
// 在service的使用
class UserService extends Service {
  async lists() {
    try {
      const { app } = this;
      const res = await app.mysql.select("user"); // 查询所有
      return res;
    } catch (err) {
      console.log(err);
      return null;
    }
  }

  async detail(id) {
    try {
      const { app } = this;
      const res = await app.mysql.get("user", { id }); // 单条查询
      return res;
    } catch (err) {
      console.log(err);
      return null;
    }
  }

  async add(params) {
    try {
      const { app } = this;
      const res = await app.mysql.insert("user", params); // 插入数据
      return res;
    } catch (err) {
      console.log(err);
      return null;
    }
  }

  async edit(params) {
    try {
      const { app } = this;
      const res = await app.mysql.update("user", params); // 插入数据
      return res;
    } catch (err) {
      console.log(err);
      return null;
    }
  }

  async delete(id) {
    try {
      const { app } = this;
      const res = await app.mysql.del("user", { id }); // 插入数据
      return res;
    } catch (err) {
      console.log(err);
      return null;
    }
  }
}
// 在controller
const res = await app.service.user.lists();
ctx.body = res;
```

## egg-sequelize

`npm i egg-sequelize mysql2`

```js
// {app_root}/config/plugin.js
exports.sequlize = {
  enable: true,
  package: "egg-sequlize",
};
```

```js
// {app_root}/config/config.default.js
config.squelize = {
  dialect: "mysql",
  host: "127.0.0.1",
  port: "3306",
  user: "root",
  password: "123456",
  database: "egg-mysql",
  // 定义
  define: {
    timestamps: false,
    freezeTableName: true, // 冻结表名称
  },
};
```

```js
// {app_root}/app/model/user.js
module.exports = (app) => {
  const { STRING, INTERGER } = app.Sequlize;
  const User = app.model.define("user", {
    id: { type: INTERGER, primaryKey: true, autoIncrement: true },
    name: STRING(20),
    pwd: STRING(50),
  });
  return User;
};
// controller中使用
const res = await ctx.model.User.findAll();
const res = await ctx.model.User.findAll({
  where: {
    id: 1,
  },
  limit: 1, // 分页条数
  offset: 0, // 偏移量
});
// 查询单条数据
const res = await ctx.model.User.findByPk(ctx.query.id);
// 插入单条数据
const res = await ctx.model.User.create(ctx.request.body);
// 编辑数据 1. 数据是否存在 2. 更新数据
const user = await ctx.model.User.findByPk(ctx.request.body.id);
if (!user) {
  ctx.body = {
    code: 404,
    msg: "id不存在",
  };
  return;
}
// update
const res = user.update(ctx.request.body);
// delete
const res = user.destroy(ctx.request.body.id);
```
