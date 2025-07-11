---
title: AD FS の証明書更新手順（トークン署名証明書、トークン暗号化解除証明書）
date: 2017-10-03
tags:
  - AD FS
---

> [!NOTE]
> 本記事は Technet Blog の更新停止に伴い https://blogs.technet.microsoft.com/jpazureid/2017/10/03/ad-fs-tokencert/ の内容を移行したものです。
> 元の記事の最新の更新情報については、本内容をご参照ください。

> [!WARNING]
> 本記事のコマンド実行例では MSOnline PowerShell を利用しています。MSOnline PowerShell は 2025 年 4 月初旬から 5 月下旬の間に廃止され、使用できなくなります。
> 詳細は、[MSOnline および AzureAD PowerShell の廃止 - 2025 年版](../azure-active-directory/msonline-and-azuread-powershell-retirement.md) の記事をご確認ください。
> Microsoft Graph PowerShell を利用する手順は [こちら](../../active-directory-federation-service/migrate-msol-update-federation-domain.md)をご覧ください。

# AD FS の証明書更新手順（トークン署名証明書、トークン暗号化解除証明書）

皆様、こんにちは。Azure & Identity サポート担当の竹村です。

前回に引き続き、本エントリではトークン署名証明書、トークン暗号化解除証明書の更新手順をご紹介いたします。

トークン署名証明書、トークン暗号化解除証明書は AD FS が発行する自己証明書であり、既定では 1 年に 1 回自動的に新しい証明書の作成、更新処理が行われます。これを、[自動ロールオーバー機能](../../active-directory-federation-service/ad-fs-auto-rollover.md) と呼びます。

自動ロールオーバー機能は、具体的には以下のように動作します。

1. 有効期限が切れる 20 日前に、セカンダリのトークン署名/トークン暗号化解除証明書を作成します。
2. 上記 1 でセカンダリの証明書が作成されてから 5 日後に、セカンダリの新しい証明書がプライマリとなり、実際に利用されるようになります。

※ この 5 日間の間に、証明書利用者信頼に新しい証明書を登録する必要があります。

※ 2019/02/14 追記: 証明書利用者信頼にアップロードされるのはトークン署名証明書ですが、トークン暗号化解除証明書につきましても認証の際に利用されますので、有効期限が切れますと認証に失敗します。トークン署名証明書だけでなく、トークン暗号化解除証明書につきましても有効期限内での更新が必要です。

ここでは、これらの証明書を手動で更新する手順をご案内いたします。また、補足として証明書の有効期限を既定の 1 年 から変更する方法についても併せてご紹介いたします。

対象リリース: Windows Server 2012 R2 (AD FS 3.0) 以降

## 更新手順

以下の流れで更新作業を行います。

<手順の概要>

1. セカンダリのトークン署名/トークン暗号化解除証明書を作成
2. 証明書利用者信頼 (Office 365) の証明書情報を更新
3. セカンダリのトークン署名/トークン暗号化解除証明書をプライマリに昇格
4. Active Directory Federation Services サービスを再起動

それぞれの詳細を後述致します。

### 1. セカンダリのトークン署名/トークン暗号化解除証明書を作成

トークン署名/トークン暗号化解除証明書を更新するためには、現在使用されているプライマリのトークン署名/トークン暗号化解除証明書の有効期限が切れる前に、セカンダリのトークン署名/トークン暗号化解除証明書を作成致します。

このセカンダリのトークン署名/トークン暗号化解除証明書は、既定では、プライマリのトークン署名/トークン暗号化解除証明書の有効期限が切れる 20 日前に、AD FS によって自動で作成されます。

セカンダリのトークン署名/トークン暗号化解除証明書が作成されているか否かは、以下の手順で確認致します。

1. プライマリ AD FS サーバーでスタート画面などから [AD FS の管理] コンソールを開きます。
2. 画面左側から [AD FS] - [サービス] - [証明書] を選択します。
3. 画面中央から、トークン署名/トークン暗号化解除証明書のセカンダリが存在するか確認します。

    セカンダリのトークン署名/トークン暗号化解除証明書は、AD FS による自動作成を待たずに、手動で作成することも可能です。
    もし、セカンダリのトークン署名/トークン暗号化解除証明書が作成されていない際には、自動で作成されるのを待つか、以下の手順で手動で作成します。

4. プライマリ AD FS サーバーで PowerShell を管理者として起動します。

