---
title: "Amplifyコンソールでビルド時にaws-exports.jsを自動生成"
author: "miruo"
tags: ['aws', '備忘録']
date: 2021-04-19T16:34:10+09:00
draft: false
secret: false
---

# 概要
Amplify CLIを使ってAPIなどを追加していざコミットすると、コンソールでaws-exports not foundというようなエラーが出てビルドが失敗した話。

# 結論
アプリケーションにロールを付与してビルドオプションに ```amplify push``` を追加すると、都度aws-exportsを生成してくれるようになる。

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
        - '# Get Amplify CLI Cloud-Formation stack info from environment cache'
        - export STACKINFO="$(envCache --get stackInfo)"
        - '# Execute Amplify CLI with the helper script'
        - amplifyPush --environment $AWS_BRANCH
        - '# Store Amplify CLI Cloud-Formation stack info in environment cache'
        - >-
          envCache --set stackInfo "$(amplify env get --json --name
          $AWS_BRANCH)"
```

再度デプロイし直せば無事ビルドできるようになります。