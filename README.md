# Worktime Tracker

Windows専用のローカル作業時間トラッカーです。
アクティブウィンドウ、入力状態、会議状態、Salesforce作業状態を記録し、勤務時間・会議時間・サポート対応時間を集計します。

## 目的
- 勤務時間の自動記録
- サポートケース別の対応時間集計
- 会議時間の集計
- 日報作成の補助
- 工数入力の補助

## 主な特徴
- Windows専用
- ローカル実行（クラウド不要）
- SQLite保存
- Salesforce作業検知
- Teams/Zoomなどの会議判定
- タスクトレイ常駐予定
- 標準ではスクリーンショットを保存しない
- ローカルLLM連携は将来対応

## 初期MVP
最初の実装範囲：
- SQLite初期化・設定保存
- アクティブウィンドウ取得・入力有無の検知
- idle判定・activity_logs保存
- コンソールログ出力

## 使用技術
- Python 3.11以降
- SQLite
- pywin32
- pynput

## 推奨環境
- Windows 10 / 11

## セットアップ
```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
python app\main.py
```

## ディレクトリ構成
```
worktime_tracker/
  app/
    main.py
    config.py
    tracker/   (monitor.py, idle.py, active_window.py)
    rules/     (classifier.py, meeting.py, support.py)
    db/        (database.py, schema.py, repositories.py)
    ui/        (tray.py, popup.py, settings.py, reports.py)
    integrations/ (ollama_client.py)
    reports/   (exporter.py, generator.py)
  tests/
  data/
  requirements.txt
```

## 仕様書
詳細仕様は以下を参照してください。
- SPEC.md
- DB_SCHEMA.md
- IMPLEMENTATION_PLAN.md
- AGENTS.md

## 注意事項
このアプリは本人の業務時間記録・日報補助を目的とします。
他人の監視や無断記録を目的とした利用は想定していません。

## プライバシー方針
- 画面キャプチャは標準では保存しません
- Salesforceの認証情報は保存しません
- 保存データはローカルSQLiteに限定します
- LLM連携はローカルLLMを前提とします
