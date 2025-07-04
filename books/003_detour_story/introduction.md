---
title: "はじめに"
slug: "001_intro"
price: 0
---
[壱]:https://inxole.jp/
[弐]:youtu.be/1MdnR_J4QlI

[1]: /images/001/model_image.png "image_1"
[2]: /images/001/model_animation_image.png "image_2"

# この本について

Blenderで作成したモデルをBabylon.jsで表示し、音声を使用してメッシュを変形させる方法を紹介します。

## 対象

- React + Vite + Typescript を使える方
- Babylon.jsを使える方
- Babylon.jsで音声を使ってメッシュを変形させたい方

## 背景

### サイトにモデルが欲しかった
「動画を作ったときに使用したモデルをサイトに追加してみたいな。」という思いから実装しました。
①blenderでモデルを作成
②babylon.jsでモデルを実装
③音声でモデルを変形させる
①～③の順に、失敗も含めてどうやってサイトに実装したか説明していきます。

実装したサイトは以下のURLになります。
[https://inxole.jp/][壱]

作成した動画は以下のURLになります。
動画：[youtu.be/1MdnR_J4QlI][弐]

### モデルについて

作成したモデルは画像1になります。

|画像1：アニメーション前|画像2：アニメーション後|
|---|---|
|![][1]|![][2]|

画像1から画像2への変化は、中央の球体の白色の範囲が拡大しているのと、球体の周囲のキューブが波打っていることです。これをサイト上で表現しました。そして、アニメーション中は音声が流れるようにしています。

## Babylon.jsを使うわけ

babylon.jsを選んだ理由は、必要な拡張機能が少なく済むのと、大規模なものでも開発が容易になると予想したためです。

Babylon.jsに触れる前はThree.jsを使用していました。タグで３D空間を作成でき、ディレクトリ構造でCGを扱える便利な技術です。しかし、自分で作成したモデルを表示するのに別の拡張機能が必要でした。その仕様を把握する時間とthree.jsの3D空間に表示する手間がありました。babylon.jsなら必要な機能がすでにあり、外部の拡張機能を用意する必要が無いので便利だと思います。

他にも、行数を少なくすることができると思っています。例えば、three.jsで新しくモデルを作成する時「geometryの変数」と「mesh要素」が必要なので、最低でも二行必要です。これはBabylon.jsでも一緒です。しかし、さらに二つ目のメッシュを加えるとThree.jsでは4行、Babylon.jsでは3行となり、Three.jsは複雑になるほど行数またはファイル数が増えていきます。把握するのも大変になるので、大規模な開発には向かないと思います。

以上からBabylon.jsを選びました。

## 必要なもの

React + Vite + Typescriptの開発環境を用意することと、次のものをインストールすればいいだけです。

@babylonjs/core
@babylonjs/loaders
@babylonjs/gui
@babylonjs/inspector
@babylonjs/materials
@babylonjs/post-processes
@babylonjs/procedural-textures
@babylonjs/serializers

## それだけでわかるか～って方はこちら

先にnode.jsをインストールする必要があります。
Linuxを使用して次のコマンドでインストールします。

```bash
sudo apt update && sudo apt install nodejs npm
npm install pnpm
```

```bash
pnpm create vite
```

表示された内容にしたがってProject Name：任意、Select to framework：React ⇒ Typescript + SWCを実行します。テンプレートが自動で作成されます。次にnode_modulesを追加するためコマンドを実行します。

```bash
pnpm install
```

pnpmを使ってパッケージをインストールしてください。
```bash
pnpm add @babylonjs/core @babylonjs/loaders
```
devDependencies(-D)はお好みで
```bash
pnpm add @babylonjs/gui @babylonjs/inspector @babylonjs/materials @babylonjs/post-processes @babylonjs/procedural-textures @babylonjs/serializers -D
```
これで必要なパッケージがインストールされ、Babylon.jsを使用することができます。
