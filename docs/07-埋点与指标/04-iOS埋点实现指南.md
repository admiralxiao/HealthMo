# 04-iOS埋点实现指南

更新时间：2026-01-30

- Event 枚举（name + payload builder）
- AnalyticsClient：队列、批量、重试
- UI 只调用 track(event)，不拼 payload
