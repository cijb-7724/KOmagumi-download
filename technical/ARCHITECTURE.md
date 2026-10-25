[← 技術資料](README.md)

対象：v0.9.0。開発側の同名文書から生成した公開参照版。

# 実装の構成

Windows x64、Python 3.12、OR-Tools CP-SAT、pywebview + WebView2。ローカルHTML/JavaScriptを専用画面で使う。SQLiteや外部サーバーは使わない。

~~~mermaid
flowchart TD
  Desktop["desktop.py / 単一exe"] --> API["web_app.py / localhost API"]
  GUI["gui + prototypes / 可否入力と推移"] <--> API
  API <--> Store["work_store.py / 保存・版・Undo"]
  Store <--> Doc["documents.py + courses.py / .komagumi"]
  API --> Service["service.py / 検証→計算→出力"]
  Excel["時間割2冊・申込Excel"] --> Reader["ooxml / reader / bundle / inputs"]
  Service --> Reader
  Reader --> Problem["problem.py / 候補・残数・独立検査"]
  Problem --> Solver["solver.py / CP-SAT"]
  Scores["objectives + preference_scores"] <--> Solver
  Solver --> Writer["writer / reports / excel_templates"]
  Writer --> Output["追記した時間割2冊・レポート"]
~~~

## 変更の入口

| 変更 | 主なファイル | 最初に確認するテスト |
|---|---|---|
| 帳票・セル契約 | ooxml.py, reader.py, bundle.py | test_import, test_input_set, test_synthetic_pipeline |
| 消化・期間・可否 | inputs.py, course_settings.py, schedule_rules.py | test_grade_window, test_browser_rules |
| 候補・制約・上界 | problem.py, classroom.py, solver.py | test_solver, test_classroom, test_soft_objectives |
| 数値評価 | objectives.py, preference_scores.py | test_combined_schedule, test_fair_scoring, test_editable_weights |
| 原本保持・出力 | writer.py, reports.py, excel_templates.py | test_synthetic_pipeline, test_excel_templates |
| 可否UI | prototypes/availability_intake, teacher_availability, gui/bridge.js | Node core/bridge、実機で両タブ |
| 保存・再開 | work_store.py, courses.py, documents.py | test_course_safety, test_results_store、匿名往復 |
| 停止・結果保持 | web_app.py, desktop.py, results_store.py, result_cache.py | test_job_lifecycle, test_web_app, test_result_cache |
| 配布 | tools/build_windows.py, desktop_entry.py | 単一exeの起動・短い計算・保存・再起動 |

prototypes/ は現在の入力画面の正本。別実装を新設して二重に同期させない。gui/bridge.js が入力モデルと親画面APIを接続する。生徒・講師・教室の可否は、紙を見てセル/範囲へ直接入力する。

## 保存の境界

| 保存先 | 内容 |
|---|---|
| 外部 .komagumi | 可否、人物/日時対応、既存欠席、メモ、設定、直近10操作。SHA付きコンテナであり暗号化ではない |
| 入力Excel | 名簿・日時・既存授業、担当/数量/PS1の正本 |
| AppData/KOmagumi/courses | 作業コピー、Excelコピー、最新結果、安全コピー。内部JSONを利用者に操作させない |
| 出力Excel | 時間割2冊とレポート。講習ファイルへ埋め込まない |

WorkStore はrevision照合、RLock、一時ファイル→置換を使う。講習のOSロックで二重編集を拒否する。切替ではAPIトークンを更新し、古い画面からの書込を拒否する。

documents.py は文書の直前指紋を照合して自動保存する。移動・削除・外部変更なら上書きせず、内部入力を残して別名保存できる。終了は入力保存キューとsolverの最終保存を待つ。WebViewのPromise callback上でworkerをjoinしない。

## 計算から出力まで

1. 現在のrevisionから実行専用入力コピーを作る。
2. 読取・既存集計・候補生成・CP-SATで同じデータを使う。
3. 実解の配置数・得点・9項目を通知し、最良を定期保持する。
4. validate_additions で残数、同時刻、日/連続、PS1、ブース等を独立再検証する。
5. 空欄セルへinline文字列を追記し、再計算設定だけを更新する。他のOOXML部品・VBA・数式・書式は保持する。
6. 出力を読み直して既存/新規を照合し、成功後に公開する。

完成結果は入力設定と独立。秒数変更・入力確認・次の計算や失敗で失わない。新しい正常結果ができてから最新を更新する。新規登録の内部結果/チェックポイントは最新2回まで。未登録・手動変更・外部結果を自動整理しない。

## 単一exe

PyInstaller onefileがPython・ライブラリ・画面・VBA資産を内包し、起動時に一時領域へ展開する。ユーザーデータはそこへ置かず、AppDataと選択文書を使う。コード資産は __file__ 基準で参照する。

Pythonの別インストールは不要。WebView2はOS側に必要であり、Python同梱だけで未導入PCの条件を満たしたことにはならない。会社PC/オフライン受入状況は開発状況を見る。

業務変更は要件本文と付属定義へ反映する。仕様・実装・テスト成功・実機受入・最適性証明を区別する。実績時間割を解のヒントとしてsolverへ渡さない。
