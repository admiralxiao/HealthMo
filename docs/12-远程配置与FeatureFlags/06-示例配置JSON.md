# 06-示例配置JSON

更新时间：2026-01-30

```json
{
  "bootstrap_version": "2026.01.30",
  "params_version": "2026.01.30",
  "server_time_utc": "2026-01-30T00:00:00Z",
  "flags": {
    "kill_switch.all": false,
    "kill_switch.payments": false,
    "kill_switch.vision": false,
    "feature.paywall.enabled": true,
    "feature.paywall.offer_B": true,
    "feature.paywall.offer_C": false,
    "feature.vision.food_enabled": true
  },
  "providers": {
    "paywall.layout_variant": "oura_v1",
    "paywall.offer_default": "A"
  },
  "params": {
    "bmr_formula": "mifflin_st_jeor",
    "tef_enabled": false,
    "met.sleep": 0.95,
    "met.still": 1.2,
    "paywall.offer_trigger.seconds": 8,
    "risk.discount.disable_level_ge": "medium"
  }
}
```
```
