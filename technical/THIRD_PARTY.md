[← 技術資料](README.md)

対象：v0.9.0。開発側の同名文書から生成した公開参照版。

# 第三者の構成物

このリポジトリは業務データを含みません。以下は実行・ビルド・表示に使用する第三者のプロジェクトです。各プロジェクトのライセンスと同梱通知を参照してください。

| 構成物 | 用途 | 出典 |
|---|---|---|
| Python 3.12 | 実行環境 | https://www.python.org/ |
| OR-Tools | CP-SAT・最大流 | https://github.com/google/or-tools |
| pywebview | Windows専用画面 | https://github.com/r0x0r/pywebview |
| Python.NET / clr-loader | .NET連携 | https://github.com/pythonnet/pythonnet / https://github.com/pythonnet/clr-loader |
| PyInstaller | 単一exeビルド | https://github.com/pyinstaller/pyinstaller |
| NSIS 3.13 | オフラインセットアップの作成 | https://nsis.sourceforge.io/ / コンパイラーZIP内のCOPYING |
| JSZip | 旧入力画面のOOXML読取 | prototypes/availability_intake/vendor/JSZip-LICENSE.md |
| WebView2 | Windowsの画面エンジン | https://developer.microsoft.com/microsoft-edge/webview2/ |
| NumPy / pandas / protobuf / absl-py 等 | OR-Tools等の依存 | requirementsの解決結果と各配布物の通知 |
| tzdata | WindowsのIANA時間帯 | https://pypi.org/project/tzdata/ |

依存の直接指定は requirements*.txt、実行時資産は komagumi/assets/ と gui/、JSZipのライセンスはvendor内にあります。実名を含む未使用の旧申込資産 application_base.xlsx はリポジトリと配布対象から除外しています。

GitHub Actionsの記法は公式の [checkout](https://github.com/actions/checkout) / [setup-python](https://github.com/actions/setup-python) を参照します。CIはWindows・Python 3.12・匿名テストを対象にします。

セットアップはMicrosoft署名付きWebView2 Evergreen Standalone Installerを再配布します。[Microsoftの配布手順](https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/distribution)に従い、未導入時だけ /silent /install で実行します。ビルド時に署名・構成物SHAを確認し、配布先のセキュリティ設定や共有ランタイムを変更・削除して導入制限を回避しません。
