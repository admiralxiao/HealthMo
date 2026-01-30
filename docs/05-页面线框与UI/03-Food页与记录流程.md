# 03-Food 页与记录流程

更新时间：2026-01-30

```mermaid
flowchart TD
  A[Food Home\nToday intake + macros] --> B[Add Food]
  B --> C{选择方式}
  C -->|Photo| D[Camera]
  C -->|Barcode| E[Barcode Scanner]
  C -->|Manual| F[Search / Manual Entry]
  D --> G[/vision/food -> structured items]
  E --> H[/nutrition/barcode -> nutrition]
  F --> I[/nutrition/search -> pick item]
  G --> J[Edit & Confirm]
  H --> J
  I --> J
  J --> K[Save FoodLog]
```


## UI 关键点
- 默认只展示：总热量 + P/F/C
- item 列表可编辑：份量、克重、替换食品
- 每次编辑都实时更新 totals
