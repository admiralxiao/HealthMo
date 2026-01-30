# App Store Connect 准备（提交材料与流程）— MVP v1

更新时间：2026-01-29

---

## 1. App 信息（必须）
- App Name（主语言 en-US）
- Subtitle（≤30 chars）
- Description（英文，短段落 + bullet）
- Keywords（≤100 chars）
- Support URL（必须）
- Privacy Policy URL（必须）
- 联系方式（邮箱）

---

## 2. 版本提交（必须）
- What’s New（英文）
- Screenshots（建议：iPhone 6.7" + 6.1"）
- App Preview（可选，MVP 可不做）
- Review Notes（给审查员的说明）
  - 登录方式（如需要测试账号）
  - 权限说明（Motion/Health/Camera/Notifications）
  - 订阅说明（如何触发 paywall/explain gate）

---

## 3. 截图建议（Oura 风）
建议 6 张（按价值顺序）：
1) Today：Energy + Recovery（核心）
2) Food：Photo → Confirm（可信度）
3) Activity：Auto segments（自动化）
4) Timing：Daily windows（差异点）
5) Sleep：Recovery explain（专业感）
6) Paywall：Premium benefits（可解释 + 个性化）

---

## 4. Review Notes 模板（可复制）
- App requires:
  - Motion & Fitness: to auto-detect walking/running/cycling/vehicle
  - Health: to read sleep summary (if available)
  - Camera: to log meals by photo
- Steps to access Premium explain:
  1) Open Today → Recovery card → Explain
  2) Paywall appears → purchase/restore
- Test account (if email login enabled):
  - user: (provide to reviewer)
  - pass: (provide to reviewer)

---

## 5. 提交流程建议
- 先提交 dev/staging 通过 TestFlight 内测
- 再提交 App Store Review
- 开启 Phased Release（7 天）降低风险（MVP 推荐）

---
