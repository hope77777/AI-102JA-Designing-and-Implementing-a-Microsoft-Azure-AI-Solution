---
lab:
    title: '顔の検出、分析、認識'
    module: 'モジュール 10 - 顔の検出、分析、認識'
---

# 顔の検出、分析、認識

人間の顔を検出、分析、認識する機能は、AI のコア機能です。この演習では、画像内の顔を操作するために使用できる 2 つの Azure Cognitive Services である **Computer Vision** サービスと **Face** サービスについて説明します。

## このコースのリポジトリを複製する

まだ行っていない場合は、このコースのコード リポジトリを複製する必要があります。

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
5. リソースがデプロイされたら、リソースに移動して、その**キーとエンドポイント**のページを表示します。次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## Computer Vision SDKを使用する準備をする

この演習では、Computer Vision SDK を使用して画像内の顔を分析する部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の**エクスプローラー** ペインで、**19-iface** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **computer-vision** フォルダーを右クリックして、統合ターミナルを開きます。次に、言語設定に適したコマンドを実行して、Computer Vision SDK パッケージをインストールします。

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```
    
3. **computer-vision** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください。
    - **C#**: appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの**エンドポイント**と認証**キー**を反映します。変更を保存します。
4. **computer-vision** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください

    - **C#**: Program.cs
    - **Python**: detect-faces&period;py

    コード ファイルを開き、上部の既存の名前空間参照の下で、**「名前空間のインポート」** というコメントを見つけます。次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision SDK を使用するために必要な名前空間をインポートします

**C#**

```C#
// 名前空間をインポートする
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```

**Python**

```Python
# 名前空間をインポートする
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
from msrest.authentication import CognitiveServicesCredentials
```

## ビューの画像は、あなたが分析します

この演習では、Computer Vision サービスを使用して、人の画像を分析します。

1. Visual Studio Code で、**computer-vision** フォルダーとそれに含まれる **images** フォルダーを展開します。
2. **people.jpg** 画像を選択して表示します。

## 画像内の顔を検出する

これで、SDK を使用して Computer Vision サービスを呼び出し、画像内の顔を検出する準備が整いました。

1. クライアント アプリケーションのコード ファイル (**Program.cs** または **detect-faces&period;py**) の **Main** 関数で、構成設定をロードするためのコードが提供されていることに注意してください。次に、コメント **「Computer Vision クライアントを認証する」** を見つけます。次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision クライアント オブジェクトを作成および認証します

**C#**

```C#
// Computer Vision クライアントを認証する
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Computer Vision クライアントを認証する
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```

2. **Main** 関数で、追加したコードの下で、コードが画像ファイルへのパスを指定し、**AnalyzeFaces** という名前の関数に画像パスを渡すことに注意してください。この関数はまだ完全には実装されていません。

3. **AnalyzeFaces** 関数のコメン **「取得する特徴を指定する (顔)」** の下に、次のコードを追加します。

**C#**

```C#
// 取得する特徴を指定する (顔)
List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
{
    VisualFeatureTypes.Faces
};
```

**Python**

```Python
# 取得する特徴を指定する (顔)
features = [VisualFeatureTypes.faces]
```
    
4. **AnalyzeFaces** 関数のコメント **「Get image analysis」** の下に、次のコードを追加します。

**C#**

```C
//  画像分析を取得する
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // 顔を取得する
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // 描画用に画像を準備する
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // 注釈付き顔を描画する
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person aged approximately {face.Age}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // 注釈付き画像を保存する
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
#  画像分析を取得する
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream（image_data、features）

    # 顔を取得する
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # 描画用に画像を準備する
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # 注釈付き顔を描画する
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person aged approximately {}'.format(face.age)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # 注釈付き画像を保存する
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```
    
5. 変更を保存し、**computer-vision** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python detect-faces.py
```

6. 検出された顔の数を示す出力を確認します。
7. コードファイルと同じフォルダーに生成された **detected_faces.jpg** ファイルを表示して、注釈付きの顔を確認します。この場合、コードは顔の属性を使用して画像内の各人物の年齢を推定し、境界ボックスの座標を使用して各顔の周りに長方形を描画しています。

## Face SDK を使用する準備をする

**Computer Vision** サービスは (他の多くの画像分析機能とともに) 基本的な顔検出を提供しますが、**Face** サービスは顔の分析と認識のためのより包括的な機能を提供します。

