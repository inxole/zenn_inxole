---
title: "はじめに"
slug: "intro"
price: 0
---
[壱]:youtu.be/1MdnR_J4QlI
[弐]:https://tome-bs-and-ss.inxole.jp/

[1]: /images/001/model_image.png "image_1"
[2]: /images/001/model_animation_image.png "image_2"

# この本について

Blenderで作成したモデルをBabylon.jsで表示する方法を紹介します。

## 対称

Babylon.jsをある程度使える方
Babylon.jsのNodeMaterialをスクリプトで実現したい方。

## 背景

### サイトにモデルが欲しかった
動画を作ったときに使用したモデルをサイト([https://tome-bs-and-ss.inxole.jp/][弐])に追加してみたいな。
という望みから試してみました。
動画：[youtu.be/1MdnR_J4QlI][壱]

### モデルはこれです⇩
|画像1|画像2|
|---|---|
|![][1]|![][2]|

画像１はアニメーション前、画像２はアニメーション後のモデルです。変化は中央の球体の白色の範囲が拡大しているのと、球体の周囲のキューブが波打っていることです。これをサイト上で実現したいです。そして、アニメーション中は音声が流れるようにします。

## Babylon.jsを使うわけ

babylon.jsを選んだ理由は、拡張性が高いことと実装が容易だからです。

Babylon.jsに触れる前はThree.jsを使用していました。しかし、自分で作成したモデルを表示するのに別の技術が必要でした。その仕様を把握する時間とthree.jsの空間に表示する手間がありました。しかし、babylon.jsなら必要なプラグインが既にそろっていました。

## 最初の準備

次のものをインストールすればいいだけです。

@babylonjs/core
@babylonjs/loaders
@babylonjs/gui
@babylonjs/inspector
@babylonjs/materials
@babylonjs/post-processes
@babylonjs/procedural-textures
@babylonjs/serializers"

loadersでglbがロードできます。

## それだけでわかるか～って方はこちら

先にnode.jsをインストールする必要があります。
linuxを使用して次のコマンドでインストールします。


```bash
sudo apt update && apt install node npm
npm install pnpm
```

pnpmを使ってパッケージをインストールしてください。
```bash
pnpm add @babylonjs/core @babylonjs/loaders
```
devDependencies(-D)はお好みで
```bash
pnpm add @babylonjs/gui @babylonjs/inspector @babylonjs/materials @babylonjs/post-processes @babylonjs/procedural-textures @babylonjs/serializers -D
```
これで必要なパッケージがインストールされます。

## （愚痴）
因みに、pnpmの本体をアップデートするように要求されるときがある。その場合は以下のコマンドで対応できる。
```bash
npm update pnpm
```
pnpmでパッケージを追加しているとたまに「Update available」と出て「pnpm add -g pnpm」と推奨コマンドを教えてくれます。指示に従ってやってみても「pnpm setup」と出ます。セットアップしても効果が無いので次のコマンドをやってみるのがいいかもです。
オプションで「-g」を付けるべきかもしれないです。