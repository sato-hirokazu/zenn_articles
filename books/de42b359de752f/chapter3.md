---
title: "第3章: AWSのロードバランサー"
---

第3章: AWS ロードバランサーの世界へようこそ！ ⚖️
ロードバランサーっていうのは、たくさんのサーバーに仕事を分けてあげる係みたいなものなんだ。😄
例えば、人気のゲームで同時にいっぱいの人が遊んでいる時、ロードバランサーが、みんなからの要求を受けて、空いているサーバーにうまく振り分けてくれるんだよ。
これによって、サーバーが忙しくなりすぎないし、みんなが快適にゲームを楽しめるようになるんだ。
AWS には、アプリケーションロードバランサーとネットワークロードバランサーという2種類のロードバランサーがあって、用途に合わせて使い分けられるよ。✨

アプリケーションの負荷分散をマスターしよう！

アプリケーションの負荷分散は、安定したサービス提供とパフォーマンス向上に欠かせない技術です。AWS は、様々な種類のロードバランサーを提供しており、アプリケーションのニーズに合わせて最適なロードバランサーを選択することができます。🚀

3.1 ロードバランサーとは？
ロードバランサーとは、複数のサーバーに負荷を分散することで、アプリケーションの安定性とパフォーマンスを向上させるサービスです。

ロードバランサーは、トラフィックを複数のサーバーに分散することで、単一のサーバーへの負荷を軽減します。また、サーバーの障害が発生した場合でも、他のサーバーにトラフィックを転送することで、サービスの可用性を高めます。

ロードバランサーのメリット：

高可用性: サーバーの障害が発生した場合でも、サービスが継続的に提供されます。
スケーラビリティ: 負荷に応じて、簡単にサーバーを追加したり削除したりできます。
パフォーマンス向上: 複数のサーバーに負荷を分散することで、パフォーマンスを向上させることができます。
セキュリティ強化: ロードバランサーは、アプリケーションのセキュリティを強化できます。
3.2 ELB（Elastic Load Balancing）とは？
ELB は、AWS が提供するロードバランサーサービスです。

ELB は、アプリケーションのニーズに合わせて、様々な種類のロードバランサーを選択できるように設計されています。

ELB の種類：

クラシックロードバランサー: 従来のアプリケーションに適したロードバランサーです。
アプリケーションロードバランサー (ALB): モダンなアプリケーションに適したロードバランサーです。
ネットワークロードバランサー (NLB): 高パフォーマンスなアプリケーションに適したロードバランサーです。
3.3 ALB（Application Load Balancer）とは？
ALB は、モダンなアプリケーションに適したロードバランサーです。

ALB は、HTTP/HTTPS トラフィックを、複数のサーバーに分散することができます。また、パスベースルーティング、スティッキーセッション、HTTPS ダイレクトなどの機能も提供しています。

ALB の特徴:

HTTP/HTTPS トラフィックのロードバランシング: HTTP/HTTPS トラフィックを、複数のサーバーに分散することができます。
パスベースルーティング: リクエストのパスに基づいて、異なるサーバーに転送することができます。
スティッキーセッション: 特定のユーザーを、常に同じサーバーに接続することができます。
HTTPS ダイレクト: ALB は、HTTPS トラフィックを暗号化して転送することができます。
ALB の構成図：


説明:

ユーザーは ALB にアクセスします。
ALB は、ターゲットグループにリクエストを転送します。
ターゲットグループは、複数のサーバー（EC2 インスタンスまたはコンテナ）にリクエストを分散します。
リスナーは、ALB がどのポートでリクエストを受け付けるかを定義します。
3.4 NLB（Network Load Balancer）とは？
NLB は、高パフォーマンスなアプリケーションに適したロードバランサーです。

NLB は、TCP/UDP トラフィックを、複数のサーバーに分散することができます。また、非常に低遅延で、高いスループットを提供します。

NLB の特徴:

TCP/UDP トラフィックのロードバランシング: TCP/UDP トラフィックを、複数のサーバーに分散することができます。
低遅延: NLB は、非常に低遅延で動作します。
高スループット: NLB は、高いスループットを提供します。
セキュリティ強化: NLB は、アプリケーションのセキュリティを強化できます。
NLB の構成図:


説明:

