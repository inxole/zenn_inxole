---
title: "babylon.jsをreact+typescripで実装する"
slug: "005_blender_in_react"
price: 0
---

# babylon.jsをreactで使用する

functionごとに説明します。

## Babylon.jsでglbファイルを読み込む

用意しておいたsrcディレクトリに移動して、App.tsxで新しいコンポーネントを作成してください。私はCampusにしています(コード1)。

```ts
コード1
1  | function App() {
2  |   return (
3  |     <div style={{ display: "flex", flexDirection: "row", height: "100%" }}>
4  |       <Campus />
5  |     </div>
6  |   )
7  | }
```

Campusコンポーネントについて紹介します（コード2）。

```ts
コード2
1  | function Campus() {
2  |     const canvasRef = useRef<HTMLCanvasElement | null>(null)
3  |     const sceneRef = useRef<Scene | null>(null)
4  | 
5  |     useEffect(() => {
6  |         if (!canvasRef.current) return
7  |         load_Assistant(canvasRef.current, sceneRef)
8  |     }, [])
9  | 
10 |     return (
11 |         <canvas ref={canvasRef} style={{ width: '30%', height: '100%' }} />
12 |     )
13 | }
```

2、3行目で変数を定義します。canvasRefとsceneRefにします。useRefは公式(React)によると「レンダリングに必要のない値を参照できるフック」だそうです。二つの変数はBabylon.jsのレンダイングで使用するためuseEffectを使わないといけません。

5～8行目はuseEffectを使用して、３D空間を作成する関数：Load_Assistantを実行させます。関数は初回レンダリング時にだけ実行させます。引数のcanvasRef.currentは内部でエンジンを作成させるために使用します。引数のsceneRefはCampusコンポーネントで新たにメッシュなどを加えるために必要です。

10～12行目にcanvasを使用して、３Dの描画を可能にします。refを使用してcanvas要素内にcanvasRefを渡します。これで要素自体を変更できます。

次に関数「Load_Assistant」を紹介します。

```ts
コード3
1  | async function load_Assistant(
2  |     canvas: HTMLCanvasElement,
3  |     sceneRef: React.RefObject<Scene | null>,
4  | ) {
5  |     const engine = new Engine(canvas, true)
6  |     const scene = new Scene(engine)
7  |     scene.clearColor = new Color4(0, 0, 0, 0)
8  | 
9  |     Load_camera(scene, canvas)
10 |     Load_Light(scene)
11 |     
12 |     sceneRef.current = scene
13 |     
14 |     engine.runRenderLoop(() => scene.render())
15 |     const resize = () => engine.resize()
16 |     window.addEventListener('resize', resize)
17 |     return () => {
18 |         engine.dispose()
19 |         window.removeEventListener('resize', resize)
20 |     }
21 | }
```

5行目で引数のcanvasを使用してDOMにエンジンを作成します。6行目で作成されたエンジンを利用してシーンを作成します。７行目はシーン全体の背景を白色にしています。

9、10行目でカメラとライトを作成しています。12行目でsceneRefの初期値を作成したsceneに置き換えてます。これで親コンポーネントで作成したメッシュなどシーンから探して使用することができます。

14行目は描画をループさせます。無い場合はエラーを起こす可能性が高いです。15行目はウィンドウ(ChromeやEdge等)の大きさが変更されたときにcanvasのサイズを変更するためのものです。17行目はサイトの動きを低下させないために利用しています。「dispose」や「remove」で不要なものは消してメモリリークを解放するそうです。これは知らなかった。