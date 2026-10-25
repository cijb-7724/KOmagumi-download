# 開発・検証の入口

[← 技術資料](README.md) · [確認状況](STATUS.md)

ソースへのアクセス権がある開発者向け。公開配布リポジトリ単体には、以下のコードやテストは含まれない。

## 環境

Windows x64、Python 3.12 x64。開発リポジトリで次を実行する。利用者のインストール作業ではない。

```powershell
py -3.12 -m venv .venv
.venv\Scripts\python.exe -X utf8 -m pip install -r requirements-desktop.txt
.venv\Scripts\python.exe -X utf8 tools/run_tests.py
node --test prototypes/availability_intake/test_core.cjs prototypes/teacher_availability/test_core.cjs tests/test_bridge.cjs tests/search_charts.test.cjs tests/weight_controls.test.cjs
```

`run_tests.py` は実データ依存クラスを明示除外する。業務ファイルを公開してテストを成立させない。

## 意味のある対象検証

| 変更 | 確認 |
|---|---|
| 制約・上界 | 小問題、既存例外、PS1、残数、生徒日/連続、ブース、独立検証 |
| 評価 | 既存＋新規の9項目、固定分母、整数floor、重み0、配置1コマ優先 |
| 保存 | .komagumi往復、Excel再接続、10操作、外部変更拒否、正常終了 |
| 結果保持 | 秒数変更・入力確認・新計算失敗・再起動で旧結果を保持 |
| Excel | 原本セル/部品の保持、申込・PS1、名簿変更と数量追従 |
| 配布 | exe単体の起動、短い計算、停止、Excel保存、再起動、更新と削除時の講習保持 |

匿名ケースは `tests/support/workbooks.py` と `synthetic.py` が物理契約からExcelを新規生成する。実名帳票を匿名化したコピーではない。`test_synthetic_pipeline.py` が2生徒・2講師・既存授業・申込・PS1・文書往復・計算・出力を確認する。

公開する [デモ一式](../user/demo.md)は、画面写真と同じ4日間・2生徒・2講師・通常1・申込3のファイル。開発側の `test_public_demo.py` は配布ZIPそのものを解凍し、別ライブラリへ .komagumi を取り込み、○×を保持したExcel再接続、PS1読取、短い計算、通常授業と入力原本の保護、Excel3冊の出力を確認する。デモ以外のExcel・講習・画像がソースの許可リストへ混入しないことも確認する。

探索の動作確認は数秒〜10秒。最適性や高得点を得るために試験を延長しない。成功済み全体を同じ理由で繰り返さず、変更対象を確認する。

## 配布

開発側の `tools/build_windows.py` がonefileのKOmagumi.exeを生成し、`tools/build_setup.py --fetch` が公式WebView2オフライン導入を内包したKOmagumiSetup.exeを生成する。既存の版は上書きせず新しいリリース番号を使う。

公開前は、許可した文書・資産の一覧、実名/機密情報、アーカイブ内の実資産、配布SHAを確認する。利用者のExcel、講習、内部作業コピー、実験ログを同梱しない。

## 実機でしか確認できないもの

会社Excelでのマクロ許可・名簿編集・表示・数式、Python未導入/オフラインPC、WebView2未導入時の実導入、会社の実行制限、非技術者による日常操作。匿名CI成功で受入済みにはしない。
