# 07-订阅StoreKit2

更新时间：2026-01-30

- StoreKit2 purchase → POST /iap/transactions → GET /entitlement 才解锁
- 监听 Transaction.updates（续订/退款/过期）
- SKU：weekly_9_99 / yearly_59_99 / weekly_4_99 / weekly_1_99
