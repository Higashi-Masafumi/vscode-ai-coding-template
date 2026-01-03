# セッション管理とUoW

## セッション方針
- FastAPIの `Depends` で1リクエスト1セッション
- セッション生成/破棄はcontrollerかDI層に寄せる
- usecaseはセッションの生存期間を管理しない
- `yield` 依存でセッションを供給し、`try/finally` で確実にcloseする

## UoW（明示的トランザクション）
- usecase内で複数リポジトリをまたぐ場合に導入
- インフラ層でUoW実装、applicationは抽象に依存

### 形（例）
- `UnitOfWork` interface: `__enter__`, `__exit__`, `commit`, `rollback`, `session`
- usecase: `with uow:` で囲み、成功時commit
- `AbstractContextManager` / `AbstractAsyncContextManager` を継承する実装を基本とする

## 推奨フロー
1) controllerでDependsによりUoW/Session注入
2) usecaseでトランザクション境界を明確化
3) domain例外はapplicationでハンドリングしcontrollerでHTTP変換

## 具体例（Depends + yield セッション）
```python
# app/infrastructure/di/db.py
from collections.abc import Generator
from sqlalchemy.orm import Session

from app.infrastructure.db import SessionLocal

def get_session() -> Generator[Session, None, None]:
    session = SessionLocal()
    try:
        yield session
    finally:
        session.close()
```

## 具体例（UoW: sync）
```python
# app/application/uow.py
from abc import ABC, abstractmethod
from contextlib import AbstractContextManager
from sqlalchemy.orm import Session

class UnitOfWork(ABC, AbstractContextManager):
    @property
    @abstractmethod
    def session(self) -> Session:
        raise NotImplementedError

    @abstractmethod
    def commit(self) -> None:
        raise NotImplementedError

    @abstractmethod
    def rollback(self) -> None:
        raise NotImplementedError
```

```python
# app/infrastructure/uow.py
from sqlalchemy.orm import Session
from app.application.uow import UnitOfWork

class SqlAlchemyUnitOfWork(UnitOfWork):
    def __init__(self, session: Session) -> None:
        self._session = session

    def __enter__(self) -> "SqlAlchemyUnitOfWork":
        return self

    def __exit__(self, exc_type, exc, tb) -> None:
        if exc_type is None:
            self.commit()
        else:
            self.rollback()

    @property
    def session(self) -> Session:
        return self._session

    def commit(self) -> None:
        self._session.commit()

    def rollback(self) -> None:
        self._session.rollback()
```

## 具体例（UoW: async）
```python
# app/application/uow_async.py
from abc import ABC, abstractmethod
from contextlib import AbstractAsyncContextManager
from sqlalchemy.ext.asyncio import AsyncSession

class AsyncUnitOfWork(ABC, AbstractAsyncContextManager):
    @property
    @abstractmethod
    def session(self) -> AsyncSession:
        raise NotImplementedError

    @abstractmethod
    async def commit(self) -> None:
        raise NotImplementedError

    @abstractmethod
    async def rollback(self) -> None:
        raise NotImplementedError
```

```python
# app/infrastructure/uow_async.py
from sqlalchemy.ext.asyncio import AsyncSession
from app.application.uow_async import AsyncUnitOfWork

class SqlAlchemyAsyncUnitOfWork(AsyncUnitOfWork):
    def __init__(self, session: AsyncSession) -> None:
        self._session = session

    async def __aenter__(self) -> "SqlAlchemyAsyncUnitOfWork":
        return self

    async def __aexit__(self, exc_type, exc, tb) -> None:
        if exc_type is None:
            await self.commit()
        else:
            await self.rollback()

    @property
    def session(self) -> AsyncSession:
        return self._session

    async def commit(self) -> None:
        await self._session.commit()

    async def rollback(self) -> None:
        await self._session.rollback()
```

## 具体例（DependsでUoW供給）
```python
# app/infrastructure/di/uow.py
from collections.abc import Generator
from fastapi import Depends
from sqlalchemy.orm import Session

from app.infrastructure.di.db import get_session
from app.infrastructure.uow import SqlAlchemyUnitOfWork

def get_uow(session: Session = Depends(get_session)) -> Generator[SqlAlchemyUnitOfWork, None, None]:
    uow = SqlAlchemyUnitOfWork(session)
    yield uow
```