1. Visual Studio Code の**エクスプローラー** ペインで、**19-iface** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **face-api** フォルダーを右クリックして、統合ターミナルを開きます。次に、言語設定に適したコマンドを実行して、Face SDK パッケージをインストールします。

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
```

**Python**

```
pip install azure-cognitiveservices-vision-face==0.4.1
```
    
3. **face-api** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください。
    - **C#**: appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの**エンドポイント**と認証**キー**を反映します。変更を保存します。
4. **face-api** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください

    - **C#**: Program.cs
    - **Python**: analyze-faces&period;py

    コード ファイルを開き、上部の既存の名前空間参照の下で、**「名前空間のインポート」** というコメントを見つけます。次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision SDK を使用するために必要な名前空間をインポートします

**C#**

```C#
// 名前空間をインポートする
using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
```

**Python**

```Python
# 名前空間をインポートする
from azure.cognitiveservices.vision.face import FaceClient
from azure.cognitiveservices.vision.face.models import FaceAttributeType
from msrest.authentication import CognitiveServicesCredentials
```

5. **Main** 関数では、構成設定をロードするためのコードが提供されていることに注意してください。次に、コメント **「Authenticate Face client」** を見つけます。次に、このコメントの下に、次の言語固有のコードを追加して、**FaceClient** オブジェクトを作成および認証します

**C#**

```C#
// Authenticate Face client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
faceClient = new FaceClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Authenticate Face client
credentials = CognitiveServicesCredentials(cog_key)
face_client = FaceClient(cog_endpoint, credentials)
```

6. **Main** 関数で、追加したコードの下に、コード内の関数を呼び出して Face サービスの機能を調べることができるメニューがコードに表示されることに注意してください。この演習の残りの部分では、これらの関数を実装します。

## 顔の検出と分析

Face サービスの最も基本的な機能の 1 つは、画像内の顔を検出し、年齢、感情表現、髪の色、眼鏡の存在などの属性を決定することです。

1. アプリケーションのコードファイルの **Main** 関数で、ユーザーがメニュー オプション **1** を選択した場合に実行されるコードを調べます。このコードは **DetectFaces** 関数を呼び出し、パスを画像ファイルに渡します。
2. コードファイルで **DetectFaces** 関数を見付け、コメント **「検索対象の顔の特徴を指定する」** の下に、次のコードを追加します。

**C#**

```C#
// 検索対象の顔の特徴を指定する
List<FaceAttributeType?> features = new List<FaceAttributeType?>
{
    FaceAttributeType.Age、
    FaceAttributeType.Emotion,
    FaceAttributeType.Glasses
};
```

**Python**

```Python
# 検索対象の顔の特徴を指定する
features = [FaceAttributeType.age,
            FaceAttributeType.emotion,
            FaceAttributeType.glasses]
