# WORKTIME_TRACKER_CODEX_PACKAGE.md

# Project Name
Worktime Tracker

---

# Overview
Windows専用のローカル作業時間トラッカー。
PC上の作業状態を自動記録し、勤務時間、会議時間、Salesforce対応時間、その他時間を集計する。

主目的：
- 勤務時間の自動記録
- 工数入力補助
- 日報作成補助
- Salesforceケース別の対応時間把握

---

# Core Policy
- Windows専用
- Python実装
- SQLite使用
- ローカル実行
- 初期版はGUIなし
- 初期版はLLMなし
- スクリーンショット保存は標準OFF
- 他人監視機能は実装しない
- 小さく段階実装する

---

# Categories
- support
- meeting
- other
- unknown
- idle

---

# Priority Order
1. manual
2. meeting
3. support
4. other
5. idle

---

# Detection Rules

## Meeting
Use: app name, window title, microphone active

Apps: Teams, Zoom, Google Meet, Webex
Titles: Meeting, 会議, 通話中, 発表, 参加者
Score: Apps +50, Title match +30, Mic active +40
Threshold: 70 points

## Support
Initial target: Salesforce

Keywords: Salesforce, Case, SR, Service Request, Account, Contact, salesforce.com, lightning.force.com
Score: Salesforce screen +70, Title match +20, Input active +20
Threshold: 70 points

---

# Case Tracking
When Salesforce is detected: Show case selection popup.

Candidate sources:
1. recent case history
2. title history
3. screenshot OCR/LLM (fallback only)

Do not auto-confirm case id. Always require user confirmation.

---

# Monitoring Cycle
- event-driven if state changed
- fallback check every 30 sec (configurable)

---

# Idle Rule
If no keyboard/mouse input for 5 min: category = idle

---

# Logging Data
Store in activity_logs:
- start_time, end_time
- category
- app_name, window_title
- case_id
- confidence, detection_source
- llm_summary

---

# Reporting
Daily report:
- total work time
- support time per case
- meeting time
- other time, idle time
- natural language summary
- CSV export

---

# SQLite Schema

## settings
CREATE TABLE IF NOT EXISTS settings (
    key TEXT PRIMARY KEY,
    value TEXT NOT NULL,
    updated_at TEXT NOT NULL
);

Initial values:
- check_interval_sec = 30
- idle_threshold_min = 5
- case_grace_period_min = 5
- llm_enabled = true
- screenshot_save_enabled = false

## activity_logs
CREATE TABLE IF NOT EXISTS activity_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    start_time TEXT NOT NULL,
    end_time TEXT,
    category TEXT NOT NULL,
    app_name TEXT,
    window_title TEXT,
    case_id TEXT,
    confidence INTEGER DEFAULT 0,
    detection_source TEXT NOT NULL,
    llm_summary TEXT,
    created_at TEXT NOT NULL
);

## case_history
CREATE TABLE IF NOT EXISTS case_history (
    case_id TEXT PRIMARY KEY,
    last_used_at TEXT NOT NULL,
    title_hint TEXT,
    use_count INTEGER DEFAULT 1
);

---

# Recommended Structure
worktime_tracker/
  app/
    main.py
    config.py
    db/       (database.py, schema.py)
    tracker/  (monitor.py, idle.py, active_window.py)
    rules/    (classifier.py, meeting.py, support.py)
    ui/       (tray.py, popup.py)
    integrations/ (ollama_client.py)
    reports/  (exporter.py, generator.py)
  tests/
  data/

---

# requirements.txt
pywin32
pynput

---

# Implementation Roadmap
Phase 1: DB init, settings, activity_logs, active window, monitor loop, idle
Phase 2: meeting rule, support rule
Phase 3: case tracking
Phase 4: tray UI
Phase 5: reports + CSV
Phase 6: local LLM (Ollama)

---

# Coding Rules
- use logging
- use type hints
- keep functions small
- add exception handling
- separate modules by responsibility
- do not hardcode secrets
- do not require cloud APIs
- screenshot save must be OFF by default

---

# First Task For Codex

Implement Phase 1 initial version.

Create or update:
- app/main.py
- app/db/database.py
- app/db/schema.py

Requirements:
1. Create data/worktime_tracker.sqlite3 (auto-create data/ folder if missing)
2. Create settings table and insert defaults
3. Create activity_logs table
4. Initialize on startup
5. Log success/failure with Python logging
6. Use type hints throughout
7. Add exception handling

Runnable by:
  python app\main.py

Completion criteria:
- no startup error
- db file created at data/worktime_tracker.sqlite3
- tables created
- default settings inserted
- success log shown in console

NOT yet in scope:
- GUI, tray, LLM, screenshot, Salesforce login, CSV output, meeting detection, case popup
