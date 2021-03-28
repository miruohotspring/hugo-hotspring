---
title: "GitHub Actionsを利用してAWS Lambdaにコードをデプロイする"
author: "miruo"
tags: ["aws", "備忘録"]
date: 2021-03-28T02:50:36+09:00
draft: false
---
# 概要

GitHub ActionsでAWS CLIコマンドを叩くことでAWS Lambdaへのデプロイを自動化する。

# IAMユーザーとポリシーの作成

GitHub ActionsからAWSにアクセスするためのユーザーとそのポリシーを作成する。作成したユーザーのアクセスキーIDとシークレットアクセスキーは後でGitHubのSecretsに登録する。

## ポリシーの作成

1. AWSマネジメントコンソールから IAM → ポリシー → ポリシーの作成
2. JSONタブに以下のコードを貼り付ける 

    ```json
    {
      "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": [
              "s3:PutObject",
              "iam:ListRoles",
              "lambda:UpdateFunctionCode",
              "lambda:CreateFunction",
              "lambda:UpdateFunctionConfiguration"
            ],
            "Resource": "*"
          }
        ]
    }
    ```

3. "MinimumDeployRoleForLambda"等の名前をつけてポリシーを保存

## ユーザーの作成

1. AWSマネジメントコンソールから IAM → ユーザー → ユーザーの追加
2. "GitHubActions"等の名前をつけて、"プログラムによるアクセス"にチェックを入れる
3. アクセス許可の設定 → 既存のポリシーを直接アタッチ と進み、先ほど作成したポリシーにチェックを入れて次のステップへ
4. タグの設定 → 確認 → ユーザーの作成 と進んだら**「成功」と表示された画面を絶対に閉じない**

# Secretsを設定

1. レポジトリのSettingsタブ → Secrets → New Repository Secrets
2. "AWS_ACCESS_KEY_ID" "AWS_SECRET_ACCESS_KEY"という名前で先ほど作成したIAMユーザーのアクセスキーIDとシークレットアクセスキーをそれぞれ登録する

# GitHub Actions

1. GitHub上でレポジトリ内のActionsタブをクリックすると.ymlの雛形を自動で生成してくれる
2. .ymlに以下のコードを記載する

    ```yaml
    name: YOUR_WORKFLOW_NAME

    on:
        push:
            branches:
                - YOUR_BRANCH_NAME
        workflow_dispatch:
            branches:
                - YOUR_BRANCH_NAME

    jobs:
        <YOUR_JOB_NAME>:
            runs-on: YOUR_ENVIRONMENT_NAME
            steps:
                - uses: actions/checkout@v2
                    with:
                        ref: YOUR_BRANCH_NAME
                - uses: actions/setup-python@v1
                    with:
                        python-version: 3.8
                - run: zip -r package.zip ./*
                - run: pip3 install awscli
                - run: aws lambda update-function-code --function-name YOUR_FUNCTION_NAME --zip-file fileb://package.zip --publish
                    env:
                        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
                        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
                        AWS_DEFAULT_REGION: YOUR_REGION_NAME
    ```

- YOUR_WORKFLOW_NAME ... workflow名。お好きなもので大丈夫です。
- YOUR_BRANCH_NAME ... このブランチにpushした際にworkflowが実行される。ちなみにmasterの場合、actions/checkout@masterが利用できます。
- YOUR_JOB_NAME ... job名。これも好きな名前でOK．
- YOUR_ENVIRONMENT_NAME ... 実行したい環境名。ubuntu-latestが無難。
- YOUR_FUNCTION_NAME ... デプロイしたいLambda関数名。
- YOUR_REGION_NAME ... リクエストを送信するリージョン。ap-northeast-1など。
- on: workflow_dispatchを記述することで、GitHubのActions画面からworkflowを手動で実行できるようになります。

以上で設定は終わりです、お疲れ様でした。awscliをインストールしてawsコマンドを使用しているのがわかりますね。それにしてもたった1行で簡単にデプロイが出来てしまうAWS CLIすごい。

<お好きな名前>.ymlを保存すると、その瞬間にworkflowが走ります。この時、レポジトリ内にlambda-functionファイルがないとAWS CLIから怒られてしまいます。事前に作成しておきましょう。(LambdaでPythonを使用している場合はlambda-function.py）

# 終わりに

今回作成したIAMポリシーは、Lambda関数をデプロイするために必要な最低限のアクションのみ利用しています。関数の削除などをする場合は適宜ポリシーを変更する必要があるため、よしなに。
