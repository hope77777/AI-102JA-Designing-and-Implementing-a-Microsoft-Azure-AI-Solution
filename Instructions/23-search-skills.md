---
lab:
    title: 'Azure Cognitive Search 用カスタム スキルの作成'
    module: 'モジュール 12 - ナレッジ マイニング ソリューションの作成'
---

# Azure Cognitive Search 用カスタム スキルの作成

Azure Cognitive Search は、コグニティブ スキルの強化パイプラインを使用して、ドキュメントから AI で生成されたフィールドを抽出し、検索インデックスに含めます。使用できる組み込みスキルの包括的なセットがありますが、これらのスキルでは満たされない特定の要件がある場合は、カスタムス キルを作成できます。

この演習では、ドキュメント内の個々の単語の頻度を表にして、最もよく使用される上位5つの単語のリストを生成するカスタムスキルを作成し、それを架空の旅行代理店である Margie'sTravel の検索ソリューションに追加します。

## このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボで作業している環境に既に複製している場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐ複製してください。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git: Clone** コマンドを実行して、 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでもかまいません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**「今はしない」** を選択します。

## Azure リソースを作成する

> **注**: 以前に **Azure Cognitive Search ソリューションの作成 (22-azure-search.md)** の演習を完了し、サブスクリプションにこれらの Azure リソースが残っている場合は、このセクションをスキップして、**「検索ソリューションの作成」** セクションから開始できます。それ以外の場合は、以下の手順に従って、必要な Azure リソースをプロビジョニングします。

1. `https;//portal.azure.com` で Azure portal を開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. サブスクリプションの**リソース グループ**を表示します。
3. リソース グループが提供されている制限付きサブスクリプションを使用している場合は、リソース グループを選択してそのプロパティを表示します。それ以外の場合は、選択した名前で新しいリソース グループを作成し、作成されたらそのグループに移動します。
4. リソース グループの **「概要」** ページで、**サブスクリプション ID** と**場所**をメモします。これらの値は、後続の手順でリソース グループの名前とともに必要になります。
5. Visual Studio Code で、**23-custom-search-skill** フォルダーを展開し、**setup.cmd** を選択します。このバッチ スクリプトを使用して、必要な Azure リソースを作成するために必要な Azure コマンド ライン インターフェイス (CLI) コマンドを実行します。
6. 2**23-custom-search-skill** を右クリックし、**「統合ターミナルで開く」** を選択します。
7. 「ターミナル」 ペインで、次のコマンドを入力して、Azure サブスクリプションへの認証済み接続を確立します。

    ```
    az login --output none
    ```

8. プロンプトが表示されたら、Azure サブスクリプションにサインインします。次に、Visual Studio Code に戻り、サインイン プロセスが完了するのを待ちます。
9. 次のコマンドを実行して、Azure の場所を一覧表示します。

    ```
    az account list-locations -o table
    ```

10. 出力で、リソース グループの場所に対応する **Name** 値を見つけます (たとえば、*米国東部*の場合、対応する名前は *eastus* です)。
11. **setup.cmd** スクリプトで、**subscription_id**、**resource_group**、および **location** 変数の宣言を、サブスクリプション ID、リソース グループ名、および場所名に適切な値で変更します。その後、変更を保存します。
12. **23-custom-search-skill** フォルダーのターミナルで、次のコマンドを入力してスクリプトを実行します。

    ```
    setup
    ```

    > **注**: Search CLI モジュールはプレビュー中であり、*- R実行中*のプロセスでスタックする可能性があります。これが 2 分以上続く場合は、CTRL+C を押して長時間実行中の操作をキャンセルし、スクリプトを終了するかどうかを尋ねられたら **N** を選択します。その後、正常に完了するはずです。
    >
    > スクリプトが失敗した場合は、正しい変数名で保存したことを確認して、再試行してください。

