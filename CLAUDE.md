# Eternal Return Documentation Project

## プロジェクト概要

### 目的

**自分自身のEternal Return上達**が一番の目的。

### フォーカス

- **ミクロ（戦闘操作）** の上達に注力
- マクロ（ルート・立ち回り）ではなく、戦闘時の動きを改善したい

### 整理したい内容

1. **チームファイト理論（キャラ非依存）** → `00_Overall/eternal_return_guide_v1_2.md`（**作成中**）
   - 3線（ライン）の概念：1線/2線/3線のポジショニング
   - アグロピンポン：視線の引きつけ・交代
   - フォーカス原則：誰を殴るか
   - 構成パターン：仕掛ける構成 vs 受ける構成
   - 構図判断：手の長さ、進入タイミング
   - ※ このガイドを完璧にすることが最終目標

2. **ロール別の戦闘基礎**
   - 全キャラ個別に最適化するのは現実的でないため、ロール（役割）ごとにカテゴライズ
   - T（タンク）/ TB（タンブルーザー）/ B（ブルーザー）/ A（アサシン）/ SD（スキル型）/ AD（平打ち型）/ S（サポート）

3. **キャラ別のミクロ**（参考情報として収集）
   - 先行入力、スキル回し、コンボ等

### 情報収集

韓国の上位プレイヤー（主にTiaメイン）のYouTube・Discord等から情報を収集・翻訳。
収集した情報は上記ガイドの改善・完成に活用する。

## ディレクトリ構造

```
er_doc/
├── CLAUDE.md                    # このファイル
├── 99_Scripts/                  # 共通スクリプト
│   ├── youtube/                 # YouTube収集ツール
│   ├── discord/                 # Discord収集ツール
│   └── translation/             # 翻訳モジュール
└── 99_Resource/                 # チャンネル別データ
    ├── Gmiho/
    ├── Magic_Daniel_ER/
    ├── Nongroot/
    ├── Tzneu/
    ├── Yeongman/
    └── ...
```

---

## YouTube チャンネル収集手順

### 新規チャンネル追加フロー

```
1. セットアップ  →  setup_channel.py
2. 設定追加      →  各スクリプトのCHANNEL_CONFIGに追加
3. コメント取得  →  fetch_channel_comments.py
4. 字幕取得      →  download_subtitles.py
5. コメント翻訳  →  translate_owner_comments.py
6. 字幕翻訳      →  translate_subtitles.py
```

### Step 1: チャンネル情報を調べる

1. YouTubeでチャンネルページを開く
2. チャンネルID取得: ページソースで `channelId` を検索

### Step 2: セットアップ

```bash
cd 99_Scripts/youtube
python setup_channel.py \
    --name "ChannelName" \
    --key "channel_key" \
    --id "UCxxxxxxxxxxxxxxxx" \
    --handle "@ChannelHandle" \
    --desc "チャンネルの説明"
```

### Step 3: 各スクリプトに設定追加

以下のファイルの `CHANNEL_CONFIG` と argparse `choices` に追加:
- `fetch_channel_comments.py`
- `download_subtitles.py`
- `translate_owner_comments.py`
- `translate_subtitles.py`

```python
CHANNEL_CONFIG = {
    # 既存エントリ...
    "channel_key": {
        "name": "ChannelName",
        "channel_id": "UCxxxxxxxxxxxxxxxx",
        "output_dir": "/app/output/ChannelName",  # fetch用
        # または
        "display_name": "ChannelName",  # translate用
    }
}
```

### Step 4: コメント取得

```bash
# APIキーは 99_Scripts/youtube/.env に設定
# YOUTUBE_API_KEY=your-api-key

# 実行
python fetch_channel_comments.py --channel channel_key
```

### Step 5: 字幕取得

```bash
python download_subtitles.py --channel channel_key
```

### Step 6: コメント翻訳

```bash
python translate_owner_comments.py --channel channel_key
```

### Step 7: 字幕翻訳

```bash
python translate_subtitles.py --channel channel_key
```

### 出力ディレクトリ構造

```
99_Resource/{ChannelName}/
├── README.md
├── data/                              # 生データ
│   ├── {ChannelName}_api_data.json    # APIデータ
│   └── subtitles/                     # 原文字幕（韓国語）
│       └── {VIDEO_ID}.ko.srt
└── output/                            # 翻訳済みデータ
    ├── video_mapping.json             # 動画ID⇔ファイル名マッピング
    ├── owner_comments_index_ja.md     # コメント一覧（字幕との対応付き）
    ├── owner_comments_ja/             # 翻訳済みコメント
    │   └── {yyyymmdd}.md              # 日付ベースのファイル名
    ├── subtitles_index_ja.md          # 字幕一覧（コメントとの対応付き）
    └── subtitles_ja/                  # 翻訳済み字幕
        ├── {yyyymmdd}.ja.srt          # SRT形式（動画プレーヤー用）
        └── {yyyymmdd}.ja.md           # Markdown形式（閲覧用）
```

