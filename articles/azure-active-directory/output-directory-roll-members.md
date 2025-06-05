---
title: Azure AD の ディレクトリロールが割り当てられたメンバーの一覧を取得したい
date: 2018-09-13
tags:
  - Entra ID Directory Roles
---

> [!NOTE]
> 本記事は Technet Blog の更新停止に伴い https://blogs.technet.microsoft.com/jpazureid/2018/09/13/outputdirectoryrollmembers/ の内容を移行したものです。
> 元の記事の最新の更新情報については、本内容をご参照ください。

> [!NOTE]
>
> (2025/3/11 更新)
>
> 本記事のコマンド実行例では MSOnline PowerShell を利用しています。MSOnline PowerShell は 2025 年 4 月初旬から 5 月下旬の間に廃止され、使用できなくなります。
> 詳細は、[MSOnline および AzureAD PowerShell の廃止 - 2025 年版](../azure-active-directory/msonline-and-azuread-powershell-retirement.md) の記事をご確認ください。
> Entra ID のディレクトリ ロールが割り当てられているユーザーの一覧を取得したい場合、現在はコマンドを利用しなくても、 Azure ポータルの [Microsoft Entra ID] - [ロールと管理者] - [割り当てのダウンロード] ボタンを利用することで簡単に一覧をダウンロードすることが可能ですので、併せて利用をご検討ください。


# Azure AD の ディレクトリロールが割り当てられたメンバーの一覧を取得したい

こんにちは！ Azure & Identity サポート チームの三浦 大です。

今回は Azure AD のディレクトリロールが割り当てられているユーザーの一覧を CSV で出力する PowerShell スクリプトをご紹介します。Azure AD はセキュリティの観点から、ディレクトリロールにて設定できるロールが細かく分類されております。各ロールのメンバーの一覧は、先日の [ブログ](./roles-and-administrators.md) で紹介しましたように、 Azure ポータルから参照することができます。

しかし、この一覧をファイルとして出力したいというご要望もあるかと思います。以下のとおり、ディレクトリロールの一覧を出力する PowerShell スクリプトを作成いたしましたので、ぜひご利用ください！

> 実行に際しては、事前に Install-Module -Name MSOnline コマンドを実行し、 Azure AD PowerShell モジュールのインストールをお願いいたします。

Azure AD PowerShell のインストールなどについては、以下の公開情報をご参照くださいませ。

[Azure Active Directory の PowerShell モジュール](./powershell-module.md)

## PowerShell スクリプト

> [!WARNING]
> 以下の実行例は MSOnline PowerShell を利用しています。MSOnline PowerShell は 2025 年 4 月初旬から 5 月下旬の間に廃止され、使用できなくなります。

```powershell
Import-Module MSOnline
Connect-MsolService

#任意の csv が出力されるディレクトリを設定してください。

$outfile = "C:\output.csv"
$dataList
$data = New-Object PSObject | Select-Object DirectoryRole, User
$role = Get-MsolRole
$dataList = @()

foreach ($test in $role) {
    $member = Get-MsolRoleMember -RoleObjectId $test.objectid
    If ($null -eq $member) {
        $role = $test.name
        Write-Output "$role,"
        $data = New-Object PSObject | Select-Object DirectoryRole, User
        $data.DirectoryRole = $role
        $data.User = $null
        $dataList += $data
    } Else {
        foreach ($m in $member) {
            $role = $test.name
            $user = $m.EmailAddress
            Write-Output "$role,$user"
            $data = New-Object PSObject | Select-Object DirectoryRole, User
            $data.DirectoryRole = $role
            $data.User = $user
            $dataList += $data
        }
    }
}

$DataList | Export-Csv $outfile -Encoding Default
```

## 出力結果の例

スクリプトを実行すると資格情報が求められますので、出力対象の Azure AD テナントの全体管理者の資格情報を入力します。その後自動的に csv ファイルにロールの一覧が出力されます。以下は出力のサンプルです。

![](./output-directory-roll-members/output-directory-roll-member011.png)

本ブログにて、管理者の方のご負担、少しでも軽減されれば幸甚です。
