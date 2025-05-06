---
title: ChatGPTだけでディレクトリやファイル等も含むサイト生成🌾🐖
emoji: 🐖
type: tech
topics:
  - ソロ開発
  - python
  - chatgpt
published: true
---

# ChatGPTを活用したディレクトリやファイル等も含む、Webサイト自動生成の手順

## 🎯 目的

ChatGPTにWebサイトの制作を依頼し、**ローカル環境で自動的にディレクトリやファイルを生成する仕組み**を構築する。

「PythonファイルをChatGPTのアプリで編集できるのであれば、そのファイルの変更を監視し、編集が行われた際に自動でそのファイルを実行して、ディレクトリやファイルを生成するコードを記述すればChatGPTだけで完結するのではないか、という考えです。」

## 🖥️ 環境

- OS: macOS
- 使用アプリケーション:
  - ChatGPT（デスクトップアプリ）
  - PyCharm（Python開発用IDE）

## 🛠️ 必要ファイル

以下の2つのPythonファイルを作成します：

### 1. `watcher.py`
- **概要**: `watchdog` ライブラリを使用し、`exe.py` の変更を監視。
- **役割**: `exe.py` が更新されると、自動的にそれを実行。

### 2. `exe.py`
- **概要**: ChatGPTが生成したWebサイト構築用コードを書き込むファイル。
- **役割**: WebディレクトリやHTMLなどの生成処理を記述。

## 🔁 動作の流れ

1. PyCharmで新しいPythonプロジェクトを作成。
2. `watcher.py` と `exe.py` の2ファイルを用意。
3. ChatGPTにサイト生成コードを依頼し、その内容を `exe.py` にコピー＆保存。
4. `watcher.py` が `exe.py` の変更を検知。
5. 自動的に `exe.py` が実行され、指定されたディレクトリ構成やファイルが生成される。

## 💡 補足

ChatGPTのアプリケーションは、ユーザーのローカルファイルを操作できるため、このような連携が可能です。Web制作の自動化やディレクトリ構成の整備を、対話形式で進められるのが特長です。

## ✏️実際のコード
- wacher.py
```
from watchdog.observers import Observer  
from watchdog.events import FileSystemEventHandler  
import time  
import subprocess  
import os  
  
WATCH_FILE = "exe.py"  # 監視するファイル  
COMMAND = ["python", "exe.py"]  # 実行するPythonスクリプト  
  
class FileChangeHandler(FileSystemEventHandler):  
    def on_modified(self, event):  
        if event.src_path.endswith(WATCH_FILE):  
            print(f"{WATCH_FILE} が更新されました。スクリプトを実行します。")  
            subprocess.run(COMMAND)  
  
if __name__ == "__main__":  
    path = os.path.dirname(os.path.abspath(WATCH_FILE))  
    event_handler = FileChangeHandler()  
    observer = Observer()  
    observer.schedule(event_handler, path=path, recursive=False)  
    observer.start()  
    print(f"{WATCH_FILE} の監視を開始します...")  
  
    try:  
        while True:  
            time.sleep(1)  
    except KeyboardInterrupt:  
        observer.stop()  
    observer.join()

```

- exe.py
  空で良い。

