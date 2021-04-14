---
lab:
    title: 'Language Understanding クライアント アプリケーションの作成'
    module: 'モジュール 5 - Language Understanding ソリューションの作成'
---

# Language Understanding クライアント アプリケーションの作成

Language Understanding サービスを使用すると、アプリケーションがユーザーからの自然言語入力を解釈し、ユーザーの*意図* (達成したいこと) を予測し、意図を適用する必要がある*エンティティ*を特定するために使用できる言語モデルをカプセル化するアプリを定義できます。Language Understanding アプリを使用するクライアント アプリケーションは、REST インターフェイスを介して直接作成するか、言語固有のソフトウェア開発キット (SDK) を使用して作成できます。

## このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボで作業している環境に既に複製している場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐ複製してください。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git: Clone** コマンドを実行して、 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでもかまいません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**「今はしない」** を選択します。

## Language Understanding リソースの作成

Azure サブスクリプションに Language Understanding オーサリングおよび予測リソースが既にある場合は、この演習でそれらを使用できます。それ以外の場合は、次の手順に従って作成してください。

1. `https://portal.azure.com` で Azure portal を開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **&#65291;「リソースの作成」** ボタンを選択し、*Language Understanding* を検索して、次の設定で **Language Understanding** リソースを作成します。
    - **作成オプション**: 両方
    - **サブスクリプション**: *お使いの Azure サブスクリプション*
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がない可能性があります - 提供されているものを使用してください)*
    - **名前**: *一意の名前を入力します*
    - **オーサリングの場所**: *希望する場所を選択します*
    - **オーサリングの価格レベル**: F0
    - **予測の場所**: *オーサリング場所と<u>同じ場所</u>を選択します*
    - **予測の価格レベル**: F0 (*F0が利用できない場合はS0を選択します*)

3. リソースが作成されるのを待ち、2 つの Language Understanding リソースがプロビジョニングされることに注意してください。1 つはオーサリング用で、もう 1 つは予測用です。作成したリソース グループに移動すると、これらの両方を表示できます。

## Language Understanding アプリをインポート、トレーニング、公開する

前の演習の **Clock** アプリを既にお持ちの場合は、この演習で使用できます。それ以外の場合は、次の手順に従って作成してください。

1. 新しいブラウザタブで、`https://www.luis.ai` の Language Understanding ポータルを開きます。
2. Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。Language Understanding ポータルに初めてサインインする場合は、アカウントの詳細にアクセスするためのアクセス許可をアプリに付与する必要がある場合があります。次に、Azure サブスクリプションと作成したオーサリングリ ソースを選択して、*ようこそ*手順を完了します。
3. **「会話アプリ」** ページを開き、**「新しいアプリ」** の横にあるドロップダウン リストを表示して、**「LU としてインポート」** を選択します。
この演習のラボ ファイルを含むプロジェクト フォルダー内の **10-luis-client** サブフォルダーを参照し、**Clock&period;lu** を選択します。次に、時計アプリの一意の名前を指定します。
4. 効果的な Language Understanding アプリを作成するためのヒントが記載されたパネルが表示されたら、それを閉じます。
5. Language Understanding ポータルの上部にある **「トレーニング」** を選択して、アプリをトレーニングします。
6. Language Understanding ポータルの右上にある **「公開」 **を選択し、アプリを**本番スロット**に公開します。
7. 公開が完了したら、Language Understanding ポータルの上部にある **「管理」** を選択します。
8. **「設定」** ページで、**アプリ ID** をメモします。クライアント アプリケーションがアプリを使用するには、これが必要です。
9. **「Azure リソース」** ページの **「予測リソース」** で、予測リソースがリストされていない場合は、Azure サブスクリプションに予測リソースを追加します。
10. 予測リソースの**プライマリ キー**、**セカンダリ キー**、および**エンドポイント URL** に注意してください。クライアント アプリケーションは、予測リソースに接続して認証されるために、エンドポイントとキーの 1 つを必要とします。

## Language Understanding SDK を使用する準備をする

