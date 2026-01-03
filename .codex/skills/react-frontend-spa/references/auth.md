# 認証

## 方針

- `AuthProvider` を作成し、`useAuth` で状態/操作を公開する。
- `useAuth` は「現在ユーザー」「認証状態」「ログイン/ログアウト/リフレッシュ」を提供する。
- トークンの保存先は要件に合わせるが、扱いは `AuthProvider` に閉じる。
- ルートガードは `ProtectedRoute` で集中管理し、各ページに分散させない。

## 状態設計

- `AuthState` は最小限にする（`user`, `isAuthenticated`, `isLoading` など）。
- `user` はAPIの型をそのまま使うか、表示用に変換した型を使う。
- 認証フローの状態（初期化中/更新中/失敗）は `status` で一元化する。

## Providerの責務

- 初期化時に `refresh` を実行し、認証状態を確定する。
- トークン更新の失敗は `logout` に集約する。
- ローカルストレージ等の永続化はProvider内に閉じる。
- APIクライアントへのトークン注入はProviderから行う。

## useAuthの責務

- UIは `useAuth` のみを参照し、ストレージに触れない。
- 返却値の形は安定させ、コンポーネント側で分岐を増やさない。

## ルートガード

- `ProtectedRoute` は `isLoading` 中はローディングを返し、未認証ならリダイレクトする。
- 認証不要のルートは `PublicRoute` を用意してもよい。

## エラー設計

- 認証エラーは `AuthError` などにまとめ、UI側で判定しやすくする。
- 401はProviderでハンドリングし、再ログイン導線を返す。

## 最小構成のイメージ

```ts
// src/providers/auth.tsx
export type AuthState = {
  user: User | null
  isAuthenticated: boolean
}

export const AuthProvider: React.FC<PropsWithChildren> = ({ children }) => {
  // ... init/refresh/login/logout
}

export const useAuth = () => useContext(AuthContext)
```
