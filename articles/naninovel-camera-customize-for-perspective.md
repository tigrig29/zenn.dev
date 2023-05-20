---
title: '【Naninovel】キャラ・背景間のパララックス（視差）表現をするカメラ設定とカスタマイズ'
emoji: '🦊'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['unity', 'csharp', 'game', 'visualnovel', 'naninovel']
published: false
---

# 環境

- Unity 2021.3.12f1
- Naninovel ( https://naninovel.com/ ) v1.18 build 2022-09-07

# Naninovel のカメラモード解説

https://naninovel.com/api/#camera

Naninovelのカメラ機能には `Orthographic` モードと `Perspective` モードの2種類が用意されており、 `@camera` コマンドの `ortho` パラメータで切り替えることが可能です。

```
; Orthographic モードに切り替える（デフォルト）
@camera ortho:true
```

```
; Perspective モードに切り替える
@camera ortho:false
```

## Orthographic モードの動作サンプル

`Orthographic` モードでは、奥行きを考慮せず平面上に背景・キャラが存在するような表現となります。

https://youtu.be/D4B73UWtZ1o

:::details 上記動画の Naninovel スクリプト

```
;@camera ortho:true ← ※デフォルトで true となっているため不要

背景表示

@back Kasenjiki

キャラクター表示

@char schoolgirl scale:2

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
:::

## Perspective モードの動作サンプル

`Perspective` モードでは、奥行きを考慮するためカメラズームや移動時に「手前側のオブジェクトは大きく動き、奥側のオブジェクトは小さく動く」という視差が生じます。

https://youtu.be/g9pmXhMMGoE

:::details 上記動画の Naninovel スクリプト

```
@camera ortho:false

背景表示

; ※ Perspective モードでは、カメラから遠いオブジェクトが小さく描画されてしまう → scale で 1.5 倍に拡大
@back Kasenjiki scale:1.5

キャラクター表示

@char schoolgirl scale:2

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
:::

# Perspective モード向けのカメラカスタマイズ

## 目的

1. `Perspective` モードをデフォルト設定としたい
2. `Perspective` モード使用時、奥行きが反映されるためにキャラや背景が画面サイズに対して極端に小さく描画されてしまう問題を改善したい

## 結論： `CameraManager` をオーバーライド

> エンジンサービスのオーバーライド手順は以下
> https://naninovel.com/guide/engine-services.html#overriding-built-in-services

```csharp:CustomCameraManager.cs
using UnityEngine;

namespace Naninovel
{
    [InitializeAtRuntime(-1, @override: typeof(CameraManager))]
    public class CustomCameraManager : CameraManager
    {
        private float initialOrthoSize, initialFOV;

        public CustomCameraManager (CameraConfiguration config, IInputManager inputManager, IEngineBehaviour engineBehaviour) :
        base(config, inputManager, engineBehaviour)
        { }

        // CameraManager サービスの初期化処理
        public override async UniTask InitializeServiceAsync ()
        {
            await base.InitializeServiceAsync();

            // 下記 ApplyZoom をオーバーライドするにあたって orthographicSize の初期値を拾っておく
            initialOrthoSize = Camera.orthographicSize;
            // 下記 ApplyZoom をオーバーライドするにあたって fieldOfView の初期値を拾っておく
            initialFOV = Camera.fieldOfView;
        }

        // タイトル画面からの初回ゲームスタート時に実行される処理
        public override void ResetService ()
        {
            base.ResetService();
            // base.ResetService() で Orthographic モードに設定されてしまうため、Perspective モードに切り替える
            Orthographic = false;
        }

        // base.InitializeServiceAsync() 内で呼び出されるカメラの初期化処理
        protected override Camera InitializeMainCamera (CameraConfiguration config, Transform parent, int uiLayer)
        {
            var camera = base.InitializeMainCamera(config, parent, uiLayer);

            // デフォルトで Perspective モードに設定
            camera.orthographic = false;

            // Project Settings > Naninovel > Camera     > Z Offset = -10
            //                                Characters > Z Offset =  30 のとき、
            // Perspective モードにおけるキャラクターサイズが Orthographic モードのキャラクターサイズとほぼ同じになる FOV を設定
            // ※元は camera.fieldOfView = 60f;
            camera.fieldOfView = 15.35f;

            return camera;
        }

        // @camera コマンドの zoom パラメータ指定時のズーム処理
        protected override void ApplyZoom (float zoom)
        {
            if (Orthographic) Camera.orthographicSize = initialOrthoSize * (1f - Mathf.Clamp(zoom, 0, .99f));
            else
            {
                // InitializeMainCamera で FOV 初期値を小さめに設定したため、
                // Zoom 処理の FOV 最小値（Mathf.Lerp の第一引数）も小さめに設定
                // ※元は Mathf.Lerp(5f, initialFOV, 1f - zoom)
                Camera.fieldOfView = Mathf.Lerp(1f, initialFOV, 1f - zoom);
            }
        }
    }
}
```

## 解説



# Perspective モードの表示サイズ調整

Perspective モードでは奥行きを表現するため、 Orthographic モードに比べて各素材が小さく表示されてしまいます。

> Naninovel のデフォルト設定状態で

## Z Offset の調整

# Perspective モードをデフォルトに設定

Naninovelのデフォルトカメラモードは `Orthographic` です。

`Perspective` モードをデフォルトに設定したいと思い、以下の対応をしました。


Project Settings > Naninovel > Camera > Custom Camera Prefab に `Projection` が `Perspective` のカメラプレハブを設定する方法でも可能です。

私の場合は前述の [Perspective モードの表示サイズ調整](#perspective-モードの表示サイズ調整) で `CameraManager` を override する必要が生じていたため、まとめて `CameraManager` の方で対応しました。

