# iOS 代码骨架（Coordinator + ViewModel + Models）

更新时间：2026-01-30  
说明：以下为“可复制”骨架片段（伪实现），用于 Claude CLI 生成可编译代码。

---

## 1) Flow State
```swift
// Features/Monetization/PaywallFlowState.swift
import Foundation

enum PaywallFlowState: Equatable {
    case idle
    case configReady
    case onboarding
    case valuePreview
    case proof
    case paywall(offer: Offer)
    case purchasing(plan: Plan)
    case purchased(entitlement: Entitlement)
    case skipped
    case error(message: String)
}
```

---

## 2) Coordinator（路由）
```swift
// Features/Monetization/PaywallCoordinator.swift
import SwiftUI

@MainActor
final class PaywallCoordinator: ObservableObject {
    @Published private(set) var state: PaywallFlowState = .idle

    let remoteConfig: RemoteConfigStore
    let entitlement: EntitlementStore
    let risk: RiskStore
    let analytics: Analytics
    let purchase: PurchaseManager

    init(remoteConfig: RemoteConfigStore,
         entitlement: EntitlementStore,
         risk: RiskStore,
         analytics: Analytics,
         purchase: PurchaseManager) {
        self.remoteConfig = remoteConfig
        self.entitlement = entitlement
        self.risk = risk
        self.analytics = analytics
        self.purchase = purchase
    }

    func start() async {
        state = .configReady
        // 可在此预取 entitlement/risk
        _ = try? await entitlement.refresh(force: false)
        _ = try? await risk.refresh()
        state = .onboarding
    }

    func toValuePreview() { state = .valuePreview }
    func toProof() { state = .proof }

    func toPaywall(personalPromiseKey: String) {
        let offer = OfferDecisionEngine.decide(
            config: remoteConfig.snapshot,
            risk: risk.snapshot,
            paywallViews7d: remoteConfig.localPaywallViews7d(),
            timeOnPaywallSec: 0,
            personalPromiseKey: personalPromiseKey
        )
        state = .paywall(offer: offer)
        analytics.track(.paywallView(offer: offer))
    }

    func skip() {
        state = .skipped
        analytics.track(.paywallNotNow)
    }
}
```

---

## 3) Models（Offer/Plan）
```swift
// Features/Monetization/MonetizationModels.swift
import Foundation

enum OfferID: String, Codable { case A, B, C }

struct Plan: Identifiable, Codable, Equatable {
    let id: String          // "weekly_9_99"
    let displayPrice: String // "$9.99/week" from StoreKit product
    let period: String      // "week" / "year"
    let isBestValue: Bool
}

struct Offer: Codable, Equatable {
    let id: OfferID
    let layoutVariant: String
    let plans: [Plan]
    let defaultPlanID: String
    let personalPromiseKey: String?
}
```

---

## 4) ViewModel（Paywall）
```swift
// Features/Monetization/ViewModels/PaywallViewModel.swift
import Foundation

@MainActor
final class PaywallViewModel: ObservableObject {
    @Published var selectedPlanID: String
    @Published var isLoading = false
    @Published var errorMessage: String?

    let offer: Offer
    let analytics: Analytics
    let purchase: PurchaseManager
    let entitlement: EntitlementStore

    init(offer: Offer,
         analytics: Analytics,
         purchase: PurchaseManager,
         entitlement: EntitlementStore) {
        self.offer = offer
        self.analytics = analytics
        self.purchase = purchase
        self.entitlement = entitlement
        self.selectedPlanID = offer.defaultPlanID
    }

    func selectPlan(_ id: String) {
        selectedPlanID = id
        analytics.track(.paywallPlanSelected(offerID: offer.id, planID: id))
    }

    func buy() async {
        isLoading = true
        defer { isLoading = false }

        analytics.track(.paywallCTAClick(offerID: offer.id, planID: selectedPlanID))

        do {
            try await purchase.purchase(productID: selectedPlanID, offerID: offer.id.rawValue)
            let ent = try await entitlement.refresh(force: true)
            if ent.isPremium {
                analytics.track(.purchaseSuccess(offerID: offer.id, planID: selectedPlanID))
            } else {
                errorMessage = "Subscription not active yet."
                analytics.track(.purchaseFail(offerID: offer.id, planID: selectedPlanID, reason: "entitlement_inactive"))
            }
        } catch {
            errorMessage = error.localizedDescription
            analytics.track(.purchaseFail(offerID: offer.id, planID: selectedPlanID, reason: "storekit_or_network"))
        }
    }

    func restore() async {
        isLoading = true
        defer { isLoading = false }
        do {
            try await purchase.restore()
            _ = try await entitlement.refresh(force: true)
            analytics.track(.restoreSuccess)
        } catch {
            analytics.track(.restoreFail(reason: error.localizedDescription))
        }
    }
}
```

---
