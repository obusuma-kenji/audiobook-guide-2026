# 🎙️ 2026年新春お年玉プレゼント

**誰でも作れる！完全自動オーディオブック作成ガイド**

Google Cloud Text-to-Speech APIとPythonを使って、PDFを高品質な音声ファイルに自動変換する完全ガイドです。

---

## 📊 実証済みの成果

このスクリプトは実際のプロジェクトで以下の成果を達成しました：

- ✅ **成功率100%** - 13チャンク全て成功（v5: 16.7% → v6: 100%）
- ⚡ **処理時間8.6秒** - 11,336文字を超高速処理（420倍の時短効果）
- 🎯 **148箇所の専門用語を自動変換** - AI、Notta、Zoom等を正確に発音
- 💰 **月間¥58,650のコスト削減** - 手動録音60分 → AI自動化8.6秒

---

## 🚀 特徴

### v6の主要改善点

1. **URL自動削除機能**
   - 正規表現で28個のURLを自動検出・削除
   - 約13,000文字の削減（24,591 → 11,336文字）
   - バイト数を大幅に圧縮

2. **バイト数制限の最適化**
   - `max_bytes: 4000 → 2500`に変更
   - Google TTS 5000バイト制限に余裕を確保
   - より小さく安全なチャンク分割

3. **専門用語辞書の拡充**
   - 30以上のエントリー追加
   - 長い単語優先のマッチングアルゴリズム
   - カスタム辞書のサポート

### 技術的特徴

- 📄 **PDF自動解析** - PyMuPDFでページ順・レイアウトを保持したテキスト抽出
- ⚡ **並列処理** - ThreadPoolExecutorで10並列の高速音声生成
- 🎯 **バイト数管理** - UTF-8ベースで2500バイト/チャンクに分割
- 🔧 **カスタマイズ可能** - 音声、速度、ピッチを自由に調整
- 📊 **詳細な統計** - 処理速度、成功率、音声サイズを表示

---

## 📋 必要なもの

1. **Python 3.7以上**
2. **Google Cloud アカウント**（無料枠あり）
3. **必要なライブラリ**
   ```bash
   pip install google-cloud-texttospeech pymupdf tqdm
   ```

---

## 🔧 セットアップ手順

### 1. Google Cloud プロジェクト作成