13. スクリプトが完了したら、表示される出力を確認し、Azure リソースに関する次の情報をメモします (これらの値は後で必要になります)。
    - ストレージ アカウント名
    - ストレージ接続文字列
    - Cognitive Services アカウント
    - Cognitive Services キー
    - 検索サービス エンドポイント
    - 検索サービス管理者キー
    - 検索サービス クエリ キー

14. Azure portal で、リソースグループを更新し、Azure Storage アカウント、Azure Cognitive Services リソース、および Azure Cognitive Search リソースが含まれていることを確認します。

## 検索ソリューションの作成

必要な Azure リソースが揃ったので、次のコンポーネントで構成される検索ソリューションを作成できます。

- Azure ストレージ コンテナー内のドキュメントを参照する**データ ソース**。
- ドキュメントから AI で生成されたフィールドを抽出するスキルの強化パイプラインを定義する**スキルセット**。
- 検索可能なドキュメントレコードのセットを定義する**インデックス**。
- データソースからドキュメントを抽出し、スキルセットを適用して、インデックスにデータを入力するインデクサー。データソースからドキュメントを抽出し、スキルセットを適用して、インデックスにデータを入力するインデクサー。 

この演習では、Azure Cognitive Search REST インターフェイスを使用して、JSON リクエストを送信することでこれらのコンポーネントを作成します。

1. Visual Studio Cod eの **23-custom-search-skill** フォルダーで、**create-search** フォルダーを展開し、**data_source.json** を選択します。このファイルには、**margies-custom-data**.という名前のデータソースの JSON 定義が含まれています。
2. **YOUR_CONNECTION_STRING** プレースホルダーを Azure ストレージアカウントの接続文字列に置き換えます。これは次のようになります。

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *接続文字列は、Azureポータルのストレージアカウントの 「**アクセスキー**」 ページにあります。*

3. 更新された JSON ファイルを保存して閉じます。
4. **create-search** フォルダーで、**skillset.json** を開きます。このファイルには、**margies-custom-skillset** という名前のスキルセットの JSO N定義が含まれています。
5. スキルセット定義の上部にある **cognitiveServices** 要素で、**YOUR_COGNITIVE_SERVICES_KEY** プレースホルダーをCognitive Services リソースのいずれかのキーに置き換えます。

    *キーは、Azure portal の Cognitive Services リソースの **「キーとエンドポイント」** ページにあります。*

6. 更新された JSON ファイルを保存して閉じます。
7. **create-search** フォルダーで、i**index.json**.を開きます。このファイルには**margies-custom-index**という名前のインデックスの JSON 定義が含まれています。
8. インデックスの JSON を確認し、変更を加えずにファイルを閉じます。
9. **create-search** フォルダーで、**indexer.json**.を開きます。このファイルには**margies-custom-indexer**という名前のインデックスの JSON 定義が含まれています。
10. インデクサーの JSON を確認し、変更を加えずにファイルを閉じます。
11. **create-search** フォルダーで、**create-search.cmd**.を開きます。このバッチ スクリプトは、cURL ユーティリティを使用して、Azure Cognitive Search リソースの REST インターフェイスに JSON 定義を送信します。
12. **YOUR_SEARCH_URL** 変数と **YOUR_ADMIN_KEY** 変数のプレースホルダーを、Azure Cognitive Search リソースの **Url** と**管理キー**の 1 つに置き換えます。

    *これらの値は、Azure portal の Azure Cognitive Search リソースの **「概要」** ページと **「キー」** ページにあります。*

13. 更新したバッチファイルを保存します。
14. **create-search** フォルダーを右クリックし、**「統合ターミナルで開く」** を選択します。
15. **create-search** フォルダーのターミナルペインで、次のコマンドを入力してバッチ スクリプトを実行します。

    ```
    create-search
    ```

16. スクリプトが完了したら、Azure portal の Azure Cognitive Search リソースのページで、**「インデクサー」** ページを選択し、インデックス作成プロセスが完了するのを待ちます。

    ***「更新」** を選択して、インデックス作成操作の進行状況を追跡できます。完了するまでに 1 分ほどかかる場合があります。*