5. 起動した PowerShell で以下のコマンドを実行してセカンダリのトークン署名/トークン暗号化解除証明書を作成します。

    ```powershell
    Update-ADFSCertificate
    ```

    ※ このコマンドを実行するためには、[自動ロールオーバー機能](../../active-directory-federation-service/ad-fs-auto-rollover.md) が有効である必要がございます。無効の場合には、事前に以下のコマンドを実行し、有効にしてください。

    ```powershell
    Set-ADFSProperties -AutoCertificateRollover $true
    ```

6. 上記 1-1. から 1-3. の手順で、セカンダリが作成されていることを確認します。

以上でセカンダリのトークン署名/トークン暗号化解除証明書作成手順は終了です。
次のステップに進みます。

### 2. 証明書利用者信頼 (Office 365) の証明書情報を更新

新しいトークン署名証明書を作成いたしましたら、証明書利用者信頼にその新しい証明書を登録する必要があります。セカンダリのトークン署名/トークン暗号化解除証明書が作成されてから 5 日以内に、証明書利用者信頼に新しい証明書を登録する必要があります。これは、セカンダリのトークン署名/トークン暗号化解除証明書が作成されてから 5 日後にプライマリに昇格されて、実際に使用されるようになるためです。

ここでは、例として Office 365 の場合の手順をご案内します (Office 365 以外の場合には、実際に利用されている証明書利用者信頼ごとに手順をご確認ください)。

