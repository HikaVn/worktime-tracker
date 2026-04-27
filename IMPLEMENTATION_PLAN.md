# IMPLEMENTATION_PLAN.md

## 目的
Worktime Tracker を小さく確実に実装する。
一気に完成版を作らず、MVP -> 安定化 -> 拡張の順で進める。

## Phase 1 : Core Logging MVP
最初に、作業状態を記録できる最小構成を作る。

### 実装項目
- SQLite初期化
- settingsテーブル初期値投入
- activity_logs保存
- アクティブウィンドウ取得
- アプリ名・ウィンドウタイトル取得
- 監視ループ（状態変化 + 30秒補完）
- idle判定
- コンソールログ出力

### 完了条件
- DBファイルが自動作成される
- activity_logsへ自動保存される
- ウィンドウ切替で新規ログが作られる
- 5分無操作でidle記録される

## Phase 2 : Rule Engine

### 実装項目
- meeting判定（Teams/Zoom/マイク状態）
- support判定（Salesforce）
- スコア判定ロジック
- confidence / detection_source保存

### 完了条件
- Teams会議でmeetingになる
- Salesforce画面でsupportになる
- 通常作業でotherになる

## Phase 3 : Case Tracking

### 実装項目
- case_history
- ケース選択ポップアップ
- ケース継続猶予（5分）
- 会議終了後のケース復帰

### 完了条件
- Salesforce検知時に候補表示される
- case_id付きログ保存される
- 5分以内の離脱でケース継続する

## Phase 4 : Tray UI

### 実装項目
- タスクトレイアイコン
- 現在状態表示
- 手動カテゴリ変更・一時停止
- 設定画面・日報画面起動

### 完了条件
- 常駐動作する
- UIから停止・再開できる

## Phase 5 : Reporting

### 実装項目
- 日次集計
- daily_reports保存
- CSV出力
- 自然文日報生成

### 完了条件
- 当日の集計表示
- CSV出力成功

## Phase 6 : Local LLM Integration

### 実装項目
- Ollama連携
- 曖昧分類
- ケース番号候補抽出
- 日報文改善

### 完了条件
- ON/OFF切替可能
- LLMなしでも動作する

---

## 推奨リポジトリ構成

worktime_tracker/
  app/
    main.py
    config.py
    tracker/  (monitor.py, idle.py, active_window.py)
    rules/    (classifier.py, meeting.py, support.py)
    db/       (database.py, schema.py, repositories.py)
    ui/       (tray.py, popup.py, settings.py, reports.py)
    integrations/ (ollama_client.py)
    reports/  (exporter.py, generator.py)
  tests/
  requirements.txt  README.md  SPEC.md  DB_SCHEMA.md
  IMPLEMENTATION_PLAN.md  AGENTS.md

---

## Codexへの依頼単位
大きく投げず、小さく依頼する。

良い例：
- database.pyを作成
- activity_logs保存関数を追加
- meeting判定関数を実装
- CSV出力機能を追加

悪い例：全部作って、完成版を一括実装して

---

## 最初の実装依頼
1. SQLite初期化（database.py, schema.py）
2. activity_logs保存
3. active_window.py（pywin32）
4. 監視ループ
5. meeting/support判定
