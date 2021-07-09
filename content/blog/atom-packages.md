---
title: "Atomで複数のパッケージをまとめてインストールする"
author: "miruo"
tags: []
date: 2021-07-09T12:10:50+09:00
draft: false
secret: false
---

# Atomで複数のパッケージをまとめてインストールする

いつも忘れるのでメモ

## インストール済みパッケージ一覧の作成

```sh
# バージョンを含める
apm list --installed --bare > packages.txt
# バージョンを含めない
apm list --installed --bare | sed -E "s/@.+//g" > packages.txt
```

## リストからインストール

```sh
apm install --packages-file packages.txt
```
