---
lab:
    title: 'Bot Framework SDK を使用したボットの作成'
    module: 'モジュール 7 - 会話型 AI と the Azure Bot Service'
---

# Bot Framework SDK を使用したボットの作成

*ボット*は、人間のユーザーとの会話型ダイアログに参加できるソフトウェア エージェントです。Microsoft Bot Framework は、Azure Bot Service を介してクラウド サービスとして提供できるボットを構築するための包括的なプラットフォームを提供します。

この演習では、Microsoft Bot Framework SDK を使用して、ボットを作成およびデプロイします。

## このコースのリポジトリを複製する

**AI-102-AIEngineer** コードのリポジトリをこのラボで作業している環境にまだ複製していない場合は、次の手順に従って複製してください。それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git: Clone** コマンドを実行して、 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでもかまいません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**「今はしない」** を選択します。

## ボットの作成

Bot Framework SDK を使用して、テンプレートに基づいてボットを作成し、特定の要件を満たすようにコードをカスタマイズできます。

> **注**: この演習では、**C#** または **Python** のいずれかから REST API を使用することを選択できます。以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の**エクスプローラー** ペインで、**13-bot-framework** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. 選択した言語のフォルダーを右クリックして、統合ターミナルを開きます。
3. ターミナルで、次のコマンドを実行して、必要なボット テンプレートとパッケージをインストールします

**C#**

```
dotnet new -i Microsoft.Bot.Framework.CSharp.EchoBot
dotnet new -i Microsoft.Bot.Framework.CSharp.CoreBot
dotnet new -i Microsoft.Bot.Framework.CSharp.EmptyBot
```

**Python**

```
pip install botbuilder-core
pip install asyncio
pip install aiohttp
pip install cookiecutter==1.7.0
```

4. テンプレートとパッケージをインストールしたら、次のコマンドを実行して、*EchoBot* テンプレートに基づいてボットを作成します。

**C#**

```
dotnet new echobot -n TimeBot
```

**Python**

```
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

Python を使用している場合、cookiecutter のプロンプトが表示されたら、次の詳細を入力します
- **bot_name**: TimeBot
- **bot_description**: A bot for our times
    
5. ターミナル ペインで、次のコマンドを入力して、現在のディレクトリを **TimeBot** フォルダーに変更し、ボット用に生成されたコード ファイルを一覧表示します。

    ```
    cd TimeBot
    dir
    ```

## Bot Framework Emulator でボットをテストします

*EchoBot* テンプレートに基づいてボットを作成しました。これで、ローカルで実行し、Bot Framework Emulator (システムにインストールする必要があります) を使用してテストできます。

1. ターミナル ペインで、現在のディレクトリがボット コード ファイルを含む **TimeBot** フォルダーであることを確認してから、次のコマンドを入力してボットをローカルで実行します。

**C#**

```
dotnet run
```

**Python**

```
python app.py
```
    
ボットが起動したら、ボットが実行されているエンドポイントが表示されていることに注意してください。これは**http://localhost:3978** のようになります。

2. Bot Framework Emulator を起動し、次のように **/api/messages** パスを追加してエンドポイントを指定してボットを開きます。

    `http://localhost:3978/api/messages`

3. 会話が**ライブ チャット**ペインで開かれた後、「*Hello and welcome!*」というメッセージを待ちます。
4. 「*Hello*」などのメッセージを入力し、ボットからの応答を表示します。これにより、入力したメッセージがエコー バックされます。
5. Bot Framework Emulator を閉じて VisualStudio Code に戻り、ターミナル ウィンドウで **CTRL+C** を入力してボットを停止します。

## ボット コードの変更

ユーザーの入力をエコー バックするボットを作成しました。これは特に有用ではありませんが、会話ダイアログの基本的な流れを説明するのに役立ちます。ボットとの会話は、テキスト、グラフィック、またはユーザーインターフェイス *カード*を使用して情報を交換する一連の*アクティビティ*で構成されます。ボットは挨拶で会話を開始します。これは、ユーザーがボットとのチャットセッションを初期化したときにトリガーされる*会話更新*アクティビティの結果です。次に、会話は、ユーザーとボットが順番に*メッセージ*を送信する一連のアクティビティで構成されます。

