# t1
# 精明消费AI平台 V1.0 软件规格说明书 SRS
文档版本：V1.0
文档状态：正式版
适用对象：开发AI、程序员、产品
项目目标：面向个人用户/私有化B端租户，提供电商商品抓取、AI凑单优惠计算、价格监控、消费记账复盘、社区分享、跨端数据同步一体化消费省钱平台，附带Chrome浏览器插件。

## 1. 引言
### 1.1 目的
本文档规定精明消费AI平台全部功能性需求、非功能需求、业务约束、用户场景、输入输出、权限模型，作为后续设计、编码、测试、交付的基准，AI开发必须严格遵守本规格，不得擅自新增、删减需求。
### 1.2 范围
包含：Web管理端、Web用户端、Chrome扩展插件、后端服务、MySQL数据库、Redis缓存、WebSocket实时推送、License租户私有化管控、增量多端同步模块。
不包含：原生APP、小程序、第三方电商爬虫付费代理池（爬虫由插件前端页面解析完成，后端不主动爬电商）
### 1.3 术语定义
- 租户Tenant：私有化B端客户，一套实例可分给多个租户隔离数据
- License：私有化授权证书，控制租户最大用户数量、有效期
- 凑单方案：用户录入多商品+优惠规则，AI求解最优下单组合，计算实付、分摊单价、风险标签
- 水位WaterTime：跨端增量同步时间戳，用于增量拉取变更
- 监控任务：绑定商品+目标降价阈值，价格变动触发站内消息+浏览器通知
### 1.4 参考文档
SDD软件设计文档、开发指南文档、框架文档、OpenAPI接口文档

## 2. 总体描述
### 2.1 产品愿景
帮助用户理性消费，自动计算满减/津贴/券最优凑单方案，长期追踪商品价格，记录消费，复盘性价比，社区互通捡漏信息，支持私有化部署给服务商。
### 2.2 用户角色
1. 超级管理员：管理租户、全局License、平台配置
2. 租户管理员：管理本租户下用户、帖子审核、租户License
3. 普通登录用户：凑单、价格监控、记账、发帖评论、跨端同步
### 2.3 运行环境
- 后端：Java SpringBoot 21
- 数据库：MySQL8.0 + Redis7
- 前端：Vue3 + Element Plus
- 插件：Chrome Extension Manifest V3
- 部署：Docker Compose 一键部署私有化
### 2.4 业务约束
1. 后端禁止主动爬取电商网站，商品信息全部由Chrome插件content script在用户浏览器内解析抓取上传
2. 用户数据按租户ID严格隔离，跨租户不可访问
3. 同步机制采用增量推拉模式，支持冲突检测
4. 所有接口统一JWT鉴权
5. License校验定时后台执行，授权失效后租户新用户禁止登录

## 3. 功能需求（FR）
### 3.1 身份认证模块
FR001：账号密码登录、手机号验证码登录、登出、获取用户信息
FR002：JWT令牌，过期自动失效，接口统一鉴权
### 3.2 租户&License管理模块（B端私有化）
FR003：超管新建/查询租户，配置租户最大用户上限
FR004：租户管理员管理租户内用户：新增、启用禁用、删除
FR005：License远程校验，查询授权信息，到期限制功能
### 3.3 AI凑单计算器模块（核心）
FR006：录入商品列表（名称、链接、平台、单价、数量）
FR007：录入优惠规则：满减、品类券、店铺券、跨店津贴、折扣
FR008：AI枚举可行组合，输出最优凑单方案、分摊单价、总节省金额
FR009：标记风险标签：先涨后降风险、限时限量风险、售后风险
FR010：保存方案、查询历史方案、查看详情、删除方案
### 3.4 价格监控模块
FR011：插件抓取商品一键添加监控，设置目标提醒价
FR012：记录历史价格，绘制价格曲线
FR013：价格低于目标价触发提醒：站内消息 + Chrome通知
FR014：修改监控参数、启停提醒、删除监控
### 3.5 消费记账&复盘模块
FR015：新增账单，关联凑单方案，录入预期使用周期、标签、备注
FR016：账单列表筛选（时间、标签、分页）
FR017：月度消费复盘报表，统计支出、单品摊薄成本
FR018：账单编辑、删除
### 3.6 社区模块
FR019：用户发布捡漏帖子，支持图文、关联商品
FR020：帖子列表、详情、点赞收藏
FR021：评论、回复评论
FR022：租户管理员帖子审核，控制是否公开
### 3.7 跨端增量同步模块（核心）
FR023：按表名+水位时间增量拉取云端变更数据
FR024：本地变更推送到云端，携带本地版本号，服务端检测冲突
FR025：同步日志记录，查看同步历史
### 3.8 用户成长与消息模块
FR026：积分流水记录，等级权益展示，操作发放积分
FR027：站内消息列表，标记已读、一键全部已读
### 3.9 Chrome插件能力
FR028：在淘宝/京东/拼多多页面一键抓取商品信息
FR029：一键加入监控、一键跳转平台凑单页面
FR030：接收降价浏览器通知

