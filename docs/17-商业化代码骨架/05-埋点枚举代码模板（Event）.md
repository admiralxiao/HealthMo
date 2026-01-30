# 埋点枚举代码模板（Swift）

更新时间：2026-01-30  
目标：让商业化埋点可统一、可静态检查，避免漏字段。

---

## 1) Event 定义
```swift
// Core/Analytics/Event.swift
import Foundation

enum EventName: String {
    // onboarding
    case onboardingStart = "onboarding_start"
    case goalSelected = "goal_selected"
    case onboardingQuestionAnswered = "onboarding_question_answered"
    case onboardingComplete = "onboarding_complete"

    // paywall
    case paywallView = "paywall_view"
    case paywallOfferShown = "paywall_offer_shown"
    case paywallPlanSelected = "paywall_plan_selected"
    case paywallCTAClick = "paywall_cta_click"
    case paywallNotNow = "paywall_not_now"
    case paywallRestoreClick = "paywall_restore_click"

    // purchase
    case purchaseStart = "purchase_start"
    case purchaseSuccess = "purchase_success"
    case purchaseFail = "purchase_fail"
    case restoreSuccess = "restore_success"
    case restoreFail = "restore_fail"

    // entitlement
    case entitlementFetch = "entitlement_fetch"
    case entitlementActive = "entitlement_active"
    case entitlementInactive = "entitlement_inactive"
    case entitlementMismatch = "entitlement_mismatch"
    case premiumApi403 = "premium_api_403"
}

struct Event {
    let name: EventName
    let ts: Date
    let props: [String: AnyCodable]
}
```

---

## 2) 事件工厂（强制带 offer_id / plan_id）
```swift
// Core/Analytics/EventPayloads.swift
import Foundation

enum AppEvents {
    static func paywallView(offer: Offer) -> Event {
        Event(
            name: .paywallView,
            ts: Date(),
            props: [
                "offer_id": AnyCodable(offer.id.rawValue),
                "layout_variant": AnyCodable(offer.layoutVariant),
                "default_plan": AnyCodable(offer.defaultPlanID),
                "personal_promise_key": AnyCodable(offer.personalPromiseKey ?? "")
            ]
        )
    }

    static func paywallPlanSelected(offerID: OfferID, planID: String) -> Event {
        Event(name: .paywallPlanSelected, ts: Date(),
              props: ["offer_id": AnyCodable(offerID.rawValue), "plan_id": AnyCodable(planID)])
    }

    static func paywallCTAClick(offerID: OfferID, planID: String) -> Event {
        Event(name: .paywallCTAClick, ts: Date(),
              props: ["offer_id": AnyCodable(offerID.rawValue), "plan_id": AnyCodable(planID)])
    }

    static func purchaseSuccess(offerID: OfferID, planID: String) -> Event {
        Event(name: .purchaseSuccess, ts: Date(),
              props: ["offer_id": AnyCodable(offerID.rawValue), "plan_id": AnyCodable(planID)])
    }

    static func purchaseFail(offerID: String, planID: String, reason: String) -> Event {
        Event(name: .purchaseFail, ts: Date(),
              props: ["offer_id": AnyCodable(offerID), "plan_id": AnyCodable(planID), "reason": AnyCodable(reason)])
    }

    static let paywallNotNow = Event(name: .paywallNotNow, ts: Date(), props: [:])
    static let restoreSuccess = Event(name: .restoreSuccess, ts: Date(), props: [:])
    static func restoreFail(reason: String) -> Event {
        Event(name: .restoreFail, ts: Date(), props: ["reason": AnyCodable(reason)])
    }
}
```

---

## 3) Analytics 发送接口
```swift
// Core/Analytics/Analytics.swift
import Foundation

protocol Analytics {
    func track(_ event: Event)
}

final class AnalyticsClient: Analytics {
    private let api: APIClient
    init(api: APIClient) { self.api = api }

    func track(_ event: Event) {
        // MVP：可以先本地队列 + 批量 POST /events
    }
}
```

---
