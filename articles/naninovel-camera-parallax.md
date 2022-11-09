---
title: '【Naninovel】パララックス（視差）を活用したカメラ演出'
emoji: '🦊'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['unity', 'csharp', 'game', 'visualnovel', 'naninovel']
published: true
---

# パララックスを活用したカメラ動作サンプル

https://youtu.be/g9pmXhMMGoE

::::details 上記動画の Naninovel スクリプト

:::details ※カメラの FOV とキャラ・背景の Z 位置を Naninovel デフォルト値から変更しています

- FOV: 60 -> 15.25
- キャラの Z Offset: 50 -> 30
- 背景の Z Offset: 100 -> 50
  :::

```nani
; カメラモードを Perspective に設定
@camera ortho:false

背景表示

; ※ Perspective モードでは、カメラから遠いオブジェクトが小さく描画されてしまう → scale で拡大
@back Kasenjiki scale:1.5

キャラクター表示

@char Schoolgirl scale:2

ズーム状態で左右移動

@camera zoom:.5
@camera offset:-5,0 time:.7
@camera offset:5,0 time:.7
@camera offset:-5,0 time:.7
@camera offset:5,0 time:.7
@camera offset:-5,0 time:.7
@camera offset:5,0 time:.7
@camera offset:-5,0 time:.7
@camera offset:0,0 time:.7

ズームリセット

@camera zoom:0 time:.7

ズーム & 左右移動

@camera zoom:.5 offset:-5,0 time:.7
@camera zoom:0 offset:0,0 time:.7
@camera zoom:.5 offset:5,0 time:.7
@camera zoom:0 offset:0,0 time:.7
@camera zoom:.5 offset:-5,0 time:.7
@camera zoom:0 offset:0,0 time:.7
@camera zoom:.5 offset:5,0 time:.7
@camera zoom:0 offset:0,0 time:.7

ズーム & 左右移動 & 回転

@camera zoom:.5 offset:-5,0 roll:15 time:.7
@camera zoom:0 offset:0,0 roll:0 time:.7
@camera zoom:.5 offset:5,0 roll:-15 time:.7
@camera zoom:0 offset:0,0 roll:0 time:.7
@camera zoom:.5 offset:-5,0 roll:15 time:.7
@camera zoom:0 offset:0,0 roll:0 time:.7
@camera zoom:.5 offset:5,0 roll:-15 time:.7
@camera zoom:0 offset:0,0 roll:0 time:.7

test
@stop
```

::::

:::details ※比較用：パララックスなしの通常動作
https://youtu.be/D4B73UWtZ1o
:::

# Naninovel のカメラモード

Naninovel のカメラ機能には `Orthographic` と `Perspective` という 2 種類のモードが用意されています。

- `Orthographic`
  - 奥行きを考慮しない 2D 表現モード
- `Perspective`
  - 奥行きを考慮した 3D 表現モード
  - → **パララックスを活用したカメラ演出が可能**

初期状態では `Orthographic` モードに設定されており、スクリプトファイルにて下記コマンドを実行することで `Perspective` モードに切り替えることが出来ます。

```nani
; Perspective モードに切り替える
@camera ortho:false
```

# Perspective モードの注意点

カメラからの距離（Z 軸方向の位置関係）が反映されるため、デフォルトの設定のままだと**キャラや背景が小さく描画**されてしまいます。

![](/images/naninovel-camera-parallax/naninovel-camera-perspective-default-size.png)

小さく描画されることを考慮して、

- キャラや背景の表示コマンドで `scale` パラメータを設定し、拡大表示する
- キャラや背景の Z 位置をカメラに寄せる
- カメラの設定で Field of View (FOV) を小さくする

と言った調整が必要となります。

# まとめ

- `@camera ortho:false` コマンドでカメラモードを `Perspective` に切り替えられる
  - → パララックスを活用したカメラ演出が可能となる
- `Perspective` モードではキャラや背景が小さく（遠くに）描画されてしまう
  - → キャラや背景のサイズ調整をする工夫が必要
