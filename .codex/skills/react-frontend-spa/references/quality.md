# Lint/Format/型安全

## 方針

- ESLintとPrettierを必ず導入する。
- `@typescript-eslint` と `eslint-config-prettier` を前提にする。
- 型チェックは `tsc --noEmit` をCIで実行できる形にする。

## ルール運用

- ルールは「例外を増やさない」前提で設計する。
- `eslint-disable` は理由コメント必須にする。
- 重大なlint違反はCIで落とす。

## フォーマット運用

- Prettierは設定ファイルに固定し、エディタ拡張で自動適用する。
- フォーマッタとlintの責務を混在させない。
