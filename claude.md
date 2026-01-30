# Claude 项目规范（health_app_mvp）

更新时间：2026-01-30

本文件用于 **Claude CLI** / 代码生成助手在本仓库内协作开发时遵循统一规范，避免“风格漂移”“接口不一致”“重复造轮子”。

---

## 1. 项目目标与范围（MVP，2 周）
- iOS：Swift 原生，iOS 15.6+
- 后端：FastAPI + Postgres
- 商业化：问卷 → 价值预览 → Paywall（周订主推 + 年卡承接 + 分层折扣）
- 安全：会员防破解、风控与反滥用
- 文档：`/docs` 为唯一权威需求与规范来源

---

## 2. 目录结构（必须遵守）
- `/ios`：iOS 工程代码（Swift）
- `/backend`：FastAPI 服务
- `/docs`：产品/算法/上线/安全/商业化文档（**需求权威**）
- 本仓库不在生成输出中夹带“临时目录”；所有生成文件必须落到以上三处之一。

---

## 3. 文档权威与变更流程
- 当代码实现与文档冲突：**先改文档，再改代码**（除非是明显文档错误）
- 每次生成/修改代码时，必须在提交说明中引用对应文档路径，例如：
  - `docs/16-商业化实现细节/05-StoreKit2交易与上报.md`

---

## 4. iOS 代码规范（Swift）
### 4.1 架构
- 使用 **MVVM + Coordinator**
- 商业化模块独立：`Features/Monetization/*`
- 权限/会员只看 `EntitlementStore`（客户端唯一权威）

### 4.2 语言与风格
- Swift 5.7+（跟随 Xcode）
- `@MainActor`：所有 UI 相关 ViewModel/Coordinator
- 严禁在 View 里写网络逻辑；一律下沉到 Store/Service
- 统一命名：类型 PascalCase，变量/函数 camelCase

### 4.3 并发
- 优先 `async/await`
- 网络请求必须支持取消（Task cancellation 友好）

### 4.4 StoreKit2（强制）
- 只接受 verified transaction
- 购买后必须 `POST /iap/transactions`，并以 `GET /entitlement` 结果决定解锁
- `Transaction.updates` 必须监听（续订/状态变化）

---

## 5. Backend 规范（FastAPI）
- 清晰分层：`api/`（路由） → `services/`（业务） → `db/`（持久化）
- 返回结构稳定、可版本化（预留 `schema_version` / `params_version`）
- Premium-only API 必须在服务端鉴权 entitlement，否则返回 403

---

## 6. Remote Config / Feature Flags（强制）
- 所有可运营调参必须通过 `/config/bootstrap` 下发（见 docs/12, docs/16）
- 必须支持 kill switch：
  - `kill_switch.all`
  - `kill_switch.payments`
- 折扣 offer（B/C）必须受风控 `risk_level` 约束

---

## 7. 埋点与隐私
- 事件命名 snake_case；商业化事件必须携带 `offer_id`、`plan_id`
- 不收集过度个人信息；IP 仅 hash 存储（若需要）
- 文案避免医疗承诺（wellness/insights/guidance 口径）

---

## 8. 生成代码的“硬要求”
当 Claude CLI 生成/修改代码时必须：
1) 说明“改动涉及哪些文件”
2) 指出“对应哪些文档条款”
3) 产出可编译/可运行的最小闭环（不要只给片段）
4) 不引入不必要的第三方依赖（除非文档明确允许）
5) 所有新增 public API/DTO 必须补充到 `/docs` 对应章节

---

## 9. 常用指令模板（建议）
- iOS：生成商业化骨架
  - `claude -p "根据 docs/17-商业化代码骨架/... 生成 Swift 文件并保证可编译"`
- Backend：生成路由/Schema/Model
  - `claude -p "根据 docs/17-商业化代码骨架/06... 生成 FastAPI 代码并给出启动命令"`

---

## 10. Definition of Done（DoD）
- iOS：商业化 flow 可跑通（含 skip、restore、购买后解锁）
- Backend：entitlement 与 iap 交易接口可用
- 关键埋点齐全（paywall_view、cta_click、purchase_success/fail）
- kill switch 可一键止血