ユーザーは NLB にアクセスします。
NLB は、ターゲットグループにリクエストを転送します。
ターゲットグループは、複数のサーバー（EC2 インスタンスまたはコンテナ）にリクエストを分散します。
リスナーは、NLB がどのポートでリクエストを受け付けるかを定義します。
3.5 ターゲットグループとは？
ターゲットグループは、ロードバランサーがトラフィックを転送するサーバーの集合です。

ターゲットグループには、複数のサーバー（EC2 インスタンスまたはコンテナ）を追加したり、削除したりすることができます。

ターゲットグループのメリット:

柔軟な管理: サーバーの追加、削除、更新を簡単に管理できます。
健康状態のチェック: ターゲットグループは、サーバーの健康状態をチェックし、正常なサーバーにのみトラフィックを転送します。
パフォーマンス向上: ターゲットグループは、サーバーのパフォーマンスを監視し、パフォーマンスの低いサーバーにトラフィックを転送しないようにします。
3.6 リスナーとは？
リスナーは、ロードバランサーがどのポートでリクエストを受け付けるかを定義するものです。

リスナーは、プロトコル、ポート番号、セキュリティ設定などを設定できます。

リスナーの例:

プロトコル: HTTP、HTTPS、TCP、UDP
ポート番号: 80、443
セキュリティ設定: SSL/TLS 証明書
3.7 パスベースルーティングとは？
パスベースルーティングは、リクエストのパスに基づいて、異なるサーバーに転送する機能です。

パスベースルーティングを使用することで、同じロードバランサーを使用して、複数のアプリケーションをホストすることができます。

パスベースルーティングの例:

パス: /products
ターゲットサーバー: 商品情報サーバー
パスベースルーティングの構成図:


説明:

ユーザーは ALB にアクセスします。
ALB は、リスナーにリクエストを転送します。
リスナーは、パスベースルーティングルールを使用して、リクエストのパスに基づいて、ターゲットグループ 1 またはターゲットグループ 2 に転送します。
ターゲットグループ 1 は、EC2 インスタンス 1 にリクエストを転送します。
ターゲットグループ 2 は、EC2 インスタンス 2 にリクエストを転送します。
3.8 スティッキーセッションとは？
スティッキーセッションは、特定のユーザーを、常に同じサーバーに接続する機能です。

スティッキーセッションを使用することで、ユーザーが同じサーバーに接続されたまま、セッション情報を維持することができます。

スティッキーセッションのメリット:

セッション管理: ユーザーのセッション情報を維持できます。
パフォーマンス向上: 同じサーバーに接続されたまま、データのやり取りを行うことができるため、パフォーマンスを向上させることができます。
スティッキーセッションの構成図:


説明:

ユーザーは ALB にアクセスします。
ALB は、リスナーにリクエストを転送します。
リスナーは、スティッキーセッションルールを使用して、ユーザーを同じサーバーに接続します。
ユーザーは、同じサーバーに接続されたまま、セッション情報を維持できます。
3.9 HTTPS ダイレクトとは？
HTTPS ダイレクトは、ALB が HTTPS トラフィックを暗号化して転送する機能です。

HTTPS ダイレクトを使用することで、アプリケーションのセキュリティを強化することができます。

HTTPS ダイレクトのメリット:

セキュリティ強化: HTTPS トラフィックを暗号化することで、セキュリティを強化できます。
パフォーマンス向上: ALB は、HTTPS トラフィックを効率的に処理します。
HTTPS ダイレクトの構成図:


説明:

ユーザーは ALB にアクセスします。
ALB は、リスナーに HTTPS トラフィックを転送します。
リスナーは、HTTPS ダイレクトルールを使用して、HTTPS トラフィックを暗号化して、ターゲットグループに転送します。
ターゲットグループは、複数のサーバー（EC2 インスタンスまたはコンテナ）にリクエストを分散します。
3.10 まとめ
本章では、AWS のロードバランサーサービスについて学びました。 これらの知識を基に、アプリケーションの負荷分散を効果的に行い、安定したサービスを提供できるようになりましょう！

今後の学習:

AWS 上で実際にロードバランサーを構築してみましょう。
異なる種類のロードバランサーの使い分けを理解しましょう。
ロードバランサーのセキュリティ設定について学びましょう。