## インデックスを検索する

インデックスができたので、検索できます。

1. Azure Cognitive Search リソースのブレードの上部で、**「検索エクスプローラー」** を選択します。
2. 検索エクスプローラーの **「クエリ文字列」** ボックスに、次のクエリ文字列を入力し、**「検索」** を選択します。

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment gt 0.5
    ```

    このクエリは、*レビュー担当者*が作成したロンドンに言及している、**感情スコア**が *0.5* を超えるすべてのドキュメント (つまり、*ロンドン*に言及している肯定的なレビュー) の **url**、**感情**、および**キーフレーズ**を取得します。

## カスタム スキル Azure 関数を作成する

検索ソリューションには、前のタスクで見た感情スコアやキー フレーズのリストなど、ドキュメントからの情報でインデックスを充実させる多くの組み込みの認知スキルが含まれています。

カスタム スキルを作成することで、インデックスをさらに強化できます。たとえば、各ドキュメントで最も頻繁に使用される単語を特定すると便利な場合がありますが、この機能を提供する組み込みのスキルはありません。

単語カウント機能をカスタムスキルとして実装するには、好みの言語で Azure 関数を作成します。

1. Visual Studio Codeで、「Azure Extensions」 タブ (**&boxplus;**) を表示し、**Azure Functions** 拡張機能がインストールされていることを確認します。この拡張機能を使用すると、Visual StudioCode から AzureFunctions を作成してデプロイできます。
2. 「Azure」 タブ (**&Delta;**) の **「Azure Functions」** ペインで、希望する言語に応じて、次の設定で新しいプロジェクト (&#128194;) を作成します。

    ### **C#**

    - **フォルダー**: **23-custom-search-skill/C-Sharp/wordcount** を参照います。
    - **言語**: C#
    - **テンプレート**: HTTP トリガー
    - **関数名**: wordcount
    - **名前空間**: margies.search
    - **承認レベル**: 関数

    ### **Python**

    - **フォルダー**: **23-custom-search-skill/Python/wordcount** 参照します。
    - **言語**: Python
    - **仮想環境**: 仮想環境をスキップ
    - **テンプレート**: HTTP トリガー
    - **関数名**: wordcount
    - **承認レベル**: 関数

    ***launch.json** を 上書きするように求められた場合は、上書きしてください。*

3. **エクスプローラ**ー (**&#128461;**) タブに戻り、**wordcount** フォルダーに Azure 関数のコード ファイルが含まれていることを確認します。

    * Pythonを選択した場合、コードファイルは **wordcount** という名前のサブフォルダーにある可能性があります。

4. 関数のメイン コード ファイルは自動的に開かれているはずです。そうでない場合は、選択した言語に適したファイルを開きます。
    - **C#**: wordcount.cs
    - **Python**: \_\_init\_\_&#46;py

5. ファイルの内容全体を選択した言語の次のコードに置き換えます、

### **C#**

```C#
using System.IO;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Newtonsoft.Json;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;
using System.Text.RegularExpressions;
using System.Linq;

namespace margies.search
{
    public static class wordcount
    {

        //応答のためにクラスを定義する
        private class WebApiResponseError
        {
            public string message { get; set; }
        }

        private class WebApiResponseWarning
        {
            public string message { get; set; }
        }

        private class WebApiResponseRecord
        {
            public string recordId { get; set; }
            public Dictionary<string, object> data { get; set; }
            public List<WebApiResponseError> errors { get; set; }
            public List<WebApiResponseWarning> warnings { get; set; }
        }

        private class WebApiEnricherResponse
        {
            public List<WebApiResponseRecord> values { get; set; }
        }

