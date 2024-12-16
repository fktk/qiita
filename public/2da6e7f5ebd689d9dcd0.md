---
title: GoogleEarth上でGPSのルートを移動するツアーの作成
tags:
  - JavaScript
  - KML
  - GPS
  - GoogleEarth
private: false
updated_at: '2022-07-19T23:40:36+09:00'
id: 2da6e7f5ebd689d9dcd0
organization_url_name: null
slide: false
ignorePublish: false
---
![tour (1).gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2591762/c2e8872c-7a4e-859a-9d46-a8cf57d4eb1c.gif)

## 概要

トップの動画のようにGoogleEarth上で、あるルートを移動するツアープロジェクトを作成する方法です。この動画は、三重県をサイクリングしたときのGPSデータから作成しました。
GPSデータを3D空間上で表示したいと思ったのですが、3D空間をフリーで提供しているサービスはGoogleEarthしか見つからなかったため、最終的にKMLファイルにツアーを記述して、GoogleEarthにレンダリングしてもらう方法にたどり着きました。

## 作成方法

基本的な説明は[Googleのドキュメント](https://developers.google.com/kml/documentation/touring?hl=ja)に書いてある通りです。ドキュメントには比較的遠い距離を飛ぶサンプルしかありませんが、近距離のデータを多数つなぎ、ルート上を移動しているように見せることができます。

ではKMLファイルにGoogleEarthが解釈できるツアー要素を記述していきます。基本的な構造は、以下のような形です。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<kml
  xmlns="http://www.opengis.net/kml/2.2"
  xmlns:gx="http://www.google.com/kml/ext/2.2"
>
  <gx:Tour>
    <gx:Playlist>
      ここにgx:FlyTo要素を足していく
    </gx:Playlist>
  </gx:Tour>
<kml>
```

\<gx:Playlist>の中に\<gx:FlyTo>要素を追加していくことで、各地点間を移動することができます。

一般的なGPSデータは、位置一点分のデータとしてタイムスタンプと位置情報(緯度、経度、標高)が含まれており、それが複数個集まって一つのルートになっています。そこでGPSデータをJavaScriptやPythonなどで読み込み、そのデータを使って\<gx:FlyTo>内に視点を制御する\<Camera>要素と、時間を制御する\<gx:duration>の数値を指定します。
位置一点分のデータに対応する\<gx:FlyTo>要素は以下のような形です。

```xml
<gx:FlyTo>
  <gx:duration>2.0</gx:duration>
  <gx:flyToMode>smooth</gx:flyToMode>
  <Camera>
    <longitude>x.xx</longitude>
    <latitude>y.yy</latitude>
    <altitude>z.zz</altitude>
    <heading>h.hh</heading>
    <tilt>t.tt</tilt>
    <altitudeMode>absolute</altitudeMode>
  </Camera>
</gx:FlyTo>
```

視点(Camera)の位置(longitude、latitude)はGPSデータを使います。高さ(altitude、altitudeMode)は標高データを使いますが、少し下駄を履かせたほうがGoogleEarth上の3Dオブジェクトに押されにくくなり、鑑賞時に見やすいです。
向き(heading)は位置2点分のデータを使って計算します。傾き(tilt)はいくつか試して良さそうな数値を探します。
durationはこの位置に移ってくる時間ですが、位置間の移動速度が最大100km/h程度であれば。私の3年前のスマホで十分キレイにレンダリングされます。
向きや距離の計算には、Node.jsの場合[geolib](https://www.npmjs.com/package/geolib)が便利です。

上記のようなKML構造をテンプレート文字列化してfunction化し、引数として数値を渡すとKMLファイルができあがります。注意点としては、

* 3D描画中、ビルなどの3Dオブジェクトよりも視点が低いと、視点がオブジェクトの高さに瞬間移動する
* レンダリングが遅いデバイスや、通信速度が遅い環境だとカクついたり、建物がきれいに描画されなかったりする

などがあります。視点の高さを高くしたり、durationを長くとることで多少の改善が可能です。

以上です。もっと実在の場所をモデル化した3D空間が気軽に扱えると面白そうなのになぁ。。。
