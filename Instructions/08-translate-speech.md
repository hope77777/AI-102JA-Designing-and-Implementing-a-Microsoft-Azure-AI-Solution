---
lab:
    title: '音声の翻訳'
    module: 'モジュール4 - 音声対応アプリケーションの構築'
---

# 音声の翻訳

**Speech** サービスには、話し言葉の翻訳に使用できる**音声翻訳** API が含まれています。たとえば、現地の言語を話さない場所を旅行するときに使用できる翻訳アプリケーションを開発するとします。「駅はどこ？」または「薬局を探す必要があります」などを自国語で話し、現地語に翻訳してもらいます。

> **注**: この演習では、マイクとスピーカー/ヘッドフォンを備えたコンピューターを使用している必要があります。

## このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボで作業している環境に既に複製している場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐ複製してください。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git: Clone** コマンドを実行して、 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでもかまいません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**「今はしない」** を選択します。

## Cognitive Services リソースをプロビジョニングする

サブスクリプションにまだない場合は、**Cognitive Services** リソースをプロビジョニングする必要があります。

1. `https://portal.azure.com` で Azure portal を開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **&#65291;「リソースの作成」** ボタンを選択し、*Cognitive Services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **サブスクリプション**: *お使いの Azure サブスクリプション*
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がない可能性があります - 提供されているものを使用してください)*
    - **リージョン**: *利用可能な任意のリージョンを選択します*
    - **名前**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェックボックスを選択して、リソースを作成します。
4. デプロイが完了するのを待ってから、デプロイの詳細を表示します。
5. リソースがデプロイされたら、リソースに移動して、その**キーとエンドポイント**のページを表示します。次の手順では、このページからサービスがプロビジョニングされるキーと場所の 1 つが必要になります。

## 音声翻訳サービスを使用する準備をする

この演習では、Speech SDK を使用して音声を認識、翻訳、および合成する、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の**エクスプローラー** ペインで、**08-speech** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **translator** フォルダーを右クリックして、統合ターミナルを開きます。次に、言語設定に適したコマンドを実行して、Speech SDK パッケージをインストールします。

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

3. **translator** フォルダーの内容を表示します。また、構成設定用のファイルが含まれていることに注意してください。
    - **C#**: appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの認証**キー**と、それが展開されている**場所**を含めます。変更を保存します。
4. **translator** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#**: Program.cs
    - **Python**: translator&period;py

    コード ファイルを開き、上部の既存の名前空間参照の下で、 **「名前空間のインポート」** というコメントを見つけます。次に、このコメントの下に、次の言語固有のコードを追加して、Speech SDK を使用するために必要な名前空間インポートします。

    **C#**
    
    ```C#
    // 名前空間をインポートする
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```
    
    **Python**
    
    ```Python
    # 名前空間をインポートする
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. **Main** 関数では、構成ファイルから Cognitive Services のキーとリージョンをロードするコードがすでに提供されていることに注意してください。これらの変数を使用して、音声入力の翻訳に使用する Cognitive Services リソースの **SpeechTranslationConfig** を作成する必要があります。コメント **「翻訳を構成する」** の下に次のコードを追加します。

    **C#**
    
    ```C#
    // 翻訳を構成する
    translationConfig = SpeechTranslationConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "ja-jp";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```
    
    **Python**
    
    ```Python
    # 翻訳を構成する
    translation_config = speech_sdk.translation.SpeechTranslationConfig(cog_key, cog_region)
    translation_config.speech_recognition_language = 'ja-jp'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

6. **SpeechTranslationConfig** を使用して音声をテキストに翻訳しますが、**SpeechConfig** を使用して翻訳を音声に合成します。コメント **「音声を認識する」** の下に次のコードを追加します。

    **C#**
    
    ```C#
    // 音声を認識する
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    ```
    
    **Python**
    
    ```Python
    # 音声を認識する
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    ```

7. 変更を保存して、**translator** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