## 4. 非功能需求（NFR）
NFR001 性能：单用户接口响应 < 800ms；同步拉取接口支持分页批量
NFR002 并发：私有化单租户支持200在线用户
NFR003 兼容性：插件支持Chrome 100+；前端支持现代浏览器
NFR004 数据安全：密码BCrypt加密，传输HTTPS；租户数据行隔离
NFR005 可部署性：Docker Compose一键启动，配置文件集中管理
NFR006 可扩展性：后续可新增平台、新增优惠算法，模块解耦
NFR007 可用性：核心模块（凑单、监控、记账）离线本地存储，联网同步

## 5. 接口约束
全部RESTful接口统一前缀 /api/v1，统一返回code/msg/data结构；WebSocket用于实时推送降价、同步变更、消息通知。

## 6. 交付物需求
1. 可运行后端jar
2. Vue前端dist包
3. Chrome扩展源码
4. 初始化SQL脚本
5. docker-compose.yml及配套Dockerfile、nginx配置
6. OpenAPI接口文档
# 精明消费AI平台 V1.0 软件设计文档 SDD
版本：V1.0
目标：指导AI进行架构设计、模块划分、表设计、接口设计、算法设计、同步逻辑设计

## 1. 架构总览
整体采用前后端分离 + 插件端三端架构
三层架构：接入层 → 业务服务层 → 数据层
- 接入层：Vue前端页面 + Chrome Extension + Nginx反向代理 + WebSocket
- 业务层：SpringBoot微内核单体（私有化场景优先单体，降低部署复杂度），按包拆分模块
- 数据层：MySQL持久业务数据，Redis缓存token、限流、临时优惠计算缓存、分布式锁

分层原则：Controller → Service → Mapper，严格分层，禁止跨层直接调用。

## 2. 模块划分（包结构）
com.smartconsume
├── auth        认证、JWT、登录
├── tenant      租户、License管理
├── calculator  AI凑单计算核心算法
├── monitor     价格监控、价格历史、告警任务
├── bill        消费账单、复盘报表
├── community   帖子、评论、点赞收藏、审核
├── sync        跨端增量同步、冲突处理、水位管理
├── user        用户信息、积分、等级
├── msg         站内消息、消息推送
├── common      公共类、统一返回、异常、工具类
├── ws          WebSocket推送服务

## 3. 数据库设计概要（完整SQL单独文件）
核心表清单：
1. sys_user 用户表，tenant_id隔离
2. sys_tenant 租户表，license_key、max_user、expire_time
3. sys_license_log license校验日志
4. calc_plan 凑单方案主表
5. calc_plan_goods 方案内商品子表
6. calc_rule 优惠规则子表
7. monitor_task 监控任务
8. monitor_price_record 历史价格流水
9. consume_bill 消费账单
10. community_post 帖子
11. community_comment 评论
12. user_point_log 积分流水
13. sys_message 站内消息
14. sync_water_mark 同步水位表
15. sync_operation_log 同步操作日志

