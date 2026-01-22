# SecurityCopilot-XDR-IncidentInvestigation
>このレポジトリは Defender XDR インシデント作成時に、Security Copilot を用いて調査するテンプレートを紹介しています

# 目的
> Security Copilot 活用推進: Defender XDR/Sentinel インシデント検知時のレポート生成

- SecurityCopilot-XDR-IncidentInvestigation
 ※ Defender XDR にて統合して検知したインシデント専用

- SecurityCopilot-Sentinel-IncidentInvestigation.json 
 ※ Microsoft Sentinel インシデント専用

# 必要となる権限
　ロジックアプリのリソース作成になるため、リソースグループの所有者権限で実施して下さい。
　ロジックアプリより、Security Copilot を利用可能なユーザーでの承認が必要になりますのでご用意ください。
　メール送信は Azure Eメール送信サービスを利用します。通信サービスの「接続文字列」をご用意ください。

# 作業手順
## 1. ロジックアプリテンプレートの導入(2つ作成します)
a) Azure ポータルより、「カスタムテンプレートのデプロイ」を選択し、ARMテンプレートを読み込ませてデプロイを行います。
　・「カスタムテンプレートのデプロイ」をクリック
　・「エディターで独自のテンプレートを作成する」をクリック
　・「ファイルの読み込み」より、テンプレート「SecurityCopilot-xxx-IncidentInvestigation」を選択
　・保存して次へ

b) カスタムデプロイの画面になりますので、以下選択
　・ロジックアプリを保存するサブスクリプション、リソースグループを選択して下さい。
　・既存のロジックアプリと分けた方がよければ、新規作成でリソースグループを作成して下さい。
　・リージョンがリソースグループのリージョンになっていることをご確認下さい。
　・Send To Email Address は、通知先の貴社SOCメールアドレスを指定して下さい。

c) 「次へ」に進んで、ロジックアプリを導入して下さい。


## 2. ロジックアプリ設定
### 2-1: Security Copilot API ユーザー承認
　ロジックアプリ「SecurityCopilot-xxx-Incident-Investigation」を開きます。
　「開発ツール」->「API接続」を開きます。
　API接続名「Securitycopilot-SecurityCopilot-xxx-Incident-Investigation」を開きます。
	「全般」->「API接続の編集」より、「承認」ボタンを押して、Security Copilot を利用するユーザー名で承認して下さい。
　完了後、保存をしてください。

### 2-2: ロジックアプリに対するAzureロールの割り当て（Sentinel Contributorロール）
　本ロジックアプリでは、インシデントのコメントに Security Copilot 解析結果をメモとして保存する設定を加えています。
　ロジックアプリがインシデント編集を行えるように、権限の設定を加えて下さい。
　・ロジックアプリより「ID」を選択
　・「Azureロールの割り当て」より、「ロールの割り当ての追加」から「Microsoft Sentinel レスポンダー」ロールを付与

### 2-3: Azure Eメール通信サービスの設定
　「開発ツール」->「API接続」を開きます。
　API接続名「Securitycopilot-SecurityCopilot-xxx-Incident-Investigation」を開きます。
　「接続文字列」の箇所に、貴社で構成された通信サービスの接続文字列をPasteしてください。

## 3. Sentinel 側設定
 ロジックアプリをトリガー起動させるため、オートメーション設定より以下を設定して下さい。
### 3.1 Sentinel 「設定」より、「プレイブックのアクセス許可」を選択
 新たにロジックアプリ用にリソースグループを構成した場合は、新規のリソースグループを許可
### 3.2 Sentinel 「オートメーション」設定
- Automation rule name : CfS-IncidentReport-MDE-High
　- Condition 条件で以下を設定
	Alert product names Contains "Microsoft Defender for Endpoint"
	Severity Equals "高"
　- Actions:Run Playbook
	SecurityCopilot-XDR-Incident-Investigation

- Automation rule name : CfS-IncidentReport-Sentinel-High
　- Condition 条件で以下を設定
	Alert product names Contains "Microsoft Sentinel"
	Severity Equals "高"
　- Actions:Run Playbook
	SecurityCopilot-Sentinel-Incident-Investigation

## 4. Security Copilot SCU 設定
 SCU リソースを使うワークフローになるため、SCU 1 -> 3 への更新をお願いいたします。