8. C# を使用している場合は、非同期メソッドで **await** 演算子を使用することに関する警告を無視できます。これは後で修正します。コードは、ja-jp から変換する準備ができているというメッセージを表示する必要があります。ENTER を押してプログラムを終了します。

## 音声翻訳の実装

認知サービス リソースに音声サービス用の **SpeechTranslationConfig** が用意されたので、**Speech translation** APIを使用して音声を認識および翻訳できます。

1. プログラムの **Main** 関数で、コードが **Translate** 関数を使用して音声入力を翻訳していることに注意してください。
2. **Translate** 関数のコメント **「音声を翻訳する」** の下に、次のコードを追加して、入力にデフォルトのシステムマイクを使用して音声を認識および翻訳するために使用できる **TranslationRecognizer** クライアントを作成します。

    > **注**: *ファイル パスを参照するように **AudioConfig** オブジェクトを変更することにより、オーディオ ファイルからの音声入力を翻訳することもできます。*

    **C#**
    
    ```C#
    // 音声を翻訳する
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # 音声を翻訳する
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **注**: アプリケーションのコードは、1 回の呼び出しで入力を 3 つの言語すべてに翻訳します。特定の言語の翻訳のみが表示されますが、結果の **translations** コレクションでターゲット言語コードを指定することにより、任意の翻訳を取得できます。

3. 変更を保存して、**translator** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

4. プロンプトが表示されたら、有効な言語コード (*fr*、*es*、または*hi*) を入力してから、マイクに向かってはっきりと話し、「where is the station?」と言います。または海外旅行の際に使用する可能性のあるその他のフレーズを言います。プログラムは、話された入力を書き起こし、指定した言語 (フランス語、スペイン語、またはヒンディー語) に翻訳します。このプロセスを繰り返し、アプリケーションでサポートされている各言語を試してください。終了したら、Enter キーを押してプログラムを終了します。

    > **注**: TranslationRecognizer を使用すると、約 5 秒で話すことができます。音声入力が検出されない場合は、「一致なし」の結果が生成されます。
    >
    > 文字エンコードの問題により、ヒンディー語への翻訳がコンソール ウィンドウに正しく表示されない場合があります。

## 翻訳を音声に合成する

これまでのところ、アプリケーションは音声入力をテキストに翻訳します。旅行中に誰かに助けを求める必要がある場合は、これで十分かもしれません。ただし、適切な声で翻訳を声に出して話してもらう方がよいでしょう。

1. **Translate** 機能では、コメント **「翻訳を合成する」** の下に次のコードを追加して、デフォルトのスピーカーから音声として翻訳を合成するために **SpeechSynthesizer** クライアントを使用します。

    **C#**
    
    ```C#
    // 翻訳を合成する
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-Julie",
                        ["es"] = "es-ES-Laura",
                        ["hi"] = "hi-IN-Kalpana"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # 翻訳を合成する
    voices = {
            "fr": "fr-FR-Julie",
            "es": "es-ES-Laura",
            "hi": "hi-IN-Kalpana"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. 変更を保存して、**translator** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

3. プロンプトが表示されたら、有効な言語コード (*fr*、*es*、または*hi*) を入力し、マイクに向かってはっきりと話し、海外旅行で使用する可能性のあるフレーズを話します。プログラムは、口頭入力を書き起こし、口頭翻訳で応答します。このプロセスを繰り返し、アプリケーションでサポートされている各言語を試してください。終了したら、Enter キーを押してプログラムを終了します。

    > **注** *この例では、**SpeechTranslationConfig** を使用して音声をテキストに翻訳してから、**SpeechConfig**を使用して翻訳を音声として合成しました。実際、**SpeechTranslationConfig** を使用して翻訳を直接合成できますが、これは単一の言語に翻訳する場合にのみ機能し、スピーカーに直接送信されるのではなく、通常はファイルとして保存されるオーディオストリームになります。*

## 詳細情報

**Speech translation** API の使用の詳細については、[音声翻訳のドキュメント](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-translation)を参照してください。
