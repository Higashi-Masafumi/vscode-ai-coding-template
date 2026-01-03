# ディレクトリ構成と依存

## 構成

- `src/app/` ルート/レイアウト/ルーティング設定のみを置く。
- `src/modules/` ドメイン/機能単位のUIとロジックをまとめる。
- `src/components/` 再利用可能なUIコンポーネントを置く。
- `src/hooks/` 再利用可能なカスタムフックを置く。
- `src/providers/` Provider群（AuthProviderなど）を置く。
- `src/services/` APIクライアントと通信ロジックを置く。
- `src/lib/` 汎用ユーティリティ・ヘルパを置く。
- `src/styles/` デザイントークンやグローバルスタイルを置く。

## 依存ルール

- `modules` は `components` / `hooks` / `services` / `lib` に依存してよい。
- `components` と `hooks` は `modules` に依存しない。
- `providers` は `services` / `lib` に依存してよいがUI層に依存しない。
- `services` はUI層に依存しない。

## modulesの構成指針

- `modules/<domain>/` は「その機能の画面/UI/ロジック」を完結させる。
- 入口は `modules/<domain>/index.ts` に集約し、外部公開APIを明確化する。
- 例: `modules/auth/`, `modules/user/`, `modules/billing/`
- 画面単位の実装は `modules/<domain>/pages/` に集約する。

## modules内の推奨サブ構成

- `modules/<domain>/pages/` ルートに紐づく画面のみを置く。
- `modules/<domain>/components/` そのドメイン専用の部品のみを置く。
- `modules/<domain>/hooks/` そのドメイン専用のフックのみを置く。
- `modules/<domain>/models/` そのドメインの型・スキーマ・変換を置く。
- `modules/<domain>/services/` そのドメイン固有のAPI呼び出しを置く。
- `modules/<domain>/routes.tsx` そのドメインのルーティング定義をまとめる。

## 境界の判断基準

- 他ドメインでも使う → `src/components` / `src/hooks` / `src/lib`
- 特定ドメイン限定 → `modules/<domain>/...`
- API通信の再利用 → `src/services`（ドメイン固有なら `modules/<domain>/services`）
- UI以外の共有ロジック → `src/lib`（React依存を持たせない）

## 命名ルール

- 具体名で揃える（例: `UserProfileCard`, `useBillingSummary`）
- 省略語はプロジェクトで統一し、拡張しない（例: `cfg` は使わない）
- ディレクトリ名は名詞、ファイル名は責務が明確な名詞/動詞にする

## 例: 最小構成スケルトン

```
src/
  app/
    routes.tsx
  modules/
    auth/
      pages/
      components/
      hooks/
      services/
      models/
      routes.tsx
      index.ts
  components/
  hooks/
  providers/
  services/
    api/
  lib/
  styles/
```
