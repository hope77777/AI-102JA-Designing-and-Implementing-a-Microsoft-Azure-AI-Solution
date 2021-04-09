---
lab:
    title: 'Speech および Language Understanding サービスの使用'
    module: 'モジュール 5 - Language Understanding ソリューションの作成'
---

# Speech および Language Understanding サービスの使用

音声サービスを Language Understanding サービスと統合して、音声入力からユーザーの意図をインテリジェントに判断できるアプリケーションを作成できます。

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

## Language Understanding アプリを準備する

前の演習の**時計**アプリを既にお持ちの場合は、`https://www.luis.ai` の Language Understanding ポータルで開きます。それ以外の場合は、次の手順に従って作成してください。

1. 新しいブラウザタブで、`https://www.luis.ai` の Language Understanding ポータルを開きます。
2. Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。Language Understanding ポータルに初めてサインインする場合は、アカウントの詳細にアクセスするためのアクセス許可をアプリに付与する必要がある場合があります。次に、Azure サブスクリプションと作成したオーサリングリ ソースを選択して、*ようこそ*手順を完了します。
3. **「会話アプリ」** ページを開き、**「新しいアプリ」** の横にあるドロップダウン リストを表示して、**「LU としてインポート」** を選択します。
この演習のラボ ファイルを含むプロジェクト フォルダー内の **11-luis-speech** サブフォルダーを参照し、**Clock&period;lu** を選択します。次に、時計アプリの一意の名前を指定します。
4. 効果的な Language Understanding アプリを作成するためのヒントが記載されたパネルが表示されたら、それを閉じます。

## *音声プライミング*でアプリをトレーニングして公開する

1. アプリがまだトレーニングされていない場合は、Language Understanding ポータルの上部にある **「トレーニング」** を選択して、アプリをトレーニングします。
2. Language Understanding ポータルの右上にある **「公開」** を選択します。次に、**「本番スロット」** を選択し、設定を変更して**音声プライミング**を有効にします (これにより、音声認識のパフォーマンスが向上します)。
3. 公開が完了したら、Language Understanding ポータルの上部にある **「管理」** を選択します。
4. **「設定」** ページで、**アプリ ID** をメモします。クライアント アプリケーションがアプリを使用するには、これが必要です。
5. **「Azure リソース」** ページの **「予測リソース」** で、予測リソースがリストされていない場合は、Azure サブスクリプションに予測リソースを追加します。
6. 予測リソースの**プライマリ キー**、**セカンダリ キー**、および**場所L** (エンドポイントでは<u>ありません</u>) に注意してください。Speech SDK クライアント アプリケーションは、予測リソースに接続して認証されるために、場所とキーの 1 つを必要とします。

## Language Understanding を備えた Speech SDK を使用する準備をする

この演習では、クロック Language Understanding アプリを使用して音声ユーザー入力から意図を予測する、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の**エクスプローラー** ペインで、**11-luis-speech** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **speaking-clock-client** フォルダーを右クリックして、統合ターミナルを開きます。次に、言語設定に適したコマンドを実行して、Language Understanding SDK パッケージをインストールします

**C#**

```
dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
```

**Python**

```
pip install azure-cognitiveservices-speech==1.14.0
```

3. **speaking-clock-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください。
    - **C#**: appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Language Understanding アプリの**アプリ ID** を含めます。**場所** (完全なエンドポイントでは<u>ありません</u> - たとえば、*eastus*) とその予測リソースの**キー**の 1 つ (Language Understanding ポータルのアプリの **「管理」** ページから)。

4. **speaking-clock-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#**: Program.cs
    - **Python**: speaking-clock-client&period;py

    コード ファイルを開き、上部の既存の名前空間参照の下で、**「名前空間のインポート」** というコメントを見つけます。次に、このコメントの下に、次の言語固有のコードを追加して、Speech SDK を使用するために必要な名前空間インポートします。

**C#**

```C#
// Import namespaces
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Intent;
```

**Python**

```Python
# Import namespaces
import azure.cognitiveservices.speech as speech_sdk
```
    
## 音声入力から予測意図を取得する

音声入力から予測意図を取得するコードを実装する準備が整いました。

1. **Main** 関数では、構成ファイルからアプリ ID、予測リージョン、およびキーを読み込むためのコードが既に提供されていることに注意してください。そして、コメント **「Configure speech service and get intent recognizer」** を見つけ、次のコードを追加して、Language Understanding 予測リソースの詳細を使用して、**Speech SDKSpeechConfig** と **IntentRecognizer** を作成します。