1. Visual Studio Code で、ボットの次のコード ファイルを開きます
    - **C#**: TimeBot/Bots/EchoBot.cs
    - **Python**: TimeBot/bot.py

    このファイルのコードは*アクティビティ ハンドラー*関数で構成されていることに注意してください。1 つは*メンバーが追加した*会話更新アクティビティ (誰かがチャット セッションに参加したとき) 用で、もう 1 つは*メッセージ*アクティビティ (メッセージが受信されたとき) 用です。会話は*ターン*の概念に基づいており、各ターンは、ボットがアクティビティを受信、処理、および応答するインタラクションを表します。*ターン コンテキスト*は、現在のターンで処理されているアクティビティに関する情報を追跡するために使用されます。

2. コード ファイルの先頭に、次の名前空間インポート ステートメントを追加します。

**C#**

```C#
using System;
```

**Python**

```Python
from datetime import datetime
```

3. *メッセージ* アクティビティのアクティビティ ハンドラー関数を次のコードに一致するように変更します。

**C#**

```C#
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    string inputMessage = turnContext.Activity.Text;
    string responseMessage = "Ask me what the time is.";
    if (inputMessage.ToLower().StartsWith("what") && inputMessage.ToLower().Contains("time"))
    {
        var now = DateTime.Now;
        responseMessage = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");
    }
    await turnContext.SendActivityAsync(MessageFactory.Text(responseMessage, responseMessage), cancellationToken);
}
```

**Python**

```Python
async def on_message_activity(self, turn_context: TurnContext):
    input_message = turn_context.activity.text
    response_message = 'Ask me what the time is.'
    if (input_message.lower().startswith('what') and 'time' in input_message.lower()):
        now = datetime.now()
        response_message = 'The time is {}:{:02d}.'.format(now.hour,now.minute)
    await turn_context.send_activity(response_message)
```
    
4. 変更を保存し、ターミナル ペインで、現在のディレクトリがボット コード ファイルを含む **TimeBot** フォルダーであることを確認してから、次のコマンドを入力して、ボットをローカルで実行します。

**C#**

```
dotnet run
```

**Python**

```
python app.py
```

前のように、ボットを開始するには、それが動作しているエンドポイントが示されていることに注意してください。

5. Bot Framework Emulator を起動し、次のように **/api/messages** パスを追加してエンドポイントを指定してボットを開きます。

    `http://localhost:3978/api/messages`

6. 会話が**ライブ チャット**ペインで開かれた後、「*Hello and welcome!*」というメッセージを待ちます。
7. 「*Hello*」などのメッセージを入力し、ボットからの応答を表示します。これは、*何時かを尋ねる必要があります*。
8. 「*What is the time?*」と入力し、応答を表示します。

    ボットは、「What is the time?」というクエリに応答するようになりました。ボットが実行されている現地時間を表示します。その他のクエリの場合は、ユーザーに時刻を尋ねるように求めます。これは非常に限定されたボットであり、言語理解サービスと追加のカスタムコードとの統合によって改善される可能性がありますが、テンプレートから作成されたボットを拡張することで Bot Framework SDK を使用してソリューションを構築する方法の実用的な例として機能します。

9. Bot Framework Emulator を閉じて VisualStudio Code に戻り、ターミナル ウィンドウで **CTRL+C** を入力してボットを停止します。

## 時間が許せば：ボットを Azure にデプロイする

これで、ボットを Azure にデプロイする準備が整いました。デプロイには、デプロイ用のコードを準備し、必要な Azure リソースを作成するための複数の手順が含まれます。

### リソース グループの作成または選択

ボットは、単一のリソース グループで作成できる複数の Azure リソースに依存しています。

