---
title: "Tesseract学習環境を構築する"
---
本Chapterでは、さっそく必要なツールやファイルをダウンロードし、セットアップしていく。具体的には、以下の３つを用意する。
- `Tesseract`: OCRの基礎技術の提供
- `jTessBoxEditor`: Fontから学習データの作成と学習をwrapして実施
- ZTMYフォント: オリジナルフォント

# Tesseract🎲
## Tesseractのダウンロード
[Tesseract User Manual](https://github.com/tesseract-ocr/tessdoc)にアクセスすると、[Windows用のinstallerを紹介するWikiページ](https://github.com/UB-Mannheim/tesseract/wiki)へのリンクがあるので、そこから最新版をダウンロードする。  

![](/images/20240907-tetunori-tesseract/02.png)
*Tesseract User Manualの該当部*

![](/images/20240907-tetunori-tesseract/03.png)
*Windows用のinstallerを紹介するWikiページの該当部*

なお、20240908現在`tesseract-ocr-w64-setup-5.4.0.20240606.exe (64 bit)`が最新の様子。

## Tesseractのインストール
ダウンロードしたexeファイルを開くと、インストーラーが起動する。
基本的には`Next`/`I agree`/`OK`/`Install`を押していくだけだが、以下のみ作業が必要なので、注意。
- 最初に聞かれるインストーラーの言語設定は`English`。残念ながら日本語非対応
- `Additional script data (download)` の＋マークをクリックして展開し、下へスクロール、 `Japanese script`および`Japanese vertical script`の両方にチェックをつける
- 同様に`Additional language data (download)` の＋マークをクリックして展開し、下へスクロール、 `Japanese script`および`Japanese vertical script`の両方にチェックをつける
  - **`Javanese`もあるので要注意！**
![](/images/20240907-tetunori-tesseract/04.png)
*Additional language dataの選択*

無事にインストールが完了すると、`C:\Program Files\Tesseract-OCR`にファイルが展開されているはず。

# jTessBoxEditor📦
上記`Tesseract`は基本的にコマンドラインやAPI経由で扱うツールなので、今回は`Tesseract`を使ってGUIでデータを準備したり学習したりできるツール`jTessBoxEditor`を導入する。  
参考までに、`jTessBoxEditor`のオフィシャルページは[こちら](https://vietocr.sourceforge.net/training.html)

## jTessBoxEditorのダウンロード
[オフィシャルページ](https://vietocr.sourceforge.net/training.html)の`Download`リンクをたどり、`SOURCE FORGE`ページから最新の`jTessBoxEditor`をダウンロードする。
なお、2024/09/08時点では`2.6.0`が最新の様子なので、今回は`Released /jTessBoxEditor/jTessBoxEditor-2.6.0.zip`というリンクをクリックしダウンロードした。

## jTessBoxEditorのインストール
特にインストーラーなどは存在しないので、適当な作業フォルダにダウンロードしたzipファイルを展開する。なお、今回は`C:\work\jTessBoxEditor`に展開したとして話を進める。
`jTessBoxEditor`は`Java`上で動作するアプリなので、使用前に`Java`もインストールする必要がある。Windows版の`Java`のインストーラーは[こちら(Java for Windowsのダウンロード)](https://www.java.com/ja/download/ie_manual.jsp)にあるが、`Java`のインストールに関する知識はネット上に沢山あるので、本書では言及しない。インストール完了後、`C:\work\jTessBoxEditor`にあるアプリケーション`jTessBoxEditor.jar`を開いて、以下の画面が表示されたらOK。

![](/images/20240907-tetunori-tesseract/05.png)
*jTessBoxEditorアプリの起動直後画面*

# ZTMYフォント🦔
[ZTMYフォントダウンロードページ](https://zutomayo.net/font/)へアクセスのうえ`Download`ボタンを押下し、`ZTMY_MOJI-R.otf`を得る。以下に、このフォントの特徴を挙げておく。
- 各ひらがな(カタカナも同様)に1:1で特殊な文字がアサインされている
- 濁点、半濁点、捨て仮名は、元の文字を回転して表現する
- 一部の記号(`？`,`！`,`ー`,`、`, `。`)らにも対応したフォントが存在する
- ZTMYファンにはなじみ深い一部の漢字群(`韮`,`強`,`栗`, `剣`, `銃`, `泥`, `土`, `猫`, `飯`, `夜`, `踊`)に対応した隠し絵文字が存在する

さて、このフォントを`jTessBoxEditor`で扱いたいのだが、Windowsのフォントとしてインストールするだけではツールから認識されなかった。`Java`アプリから認識させるために、以下にフォントをコピーしておくこと。
`C:\Program Files\Java\jre.<version>\lib\fonts`
![](/images/20240907-tetunori-tesseract/06.png)
*Javaのfontsフォルダに該当フォントをコピーしておく*

ここまでツールをインストールできたら、OCRの学習環境としては整っているので、次Chapterから早速学習を開始する。

