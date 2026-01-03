# ディレクトリ構成と実装テンプレ

## 例: ディレクトリ構成
```
app/
  domain/
    entity/
    repository/
    gateway/
    error/
    service/
  application/
    usecase/
    service/
  infrastructure/
    repository/
    gateway/
    di/
    middleware/
  controller/
    router/
    schemas/
    factory/
```

## 例: domain entity（Pydantic）
```python
# app/domain/entity/user.py
from pydantic import BaseModel, EmailStr

class User(BaseModel):
    id: str
    email: EmailStr

    def change_email(self, new_email: str) -> "User":
        return User(id=self.id, email=new_email)
```

## 例: repository interface
```python
# app/domain/repository/user_repository.py
from abc import ABC, abstractmethod
from app.domain.entity.user import User

class UserRepository(ABC):
    @abstractmethod
    def get(self, user_id: str) -> User | None:
        raise NotImplementedError

    @abstractmethod
    def save(self, user: User) -> None:
        raise NotImplementedError
```

## 例: application usecase
```python
# app/application/usecase/change_user_email.py
from app.domain.repository.user_repository import UserRepository

class ChangeUserEmail:
    def __init__(self, users: UserRepository) -> None:
        self._users = users

    def execute(self, user_id: str, new_email: str) -> None:
        user = self._users.get(user_id)
        if user is None:
            raise ValueError("user_not_found")
        self._users.save(user.change_email(new_email))
```

## 例: controller router
```python
# app/controller/router/users.py
from fastapi import APIRouter, Depends
from app.controller.schemas.user import ChangeEmailRequest
from app.controller.factory.usecase import get_change_user_email

router = APIRouter()

@router.post("/users/{user_id}/email")
async def change_email(
    user_id: str,
    req: ChangeEmailRequest,
    uc = Depends(get_change_user_email),
):
    uc.execute(user_id, req.email)
    return {"status": "ok"}
```

## 例: infrastructure repository
```python
# app/infrastructure/repository/user_repository.py
from app.domain.entity.user import User
from app.domain.repository.user_repository import UserRepository

class SqlUserRepository(UserRepository):
    def __init__(self, session) -> None:
        self._session = session

    def get(self, user_id: str) -> User | None:
        # ORM取得 -> domain entity へ変換
        raise NotImplementedError

    def save(self, user: User) -> None:
        # domain entity -> ORMへ変換
        raise NotImplementedError
```