主键统一bigint自增；所有业务表带tenant_id、create_time、update_time、del_flag逻辑删除。

## 4. 核心模块详细设计
### 4.1 JWT认证设计
登录成功下发token，存入Redis黑名单用于登出失效；请求拦截器校验Authorization头；401无权限统一返回。

### 4.2 License & 租户隔离设计
所有查询强制拼接tenant_id；超管tenantId=0；定时任务每12h远程校验license，校验失败标记租户冻结，冻结后禁止新用户登录。

### 4.3 AI凑单计算器核心算法设计
输入：商品数组（单价、数量）、优惠规则数组（满减门槛、减免金额、是否可叠加、品类限制）
算法策略：
1. 过滤无效规则，按优先级排序（券>满减>津贴）
2. 回溯枚举可行组合（控制组合数量，避免爆炸，限制商品上限15件）
3. 计算每种组合实付、单品分摊价、优惠利用率
4. 打分排序：省钱金额优先，叠加风险标签
输出：top1最优方案 + 备选方案列表
> 注意：不接入大模型API，纯组合数学计算，降低私有化成本

### 4.4 价格监控&告警设计
插件上报价格变更 → 写入monitor_price_record；对比target_price；达到阈值，写入sys_message，通过WebSocket推送到在线客户端；Chrome通知由插件service worker接收推送。
轮询：后端不主动爬，价格全部来自前端插件上报。

### 4.5 增量同步模块（重点设计）
核心字段：table_name、water_time(水位时间戳)、data_version版本号
- Pull流程：客户端传入表名+lastWaterTime，服务端查询该表 > water_time、未删除数据，返回数据+新水位
- Push流程：客户端推送变更，携带localVersion；服务端查询云端当前版本；
    - 如果云端版本 > localVersion → 标记冲突 conflict=true，返回最新云端数据，由前端做合并弹窗
    - 无冲突：覆盖写入，版本+1
冲突策略：服务端只检测，**合并策略交给前端**（用户选择以本地为准 / 云端为准）
同步表白名单：账单、凑单方案、监控任务、收藏帖子；配置类、租户管理数据不开放同步。

### 4.6 WebSocket推送设计
连接时携带token鉴权；按用户id订阅通道；推送事件类型：price_alert、sync_change、comment_notice；断线自动重连由前端实现。

## 5. 前端架构设计
Vue3 + Vite + Element Plus + Pinia状态管理
页面路由划分：
/login 登录
/home 首页
/calc 凑单计算器
/monitor 价格监控列表
/bill 账单、月度复盘
/community 社区帖子
/user 用户中心、积分等级
/msg 消息中心
/tenant 租户后台（租户管理员）
/super-admin 超管后台
/personal-sync 同步管理页面
本地存储：localStorage存放token、本地缓存账单/方案，用于离线模式

## 6. Chrome插件设计
Manifest V3；content脚本注入电商页面，隔离环境，只做DOM解析提取商品；background service worker负责通知监听、推送接收；popup弹窗作为操作面板；存储token在chrome.storage.local。

## 7. 异常与错误码设计
200成功
401 token失效未登录
403 权限不足 / license冻结
404 资源不存在
409 同步数据冲突
500 服务内部异常

## 8. 安全设计
- 密码BCrypt哈希，不存明文
- 接口防重、简单限流（Redis）
- 租户行级数据隔离，禁止跨租户查询
- 插件host_permissions严格限定电商域名，不开放全网权限
# 精明消费AI平台 V1.0 软件开发文档（开发指引）
面向AI开发执行使用，规定编码规范、开发顺序、任务拆分、自测清单、交付验收标准

