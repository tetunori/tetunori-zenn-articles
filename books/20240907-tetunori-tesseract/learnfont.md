---
title: "オリジナルフォントを学習する"
---

このChapterでは、オリジナルフォントから文字認識時に使用する学習データを作成していく。基本的な流れとしては、以下の通り。
1. 学習する対象の文字もしくは文字列をテキストで準備する
1. `jTessBoxEditor`上にて該当フォントで`Tesseract`の学習で使用する事前(box)データを作成する。
1. `jTessBoxEditor`上にてboxデータを編集する
1. boxデータを使って`jTessBoxEditor`経由で`Tesseract`を使って学習する

# 学習する文字・文字列の準備🖊
文字認識をする際には、文字単体の認識と同様に文字の組み合わせ等も学習しておくと、認識時の精度が上がると思われる。よって、どのような文字や文字列、文章を学習させるかによって、最終的な精度の違いが生まれることになる。`jTessBoxEditor`では、何を学習させるかをテキストで指定することになっているので、そのファイルを準備したい。

今回のZTMYフォントはその性質上、組み合わせはほとんど不要なので、以下のように、対応している文字をすべて列挙する形をとった。今回は`ztmy-chars.txt`という名前でどこかに保存しておいたとする。
https://github.com/tetunori/ztmy-font-decoder/blob/main/experiment/20240831/ztmy-chars.txt

上記をみてわかるように、ただ列挙しているだけでなく、いくつか工夫した点がある。一見簡単にならべただけの様にみえるが、`Tesseract`の学習がブラックボックス化されている(AI共通の課題)ので、学習結果や精度をみてからまたここに戻ってきて、テキストを変更して、、、の繰り返しが必要で、ある程度の時間を要した。

## 学習対象テキストの工夫点💡
- 文字同士は１文字毎の切れ目がわかりやすいようにスペースで分割
- 文字の配置によって、`Tesseract`から学習対象と認識されない場合がある。この際は、文字を別の場所に移動してみると解決することがあった。`、`, `。`, `韮`, `強`などがそれにあたる。
- ZTMYフォントのオフィシャルな使われ方を見ていると、`イイ`がリガチャと思えるほど近接して配置されることがあったので、ここのみ文字列として学習できるようにしている。
- ZTMYフォントにおいて`ォ`と`オ`が同一形状になってしまう(仕様バグ)ので、`ォ`を学習対象から外した。

# 事前(box)データを作成する📦
`Tesseract`にデータを学習させるにあたっては画像と「その画像のどの部分にどの文字が表記されているか」のデータを渡してあげるのだが、boxデータと呼ばれている。`jTessBoxEditor`では、先ほどのテキストを指定すると、とても簡単にboxデータを作成して編集することができる。

まずは`jTessBoxEditor`を立ち上げ`TIFF/Box Generator`というタブを選択し、`Input`ボタンから先ほどのテキストファイルを選択する。続けて、以下の設定を行う。
- Output: `...`ボタンを押下してboxデータの生成先を指定
- Prefix(Language Code): `ztmy`と指定。`eng`や`jp`の様に言語コードして後に扱う
- Filename: 自動で入るのでいじらなくてよし
- Font: `ZTMY_MOJI R`を選択する。Sizeは`14`位にしておく
- Tracking: `-0.04`とした。なぜかはわからないがこの設定が最も高精度だった
これらが全て完了すると、以下のようにZTMYフォントで学習対象テキストが表示される
![](/images/20240907-tetunori-tesseract/08.png)
*Box Generatorへの設定画面*

この状態で`Generate`ボタンを押すと、`Output`フォルダにboxデータ`ztmy.ztmy_mojir.exp0.box`およびTIFF画像`ztmy.ztmy_mojir.exp0.tif`が生成される。

# boxデータを編集する📝
さて、上記得られた画像とboxデータのを可視化してみよう。  
`jTessBoxEditor`の`Box Editor`のタブを選択し、`Open`ボタンで先ほどのboxデータを開くと、先ほどの画像の上に各フォントを囲う青い枠(=box)が描画される。各boxは、X座標、Y座標、幅、高さ、何の文字を示すのか？のデータで構成されている。  
![](/images/20240907-tetunori-tesseract/09.png)
*Boxデータの可視化*

