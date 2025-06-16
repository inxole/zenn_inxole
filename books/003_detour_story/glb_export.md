---
title: "ファイル容量が大きすぎた"
slug: "006_glb_export"
price: 500
---

[1]: /images/006/glb_import_01.png "image_1"
[2]: /images/006/re_import.png "image_2"
[3]: /images/006/re_import_in_web.png "image_3"
[4]: /images/006/NLA_voice4.png "image_4"
[5]: /images/006/NLA_voice5.png "image_5"
[6]: /images/006/voice2_delete_shape.png "image_6"

# glbでエクスポートするときに問題が発生した話し

## glbをBabylon.jsで読み込む

### glbファイルのインポートの仕方

画像3、画像6のモデルをエクスポートしま
画像4は「glbの設定を」の章で紹介した設定でいいですが、画像3は次の設定をしてください。

最上位メニュー：File --> export --> gltf2.0 実行
gltf2.0の設定：Include --> Limit to:Selected Objectsにチェック
gltf2.0の設定：Transform --> +Y Up にチェック
gltf2.0の設定：Data --> Material：デフォルトのまま
他は使用しないのでチェックを全外ししています。

作成されたglbファイルをpnpmで作成されたpublicに保存します。
次のコード4でglbを読み込ませることができます。

```ts
コード4
 1 | async function load_Assistant(
 2 |     canvas: HTMLCanvasElement,
 3 |     sceneRef: React.RefObject<Scene | null>
 4 | ) {
 5 |     const engine = new Engine(canvas, true)
 6 |     const scene = new Scene(engine)
 7 |     scene.clearColor = new Color4(0, 0, 0, 0)
 8 | 
 9 |     Load_camera(scene, canvas)
10 |     Load_Light(scene)
11 | 
12 |     sceneRef.current = scene
13 | 
14 |     await load_body(scene)
15 |     await load_ring(scene)
16 | 
17 |     engine.runRenderLoop(() => scene.render())
18 |     const resize = () => engine.resize()
19 |     window.addEventListener('resize', resize)
20 |     return () => {
21 |         engine.dispose()
22 |         window.removeEventListener('resize', resize)
23 |     }
24 | }
```

先に挙げたコード3に14、15行目を追加しました。load_bodyは画像3のモデルを、load_ringは画像5のモデルを読み込むものです。二つの関数はコード5で詳細を確認して下さい。

```ts
コード5
 1 | export async function load_body(scene: Scene): Promise<Mesh> {
 2 |     const url = "/AI_assistant_body.glb"
 3 |     const container = await LoadAssetContainerAsync(url, scene)
 4 |     container.addAllToScene()
 5 |     const parentMesh = container.createRootMesh()
 7 |     return parentMesh
 8 | }
 9 | 
10 | export async function load_ring(scene: Scene): Promise<Mesh> {
11 |     const url = "/AI_assistant_ring.glb"
12 |     const container = await LoadAssetContainerAsync(url, scene)
13 |     container.addAllToScene()
14 |     const parentMesh = container.createRootMesh()
15 |     return parentMesh
16 | }
```

変数：urlの内容以外は一緒なので、片方の関数だけ説明します。2行目でファイルの場所を指定しています。3行目でLoadAssetContainerAsyncを使用してファイルを読み込ませています。非同期処理で読み込みが完了するまで待ちます。ただ読み込んでいるだけで何もしませんからね。なので4行目を追加しています。4行目がないとシーンに作成されません。

今まではSceneLoaderでインポートできてました。しかし、コード4,5を作成した時には非推奨で、代わりに「ImportMeshAsyncを使ってね」とありました。

推奨されたものを使ってみましたが、何故かファイルではなくURLを入力するようになっていたので、fetchやらCORSに対応しないといけなく、とても手間な仕様変更になっていました。手間すぎたので「LoadAssetContainerAsync」で代替しています。

現在(2025/06/13)は「ImportMeshAsync」で簡単にインポートできます。使うならImportMeshAsyncでいいと思います。

|画像16|
|---|
|![][1]|

画像16より、薄いピンク色の球体と、色の無いリング状のキューブが作成されました。意図しない結果になりました。

## 問題があった

インポートしたけど問題が起きました。

1. 球体のモデルの透過が反映されていない。色も違う。
3. キューブのモデルでマテリアルが読み込まれていない。
4. voice4,5のアニメーションが途中で停止する。

### 一つ目の問題に対して

|画像17|画像18|
|---|---|
|![][2]|![][3]|

