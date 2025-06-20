---
title: "babylon.jsでアニメーションを"
slug: "007_glb_analyzer"
price: 0
---

# babylon.jsでアニメーションを行う

個々では音声ファイルを利用してメッシュを変形させる方法を紹介します。Babylon.jsの機能のみで実装しています。モデルの作成、音声の順に説明します。

## モデルを作成する

キューブのモデルを作成します。次のコード8になります。

```ts
コード8
1  | const ringCount = 200
2  | const radius = 1.5
3  | const angleStep = (2 * Math.PI) / ringCount
4  | const squareSize_xz = 0.028
5  | const squareSize_y = 0.5
6  | const ring_tilt = Math.PI / 8.5
7  | function rings_instance(scene: Scene): Mesh[] {
8  |     const rings: Mesh[] = []
9  |     const parent = new Mesh("ring", scene)
10 |     for (let i = 0; i < ringCount; i++) {
11 |         const angle = i * -angleStep
12 |         const ring = MeshBuilder.CreateBox("ring" + i, {
13 |             width: squareSize_xz,
14 |             height: squareSize_y,
15 |             depth: squareSize_xz
16 |         }, scene)
17 |         ring.material = createRingGradientMaterial(scene)
18 |         ring.position = new Vector3(radius * Math.sin(angle), 0, radius * Math.cos(angle))
19 |         ring.rotation = new Vector3(ring_tilt, angle, 0)
20 |         ring.scaling.y = 0.4
21 |         ring.parent = parent
22 |         rings.push(ring)
23 |     }
24 |     return rings
25 | }
```

1～6行は上から、複製する数、半径、角度、xyのサイズ、zのサイズ、傾きを設定しています。6行目は約21°です。8行目で作成されたメッシュを格納する配列を作成し、9行目で親メッシュを作成しています。一度に移動させたい場合に使用できます。

for文では一つずつメッシュを作成しています。11行目でアングルを指定しておきます。12～16行目でメッシュを作成してシーンに追加しています。その際、名前を一意のものにしています。13～15行目はオプションでサイズを指定しています。17行でマテリアルを設定してます。18行目は、円形に配置するためにx軸はsinで指定し、y軸はcosで指定しています。これできれいに配置されます。20行目は、yのサイズに0.4の倍率を掛けていて、アニメーション前のサイズを指定しています。21行目で親子関係をつくり、22行目で8行目の配列に格納しています。最後の24行目でリングを返り値にしています。

マテリアルは「createRingGradientMaterial」で設定しています。詳細についてはコード7(有料)で確認して下さい。

## 音声データでメッシュを変形させる方法

音声ファイルを読み込んでメッシュを変形させる実装を順番に説明します。

### 音声ファイルの読み込み

音声ファイルを読み込み、音声解析を有効にします。

```ts
コード9
1  | export async function voice_loaded() {
2  |     return [
3  |         null,
4  |         await CreateSoundAsync("voice1", "/voices/001.wav", { loop: false, analyzerEnabled: true }),
5  |         await CreateSoundAsync("voice2", "/voices/002.wav", { loop: false, analyzerEnabled: true }),
6  |         await CreateSoundAsync("voice3", "/voices/003.wav", { loop: false, analyzerEnabled: true }),
7  |         await CreateSoundAsync("voice4", "/voices/004.wav", { loop: false, analyzerEnabled: true }),
8  |     ]
9  | }
```

CreateSoundAsyncについて、設定するときに必要なものは、名前、参照、オプションです。オプションのloopは一回だけ再生させるためにfalseに設定しています。オプションのanalyzerEnabledは周波数解析をさせるかどうか設定できます。trueなので解析可能です。

**重要**: 音声機能を利用するには、事前にオーディオエンジンの作成が必要です。Reactコンポーネント内でuseMemoを使用してオーディオエンジンを初期化してください：

```tsx
コード10
1  | useMemo(() => {
2  |     const loadSounds = async () => {
3  |         const audioEngine = await CreateAudioEngineAsync()
4  |         await audioEngine.unlockAsync()
5  |         soundsRef.current = (await voice_loaded()) as StaticSound[]
6  |     }
7  |     loadSounds()
8  | }, [])
```

サウンドだけでは音声は読み込まれないので、３行目でBabylon.jsの機能：オーディオエンジンを作成しておきます。４行目で音声を再生する準備ができるまで待つ処理をしています。

### 音声解析とメッシュ変形の実装

