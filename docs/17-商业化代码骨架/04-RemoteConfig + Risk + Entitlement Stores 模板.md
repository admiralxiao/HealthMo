# RemoteConfig / Risk / Entitlement Stores（客户端权威与缓存）

更新时间：2026-01-30

---

## 1) RemoteConfigStore（bootstrap）
```swift
// Core/RemoteConfig/RemoteConfigStore.swift
import Foundation

@MainActor
final class RemoteConfigStore: ObservableObject {
    @Published private(set) var snapshot = RemoteConfigSnapshot.default

    private let api: APIClient
    private let cache: CacheStore

    init(api: APIClient, cache: CacheStore) {
        self.api = api
        self.cache = cache
    }

    func load() async {
        if let cached: RemoteConfigSnapshot = cache.get("bootstrap") {
            snapshot = cached
        }
        do {
            let fresh: RemoteConfigSnapshot = try await api.get("/config/bootstrap")
            snapshot = fresh
            cache.set("bootstrap", fresh, ttl: 3600)
        } catch {
            // 使用缓存/默认值兜底
        }
    }

    func localPaywallViews7d() -> Int {
        cache.getInt("paywall_views_7d") ?? 0
    }
}
```

---

## 2) RiskStore（用于禁折扣/降级）
```swift
// Core/Risk/RiskStore.swift
import Foundation

struct RiskState: Codable, Equatable {
    let riskLevel: String // low/medium/high/critical
    let riskScore: Int
    static let `default` = RiskState(riskLevel: "low", riskScore: 0)
}

@MainActor
final class RiskStore: ObservableObject {
    @Published private(set) var snapshot: RiskState = .default
    private let api: APIClient

    init(api: APIClient) { self.api = api }

    func refresh() async throws -> RiskState {
        let s: RiskState = try await api.get("/risk/state")
        snapshot = s
        return s
    }
}
```

---

## 3) EntitlementStore（客户端唯一权威）
```swift
// Core/Entitlement/EntitlementStore.swift
import Foundation

struct Entitlement: Codable, Equatable {
    let isPremium: Bool
    let productID: String?
    let expiresAt: Date?
    let offerID: String?
}

@MainActor
final class EntitlementStore: ObservableObject {
    @Published private(set) var snapshot = Entitlement(isPremium: false, productID: nil, expiresAt: nil, offerID: nil)

    private let api: APIClient
    private let cache: CacheStore
    private let ttlSec: Int

    init(api: APIClient, cache: CacheStore, ttlSec: Int = 86400) {
        self.api = api
        self.cache = cache
        self.ttlSec = ttlSec
    }

    func refresh(force: Bool) async throws -> Entitlement {
        if !force, let cached: Entitlement = cache.get("entitlement") {
            snapshot = cached
            return cached
        }
        let ent: Entitlement = try await api.get("/entitlement")
        snapshot = ent
        cache.set("entitlement", ent, ttl: ttlSec)
        return ent
    }
}
```

---

## 4) OfferDecisionEngine（集中决策）
```swift
// Features/Monetization/OfferDecisionEngine.swift
import Foundation

enum OfferDecisionEngine {
    static func decide(config: RemoteConfigSnapshot,
                       risk: RiskState,
                       paywallViews7d: Int,
                       timeOnPaywallSec: Int,
                       personalPromiseKey: String?) -> Offer {

        let riskBlocksDiscount = ["medium", "high", "critical"].contains(risk.riskLevel)

        // 默认 A
        var offerID: OfferID = .A

        if !riskBlocksDiscount, config.flags.offerCEnabled, paywallViews7d >= config.params.offerCMinViews7d {
            offerID = .C
        } else if !riskBlocksDiscount, config.flags.offerBEnabled, timeOnPaywallSec >= config.params.offerBTriggerSeconds {
            offerID = .B
        }

        return OfferFactory.make(offerID: offerID,
                                 layoutVariant: config.providers.layoutVariant,
                                 personalPromiseKey: personalPromiseKey)
    }
}
```

---

## 5) 说明
- RemoteConfigSnapshot / OfferFactory 由 Claude CLI 继续补齐
- risk>=medium：禁用 B/C（与 docs/14 对齐）
- 所有 offer 与 plan 必须来自 IAP product IDs（避免 UI 显示与真实价格不一致）

---
