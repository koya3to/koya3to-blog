+++ 
draft = false
date = 2022-07-02T22:00:09+09:00
title = "FeatureGroupとLayerControlを使って地図に表示する情報を制御する+popupをいい感じに整形する"
description = "FeatureGroupとLayerControlを使って地図に表示する情報を情報の種類に応じて制御する+popupをいい感じに整形します"
slug = ""
authors = []
tags = []
categories = ['data visualization', 'foliuim']
externalLink = ""
series = []
+++

## 初めに

地図情報を可視化するPythonパッケージとして代表的なものにfoliumがあります．
今回はfoliumを利用して以下の2つの可視化を行う際の方法について紹介します．

- 情報の種類ごとに表示有無を制御する方法
- ポップアップ（文化財がクリックされたときに表示されるウィンドウ）の表示をテーブル形式でいい感じに表示する方法

具体例として，国が公開している[都道府県指定文化財データ](https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-P32.html)を利用して，文化財の種類ごとに表示有無を制御します．また，各文化財がクリックされた場合の文化財情報をテーブル形式でいい感じに表示します．

コードの全体については[こちら](https://github.com/koya3to/blog-ingredient/blob/main/notebooks/FoliumFeatureGroup.ipynb)においてあります．

## コードの説明

### 情報の種類に応じて表示有無を切り替える

方針としては，

1. ベースレイヤーを作成する．
2. FeatureGroupで文化財の種類ごとにグループを作成する
3. 作成したグループにマーカーを追加する
4. それぞれ作成したグループをベースレイヤーに追加する．
5. レイヤーコントロールをベースレイヤーに追加して，制御可能にする

実際のコードは以下のようになります．

```python
import folium
from folium import FeatureGroup, LayerControl

# ベースレイヤーを作成する
m = folium.Map(location=['37.5178', '138.9270'], zoom_start=10)

for index_i, prop_type in enumerate(cultural_gpd['種別大分'].value_counts().index):
    show = False
    if prop_type in ['有形文化財', '無形文化財']:
        show=True
    # FeatureGroupで文化財の種類ごとにグループを作成する．showで最初にどの情報を選択するか制御可能
    temp_group = FeatureGroup(name=prop_type, show=show) 
    for index_j, row in cultural_gpd[cultural_gpd['種別大分']==prop_type].iterrows():
        # 作成したグループにマーカーを追加する
        folium.Marker(
        location=[row['geometry'].y, row['geometry'].x],
        popup=make_popup_html(row),
        icon=folium.Icon(color=dic_color[str(row['P32_004'])]),
        ).add_to(temp_group)
    
    # グループをベースレイヤーに追加する．
    temp_group.add_to(m)

# レイヤーコントロールをベースレイヤーに追加して，制御可能にする
folium.LayerControl(collapsed=False).add_to(m)
m
```

### ポップアップをいい感じに表示する

上のコードでは，make_popup_htmlという関数を作成して，これをMarkerのpopup引数に渡しています．これにより，ポップアップの表示をいい感じにしています．make_popup_htmlについては以下のような関数になっていて，html形式で表示方法を記述しています．

```python
def make_popup_html(row):
    popup=folium.Popup(html = f"""
        <table border="1">
            <caption> {row['P32_006']}<br></caption>
            <tr>
                <td>所在地
                <td>{row['P32_007']}</td>
            </tr>
            <tr>
                <td>種別大分</td>
                <td>{row['種別大分']}</td>
            </tr>
        </table>
    """,
    max_width=400,
    )
    return popup
```

## 結果

最終的な結果は以下のようになります．右上のパネルで表示する文化財の種類を選択することが可能です．
すべて表示したほうが綺麗だったので，自分の方でチェックボックスにチェックを入れ，すべての文化財の表示を行っています．実際には，グループを作成するときに，'有形文化財'と'無形文化財'だけを表示するように指定しているため，実際にコードを動かすと最初に表示されるのは'有形文化財'と'無形文化財'の2つになるかと思います．
また，マーカーをクリックすることで，表敬式で各文化財の情報を可視化することが可能になっています．

![最終的な出力](/fig/folium-layer/result.png)

## まとめ

今回はfoliumを利用した可視化例として，以下の2点について紹介しました．

- 国が公開している都道府県指定文化財データを利用して，文化財の種類ごとに表示有無を制御する．
- 各文化財がクリックされた場合の文化財情報をテーブル形式でいい感じに表示する．

他にもいろいろなデータを可視化することが可能なので，皆さんぜひfoliumをいじってみてくださいね．

## 参考

- [都道府県指定文化財データ](https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-P32.html)
- [Leaflet](https://leafletjs.com/)
- [folium](https://python-visualization.github.io/folium/)
- [コード全体](https://github.com/koya3to/blog-ingredient/blob/main/notebooks/FoliumFeatureGroup.ipynb)
