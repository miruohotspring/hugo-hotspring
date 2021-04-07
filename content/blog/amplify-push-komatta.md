---
title: "schema.graphqlを編集してamplify pushしようとしたらハマった時のこと"
author: "miruo"
tags: ['aws', '備忘録']
date: 2021-04-07T23:28:10+09:00
draft: false
secret: false
---

# 起きたこと
```.txt
Attempting to add and remove a global secondary index at the same time on the HogeHoge table in the HogeHoge stack. 
```

# やったこと
global secondary indexを追加したり削除したりするなということなので、schema.graphqlの該当箇所をコメントアウト。

```.graphql
#@key(name: "SortByCreatedAt", fields:["type", "createdAt"], queryField: "listPostsSortedByCreatedAt")
#@key(name: "BySpecificOwner", fields:["owner", "createdAt"], queryField: "listPostsBySpecificOwner")
```

\@keyのnameがそれですね。

この状態で `amplify push` して、再びコメントアウトしてからもう一度 `push` 

# 結果
```.txt
Attempting to create an index which already exists
```
resource failed to update, failed to createのオンパレード。どうして。

# 敗北
結局、schema.graphqlのバックアップを取ってから `amplify remove api` して、再度 `amplify add api` しました。

自動生成されたものの他にqueryやmutationを作成していた場合は、それらのバックアップも忘れずに。

# まとめ
開発初期段階であれば問題ないのですが、かなり最終手段に近いです。

こまめにpushするべきなのか、それとも何か方法があるのか・・・

何か分かったら更新します。