1. `https://portal.azure.com` で Azure portal を開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **「リソース グループ」** ページを表示して、サブスクリプションに存在するリソース グループを確認します。
3. **「追加」** ボタンを使用して、使用可能な任意のリージョンに一意の名前で新しいリソース グループを作成します。(既存のリソース グループに制限する「サンドボックス」サブスクリプションを使用している場合は、リソース グループ名をメモしてください)。

### Azure アプリケーション登録を作成する

ボットがユーザーや Web サービスと通信できるようにするには、アプリケーション登録が必要です。

1. **TimeBot**フォルダーのターミナル ウィンドウで、次のコマンドを入力して、Azure コマンド ライン インターフェイス (CLI) を使用して Azure にログインします。ブラウザーが開いたら、Azure サブスクリプションにサインインします。

```
az login
```

2. 複数の Azure サブスクリプションがある場合は、次のコマンドを入力して、ボットをデプロイするサブスクリプションを選択します。

```
az account set --subscription "<YOUR_SUBSCRIPTION_ID>"
```

3. 次のコマンドを入力して、パスワード **Super$ecretPassw0rd** を使用して **TimeBot** のアプリケーション登録を作成します (必要に応じて別の表示名とパスワードを使用できますが、それらは後で必要になります)。

```
az ad app create --display-name "TimeBot" --password "Super$ecretPassw0rd" --available-to-other-tenants
```

4. コマンドが完了すると、大きな JSON 応答が表示されます。この応答で、**appId** 値を見つけてメモします。これは、次の手順で必要になります。

### Azure リソースを作成する

SDKを使用してテンプレートからボットを作成すると、必要な Azure リソースの作成に必要な Azure Resource Manager テンプレートが提供されます。

1. **TimeBot** フォルダーのターミナル ペインで、次のコマンドを (1 行で) 入力し、PLACEHOLDER 値を次のように置き換えます
    - **YOUR_RESOURCE_GROUP**: 既存のリソース グループの名前。
    - **YOUR_APP_ID**: 前の手順でメモした **appId** 値。
    - **REGION**: Azure リージョン コード (*eastus* など)。
    - **他のすべてのプレースホルダー**: 新しいリソースに名前を付けるために使用される一意の値。指定するリソース ID は、4 ? 42 文字の長さのグローバルに一意の文字列である必要があります。**BotId** パラメーターと **newWebAppName** パラメーターに使用する値をメモします。後で必要になります。

```
az deployment group create --resource-group "YOUR_RESOURCE_GROUP" --template-file "deploymenttemplates/template-with-preexisting-rg.json" --parameters appId="YOUR_APP_ID" appSecret="Super$ecretPassw0rd" botId="A_UNIQUE_BOT_ID" newWebAppName="A_UNIQUE_WEB_APP_NAME" newAppServicePlanName="A_UNIQUE_PLAN_NAME" appServicePlanLocation="REGION" --name "A_UNIQUE_SERVICE_NAME"
```

2. コマンドが完了するのを待ちます。成功すると、JSON 応答が表示されます。

    エラーが発生した場合は、コマンドのタイプミスまたは既存のリソースとの一意の名前の競合が原因である可能性があります。問題を修正して、再試行してください。障害が発生する前に作成されたリソースを削除するには、Azure portal を使用する必要がある場合があります。

3. コマンドが完了したら、Azure portal でリソース グループを表示して、作成されたリソースを確認します。

### デプロイ用のボット コードの準備

必要な Azure リソースが用意できたので、それらにデプロイするためのコードを準備できます。

1. Visual Studio Code の **TimeBot** フォルダーのターミナル ペインで、次のコマンドを入力して、コードの依存関係をデプロイ用に準備します。

**C#**

```
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "TimeBot.csproj"
```

**Python**

```
rmdir /S /Q  __pycache__
notepad requirements.txt
```

- 2 番目のコマンドは、メモ帳で Python 環境のため requirements.txt ファイルが開きます。修正を以下に一致するように変更を保存し、メモ帳を閉じます。

```
botbuilder-core==4.11.0
aiohttp
```

### デプロイ用 zip アーカイブを作成する