**C#**

```C#
// Configure speech service and get intent recognizer
SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
IntentRecognizer recognizer = new IntentRecognizer(speechConfig);
```

**Python**

```Python
# Configure speech service and get intent recognizer
speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
recognizer = speech_sdk.intent.IntentRecognizer(speech_config)
```
    
2. 追加したコードのすぐ下に、コメント **「Get the model from the AppID and add the intents we want to use」** を見つけ、使用する意図を追加し、次のコードを追加して、(アプリ ID に基づいて) Language Understanding モデルを取得し、認識機能に識別させたい意図を指定します。

**C#**

```C#
// Get the model from the AppID and add the intents we want to use
var model = LanguageUnderstandingModel.FromAppId(luAppId);
recognizer.AddIntent(model, "GetTime", "time");
recognizer.AddIntent(model, "GetDate", "date");
recognizer.AddIntent(model, "GetDay", "day");
recognizer.AddIntent(model, "None", "none");
```

*各インテントに文字列ベースのIDを指定できることに注意してくださいを*

**Python**

```Python
# Get the model from the AppID and add the intents we want to use
model = speech_sdk.intent.LanguageUnderstandingModel(app_id=lu_app_id)
intents = [
    (model, "GetTime"),
    (model, "GetDate"),
    (model, "GetDay"),
    (model, "None")
]
recognizer.add_intents(intents)
```

3. **Main** のコードは、ユーザーが「stop」と言うまで継続的にループすることに注意してください。このループ内で、コメント **「Process speech input」** を見つけ、次のコードを追加します。このコードは、レコグナイザーを使用して、音声入力を使用して Language Understanding サービスを非同期的に呼び出し、応答を取得します。応答に予測された意図が含まれている場合、音声クエリ、予測された意図、および完全な JSON 応答が表示されます。それ以外の場合、コードは返された理由に基づいて応答を処理します。
    
**C#**

```C
// Process speech input
var result = await recognizer.RecognizeOnceAsync().ConfigureAwait(false);
if (result.Reason == ResultReason.RecognizedIntent)
{
    // Intent was identified
    intent = result.IntentId;
    Console.WriteLine($"Query: {result.Text}");
    Console.WriteLine($"Intent Id: {intent}.");
    string jsonResponse = result.Properties.GetProperty(PropertyId.LanguageUnderstandingServiceResponse_JsonResult);
    Console.WriteLine($"JSON Response:\n{jsonResponse}\n");
    
    // Get the first entity (if any)

    // Apply the appropriate action
    
}
else if (result.Reason == ResultReason.RecognizedSpeech)
{
    // Speech was recognized, but no intent was identified.
    intent = result.Text;
    Console.Write($"I don't know what {intent} means.");
}
else if (result.Reason == ResultReason.NoMatch)
{
    // Speech wasn't recognized
    Console.WriteLine($"Sorry. I didn't understand that.");
}
else if (result.Reason == ResultReason.Canceled)
{
    // Something went wrong
    var cancellation = CancellationDetails.FromResult(result);
    Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");

    if (cancellation.Reason == CancellationReason.Error)
    {
        Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
        Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
    }
}
```

**Python**

```Python
# Process speech input
result = recognizer.recognize_once_async().get()
if result.reason == speech_sdk.ResultReason.RecognizedIntent:
    intent = result.intent_id
    print("Query: {}".format(result.text))
    print("Intent: {}".format(intent))
    json_response = json.loads(result.intent_json)
    print("JSON Response:\n{}\n".format(json.dumps(json_response, indent=2)))
    
    # Get the first entity (if any)
    
    # Apply the appropriate action
    
elif result.reason == speech_sdk.ResultReason.RecognizedSpeech:
    # Speech was recognized, but no intent was identified.
    intent = result.text
    print("I don't know what {} means.".format(intent))
elif result.reason == speech_sdk.ResultReason.NoMatch:
    # Speech wasn't recognized
    print("Sorry. I didn't understand that.")
elif result.reason == speech_sdk.ResultReason.Canceled:
    # Something went wrong
    print("Intent recognition canceled: {}".format(result.cancellation_details.reason))
    if result.cancellation_details.reason == speech_sdk.CancellationReason.Error:
        print("Error details: {}".format(result.cancellation_details.error_details))
```
    
