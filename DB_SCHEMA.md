# DB_SCHEMA.md

## 目的
このアプリは、Windows上の作業状態を自動記録し、勤務時間、サポート対応時間、会議時間、その他時間を集計する。
DBには、時系列ログ、ケース履歴、設定値、日報生成結果を保存する。

## 使用DB
SQLite

## テーブル一覧
- settings
- activity_logs
- case_history
- case_sessions
- daily_reports

---

## 1. settings
アプリ設定を保存する。

CREATE TABLE IF NOT EXISTS settings (
    key         TEXT PRIMARY KEY,
    value       TEXT NOT NULL,
    updated_at  TEXT NOT NULL
);

初期値：
INSERT OR IGNORE INTO settings VALUES
('check_interval_sec', '30', datetime('now')),
('idle_threshold_min', '5', datetime('now')),
('case_grace_period_min', '5', datetime('now')),
('llm_enabled', 'true', datetime('now')),
('screenshot_save_enabled', 'false', datetime('now'));

---

## 2. activity_logs
作業状態の時系列ログを保存する。

CREATE TABLE IF NOT EXISTS activity_logs (
    id               INTEGER PRIMARY KEY AUTOINCREMENT,
    start_time       TEXT    NOT NULL,
    end_time         TEXT,
    category         TEXT    NOT NULL,
    app_name         TEXT,
    window_title     TEXT,
    case_id          TEXT,
    confidence       INTEGER DEFAULT 0,
    detection_source TEXT    NOT NULL,
    llm_summary      TEXT,
    created_at       TEXT    NOT NULL
);

categoryの値：support / meeting / other / unknown / idle
detection_sourceの値：rule / manual / llm / system

---

## 3. case_history
Salesforce検知時のケース候補に使う。

CREATE TABLE IF NOT EXISTS case_history (
    case_id      TEXT PRIMARY KEY,
    last_used_at TEXT NOT NULL,
    title_hint   TEXT,
    use_count    INTEGER DEFAULT 1
);

---

## 4. case_sessions
ケース単位の作業セッションを保存する。

CREATE TABLE IF NOT EXISTS case_sessions (
    id         INTEGER PRIMARY KEY AUTOINCREMENT,
    case_id    TEXT    NOT NULL,
    start_time TEXT    NOT NULL,
    end_time   TEXT,
    status     TEXT    NOT NULL,
    note       TEXT,
    created_at TEXT    NOT NULL
);

statusの値：active / paused / closed

---

## 5. daily_reports
日次集計とLLM生成日報を保存する。

CREATE TABLE IF NOT EXISTS daily_reports (
    report_date          TEXT    PRIMARY KEY,
    total_work_minutes   INTEGER DEFAULT 0,
    support_minutes      INTEGER DEFAULT 0,
    meeting_minutes      INTEGER DEFAULT 0,
    other_minutes        INTEGER DEFAULT 0,
    idle_minutes         INTEGER DEFAULT 0,
    report_text          TEXT,
    created_at           TEXT    NOT NULL
);

---

## インデックス

CREATE INDEX IF NOT EXISTS idx_activity_logs_time ON activity_logs(start_time, end_time);
CREATE INDEX IF NOT EXISTS idx_activity_logs_category ON activity_logs(category);
CREATE INDEX IF NOT EXISTS idx_activity_logs_case_id ON activity_logs(case_id);
CREATE INDEX IF NOT EXISTS idx_case_sessions_case_id ON case_sessions(case_id);

---

## 実装方針
初期実装では以下を優先する。
- settings
- activity_logs
- case_history

case_sessionsとdaily_reportsは、基本ログ保存が安定してから実装する。
