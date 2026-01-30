# 运营面板与处置 Playbook（上线后每日必看）

更新时间：2026-01-30  
目标：让团队能“每天 10 分钟”判断是否有滥用、成本是否健康、是否需要止血。

---

## 1. 每日看板（最小）
- 新注册数（按国家/locale）
- 登录失败率
- 订阅转化（paywall_view → subscribe_success）
- 退款/撤销数量
- vision 请求量、失败率、cost_usd
- top 10 风险用户/设备（risk_score）

---

## 2. 告警阈值（建议）
- vision cost 日增幅 > 50%（或绝对值超预算）
- vision 失败率 > 40% 且请求量高
- 注册数异常（同 IP/device 激增）
- premium 403 激增（可能出现破解或 entitlement 同步问题）

---

## 3. Playbook：vision 成本爆炸
1) 打开 dashboard 确认来源（地区/版本/渠道）
2) 临时动作：
   - 降低 quota（远程配置）
   - 对异常段启用 throttle
3) 紧急：
   - kill_switch.vision=true（全局）
4) 复盘：
   - 找到触发路径（UI 重试/脚本攻击）
   - 修复后逐步恢复（phased rollout）

---

## 4. Playbook：疑似破解 Premium
1) 查看 premium API 403 分布（user/device）
2) 对高风险设备：
   - 强制 step-up（App Attest）
   - 临时封禁
3) 核查 entitlement 链路（Apple 验证/通知）

---

## 5. Playbook：trial 滥用
1) 同 device 多账号 trial 命中率
2) 立刻：
   - `trial.max_per_device=1`
   - medium risk 默认不展示 trial
3) 加强：邮箱验证/人机验证（如需要）

---
