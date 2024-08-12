---
title: "第2章: AWSのネットワーク"
---

第2章: AWS ネットワークの世界へようこそ！ 🌐
AWS には VPC っていう自分だけのネットワークを作れるサービスがあるんだ。😄
そこにはサブネットやルーティングテーブル、ファイアウォールみたいなセキュリティグループとかがあって、安全で使いやすいネットワークが作れるよ。
他にもインターネットに繋げるためのゲートウェイとか、違うネットワークと繋げるためのピアリングとかの機能もあるんだ。ログを記録したりもできるから、ネットワークの問題が起きてもすぐ対処できるね！

AWS でのネットワーク構築の基礎を理解しよう！

AWS を使いこなすためには、ネットワークの知識は必須です！この章では、AWS ネットワークの基本的な概念から、実際にサービスを構築する際に必要となる重要な要素まで、わかりやすく解説していきます。🚀

2.1 VPC（仮想プライベートクラウド）とは？
VPC は、AWS 上で独自の仮想ネットワーク環境を構築できるサービスです。🏢

VPC を使用することで、オンプレミス環境と同じように、独自のサブネット、ルーティングテーブル、セキュリティグループなどを設定できます。これにより、AWS 上で安全でセキュアなネットワーク環境を構築することができます。🔒

VPC のメリット:

分離されたネットワーク環境: 他のアカウントやサービスと隔離された、独自のネットワーク環境を構築できます。
柔軟な設定: サブネット、ルーティングテーブル、セキュリティグループなどを自由に設定できます。
セキュリティ強化: VPC を使用することで、ネットワークセキュリティを強化できます。
プライベートネットワーク: インターネットに公開されていない、プライベートなネットワーク環境を構築できます。
VPC の構成図：


2.2 サブネットとは？
サブネットは、VPC 内のネットワークをさらに分割するための要素です。

各サブネットは、独自の CIDR ブロック（IPアドレス範囲）を持ち、それぞれに異なるセキュリティ設定やルーティング設定を適用できます。

サブネットの例：

ウェブサーバーを配置するサブネット
データベースサーバーを配置するサブネット
アプリケーションサーバーを配置するサブネット
サブネットのメリット:

分離: 異なる目的のサーバーを、それぞれ独立したサブネットに配置することで、セキュリティを強化できます。
セキュリティ設定: 各サブネットに異なるセキュリティグループや Network ACL を適用できます。
ルーティング: 各サブネットに異なるルーティングテーブルを適用できます。
2.3 ルーティングテーブルとは？
ルーティングテーブルは、ネットワークトラフィックがどの経路で転送されるかを定義するものです。

ルーティングテーブルには、ルートと呼ばれる情報が登録されており、各ルートには、宛先ネットワークと次ホップが設定されています。

ルーティングテーブルの例：

宛先ネットワーク: 10.0.0.0/16
次ホップ: Internet Gateway
ルーティングテーブルのメリット:

トラフィックの制御: ネットワークトラフィックを適切な経路に転送できます。
セキュリティ強化: 特定のネットワークへのアクセスを制限できます。
柔軟性: ルーティングテーブルを柔軟に設定できます。
2.4 インターネットゲートウェイとは？
インターネットゲートウェイ（IGW）は、VPC 内のインスタンスがインターネットにアクセスできるようにするサービスです。🌎

IGW は、VPC とインターネットを接続するためのゲートウェイとして機能します。

IGW のメリット:

インターネットアクセス: VPC 内のインスタンスがインターネットにアクセスできます。
セキュリティ強化: IGW は、VPC のセキュリティ境界として機能します。
柔軟な設定: IGW を使用することで、インターネットアクセスを柔軟に制御できます。
IGW の構成図：


2.5 NAT ゲートウェイとは？
NAT ゲートウェイは、VPC 内のプライベートインスタンスがインターネットにアクセスできるようにする、別の方法です。

NAT ゲートウェイは、ネットワークアドレス変換 (NAT) を行い、プライベートインスタンスからインターネットへの通信を、パブリック IP アドレスを使用して行うことができます。

NAT ゲートウェイのメリット:

プライベートネットワーク: インスタンスはプライベート IP アドレスを使用できます。
インターネットアクセス: プライベートインスタンスがインターネットにアクセスできます。
セキュリティ強化: NAT ゲートウェイは、プライベートネットワークとインターネットの間のセキュリティ境界として機能します。
NAT ゲートウェイの構成図：


2.6 EIP（弾性IPアドレス）とは？
EIP は、動的パブリック IP アドレスです。

EIP は、インスタンスに割り当てることで、インスタンスが停止または再起動されても、パブリック IP アドレスが維持されます。

EIP のメリット:

固定IPアドレス: インスタンスのIPアドレスが変更されません。
高可用性: インスタンスが停止または再起動されても、IPアドレスが維持されます。
セキュリティ強化: EIP を使用することで、セキュリティを強化できます。
2.7 セキュリティグループとは？
セキュリティグループは、インスタンスへのネットワークアクセスを制御するファイアウォールのようなものです。🔒

セキュリティグループは、インバウンドルールとアウトバウンドルールを使用して、インスタンスへのアクセスを許可または拒否します。

セキュリティグループの例：

インバウンドルール: ポート80（HTTP）からのアクセスを許可する
アウトバウンドルール: すべてのネットワークへのアクセスを許可する
セキュリティグループのメリット:

