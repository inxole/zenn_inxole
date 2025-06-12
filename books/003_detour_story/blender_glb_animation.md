---
title: "blenderでアニメーションを"
slug: "003_blender_glb_animation"
price: 500
---

[1]: /images/003/wave_all.png "image_1"
[2]: /images/003/voice2_sample.png "image_2"
[3]: /images/003/nla_tracks.png "image_3"

# シェイプキーに変換してアニメーションを読み込めるようにする方法(失敗談)

シェイプキーをBabylon.jsで扱えることはできましたが、容量で問題があった話しです。

## 波形データをシェイプキーに変換

シェイプキーを雑に説明すると、同じ頂点数、見た目は異なるのメッシュを利用して、互いの形をアニメーションで行き来できるものです。これを大量に作成させる方法を説明します。

### 大量のシェイプキーを作成

blenderのGeometry nodesで作成したデフォームアニメーションはgltf2.0でエクスポートすることはできません。なぜなら、公式で「Export Geometry nodes instances. This feature is experimental.」と書いてあるから、実験的なものらしいです。(まぁ、音声データやアニメーションカーブをglbのデータと一緒にすると重くなるだろうし、描画を簡単にする目的から離れるから難しいのかも？)

|画像10|
|---|
|![][1]|

画像10は各EmptyのCustom Propertiesで読み込んだ波形です。読み込んだ音声は画像①～④の四つです。30fpsで設定しています。

四つの波形を順番に一つにまとめたものが画像10の⑤になります。⑤の最初は一秒間だけ何もしない状態を用意するために30フレームほど余分に足しています。

①：88 f ②：156 f ③：83 f ④：113 f です。
フレームの合計は、最初の30frameも合わせて 416 fです。

各フレームのシェイプキーを一つずつ作成するのは非常に手間なので、コード1を使用します。

```python
コード1
import bpy

start_frame = 1
end_frame = 416
anim_name = "Wave"

# === 元オブジェクトを取得 ===
src_obj = bpy.context.active_object
if not src_obj:
    raise Exception("オブジェクトが選択されていません。")

# === bakeターゲット（Shape Key登録先）を複製作成 ===
bpy.ops.object.duplicate()
bake_target = bpy.context.active_object
bake_target.name = f"{src_obj.name}_bake"
bpy.ops.object.convert(target='MESH')
bpy.ops.object.shape_key_add(from_mix=False)  # Basis

# === 各フレームのメッシュ状態を保存 ===
all_frame_vertices = []

for frame in range(start_frame, end_frame + 1):
    bpy.context.scene.frame_set(frame)

    # 複製 → GN適用
    bpy.ops.object.select_all(action='DESELECT')
    src_obj.select_set(True)
    bpy.context.view_layer.objects.active = src_obj
    bpy.ops.object.duplicate()
    temp_obj = bpy.context.selected_objects[-1]
    bpy.context.view_layer.objects.active = temp_obj
    bpy.ops.object.mode_set(mode='OBJECT')
    bpy.ops.object.convert(target='MESH')

    # 頂点座標を取得
    vertex_positions = [v.co.copy() for v in temp_obj.data.vertices]
    all_frame_vertices.append(vertex_positions)

    # 一時オブジェクト削除
    bpy.data.objects.remove(temp_obj, do_unlink=True)

# === Shape Key登録 ===
bpy.context.view_layer.objects.active = bake_target
bake_target.select_set(True)

for idx, verts in enumerate(all_frame_vertices):
    frame_num = start_frame + idx
    key_name = f"{anim_name}_{frame_num}"
    bpy.ops.object.shape_key_add(from_mix=False)
    sk = bake_target.data.shape_keys.key_blocks[-1]
    sk.name = key_name
    sk.value = 1
    for i, v in enumerate(verts):
        sk.data[i].co = v

print("Shape Keys を {start_frame}～{end_frame} フレーム分一括作成しました。")
```

これで大量のシェイプキーが作成されます。おそらく画面のメッシュはかなり大きい変化になると思います。元のメッシュにしたい場合は全てのシェイプキーの値を0にしてください。元の大きさに戻るはずです。今回はそのままにして次に進んでください。

### 大量のキーフレームを作成する

次に大量のシェイプキーでアニメーションキーを設定していきます。該当するフレームのシェイプキーの値を1でキーをセットして、該当するフレームの前後の値は0にしてキーをセットする。これを全てのシェイプキーで行います。シェイプキーの最初と最後のキーのセットは二つだけですけどね。416個x3-2＝1246個のキーをセットするので大変です。なので、次のスクリプトで自動で作成していきます。

```python
コード2
import bpy
import re

start_frame = 1
end_frame = 416 # 任意

obj = bpy.context.active_object
if not obj or not obj.data.shape_keys:
    raise Exception("Shape Key を持つオブジェクトを選択してください。")

key_blocks = obj.data.shape_keys.key_blocks
pattern = re.compile(r"Wave_(\d+)$")

for key in key_blocks:
    match = pattern.match(key.name)
    if not match:
        continue

    wave_num = int(match.group(1))
    for frame in range(start_frame, end_frame + 1):
        if wave_num == frame:
            key.value = 1.0
            key.keyframe_insert(data_path="value", frame=frame)
        elif wave_num == frame - 1 or wave_num == frame + 1:
            key.value = 0.0
            key.keyframe_insert(data_path="value", frame=frame)

print("アクションを作成しました。") 
```

紹介したPythonコードは両方ともメッシュを選択する必要があるので注意してください。
作成されたシェイプキーを確認するには、Animation Windowに移動してGraph Editorを「Dope Sheet」に変更してください。さらに「Dope Sheet」を「Shape Key Editor」にしてください。

メッシュがとても大きく変化していると思いますが、そのままで大丈夫です。気になるのなら「Shape Key Editor」で全てのシェイプキーを0にしてみてください。見た目が元に戻るはずです。

## NALにしてActionsに登録

NLA(NonLiner Animation)を使う必要は無いのですが、特定の理由があるため、あえてそうしています。

### アニメーションを四つに分ける

コード2を実行した時点でActionsが作成されています。先ほど作成したシェイプキーを複製して五つに分けてください。私はフェイクユーザーを使用してvoice1 ~ voice5に分けました。この状態で各シェイプキーに分けていきます。

使用するフレームは以下になります(画像10より)。
voice1   0 ~  30(アニメーションさせないためのデータ)
voice2  30 ~ 155
voice3 154 ~ 248
voice4 248 ~ 335
voice5 335 ~ 416

|画像11|
|---|
|![][2]|

|画像12|
|---|
|![][3]|

画像11はvoice2のシェイプキーです。不要なフレームのキーを削除したら、キー全体を移動してください。キー全体の一番左端を1 frame目まで移動してください。移動しておかないとBabylon.jsで読み込んだ時、不要なフレームが発生してしまいます。例えば、voice2でキーを移動していないと30frameほどプラスされたアニメーションが実行されます。なので、0 ~ 155 までのフレームが格納されてしまいます。

画像11の上部に「Push Down」があります。これでNLA内のトラックとして使用できます。プッシュダウンをして「NonLiner Animation」windowに切り替えて画像12のようにしてください。おそらく、すべて一フレームから始まっているはずなので移動させてください。そうしないとシェイプキーがブレンドされてしまうので、タイムライン上で被らないようにしてください。