```ts
コード11
1  | function voice_analyzer(scene: Scene, sound: StaticSound, rings: Mesh[]) {
2  |     sound.volume = 0.75
3  |     const sound_analyzer = sound.analyzer!
4  |     sound_analyzer.fftSize = 512
5  |     sound_analyzer.smoothing = 0.3
6  | 
7  |     sound.play()
8  | 
9  |     const sampleRate = sound.buffer.sampleRate  // 48000
10 |     const binWidth = sampleRate / sound.analyzer.fftSize
11 |     const startIndex = Math.floor(20 / binWidth)
12 |     const endIndex = Math.ceil(20000 / binWidth)
13 |     
14 |     const handle = scene.onBeforeRenderObservable.add(() => {
15 |         const frequencyData = sound_analyzer.getByteFrequencyData()
16 |         const audibleRangeData = frequencyData.slice(startIndex, endIndex)
17 |         const maxIndex = Math.min(rings.length, audibleRangeData.length)
18 | 
19 |         for (let i = 0; i < maxIndex; i++) {
20 |             const phase = Math.PI * i / (maxIndex - 1)
21 |             const dynamicValue = audibleRangeData[i]
22 |             const wave_sine = Math.sin(phase * 6)
23 |             const scaledY = 0.4 + (dynamicValue * 0.01) * wave_sine
24 |             rings[i].scaling.y = scaledY
25 |         }
26 |     })
27 | 
28 |     sound.onEndedObservable.add(() => {
29 |         scene.onBeforeRenderObservable.remove(handle)
30 |     })
31 | }
```

2～4行目は上から、音量、解析するためのアナライザーを設定、高速フーリエ変換の周波数帯域を決める数値を指定(1回の解析で512個の周波数データを得る)、変化速度を滑らかにしています。7行目で音声を再生しています。

9、10行は上から、サンプリング周波数、周波数帯域を計算しています。11、12行は人間の可聴域を指定しています。

14行目でメッシュを変形させるイベントを加えています。15行目で高速フーリエ変換を実行、16行目で可聴域の音を配列に格納、17行目で使用する配列の要素を必ず200個以下になるようにしています。変形させるメッシュの数以上は無視されます。

for文内は次のことをしています。20行目で200個のメッシュを0～πの範囲で変形するようにしています。21行目で得られた帯域の音声データを格納格納しています。22行目は反周期に6を掛けて三周分の波形を計算しています。23行目で基本の大きさ0.4に、得られた音声データの変化を抑えるために0.01倍してとsine波を掛けた値を足しています。24行目でメッシュを変形させています。

28～30行は音声が停止したらメッシュの変形のイベントを消すためのものです。これでブラウザの負荷が軽減されます。

## アニメーションのトリガーの設定

必要は無いと思いますが、簡易的なアニメーションの起動方法を載せておきます。
音声アニメーションをボタンで制御するReactコンポーネントの例をコード12で示します。Jotaiを使用してコンポーネント間で状態を共有しています。

```tsx
コード12
1  | function VoiceController() {
2  |     const [voiceNumber, setVoiceNumber] = useAtom(voiceNumberAtom)
3  |     const [trigger, setTrigger] = useAtom(voiceTriggerAtom)
4  | 
5  |     const handlePlayVoice = (voiceIndex: number) => {
6  |         setVoiceNumber(voiceIndex)
7  |         setTrigger(prev => prev + 1)
8  |     }
9  | 
10 |     return (
11 |         <div>
12 |             <button onClick={() => handlePlayVoice(1)}>
13 |                 音声1を再生
14 |             </button>
15 |             <button onClick={() => handlePlayVoice(2)}>
16 |                 音声2を再生
17 |             </button>
18 |             <button onClick={() => handlePlayVoice(3)}>
19 |                 音声3を再生
20 |             </button>
21 |             <button onClick={() => handlePlayVoice(4)}>
22 |                 音声4を再生
23 |             </button>
24 |         </div>
25 |     )
26 | }
```

3Dシーンコンポーネントの例をコード13に示します。

```tsx
コード13
1  | function VoiceAnimationScene() {
2  |     const voiceNumber = useAtomValue(voiceNumberAtom)
3  |     const trigger = useAtomValue(voiceTriggerAtom)
4  | 
5  |     useEffect(() => {
6  |         if (!scene || !sounds) return
7  |         const selectedSound = sounds[voiceNumber]
8  |         if (selectedSound) {
9  |             voice_analyzer(scene, selectedSound, rings)
10 |         }
11 |     }, [voiceNumber, trigger])
12 | 
13 |     return <canvas ref={canvasRef} />
14 | }
```

この実装では`useAtom`と`useAtomValue`を使用してJotaiのatomで状態を管理し、コンポーネント間でvoiceNumberとtriggerの状態を共有しています。ボタンクリック時にatomの値が更新され、3Dシーンコンポーネントがその変化を検知してアニメーションを実行します。