ボット ファイルをデプロイするには、それらを.zip アーカイブにパッケージ化します。これは、ボットのルート フォルダー内のファイルとフォルダーから作成する必要があります (ルート フォルダー自体を圧縮<u>しない</u>でください。その内容を圧縮してください)。

1. Visual Studio Code の**エクスプローラー** ペインで、**TimeBot** フォルダー内のファイルまたはフォルダーのいずれかを右クリックし、**「ファイル エクスプローラーで表示」**を選択します。
2. 「ファイルエクスプローラー」 ウィンドウで、**TimeBot** フォルダー内の<u>すべての</u>ファイルを選択します。次に、選択したファイルのいずれかを右クリックして、 > **「圧縮 (zip 形式)フォルダー**に**送信」**を選択します。
3. **TimeBot** フォルダー内の結果の zip ファイルの名前を **TimeBot.zip** に変更します。

### ボットのデプロイとテスト

コードの準備ができたので、デプロイできます。

1. Visual Studio Code の **TimeBot** フォルダーのターミナル ペインで、次のコマンドを (1 行で) 入力して、パッケージ化されたコードファイルを展開し、PLACEHOLDER 値を次のように置き換えます。
    - **YOUR_RESOURCE_GROUP**: 既存のリソース グループの名前。
    - **YOUR_WEB_APP_NAME**: Azure リソースの作成時に **newWebAppName** パラメーターに指定した一意の名前。

```
az webapp deployment source config-zip --resource-group "YOUR_RESOURCE_GROUP" --name "YOUR_WEB_APP_NAME" --src "TimeBot.zip"
```

2. Azure portal で、リソースを含むリソース グループで、**ボット チャネル登録リソース** (Azure リソースの作成時に **BotId** パラメーターに割り当てた名前が付けられます)。
3. **「ボット管理」**セクションで、**「Web チャットでテスト」** を選択します。次に、ボットが初期化されるのを待ちます。
4. 「*Hello*」などのメッセージを入力し、ボットからの応答を表示します。これは、*何時かを尋ねる必要があります*。
5. 「*What is the time?*」と入力し、応答を表示します。

## Web ページで Web チャット チャネルを使用する

Azure Bot Service の主な利点の 1 つは、複数の*チャネル*を介してボットを配信できることです。

1. Azure portal で、以前にボットをテストしていたページで、**「チャネル」** を選択します。
2. **Web チャット** チャネルが自動的に追加され、一般的な通信プラットフォーム用の他のチャネルが利用可能であることに注意してください。
3. **Webチャット** チャネルの横にある **「編集」** をクリックします。これにより、ボットを Web ページに埋め込むために必要な設定を含むページが開きます。ボットを埋め込むには、提供されている HTML 埋め込みコードと、ボット用に生成された秘密鍵の 1 つが必要です。
4. **埋め込みコード**をコピーします。
5. Visual Studio Code で、**13-bot-framework/web-client** フォルダーを展開し、そこに含まれる **default.html** ファイルを選択します。
6. HTML コードで、コメントのすぐ下にコピーした埋め込みコードを貼り付け、**ここにボットのiframeを追加します**
7. Azure portalに戻り、秘密鍵の 1 つに **「表示」** を選択して (どちらでも構いません)、コピーします。次に、Visual Studio Code に戻り、前に追加したHTML埋め込みコードに貼り付けて、**YOUR_SECRET_HERE** を置き換えます。
8. Visual Studio Code の**エクスプローラー** ペインで、**default.html** を右クリックし、**「ファイル エクスプローラーで表示」** を選択します。
9. Microsoft Edge の 「ファイル エクスプローラー」 ウィンドウで、**default.html** を開きます。
10. 開いた Web ページで、「*Hello*」と入力してボットをテストします。メッセージを送信するまで初期化されないため、グリーティング メッセージの直後に、時刻を尋ねるプロンプトが表示されることに注意してください。
11. 「*What is the time?*」を送信してボットをテストします。

## 詳細情報

Bot Framework の詳細については、[Bot Framework のドキュメント](https://docs.microsoft.com/azure/bot-service/index-bf-sdk)を参照してください。