セキュリティ強化: 特定のポートや IPアドレスからのアクセスを制限できます。
柔軟な設定: セキュリティグループは、インスタンスごとに異なる設定を適用できます。
管理の容易さ: セキュリティグループは、簡単に作成、変更、削除できます。
2.8 ネットワークACLとは？
ネットワークACL（Network Access Control List）は、サブネットレベルでネットワークアクセスを制御するファイアウォールです。🔒

ネットワークACLは、インバウンドルールとアウトバウンドルールを使用して、サブネットへのアクセスを許可または拒否します。

ネットワークACLの例：

インバウンドルール: ポート80（HTTP）からのアクセスを許可する
アウトバウンドルール: すべてのネットワークへのアクセスを許可する
ネットワークACLのメリット:

サブネットレベルのセキュリティ: サブネット全体にセキュリティを適用できます。
セキュリティ強化: 特定のポートや IPアドレスからのアクセスを制限できます。
柔軟な設定: ネットワークACLは、サブネットごとに異なる設定を適用できます。
セキュリティグループとネットワークACLの違い:

適用範囲: セキュリティグループはインスタンスレベルで適用され、ネットワークACLはサブネットレベルで適用されます。
ルール: セキュリティグループのルールは、インスタンスへのアクセスを制御し、ネットワークACLのルールは、サブネットへのアクセスを制御します。
優先順位: ネットワークACLは、セキュリティグループよりも優先順位が高いです。
2.9 VPC ピアリングとは？
VPC ピアリングは、2つの異なる VPC を接続するサービスです。🤝

VPC ピアリングを使用すると、2つの VPC 内のインスタンスが、まるで同じ VPC 内にいるかのように通信できます。

VPC ピアリングのメリット:

ネットワークの拡張: 複数の VPC を接続することで、ネットワークを拡張できます。
セキュリティ: VPC ピアリングは、2つの VPC 間のセキュリティ境界を維持します。
コスト削減: VPC ピアリングを使用することで、複数の VPC を別々に管理する必要がなくなり、コストを削減できます。
VPC ピアリングの構成図：


2.10 ENI（弾性ネットワークインターフェース）とは？
ENI は、インスタンスに割り当てることができるネットワークインターフェースです。

ENI は、インスタンスに複数のネットワークインターフェースを追加したり、異なるサブネットに接続したりすることができます。

ENI のメリット:

高帯域幅: ENI を使用することで、インスタンスの帯域幅を向上させることができます。
高可用性: ENI を使用することで、インスタンスの可用性を向上させることができます。
ネットワーク分離: ENI を使用することで、インスタンスのネットワークを分離できます。
2.11 VPC エンドポイントとは？
VPC エンドポイントは、VPC 内のインスタンスが、インターネットにアクセスすることなく、AWS サービスにアクセスできるようにするサービスです。

VPC エンドポイントには、ゲートウェイ型とインターフェース型の2種類があります。

ゲートウェイ型:

AWS サービスへのアクセス: VPC 内のインスタンスが、プライベート IP アドレスを使用して、AWS サービスにアクセスできます。
インターネットへのアクセス: インターネットへのアクセスは許可されません。
インターフェース型:

AWS サービスへのアクセス: VPC 内のインスタンスが、プライベート IP アドレスを使用して、AWS サービスにアクセスできます。
インターネットへのアクセス: インターネットへのアクセスは許可されません。
直接接続: インターフェース型は、AWS サービスに直接接続されます。
VPC エンドポイントのメリット:

セキュリティ強化: VPC 内のインスタンスが、インターネットにアクセスすることなく、AWS サービスにアクセスできます。
パフォーマンス向上: VPC エンドポイントを使用することで、パフォーマンスを向上させることができます。
コスト削減: VPC エンドポイントを使用することで、インターネットトラフィックを削減できます。
2.12 プライベートリンクとは？
プライベートリンクは、VPC 内のインスタンスが、インターネットにアクセスすることなく、AWS サービスやオンプレミス環境にアクセスできるようにするサービスです。

プライベートリンクは、AWS サービスへのアクセスとオンプレミス環境へのアクセスの2種類があります。

プライベートリンクのメリット:

セキュリティ強化: VPC 内のインスタンスが、インターネットにアクセスすることなく、AWS サービスやオンプレミス環境にアクセスできます。
パフォーマンス向上: プライベートリンクを使用することで、パフォーマンスを向上させることができます。
コスト削減: プライベートリンクを使用することで、インターネットトラフィックを削減できます。
2.13 VPC FlowLogとは？
VPC FlowLog は、VPC 内のネットワークトラフィックに関する情報をログに記録するサービスです。

VPC FlowLog を使用することで、ネットワークトラフィックを監視し、セキュリティ問題を検出できます。

VPC FlowLog のメリット:

セキュリティ監視: VPC FlowLog を使用することで、ネットワークトラフィックを監視できます。
問題解決: ネットワークの問題を解決するために、トラフィック情報をログに記録できます。
コンプライアンス: コンプライアンス要件を満たすために、トラフィック情報をログに記録できます。
2.14 まとめ
本章では、AWS ネットワークの基本的な概念と、サービスを構築する際に必要となる重要な要素について学びました。これらの知識を基に、AWS 上で安全で高性能なネットワーク環境を構築していきましょう！😊

今後の学習:

AWS 上で実際にネットワーク環境を構築してみましょう。
AWS のセキュリティサービスについて学び、ネットワークをより安全にしましょう。
AWS のネットワーク関連の認定資格を取得しましょう。