## 1. 开发顺序（AI必须按顺序做，避免依赖混乱）
阶段1：数据库初始化脚本 init.sql 编写，建表、索引、基础数据
阶段2：后端基础脚手架搭建：统一返回、全局异常、JWT工具、Redis封装、MyBatis-Plus
阶段3：认证模块开发（登录、登出、获取用户信息、拦截器）
阶段4：租户+License模块、权限隔离逻辑
阶段5：核心凑单计算器算法 + 相关CRUD接口
阶段6：价格监控模块、价格记录、告警消息逻辑
阶段7：消费账单、月度复盘报表接口
阶段8：社区帖子评论点赞收藏、审核接口
阶段9：增量同步模块（pull/push、冲突检测、水位表）
阶段10：站内消息、WebSocket实时推送服务
阶段11：后端全部接口自测，对齐OpenAPI文档
阶段12：Vue3前端页面开发，路由、权限、页面组件
阶段13：前端同步逻辑、离线缓存、冲突弹窗处理
阶段14：Chrome扩展插件开发、页面抓取逻辑、通知逻辑
阶段15：Docker打包、docker-compose.yml、Nginx配置、部署测试
阶段16：全流程联调、回归测试、输出全部交付文件

## 2. 编码规范
### 后端 Java
1. 使用SpringBoot 3.x + JDK21 + MyBatis-Plus
2. 所有数据库实体继承基础BaseEntity：id、tenant_id、create_time、update_time、del_flag
3. 禁止硬编码常量，统一放在Constants类
4. 业务校验在Service层抛出自定义异常，全局异常处理器捕获包装为统一返回
5. 定时任务使用Spring @Scheduled，License校验任务
6. 不使用复杂分布式事务，本项目单体，业务最终一致性即可
### 前端 Vue3
1. <script setup>语法；Pinia管理全局用户、token状态
2. 接口统一封装axios请求，拦截401自动跳转登录
3. 分页组件统一封装；表格支持筛选
4. 同步冲突弹窗：两个选项【以本地覆盖云端 / 以云端覆盖本地】
### 插件
1. 严格区分content.js（页面读取）和background.js（后台任务）
2. 禁止在content脚本写业务提交逻辑，数据通过popup转发后端

## 3. 接口开发约束
严格按照【05-OpenAPI-接口文档.md】实现，入参、出参字段名大小写、结构不能随意修改。

## 4. 自测清单（每模块完成必须自测）
✅ 登录、token过期、退出登录
✅ 租户隔离：A租户用户看不到B租户任何数据
✅ License过期后租户用户无法登录
✅ 凑单：多商品、多券叠加，计算金额正确，风险标签输出
✅ 监控上报价格、降价触发消息、websocket推送
✅ 账单新增、月度报表统计正确
✅ 发帖、评论、审核、点赞收藏
✅ 多设备同步：新增、修改、删除；冲突场景能识别并弹窗
✅ 消息已读、全部已读
✅ Chrome插件淘宝/京东/pdd抓取商品、添加监控
✅ docker compose up 一键启动整套服务，无报错
✅ 前端路由权限：普通用户看不到租户/超管页面

## 5. 交付包目录结构（最终交付）
smart-consume-package/
├── backend/
│   ├── Dockerfile
│   └── target/smart-consume.jar
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── dist/
├── chrome-extension/
│   ├── manifest.json
│   ├── popup.html popup.js content.js background.js icon
├── sql/init.sql
├── docker-compose.yml
├── 文档文件夹/
│   ├── SRS规格.md
│   ├── SDD设计.md
│   ├── DEV开发指南.md
│   ├── FRAMEWORK框架.md
│   ├── OpenAPI接口.md

## 6. 验收标准
1. 全部接口和OpenAPI文档一致，请求返回结构匹配
2. 租户数据完全隔离
3. 凑单数学计算结果准确，无计算溢出
4. 同步推拉、冲突检测功能稳定
5. 插件可正常抓取三大平台商品
6. Docker一键部署成功，访问Web页面正常，WebSocket推送正常
7. 所有文档和代码配套，注释清晰
# 精明消费AI平台 V1.0 开发框架文档 FRAMEWORK
作用：给AI固定依赖版本、脚手架结构、配置模板、第三方组件清单，防止AI随意更换技术栈

