---
title: "p5.jsでオリジナルフォントを文字認識＆デコードする"
---

このChapterでは、得られた学習済データを使って文字認識ができるツールを作成したい。ツールはどんな環境でもよいのだが、初学者にも使いやすい`p5.js`を使ってツールを作っていくことにする。

最終的な成果物としては、以下にあるような`index.html`, `mySketch.js`ができあがるイメージ。
https://github.com/tetunori/ztmy-font-decoder/tree/main
なお、[GitHub Pagesにお試しできる環境](tetunori.github.io/ztmy-font-decoder/)を用意している。

# ツールの機能概要(使い方)
## カメラを使って解析する場合
1. [ztmy-font-decoder](https://tetunori.github.io/ztmy-font-decoder/)にアクセスする。
2. 画面上半分の📷アイコンをタップする。
3. 解析したい部分をタッチで選択すると、画面下半分に解析結果が表示される。

## 画像から解析する場合
1. [ztmy-font-decoder](https://tetunori.github.io/ztmy-font-decoder/)にアクセスする。
2. 画面下半分の📁アイコンをタップして、解析する画像を選択する。
3. 解析したい部分をタッチで選択すると、画面下半分に解析結果が表示される。

なお、デフォルトのままで解析できるケースはほとんどないので、右上のコントローラーから調整できるようにしている。
- 白黒画像化としきい値設定
- (上記の)設定のリセット
- [ztmy-font-tester](https://github.com/tetunori/ztmy-font-tester/)への移動

# 使用する外部ライブラリ
上記機能を実現するにあたり、下記のライブラリを使用した。
- [p5.js](https://github.com/processing/p5.js): `Processing`のJavaScript版。本書では説明は割愛するので、必要な場合は[本家のTutorial](https://p5js.org/tutorials/)を見ること
- [tesseract.js](https://github.com/naptha/tesseract.js): `Tesseract`をJavaScriptから使用できるようにしてくれる神ライブラリ。必要に応じて[API仕様のページ](https://github.com/naptha/tesseract.js/blob/master/docs/api.md)を見る。
- [dat.gui](https://github.com/dataarts/dat.gui): GUIで変数の調整等を行うためのライブラリ

# ソースコードのポイント

## `Tesseract`の使い方
前のChapterで得られた学習済データ`ztmy.traineddata`は`tesseract`フォルダを作り、その中へコピーしておく。
まずは、`preload`関数内で`Tesseract`のworkerを準備する。
```JavaScript
let tessWorker = undefined;

function preload() {
  // Prepare Tesseract for ZTMY font
  (async () => {
    tessWorker = await Tesseract.createWorker('ztmy', 0, {
      langPath: './tesseract/',
      gzip: false,
    });
  })();
}
```
loadが完了した後は、認識したいタイミングで以下を呼び出すだけで、認識結果のテキストを得ることができる。
```JavaScript
  const ret = await tessWorker.recognize(image);
  console.log(ret.data.text);  // 認識結果にアクセスできる
```
[`tessWorker.recognize`](https://github.com/naptha/tesseract.js/blob/master/docs/api.md#worker-recognize)はとても強力な関数で、imageの部分はbmp, jpgなどの画像はもちろんのこと、`File`や`Blob`、`img`や`canvas`など[幅広いソース](https://github.com/naptha/tesseract.js/blob/master/docs/image-format.md)を選択できる。

## 

