# Claude CLI 生成指令模板（直接开干）

更新时间：2026-01-30  
目标：把本章骨架转化为“可编译 iOS 工程 + 可运行 FastAPI”。

---

## 1) iOS：先生成商业化骨架文件
示例（在 iOS repo 根目录）：
```bash
claude -p "根据 docs/17-商业化代码骨架/02-iOS代码骨架（Coordinator+VM+Models）.md 生成对应 Swift 文件，补齐缺失类型（RemoteConfigSnapshot/OfferFactory/APIClient/CacheStore/AnyCodable），并确保 iOS 15.6+ 可编译。输出到 Features/Monetization 与 Core/ 目录。"
```

## 2) iOS：生成 PaywallView（SwiftUI）
```bash
claude -p "按 docs/15 与 docs/16 的 Oura 风 Paywall 规范，生成 PaywallView.swift：包含 Value cards、Plan picker、CTA、Fine print、Restore。Plan picker 使用 Offer.plans 渲染。"
```

## 3) iOS：生成 StoreKitProducts 映射
```bash
claude -p "生成 StoreKitProducts.swift：列出 IAP product IDs（weekly_9_99, yearly_59_99, weekly_4_99, weekly_1_99），并提供根据 OfferID 返回默认可见 plans 的函数。"
```

## 4) Backend：生成 FastAPI 骨架
（在 backend repo 根目录）
```bash
claude -p "根据 docs/17-商业化代码骨架/06-后端骨架（FastAPI 路由+Schema+Model）.md 生成 FastAPI 路由、schema、model 与 service 占位实现，并给出启动命令与本地测试 curl。"
```

## 5) 联调：一次跑通
```bash
claude -p "生成一份联调 checklist：iOS purchase -> POST /iap/transactions -> GET /entitlement -> UI 解锁。包含常见错误处理与日志点。"
```

---

## 6) 输出验收（DoD）
- iOS 工程可编译，商业化 flow 可跑通（即使后端为 mock）
- 后端 FastAPI 可启动并返回 mock entitlement
- purchase/restore 路径都有埋点调用点