```

3. **DetectFaces** 関数で、追加したコードの下に、コメント「**顔を取得する**」を見つけて、次のコードを追加します。

**C#**

```C
// 顔を取得する
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // 描画用に画像を準備する
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);

        // 注釈付き顔を描画する
        foreach (var face in detected_faces)
        {
            // 顔のプロパティを取得する
            Console.WriteLine($"\nFace ID: {face.FaceId}");
            Console.WriteLine($" - Age: {face.FaceAttributes.Age}");
            Console.WriteLine($" - Emotions:");
            foreach (var emotion in face.FaceAttributes.Emotion.ToRankedList())
            {
                Console.WriteLine($"   - {emotion}");
            }

            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // 注釈付き顔を描画する
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face ID: {face.FaceId}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // 注釈付き画像を保存する
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# 顔を取得する
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # 描画用に画像を準備する
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # 注釈付き顔を描画する
        for face in detected_faces:

            # 顔のプロパティを取得する
            print('\nFace ID: {}'.format(face.face_id))
            detected_attributes = face.face_attributes.as_dict()
            age = 'age unknown' if 'age' not in detected_attributes.keys() else int(detected_attributes['age'])
            print(' - Age: {}'.format(age))

            if 'emotion' in detected_attributes:
                print(' - Emotions:')
                for emotion_name in detected_attributes['emotion']:
                    print('   - {}: {}'.format(emotion_name, detected_attributes['emotion'][emotion_name]))
            
            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # 注釈付き顔を描画する
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face ID: {}'.format(face.face_id)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # 注釈付き画像を保存する
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. **DetectFaces** 関数に追加したコードを調べます。画像ファイルを分析し、年齢、感情、眼鏡の存在など、画像ファイルに含まれるすべての顔を検出します。各顔に割り当てられた一意の顔識別子を含む、各顔の詳細が表示されます。顔の位置は、境界ボックスを使用して画像に示されます。
5. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します

**C#**

```
dotnet run
```

*C# 出力には、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これらは無視してかまいません。*

**Python**

```
python analyze-faces.py
```

6. プロンプトが表示されたら、**1** を入力し、出力を観察します。出力には、検出された各顔の ID と属性が含まれている必要があります。
7. コードファイルと同じフォルダーに生成された **detected_faces.jpg** ファイルを表示して、注釈付きの顔を確認します。

## 顔比較

一般的なタスクは、顔を比較し、類似している顔を見つけることです。このシナリオでは、画像内の人物を*特定する*必要はありません。複数の画像が*同じ*人物 (または少なくとも似ている人物) を示しているかどうかを判断するだけです。 

1. アプリケーションのコードファイルの **Main** 関数で、ユーザーがメニュー オプション **2** を選択した場合に実行されるコードを調べます。このコードは **CompareFaces** 関数を呼び出し、パスを 2 つの画像ファイル (**person1.jpg** と **people.jpg**) に渡します。
2. コード ファイルで **CompareFaces** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します。

**C#**

```C
// 画像 1 の顔と、画像 2 の顔が同一であるかどうか判断する
DetectedFace image_i_face;
using (var image1Data = File.OpenRead(image1))
{    
    // 画像 1 から最初の顔を取得する
    var image1_faces = await faceClient.Face.DetectWithStreamAsync(image1Data);
    if (image1_faces.Count > 0)
    {
        image_i_face = image1_faces[0];
        Image img1 = Image.FromFile(image1);
        Graphics graphics = Graphics.FromImage(img1);
        Pen pen = new Pen(Color.LightGreen, 3);
        var r = image_i_face.FaceRectangle;
        Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        String output_file = "face_to_match.jpg";
        img1.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file); 

        //画像 2 からすべての顔を取得する
        using (var image2Data = File.OpenRead(image2))
        {    
            var image2Faces = await faceClient.Face.DetectWithStreamAsync(image2Data);

            // 顔を取得する
            if (image2Faces.Count > 0)
            {

                var image2FaceIds = image2Faces.Select(f => f.FaceId).ToList<Guid?>();
                var similarFaces = await faceClient.Face.FindSimilarAsync((Guid)image_i_face.FaceId,faceIds:image2FaceIds);
                var similarFaceIds = similarFaces.Select(f => f.FaceId).ToList<Guid?>();

                // 描画用に画像を準備する
                Image img2 = Image.FromFile(image2);
                Graphics graphics2 = Graphics.FromImage(img2);
                Pen pen2 = new Pen(Color.LightGreen, 3);
                Font font2 = new Font("Arial", 4);
                SolidBrush brush2 = new SolidBrush(Color.Black);

                // 注釈付き顔を描画する
                foreach (var face in image2Faces)
                {
                    if (similarFaceIds.Contains(face.FaceId))
                    {
                        // 注釈付き顔を描画する
                        var r2 = face.FaceRectangle;
                        Rectangle rect2 = new Rectangle(r2.Left, r2.Top, r2.Width, r2.Height);
                        graphics2.DrawRectangle(pen2, rect2);
                        string annotation = "Match!";
                        graphics2.DrawString(annotation,font2,brush2,r2.Left, r2.Top);
                    }
                }

                // 注釈付き画像を保存する
                String output_file2 = "matched_faces.jpg";
                img2.Save(output_file2);
                Console.WriteLine(" Results saved in " + output_file2);   
            }
        }

    }
}
```

**Python**

```Python
# 画像 1 の顔と、画像 2 の顔が同一であるかどうか判断する
with open(image_1, mode="rb") as image_data:
    # 画像 1 から最初の顔を取得する
    image_1_faces = face_client.face.detect_with_stream(image=image_data)
    image_1_face = image_1_faces[0]

    # 画像内の顔をハイライトする
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_1)
    draw = ImageDraw.Draw(image)
    color = 'lightgreen'
    r = image_1_face.face_rectangle
    bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
    draw = ImageDraw.Draw(image)
    draw.rectangle(bounding_box, outline=color, width=5)
    plt.imshow(image)
    outputfile = 'face_to_match.jpg'
    fig.savefig(outputfile)

# 画像 2 からすべての顔を取得する
with open(image_2, mode="rb") as image_data:
    image_2_faces = face_client.face.detect_with_stream(image=image_data)
    image_2_face_ids = list(map(lambda face: face.face_id, image_2_faces))

    # 画像 1 に似ている画像 2 から顔を探す
    similar_faces = face_client.face.find_similar(face_id=image_1_face.face_id, face_ids=image_2_face_ids)
    similar_face_ids = list(map(lambda face: face.face_id, similar_faces))

    # 描画用に画像を準備する
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_2)
    draw = ImageDraw.Draw(image)

    # 一致する顔を描画して注釈をつける
    for face in image_2_faces:
        if face.face_id in similar_face_ids:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline='lightgreen', width=10)
            plt.annotate('Match!',(r.left, r.top + r.height + 15), backgroundcolor='white')

    # 注釈付き画像を保存する
    plt.imshow(image)
    outputfile = 'matched_faces.jpg'
    fig.savefig(outputfile)
```

3. **CompareFaces** 関数に追加したコードを調べます。画像 1 で最初の顔を見つけ、**face_to_match.jpg** という名前の新しい画像ファイルで注釈を付けます。次に、画像 2 のすべての顔を検索し、それらの顔 ID を使用して画像 1 に類似した顔を検索します。類似した顔に注釈が付けられ、**matched_faces.jpg** という名前の新しい画像に保存されます。
4. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します

**C#**

```
dotnet run
```

*C# 出力には、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これらは無視してかまいません。*

**Python**

```
python analyze-faces.py
```
    
5. プロンプトが表示されたら、**2** を入力して出力を確認します。次に、コードファイルと同じフォルダーに生成された **face_to_match.jpg** ファイルと **matched_faces.jpg** ファイルを表示して、一致した顔を確認します。
6. メニュー オプション **2** の **Main** 関数のコードを編集して **person2.jpg** と **people.jpg** を比較し、プログラムを再実行して結果を表示します。

## 顔認識モデルをトレーニングする

AI アプリケーションで顔を認識できる特定の人々のモデルを維持する必要があるシナリオがあるかもしれません。たとえば、顔認識を使用して安全な建物内の従業員の身元を確認する生体認証セキュリティシステムを促進するためです。

1. アプリケーションのコードファイルの **Main** 関数で、ユーザーがメニュー オプション **3** を選択した場合に実行されるコードを調べます。このコードは **TrainModel** 関数を呼び出し、Cognitive Services リソースに登録される新しい **PersonGroup** の名前と ID 、および従業員名のリストを渡します。
2. **face-api/images** フォルダーに、employees と同じ名前のフォルダーがあることを確認します。各フォルダにーは、指定された従業員の複数の画像が含まれています。
3. コード ファイルで **TrainModel** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します。

**C#**

```C
// グループが残っている場合は削除する
var groups = await faceClient.PersonGroup.ListAsync();
foreach(var group in groups)
{
    if (group.PersonGroupId == groupId)
    {
        await faceClient.PersonGroup.DeleteAsync(groupId);
    }
}

// グループを作成する
await faceClient.PersonGroup.CreateAsync(groupId, groupName);
Console.WriteLine("Group created!");

// 各ユーザーをグループに追加する
Console.Write("Adding people to the group...");
foreach(var personName in imageFolders)
{
    // 新しいユーザーを追加する
    var person = await faceClient.PersonGroupPerson.CreateAsync(groupId, personName);

    // 人物の複数の写真を追加する
    string[] images = Directory.GetFiles("images/" + personName);
    foreach(var image in images)
    {
        using (var imageData = File.OpenRead(image))
        { 
            await faceClient.PersonGroupPerson.AddFaceFromStreamAsync(groupId, person.PersonId, imageData);
        }
    }

}

    // モデルをトレーニングする
Console.WriteLine("Training model...");
await faceClient.PersonGroup.TrainAsync(groupId);

// グループ内のユーザー リストを取得する
Console.WriteLine("Facial recognition model trained with the following people:");
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    Console.WriteLine($"-{person.Name}");
}
```

**Python**

```Python
# グループが残っている場合は削除する
groups = face_client.person_group.list()
for group in groups:
    if group.person_group_id == group_id:
        face_client.person_group.delete(group_id)

# グループを作成する
face_client.person_group.create(group_id, group_name)
print ('Group created!')

# 各ユーザーをグループに追加する
print('Adding people to the group...')
for person_name in image_folders:
    # 新しいユーザーを追加する
    person = face_client.person_group_person.create(group_id, person_name)

    # 人物の複数の写真を追加する
    folder = os.path.join('images', person_name)
    person_pics = os.listdir(folder)
    for pic in person_pics:
        img_path = os.path.join(folder, pic)
        img_stream = open(img_path, "rb")
        face_client.person_group_person.add_face_from_stream(group_id, person.person_id, img_stream)

# モデルをトレーニングする
print('Training model...')
face_client.person_group.train(group_id)

# グループ内のユーザー リストを取得する
print('Facial recognition model trained with the following people:')
people = face_client.person_group_person.list(group_id)
for person in people:
    print('-', person.name)
```

4. **TrainModel** 関数に追加したコードを調べます。次のタスクを実行します。
    - サービスに登録されている **PersonGroup** のリストを取得し、指定されているものが存在する場合は削除します。
    - 指定された ID と名前でグループを作成します。
    - 指定した名前ごとにグループに人物を追加し、各人物の複数の画像を追加します。
    - グループ内の指名された人々とその顔画像に基づいて顔認識モデルをトレーニングします。
    - グループ内の名前付きの人のリストを取得して表示します。
5. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します

**C#**

```
dotnet run
```

*C# 出力には、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これらは無視してかまいません。*

**Python**

```
python analyze-faces.py
```

6. プロンプトが表示されたら、**3** と入力して出力を確認し、作成された **PersonGroup** に 2 人が含まれていることに注意してください。

## 顔の認識

**PeopleGroup** を定義し、顔認識モデルをトレーニングしたので、これを使用して、画像内の名前付きの個人を認識することができます。

1. アプリケーションのコードファイルの **Main** 関数で、ユーザーがメニュー オプション **4** を選択した場合に実行されるコードを調べます。このコードは **RecognizeFaces** 関数を呼び出し、画像ファイル (**people.jpg**) へのパスと、顔の識別に使用される **PeopleGroup** の ID を渡します。
2. コードファイルで **RecognizeFaces** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します

**C#**

```C
// 画像から顔を検出する
using (var imageData = File.OpenRead(imageFile))
{    
    var detectedFaces = await faceClient.Face.DetectWithStreamAsync(imageData);

    // 顔を取得する
    if (detectedFaces.Count > 0)
    {
        
        // 顔 ID の一覧を取得する
        var faceIds = detectedFaces.Select(f => f.FaceId).ToList<Guid?>();

        // ユーザー グループ内の顔を特定する
        var recognizedFaces = await faceClient.Face.IdentifyAsync(faceIds, groupId);

        // 認識された顔の名前を取得する
        var faceNames = new Dictionary<Guid?, string>();
        if (recognizedFaces.Count> 0)
        {
            foreach(var face in recognizedFaces)
            {
                var person = await faceClient.PersonGroupPerson.GetAsync(groupId, face.Candidates[0].PersonId);
                Console.WriteLine($"-{person.Name}");
                faceNames.Add(face.FaceId, person.Name);

            }
        }

        
        // 画像の中の顔に注釈をつける
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen penYes = new Pen(Color.LightGreen, 3);
        Pen penNo = new Pen(Color.Magenta, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Cyan);
        foreach (var face in detectedFaces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            if (faceNames.ContainsKey(face.FaceId))
            {
                // 顔が認識されると、名前に緑で注釈をつける
                graphics.DrawRectangle(penYes, rect);
                string personName = faceNames[face.FaceId];
                graphics.DrawString(personName,font,brush,r.Left, r.Top);
            }
            else
            {
                // それ以外の場合は、認識されない顔をマジェンタで注釈をつける
                graphics.DrawRectangle(penNo, rect);
            }
        }

        // 注釈付き画像を保存する
        String output_file = "recognized_faces.jpg";
        image.Save(output_file);
        Console.WriteLine("Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# 画像から顔を検出する
with open(image_file, mode="rb") as image_data:

    # 顔を取得する
    detected_faces = face_client.face.detect_with_stream(image=image_data)

    # 顔 ID の一覧を取得する
    face_ids = list(map(lambda face: face.face_id, detected_faces))

    # ユーザー グループ内の顔を特定する
    recognized_faces = face_client.face.identify(face_ids, group_id)

    # 認識された顔の名前を取得する
    face_names = {}
    if len(recognized_faces) > 0:
        print(len(recognized_faces), 'faces recognized.')
        for face in recognized_faces:
            person_name = face_client.person_group_person.get(group_id, face.candidates[0].person_id).name
            print('-', person_name)
            face_names[face.face_id] = person_name

    # 画像の中の顔に注釈をつける
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    for face in detected_faces:
        r = face.face_rectangle
        bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
        draw = ImageDraw.Draw(image)
        if face.face_id in face_names:
            # 顔が認識されると、名前に緑で注釈をつける
            draw.rectangle(bounding_box, outline='lightgreen', width=3)
            plt.annotate(face_names[face.face_id],
                        (r.left, r.top + r.height + 15), backgroundcolor='white')
        else:
            # それ以外の場合は、認識されない顔をマジェンタで注釈をつける
            draw.rectangle(bounding_box, outline='magenta', width=3)

    # 注釈付き画像を保存する
    plt.imshow(image)
    outputfile = 'recognized_faces.jpg'
    fig.savefig(outputfile)

    print('\nResults saved in', outputfile)
```
    
3. **RecognizeFaces** 関数に追加したコードを調べます。画像内の顔を見つけて、ID のリストを作成します。次に、people グループを使用して、顔 ID のリストで顔を識別しようとします。認識された顔には、識別された人物の名前が注釈として付けられ、結果は **recognized_faces.jpg** に保存されます。
4. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します

**C#**

```
dotnet run
```

*C# 出力には、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これらは無視してかまいません。*

**Python**

```
python analyze-faces.py
```

5. プロンプトが表示されたら、**4** を入力して出力を確認します。次に、コード ファイルと同じフォルダーに生成された **recognized_faces.jpg** ファイルを表示して、識別された人物を確認します。
6. メニューオプション **4** の **Main** 関数のコードを編集して **people2.jpg** の顔を認識し、プログラムを再実行して結果を表示します。同じ人が認識されるべきです。

## 顔の確認

顔認識は、身元の確認によく使用されます。Face サービスを使用すると、画像内の顔を別の顔と比較したり、**PersonGroup** に登録されている人物から確認したりできます。

1. アプリケーションのコードファイルの **Main** 関数で、ユーザーがメニュー オプション **5** を選択した場合に実行されるコードを調べます。このコードは **VerifyFace** 関数を呼び出し、画像ファイル (**person1.jpg**) へのパスと、顔の識別に使用される **PeopleGroup** の ID を渡します。
2. コード ファイルで **VerifyFace** 関数を見付け、コメント **「ユーザー グループからユーザー ID を取得する」** の下 (結果を出力するコードの上) に次のコードを追加します。

**C#**

```C
// ユーザー グループからユーザー ID を取得する
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    if (person.Name == personName)
    {
        Guid personId = person.PersonId;

        // 画像の最初の顔を取得する
        using (var imageData = File.OpenRead(personImage))
        {    
            var faces = await faceClient.Face.DetectWithStreamAsync(imageData);
            if (faces.Count > 0)
            {
                Guid faceId = (Guid)faces[0].FaceId;

                //顔と ID があります。一致しますか?
                var verification = await faceClient.Face.VerifyFaceToPersonAsync(faceId, personId, groupId);
                if (verification.IsIdentical)
                {
                    result = "Verified";
                }
            }
        }
    }
}
```

**Python**

```Python
# ユーザー グループからユーザー ID を取得する
people = face_client.person_group_person.list(group_id)
for person in people:
    if person.name == person_name:
        person_id = person.person_id

        # 画像の最初の顔を取得する
        with open(person_image, mode="rb") as image_data:
            faces = face_client.face.detect_with_stream(image=image_data)
            if len(faces) > 0:
                face_id = faces[0].face_id

                # 顔と ID があります。一致しますか?
                verification = face_client.face.verify_face_to_person(face_id, person_id, group_id)
                if verification.is_identical:
                    result = 'Verified'
```

3. **VerifyFace** 関数に追加したコードを調べます。指定された名前のグループ内の人の ID を検索します。人物が存在する場合、コードは画像の最初の顔の顔 ID を取得します。最後に、画像に顔がある場合、コードは指定された人物の ID に対して顔を検証します。
4. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します

**C#**

```
dotnet run
```

**Python**

```
python analyze-faces.py
```

5. プロンプトが表示されたら、**5** を入力して結果を確認します。
6. メニュー オプション **5** の **Main** 関数のコードを編集して、名前と画像 **person1.jpg** および **person2.jpg** のさまざまな組み合わせを試してみてください。

## 詳細情報

顔検出に **Computer Vision** サービスを使用する方法の詳細については、[Computer Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces) のドキュメントを参照してください。

**Face** サービスの詳細については、[Face のドキュメント](https://docs.microsoft.com/azure/cognitive-services/face/)を参照してください。
