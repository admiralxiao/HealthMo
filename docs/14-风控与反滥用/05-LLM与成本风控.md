# LLM / Vision 成本控制（配额、限频、降级、审计）

更新时间：2026-01-30  
目标：防止 photo/文本识别被刷导致成本不可控，并保持正常用户体验。

---

## 1. 必须具备的“四道闸”
1) **限频**（rate limit）：每用户/每 device/每 ip_hash  
2) **配额**（quota）：每日/月度次数  
3) **降级**（degrade）：高风险时禁用 photo，改用条码/搜索  
4) **审计**（audit）：记录每次调用 provider、tokens、cost_estimate

---

## 2. 推荐默认配额（MVP）
- Free：
  - photo：`10/day`
  - text scan：`20/day`
- Premium：
  - photo：`50/day`
  - text scan：`100/day`

参数：
- `quota.vision.photo.free.daily = 10`
- `quota.vision.photo.premium.daily = 50`
- `quota.vision.text.free.daily = 20`
- `quota.vision.text.premium.daily = 100`

---

## 3. 价格/成本核算（服务端）
每次调用记录：
- provider（openai/gemini/…）
- model
- image_size（是否压缩）
- tokens_in/out（若可得）
- cost_estimate_usd（按你内部定价表估算）

表：`usage_vision`
- user_id, device_id, day, count, cost_usd

---

## 4. 降级策略（与 docs/12 联动）
- medium risk：
  - photo quota 降为 3/day
- high risk：
  - kill_switch.vision=true（仅对该用户/设备；需要 per-user override）
- critical：
  - block vision endpoint

> 如果你暂时没有 per-user override：先用 throttle + 403 + 提示“Try again later”。

---

## 5. 图片入口防滥用（客户端）
- 先本地做压缩/尺寸限制（例如最长边 1024）
- 明确提示：只拍食物、避免重复提交
- 识别失败时引导条码/搜索（降低重试次数）

---
