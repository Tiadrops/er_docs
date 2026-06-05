# Eternal Return Documentation

Eternal Return（エターナルリターン）の **ミクロ（戦闘操作）上達** を目的とした、ガイド・攻略情報のまとめリポジトリ。

韓国の上位プレイヤー（主にTiaメイン）のYouTube / Discord から情報を収集・翻訳し、
チームファイト理論やロール別の戦闘基礎、キャラ別のコンボ・スキル回しを整理している。

> 最終目標は **チームファイト理論ガイド（キャラ非依存）の完成**。
> → [`00_Overall/eternal_return_guide_v1_2.md`](00_Overall/eternal_return_guide_v1_2.md)（作成中）

## このリポジトリに入っているもの

ドキュメント（`00_Overall` / `01_Character`）のみを Git 管理している。
収集スクリプト（`99_Scripts/`）と生データ・翻訳済みデータ（`99_Resource/`）は
**APIキーや大容量データを含むためローカル専用**（`.gitignore` で除外）。

```
er_doc/
├── README.md                  # このファイル
├── CLAUDE.md                  # 収集・翻訳パイプラインの運用手順（エージェント向け）
├── season_timeline.md         # 日付 ⇔ シーズン対応表（翻訳時のシーズン補足用）
│
├── 00_Overall/                # キャラ非依存の理論・リファレンス
│   ├── eternal_return_guide_v1_2.md   # チームファイト理論ガイド（★作成中・最終目標）
│   ├── teamfight_guide.md             # チームファイトガイド（ja / _ko）
│   └── Hard_CC.md / Hard_CC_by_character.md  # ハードCCリファレンス
│
├── 01_Character/              # キャラ別のミクロ（コンボ・スキル回し・先行入力）
│   ├── Tia/                   # メインキャラ
│   ├── Haze/  Hyunwoo/  NiaH/  Xuelin/
│
├── 99_Scripts/   （gitignore）# 収集・翻訳ツール群
└── 99_Resource/  （gitignore）# チャンネル別の収集データ
```

## ガイドの整理方針

1. **チームファイト理論（キャラ非依存）** — 3線ポジショニング / アグロピンポン / フォーカス原則 / 構成パターン / 構図判断
2. **ロール別の戦闘基礎** — T / TB / B / A / SD / AD / S にカテゴライズ
3. **キャラ別のミクロ** — 先行入力・スキル回し・コンボ（参考情報として収集）

## 収集・翻訳パイプライン

YouTube / Discord からの収集と韓日翻訳の手順は [CLAUDE.md](CLAUDE.md) に集約。

- **YouTube**: コメント・字幕の取得 → DeepL/Google翻訳 → 投稿日ベースで整理
- **Discord**: フォーラムスレッドのエクスポート → 翻訳
- **動画**: Whisper 文字起こし → 字幕翻訳（`.claude/skills/` のスキル）
- **翻訳**: DeepLX / DeepL Pro / Google翻訳 ＋ 用語集（`ko_ja_glossary.json`）で用語制御

## シーズン情報

翻訳・まとめた情報が **どのシーズンのものか** を補足するための対応表 →
[season_timeline.md](season_timeline.md)（EA〜S11、パッチバージョン対応付き）
