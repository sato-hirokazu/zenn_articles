---
title: "AWS CDK v2への移行手順(TypeScript)"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

AWS CDK v1は2022年6月からメンテナンスモードに入りました。

AWS CDK v2への移行手順は以下に公式ドキュメントがあります。

実際にTypeScript版で移行を実施したため、簡潔に知りたい方向けにこの記事でサマリします。

https://docs.aws.amazon.com/ja_jp/cdk/v2/guide/migrating-v2.html#migrating-v2-prerequisites

移行編
1. cliのアップデート
npm update -g aws-cdk
2. AWS CDK v2用のパッケージインストール（aws-cdk-lib, constructs）
npm install aws-cdk-lib constructs
3. AWS CDK v1用のパッケージ削除
AWS CDK v1用のパッケージ削除
@aws-cdk/core
@aws-cdk/aws-apigateway などの各種サービス用パッケージ
4. import文を修正
パターン1：aws-cdk-libのインポート
@aws-cdk を aws-cdk-lib に書き換える。
ファイル上部のインポート文を直接置換すると速い。

注意：'@aws-cdk/core' だけは 'aws-cdk-lib' に書き換えます。'aws-cdk-lib/core' ではありません。
【修正イメージ】
- import * as cdk from '@aws-cdk/core'
+ import * as cdk from 'aws-cdk-lib'

パターン2：constructsのインポート
Constructをインポートしている部分は、Constructパッケージからインポートする。

- import { Construct } from '@aws-cdk/core'
+ import { Construct } from 'constructs'

パターン3：deprecatedになっているAPIの修正
v2に上げたことでdeprecatedになっている・削除されているメソッドがある場合は、その部分に修正を入れる。

※これはv1→v1の新しいバージョンへのアップデートでも発生します。

下記のように、公式ドキュメントに代わりに使ってほしいAPIや、その使い方のサンプルなど丁寧に載っている。



パターン4：cdk.json内の不要なフラグ削除
v2ではv1時代のcdk.jsonのフラグはデフォルトの動作になっている。また、削除されているものもある。

以下のような、context配下のフラグを全て削除する。


5. cdk diffで差分が出ないことを確認
筆者環境ではIAM Role周りの差分のみ発生したが、問題なくデプロイできた。

6. ハマった場合に確認
筆者環境では特に発生しなかった。
例えば以下のようなトラブルシュートが存在する模様。

TypeScriptのバージョンアップ
For TypeScript developers, TypeScript 3.8 or later is required.
cdk bootstrapコマンドの再実行
CDK v2 furthermore requires a new version of the modern stack. Simply re-bootstrap your existing environments to upgrade them.
詳細はこちらを参照。


補足編：v2に移行するメリット
以下のようなメリットがある。

aws-cdk-libから大半インポートでき、構築や運用（バージョン管理）が楽
これは移行しながら実際に便利になるイメージが掴めました。
semverの導入
コミュニティとしてリリースバージョンをsemverにきっちり従わせることで、マイナーやパッチのアップデートを安心してできるようにする
強化されたAPI Reference（コピペできるサンプルがついている）
watch modeの導入
専用テストライブラリの導入
4、5については、別途検証記事を投稿したいと思います。

参考：https://aws.amazon.com/about-aws/whats-new/2021/12/aws-cloud-development-kit-cdk-generally-available/

追伸1
実際に移行しながら、aws-cdk-lib一本からインポートができて生産的だなぁという実感が湧いたかと思います。これだけで移行する価値があると思いました。

また、ドキュメントも確かに強化されていて、サンプルコードつきなのでdeprecatedなAPIの置き換えも爆速で対応できました（以前はなかった気がしています）。

加えて、専用テストライブラリは気になっています。v1時代は「リソースが存在するか」レベルのテスト以上はあまり現実的ではなかったように思うので、どのくらいセマンティックにテストできるようになっているのか、ワクワクしています。
