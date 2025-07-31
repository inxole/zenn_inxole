---
title: "FreeMoCapでモーションキャプチャを始める：セットアップから3Dデータ出力まで"
emoji: "🎥"
type: "tech"
topics: ["FreeMoCap", "Blender", "Python"]
published: true
---

[壱]:https://github.com/pyenv/pyenv
[弐]:https://python-poetry.org
[参]:https://freemocap.github.io/documentation/installation.html#detailed-pip-installation-instructions
[肆]:https://github.com/freemocap/freemocap_blender_addon

[1]: /images/freemocap_texture/001_free_mo_cap.png "image_1"
[2]: /images/freemocap_texture/002_we_do_it.png "image_2"
[3]: /images/freemocap_texture/003_oops_delete_recommend.png "image_3"
[4]: /images/freemocap_texture/004_mocap_data.png "image_4"
[5]: /images/freemocap_texture/005_local_directory.png "image_5"

[I]: /images/freemocap_texture/freemocap_test.gif "movie_1"

## FreeMoCapとは？

FreeMoCapは、無料で使えるオープンソースのモーションキャプチャーツールです。複数カメラを使って人の動きをキャプチャし、CSVやBlender形式で保存することができます。

## FreeMoCapセットアップ手順

ローカルに直接インストールすると他のプロジェクトと競合するため仮想環境で実行しました。仮想環境 ⇒ FreeMoCapの順に説明します。
使用した各ツールのバージョンはこちらです。
```
pyenv + python 3.12.0
poetry 1.7.1
blender 4.4.3
```

### 環境の準備

仮想環境構築には `poetry`を使用しました。以下のコードを実行してプロジェクトを作成します。

```pwsh
poetry init
```

以下のようにして実行しました。
```
Package name [directory_name]:push "Enter"
Version [0.1.0]:push "Enter"
Description []:  Create motion data using FreeMoCap.
Author [user_name <user_name@gmail.com>, n to skip]:  user_name
License []:  MIT
Compatible Python versions [^3.12]:push "Enter"
Would you like to define your main dependencies interactively? (yes/no) [yes] no
Would you like to define your development dependencies interactively? (yes/no) [yes] no
Do you confirm generation? (yes/no) [yes] yes
```

ライブラリをインストールしていないのはFreeMoCapが依存関係管理されたパッケージ（dependency-managed package）だからです。

`pyenv` と `poetry` のインストールは、それぞれの[公式GitHubページ][壱]、[Poetry公式][弐]を参照してください。

### FreeMoCapのインストール

仮想環境に以下のコードで入ります。
```pwsh
poetry shell
```

次に[FreeMoCap公式GitHub][参]より
```pwsh
pip install freemocap
```
インストールされたら次のコマンドで起動します。
```pwsh
freemocap
```

## FreeMoCapの使い方

### 使用する前に

GUIを起動後、画像1のFreeMoCapと画像2の注釈？が表示されます。

|画像1：FreeMoCap|
|---|
|![][1]|

|画像2：注釈v1.6.0について|
|---|
|![][2]|

「右上の×」を押して初期チュートリアルを閉じてください。すると次に画像3でライブラリの変更を推奨されます。

|画像3：FreeMoCap|
|---|
|![][3]|

OpenCVが競合するため「opencv-contrib-python」を使用することを勧めています。なので、「Fix OpenCV conflict (Recommended)」を押してください。
自動で再起動されないので再度Powershell上で「freemocap」を実行してください。

### 使い方

画像1の「Home」メニューから「New Recording」を選びます。次に「Detect Cameras」でカメラを検出します。

画像1の「Camera」メニューに自動的に切り替わるので、右側に表示されるカメラの設定を調整してください。私はデフォルトのままにしています。

「Cameras」メニューで「Start Motion Capture Recording」で録画開始。モーションを撮影し、「Stop Recording」で停止します。

自動でデータ(画像4)が作成されるので待ちます。他にも「timestamp_diagnostic_plots.png」と「first_and_last_frames.png」が作成されます。

|画像4：3D、2Dのデータ|
|---|
|![][4]|

ログに「Auto Open in Blender checkbox is checked - triggering 'Create Blender Scene'.」と表示されていれば、Blenderとの連携が成功します。どのバージョンのBlenderを使用するか選んでください。自動でBlenderが開き、モーションデータと映像データが両方インポートされたシーンを作成するようです。
実際に作成されたものが映像1になります。

|映像1：FreeMoCapの骸骨|
|---|
|![][I]|

映像1から、カメラ一台だと奥行き情報がうまく読み取れないようです。撮影時にカメラから離れてしまったため、xy平面から離れた位置にデータが作成されてしまいました。
キャリブレーションもカメラ一台だと動作しないようで、奥行き情報が縦軸の情報に誤認？されるかもしれません。カメラは二台以上がいいですね。

処理が完了すると、「Directory View」メニューで以下のような構成のフォルダが生成されます。

```
recording_XX_XX_XX_gmt+9/
├── 3D_models/
├── annotated_videos/
├── output_data/
├── saved_data/
├── synchronized_videos/
├── recording_xx.blend
├── recording_xx.ipynb
├── recording_xx_by_frame.csv
└── recording_xx_by_frame.json
```

「Active Recording Info」で現在の録画データの情報を確認可能です。

## まとめ

ローカル環境で簡単にモーションキャプチャーができるのでお勧めです。公式ドキュメントも分かりやすいので導入が簡単でした。
何らかのサービスに課金することもなく、高額な機械もいらず、その場でできる。すごい技術だ。

さて、二台目のカメラを買ってこよ。