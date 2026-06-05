# Eternal Return Documentation Project

> プロジェクトの目的・整理方針・ドキュメント構成は [README.md](README.md) を参照。
> このファイルは **収集・翻訳パイプラインの運用手順**（エージェント向け）に特化する。

## プロジェクト要点（最小）

- 目的: **自分自身のEternal Return上達**（マクロより**ミクロ＝戦闘操作**にフォーカス）
- 最終目標: チームファイト理論ガイド `00_Overall/eternal_return_guide_v1_2.md`（**作成中**）の完成
- 韓国上位プレイヤー（主にTiaメイン）の YouTube / Discord から収集・翻訳して活用

## ディレクトリ構造

```
er_doc/
├── README.md                    # プロジェクト概要（人間向け）
├── CLAUDE.md                    # このファイル（パイプライン運用手順）
├── season_timeline.md           # 日付 ⇔ シーズン対応表
├── 00_Overall/                  # キャラ非依存の理論・リファレンス  ← Git管理
├── 01_Character/                # キャラ別ミクロ（Tia/Haze/Hyunwoo/NiaH/Xuelin） ← Git管理
├── 99_Scripts/                  # 収集・翻訳ツール（gitignore）
│   ├── youtube/                 #   YouTube収集ツール
│   ├── discord/                 #   Discordフォーラム収集ツール
│   ├── translation/             #   翻訳モジュール・用語集
│   ├── transcribe_video.py      #   動画文字起こし（Whisper）
│   └── *_srt_*.py               #   字幕クリーニング・マージ
└── 99_Resource/                 # チャンネル別データ（gitignore）
    ├── {ChannelName}/           #   YouTube収集データ
    ├── autoarms/                #   Discordフォーラム（exports / translated）
    └── videos/                  #   ローカル動画＋字幕
```

**注意**: `99_Scripts/` `99_Resource/` `.claude/` は `.gitignore` 済み（APIキー・大容量データを含むため）。
Git管理対象は `00_Overall` / `01_Character` / ルートのドキュメントのみ。

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

## Discord フォーラム収集手順

`99_Scripts/discord/` でDiscordフォーラムのスレッドを収集・翻訳する。
出力は `99_Resource/autoarms/`（`exports/` 生データ → `translated/` 翻訳済み）。

```
1. スレッド取得    →  fetch_forum_threads.py / export_threads.py
2. タイトル抽出    →  extract_thread_titles.py
3. 翻訳用に抽出    →  extract_threads_for_translation.py
4. セクション翻訳  →  translate_sections.py
```

---

## 動画文字起こし・字幕翻訳

ローカル動画やVODから字幕を起こす。スキルとスクリプトの2系統。

- **スキル**（`.claude/skills/`）
  - `soop-download`: SoopLive VODダウンロード
  - `video-to-ja-subtitle`: 動画→日本語字幕（Whisper large-v3 ＋ Claude翻訳）
- **スクリプト**（`99_Scripts/`）
  - `transcribe_video.py`: Whisperで文字起こし
  - `check_srt_hallucinations.py` / `clean_srt_hallucinations.py`: 字幕の幻覚（誤認識）検出・除去
  - `truncate_srt.py` / `merge_translations.py`: 字幕の整形・マージ
  - `youtube/extract_burned_subtitles.py` / `translate_burned_subtitles.py`: 焼き込み字幕の抽出・翻訳

出力例: `99_Resource/videos/`（`*.ko.srt` → `*.ko.cleaned.srt` → `*.ja.srt`）

---

## シーズン情報の補足

翻訳・まとめた情報が **どのシーズンのものか** を添える際は
[`season_timeline.md`](season_timeline.md)（EA〜S11、パッチバージョン対応）を参照。
動画の投稿日からシーズンを逆引きできる。

---

## 翻訳記事・動画のヘッダー（必須）

記事・動画を翻訳したら、ファイル冒頭に **必ず以下のメタ情報ヘッダー** を付けること。

| 項目 | 内容 |
|---|---|
| 原題 | 이터) 현우1위 공략 |
| 著者 | 조용히좀해 (freeingans**) |
| 投稿日 | 2023.08.19 10:27 |
| シーズン | S1 |
| 原文URL | https://gall.dcinside.com/mgallery/board/view/?id=bser&no=4864487 |

