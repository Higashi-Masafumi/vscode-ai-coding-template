# APIクライアント生成（OpenAPI）

## 方針

- 基本はorvalで「クライアント+型」を生成する。
- 型のみ必要な場合はopenapi-typescriptを採用する。
- 生成物は `src/services/api/` に集約し、手書きのAPI型を作らない。
- 生成設定は `orval.config.ts` などに固定し、CIで再生成を検出できるようにする。

## orval運用指針

- 出力先は `src/services/api/` に固定する。
- `schemas` や `models` を別フォルダに分ける場合も `api/` 配下に閉じる。
- fetch/axiosどちらでもよいが、プロジェクトで統一する。
- 生成コードには直接手を入れず、拡張は薄いラッパーで行う。

## openapi-typescript運用指針

- 生成型は `src/services/api/types.ts` に集約する。
- API呼び出しは手書きでも、型参照は必ず生成型から行う。

## 失敗時の扱い

- 生成エラーはCIで検知させる。
- OpenAPI差分はPRでレビュー対象にする。

## 命名と整理

- APIごとに `api/<service-name>/` を切ってもよい。
- 名前が衝突する場合はプレフィックスを付ける。