> [!NOTE]
> 2023 年 6 月 23 日更新: <br>
MS Online モジュールは、現時点で [2024 年 3 月 30 日に廃止予定](https://techcommunity.microsoft.com/t5/microsoft-entra-azure-ad-blog/important-azure-ad-graph-retirement-and-powershell-module/ba-p/3848270)です。
Microsoft Graph PowerShell を利用する手順は [こちら](../../active-directory-federation-service/migrate-msol-update-federation-domain.md)をご覧ください。


1. プライマリ AD FS サーバーでスタート画面などから [Windows PowerShell 用 Windows Azure Active Directory モジュール] を管理者として起動します。
2. 起動した PowerShell で以下のコマンドを実行し、Office 365 に接続します。

    ```powershell
    Connect-MsolService
    ```

    資格情報の入力を求められるので、管理者アカウントの情報を入力します。

3. 続けて、以下のコマンドを実行して現在の証明書情報を調べて、更新が必要か確認します。

    ```powershell
    Get-MsolFederationProperty -DomainName
    ```

    このコマンドで得られる結果から "Source : ADFS Server" と、"Source : Microsoft Office 365" のブロックを比較します。この 2 つのブロックの情報が異なる場合は、引き続き下記の手順に従って更新を行う必要があります。

4. 続けて、以下のコマンドを実行して Office 365 の証明書情報を更新します。

    ```powershell
    Update-MSOLFederatedDomain -DomainName
    ```

    ※ 複数のトップレベル ドメインをサポートしている場合は、代わりに以下コマンドを実行します。

    ```powershell
    Update-MSOLFederatedDomain -DomainName -SupportMultipleDomain
    ```

5. 上記 3. の手順で再度確認を行います。

以上で Office 365 の証明書情報更新手順は終了です。
次のステップに進みます。

### 3. セカンダリのトークン署名/トークン暗号化解除証明書をプライマリに昇格

※ この手順以降、手順 4. が完了するまで、正しく認証ができない場合があります。手順 4. まで続けて実行してください。

手順 1. で自動、または手動でセカンダリのトークン署名/トークン暗号化解除証明書が作成されてから 5 日後に、AD FS によって自動でセカンダリがプライマリに昇格されます。それ以降、AD FS では、この新しいプライマリのトークン署名/トークン暗号化解除証明書が使用されるようになります。そのため、上述の手順 2. は、セカンダリの証明書が作成されてから 5 日以内に行う必要があります。

この AD FS によるプライマリへの自動昇格を待たずに、手動で昇格することも可能です。その手順は、以下になります。

1. プライマリ AD FS サーバー で PowerShell を管理者として起動します。
2. 起動した PowerShell で以下のコマンドを実行して自動ロールオーバー機能を無効化します。

    ```powershell
    Set-ADFSProperties -AutoCertificateRollover $false
    ```

    ※ すでに無効化されている場合は、このコマンドの実行は不要です。

3. スタート画面などから [AD FS の管理] コンソールを開き、画面左側から [AD FS] - [サービス] - [証明書] を選択します。
4. 画面中央から、プライマリに昇格したいセカンダリのトークン署名/トークン暗号化解除証明書を右クリックし、[プライマリとして設定] を選択します。
5. 警告が表示される場合は [はい] をクリックします。
6. 続けて、PowerShell で以下のコマンドを実行して自動ロールオーバー機能を有効に戻します。

    ```powershell
    Set-ADFSProperties -AutoCertificateRollover $true
    ```

    ※ 無効化された状態で運用をされる場合は、このコマンドの実行は不要です。

    以下の手順でセカンダリのトークン署名/トークン暗号化解除証明書が自動、もしくは手動でプライマリに昇格されていることを確認致します。

7. プライマリ AD FS サーバーでスタート画面などから [AD FS の管理] コンソールを開きます。
8. 画面左側から [AD FS] - [サービス] - [証明書] を選択します。
9. 画面中央から、トークン署名/トークン暗号化解除証明書のプライマリとセカンダリが入れ替わっていることを確認します。

以上でセカンダリのトークン署名/トークン暗号化解除証明書をプライマリに昇格する手順は終了です。次のステップに進みます。

### 4. Active Directory Federation Services を再起動

手順 3. の後、トークン署名/トークン暗号化解除証明書の入れ替えが各 AD FS サーバーに反映されるまで、10 分ほど待った後、各 AD FS サーバーの Active Directory Federation Services を再起動致します。各 AD FS サーバーで以下の手順を行い、Active Directory Federation Services を再起動致します。

1. スタート画面などから [サービス] コンソールを開きます。
2. [Active Directory Federation Services] を右クリックし、[再起動] を選択します。

以上で Active Directory Federation Services の再起動手順は終了です。

## 補足

以下について補足をさせて頂きます。トークン署名/トークン暗号化解除証明書の更新に際しまして、ご参考になれば幸いです。

1. トークン署名/トークン暗号化解除証明書の有効期間を延ばす手順
2. 自動ロールオーバー機能を無効化する手順
3. セカンダリのトークン署名/トークン暗号化解除証明書を削除する手順

それぞれの詳細を後述致します。

### 1. トークン署名/トークン暗号化解除証明書の有効期間を変更する手順

既定では、トークン署名/トークン暗号化解除証明書の有効期間は 1 年です。以下の手順に沿って、トークン署名/トークン暗号化解除証明書の有効期間を変更することで、次回以降作成される証明書に反映されます。

なお、この手順は現行の証明書の期間を変更するものではなく、この設定以降に作成される証明書の有効期間を変更するものとなります。そのため、トークン署名/トークン暗号化解除証明書の有効期間を変更する際には、最初にこの手順を実施していただいた後に証明書の更新作業を行ってください。

1. プライマリ AD FS サーバー で PowerShell を管理者として起動します。
2. 起動した PowerShell で以下のコマンドを実行して有効期間を設定します。

    ```powershell
    Set-AdfsProperties -CertificateDuration
    ```

    例えば、およそ 10 年 (3650 日) に設定する場合は以下のようになります。

    ```powershell
    Set-AdfsProperties -CertificateDuration 3650
    ```

弊社内部情報より、最長で 50 年の設定を行っても問題ないことを確認しております。

### 2. 自動ロールオーバー機能を無効化する手順

先述のとおり、自動ロールオーバー機能は既定で以下のように動作します。

- 現在使用されているプライマリのトークン署名/トークン暗号化解除証明書の有効期限が切れる 20 日前に、新証明書をセカンダリとして作成する
- セカンダリのトークン署名/トークン暗号化解除証明書が作成されてから 5 日後に、プライマリ (旧証明書) とセカンダリ (新証明書) を入れ替える

そのため、各証明書の有効期限に合わせて (既定では 1 年ごとに) 新しい証明書を証明書利用者信頼に登録する作業を行っていただく必要がございます。

※ 証明書利用者信頼の中には、AD FS からフェデレーションメタデータを受信できるネットワーク環境であれば、自動的に更新作業が行われるものもございます。

上述 1 の証明書の有効期限を変更する手順と併せて、自動ロールオーバー機能を無効にしていただくことで、各作業を、各証明書の有効期限が切れるまでにお客様の任意のタイミングで進めていただくことができます。自動ロールオーバー機能を無効にする方法は以下になります。

1. プライマリ AD FS サーバー で PowerShell を管理者として起動します。
2. 起動した PowerShell で以下のコマンドを実行して自動ロールオーバー機能を無効化します。

    ```powershell
    Set-ADFSProperties -AutoCertificateRollover $false
    ```

    なお、自動ロールオーバー機能を有効に戻す場合は、以下のコマンドを実行します。

    ```powershell
    Set-ADFSProperties -AutoCertificateRollover $true
    ```

### 3. セカンダリのトークン署名/トークン暗号化解除証明書を削除する手順

トークン署名/トークン暗号化解除証明書の更新作業を行っていただいた後、元々プライマリであった証明書がセカンダリとして残ります。以下の手順に沿って、このセカンダリの証明書を削除することが可能です。

なお、このセカンダリの証明書は残っていても AD FS の動作などに影響を及ぼすものではございません。

1. プライマリ AD FS サーバー で PowerShell を管理者として起動します。
2. 起動した PowerShell で以下のコマンドを実行して現在の証明書情報を確認します。

    ```powershell
    Get-AdfsCertificate
    ```

    CertificateType の値が Token-Signing になっていて IsPrimary の値が False になっているブロックがセカンダリのトークン署名証明書の情報ブロックです (例)。

    ```txt
    Certificate : [Subject]
    CN=ADFS Signing - adfs.test.com
    [Issuer]
    CN=ADFS Signing - adfs.test.com
    [Serial Number]
    1E841605620028AC4FB542149E31A4A7
    [Not Before]
    2017/10/11 15:09:47
    [Not After]
    2057/10/01 15:09:47
    [Thumbprint]
    41D3FE9CB98FA89BE0899904B3B227DECB8A512A
    CertificateType : Token-Signing ★<<<
    IsPrimary : False ★<<<
    StoreLocation : CurrentUser
    StoreName : My
    Thumbprint : 41D3FE9CB98FA89BE0899904B3B227DECB8A512A ★<<<
    ```

    ※ 念のため、AD FS 管理ツールからセカンダリのトークン署名証明書をダブルクリックし、[詳細] タブの「拇印」が Thumbprint と一致していることを確認します。

    CertificateType の値が Token-Decrypting になっていて IsPrimary の値が False になっているブロックがセカンダリのトークン暗号化解除証明書の情報ブロックです (例)。

    ```txt
    Certificate : [Subject]
    CN=ADFS Encryption - adfs.test.com
    [Issuer]
    CN=ADFS Encryption - adfs.test.com
    [Serial Number]
    3E79CAA431F92DB6454084FEA1AE9995
    [Not Before]
    2017/10/11 15:09:47
    [Not After]
    2057/10/01 15:09:47
    [Thumbprint]
    57CA6410E8C213DC14645AEBCCDEF0481FC8C641
    CertificateType : Token-Decrypting ★<<<
    IsPrimary : False ★<<<
    StoreLocation : CurrentUser
    StoreName : My
    Thumbprint : 57CA6410E8C213DC14645AEBCCDEF0481FC8C641 ★<<<
    ```

    ※ 念のため、AD FS 管理ツールからセカンダリのトークン暗号化解除証明書をダブルクリックし、[詳細] タブの「拇印」が Thumbprint と一致していることを確認します。

3. 続けて、PowerShell で以下のコマンドを実行して自動ロールオーバー機能を無効化します。

    ```powershell
    Set-ADFSProperties -AutoCertificateRollover $false
    ```

    ※ すでに無効化されている場合はこのコマンドの実行は不要です。

4. 上記 2. で確認した結果を元に、PowerShell で以下のコマンドを実行してセカンダリのトークン署名証明書を削除します。

    ```powershell
    Remove-AdfsCertificate -Thumbprint 【削除する証明書の拇印】 -CertificateType Token-Signing
    ```

    上記 2 の例ですと、以下のコマンドを実行します。

    ```powershell
    Remove-AdfsCertificate -Thumbprint 41D3FE9CB98FA89BE0899904B3B227DECB8A512A -CertificateType Token-Signing
    ```

5. 上記 2. で確認した結果を元に、PowerShell で以下のコマンドを実行してセカンダリのトークン暗号化解除証明書を削除します。

    ```powershell
    Remove-AdfsCertificate -Thumbprint 【削除する証明書の拇印】 -CertificateType Token-Decrypting
    ```

    上記 2 の例ですと、以下のコマンドを実行します。

    ```powershell
    Remove-AdfsCertificate -Thumbprint 57CA6410E8C213DC14645AEBCCDEF0481FC8C641 -CertificateType Token-Decrypting
    ```

6. 続けて、PowerShell で以下のコマンドを実行して自動ロールオーバー機能を有効に戻します。

    ```powershell
    Set-ADFSProperties -AutoCertificateRollover $true
    ```

    ※ 無効化された状態で運用をされる場合はこのコマンドの実行は不要です。

手順は以上の通りとなります。
上記手順は過去にも同様のお問い合わせをお受けした際、多くのお客様にご案内しております実績のある手順となっておりますので、ご参考いただけますと幸いです。