どうやらTransmissionが原因のようでした(画像17)。画像18はブラウザ上で確認したものになります。変更した箇所はRoughness：1.0、Transmission：0.0です。色をもっと濃くしたいのと、球体ならBabylon.jsの機能で作成した方がいいと思ったので変更します。

```ts
コード6
 1 | export function load_body(scene: Scene) {
 2 |     const mesh = MeshBuilder.CreateSphere("body", { diameter: 2, segments: 32 }, scene)
 3 |     mesh.material = load_body_material(scene)
 4 | }
 5 | 
 7 | export function load_body_material(scene: Scene): StandardMaterial {
 8 |     const newMaterial = new StandardMaterial("bodyMaterial", scene)
 9 |     newMaterial.diffuseColor = Color3.FromHSV(320, 0.8, 0.65)
10 |     newMaterial.alpha = 0.9
11 |     newMaterial.specularColor = Color3.FromHSV(0, 0, 0)
12 |     return newMaterial
13 | }
```

コード6は変更したload_bodyです。Babylon.jsの機能で2行目の球体を作成しています。3行目でマテリアルを定義しています。作成した時点でScene内で作成されます。

8行目でBabylon.jsの基本的なマテリアルを定義しています。9～11行で順番に、色相、彩度、明度をRGBに変換して、透明度を少し薄く、反射を無くしています。12行目で返り値を返している理由はシーンに追加しても、どのメッシュにマテリアルを設定すればいいのか判断できるようにするためです。

### 二つ目の問題に対して

マテリアルが表示されない理由は、glbにエクスポートする時に、Geometry Nodesを読み込ませない設定にしていたからです。Geometry Nodesは実験的に実装されているので、使用すべきでないと判断しました。そして、通常のマテリアルが読み込まれるかどうか試しました。結果は画像16の通りです。なので、Babylon.jsでマテリアルを設定することにしました。実装したコードはコード7です。

```ts
コード7
 1 | export function createRingGradientMaterial(scene: Scene): NodeMaterial {
 2 |     const nodeMaterial = new NodeMaterial("ringGradientMaterial", scene, { emitComments: false })
 3 | 
 4 |     const position = new InputBlock("mesh.position")
 5 |     position.setAsAttribute("position")
 6 | 
 7 |     const world = new InputBlock("World")
 8 |     world.setAsSystemValue(NodeMaterialSystemValues.World)
 9 | 
10 |     const worldPos = new TransformBlock("WorldPos")
11 |     position.output.connectTo(worldPos.vector)
12 |     world.output.connectTo(worldPos.transform)
13 | 
14 |     const vectorSplitter = new VectorSplitterBlock("VectorSplitter")
15 |     worldPos.xyz.connectTo(vectorSplitter.xyzIn)
16 | 
17 |     const viewProjection = new InputBlock("View x Projection")
18 |     viewProjection.setAsSystemValue(NodeMaterialSystemValues.ViewProjection)
19 | 
20 |     const worldViewProjection = new TransformBlock("WorldPos & ViewProjection")
21 |     worldPos.output.connectTo(worldViewProjection.vector)
22 |     viewProjection.output.connectTo(worldViewProjection.transform)
23 | 
24 |     const vertexOutput = new VertexOutputBlock("VertexOutput")
25 |     worldViewProjection.output.connectTo(vertexOutput.vector)
26 |     nodeMaterial.addOutputNode(vertexOutput)
27 | 
28 |     const remap = new RemapBlock("Remap")
29 |     remap.sourceRange = new Vector2(-Math.PI, Math.PI)
30 |     remap.targetRange = new Vector2(0, 1)
31 | 
32 |     const ArcTanTwo = new ArcTan2Block("ArcTan2")
33 |     vectorSplitter.x.connectTo(ArcTanTwo.x)
34 |     vectorSplitter.z.connectTo(ArcTanTwo.y)
35 |     ArcTanTwo.output.connectTo(remap.input)
36 | 
37 |     const gradient = new GradientBlock("RainbowGradient")
38 |     gradient.colorSteps = [
39 |         new GradientBlockColorStep(0.0, new Color3(1, 0, 0)),
40 |         new GradientBlockColorStep(0.142, new Color3(1, 0.5, 0)),
41 |         new GradientBlockColorStep(0.284, new Color3(1, 1, 0)),
42 |         new GradientBlockColorStep(0.426, new Color3(0, 1, 0)),
43 |         new GradientBlockColorStep(0.568, new Color3(0, 0, 1)),
44 |         new GradientBlockColorStep(0.71, new Color3(0.29, 0, 0.51)),
45 |         new GradientBlockColorStep(0.852, new Color3(0.56, 0, 1)),
46 |         new GradientBlockColorStep(1.0, new Color3(1, 0, 0)),
47 |     ]
48 |     remap.output.connectTo(gradient.inputs[0])
49 | 
50 |     const fragmentOutput = new FragmentOutputBlock("FragmentOutput")
51 |     gradient.output.connectTo(fragmentOutput.rgb)
52 |     nodeMaterial.addOutputNode(fragmentOutput)
53 | 
54 |     nodeMaterial.build(false)
55 | 
56 |     return nodeMaterial
57 | }
```