### ファイル命名規則

翻訳済みファイルは動画IDではなく、投稿日ベースのファイル名を使用：

- **形式**: `yyyymmdd` （例: `20260203`）
- **同日複数投稿**: `yyyymmdd_1`, `yyyymmdd_2` のように連番を付与
- **マッピング**: `video_mapping.json` で動画IDとファイル名の対応を管理

**インデックスファイルの特徴**:
- 投稿日順（新しい順）でソート
- 字幕とコメントの相互リンクを表示
- 各動画に対して字幕・コメントの有無がわかる

### 既存ファイルの移行

旧形式（video_id）から新形式（yyyymmdd）への移行手順：

```bash
# 1. 移行内容を確認（実際の変更なし）
python migrate_to_date_format.py --channel ldy818 --dry-run

# 2. 移行を実行
python migrate_to_date_format.py --channel ldy818
```

移行スクリプトの動作：
1. 既存の翻訳ファイルをスキャン
2. APIデータから投稿日を取得し、新しいファイル名を決定
3. ファイルをリネーム
4. `video_mapping.json` を作成・更新

**注意**: 移行後も元のファイルは削除されません（リネームのみ）。

### 差分更新

翻訳スクリプトは差分更新に対応しています：

- `video_mapping.json` で翻訳済みの動画を管理
- 字幕翻訳: `has_subtitle: true` のものはスキップ
- コメント翻訳: `has_comment: true` のものはスキップ
- インデックスは毎回全件で再生成（字幕・コメントの対応を更新）

```bash
# 新規のみ翻訳（差分更新）
python translate_subtitles.py --channel ldy818
python translate_owner_comments.py --channel ldy818

# 特定の動画のみ再翻訳
python translate_subtitles.py --channel ldy818 --video VIDEO_ID
```

---

## 翻訳辞書

### 辞書ファイル

`99_Scripts/translation/ko_ja_dictionary.json`

### 辞書に追加すべき用語

1. **ゲーム固有用語**: スキル名、アイテム名、システム用語
2. **キャラクター名**: プレイヤー名、NPC名
3. **略語・スラング**: ゲームコミュニティで使われる略語
4. **誤訳されやすい用語**: 機械翻訳で意味が変わってしまう用語

### 使い方

```python
from translator import KoJaTranslator
translator = KoJaTranslator()
result = translator.translate("韓国語テキスト")
```

---

## 登録済みチャンネル

| キー | チャンネル名 | 説明 |
|------|-------------|------|
| `tzneu` | 쯔느 | Tia解説 |
| `yeongman` | 영만 | ER解説 |
| `magic_daniel` | Magic_Daniel_ER | エマ解説 |
| `nongroot` | 농루트 | ER攻略・解説 |
| `gmiho` | 한미호 | マルティナ・ニア解説 |
| `ldy818` | ldy818 | ER解説 |

---

## 注意事項

### YouTube API

- APIキーは `99_Scripts/youtube/.env` に保存（`YOUTUBE_API_KEY=xxx`）
- 1日10,000単位のクォータ制限あり
- 大量取得は複数日に分けて実行

### Windows環境

スクリプトにUTF-8設定を追加:
```python
if sys.platform == 'win32':
    sys.stdout.reconfigure(encoding='utf-8', errors='replace')
```

### 差分更新

`fetch_channel_comments.py` は差分更新対応。`--full` で全件再取得。

---

## 既知の問題と対策

### APIエラーによるデータ欠損問題（2026-02-05解決済み）

#### 問題の概要

YouTube Data APIのリクエスト中にエラー（レート制限、クォータ超過など）が発生した場合、
コメントが不完全に取得される問題があった。

**例**: 動画に53件のコメントスレッドがあるのに、APIエラーで途中終了し1件しか保存されない。

#### 発生状況

- ldy818チャンネルの`vWuYrLASC38`で発見
- 差分更新では既存動画の不完全データが修正されない
- 初回取得時にエラーが発生すると、以降の差分更新でも補完されない

#### 解決策

`fetch_channel_comments.py` に以下の機能を追加：

1. **リトライ処理（指数バックオフ）**
   - 最大3回リトライ
   - 初回5秒、最大60秒まで待機時間を増加
   - 一時的なエラーからの自動復旧

2. **中断・再開機能**
   - APIクォータ超過時に進捗を保存して中断
   - `{channel_name}_progress.json` に状態を保存
   - 翌日以降、同じコマンドで自動再開

3. **致命的エラーの検出**
   - `quotaExceeded`, `dailyLimitExceeded` などを検出
   - リトライせずに即座に中断・状態保存

#### 使用方法

```bash
# 通常実行（中断があれば自動再開）
python fetch_channel_comments.py --channel ldy818

# 完全に最初からやり直す場合
python fetch_channel_comments.py --channel ldy818 --full
```

#### 不完全データの修復

既に不完全なデータがある場合は `--full` オプションで全件再取得：

```bash
python fetch_channel_comments.py --channel channel_key --full
```
