---
title: "はじめに"
---

# サマリ🖊
OSSのOCR(光学文字認識)エンジンとしてよく使われる[Tesseract](https://github.com/tesseract-ocr/tesseract)について、オリジナルフォントの読み取りに良さそう！簡単に使ってみたいな！と思っても大抵Python環境が前提で語られる記事が多いので、プログラミング初学者にはハードルは高めである。そこで、より気軽にOCRツールを作ることができる手段を模索した結果、以下で実現できたので、本書をノウハウとして共有しておく。  
- オリジナルフォントの学習はWindows GUI環境のみで完結
- オリジナルフォントの読み取りは、初学者にも易しくユーザーも多い[p5.js](https://p5js.org/)上のツールとして作成

なお、今回はオリジナルフォントのサンプルとして[ZUTOMAYOフォント](https://zutomayo.net/font/)^[『ずっと真夜中でいいのに。』のオリジナル文字をフォント化したもの](以降ZTMYフォントと呼ぶ)を使用した。🦔
![](/images/20240907-tetunori-tesseract/01.jpg)
*ZUTOMAYO Font from https://zutomayo.net/font/*

# 成果物🎁
https://github.com/tetunori/ztmy-font-decoder
スマホ・PCのカメラから直接、もしくは画像ファイルを読み込んで、ZTMYフォントを認識・デコードできるツール。  
以下、画像ファイル読み込みの使用例を挙げる。
https://x.com/tetunori_lego/status/1830059145417970044

# 開発環境💻
- Windows 11 Home
以上。特に高性能なPCである必要もありません。