1. [Google Cloud Console](https://console.cloud.google.com/)にアクセス
2. 新しいプロジェクトを作成
3. **Text-to-Speech API**を有効化
4. サービスアカウントを作成してJSONキーをダウンロード

### 2. 認証情報の配置

```bash
# 認証情報を適切な場所に保存
mkdir -p C:\credentials
# ダウンロードしたJSONファイルを移動
```

### 3. スクリプトのダウンロード

下記の`audiobook_creator_v6.py`をコピーして保存してください。

---

## 📝 完全なスクリプトコード

```python
# -*- coding: utf-8 -*-
"""
Audiobook Creator v6 - URL削除・バイト数制限強化版
"""

from google.cloud import texttospeech
import fitz  # PyMuPDF
import os
import re
import json
import time
from concurrent.futures import ThreadPoolExecutor, as_completed
from tqdm import tqdm

class AudiobookCreator:
    def __init__(self, credentials_path, max_workers=10, pronunciation_dict_path=None):
        os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = credentials_path
        self.max_workers = max_workers
        self.pronunciation_dict = self.load_pronunciation_dict(pronunciation_dict_path)
        self.stats = {
            'total_chars': 0,
            'total_chunks': 0,
            'success_count': 0,
            'error_count': 0,
            'replacements': 0,
            'start_time': None,
            'end_time': None
        }
    
    def load_pronunciation_dict(self, dict_path):
        """読み方辞書を読み込み"""
        
        default_dict = {
            # 英語専門用語
            "AI": "エーアイ",
            "ASR": "エーエスアール",
            "LLM": "エルエルエム",
            "SaaS": "サース",
            "Notta": "ノッタ",
            "Zoom": "ズーム",
            "Teams": "チームズ",
            "Google Meet": "グーグルミート",
            "tl;dv": "ティーエルディーブイ",
            "ScribeAssist": "スクライブアシスト",
            "AmiVoice": "アミボイス",
            "YVC-1000": "ワイブイシーセンエイチャク",
            "Yamaha": "ヤマハ",
            "toruno": "トルノ",
            "CLOVA Note": "クローバノート",
            "LINE WORKS": "ラインワークス",
            "PLAUD NOTE": "プラウドノート",
            "Notta Memo": "ノッタメモ",
            "ROI": "アールオーアイ",
            "GDPR": "ジーディーピーアール",
            "Copilot": "コパイロット",
            "Gemini": "ジェミニ",
            "Salesforce": "セールスフォース",
            "HubSpot": "ハブスポット",
            "DX": "ディーエックス",
            "CEO": "シーイーオー",
            "HR": "エイチアール",
            "SNS": "エスエヌエス",
        }
        
        # カスタム辞書を読み込み
        if dict_path and os.path.exists(dict_path):
            try:
                with open(dict_path, 'r', encoding='utf-8') as f:
                    custom_dict = json.load(f)
                    # 空文字キーを除外
                    custom_dict = {k: v for k, v in custom_dict.items() if k and v}
                    default_dict.update(custom_dict)
                    print(f"✅ カスタム辞書読み込み: {dict_path}")
                    print(f"   エントリー数: {len(custom_dict)}")
            except Exception as e:
                print(f"⚠️  辞書読み込みエラー: {e}")
        
        print(f"\n📖 読み方辞書準備完了")
        print(f"   合計エントリー数: {len(default_dict)}")
        
        return default_dict
    
    def apply_pronunciation_dict(self, text):
        """読み方辞書を適用"""
        
        if not self.pronunciation_dict:
            return text
        
        replacements = 0
        sorted_dict = sorted(self.pronunciation_dict.items(), key=lambda x: len(x[0]), reverse=True)
        
        for original, replacement in sorted_dict:
            if original and original in text:
                count = text.count(original)
                text = text.replace(original, replacement)
                replacements += count
        
        self.stats['replacements'] = replacements
        
        if replacements > 0:
            print(f"\n📝 読み方修正適用: {replacements}箇所")
        
        return text
    
    def get_byte_length(self, text):
        """UTF-8バイト数を取得"""
        return len(text.encode('utf-8'))
    
    def extract_text_from_pdf(self, pdf_path):
        """PDFからテキストを抽出"""
        print(f"\n📖 PDF読み込み: {pdf_path}")
        
        try:
            doc = fitz.open(pdf_path)
            total_pages = len(doc)
            print(f"   ページ数: {total_pages}")
            
            full_text = ""
            
            for page_num in tqdm(range(total_pages), desc="テキスト抽出中"):
                page = doc[page_num]
                blocks = page.get_text("blocks")
                blocks.sort(key=lambda b: (b[1], b[0]))
                
                page_text = ""
                for block in blocks:
                    text = block[4]
                    if text.strip():
                        page_text += text.strip() + "\n"
                
                full_text += page_text + "\n"
            
            doc.close()
            
            full_text = self.clean_text(full_text)
            
            print(f"✅ 抽出完了")
            print(f"   文字数: {len(full_text):,}")
            print(f"   バイト数: {self.get_byte_length(full_text):,}")
            
            return full_text
            
        except Exception as e:
            print(f"❌ PDF読み込みエラー: {e}")
            import traceback
            traceback.print_exc()
            return None
    
    def clean_text(self, text):
        """テキストをクリーニング（URL削除を含む）"""
        
        text = text.replace('\r\n', '\n')
        text = text.replace('\r', '\n')
        
        # URLを削除（NEW!）
        print("🔧 URL削除中...")
        url_pattern = r'https?://[^\s]+'
        urls_found = len(re.findall(url_pattern, text))
        text = re.sub(url_pattern, '', text)
        if urls_found > 0:
            print(f"   削除したURL数: {urls_found}")
        
        text = re.sub(r' +', ' ', text)
        text = re.sub(r'\n{3,}', '\n\n', text)
        text = re.sub(r'\n\d+\n', '\n', text)
        text = re.sub(r'^\d+$', '', text, flags=re.MULTILINE)
        
        lines = [line.strip() for line in text.split('\n')]
        text = '\n'.join(lines)
        text = re.sub(r'\n{3,}', '\n\n', text)
        
        return text.strip()
    
    def split_into_chunks(self, text, max_bytes=2500):  # 2500に変更！
        """バイト数でチャンクに分割"""
        print(f"\n✂️  チャンク分割中（最大{max_bytes}バイト）...")
        
        paragraphs = text.split('\n\n')
        chunks = []
        current_chunk = ""
        
        for para in paragraphs:
            para = para.strip()
            if not para:
                continue
            
            para_bytes = self.get_byte_length(para)
            current_bytes = self.get_byte_length(current_chunk)
            
            if para_bytes > max_bytes:
                sentences = re.split(r'([。！？\n])', para)
                for i in range(0, len(sentences)-1, 2):
                    sentence = sentences[i] + (sentences[i+1] if i+1 < len(sentences) else "")
                    sentence_bytes = self.get_byte_length(sentence)
                    
                    if current_bytes + sentence_bytes < max_bytes:
                        current_chunk += sentence
                        current_bytes += sentence_bytes
                    else:
                        if current_chunk:
                            chunks.append(current_chunk.strip())
                        current_chunk = sentence
                        current_bytes = sentence_bytes
            else:
                if current_bytes + para_bytes + 2 < max_bytes:
                    current_chunk += para + "\n"
                    current_bytes = self.get_byte_length(current_chunk)
                else:
                    if current_chunk:
                        chunks.append(current_chunk.strip())
                    current_chunk = para + "\n"
                    current_bytes = para_bytes + 1
        
        if current_chunk:
            chunks.append(current_chunk.strip())
        
        print(f"   チャンク数: {len(chunks)}")
        
        for i, chunk in enumerate(chunks, 1):
            chunk_bytes = self.get_byte_length(chunk)
            if chunk_bytes > max_bytes:
                print(f"   ⚠️  警告: チャンク{i}が{chunk_bytes}バイト（制限超過）")
        
        return chunks
    
    def synthesize_chunk(self, chunk_data):
        """音声を合成"""
        chunk_id, text, output_dir, voice_name = chunk_data
        
        try:
            byte_length = self.get_byte_length(text)
            if byte_length > 5000:
                return (chunk_id, False, 0, f"テキストが長すぎます: {byte_length}バイト")
            
            client = texttospeech.TextToSpeechClient()
            
            synthesis_input = texttospeech.SynthesisInput(text=text)
            voice = texttospeech.VoiceSelectionParams(
                language_code="ja-JP",
                name=voice_name
            )
            audio_config = texttospeech.AudioConfig(
                audio_encoding=texttospeech.AudioEncoding.MP3,
                speaking_rate=0.95,
                pitch=0.0
            )
            
            response = client.synthesize_speech(
                input=synthesis_input,
                voice=voice,
                audio_config=audio_config
            )
            
            output_file = os.path.join(output_dir, f"chapter_{chunk_id:03d}.mp3")
            with open(output_file, 'wb') as out:
                out.write(response.audio_content)
            
            return (chunk_id, True, len(response.audio_content), None)
            
        except Exception as e:
            return (chunk_id, False, 0, str(e))
    
    def create_audiobook(self, pdf_path, output_dir='audiobook_output_v6', voice_name='ja-JP-Neural2-B'):
        """オーディオブック作成メイン処理"""
        
        print("=" * 60)
        print("🎙️  オーディオブック作成ツール v6")
        print("    URL削除・バイト数制限強化版")
        print("=" * 60)
        
        self.stats['start_time'] = time.time()
        
        os.makedirs(output_dir, exist_ok=True)
        print(f"\n📁 出力先: {output_dir}")
        
        text = self.extract_text_from_pdf(pdf_path)
        if not text:
            print("❌ テキスト抽出失敗")
            return
        
        original_file = os.path.join(output_dir, "_01_元のテキスト.txt")
        with open(original_file, 'w', encoding='utf-8') as f:
            f.write(text)
        print(f"\n📝 元のテキスト保存: {original_file}")
        
        text = self.apply_pronunciation_dict(text)
        
        corrected_file = os.path.join(output_dir, "_02_修正後のテキスト.txt")
        with open(corrected_file, 'w', encoding='utf-8') as f:
            f.write(text)
        print(f"📝 修正後のテキスト保存: {corrected_file}")
        
        self.stats['total_chars'] = len(text)
        
        chunks = self.split_into_chunks(text, max_bytes=2500)
        self.stats['total_chunks'] = len(chunks)
        
        chunks_file = os.path.join(output_dir, "_03_チャンク一覧.txt")
        with open(chunks_file, 'w', encoding='utf-8') as f:
            for i, chunk in enumerate(chunks, 1):
                chunk_bytes = self.get_byte_length(chunk)
                f.write(f"=== チャンク {i} （{len(chunk)}文字, {chunk_bytes}バイト） ===\n")
                f.write(chunk)
                f.write("\n\n")
        print(f"📝 チャンク一覧保存: {chunks_file}")
        
        chunk_data_list = [
            (i+1, chunk, output_dir, voice_name)
            for i, chunk in enumerate(chunks)
        ]
        
        print(f"\n🎙️  音声生成中（{self.max_workers}並列処理）...")
        
        results = []
        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            futures = {
                executor.submit(self.synthesize_chunk, data): data[0]
                for data in chunk_data_list
            }
            
            with tqdm(total=len(chunks), desc="進捗") as pbar:
                for future in as_completed(futures):
                    result = future.result()
                    results.append(result)
                    
                    chunk_id, success, size, error = result
                    if success:
                        self.stats['success_count'] += 1
                    else:
                        self.stats['error_count'] += 1
                        print(f"\n❌ チャンク{chunk_id}失敗: {error}")
                    
                    pbar.update(1)
        
        self.stats['end_time'] = time.time()
        self.display_results(output_dir, results)
    
    def display_results(self, output_dir, results):
        """結果を表示"""
        elapsed_time = self.stats['end_time'] - self.stats['start_time']
        total_size = sum(size for _, success, size, _ in results if success)
        
        print("\n" + "=" * 60)
        print("✅ オーディオブック作成完了！")
        print("=" * 60)
        
        print(f"\n📊 統計情報:")
        print(f"   文字数: {self.stats['total_chars']:,}")
        print(f"   読み方修正: {self.stats['replacements']}箇所")
        print(f"   チャンク数: {self.stats['total_chunks']}")
        print(f"   成功: {self.stats['success_count']}")
        print(f"   失敗: {self.stats['error_count']}")
        print(f"   音声サイズ: {total_size / 1024 / 1024:.2f} MB")
        print(f"   処理時間: {elapsed_time:.1f}秒 ({elapsed_time/60:.2f}分)")
        if elapsed_time > 0:
            print(f"   処理速度: {self.stats['total_chars'] / elapsed_time:.0f} 文字/秒")
        
        print(f"\n📁 出力先:")
        print(f"   {os.path.abspath(output_dir)}")
        
        print(f"\n🎉 完成です！")


def main():
    import sys
    
   CREDENTIALS_PATH = r'C:\credentials\your-google-credentials.json'
    VOICE_NAME = 'ja-JP-Neural2-B'
    MAX_WORKERS = 10
    
    if len(sys.argv) < 2:
        print("使い方: python audiobook_creator_v6.py <PDFファイル> [出力先フォルダ]")
        return
    
    pdf_path = sys.argv[1]
    output_dir = sys.argv[2] if len(sys.argv) > 2 else 'audiobook_output_v6'
    dict_path = 'custom_pronunciation.json'
    
    if not os.path.exists(pdf_path):
        print(f"❌ PDFファイルが見つかりません: {pdf_path}")
        return
    
    creator = AudiobookCreator(
        credentials_path=CREDENTIALS_PATH,
        max_workers=MAX_WORKERS,
        pronunciation_dict_path=dict_path
    )
    
    creator.create_audiobook(
        pdf_path=pdf_path,
        output_dir=output_dir,
        voice_name=VOICE_NAME
    )


if __name__ == '__main__':
    main()
```

---

## 💡 使い方

### 基本的な使い方

```bash
# スクリプトの設定を編集（認証情報パスを自分の環境に合わせて変更）
# CREDENTIALS_PATH = r'C:\credentials\your-credentials.json'

# 実行
python audiobook_creator_v6.py "あなたのPDF.pdf"
```

### カスタム出力先を指定

```bash
python audiobook_creator_v6.py "document.pdf" "custom_output_folder"
```

### カスタム読み方辞書の作成

`custom_pronunciation.json`を作成：

```json
{
  "会社名": "かいしゃめい",
  "固有名詞": "こゆうめいし",
  "専門用語": "せんもんようご"
}
```

---

## 📊 出力ファイル

実行すると以下のファイルが生成されます：

```
audiobook_output_v6/
├── chapter_001.mp3          # 音声ファイル（チャンク1）
├── chapter_002.mp3          # 音声ファイル（チャンク2）
├── ...
├── _01_元のテキスト.txt     # PDF抽出テキスト
├── _02_修正後のテキスト.txt # 読み方修正後
└── _03_チャンク一覧.txt     # チャンク分割詳細
```

---

## ⚙️ カスタマイズ

### 音声設定の変更

```python
# 話速を変更（0.25 - 4.0）
speaking_rate=1.0  # デフォルト: 0.95

# ピッチを変更（-20.0 - 20.0）
pitch=2.0  # デフォルト: 0.0

# 音声を変更
voice_name='ja-JP-Neural2-C'  # 女性音声
voice_name='ja-JP-Neural2-D'  # 男性音声
```

### 並列処理数の変更

```python
MAX_WORKERS = 5  # CPUに応じて調整（デフォルト: 10）
```

---

## 💰 料金について

Google Cloud Text-to-Speech APIの料金：

- **無料枠**: 月間100万文字まで無料
- **有料**: 100万文字あたり$16（約2,400円）

**例**: 30ページのPDF（約15,000文字）なら、月間66冊まで無料！

---

## 🔍 トラブルシューティング

### エラー: "認証情報が見つかりません"

```bash
# 環境変数を設定
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/credentials.json"
```

### エラー: "バイト数超過"

- PDFにURLや特殊文字が多い場合、`max_bytes`を2000に下げてみる
- v6では自動的にURLが削除されます

### 音声が途切れる

- `max_bytes`を小さくする（例: 2000）
- PDFのテキスト抽出品質を確認

---

## 🎯 応用例

### ビジネスレポート

```bash
python audiobook_creator_v6.py "quarterly_report.pdf" "reports_audio"
```

### 学術論文

```bash
python audiobook_creator_v6.py "research_paper.pdf" "papers_audio"
```

### 技術ドキュメント

```bash
python audiobook_creator_v6.py "technical_manual.pdf" "manuals_audio"
```

---

## 📚 参考リンク

- [Google Cloud Text-to-Speech ドキュメント](https://cloud.google.com/text-to-speech/docs)
- [PyMuPDF ドキュメント](https://pymupdf.readthedocs.io/)
- [利用可能な音声一覧](https://cloud.google.com/text-to-speech/docs/voices)

---

## 🤝 貢献

バグ報告や機能リクエストは、GitHubのIssuesでお願いします。

---

## 📝 ライセンス

このプロジェクトはMITライセンスの下で公開されています。

---

## 🎉 おわりに

このガイドがあなたのオーディオブック作成の助けになれば幸いです！

質問や感想は、SNSで `#オーディオブック自動化` のハッシュタグをつけてシェアしてください。

**Happy Audiobook Creating! 🎙️✨**

---

**作成者**: おぶすま社会保険労務事務所  
**公開日**: 2026年1月1日  
**バージョン**: 6.0
