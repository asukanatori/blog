---
title: どこからでもどんなリソースへのアクセスも保護
date: 2024-06-20 10:00
tags:
  - Azure AD
  - US Identity Blog
---

# どこからでもどんなリソースへのアクセスも保護

こんにちは、Azure Identity サポート チームの 名取 です。

本記事は、2024 年 5 月 29 日に米国の Microsoft Entra Blog で公開された [Securing access to any resource, anywhere](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/securing-access-to-any-resource-anywhere/ba-p/4120308) を意訳したものになります。ご不明点等ございましたらサポート チームまでお問い合わせください。

----

ゼロ トラストは、デジタル資産全体を保護するための業界標準となっています。ゼロ トラストの中心となるのは ID とアクセスを安全に保つことであり、今日のダイナミックなデジタル環境においては、リソースの保護、セキュリティ ポリシーの強制、コンプライアンス確保が不可欠になります。

Microsoft Entra は、[信頼できる ID をあらゆる場所であらゆるものと安全につなぎ合わせるという相互信頼の基盤 (トラスト ファブリック) 構築](https://www.microsoft.com/en-us/security/blog/2024/05/08/how-implementing-a-trust-fabric-strengthens-identity-and-network/) を支援します。AI の時代においてマルチクラウド戦略が進んだことで、お客様は、複数のパブリック クラウドとプライベート クラウドの間だけでなく、ビジネス アプリケーションとオンプレミス リソースに対しても、いかに安全にアクセスを行うかという多くの課題に直面しています。単にユーザーを保護したり、単一の環境内でのアクセスを保護したりする場合 (すでに対処方法が確立している) とは異なり、今日の常に変化し続けるデジタル資産を考慮すると、どこにいてもアクセスを保護するということはより複雑なものとなります。新たな課題に対処するためには、さらなるツール開発も必要となります。そこで、そのようなお客様をサポートするため、弊社は [今年の RSA カンファレンスで、あらゆるクラウドにおけるアクセスのセキュリティを確保するためのビジョン](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/microsoft-entra-announcements-and-demos-at-rsac-2024/ba-p/2520429) を発表しました。本日は、多様なクラウド環境におけるあらゆる ID からクラウド リソースへのアクセスを保護することを目的とした、当社の今後の投資対象について詳しく紹介します。

## 急速に進化するデジタル環境において複雑なマルチクラウド構成を管理下に置く

多くの組織がクラウドにおけるアクセスの複雑さを克服するための課題に取り組んでいますが、この問題の例としては、ばらばらに管理されたロールベースのアクセス制御 (RBAC) システムや、社内ポリシーからの逸脱などが挙げられます。こうした課題は、さまざまなベンダーのクラウド サービスを利用する機会が増えることで、さらに深刻化しています。また、過剰にアクセス許可が付与された ID によって、重大な情報漏洩が度々発生しています。弊社がお客様やり取りしていると、多くの組織が現在、特権アクセス管理 (PAM) や ID ガバナンス/管理 (IGA) ソリューションなどを 7 ～ 8 製品ほど組み合わせてマルチクラウド アクセスの課題に取り組んでいるということが分かっています。複数のソリューションの組み合わせや人員増強など大変な取り組みを進めているにもかかわらず、現在も多くの企業がクラウドのアクセス全体を俯瞰できる形にするという点では苦戦しています。

[当社の 2024 年マルチクラウド セキュリティ リスク レポート](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/final/en-us/microsoft-brand/documents/2024-State-of-Multicloud-Security-Risk-Report.pdf) では、過剰にアクセス許可が付与されたユーザー ID およびワークロード ID から生じる継続的な課題を強調しています。[Microsoft Entra Permissions Management](https://www.microsoft.com/ja-jp/security/business/identity-access/microsoft-entra-permissions-management) の過去 1 年間の使用データを分析した結果、マルチクラウド環境における複雑さは、主に ID の急激な増加と過剰に付与された権限に起因することが確認されました。([他にも](https://aka.ms/multicloudinfographic)) 以下の点が挙げられます:

- ID に付与可能なアクセス許可は 51,000 以上あり、そのうち半分がリスクの高いアクセス許可である。
- 51,000 のアクセス許可のうち、使用されているのはわずか 2% である。
- 2 億 900 万件の ID のうち、50% 以上がすべてのリソースにアクセス可能な権限を持つ最高特権の ID であることが確認された。

![図 1: 2024 年マルチクラウド セキュリティ リスクの現状と主な調査結果](./securing-access-to-any-resource,-anywhere/1.png)

## クラウド アクセス管理に対する当社のビジョンの紹介: 統合プラットフォームの構築

クラウド アクセス マネジメントのビジョンについてお話ししたいと思います。

ビジネスが拡大するにつれて、組織は必然的にさまざまなレベルでアクセス許可の過剰付与という課題に直面します。最初は、これは拡大するチームとワークロードに対応するためにより多くのアクセス許可を付与することから始まり、次第に不要なアクセス許可が複数重複して付与されるということにつながっていきます。これらの問題に対処するには、組織が ID とアクセス許可における問題個所を積極的に見つけ、迅速に対応する必要があり、最終的には自動化するという対応も必要です。

この重要なニーズに対応するため、弊社は包括的な機能を網羅する統合プラットフォームを開発しています。 この次期プラットフォームは、逸脱が発生した際もクラウド リソースに安全にアクセスできるよう、リスクの発見から修復までの行程を効率的に行えるように設計されており、以下のような機能を提供します:

- **可視性**: 割り当ておよび使用されているすべての ID と権限を把握し、リスクのある権限を検出する。
- **リスクの修復**: 推奨事項を提示して、危険なアクセス許可を修復する。
- **細かな制御**: そのロールにあった期間だけ適切な権限を付与する。
- **自動化されたガバナンス**： 自動化されたポリシーを用いて継続的に準拠状態を維持する。

![図 2: 複数のクラウドにわたりあらゆる ID からのアクセスを保護する](./securing-access-to-any-resource,-anywhere/2.png)

業界をリードする Microsoft Entra 製品を基盤として、あらゆるクラウド内のリソースへ安全にアクセスできるようにする取り組みが進んでいます:

- **[Permissions Management](https://www.microsoft.com/ja-jp/security/business/identity-access/microsoft-entra-permissions-management) (CIEM)**: ID、アクセス許可、使用状況を可視化するのに利用
- **[Privileged Identity Management](https://www.microsoft.com/ja-jp/security/business/identity-access/microsoft-entra-id) (PAM)**: 人間の ID とワークロードの ID の両方に対して最小限の権限の制御を適用するのに利用
- **[ID ガバナンス](https://www.microsoft.com/ja-jp/security/business/identity-access/microsoft-entra-id-governance) (IGA)**: 所属や使用法に関係なく ID のライフサイクルとアクセス ワークフローを自動化するのに利用
- **[ワークロード ID](https://www.microsoft.com/ja-jp/security/business/identity-access/microsoft-entra-workload-id) (ワークロード用の ID とアクセス許可)**: ワークロード ID に対してカスタマイズした認証ポリシーを適用するのに利用

![図 3: 4 つの重要分野の融合](./securing-access-to-any-resource,-anywhere/3.png)

さらに、継続的な Copilot の取り組みの一環として、AI と機械学習を活用してクラウド上のアクセス管理プラットフォーム内のすべてのテクノロジーを強化しています。これにより企業は、手動で検出することが困難なリスクを発見することが可能となり、その中でも最も重大なリスクを特定することが可能となりました。また、それに対してより良い改善策を提案することも可能です。プラットフォームの採用が拡大するにつれて、あらゆる ID に対して実際の利用状況に基づくカスタム ロールとポリシーを提示できるようになります。これらにより、クラウド上でのアクセス管理が簡素化され、組織がクラウド環境をより効果的に保護できるようになります。

## 弊社のコミットメント

Microsoft は、新しい統合プラットフォームの進歩と革新を通じて、あらゆるクラウド内のリソースへのアクセスを保護するというビジョンを実現し、お客様に提供するため全力で取り組んでいます。さらに、このビジョンは、クラウドだけにとどまらず、オンプレミスの業務アプリケーションを含む、あらゆる場所にあるリソースへのアクセスも保護するものです。私たちは、AI およびワークロード ID の時代において、お客様があらゆるリソースへのあらゆるアクセスに対するセキュリティを強化できるよう支援してまいる所存です。

弊社は皆様と協力してこのビジョンを実現し、あらゆる企業がマルチクラウドおよびハイブリッド環境のすべての ID に対して最小特権のアクセスとアクセス許可を実装できるよう取り組んでまいります。このビジョンに向けた進捗状況については、随時お知らせしていきます。次回のお知らせまで、このビジョンの基盤となる当社の製品をぜひご検討ください。 詳細については、[Microsoft Entra ID ガバナンス](https://www.microsoft.com/ja-jp/security/business/identity-access/microsoft-entra-id-governance) と [権限管理](https://www.microsoft.com/ja-jp/security/business/identity-access/microsoft-entra-permissions-management) をご覧いただけますと幸いです。

Joseph Dadzie  
Partner Director of Product Management  
[LinkedIn](https://www.linkedin.com/in/joedadzie/)  
[Twitter](https://twitter.com/joe_dadzie)
