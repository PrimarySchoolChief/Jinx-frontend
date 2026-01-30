📘 Jinx-frontend 项目目录结构说明文档

Jinx Frontend 是一个基于 HTML + 原生 JavaScript + CSS 的轻量级前端面板，用于端口转发 / NAT / 隧道 / 计费 / 用户管理等功能。

项目采用 多页面 (MPA) + 模块化 JS 结构，而非 React/Vue 单页框架，部署简单，可直接由后端或静态服务器托管。

🌳 项目整体结构
.
├── admin/        管理员页面（HTML）
├── user/         用户页面（HTML）
├── pages/        通用独立页面（登录/注册/404）
├── src/          所有前端 JS + CSS 逻辑源码
├── favicon.ico   网站图标
├── tree.txt      本地生成的目录树

📁 admin（管理员界面页面）

存放 管理员后台 HTML 页面模板

每个 html 页面通常：

引入 /src/global.css

引入 /src/core.js

引入对应 /src/admin/*.js

页面列表
文件	功能说明
announcements.html	公告管理
forward_rules.html	端口转发规则管理
nat_forward_rules.html	NAT 转发规则管理
nodes.html	节点/服务器管理
users.html	用户管理
permissions.html	权限/角色管理
plans.html	套餐/订阅方案管理
invoices.html	账单/订单管理
settings.html	系统设置
terminal.html	Web 终端 / 远程控制台
🔹 设计模式

典型结构：

<script src="/src/core.js"></script>
<script src="/src/admin/users.js"></script>


👉 HTML 负责布局
👉 JS 负责 API + 业务逻辑 + DOM 渲染

📁 user（用户前台页面）

用户自助控制台页面

页面列表
文件	功能
index.html	用户首页 / 控制台
user.html	用户信息/个人中心
announcements.html	公告查看
forward_rules.html	用户转发规则
nat_forward_rules.html	NAT 转发规则
nat_forward_devices.html	NAT 设备管理
tunnel_devices.html	隧道设备管理
invoices.html	我的账单
cart.html	购物车
plan.html	套餐购买
looking-glass.html	网络探测/Looking Glass
documents.html	文档/说明
help.html	帮助中心
📁 pages（独立通用页面）

非 admin / user 的公共页面

文件	说明
login.html	登录页
register.html	注册页
announcement.html	单条公告展示
invoice.html	单个账单详情
404.html	404 错误页
📁 src（核心前端源码）

⭐ 整个项目最重要目录

包含：

JS 业务逻辑

API 调用封装

全局样式

公共组件

🔷 src 根级核心文件
文件	作用
core.js	⭐ 全局核心框架（请求封装 / token / 工具函数 / 初始化）
global.css	全局样式
theme.js	主题/颜色/暗黑模式
foot.js	页脚组件
status.js	系统/服务状态展示
info.js	全局信息展示
logout.js	退出登录逻辑
🔷 用户功能模块
文件	功能
login.js	登录逻辑
register.js	注册逻辑
user.js	用户信息管理
cart.js	购物车
plan.js	套餐购买
invoice.js	单账单
invoices.js	账单列表
announcement.js	单公告
announcements.js	公告列表
forward_rules.js	用户转发规则
nat_forward_rules.js	NAT 规则
nat_devices.js	NAT 设备
tunnel_devices.js	隧道设备
looking-glass.js	网络测试工具
🔷 src/admin（管理员功能模块）

与 admin/*.html 一一对应

文件	对应页面	功能
announcements.js	announcements.html	公告管理
forward_rules.js	forward_rules.html	转发规则管理
nat_forward_rules.js	nat_forward_rules.html	NAT 规则管理
nodes.js	nodes.html	节点管理
users.js	users.html	用户管理
permissions.js	permissions.html	权限控制
plans.js	plans.html	套餐管理
invoices.js	invoices.html	账单管理
settings.js	settings.html	系统设置
🧠 前端架构设计说明
技术特点

本项目采用：

✅ 多页面应用（MPA）
✅ 原生 JavaScript（无框架依赖）
✅ 按功能拆分模块
✅ 后端 API 驱动渲染

页面加载流程
HTML
   ↓
core.js 初始化
   ↓
加载对应模块 JS
   ↓
请求 API
   ↓
渲染 DOM

模块职责划分
core.js

负责：

fetch/axios 封装

token 管理

统一错误处理

全局配置

通用工具函数

👉 相当于「小型前端框架内核」

各功能 js

负责：

页面事件监听

API 调用

表格渲染

表单提交

数据交互