## 1. 后端框架信息
基础框架：SpringBoot 3.2.x
JDK：21
构建工具：Maven
ORM：MyBatis-Plus 3.5.5
数据库：MySQL 8.0
缓存：Redis 7.2
安全：jjwt（JWT）、BCrypt密码加密
WebSocket：Spring WebSocket
定时任务：Spring自带Scheduled
工具包：hutool-all
依赖清单pom.xml核心依赖：
spring-boot-starter-web
spring-boot-starter-websocket
mybatis-plus-boot-starter
mysql-connector-j
spring-boot-starter-data-redis
jjwt-api, jjwt-impl, jjwt-jackson
hutool-all
lombok

### 后端application.yml模板
```yaml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://mysql:3306/smart_consume?useUnicode=true&characterEncoding=utf8mb4&serverTimezone=Asia/Shanghai
    username: root
    password: Smart@2026
  redis:
    host: redis
    port: 6379
mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
  global-config:
    db-config:
      logic-delete-field: delFlag
      logic-delete-value: 1
      logic-not-delete-value: 0
jwt:
  secret: QWxhZGRpbjpvcGVuIHNlc2FtZQ==
  expire: 86400000
license:
  check-cron: 0 0 */12 * * ?
2. 前端框架信息
 
框架：Vue3 + Vite
UI库：Element Plus
状态管理：Pinia
网络请求：Axios
路由：Vue Router 4
构建目标：静态dist，部署到Nginx
目录规范：
src/
├── api/        所有接口请求封装
├── components/ 公共组件
├── views/      页面
├── store/      pinia
├── router/     路由+权限守卫
├── utils/      jwt、格式化、同步工具
├── assets/     静态资源
 
3. Chrome插件框架
 
Manifest版本：V3
原生JS，不引入Vue/React，轻量化，减少插件体积
文件固定清单：manifest.json, popup.html, popup.js, content.js, background.js
 
4. 部署框架
 
容器：Docker + Docker Compose
镜像版本锁定：mysql:8.0.36 redis:7.2-alpine nginx:alpine openjdk:21-jdk-slim
网络：bridge独立网络smart-net
反向代理：Nginx统一转发前端静态、/api后端、/ws websocket
 
5. 外部依赖说明
 
本项目不依赖外部LLM大模型接口，凑单是本地组合算法，私有化离线可用；
不搭建爬虫服务，商品数据来源用户浏览器插件DOM解析上报；
消息推送只使用WebSocket站内推送，可选扩展邮件/短信（本次版本不做）。
 
6. 扩展预留点（AI不要实现，预留）
 
1. 新增电商平台解析规则，只需要新增content.js分支
​
2. 新增优惠类型，计算器模块增加RuleHandler实现类
​
3. 后续可拆分为微服务，当前版本单体优先
​
4. 同步模块可新增同步表，只需要在白名单配置增加表名
 
7. 禁止修改的框架约束
 
❌ 禁止替换SpringBoot为其他后端框架
❌ 禁止把Manifest V3降级V2
❌ 禁止把同步推拉模型改成只拉或者只推
❌ 禁止去掉租户+License隔离模型
❌ 禁止后端主动发起HTTP爬虫访问电商站点
---

# 配套提示（给你交给AI的提示词，你可以复制一起发）
> 下面这4份文档：SRS软件规格说明书、SDD软件设计文档、DEV软件开发指南、FRAMEWORK开发框架文档，加上之前的PRD、SQL、OpenAPI、Docker、Chrome插件文档，作为整套项目基准。请严格全部文档对齐开发，不擅自修改需求、字段、接口结构、技术栈；开发顺序按照开发指南的阶段顺序执行，每个阶段完成自测，输出对应代码和文件。

如果你想要，我下一步可以：
方案A：我把**全部8份文档（这4份 + 之前4份）合并成一个完整大markdown总文档**
方案B：我给你生成一个 `readme.md` 打包说明 + 一个一键打包脚本（windows bat）方便你本地打包zip
你选哪一个？