コード7はノードマテリアルを使用して七色のマテリアルを作成しています。マテリアルを表示している箇所と、七色を作り出している箇所に分けて説明します。

コード7でマテリアルを表示している部分は、4～12行と17～26行の範囲です。ブロックで設定した名前で説目していきます。「mesh.position」、「World」、「WorldPos」と「View x Projection」、「WorldPos & ViewProjection」、「VertexOutput」と「FragmentOutPut」が基本的な構成です。FragmentOutPutに色情報を与えれば色が変わります。

接続方法について、基本的にはノードブロックのアウトプットから接続したいノードブロックに「connectTo」を使用して接続することができます。

色は「GradientBlock」で作成しています。このブロックをFragmentOutPutに接続しても色は変わりません。なぜなら、グラデーションをどこから始めるのかを指定していないためです。WorldPosのoutput.xyzからGradientのinputに接続すれば、グラデーションが反映されます。ですが、このままだと3D空間上のxy平面のみに適用されるグラデーションしか表示されません。

y軸を中心に円形のグラデーションを作るためには、「VectorSlitter」と「ArcTan2」と「Remap」が必要です。VectorSplitterは公式でも詳細を紹介されていません。おそらくですが、与えられたVectorを分割できるノードブロックだと思います。VectorSplitterで使用する軸はxzです。atan2は入力されたxz座標から角度を求めます。

$$
\mathrm{atan2}(z, x)
$$

円筒座標をで角度を求めるのと同じことをしていると考えてください。
[参考1](https://eman-physics.net/math/calculus04.html)
[参考2](https://www.cradle.co.jp/glossary/ja_A/detail0167.html)

ArcTan2の範囲は-π～πの範囲ですが、Remapが無いとGradientの影響で0～1の範囲しか扱われません。0～1の範囲は次のように考えるとわかりやすいかもです。円筒座標がz軸を縦軸としたxyz平面にあるとして、z軸の正の方向からxy平面を見たとします。この時の0～1の範囲は、sin(0°)=0～sin(π/2)=1になります。よって角度の値は0°～90°の値になります。結果、remapが存在しないとグラデーションの範囲が意図しなくなります。remapで範囲をArcTan2と同じにするために、コード7の29行目：remapの「sourceRange」を変更しています。

### 三つ目の問題に対して

5章で説明したinspectorを利用してvoice3、4をアニメーションさせると、最後の5フレームが両方とも消失して途中で止まってしまう現象に逢いました。原因を特定することはできませんでしたが、正しく読み込ませることはできました。blenderのモデルのアニメーションの再生範囲を変更することで正しく読み込ませることができます。

解決方法は、blenderのNLAウィンドウ内のトラック：Action Frameを変更することです。voice4、5の変更した箇所は画像19、20で確認です。

|画像19|画像20|
|---|---|
|![][4]|![][5]|

voice4はFrame start：0 --> 5、Frame End：92にしました。voice5はFrame start：5 --> 5、Frame End：94にしました。

どうしても原因がわからなかった。他のvoice1～3の三つは正常に再生できたので全く原因がわかりません。あるとしたら、セットしたアニメーションキーが多すぎたのかもしれません。

## さらに問題があった

デプロイするときにファイルの容量制限に引っ掛かってしまった。25MB以上のファイルは使用できないため、シェイプキーを削除する必要がありました。画像21のようにシェイプキーを削除する削除しました。

|画像21|
|---|
|![][6]|

画像11にあったシェイプキーの一部を消したものが画像21になります。やっていることはアニメーションの前後のシェイプキーを削除しているだけです。これをすべてのvoice1~5で実行します。メッシュの変形をしていない箇所のみ削除した結果、ファイルの容量は26645KB --> 24371KB になりました。

というか、24371KBは24MB以上なので、かなり重いです。サイトを読み込むときにでもかなり時間を要すると思います。なので、Babylon.jsの機能だけで実装する方法に変えました。次の章へ向かってください。