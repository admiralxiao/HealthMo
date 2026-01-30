# 02-CoreMotion 分段与强度规则

更新时间：2026-01-30

## iOS 数据源
- CMMotionActivityManager：walk/run/cycle/automotive/stationary
- CMPedometer：steps/distance（walk/run）

## 分段规则（MVP）
- 同类 activity 连续合并（短间断<2min 合并）
- 优先级：sleep > automotive > run > bike > walk > still
- unknown：按 still

## 强度分级（MVP）
- 基于步频 steps/min：
  - light <100
  - moderate 100–130
  - vigorous >130
- bike：有速度则按速度分级；否则默认 moderate
