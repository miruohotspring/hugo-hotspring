---
title: "Beatport x Spotify でEDMジャンル分類器を作る その1"
author: "miruo"
tags: ["プロジェクト"]
date: 2021-09-15T03:09:46+09:00
draft: false
secret: false
---
# Beatport x Spotify でEDMジャンル分類器を作る その1
## 概要
こんにちは。お久しぶりになってしまいすみません、みるをです。

皆さんは今聴いているEDM曲がどんなジャンルの音楽なのか、気になったことはありませんか？僕はあります。

Beatportは全ての楽曲にジャンルがタグ付けられていてとても便利ですが、
Beatportにない曲でもジャンルが知りたいですよね。例えばSpotifyにある曲。
Spotifyにも一応のジャンル分けがありますが、特にEDMに限るとなるとその分類は少し大雑把すぎます。
その代わりに，energyやdanceability, acousticnessなど、様々な観点からの楽曲の特徴をメタデータとして保持しています。

そこで、BeatportにもSpotifyにもある曲のうち，
Spotify上のメタデータとBeatportのジャンルを組み合わせとした教師データを用意して分類器を学習させれば、
BeatportにはなくSpotifyにある曲もそのメタデータからジャンルを推定することが出来るのではないかと考えました。

今後何回かに分けて進捗を報告していこうと思います。

## 前提
想定している分類器は以下のようなものです。
- 入力: Spotifyのメタデータ
- 出力: Beatport上のジャンル

ロジスティック回帰モデルを使用する予定です。
Pythonを使って実装していきます。

## 教師データの準備
教師データを用意します。まずはBeatportとSpotify両方にある曲を見つけ、そのメタデータとジャンルを組にする必要があります。
Beatport上にある曲をジャンルごとに取得してSpotify上で検索し、同じ曲が見つかればその組を保存するという作業をします。
検索にはISRCコードを使用し、確実に同じ曲を取得できるようにします。

### Beatportからジャンルごとに曲を取得
[BeatportのAPI](https://www.beatport.com/api/v4/catalog)には
'genres'といういかにもそれらしいカテゴリが存在し、そこから現在Beatport上で用いられている31個のジャンル一覧を取得することが出来ました。

このリストにはジャンルごとの曲一覧を返すようなエンドポイントも含まれており、ラッキーと思っていたのですが、
どうやらこれはクライアントが叩くことを想定されていないようで残念ながら取得することが出来ませんでした。
一曲ごとのデータは取れるのになぜ・・・

仕方ないのでwww.beatport.com/genre/~のhtmlからトラックIDをスクレイピングして一曲ずつISRCを取得するという方法を使いました。これがめちゃくちゃ時間かかる（この記事はこの時間に書かれています）

とりあえず各ジャンルごとに100曲ずつISRCを取ってきてテキストファイルに保存する処理を書きました。以下コード。

```python
def create_isrc_files(n):
    r = requests.get("https://www.beatport.com/api/v4/catalog/genres")
    lists = r.json()["results"]
    
    for i, genre in enumerate(lists):
        print("loading " + genre["name"] + f"...[{i+1}/{len(lists)}]")
        track_ids = []
        isrc = []
        slug = genre["slug"]
        id = genre["id"]
        path = f"./isrc/{slug}.txt"
        for batch in range(n):
            uri = f"https://www.beatport.com/genre/{slug}/{id}/tracks?page={batch+1}"
            r = requests.get(uri)
            track_ids.extend(re.findall(r'data-track="([0-9]+)', r.text))
        print("loaded ids")
        print("loading ISRC...")
        for j, track_id in enumerate(track_ids):
            print(f"[{j+1}/{len(track_ids)}]")
            uri = f"https://www.beatport.com/api/v4/catalog/tracks/{track_id}"
            r = requests.get(uri)
            try:
                isrc.append(r.json()["isrc"])
            except:
                print("no isrc")
        print(isrc, file=codecs.open(path, 'w', 'utf-8'))
```

今回はここまでです。次回はSpotifyのAPIを叩いて各曲のメタデータを取りたいと思います。

ちなみにごくわずかですがISRCが登録されていない曲もあり、例外処理を書き足すはめになりました。
また、スクレイピングが上手くいってないのかたまに100曲中99曲しか読み込めなかったりします。

レポジトリは[こちら](https://github.com/miruohotspring/edmga-trainer)

それではまた次回。