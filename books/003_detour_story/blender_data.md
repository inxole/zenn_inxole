---
title: "blenderでアニメーションを"
slug: "blender_data"
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

この章は備忘録になります。blenderはバージョン4.3.2を使用しています。マテリアルの説明は不要、Babylon.jsのことだけ知りたいんだ、という方は飛ばしてください。

## blenderで作成したモデルについて

Babylon.jsでインポートするモデルは次の二つです。「球体のモデル」と「キューブのモデル」です。
球体のモデルはただのスフィア(プリミティブ)です。マテリアルでアニメーションを設定しています。キューブのモデルはマテリアルとGeometry nodeを使用しています。

### 球体のモデルについて

|画像3|
|---|
|![][1]|

上中央のウィンドウは3D viewport、左下のウィンドウはshader editor、右下のウィンドウはGraph Editorです。モデルを選択 --> マテリアルのvalueを選択 --> アニメーションカーブが見れます。このカーブで鼓動のようなアニメーションを設定しています。このカーブはvalue nodeで設定しているものです。

shader editor内の【Material Output】のVolumeと【Principled Volume】のVolumeを接続しています。この状態でglbにエクスポートしても【Material Output】のVolumeのデータは作成されません。理由としては、gltfのエクスポートで使用できるvolumeは【volume Absorption】のみ(の可能性が高いから)だからです。下記のリンクを参考にしてください。

[gltf2.0のvolumeの詳細はこちらで確認して下さい。](https://docs.blender.org/manual/ja/4.3/addons/import_export/scene_gltf2.html#volume)
[上記のサイトで紹介されている「KHR_materials_volume」についてはこちらで確認できます。](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_materials_volume)

Blenderの公式によると「For volume to be exported, some transmission must be set on Principled BSDF node.」と記載してあります。Principled BSDFのtransmissionを0以上しないといけないようです。マテリアルで透明度を調整してボリュームで透明な色を内部に付け加えられるというのが【volume Absorption】の特徴です（画像4）。【Principled Volume】はエクスポートできないので、Babylon.jsで鼓動のアニメーションを実装します。

|画像4|
|---|
|![][2]|

### キューブのモデルについて

|画像5|
|---|
|![][3]|

blenderを起動して作成されるデフォルトのメッシュを扱っています。メッシュの大きさはx&y:0.025,z:0.5に、傾きはx軸のみ21°にしてます。
ModifierのArrayを追加してCount:178に、Curveを追加してRelative OffsetのFactor X:1.058に設定してます。使用するサークルは3D viewportの左上メニュー：add --> curve --> circleで追加してCurve Objectで指定してます。これでキューブのリングができます。

|画像6|
|---|
|![][4]|

画像3のモデルの周りを囲むリング状のモデルです。モデルは一つだけでmodifierのArrayとCurveを使用して配置しています(画像5より)。
画像4の右下のウィンドウがGeometry Node Editorに変わっています。マテリアル --> ジオメトリの順に説明します。

#### マテリアル

マテリアルを新規作成すると自動で【Material Output】と【Principled BSDF】が接続された状態になります。二つのノードの間に【Color Ramp】を追加してグラデーションを設定します。blenderの公式によるとColor Rampと白黒画像を使用しないと色が作成されないようなので、別で用意する必要があります。

|映像1|
|---|
|![][I]|

映像1より、白黒画像を用意するために【Gradient Texture】を使用します。【Gradient Texture】だけだと色情報をどこから始めればいいのか不明になるので、【Texture Coordinate】のObjectを使用してます。【Gradient Texture】のドロップダウンメニュー「Radial」でZ軸を基準にしてグラデーションを円周上に配置できます。このマテリアルをGeometry nodesの【set Material】で読み込ませないと描画が変わないです。

【Mapping】はグラデーションを回転させるために用意しました。RotationのZを調整すれば回転できます。動画では使用していません。あとは紫の本体と色がかぶらないようにしています。

#### ジオメトリ

|画像7|
|---|
|![][5]|

最初は【Group input】--【Group Output】の二つのGeometryが接続されている状態でした。その中間に【Scale Elements】をドラッグ＆ドロップして接続します。

【Scale Elements】のScaleで面を変化させます。Uniformで全ての方向に対してスケールが適用されます。Uniformをsingle Axisに変えると単一方向のみのスケールの変化を与えられますが、今回は使いません。ScaleはMathのVectorに接続します。Mathのドロップダウンメニュー:Add --> Sineに変更してsin波で大きさを変化させます。

実はこのSineは必要無いです。【Wave Texture】で既にSineと指定しているからです。なぜ付けているかというと、【Map Range】で決めた「To Min」と「To Max」の値を変えずに大きさを変化させるためです。例えば【Sine】をCosineやTangentに変更すると大きく変化します。声量が大きくなったときはTamgentを使うと楽に演出できるため、付けています。

【Sine】の計算は、sin(30)=-9.880,sin(sin(30))=-0.8349となるので、オブジェクトの大きさは余り変化しないです。メッシュはちょっとだけ小さくなります。そして、【Scale Elements】のCenterに何も指定していないので、オブジェクトのピボットが基準になります。オブジェクトの中央が基準になるので、尚更、変化を感じにくくなっている状態です。

【Wave Texture】は、X --> Zにしておいて他は初期値のまま利用しています。【value】にドライバーを設定して音声の波形データを扱うことができます。ドライバーの設定前に3D viewportのaddでemptyを追加します。emptyのObject --> Custom Properties --> New を押してpropを出現させます。propで任意のkeyを右クリックメニューの「Insert Keyframe」で設定してください。

Animation windowsのDope sheetをGraph Editorに変更します。Graph Editorのメニュー：Channel --> Sound to Samplesで音声ファイルをアタッチすれば、音声データがグラフ化されます。
「Geometry Nodes」に戻ります。

|画像8|画像9|
|---|---|
|![][6]|![][7]|

ドライバーは【value】の入力の右クリックメニューから「Add Driver」を選択します。そして、画像8のようなプロパティが出現します。Type：Averaged Valueに変更し、+Add Input Variableを選択して画像9のように変更します。あとは、Object：先ほどのEmpty、Prop：先ほどのEmpty、Propを変更すると出るPath：["prop"]を設定します。

え？波形が消えた？それならEmptyを選択して確認できます。blenderについては以上です。次は失敗談を紹介します。失敗を知りたいなら買ってください。興味が無ければ次の章に向かってください。
