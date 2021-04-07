---
lab:
    title: 'ラボ環境のセットアップ'
    module: 'セットアップ'
---

# ラボ環境のセットアップ

これらの演習は、ホストされたラボ環境で完了するように設計されています。ただし、ご自分のパソコンで完成させたい場合は、以下のソフトウェアをインストールしてください。独自の環境を使用すると、予期しないダイアログや動作が発生する場合があります。

> **注**: 以下の手順は、Windows 10 コンピューター用です。Linux または MacOS を使用することもできます。

### 基本オペレーティング システム (Windows 10)

#### Windows 10

Windows 10 をインストールし、すべての更新プログラムを適用します。

#### Edge

[Edge (Chromium)](https://microsoft.com/edge) をインストールします

### .NET Core SDK

1. https://dotnet.microsoft.com/download からダウンロードしてインストールします (ランタイムだけでなく、.NET Core SDK をダウンロードします)

### C++ 再頒布可能パッケージ

1. Visual C++ 再頒布可能パッケージ (x64) を https://aka.ms/vs/16/release/vc_redist.x64.exe からダウンロードしてインストールします。

### Node.JS

1. https://nodejs.org/en/download/ から最新の LTS バージョンをダウンロードします 
2. 既定ののオプションを使用してインストールします

### Python (および必要なパッケージ)

1. https://docs.conda.io/en/latest/miniconda.html からバージョン 3.8 をダウンロードします 
2. セットアップを実行してインストールします - **重要**: Miniconda を PATH 変数に追加し、Miniconda をデフォルトの Python 環境として登録するオプションを選択します。
3. インストール後、Anaconda プロンプトを開き、次のコマンドを入力してパッケージをインストールします。 

```
pip install flask requests python-dotenv pylint matplotlib pillow
pip install --upgrade numpy
```

### Azure CLI

1. https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli?view=azure-cli-latest からダウンロードします 
2. 既定ののオプションを使用してインストールします

### Git

1. 既定のオプションを使用して、https://git-scm.com/download.html からダウンロードしてインストールします


### Visual Studio Code (および拡張機能)

1. https://code.visualstudio.com/Download からダウンロードします 
2. 既定ののオプションを使用してインストールします 
3. インストール後、Visual Studio Code を起動し、**「拡張機能」** タブ (CTRL+SHIFT+X) で、Microsoft から次の拡張機能を検索してインストールします。
    - Python
    - C#
    - Azure Functions
    - PowerShell


### Bot Framework Emulator

https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md の指示に従って、オペレーティング システム用の Bot Framework Emulator の最新の安定したバージョンをダウンロードしてインストールします。

### Bot Framework Composer

https://docs.microsoft.com/ja-jp/composer/install-composer からインストールします。