この演習では、クロック Language Understanding アプリを使用してユーザー入力から意図を予測し、適切に応答する、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の**エクスプローラー** ペインで、**10-luis-client** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **clock-client** フォルダーを右クリックして、統合ターミナルを開きます。次に、言語設定に適したコマンドを実行して、Language Understanding SDK パッケージをインストールします

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
```

***ランタイム** (予測) パッケージに加えて、Language Understanding モデルを作成および管理するためのコードを記述するために使用できる **オーサリング** パッケージがあります。*

**Python**

```
pip install azure-cognitiveservices-language-luis==0.7.0
```
    
*Python SDK パッケージには、**予測**と**オーサリング**の両方のクラスが含まれています。*

3. **clock-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください
    - **C#**: appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Language Understanding アプリの **アプリ ID**、**エンドポイント URL**、およびその予測リソースの**キー**の 1 つを含めます (Language Understanding ポータルのアプリの **「管理」** ページから)。

4. **clock-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#**: Program.cs
    - **Python**: clock-client&period;py

    コード ファイルを開き、上部の既存の名前空間参照の下で、**「名前空間のインポート」** というコメントを見つけます。次に、このコメントの下に、次の言語固有のコードを追加して、Language Understanding 予測 SDK を使用するために必要な名前空間をインポートします。

**C#**

```C#
// 名前空間をインポートする
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
```

**Python**

```Python
# 名前空間をインポートする
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
```

## Language Understanding アプリから予測を取得する

これで、SDK を使用して Language Understanding アプリから予測を取得するコードを実装する準備が整いました。

1. **Main** 関数では、構成ファイルからアプリ ID、予測エンドポイント、およびキーを読み込むためのコードが既に提供されていることに注意してください。次に、コメント **「Create a client for the LU app」** を見つけ、次のコードを追加して、Language Understanding アプリの予測クライアントを作成します。

**C#**

```C#
// LU アプリのクライアントを作成する
var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.ApiKeyServiceClientCredentials(predictionKey);
var luClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
```

**Python**

```Python
# LU アプリのクライアントを作成する
credentials = CognitiveServicesCredentials(lu_prediction_key)
lu_client = LUISRuntimeClient(lu_prediction_endpoint, credentials)
```

2. ユーザーが「quit」と入力するまで、**Main** 関数のコードはユーザー入力を求めるプロンプトを表示することに注意してください。このループ内で、コメント **「LU アプリを呼び出して意図とエンティティを取得する」** を見つけて、次のコードを追加します。

**C#**

```C#
// LU アプリを呼び出して意図とエンティティを取得する
var slot = "Production";
var request = new PredictionRequest { Query = userText };
PredictionResponse predictionResponse = await luClient.Prediction.GetSlotPredictionAsync(luAppId, slot, request);
Console.WriteLine(JsonConvert.SerializeObject(predictionResponse, Formatting.Indented));
Console.WriteLine("--------------------\n");
Console.WriteLine(predictionResponse.Query);
var topIntent = predictionResponse.Prediction.TopIntent;
var entities = predictionResponse.Prediction.Entities;
```

**Python**

```Python
# LU アプリを呼び出して意図とエンティティを取得する
request = { "query" : userText }
slot = 'Production'
prediction_response = lu_client.prediction.get_slot_prediction(lu_app_id, slot, request)
top_intent = prediction_response.prediction.top_intent
entities = prediction_response.prediction.entities
print('Top Intent: {}'.format(top_intent))
print('Entities: {}'.format (entities))
print('-----------------\n{}'.format(prediction_response.query))
```

Language Understanding アプリを呼び出すと、予測が返されます。これには、入力発話で検出されたエンティティだけでなく、最上位の (最も可能性の高い) 意図も含まれます。クライアント アプリケーションは、その予測を使用して適切なアクションを決定および実行する必要があります。

3. コメント **「適切なアクションを適用する」** を見つけ、次のコードを追加します。このコードは、アプリケーションでサポートされている意図 (**GetTime**、**GetDate**、および**GetDay**) をチェックします。また、適切な応答を生成するために既存の関数を呼び出す前に、関連するエンティティが検出されたかどうかを判断します。

**C#**

```C#
// 適切なアクションを適用する
switch (topIntent)
{
    case "GetTime":
        var location = "local";
        // エンティティを確認する
        if (entities.Count > 0)
        {
            // 場所のエンティティを確認する
            if (entities.ContainsKey("Location"))
            {
                //エンティティの JSON を取得する
                var entityJson = JArray.Parse(entities["Location"].ToString());
                // ML エンティティは文字列で、一番上の項目を取得する
                location = entityJson[0].ToString();
            }
        }

        // 指定した型のファセットを取得する
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;

    case "GetDay":
        var date = DateTime.Today.ToShortDateString();
        // エンティティを確認する
        if (entities.Count > 0)
        {
            // データ エンティティを確認する
            if (entities.ContainsKey("Date"))
            {
                //エンティティの JSON を取得する
                var entityJson = JArray.Parse(entities["Date"].ToString());
                // Regex  エンティティは文字列で、一番上の項目を取得する
                date = entityJson[0].ToString();
            }
        }
        // 指定した日付の「日」を取得する
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;

    case "GetDate":
        var day = DateTime.Today.DayOfWeek.ToString();
        // エンティティを確認する
        if (entities.Count > 0)
        {
            // Weekday エンティティを確認する
            if (entities.ContainsKey("Weekday"))
            {
                //エンティティの JSON を取得する
                var entityJson = JArray.Parse(entities["Weekday"].ToString());
                // エンティティ リストを列挙する
                day = entityJson[0][0].ToString();
            }
        }
        // 指定した日の日付を取得する
        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;

    default:
        // 他のインテント (例: "None") が予測されました
        Console.WriteLine("Try asking me for the time, the day, or the date.");
        break;
}
```

**Python**

```Python
# 適切なアクションを適用する
if top_intent == 'GetTime':
    location = 'local'
    # エンティティを確認する
    if len(entities) > 0:
        # 場所のエンティティを確認する
        if 'Location' in entities:
            # ML エンティティは文字列で、一番上の項目を取得する
            location = entities['Location'][0]
    # 指定した型のファセットを取得する
    print(GetTime(location))