これまでに追加したコードは*意図*を識別しますが、一部の意図は*エンティティ*を参照できるため、サービスから返される JSON からエンティティ情報を抽出するコードを追加する必要があります。

4. 追加したコードで、コメント **「Get the first entity (if any)」** を見つけ、その下に次のコードを追加します。

**C#**

```C
// Get the first entity (if any)
JObject jsonResults = JObject.Parse(jsonResponse);
string entityType = "";
string entityValue = "";
if (jsonResults["entities"].HasValues)
{
    JArray entities = new JArray(jsonResults["entities"][0]);
    entityType = entities[0]["type"].ToString();
    entityValue = entities[0]["entity"].ToString();
    Console.WriteLine（entityType + "： " + entityValue);
}
```

**Python**

```Python
# Get the first entity (if any)
entity_type = ''
entity_value = ''
if len(json_response["entities"]) > 0:
    entity_type = json_response["entities"][0]["type"]
    entity_value = json_response["entities"][0]["entity"]
    print(entity_type + ': ' + entity_value)
```
        
コードは、言語理解アプリを使用して、入力発話で検出されたエンティティだけでなく、インテントも予測するようになりました。クライアント アプリケーションは、その予測を使用して適切なアクションを決定および実行する必要があります。

5. 追加したコードの下に、 **「Apply the appropriate action」** というコメントを見つけ、次のコードを追加します。このコードは、アプリケーションでサポートされている意図 (**GetTime**、**GetDate**、および **GetDay**) をチェックし、適切な応答を生成するために既存の関数を呼び出す前に、関連するエンティティが検出されたかどうかを判断します。

**C#**

```C#
// Apply the appropriate action
switch (intent)
{
    case "time":
        var location = "local";
        // Check for entities
        if (entityType == "Location")
        {
            location = entityValue;
        }
        // Get the time for the specified location
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;
    case "day":
        var date = DateTime.Today.ToShortDateString();
        // Check for entities
        if (entityType == "Date")
        {
            date = entityValue;
        }
        // Get the day for the specified date
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;
    case "date":
        var day = DateTime.Today.DayOfWeek.ToString();
        // Check for entities
        if (entityType == "Weekday")
        {
            day = entityValue;
        }

        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;
    default:
        // Some other intent (for example, "None") was predicted
        Console.WriteLine("You said " + result.Text.ToLower());
        if (result.Text.ToLower().Replace(".", "") == "stop")
        {
            intent = result.Text;
        }
        else
        {
            Console.WriteLine("Try asking me for the time, the day, or the date.");
        }
        break;
}
```

**Python**

```Python
# Apply the appropriate action
if intent == 'GetTime':
    location = 'local'
    # Check for entities
    if entity_type == 'Location':
        location = entity_value
    # Get the time for the specified location
    print(GetTime(location))

elif intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if entity_type == 'Date':
        date_string = entity_value
    # Get the day for the specified date
    print(GetDay(date_string))

elif intent == 'GetDate':
    day = 'today'
    # Check for entities
    if entity_type == 'Weekday':
        # List entities are lists
        day = entity_value
    # Get the date for the specified day
    print(GetDate(day))

else:
    # Some other intent (for example, "None") was predicted
    print('You said {}'.format(result.text))
    if result.text.lower().replace('.', '') == 'stop':
        intent = result.text
    else:
        print('Try asking me for the time, the day, or the date.')
```

6. 変更を保存し、**speaking-clock-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python speaking-clock-client.py
```

7. プロンプトが表示されたら、アプリケーションをテストするために声を出して発言を話します。たとえば、次のことを試してください。

    *What's the time?*
    
    *What time is it?*

    *What day is it?*

    *What is the time in London?*

    *What's the date?*

    *What date is Sunday?*

    > **注**: アプリケーションのロジックは意図的に単純であり、いくつかの制限がありますが、Speech SDK を使用して音声入力から意図を予測する言語理解モデルの機能をテストする目的に役立つはずです。*MM/DD/YYYY* 形式で日付を言語化するのが難しいため、特定の日付エンティティで **GetDay** 意図を認識できない場合があります。

8. テストが終了したら、「stop」と言います。

## 詳細情報

音声と Language Understanding の統合の詳細については、[音声のドキュメント](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/intent-recognition)を参照してください。
