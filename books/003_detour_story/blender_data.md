---
title: "第1章：blenderでマテリアルを設定する"
slug: "002_blender_data"
price: 0
---

[1]: /images/002/body_material.png "image_3"
[2]: /images/002/transmission_volume.png "image_4"
[3]: /images/002/modifier_sample.png "image_5"
[4]: /images/002/ring_material.png "image_6"
[5]: /images/002/ring_geometry.png "image_7"
[6]: /images/002/driver01.png "image_8"
[7]: /images/002/driver02.png "image_9"

[I]: /images/002/gradient_material_movies.gif "movie_1"

# blenderで作成したデータを使う

blender：バージョン4.3.2

## blenderで作成したモデルについて

Babylon.jsでインポートするモデルは次の二つです。「球体のモデル」と「リング状キューブのモデル」です。
球体のモデルはスフィア(プリミティブ)です。マテリアルでアニメーションを設定しています。リング状キューブのモデルはマテリアルとGeometry Nodesを使用しています。

### 球体のモデルについて

|画像3：球体のモデルのマテリアルの設定|
|---|
|![][1]|

画像3の上中央のウィンドウは3D viewport、左下のウィンドウはshader editor、右下のウィンドウはGraph Editorです。モデルを選択 ⇒ マテリアルのvalueを選択 ⇒ アニメーションカーブが見れます。このカーブで鼓動のようなアニメーションを設定しています。このカーブはvalue nodeで設定しているものです。

shader editor内の【Material Output】のVolumeと【Principled Volume】のVolumeを接続しています。この状態でglbにエクスポートしても【Material Output】のVolumeのデータは作成されません。理由としては、gltfのエクスポートで使用できるvolumeは【volume Absorption】のみだからです。下記のリンクを参考にしてください。

[gltf2.0のvolumeの詳細はこちらで確認して下さい。](https://docs.blender.org/manual/ja/4.3/addons/import_export/scene_gltf2.html#volume)

[上記のサイトで紹介されている「KHR_materials_volume」についてはこちらで確認できます。](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_materials_volume)

Blenderの公式によると「For volume to be exported, some transmission must be set on Principled BSDF node.」と記載してあります。直訳ではないですが、Principled BSDFのtransmissionを0以上にしないといけないようです。マテリアルで透明度を調整してボリュームで透明な色を内部に付け加えられるというのが【volume Absorption】の特徴です（画像4）。【Principled Volume】はエクスポートできないので、Babylon.jsで鼓動のアニメーションを実装します。

|画像4：volume Absorptionを適用したキューブの例|
|---|
|![][2]|

### キューブのモデルについて

blenderを起動して作成されるデフォルトのモデル(キューブメッシュ)を扱っています。モデルの大きさはX&Y:0.025、Z:0.5に、傾きはX軸のみ21°に設定しています。

|画像5：ArrayとCurveの設定|
|---|
|![][3]|

画像5より、ModifierのArrayを追加してCount : 178に、Curveを追加してRelative OffsetのFactor X : 1.058に設定しています。使用するサークルは3D Viewportの左上メニュー：Add → Curve → Circleで追加してCurve Objectで指定しています。これでリング状キューブができます。

|画像6：z軸周りの色付け|
|---|
|![][4]|

画像6は、球体モデルの周りを囲むリング状キューブに色を付けたものです。モデルは一つだけでmodifierのArrayとCurveを使用して配置しています。右下のウィンドウをGeometry Node Editorに変更してます。マテリアル --> ジオメトリの順に説明します。

#### マテリアル

マテリアルを新規作成すると自動で【Material Output】と【Principled BSDF】が接続された状態になります。二つのノードの間に【Color Ramp】を追加してグラデーションを設定します。blenderの公式によるとColor Rampと白黒画像を使用しないと色が作成されないようなので、別で白黒画像を用意する必要があります。

|映像1：z軸周りの円形グラデーションマテリアルの設定|
|---|
|![][I]|

映像1より、白黒画像を用意するために【Gradient Texture】を使用します。【Gradient Texture】だけだと色情報をどこから始めればいいのか不明になるので、【Texture Coordinate】のObjectを使用してます。【Gradient Texture】のドロップダウンメニュー「Radial」でZ軸を基準にしてグラデーションを円周上に適用できます。このマテリアルをGeometry nodesの【set Material】で読み込ませないと描画が変わらないです。

【Mapping】はグラデーションを回転させるために用意しました。RotationのZを調整すれば回転できます。動画では紫の球体モデルと色が被らないようにしています。

#### ジオメトリ

|画像7：Geometry Nodeの設定|
|---|
|![][5]|

最初は【Group input】--【Group Output】の二つのGeometryが接続されている状態でした。その中間に【Scale Elements】をドラッグ＆ドロップして接続します。

【Scale Elements】のScaleで面を変化させます。Uniformで全ての方向に対してスケールが適用されます。Uniformをsingle Axisに変えると単一方向のみのスケールの変化を与えられますが、今回は使いません。ScaleはMathのVectorに接続します。Mathのドロップダウンメニュー:Add --> Sineに変更することで、Sin波を使ってメッシュの大きさを変化させることができます。

実はこのSineは必要無いです。【Wave Texture】で既にSineと指定しているからです(ドロップダウンメニュー上から三つ目)。なぜ付けているかというと、【Map Range】で決めた「To Min」と「To Max」の値を変えずに大きさを変化させるためです。例えば【Sine】をCosineやTangentに変更すると大きく変化します。声量が大きくなったときはTangentを使うと楽に演出できるため、付けています。

【Sine】を計算してみると、sin(30) = -9.880, sin(sin(30)) = -0.8349となるので、オブジェクトの大きさはほとんど変化しないです。ちょっとだけ小さくなります。そして、【Scale Elements】のCenterに何も指定していないので、オブジェクトのピボットが基準になります。ピボットはオブジェクトの中心になっているので、変化するのは上面と底面だけに見えるかもしれません。

【Wave Texture】のドロップダウンメニュー上から二つ目は、X --> Zにしておいて、他は初期値のまま利用しています。【value】にドライバーを設定して音声の波形データを扱うことができます。ドライバーの設定前に3D viewportのaddでemptyを追加します。emptyのObject --> Custom Properties --> New を押してpropを出現させます。propで任意のkeyを右クリックメニューの「Insert Keyframe」で設定してください。

Animation windowsのDope sheetをGraph Editorに変更します。Graph Editorのメニュー：Channel --> Sound to Samplesで音声ファイルを読み込めば、音声データがグラフ化されます。

最後に「Geometry Nodes」に戻ってドライバーの設定をします。

|画像8：ドライバーのプロパティ|画像9：ドライバーの設定|
|---|---|
|![][6]|![][7]|

ドライバーは【value】の入力の右クリックメニューから「Add Driver」を選択します。そして、画像8のようなプロパティが出現します。Type：Averaged Valueに変更し、+Add Input Variableを選択して画像9のように変更します。あとは、「Object：先ほどのEmpty」と「Prop：先ほどのEmpty」、Propを変更すると出るPath：["prop"]を設定します。

これで音声データを利用してメッシュの変形を行えます。

次は失敗談を紹介します。