        //カスタム スキルの関数
        [FunctionName("wordcount")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)]HttpRequest req, ILogger log)
        {
            log.LogInformation("Function initiated.");

            string recordId = null;
            string originalText = null;

            string requestBody = new StreamReader(req.Body).ReadToEnd();
            dynamic data = JsonConvert.DeserializeObject(requestBody);

            // 検証
            if (data?.values == null)
            {
                return new BadRequestObjectResult(" Could not find values array");
            }
            if (data?.values.HasValues == false || data？.values.First.HasValues == false）
            {
                return new BadRequestObjectResult("Could not find valid records in values array");
            }

            WebApiEnricherResponse response = new WebApiEnricherResponse（）;
            response.values = new List<WebApiResponseRecord>();
            foreach（var record in data？.values）
            {
                recordId = record.recordId?.Value as string;
                originalText = record.data?.text?.Value as string;

                if (recordId == null)
                {
                    return new BadRequestObjectResult("recordId cannot be null");
                }

                // 応答をまとめる。
                WebApiResponseRecord responseRecord = new WebApiResponseRecord();
                responseRecord.data = new Dictionary<string, object>();
                responseRecord.recordId = recordId;
                responseRecord.data.Add("text", Count(originalText));

                response.values.Add(responseRecord);
            }

            return (ActionResult)new OkObjectResult(response); 
        }


            public static string RemoveHtmlTags(string html)
        {
            string htmlRemoved = Regex.Replace(html, @"<script[^>]*>[\s\S]*?</script>|<[^>]+>| ", " ").Trim();
            string normalised = Regex.Replace(htmlRemoved, @"\s{2,}", " ");
            return normalised;
        }

        public static List<string> Count(string text)
        {
            
            //html エレメントを削除する
            text=text.ToLowerInvariant();
            string html = RemoveHtmlTags(text);
            
            //単語リストを分割する
            List<string> list = html.Split(" ").ToList();
            
            //英数文字以外を削除する
            var onlyAlphabetRegEx = new Regex(@"^[A-z]+$");
            list = list.Where(f => onlyAlphabetRegEx.IsMatch(f)).ToList();

            //ストップ ワードを削除する
            string[] stopwords = { "", "i", "me", "my", "myself", "we", "our", "ours", "ourselves", "you", 
                    "you're", "you've", "you'll", "you'd", "your", "yours", "yourself", 
                    "yourselves", "he", "him", "his", "himself", "she", "she's", "her", 
                    "hers", "herself", "it", "it's", "its", "itself", "they", "them", 
                    "their", "theirs", "themselves", "what", "which", "who", "whom", 
                    "this", "that", "that'll", "these", "those", "am", "is", "are", "was",
                    "were", "be", "been", "being", "have", "has", "had", "having", "do", 
                    "does", "did", "doing", "a", "an", "the", "and", "but", "if", "or", 
                    "because", "as", "until", "while", "of", "at", "by", "for", "with", 
                    "about", "against", "between", "into", "through", "during", "before", 
                    "after", "above", "below", "to", "from", "up", "down", "in", "out", 
                    "on", "off", "over", "under", "again", "further", "then", "once", "here", 
                    "there", "when", "where", "why", "how", "all", "any", "both", "each", 
                    "few", "more", "most", "other", "some", "such", "no", "nor", "not", 
                    "only", "own", "same", "so", "than", "too", "very", "s", "t", "can", 
                    "will", "just", "don", "don't", "should", "should've", "now", "d", "ll",
                    "m", "o", "re", "ve", "y", "ain", "aren", "aren't", "couldn", "couldn't", 
                    "didn", "didn't", "doesn", "doesn't", "hadn", "hadn't", "hasn", "hasn't", 
                    "haven", "haven't", "isn", "isn't", "ma", "mightn", "mightn't", "mustn", 
                    "mustn't", "needn", "needn't", "shan", "shan't", "shouldn", "shouldn't", "wasn", 
                    "wasn't", "weren", "weren't", "won", "won't", "wouldn", "wouldn't"}; 
            list = list.Where(x => x.Length > 2).Where(x => !stopwords.Contains(x)).ToList();
            
            //キーとカウント、さらにはカウントによる順番によって特徴のある単語を取得する
            var keywords = list.GroupBy(x => x).OrderByDescending(x => x.Count());
            var klist = keywords.ToList();

            // 上位 10 個の単語を返す
            var numofWords = 10;
            if(klist.Count<10)
                numofWords=klist.Count;
            List<string> resList = new List<string>();
            for (int i = 0; i < numofWords; i++)
            {
                resList.Add(klist[i].Key);
            }
            return resList;
        }
    }
}
```

## **Python**

```Python
import logging
import os
import sys
import json
from string import punctuation
from collections import Counter
import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Wordcount function initiated.')

    # 結果は "values" バッグになります
    result = {
        "values": []
    }
    statuscode = 200

    # 単語のカウント内のこのリストから、単語を除外します
    stopwords = [ ''、 'i'、 'me'、 'my'、 'myself'、 'we'、 'our'、 'ours'、 'ourselves'、 'you'、 
                "you're"、 "you've" 、 "you'll"、 "you'd"、 'your'、 'yours'、 'yourself'、 
                'yourselves', 'he', 'him', 'his', 'himself', 'she', "she's", 'her', 
                'hers', 'herself', 'it', "it's", 'its', 'itself', 'they', 'them', 
                'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
                'this', 'that', "that'll", 'these', 'those', 'am', 'is', 'are', 'was',
                'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
                'does'、 'did'、 ' '、' a '、' an '、' the '、' and '、' but '、' if '、' or '、 
                ' because '、' as '、' until '、' while '、' of '、' at '、' by '、' for '、' with '、 
                ' about '、' Against '、' between '、' into '、' through '、' during '、' before ' 、 
                ^after'、 'above'、 'below'、 'to'、 'from'、 'up'、 'down'、 'in'、 'out'、 
                'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
                'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
                'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
                'only', 'own', 'same', 'so', 'than', 'too', 'very', 's', 't', 'can', 
                'will', 'just', 'don', "don't", 'should', "should've", 'now', 'd', 'll',
                'm', 'o', 're', 've', 'y', 'ain', 'aren', "aren't", 'couldn', "couldn't", 
                'didn', "didn't", 'doesn', "doesn't", 'hadn', "hadn't", 'hasn', "hasn't", 
                'haven', "haven't", 'isn', "isn't", 'ma', 'mightn', "mightn't", 'mustn', 
                "mustn't", 'needn', "needn't", 'shan', "shan't", 'shouldn', "shouldn't", 'wasn', 
                "wasn't", 'weren', "weren't", 'won', "won't", 'wouldn', "wouldn't"]

    try:
        values = req.get_json().get('values')
        logging.info（values）

        for rec in values：
            # このレコードのベーシックな JSON 応答を作成する
            val = {
                    "recordId": rec['recordId'],
                    "data": {
                        "text":None
                    },
                    "errors": None,
                    "warnings": None
                }
            try:
                # 入力レコードから処理されるテキストを取得する
                txt = rec['data']['text']
                # 数値を削除する
                txt = ''.join(c for c in txt if not c.isdigit())
                # 句読点と小文字を削除する
                txt = ''.join(c for c in txt if c not in punctuation).lower()
                # ストップワードを削除する
                txt = ' '.join(w for w in txt.split() if w not in stopwords)
                # 単語をカウントして、出現個数の多い上位 10 を取得する
                wordcount = Counter(txt.split()).most_common(10)
                words = [w[0] for w in wordcount]
                # このテキスト レコードの出力に上位 10 個の単語を追加する
                val["data"]["text"] = words
            except:
                # このテキスト レコードではエラーが発生するため、エラーと警告のリストを追加する
                val["errors"] =[{"message": "An error occurred processing the text."}]
                val["warnings"] = [{"message": "One or more inputs failed to process."}]
            finally:
                # このレコードの数値を応答に追加する
                result["values"].append(val)
    except Exception as ex:
        statuscode = 500
        # グローバル エラーが発生したため、エラー応答が返される
        val = {
                "recordId": None,
                "data": {
                    "text":None
                },
                "errors": [{"message "：ex.args}]、
                "warnings": [{" message "： "The request failed to process."}]
            }
        result["values"].append(val)
    finally:
        # 応答を返す
        return func.HttpResponse(body=json.dumps(result), mimetype="application/json", status_code=statuscode)
