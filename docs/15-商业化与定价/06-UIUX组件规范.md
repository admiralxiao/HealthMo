# UI/UX 组件规范（Oura 风）— Onboarding & Paywall

更新时间：2026-01-30  
目标：高级感、克制、可信，减少“营销味”，但依然高转化。

---

## 1. 视觉基调
- 深色底 + 轻微渐变（避免纯黑）
- 卡片：半透明层 + 细边框（1px）
- 高亮：单一主色（由品牌色决定），避免花哨
- 文案：短句、可读性强、少感叹号

---

## 2. 组件清单（MVP）
### 2.1 Question Card（问卷）
- title（1 行）
- options（radio list，整行可点）
- helper text（可选）
交互：
- 选中后“下一步”按钮解锁（减少误触）

### 2.2 Value Card（价值预览/Paywall）
- icon（线性）
- title（短）
- desc（一句话结果）
- tag（可选：Recovery / Timing / Energy）

### 2.3 Plan Picker（定价选择）
- Radio Card x2
- badge：Most flexible / Best value
- secondary：折算 $/week

### 2.4 Primary CTA
- 全宽
- 高度 52–56
- 圆角 16–18
- 文案：Start Premium / Continue

### 2.5 Fine Print
- 自动续订
- Terms / Privacy
- Restore

---

## 3. Layout 规格（建议）
- 页面左右 padding：20
- 卡片圆角：22
- 卡片间距：12
- 标题字号：28–32
- 正文字号：13–15

---

## 4. 交互细节（提升转化但不油腻）
- paywall 关闭按钮：可用但不抢眼
- “Not now”：低对比度文字按钮
- 折扣弹层：只在用户犹豫时出现，默认不打扰

---
