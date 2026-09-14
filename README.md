# BookStack 软件测试项目

> 基于开源知识库系统 BookStack 的完整软件测试实践项目，覆盖需求分析、测试设计、功能测试、REST API 测试、SQL 数据验证、JMeter 基础性能测试以及测试报告输出。

## 项目概述

本项目以 BookStack 为被测系统，在 macOS + Docker 本地环境中按照较完整的软件测试流程开展实践。项目重点不是单纯“跑用例”，而是从业务梳理开始，逐步完成测试点设计、功能用例设计与执行、问题记录、接口测试、数据库一致性验证和基础性能测试，并最终形成可复现、可追溯的测试产物。

本项目主要采用 **手工功能测试 + Postman 接口测试 + SQL 数据验证 + JMeter 基础性能测试**。

## 测试流程

```text
需求/业务分析
    ↓
测试点设计（XMind）
    ↓
功能测试用例设计与执行
    ↓
测试发现与优化建议
    ↓
Postman REST API 测试
    ↓
Newman 批量执行与 HTML 报告
    ↓
SQL 数据一致性验证
    ↓
JMeter 基础性能测试
    ↓
测试报告汇总
```

---

## 测试结果总览

| 测试阶段 | 结果 |
| --- | --- |
| 功能测试 | 设计并执行 **291** 条用例，**290 通过、1 阻塞、0 失败** |
| 测试发现 | **1 项优化建议、1 项待确认项** |
| Postman 接口测试 | **15 个请求、44 个断言、0 失败** |
| Newman 批量执行 | 1 次完整 Collection Run，**15/15 请求成功，44/44 断言通过** |
| SQL 数据验证 | **9 项验证，9 项通过** |
| JMeter 基础性能场景 | **90 次请求，错误率 0%，平均响应时间约 197 ms，最大 260 ms** |
| JMeter 高频访问场景 | **300 次请求，约 39.67% 请求受限；观察到 API 180 请求限流机制** |
| 最终测试报告 | 已完成 |

---

## 测试环境与工具

| 类别 | 环境 / 工具 |
| --- | --- |
| 操作系统 | macOS |
| 被测系统 | BookStack |
| 部署方式 | Docker 本地部署 |
| Web 地址 | `http://localhost:6875` |
| 功能测试 | 手工测试 |
| 测试设计 | XMind、Excel |
| 接口测试 | Postman |
| 接口批量执行 / 报告 | Newman + htmlextra |
| 数据库验证 | MariaDB / SQL / DBeaver |
| 性能测试 | Apache JMeter 5.6.3 |
| 测试报告 | Word |

> API Token、数据库密码等敏感信息仅保存在本地环境中

---

## 功能测试

### 测试范围

功能测试覆盖 BookStack 核心业务模块：

| 模块 | 用例数 |
| --- | ---: |
| Login / Logout | 17 |
| Shelves | 113 |
| Books | 32 |
| Chapters | 8 |
| Pages | 61 |
| Search | 14 |
| Tags | 17 |
| Recycle Bin | 29 |
| **合计** | **291** |

测试内容主要包括：

- 正常业务流程与核心 CRUD 操作
- 必填项、空值及输入合法性校验
- 等价类、边界值和异常场景
- 长文本、特殊字符、重复数据等输入场景
- Book / Chapter / Page 等层级关系
- Shelf 与 Book 的关联关系
- 搜索与标签功能
- 删除、回收站恢复及数据关系保持
- 登录、退出与会话相关行为

### 执行结果

共执行 291 条功能测试用例：

```text
PASS     290
BLOCKED    1
FAIL       0
```

唯一阻塞项为 **Remember Me 未勾选场景**。在当前环境下，勾选与不勾选 Remember Me 均观察到重新打开浏览器后保持登录状态。由于缺少明确的产品需求说明，无法确认普通 Session 与 Remember Me 的预期差异，因此该项记录为“阻塞 / 待确认”，未直接判定为缺陷。

---

## 测试发现与优化建议

功能测试过程中记录了 2 项需要关注的现象：

### 1. 重复名称 / 重复值缺少提示与区分机制

影响范围包括 Shelf、Book、Chapter、Page、Tag 等需要填写 Name / Value 的创建场景。

系统允许与已有数据完全相同的名称或值再次创建，新旧数据可同时存在，不会覆盖原数据，也不会出现页面异常；但系统不会提示重复，也不会自动区分名称。

该现象未违反当前可确认的明确需求，因此记录为 **优化建议**，而非直接判定为 Bug。建议在允许重复数据的前提下提供提示或更清晰的区分机制，降低误操作、搜索混淆及数据冗余风险。

### 2. Remember Me 行为待确认

勾选和不勾选 Remember Me 时均观察到浏览器重新打开后保持登录状态。由于缺少明确的会话保持规则，当前记录为 **待确认项**，需要结合产品需求或配置进一步判断。

---

## Postman REST API 测试

### 认证方式

BookStack API 使用 Token 认证：

```http
Authorization: Token {{token_id}}:{{token_secret}}
```

Postman Environment 管理 `base_url`、Token 以及接口间动态传递的资源 ID。真实 Token 不提交到仓库。

### API 覆盖

#### Books

完成 Book 模块 CRUD 与异常场景测试：

```text
GET     /api/books
POST    /api/books
GET     /api/books/{id}
PUT     /api/books/{id}
DELETE  /api/books/{id}
```

覆盖：

- Book 列表查询
- Book 创建 / 查询 / 修改 / 删除
- 查询不存在的 Book
- 重复删除不存在的 Book
- 缺少必填 Name
- 无效 API Token
- `401 / 404 / 422` 等异常响应验证

#### Shelves

```text
GET   /api/shelves
POST  /api/shelves
GET   /api/shelves/{id}
```

