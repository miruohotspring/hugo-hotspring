---
title: "[Amplify]ビルド時にaws-exportsを自動生成"
author: "miruo"
tags: ['aws', '備忘録']
date: 2021-04-19T16:34:10+09:00
draft: false
secret: false
---

※ビルドオプションを訂正しました

# 概要
Amplify CLIを使ってAPIなどを追加していざコミットすると、 ```Cannot find file './aws-exports' in './src'``` と言われてビルドが失敗した話。

# 結論
アプリケーション自体にバックエンドのサービスをビルドする権限を付与して、ビルドオプションに ```amplify push``` を追加すると、都度aws-exportsを生成してくれるようになる。

# 方法
## ロールの作成
IAMマネジメントコンソールからロールの作成->Amplify->Amplify - Backend Deploymentを選択し、保存。

## ロールの付与
Amplifyコンソールからアプリの設定->編集->Service Roleで先ほど作成したロールを選択。

## ビルドオプションの追記
同じくAmplifyコンソールのビルドの設定から、amplify.ymlに以下を追記する。

```yaml
backend:
  phases:
    build:
      commands:
        - amplifyPush --simple
```

再度デプロイし直せば無事ビルドできるようになります。

# 参考
[Build Fails Only on Amplify with Error "Cannot find file '../aws-exports' in './src/components'." #5979](https://github.com/aws-amplify/amplify-js/issues/5979)

[Adding a service role to the Amplify Console when you connect an app](https://docs.aws.amazon.com/ja_jp/amplify/latest/userguide/how-to-service-role-amplify-console.html)