elif top_intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # エンティティを確認する
    if len(entities) > 0:
        # データ エンティティを確認する
        if 'Date' in entities:
            # Regex  エンティティは文字列で、一番上の項目を取得する
            date_string = entities['Date'][0]
    # 指定した日付の「日」を取得する
    print(GetDay(date_string))

elif top_intent == 'GetDate':
    day = 'today'
    # エンティティを確認する
    if len(entities) > 0:
        # Weekday エンティティを確認する
        if 'Weekday' in entities:
            # エンティティ リストを列挙する
            day = entities['Weekday'][0][0]
    # 指定した日の日付を取得する
    print(GetDate(day))

else:
    # 他のインテント (例: "None") が予測されました
    print('Try asking me for the time, the day, or the date.')
```
    
4. 変更を保存し、**clock-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python clock-client.py
```

5. プロンプトが表示されたら、発話を入力してアプリケーションをテストします。たとえば、次のことを試してください。

    *Hello*
    
    *What time is it?*

    *What's the time in London?*

    *What's the date?*

    *What date is Sunday?*

    *What day is it?*

    *What day is 01/01/2025?*

> **注**: アプリケーションのロジックは意図的に単純であり、いくつかの制限があります。たとえば、時間を取得する場合、制限された都市のセットのみがサポートされ、夏時間は無視されます。目標は、アプリケーションが次のことを行う必要がある言語理解を使用するための一般的なパターンの例を確認することです。
>
>   1. 予測エンドポイントに接続します。
>   2. 予測を得るために発話を送信します。
>   3. 予測された意図とエンティティに適切に応答するロジックを実装します。

6. テストが終了したら、*quit* と入力します。

## 詳細情報

Language Understanding クライアントの作成の詳細については、[開発者向けドキュメント](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)を参照してください。
