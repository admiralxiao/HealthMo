# 埋点事件枚举与 payload（商业化）

更新时间：2026-01-30  
要求：商业事件必须携带 offer_id、plan_id（如有）；统一 snake_case。

## 1) Onboarding
- onboarding_start
- goal_selected
- onboarding_question_answered
- onboarding_complete

## 2) Value & Proof
- value_preview_view
- tech_proof_view
- tech_proof_expand

## 3) Paywall
- paywall_view
- paywall_offer_shown（B/C 弹层出现）
- paywall_plan_selected
- paywall_cta_click
- paywall_not_now
- paywall_restore_click

payload 示例：
```json
{
  "offer_id": "A",
  "layout_variant": "oura_v1",
  "plans": ["weekly_9_99","yearly_59_99"],
  "default_plan": "weekly_9_99",
  "personal_promise_key": "cut_appetite_window"
}
```

## 4) Purchase / Restore（StoreKit2）
- purchase_start
- purchase_success
- purchase_fail
- restore_success
- restore_fail

## 5) Entitlement
- entitlement_fetch
- entitlement_active
- entitlement_inactive
- entitlement_mismatch
- premium_api_403

## 6) 公共字段（所有事件）
- user_id（如有）、device_id、session_id、app_version
- locale、timezone、risk_level
- bootstrap_version、params_version
