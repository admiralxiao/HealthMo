# 04-iOS实现指南

更新时间：2026-01-30

## RemoteConfigStore
- 启动：拉取并缓存（UserDefaults/SQLite）
- TTL：1h；支持强刷新
- 提供：
  - `flag(_ key: String) -> Bool`
  - `param<T>(_ key: String, default: T) -> T`
  - `provider(_ key: String, default: String) -> String`

## 使用约束
- 所有可调参必须来自 config（不得散落常量）
- Paywall 展示与购买后必须 refresh entitlement + refresh config（可选）
