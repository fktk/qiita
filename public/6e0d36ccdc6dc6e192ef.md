---
title: Bokeh 凡例クリックにカスタムイベントを登録する
tags:
  - Python
  - Bokeh
private: false
updated_at: '2022-07-06T00:09:46+09:00'
id: 6e0d36ccdc6dc6e192ef
organization_url_name: null
slide: false
ignorePublish: false
---
Pythonのグラフライブラリ [Bokeh](https://docs.bokeh.org/en/latest/index.html)のTIPSです。

凡例クリックで、関連するグラフを一斉に表示させたり、非表示にしたりしたいことありますよね。
ですがBokehでは、凡例にカスタムイベントを直接登録することはできなさそうでした。

解決法を探したところ、[スタックオーバーフローにありました。](https://stackoverflow.com/questions/70392754/how-to-create-working-callback-on-legenditem-in-bokeh)
プロット(Renderer)の表示非表示の状態変化には、カスタムイベントが登録できるようです。

簡単な例ですが、以下のような感じです。

```python
from bokeh.plotting import figure, save
from bokeh.layouts import gridplot
from bokeh.models import CustomJS

p1 = figure(width=400, height=200)
p2 = figure(width=400, height=200)

r1 = p1.circle([1, 2, 3], [5, 3, 1], legend_label='main')
r2 = p2.line([1, 2, 3], [3, 1, 5])

# 凡例へのクリックポリシーの指定は、プロットを作成した後でないと正常に動かない
p1.legend.click_policy = 'hide'

# プロットr1の表示非表示の状態に対し、カスタムイベントを付与する
r1.js_on_change(
    'visible',
    CustomJS(
        args=dict(r2=r2),  # r2をJavaScriptに渡す
        code='r2.visible = !(r2.visible);'))  # JavaScriptでr2の表示状態を反転する

save(gridplot([p1, p2], ncols=1), filename='test.html')
```

![Screen-recording-2022-07-05-23.56.09.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2591762/dac6be4a-bcf5-13bd-502c-6c498d4eafdd.gif)

狙い通り動きました！
