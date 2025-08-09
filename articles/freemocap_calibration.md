---
title: "FreeMoCapで奥行きを扱えるか試した話 + blenderでデータを扱う方法"
emoji: "🎥"
type: "tech"
topics: ["FreeMoCap", "Blender"]
published: false
---

[壱]:https://zenn.dev/inxole/articles/freemocap_cleanup_guide "url_1"
[弐]:https://www.amazon.co.jp/gp/product/B09241T966 "url_2"
[参]:https://freemocap.github.io/documentation/multi-camera-calibration.html "url_3"

[1]: /images/freemocap_calibration/001.png "image_1"
[2]: /images/freemocap_calibration/002.png "image_2"
[3]: /images/freemocap_calibration/003.png "image_3"
[4]: /images/freemocap_calibration/004.png "image_4"
[5]: /images/freemocap_calibration/005.png "image_5"
[6]: /images/freemocap_calibration/006.png "image_6"
[7]: /images/freemocap_calibration/007.png "image_7"

[I]: /images/freemocap_calibration/threeD_scene_data.gif "movie_1"
[II]: /images/freemocap_calibration/blender_scene_data.gif "movie_2"
[III]: /images/freemocap_calibration/attached_data.gif "movie_3"

## 経緯

[前回の記事][壱]で一台のカメラでは奥行きをうまく扱えないと予測した。なので、二台のカメラで奥行情報を得られるか試してみた。

## FreeMoCapのキャリブレーションを設定する方法

FreeMoCapの二台以上で撮影する手順(URL_1)に従って、キャリブレーションの設定 ⇒ 撮影の順に説明します。ついでにblenderでデータを扱いやすくします。

URL_1:[https://freemocap.github.io/documentation/multi-camera-calibration.html][参]

### その前に、キャリブレーションとは？

カメラの映像に基準がないため正確な位置が計測できない。カメラ毎に向き、レンズも異なるため不正確なデータを取得してしまうらしく、使えないので基準を作っておこうというものらしい。

### 使用したもの

python、poetry、blenderを使用しています。

```
pyenv + python 3.12.0
poetry 1.7.1
blender 4.4.3
```

URL_2のカメラとiphoneを使用してます。Windows PCには「Camo Studio」を、iphoneには「Camo」をインストールしました。
公式のドキュメントより「Charuco Board」が必要なのでURL_3からダウンロード後、プリントアウトしてください。プリントアウトしたものをダンボールに張り付けて使用しました(画像1)。

URL_2:[カメラのURL][弐]
URL_3:[Charuco BoardのURL][参]

|画像1：貼り付け後のCharuco Board|
|---|
|![][1]|

### 試してみる

プリントアウトしたCharuco Boardを使ってキャリブレーション用の撮影データを作成します。画像2より「Record Calibration Videos」にチェックを入れておきます。さらに、ボードのチェックの大きさをはかって「Charuco square size」を 43.0mm にしました。

チェックの大きさは、画像1の一番左上の黒ブロックです。横の大きさを計ってください。

|画像2：FreeMoCapのCameras|
|---|
|![][2]|

次に、Camerasの一番大きいボタン「detect Available Cameras」か、「Control Panel」の「detect Available Cameras」を押します。
「Display Charuco Overlay」にチェックを入れているので、画像3・4のように「id？」が表示されます。このid達が基準になるのでしょうか。

|画像3：webカメラのボード|画像4：iphoneカメラのボード|
|---|---|
|![][3]|![][4]|

ボードの撮影する時、二台カメラの両方に映るようにボードを掲げて、上下に動かしました(数秒でいいようです)。画像3・4のボードはカメラに近づけた時のものです。実際にはもっと離れて撮影すると思います。その時、ボードが粗く描画されます。それでもちゃんと機能しました。

ログに「Anipose camera calibration data saved to calibrations folder, recording folder, and 'Last Successful Calibration' file.」が出れば、キャリブレーションに関わるディレクトリが作成されたことになると思います。

後は、奥行情報を与えた映像でデータがどうなるのか試してみます。結果は映像1・2です。

|映像1：FreeMoCapの位置データ|映像2：blenderのモーションデータ|
|---|---|
|![][I]|![][II]|

映像1はFreeMoCapの「Data Viewer」で確認できます。位置データの顔は動いていないですが、blender上では動いている。補完によるものでしょうか。進みながら羽ばたいているので、奥行情報は手に入れられたようです。カメラの視界の外や遮蔽物に人物が遮られると、データが取れずにカクつくようです。

カメラの位置や設定を変えた場合は、キャリブレーションのデータを作り直す必要があります。

## blenderでデータを扱う

映像1は作成されたデータです。鳥の羽ばたきをイメージしています。扱いたいデータは両腕の「left_wrist」と「right_wrist」です。

データを適用したいモデルは画像5：①のモデルで、カラスをイメージしています。このモデルの両翼にアニメーションを適用します。

|画像5：烏のモデル|
|---|
|![][5]|

### モーションデータの追加方法

blenderのFileメニュー ⇒ Appendで「recording_XX_XX_XX_gmt+9.blend」をダブルクリックします。次に、ディレクトリ名：Object内の「left_wrist」と「right_wrist」の二つを選択してアペンドします。これでシーンにエンプティが作成されます(画像5：②)。

データを適用したいボーンをポーズモードで選択して、プロパティ：Bone Constraints ⇒ Add Bone Constraint ⇒ Copy Transforms を適用して、Copy Transforms の Target に先ほど追加したEmptyを指定しておきます。するとEmptyの位置にコントローラーが移動するはずです。画像5：③に使用するセットキーのみを残してボーンにアニメーションを適用します。

ポーズモードのPoseメニュー ⇒ Animation ⇒ Bake Action... の順に適用すればデータが使えます(画像6)。コンストレイントでコピーしたすべてのモーションが一つのActionsになります。

|画像6：Back Action|
|---|
|![][6]|

読み込ませたモデルは映像3になります。

|映像3：適用後のアニメーション|
|---|
|![][III]|

この後、Copy Transformsのコンストレイントとコピー元を削除してOKです。最後に不要になったキーを消して自分好みにするだけです。

### キーを扱いやすくする

大量のデータを1フレームごとにやってみが、大変だったので次のようにした。
画像7：①のkeyメニュー ⇒ Interpolation Mode ⇒ Bezier で通常のキーフレームと同じにする。次に、keyメニュー ⇒ Density ⇒ Decimate(Ratio) で大量のキーフレームをある程度消すようにした(画像7：②)。左右にスクロールすれば何％消せるかを調整できます。大体80％削減すれば扱いやすくなると思います（画像7：③）。

|画像7：Back Action後の処理|
|---|
|![][7]|

## まとめ

奥行情報は確かに得られるが、撮影時に移動するのはやめた方がいいです。理由は、先ほど記載した「視界の外」や「遮蔽物」によって安定したデータが得られないためです。データによってはビックフットやビックハンドになる。さらに、背骨側に顔が向くこともある。なので、狭い範囲で上半身だけ映し、すべてのモーションを腕だけで表現すればいいと思った。

例えば、リビングで撮影すれば全身が映り、遮蔽物も少なく、動ける範囲も広がる。しかし、撮影を開始できるPCとリビングが離れている場合、撮影スタート⇒リビングに移動⇒モーション終了⇒PCへ移動することになる。手間なので、上半身が映る範囲で腕のモーションを撮った方がいいです。

顔や手の詳細なモーションキャプチャーもしてみたいけど、私は人外が好きなのでいつになるやらです。