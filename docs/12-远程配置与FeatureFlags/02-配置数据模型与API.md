# 02-配置数据模型与API

更新时间：2026-01-30

## Bootstrap 响应结构（建议）
- `bootstrap_version`
- `params_version`
- `flags`: {string: bool}
- `params`: {string: number|string|array|object}
- `providers`: {string: string}
- `server_time_utc`

## iOS 使用
- App 启动拉取一次
- Paywall/识别前强刷新（可配置）
