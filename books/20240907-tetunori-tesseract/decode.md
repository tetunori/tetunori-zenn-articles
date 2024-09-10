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
まずは、`p5.js`の`preload`関数内で`Tesseract`のworkerを準備する。
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

preloadが完了した後は、認識したいタイミングで`tessWorker.recognize`を呼び出すだけで、認識結果のテキストを得ることができる。

```JavaScript
  const ret = await tessWorker.recognize(image);
  console.log(ret.data.text);  // 認識結果にアクセスできる
```
[`tessWorker.recognize`](https://github.com/naptha/tesseract.js/blob/master/docs/api.md#worker-recognize)はとても強力な関数で、引数には`bmp`, `jpg`などの画像系はもちろんのこと、`File`や`Blob`、`img`や`canvas`など[幅広いソース](https://github.com/naptha/tesseract.js/blob/master/docs/image-format.md)を渡すことができる。`tesseract.js`でいかに容易に認識が可能か理解できただろうか。

## ファイルのinputとdrop
最初の画像ファイルの登録時、本ツールでは２通りの方法に対応している。１つがファイル選択(input)、もう１つがファイルのdropである。`p5.js`では、それぞれ以下の様に簡単に実装できるようになっている。
```JavaScript
function setup() {
  const cvs = createCanvas(windowWidth, windowHeight);
  cvs.drop(handleFile); // canvasへdrop処理の追加

  // 中略

  input = createFileInput(handleFile);  // File inputを作成
  input.style('display', 'none');
}

const _mouseClicked = () => {
  input.elt.click();  // 画面タップされたら、File選択画面開く
  // 中略
}
```

その後、実際にファイルが選択されると、`handleFile`関数がコールされるので、渡されたfileからデータが画像であることを確認したうえで、上述の認識関数に渡してあげればOK。なお、ここでは`p5.js`の`createImg`関数で`<img>`を作ってからそれを渡している。

```JavaScript
// Handle file input
const handleFile = (file) => {
  if (file.type === 'image') {
    createImg(file.data, '', '', (img) => {
      // Start 1st time decod
      let recogOptions = {};
      const ret = await tessWorker.recognize(img.elt, recogOptions);
      console.log(ret.data.text);
    });
  }
};
```
なお、`tessWorker.recognize`の第2引数にはオプションを渡すことができ、例えば矩形の領域を渡してあげると、画像のうちその部分だけの認識をすることが可能である。

## 画像の白黒化
`p5.js`にはもともと画像をfilterできる機能があるので、それを使って白黒2値化およびしきい値の指定をしている。それ自体は簡単なのだが、`canvas`全体を認識対象にしてしまうと画像以外の領域も認識しようとしてしまうので、ノイズになり認識時間も伸びてしまう。よって、`p5.js`の`CreateGraphics`関数を使って別の`canvas`をつくりそれを認識の対象として指定している。

```JavaScript
const decode = () => {
  (async () => {
    const gpx = createGraphics(gImg.width, gImg.height);  // 別のcanvasを準備
    gpx.image(gImg, 0, 0);  // 用意したcanvasに画像を配置

    if (options.enableFilter) {  // フィルタが有効なら
      gpx.filter(THRESHOLD, options.threshold);  // 白黒2値、しきい値を指定してfilterする
      recogTarget = gpx.elt;  // 認識対象はこの新しいcanvas element 
    }

    let recogOptions = {};
    const ret = await tessWorker.recognize(recogTarget, recogOptions);
    console.log(ret.data.text);
  })();
};
```


## iPhoneのSafariでタップ系の操作が効かない
Chromeでは大丈夫なのだが、Safariでのみこのような問題が起きることがあるが、`p5.js`の`mouseClicked`がそのままでは動作しないことが問題である。これに対するwork-aroundとして`canvas`に対して、`.mouseClicked()`を登録してあげるだけでSafari/Chrome両方で動作するようになる。筆者は元の`mouseClicked`を`_mouseClicked`にリネームして解決した。
```JavaScript
function setup() {
  const cvs = createCanvas(windowWidth, windowHeight);
  cvs.mouseClicked(_mouseClicked); // For Safari.
  // 中略
}
```


