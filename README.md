# 🎁 2026年新春お年玉プレゼント

# 誰でも作れる！オーディオブック完全作成ガイド

Google Cloud Text-to-Speechを使った、PDFからオーディオブックを作成する完全ガイドです。

## 📖 ガイドを読む

👉 **[オーディオブック作成完全ガイド](https://obusuma-kenji.github.io/audiobook-guide-2026/)**

## 🎯 このガイドでできること

- ✅ PDFを高品質な日本語音声に変換
- ✅ 専門用語の読み方をカスタマイズ
- ✅ 縦書き・横書きPDF両方に対応
- ✅ 並列処理で高速変換（200ページ約1-2分）

## 💰 コスト

- Google Cloud TTS: 200ページで約500円
- 初回無料枠: 毎月100万文字（Neural2音声）

## ⏱️ 所要時間

- 初回セットアップ: 30-60分
- 2回目以降: 数分（ファイルをドラッグ＆ドロップするだけ）

## 📂 ファイル構成

```
audiobook-guide-2026/
├── index.html                    # ガイドHTML版（ブラウザで閲覧）
├── audiobook_creator.py          # Pythonスクリプト本体
├── pronunciation_dict.json       # 読み方辞書サンプル
└── README.md                     # このファイル
```

## 🚀 クイックスタート

### 1. スクリプトをダウンロード

```bash
# このリポジトリをクローン
git clone https://github.com/obusuma-kenji/audiobook-guide-2026.git
cd audiobook-guide-2026
```

### 2. 必要なライブラリをインストール

```bash
pip install google-cloud-texttospeech pymupdf tqdm
```

### 3. Google Cloud設定

[完全ガイド](https://obusuma-kenji.github.io/audiobook-guide-2026/)の手順に従ってGoogle Cloud Text-to-Speech APIを設定してください。

### 4. 実行

```bash
python audiobook_creator.py your_file.pdf
```

## 📝 読み方辞書のカスタマイズ

`pronunciation_dict.json` を編集して、専門用語の読み方を追加できます：

```json
{
  "AI": "エーアイ",
  "専門用語": "せんもんようご",
  "人名・太郎": "じんめい たろう"
}
```

## 💡 使い方の詳細

完全な手順、トラブルシューティング、よくある質問は、以下のガイドをご覧ください：

👉 **[完全ガイドを読む](https://obusuma-kenji.github.io/audiobook-guide-2026/)**

## 🎤 利用可能な音声

| 音声名 | 性別 | 特徴 |
|--------|------|------|
| ja-JP-Neural2-B | 男性 | 落ち着いた声 |
| ja-JP-Neural2-C | 女性 | 明瞭な声 |
| ja-JP-Neural2-D | 男性 | フォーマル |

## 📊 処理速度の例

| ページ数 | 文字数 | 処理時間 | コスト |
|---------|--------|---------|--------|
| 50 | 25,000 | 約30秒 | 約250円 |
| 100 | 50,000 | 約1分 | 約500円 |
| 200 | 100,000 | 約2分 | 約1,000円 |

## 🔧 トラブルシューティング

問題が発生した場合は、[完全ガイドのトラブルシューティングセクション](https://obusuma-kenji.github.io/audiobook-guide-2026/#troubleshooting)をご覧ください。

## 📄 ライセンス

このガイドは教育目的で作成されています。実際の使用は自己責任でお願いします。

## 🙏 謝辞

このガイドを読んでくださった皆様、ありがとうございます。

**新年おめでとうございます！2026年が素晴らしい一年になりますように！**

---

**作成者**: おぶすま社会保険労務事務所  
**バージョン**: 1.0.0  
**公開日**: 2026年1月1日

#オーディオブック #GoogleCloudTTS #2026年お年玉 #新春プレゼント
