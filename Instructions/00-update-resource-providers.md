---
lab:
    title: 'リソース プロバイダーを有効にする'
    module: 'セットアップ'
---

# リソース プロバイダーを有効にする

Azure サブスクリプションに登録する必要のあるリソース プロバイダーがいくつかあります。次の手順に従って、登録されていることを確認してください。

1. ご利用の Azure サブスクリプションに関連付けられている Microsoft 資格情報を使用して、`https://portal.azure.com` で Azure portal にサインインします。
2. **「ホーム」** ページで、**「サブスクリプション」** を選択します (または **「&#8801;」** メニューを展開し、**「すべてのサービス」** を選択し、**すべての**カテゴリで、「**サブスクリプション**」を選択します)。
3. Azure サブスクリプションを選択します (複数のサブスクリプションがある場合は、Azure Pass を利用して作成したサブスクリプションを選択します)。
4. サブスクリプションのブレードの左側のウィンドウの **「設定」** セクションで、**「リソース プロバイダー」** を選択します。
5. リソース プロバイダーのリストで、次のプロバイダーが登録されていることを確認します (登録されていない場合は、それらを選択して 「**登録**」 をクリックします)。
    - Microsoft.BotService
    - Microsoft.Web
    - Microsoft.ManagedIdentity
    - Microsoft.Search
    - Microsoft.Storage
    - Microsoft.CognitiveServices
    - Microsoft.AlertsManagement
    - Microsoft.Insights
    - Microsoft.KeyVault
    - Microsoft.ContainerInstance