覆盖 Query Params、`count` 数量限制、`sort` 排序、Shelf 创建以及 Shelf 与 Book 关联关系验证。

#### Pages

```text
POST  /api/pages
GET   /api/pages/{id}
PUT   /api/pages/{id}
```

覆盖 Page 创建、查询、修改、所属 Book 验证以及 HTML 正文更新验证。

### 自动断言与数据关联

接口测试使用 Post-response Script 完成：

- Status Code 校验
- JSON 格式校验
- 字段存在性与字段值校验
- 数组类型及包含关系校验
- Query Params 结果校验
- API 错误信息校验
- Environment 动态变量读取与写入

同时实现接口之间的数据传递，例如：

```text
Create Book → 保存 book_id → Get → Update → Delete
Create Shelf → 保存 shelf_id → Get Single Shelf → 验证 Book 关联
Create Page → 保存 page_id → Get Single Page → Update Page
```

### Newman 执行结果

使用 Newman 对完整 Collection 进行批量执行并生成 HTML 报告：

```text
Iterations          1
Requests            15 / 15 passed
Test Scripts        15 / 15 passed
Assertions          44 / 44 passed
Failed Tests        0
Total Run Duration  约 3.2 s
Average Response    约 194 ms
```

Newman 中的响应时间仅作为接口运行记录，不作为正式性能结论；性能场景单独使用 JMeter 执行。

---

## SQL 数据验证

接口测试完成后，通过 SQL 对关键业务数据进行数据库层验证，确认接口返回结果与数据库实际持久化结果一致。

BookStack 当前数据库结构中，Book、Bookshelf、Chapter、Page 的公共字段主要存储在 `entities`，并结合以下表完成扩展数据与关系验证：

```text
entities
entity_container_data
entity_page_data
bookshelves_books
deletions
```

共完成 **9 项 SQL 验证，全部通过**，主要包括：

- Page 更新后的基础字段落库验证
- Page HTML / Text 正文持久化验证
- Shelf 与 Book 关联表验证
- JOIN 查询验证 Shelf / Book 业务关系
- Book 创建数据落库验证
- Book 修改结果验证
- Book 删除 / Recycle Bin 数据状态验证
- Page 与父 Book 关联关系验证
- 关键实体孤立数据完整性检查

验证链路：

```text
API 操作
  → API Response / Assertion
  → SQL 查询
  → 数据库持久化结果
  → 业务关系一致性确认
```

---

## JMeter 基础性能测试

性能测试选择 3 个代表性只读接口：

```text
GET /api/books
GET /api/shelves
GET /api/pages/{id}
```

每个请求配置 Response Assertion，要求响应状态码为 `200`。正式负载执行时关闭 `View Results Tree`，并通过 CLI/non-GUI 方式生成 JMeter HTML Dashboard。

### 场景 1：基础并发场景

```text
并发用户数：10
Ramp-up：10 s
Loop Count：3
接口数量：3
总请求数：90
```

结果：

| 指标 | 结果 |
| --- | ---: |
| 总请求数 | 90 |
| Error % | **0.00%** |
| Average | **约 197 ms** |
| Min | 约 190 ms |
| Max | **260 ms** |
| Throughput | 约 8.4 req/s |

在当前本地测试环境与该负载场景下，90 次请求全部成功，未观察到请求失败，整体执行稳定。

### 场景 2：高频访问与限流观察

```text
并发用户数：20
Loop Count：5
接口数量：3
总请求数：300
```

该场景下约 **39.67%** 请求被限制。结合接口响应头中观察到的 `X-RateLimit-Limit: 180`，本次高频访问触发了 BookStack API 的请求限流机制。

该结果用于验证系统在高频访问下的限流行为，**不能直接等同于服务器性能不足**。


---

## 测试结论

本项目已完成从测试分析到结果输出的完整实践闭环。

功能层共执行 291 条测试用例，其中 290 条通过、1 条因需求规则不明确而阻塞，未记录明确的功能失败用例；同时记录 1 项优化建议和 1 项待确认项。接口层通过 Postman/Newman 完成 15 个代表性请求与 44 个自动断言，最终批量执行无失败。数据库层完成 9 项 SQL 一致性验证并全部通过。性能层完成基础并发场景及高频访问限流观察，并生成 JMeter HTML Dashboard。

基于当前测试范围与本地测试环境，BookStack 核心业务流程、代表性 API 以及关键数据关系运行稳定。已识别的问题主要集中在重复数据的可识别性和 Remember Me 行为规则确认，建议结合明确产品需求进一步评估。

---

## 项目实践能力

通过本项目实践并形成可展示成果的能力包括：

- 根据实际系统梳理业务流程并确定测试范围
- 使用 XMind 拆分测试点
- 使用等价类、边界值、异常场景等方法设计功能测试用例
- 执行手工功能测试并记录实际结果
- 区分明确缺陷、优化建议与需求待确认项
- 使用 Postman 完成 REST API 正常与异常场景测试
- 使用 Environment 与 Post-response Script 完成接口参数化、断言和动态数据传递
- 使用 Newman 批量执行 Collection 并生成 HTML 报告
- 使用 SQL 验证 API 操作后的数据持久化及表关联一致性
- 使用 JOIN 对多表业务关系进行验证
- 使用 JMeter 配置 Thread Group、HTTP Request、Header Manager、Assertion 并执行基础并发测试
- 分析响应时间、吞吐量、错误率以及 API 限流现象
- 输出完整的软件测试报告并整理 GitHub 测试项目产物


## 说明

本项目基于开源项目 **BookStack** 进行软件测试实践，仅用于个人学习、测试能力训练及求职项目展示。

BookStack 项目版权归原项目作者及贡献者所有。
