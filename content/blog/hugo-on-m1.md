---
title: "M1 MacでHugoを使う"
author: "miruo"
tags: ["備忘録"]
date: 2022-04-30T20:15:00+09:00
draft: false
secret: false
---

長らくブログを更新していない間にPCをM1 Macに買い替えていたので、更新のテストもかねてメモを上げておきます。

# 試したこと

```.sh
brew install hugo
```
Rosettaを使え的なことを言われる。

# 解決策

GoをインストールしてソースからHugoをビルドする。

## Goのインストール

[ダウンロードページ](https://go.dev/dl/)からApple 64-bit processorと書かれているパッケージファイルをダウンロードして、インストールを行う。

```
$go version
go version go1.18.1 darwin/arm64
```

## Hugoのビルド

```
$git clone https://github.com/gohugoio/hugo.git
$cd hugo
$go build
```

すると、直下にhugoのバイナリファイルが作られている。

```
$./hugo version
hugo v0.99.0-DEV-e5f21731696bd4a9a396936b18d9ae72291b01b1 darwin/arm64 BuildDate=2022-04-28T15:47:17Z
$sudo mv ./hugo /usr/local/bin
$which hugo
/usr/local/bin/hugo
```