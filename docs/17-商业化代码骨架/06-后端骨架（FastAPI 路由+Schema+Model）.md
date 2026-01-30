# 后端骨架（FastAPI）— 商业化相关最小可用代码

更新时间：2026-01-30  
目标：让 Claude CLI 能直接生成可运行的接口与数据库模型（MVP）。

---

## 1) Pydantic Schemas（示例）
```python
# app/schemas/entitlement.py
from pydantic import BaseModel
from datetime import datetime
from typing import Optional

class EntitlementOut(BaseModel):
    is_premium: bool
    product_id: Optional[str] = None
    expires_at: Optional[datetime] = None
    offer_id: Optional[str] = None
```

```python
# app/schemas/iap.py
from pydantic import BaseModel

class IAPTransactionIn(BaseModel):
    transaction_id: str
    original_transaction_id: str
    product_id: str
    offer_id: str
    signed_transaction: str
    app_receipt: str | None = None
```

---

## 2) SQLAlchemy Models（示例）
```python
# app/db/models.py
from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy import String, Boolean, DateTime, Integer
from datetime import datetime
from .base import Base

class Entitlement(Base):
    __tablename__ = "entitlements"
    user_id: Mapped[str] = mapped_column(String, primary_key=True)
    is_premium: Mapped[bool] = mapped_column(Boolean, default=False)
    product_id: Mapped[str | None] = mapped_column(String, nullable=True)
    expires_at: Mapped[datetime | None] = mapped_column(DateTime, nullable=True)
    offer_id: Mapped[str | None] = mapped_column(String, nullable=True)
    updated_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow)

class IAPLog(Base):
    __tablename__ = "iap_logs"
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    user_id: Mapped[str] = mapped_column(String)
    transaction_id: Mapped[str] = mapped_column(String, index=True)
    product_id: Mapped[str] = mapped_column(String)
    offer_id: Mapped[str] = mapped_column(String)
    created_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow)
```

---

## 3) Routes（示例）
```python
# app/api/routes_entitlement.py
from fastapi import APIRouter, Depends
from app.schemas.entitlement import EntitlementOut
from app.services.entitlement_service import get_entitlement_for_user

router = APIRouter()

@router.get("/entitlement", response_model=EntitlementOut)
def entitlement(user=Depends(...)):
    return get_entitlement_for_user(user.id)
```

```python
# app/api/routes_iap.py
from fastapi import APIRouter, Depends
from app.schemas.iap import IAPTransactionIn
from app.schemas.entitlement import EntitlementOut
from app.services.iap_service import verify_and_upsert_entitlement

router = APIRouter()

@router.post("/iap/transactions", response_model=EntitlementOut)
def post_tx(body: IAPTransactionIn, user=Depends(...)):
    return verify_and_upsert_entitlement(user.id, body)
```

---

## 4) MVP 验证策略（占位）
- verify_and_upsert_entitlement：
  - 先记录 IAPLog
  - 调用 Apple 验证（占位实现）
  - 更新 entitlements 表
  - 返回 EntitlementOut

> Apple 验证的具体实现建议放在 docs/订阅与付费上线准备（你已写过）对应章节中。

---
