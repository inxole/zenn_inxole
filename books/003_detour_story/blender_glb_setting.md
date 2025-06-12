---
title: "glbの設定を"
slug: "004_blender_glb_setting"
price: 0
---

[1]: /images/004/glb_setting_01.png "image_1"
[2]: /images/004/glb_setting_02.png "image_2"
[3]: /images/004/glb_setting_03.png "image_3"


# シェイプキーに変換してアニメーションを読み込めるようにする方法(失敗談)

## 各設定について

準備ができたのでこれでglbにエクスポートします。メニュー：File --> Export --> gltf2.0を実行します。

|画像13|
|---|
|![][1]|

画像13のIncludeはSelected Objectsだけ使用します。

### Dataの設定

#### Scene Graphについて

|画像14|
|---|
|![][2]|

画像14のDataで「Scene Graph」は利用しません。Scene Graphは雑に説明すると階層構造のこと、各階層のプロパティもその内容に含まれる可能性が高いです。[こちらのサイト](https://dskjal.com/blender/database-and-scene-graph.html)だと分かりやすいかもしれません。

Geometry Nodes Instance(Experimental)は、Geometryの機能で複製したメッシュ（インスタンス）を扱えます。GPU Instanceは、Geometryの機能とは別で複製したインスタンスを扱う可能性が高いです。

Flatten Object Hierarchyは公式によると「Useful in case of non-decomposable TRS matrix.」と出ます。単純にオブジェクトの階層構造を平坦化するためのものです。

---

※おそらくですが、階層構造を作成しているときに、Rotation Mode等を変更することがあります。構造内で片方ではオイラー角を使用して、片方ではQuaternionを使用している場合、複雑になると思います。それに対応するためのものだと思っています。

[TRSとは？](https://kcoley.github.io/glTF/specification/2.0/#transformations)
上記のものでTRSを紹介しています。まあ、単純にT:translate、R:rotation、S:scaleのことですね。

---

Full Collection Hierarchyは、コレクションの階層構造を維持してエクスポートするものです。どうやら、全てのオブジェクトを一つのコレクションにまとめて格納してエクスポートするようです。

利用するのは選択したオブジェクトのみなので(画像13より)、この設定には触れません。

#### Meshについて

Apply Modifiersは使用されているModifierを全てapplyした状態にした後、エクスポートするものです。エクスポート時、Modifierを計算してエクスポートするので、自分でapplyする必要は無いです。ただ、Modifierの一部であるGeometry nodesはそのフレーム時点のメッシュの形を保ったまま適用されてしまうので、アニメーションカーブによる変形は読み込まれません。

UVsはメッシュでUVを使用している場合はチェックをつけてください。今回は不要です。
Normalsは法線のことですね。マテリアルで付けた色が法線に従って表示されるので付けています（念の為）。
Tangentsはノーマルマップを使用するならチェックを付けてください。マテリアルでノーマルマップのテクスチャを読み込んでいる場合は、読み込むデータがあるので不要だと思います。

Attributesは、オブジェクトメッシュのPropertiesウィンドウ内：DataのAttributesのことのようです。使用してないので不要です。

Loose Edgesは、どの面にも属さない辺のことらしいです。面を消したい時に辺を残してしまうことは無いでしょうか？カーブをメッシュに変換して、それをエクスポートするときに必要かもしれません。

Loose Edgesは、どの面、辺にも属さない点のことらしいです。点群等のエクスポートに使用できるかもしれません。

Shared Accessorは、「glTFファイル内で インデックスバッファ（頂点インデックス）を再利用する機能」らしいです。三角形だけのメッシュを考えてみると、３点の情報（色・法線・UVなど）が一致していれば機能する。ただ、UVでテクスチャを使用している場合は機能しないようです。今回は不要です。

---

Vertex Colorはデフォルトで使用しています。

Use Vertex Colorは基本「Material」でいいです。設定してあるマテリアルを読み込ませることができます。他にActiveかNoneがあります。Activeはマテリアルを使用しないで色を付けた時、その色を使用するために必要らしいです。Noneは使用しないだけです。

Export all vertex colorsは一つのメッシュにマテリアルを複数設定している場合に使えます。
Export active vertex color when no materialはマテリアルの基本であるBSDFに頂点カラーを接続していなくても利用できる機能です。マテリアルを使用しないで色を付けたものを利用できるということです。

---

#### Materialについて

MaterialsをExportにすると、使用しているマテリアルをそのまま使用します。placeholdersにすると最低限のプリミティブ（ダミー）を作成するそうです。glb以外で色付けなどが行えるなら使用してください。Noneはマテリアルを使用しないだけです。

Imagesは使用している画像をglbでどう扱うかということです。フォーマットはAutoMaticでいいと思います。他はjpeg、webp、noneが選べます。Image qualityは任意で調整してください。Create WebPは使用している画像+全ての画像をWebPに変換したものを使用するそうです。WebP fallbackはWebPの作成もするしpngの作成もするように指定するものです。

Unused Textures & Imagesは空のイメージを使用するか、空のテクスチャを使用するか指定するそうです。

#### Shape Keysについて

Shape Key Normalsは法線を使用することで変形したときの見た目が保たれるようにするそうです。
Shape key Tangentsはノーマルマップを使用しているときに必要です。

Optimize Shape Keysについて

Use Sparse Accessor if betterは、一部の頂点だけ変化がある場合に、変化してる部分だけ書き出すそうです。

Sparse Accessorについては[こちら](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#sparse-accessors)を見てください。

Omitting Sparse Accessor if data is emptyは、空のシェイプキーを除いてエクスポートするか選べます。今回は不要なシェイプキーは消しているのでチェックは付けません。

#### ほかの設定について

Armatureは今回使用していないのでデフォルトのままでいいです。
SkinningはArmatureでスキンメッシュを使用しているなら必要です。
Lightingは使用しないです。セレクトしているオブジェクトだけ使用しているためです。
Compressionは公式より「Google Dracoを使ってメッシュを圧縮します。」とあります。わたしはFireFox派なので使用しません。

### Animationの設定

|画像15|
|---|
|![][3]|

Animation modeはどのアニメーションを利用するのか指定できます。Actionsに登録してあるので画像の通りでいいです。NLA Tracksでもいいです。「Bake All Objects Animations」でアニメーションがうまくいかないときは使用するものらしいです。

#### Rest & Rangesについて

基本的には再生範囲のことを言っています。しっかり試してはいないですが、Frame Rangeに合わせて自動的に保存されるはずです。

#### Armatureについて

チェックはつけていますが、Armatureを使用してないので無視されます。

#### Shape Keys Animation

Reset shape keys between actionsはアクション間のシェイプキーを削除するそうです。余分なシェイプキーを削除するということかもしれません。
※NLA Actionsを設定したときにframe 1までセットキーを移動していました。もし、していない場合は余分なフレームが適用されてしまいます。この余分なフレームが削除されるということではないです。

#### Sampling Animations

セットキーを1フレームごとに設定しているなら1でいいです。

#### Animation Pointer(Experimental)

内部の特定のプロパティをアニメーションさせることができるようです。

#### Optimize Animation

Optimize Animation Sizeはアニメーションのキーフレームを最適化するようです。重複したキーは削除して最適化するということです。

Force keeping channels for bonesは、ボーンでアニメーションを設定していない一部のものでも強制的に保存するそうです。チェックを付けていますが、Armatureがないので影響はないです。

Force keeping channel for objectsは、オブジェクトに対してアニメーションが設定してなくても保存するそうです。

Disable viewport for other objectsは、対称オブジェクト以外のものがview portにある場合、処理が重くなるのを防ぐために使用されます。非表示にするだけのようです。

#### Extra Animation 

Prepare extra animationsは「標準のアクション以外の方法で割り当てられたアニメーションデータも出力したいときに使う」そうです。公式は明示していないので詳細は不明です。

#### Action Filter

指定したフィルターに一致するもののみ使用するそうです。チェックをつけると登録した、もしくは、作成したActionsが表示されます。指定されたActionsのみエクスポートできます。