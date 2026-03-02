# ALICE-SDF-Experiment — Claude Code 設定

## プロジェクト概要

ALICE SDFメタバースのスタンドアロン実験リポジトリ。純粋なHTML/Canvas/WebGL/WebAudioで構成（ビルドシステムなし）。

| 項目 | 値 |
|------|-----|
| リポジトリ | `ext-sakamoro/ALICE-SDF-Experiment` |
| リモート | `origin` (`https://github.com/ext-sakamoro/ALICE-SDF-Experiment.git`) |
| ブランチ | `main` |
| 技術スタック | HTML, WebGL (GLSL fragment shader), JavaScript |
| ビルド | なし（静的HTMLのみ） |
| デプロイ | CloudFlare Pages (`alice-sdf-experiment.pages.dev`) |

## コーディングルール

- コミットメッセージ: 日本語
- コード内コメント: 日本語
- 署名禁止: Co-Authored-By / Generated with Claude Code は一切追加しない
- 作成者名: `Moroya Sakamoto`

## ファイル構成

| ファイル | 内容 |
|---------|------|
| `index.html` | UIシェル（Canvas, ナビ, ミニマップ, ラベル） |
| `alice-kernel.js` | カメラ, 移動, テレポート, 天候, エントロピー, WebGLセットアップ, JS側SDF物理 |
| `alice-universe.glsl` | フラグメントシェーダー: SDF全景, バイオーム, マテリアル, PBR, 空, ポスト処理 |

## CloudFlare Pages デプロイ

### デプロイコマンド

```bash
npx wrangler pages deploy . --project-name=alice-sdf-experiment --commit-message="ASCII message here"
```

### 日本語コミットメッセージ拒否問題

CloudFlare Pages APIは日本語（非ASCII）のコミットメッセージを拒否する（エラーコード 8000111: "Invalid commit message, it must be a valid UTF-8 string"）。

**回避策**: `--commit-message` フラグで英語ASCIIメッセージを明示的に指定する。Git側のコミットメッセージは日本語のまま問題ない。CloudFlareデプロイ時のみ英語メッセージを渡す。

## SDF実装規定

本プロジェクトは以下のSDF構築基準に従う。詳細は各ファイルを参照:
- `~/CLAUDE.md` — 真理の地形法, SDF メタバース四法, TDR防止規定
- `~/ALICE-SDF-LAWS.md` — SDF構築基準全文
- `~/alicelaw-net/CLAUDE.md` — Windows TDR制限の具体的数値

### TDR制限 (Windows ANGLE対策)

- Raymarchステップ: 64以下
- FBM: 3反復以下 (GLSL/JS共通)
- SSR: 16ステップ以下
- map_lite(): IFS/Ring/Crystal/Meteorを省略した最小構成
- dWarp(): 1パス以下
- フロアバンプ: vnoise(2D)前方差分のみ (vnoise3禁止)
- Sky装飾ノイズ: vnoise のみ (fbm禁止)

### Division Exorcism / Zero-Branching

- 全除算 → `rcp()`パターン / `inversesqrt()` / 定数乗算
- 全条件分岐 → `step()` + `mix()` + `smoothstep()`

## GLSL 注意事項

### 前方参照不可

WebGL 1.0 (GLSL ES 1.00) は関数の前方宣言・前方参照をサポートしない。**関数は使用箇所より前に定義すること**。共用関数（例: `voronoi2`）を複数セクションから呼ぶ場合、最も早い使用箇所より上に配置する。

### 予約語

GLSL予約語を変数名に使わないこと。よく踏む例: `flat`（補間修飾子）, `input`, `output`, `sample`, `patch`