```
    
6. 更新したファイルを保存します。
7. コードファイルを含む **wordcount** フォルダーを右クリックし、**「関数アプリにデプロ**イ」 を選択します。次に、次の言語固有の設定で関数をデプロイします (プロンプトが表示されたら Azure にサインインします)

    ### **C#**

    - **Subscription** (必要な場合): Azure サブスクリプションを選択します。
    - **関数**: Azureで新しい関数アプリを作成する (詳細)
    - **関数アプリ名**: グローバルに一意な名前を入力します。
    - **Runtime**: .NET Core 3.1
    - **OS**: Linux
    - **ホスティング プラン**: Consumption
    - **リソース グループ**: Azure Cognitive Search リソースを含むリソースグループ。
        - 注記: このリソースグループに既に Windows ベースの Web アプリが含まれている場合、Linux ベースの関数をそこにデプロイすることはできません。既存の Web アプリを削除するか、関数を別のリソース グループにデプロイします。
    - **ストレージ アカウント**: Margie's Travel のドキュメントが保存されているストレージ数。
    - **Application Insights**: 今はしない

    *Visual Studio Code は、関数プロジェクトの作成時に保存された v**vscode** フォルダーの構成設定に基づいて、コンパイルされたバージョンの関数を (**bin** ーに) 展開します。*

    ### **Python**

    - **Subscription** (必要な場合): Azure サブスクリプションを選択します。
    - **関数**: Azureで新しい関数アプリを作成する (詳細)
    - **関数アプリ名**: グローバルに一意な名前を入力します。
    - **Runtime**: Python 3.8
    - **ホスティング プラン**: Consumption
    - **リソース グループ**: Azure Cognitive Search リソースを含むリソースグループ。
        - 注記: このリソースグループに既に Windows ベースの Web アプリが含まれている場合、Linux ベースの関数をそこにデプロイすることはできません。既存の Web アプリを削除するか、関数を別のリソース グループにデプロイします。
    - **ストレージ アカウント**: Margie's Travel のドキュメントが保存されているストレージ数。
    - **Application Insights**: 今はしない

8. Visual Studio Codeが関数をデプロイするのを待ちます。展開が完了すると、通知が表示されます。

## 関数をテストする

Azure にデプロイしたので、Azureポー タルで関数をテストできます。

1. [Azure portal](https://portal.azure.com) を開き、関数アプリを作成したリソースグループを参照します。次に、関数アプリのアプリサービスを開きます。
2. アプリ サービスのブレードの **関数** ページで**wordcount** 関数を開きます。
3. **wordcount** 関数ブレードで、**「コード+テスト」** ページを表示し、**「テスト/実行」** ペイン開きます。
4. **「テスト/実行」** ペインで、既存の **Body** を次の JSON に置き換えます。これは、1 つ以上のドキュメントのデータを含むレコードが処理のために送信される Azure Cognitive Search スキルで期待されるスキーマを反映しています

    ```
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```
    
5. **「実行」** をクリックして、HTTP 応答コンテンツを表示します。これは、スキルを消費するときに Azure Cognitive Search が期待するスキーマを反映しており、各ドキュメントの応答が返されます。この場合、応答は、出現頻度の降順で各ドキュメントの最大 10 個の用語で構成されます。

    ```
    {
    "values": [
        {
        "recordId": "a1",
        "data": {
            "text": [
            "tiger"、
            "burning"、
            "bright"、
            "darkness"、
            "night"
            ]
        },
        "errors": null,
        "warnings": null
        },
        {
        "recordId": "a2",
        "data": {
            "text": [
            "rain",
            "spain",
            "stays",
            "mainly",
            "plains",
            "thats",
            "youll",
            "find"
            ]
        },
        "errors": null,
        "warnings": null
        }
    ]
    }
    ```

6. **「テスト/実行」** ペインを閉じ、**wordcount** 関数ブレードで「**関数 URL の取得**」 をクリックします。次に、デフォルト キーの URL をクリップボードにコピーします。これは次の手順で必要になります。

## 検索ソリューションにカスタム スキルを追加する

次に、関数をカスタム スキルとして検索ソリューション スキル セットに含め、生成された結果をインデックスのフィールドにマップする必要があります。 

1. Visual Studio Code の **23-custom-search-skill/update-search** フォルダーで、**update-skillset.json** ファイルを開きます。これには、スキルセットの JSON 定義が含まれています。
2. スキルセットの定義を確認します。これには、以前と同じスキルに加えて、**get-top-words** という名前の新しい **WebApiSkill**l スキルが含まれています。
3. **get-top-words** スキル定義を編集して、**uri** 値をAzure関数のURL（前の手順でクリップボードにコピーしたもの）に設定し、**YOUR-FUNCTION-APP-URL** を置き換えます。
4. スキルセット定義の上部にある **cognitiveServices** 要素で、**YOUR_COGNITIVE_SERVICES_KEY** プレースホルダーを Cognitive Services サービスリ ソースのいずれかのキーに置き換えます。

    *キーは、Azure portal の Cognitive Services リソースの **「キーとエンドポイント」** ページにあります。*

5. 更新された JSON ファイルを保存して閉じます。
6. **update-search** フォルダーで、**update-index.json** を開きます。このファイルには、**margies-custom-index** インデックスの JSON 定義が含まれており、インデックス定義の下部に **top_words**という名前の追加フィールドがあります。
7. インデックスの JSON を確認し、変更を加えずにファイルを閉じます。
8. **update-search** フォルダーで、**update-indexer.json** を開きます。このファイルには、**margies-custom-indexer** のJSON定義と、**top_words** フィールドの追加のマッピングが含まれています。
9. インデクサーの JSON を確認し、変更を加えずにファイルを閉じます。
10. **update-search** フォルダーで、**update-search.cmd**.を開きます。このバッチ スクリプトは、cURL ユーティリティを使用して、更新されたJSON 定義を Azure CognitiveSearch リソースの REST インターフェイスに送信します。
11. **YOUR_SEARCH_URL** 変数と **YOUR_ADMIN_KEY** 変数のプレースホルダーを、Azure Cognitive Search リソースの **Url** と**管理キー**の 1 つに置き換えます。

    *これらの値は、Azure portal の Azure Cognitive Search リソースの **「概要」** ページと **「キー」** ページにあります。*

12. 更新したバッチファイルを保存します。
13. **update-search** フォルダーを右クリックし、**「統合ターミナルで開く」** を選択します。
14. **update-search** フォルダーのターミナルペインで、次のコマンドを入力してバッチスクリプトを実行します。

    ```
    update-search
    ```

15. スクリプトが完了したら、Azure portal の Azure Cognitive Search リソースのページで、**「インデクサー」** ページを選択し、インデックス作成プロセスが完了するのを待ちます。

    ***「更新」** を選択して、インデックス作成操作の進行状況を追跡できます。完了するまでに 1 分ほどかかる場合があります。*

## インデックスを検索する

インデックスができたので、検索できます。

1. Azure Cognitive Search リソースのブレードの上部で、**「検索エクスプローラー」** を選択します。
2. 検索エクスプローラーの **「クエリ文字列」** ボックスに、次のクエリ文字列を入力し、**「検索」** を選択します。

    ```
    search=Las Vegas&$select=url,top_words
    ```

    このクエリは、*Las Vegas* に言及しているすべてのドキュメントの **url** フィールドと **top_words** フィールドを取得します。

## 詳細情報

Azure Cognitive Search のカスタム スキルの作成の詳細については、[Azure Cognitive Searchのドキュメント](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface)を参照してください。