- プロンプト
```
下記のプロンプトを見て順次実行するpythonプログラムをexe.pyに書いてください

以下の要件をすべて満たす Python スクリプト `exe.py` のコードを生成してください。
レイアウトを今風のモダンなbootstrap5を使ったオシャレなサイトデザインで生成してください。

1. **フォルダ作成**  
   ルートに `css/`、`js/`、`img/`、`videos/` を自動で作成する。

2. **ライブラリ自動インストール**  
   - `opencv-python` と `numpy` がインポートに失敗したら自動で `pip install opencv-python numpy` を行い、再度インポートを試みる。失敗時は動画はダミー（空ファイル）を生成する。  
   - ※Excel出力は不要のため `pandas` 等は不要。

3. **CSS生成（`css/style.css`）**  
   - Google Fonts “Inter” を読み込み。  
   - CSS変数によるカラーパレット（primary, accent, bg）設定。  
   - ヘッダー：sticky、左右ナビメニュー、ホワイトバック、ドロップシャドウ。  
   - ヒーロー：動画背景を敷き、グリッド中央配置、高さ80vh。  
   - ヒーロー見出し：タイピング＋点滅アニメーション。  
   - CTAボタン：accent色、ホバーで浮くアニメ。  
   - セクション見出し：左ボーダー、フォントサイズ2rem。  
   - 制作実績カード：グリッド、カードホバー浮上、ポップインアニメ（staggered）。  
   - フェードイン：スクロール検知で要素を下からフェードアップ。  
   - レスポンシブ対応（600px以下でヘッダー縦並び・ヒーロー高さ60vh）。

4. **JavaScript生成（`js/app.js`）**  
   - `.html` へのリンククリックをスムースに遷移。  
   - IntersectionObserver で `.fade-in` 要素に `.visible` を付与。  
   - 制作実績カードに CSS変数 `--i` をセットして staggered pop-in。

5. **ダミー画像生成**  
   - `img/work1.jpg`〜`work3.jpg` を空ファイルで生成。

6. **動画生成**  
   - OpenCV が使える場合、左右グラデーションが5秒間動く `videos/hero.mp4` を生成。  
   - 使えない場合は空の `videos/hero.mp4` を生成。

7. **HTMLファイル自動生成**  
   ルート直下に以下5ファイルを出力：
   - **共通ヘッダー**：  
     ```html
     <header>
       <h1>TestSite<span style="display:block;font-size:0.7em;font-weight:400;">テストサイト</span></h1>
       <nav>
         <a href="index.html">トップ</a>
         <a href="about.html">私たちについて</a>
         <a href="services.html">サービス</a>
         <a href="works.html">制作実績</a>
         <a href="contact.html">お問い合わせ</a>
       </nav>
     </header>
     ```
   - **index.html**：Hero動画背景＋各セクション（`hero`/`about`/`services`/`works`/`contact`、すべて `class="fade-in"`）＋制作実績カードグリッド＋フッター  
   - **about.html**, **services.html**, **works.html**, **contact.html**：本体は `<main><section class="fade-in">`＋見出し＋ダミーテキスト＋フッター。

8. **完了メッセージ**  
   最後に実行した**現在時刻**を  
   ```python
   print(f"生成完了: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")

```

- ChatGPTアプリのpycharm操作設定画面（watcher.pyは先に起動しておく）
![](https://github.com/solodev-jp/zenn-content/blob/main/images/スクリーンショット%202025-05-07%204.21.58.png)

- pycharm画面（必ずexe.pyの編集ページを表示）
![](https://github.com/solodev-jp/zenn-content/blob/main/images/スクリーンショット%202025-05-07%204.23.33.png)

- ChatGPTアプリ実行画面（必ずexe.pyが選択されているのを確認）
![](https://github.com/solodev-jp/zenn-content/blob/main/images/スクリーンショット%202025-05-07%204.24.53.png)

- ChatGPTアプリ自動実行設定画面
![](https://github.com/solodev-jp/zenn-content/blob/main/images/スクリーンショット%202025-05-07%204.25.20.png)

- サイト自動生成後構造画面
![](https://github.com/solodev-jp/zenn-content/blob/main/images/スクリーンショット%202025-05-07%205.14.43.png)

- サイト自動生成後のブラウザ表示画面
![](https://github.com/solodev-jp/zenn-content/blob/main/images/スクリーンショット%202025-05-07%204.49.20.png)


## ⚠️ Pycharm以外での操作
理由は分かりませんが自分の環境では、Pycharm以外（CursorやVsCode）では書き換えは可能でしたが自動保存は出来ず各アプリケーションで保存する一手間が必要でした。
![[]](/images/スクリーンショット 2025-05-07 3.02.25.png)