**ルール**:
- **原題**: 翻訳せず原文（韓国語）のまま記載
- **著者**: ハンドルネーム（IDがあれば併記）
- **投稿日**: 原文の表記をそのまま（`yyyy.mm.dd hh:mm` 等）
- **シーズン**: 投稿日から [`season_timeline.md`](season_timeline.md) で逆引きして記入（例: `S1`, `S9`）。不明なら `不明`
- **原文URL**: 出典リンク。無い場合は `—`

---

## 翻訳システム（DeepLX + 辞書プレースホルダー）

### 概要

DeepLXプロキシ（無料）+ 辞書プレースホルダー方式で韓日翻訳を行う。
用語集の用語をプレースホルダーに置換→翻訳→復元で用語を制御する。

### ファイル構成

| ファイル | 説明 |
|---------|------|
| `99_Scripts/translation/translator.py` | 翻訳モジュール本体 |
| `99_Scripts/translation/ko_ja_glossary.json` | 用語集ソースデータ（約230エントリ） |
| `99_Scripts/translation/ko_ja_glossary_table.md` | 用語集の対応表（確認用） |
| `99_Scripts/translation/ko_ja_dictionary.json` | 旧辞書（未使用、参考用に残存） |

### 用語集のカテゴリ

- **characters**: キャラクター名（97件）
- **weapons**: 武器タイプ（23件）
- **areas**: マップエリア（21件）
- **game_modes**: ゲームモード（8件）
- **mechanics**: ゲームメカニクス（68件）
- **streaming**: 配信関連（3件）

### 用語集の修正手順

`ko_ja_glossary.json` を編集するだけで即反映（再アップロード不要）。

### 使い方

```python
from translator import KoJaTranslator

translator = KoJaTranslator()
result = translator.translate("韓国語テキスト")
results = translator.translate_batch(["テキスト1", "テキスト2"])
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
| `nebjjang` | 네브짱 | ER配信アーカイブ |
| `s_hyun23` | 성현23 | ER解説 |
| `kiro` | 키로 | ER解説 |
| `jiseuk` | 지슥 | エイデン・ブレア・ヒスイ（ER PARTNERS） |
| `HydeeeeE0304` | 하이도 | ER解説 |
| `OneCircle` | 한동그라미 | ER解説 |

### Discord ソース

| ソース | 説明 |
|--------|------|
| `autoarms` | Discordフォーラム（キャラ質問・立ち回り・チーム質問・設定 等のスレッド）→ `99_Resource/autoarms/` |

---

## 注意事項

### APIキー

`99_Scripts/youtube/.env` に保存:

```
YOUTUBE_API_KEY=xxx              # YouTube Data API
DEEPL_API_KEY=xxx                # DeepL Pro API
DEEPLX_URL=http://localhost:1188/translate  # DeepLX（旧方式、現在未使用）
```

### YouTube API

- 1日10,000単位のクォータ制限あり
- 大量取得は複数日に分けて実行

### 翻訳バックエンド

| 対象 | API | クラス | 理由 |
|------|-----|--------|------|
| コメント | DeepL Pro（有料） | `DeepLTranslator` | 量が少ない・重要情報が多い |
| 動画タイトル | DeepL Pro（有料） | `DeepLTranslator` | 量が少ない |
| 字幕 | Google翻訳（無料） | `KoJaTranslator` | 量が多い |

**重要ルール**: DeepL Pro使用時は必ず**予想文字数をユーザーに提示し、承認を得てから**実行すること。`echo "y"` 等で自動承認してはいけない。

### DeepL Pro

- `DeepLTranslator` クラス（`translator.py`）
- DeepL Glossary機能で `ko_ja_glossary.json` を使用（プレースホルダー方式不要）
- `translate_owner_comments.py` 実行時に文字数見積もり＋確認UIあり

### Google翻訳

- `KoJaTranslator` クラス（`translator.py`）
- 無料、字幕用
- プレースホルダー方式で `ko_ja_glossary.json` を用語制御

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
