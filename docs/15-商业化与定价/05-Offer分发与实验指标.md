# Offer 分发、A/B 实验与指标（Growth Ops）

更新时间：2026-01-30

---

## 1. Offer 分发（Remote Config）
通过 docs/12 的 bootstrap 下发：
- `providers.paywall_offer = A/B/C`
- `flags.feature.paywall.offer_B = true`
- `flags.feature.paywall.offer_C = true`
- `params.paywall.offer_trigger.*`

支持 per-user override（推荐 v1.1）：
- user_id → offer_id

---

## 2. A/B 实验（MVP 必做 3 个）
### Experiment 1：是否需要“价值预览页”
- A：直接 paywall
- B：价值预览 → paywall
指标：
- paywall_conversion
- refund_rate（30d）
- 7d retention

### Experiment 2：年卡承接策略
- A：年卡与周卡并列
- B：周卡默认，年卡在下方“Try yearly instead”
指标：
- yearly share
- overall conversion

### Experiment 3：折扣弹层触发时机
- A：停留 8 秒触发
- B：点击 Not now 触发
指标：
- conversion uplift
- refund uplift（折扣用户可能退款高）

---

## 3. 核心指标（North Star + Guardrail）
North Star：
- Paid conversion（D0→D1）
- 7d paid retention（是否续订/未退款）

Guardrails：
- refund rate
- trial abuse rate
- support tickets（计费相关）

---

## 4. 事件埋点（建议最小集合）
- onboarding_start
- goal_selected
- value_preview_view
- paywall_view (offer_id)
- plan_selected (weekly/yearly)
- paywall_cta_click
- subscribe_success / subscribe_fail
- refund_event（来自通知）
- restore_success / restore_fail

维度：
- offer_id, locale, channel, risk_level, app_version

---
