# 06-LBI（瘦素平衡支持度）算法

更新时间：2026-01-30

## 输入（MVP）
- sleep_debt_minutes
- deep_quality：deep_ratio + deep_first2_ratio
- regularity：sleep_midpoint_std_minutes

## 示例计算
- debt_component = clamp(1 - debt/180, 0, 1)
- deep_component = clamp(deep_ratio*0.7 + deep_first2_ratio*0.3, 0, 1)
- regular_component = clamp(1 - std/120, 0, 1)
- LBI = 100*(0.4*debt + 0.35*deep + 0.25*regular)

权重与阈值全部在 docs/11 参数表中版本化。
