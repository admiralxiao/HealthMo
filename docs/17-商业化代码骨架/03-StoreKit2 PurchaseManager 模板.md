# StoreKit2 PurchaseManager 模板（可编译实现方向）

更新时间：2026-01-30  
要求：购买后必须 POST /iap/transactions → 后端验证 → 返回 entitlement → 再解锁。

---

## 1) PurchaseManager 接口
```swift
// Features/Monetization/StoreKit/PurchaseManager.swift
import Foundation
import StoreKit

protocol PurchaseManaging {
    func loadProducts(productIDs: [String]) async throws -> [Product]
    func purchase(productID: String, offerID: String) async throws
    func restore() async throws
    func startListeningForUpdates() async
}

@MainActor
final class PurchaseManager: PurchaseManaging {
    private let api: APIClient
    private let analytics: Analytics

    init(api: APIClient, analytics: Analytics) {
        self.api = api
        self.analytics = analytics
    }

    func loadProducts(productIDs: [String]) async throws -> [Product] {
        try await Product.products(for: productIDs)
    }

    func purchase(productID: String, offerID: String) async throws {
        let products = try await Product.products(for: [productID])
        guard let product = products.first else { throw StoreKitError.unknown }

        let result = try await product.purchase()

        switch result {
        case .success(let verification):
            let transaction = try verification.payloadValue
            // 1) 上报后端验证
            try await postTransaction(transaction, offerID: offerID)
            // 2) 完成交易
            await transaction.finish()
        case .userCancelled:
            throw StoreKitError.userCancelled
        case .pending:
            throw StoreKitError.pending
        @unknown default:
            throw StoreKitError.unknown
        }
    }

    func restore() async throws {
        // StoreKit2 恢复通常通过 sync + 监听 updates 实现
        try await AppStore.sync()
        // 同步后可尝试从最新交易/receipt 上报
        // MVP：让 entitlement refresh 触发后端确认
    }

    func startListeningForUpdates() async {
        for await verification in Transaction.updates {
            do {
                let transaction = try verification.payloadValue
                try await postTransaction(transaction, offerID: "update")
                await transaction.finish()
            } catch {
                // 记录错误但不阻塞
                analytics.track(.purchaseFail(offerID: "update", planID: "unknown", reason: "transaction_updates_error"))
            }
        }
    }

    private func postTransaction(_ transaction: Transaction, offerID: String) async throws {
        let payload = IAPTransactionPayload(
            transactionID: String(transaction.id),
            originalTransactionID: String(transaction.originalID),
            productID: transaction.productID,
            offerID: offerID,
            // 可选：JWS/receipt
            signedTransaction: transaction.jwsRepresentation
        )
        _ = try await api.post("/iap/transactions", body: payload)
    }
}

enum StoreKitError: LocalizedError {
    case userCancelled
    case pending
    case unknown

    var errorDescription: String? {
        switch self {
        case .userCancelled: return "Cancelled"
        case .pending: return "Pending"
        case .unknown: return "Unknown error"
        }
    }
}
```

---

## 2) IAPTransactionPayload
```swift
// Core/Networking/Models/IAPTransactionPayload.swift
import Foundation

struct IAPTransactionPayload: Codable {
    let transactionID: String
    let originalTransactionID: String
    let productID: String
    let offerID: String
    let signedTransaction: String
}
```

---

## 3) 注意事项（MVP）
- 只处理 verified transaction（payloadValue）
- userCancelled 不弹“错误弹窗”，只回到 paywall
- entitlement 不生效时不要解锁（防破解/不同步）
- restore：MVP 可以先让 AppStore.sync() + entitlement.refresh(force:true)

---