基本的には、自動でboxがアサインされているが、ところどころフォントの認識にミスっている箇所があるので、手動でデータを書き換える必要がある。また、認識の間違いではないが`、`や`。`は余白も含めてフォントっぽくなるように枠を設定すること。そして不具合だと思うが、`ヒ`と`ビ`の文字が逆転していることもあったので、最後に認識するときにおかしいなとおもったら、いつでもここに戻れるようにしておこう。
![](/images/20240907-tetunori-tesseract/10.png)
*修正が必要なデータの例。この場合はwidthを大きくする。*

連続する文字列のマージは少し難易度が高い。今回は`イイ`というデータを持ってきたが、boxの認識は連続した１つの文字列とは思ってくれていない。そのような場合はこの２つのboxを選択した状態（`Ctrl`キーを押しながら選択するとできる）で`Merge`ボタンを押すと、１つのboxにマージし、文字列もちゃんと`イイ`の２文字で解釈してくれる。ただし、自分が欲しいのは文字間が詰まったなので、一度tif画像をペイントアプリなどで手動で修正して、再度boxデータを調整するなどしている。
![](/images/20240907-tetunori-tesseract/11.png)
*もろもろ調整した後の`イイ`画像と該当box*

さて、一通りboxデータに問題がなさそうであれば、`Save`ボタンを押してboxデータの編集を完了する。当然、変更内容はファイル`ztmy.ztmy_mojir.exp0.box`に保存される。

# 学習する🧑‍🏫
では、上記のデータを使って学習を行ってみよう。  
`jTessBoxEditor`の`Trainer`のタブを選択し、以下の設定を行う。
- `Tesseract Executables`は前Chapterでインストールしたパスのexeを指定。今回は`C:\Program Files\Tesseract-OCR\tesseract.exe`。
- `Training Data`は先ほどのboxデータを指定する。今回は`ztmy.ztmy_mojir.exp0.box`だ。
- `Language`は`ztmy`。
- `Bootstrap Language`は空欄でOK。
- `Training Mode`は`Train with Existing Box`を選択。

ここまで設定できたら、`Run`ボタンを押すだけで学習を始めてくれる。今回のデータくらいだと、2, 3秒程度で学習が終わり、うまく完了すると、boxデータと同じ場所へ学習済データ`tessdata/ztmy.traineddata`が保存される。このデータがあれば文字認識を実現できるようになる。  

## 学習がうまくいかないとき🤔
学習時のログを見ていると、特定の文字の学習に失敗していることがある。学習のエンジンをブラックボックスとして使っているため、一概には言えないが自分が改善した時の対応を以下に記載する。
- テキスト時点で文字の並びを変更する。
- boxデータ作成時のパラメータを調整する。文字間サイズやフォントサイズを変えるだけで学習が成功したりする。
- Latin/Katakanaなどのunicharsetを用意すると成功することがある。自分は考えるのも面倒なので[tesseract-ocr/langdata](https://github.com/tesseract-ocr/langdata/tree/main)から、`unicharset`, `Hiragana.unicharset`,`Katakana.unicharset`,`Latin.unicharset`らをすべてboxデータと同じフォルダに入れている。
- とにかく答えは学習時のエラーログに書いてあるので、それを解消するようにデータを調整する。

よくある`Tesseract`の失敗例としては、最初に画像から文字を認識するのだが、精度が甘くこちらが指定したboxのところに文字などないとエラーを出してくることがある。そんな時は`Validate`ボタンをおしてtiff画像を選択すると、いくつ・どこで文字が認識できているかを可視化してくれているので、これが意図通りの配置がを確認するもの1手である。

いずれにせよ根気強くエラーを取り除いて、すべての文字が学習されることをログから確認しよう。

```bash
** Run Tesseract for Training **
[C:\work\jTessBoxEditor\tesseract-ocr/tesseract, ztmy.ztmy_mojir.exp0.tif, ztmy.ztmy_mojir.exp0, box.train]
row xheight=15, but median xheight = 48.5
APPLY_BOXES:
   Boxes read from boxfile:      97
   Found 97 good blobs. # blobの数が自分が登録したboxの数とあっていればOK。
   Leaving 1 unlabelled blobs in 0 words.
Generated training data for 11 words
...
...
** Moving generated traineddata file to tessdata folder **
** Training Completed ** # ここまでくれば学習は完了している。
```

自分が学習したときの環境一式は以下においてあるので、必要に応じて参照すること。
https://github.com/tetunori/ztmy-font-decoder/tree/main/experiment/20240831

それでは、次Chapterでは学習済のデータを使って文字認識を実装していく。
