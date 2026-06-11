# 資料庫資料表結構說明


---

## 1 使用者管理模組

<p align="center"><b>表 C-1：users（使用者資料表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 使用者唯一識別碼（Primary Key，自動遞增） |
| username | TEXT | 使用者登入帳號（Unique） |
| password | TEXT | 使用者加密密碼（Hash 雜湊值） |
| registered_at | TEXT | 帳號註冊時間（ISO 8601 格式字符串） |

<br>

<p align="center"><b>表 C-2：login_logs（登入紀錄表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 紀錄唯一識別碼（Primary Key） |
| user_id | INTEGER | 使用者 ID（Foreign Key，對應 users.id） |
| login_time | TEXT | 使用者登入時間 |
| logout_time | TEXT | 使用者登出時間（若異常離線則可為空） |

---

## 2 感測器資料模組

<p align="center"><b>表 C-3：sensor_data（感測器資料表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 資料唯一識別碼（Primary Key） |
| pool_id | INTEGER | 養殖池塘/水族箱編號 |
| temp | REAL | 實測水溫（攝氏度 °C） |
| psu | REAL | 實測鹽度（PSU） |
| ph | REAL | 實測酸鹼值（pH） |
| do | REAL | 實測水中溶氧量（mg/L） |
| orp | REAL | 實測氧化還原電位（mV） |
| timestamp | TEXT | 邊緣端感測數據採集時間 |

<br>

<p align="center"><b>表 C-4：threshold_settings（閾值設定表，原 oddset）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 設定唯一識別碼（Primary Key） |
| pool_id | INTEGER | 對應之養殖池塘編號 |
| temp_min | REAL | 水溫安全下限值 |
| temp_max | REAL | 水溫安全上限值 |
| ph_min | REAL | pH 酸鹼值安全下限值 |
| ph_max | REAL | pH 酸鹼值安全上限值 |
| do_min | REAL | 溶氧量安全下限值 |
| do_max | REAL | 溶氧量安全上限值 |
| psu_min | REAL | 鹽度安全下限值（推理補齊） |
| psu_max | REAL | 鹽度安全上限值（推理補齊） |
| orp_min | REAL | 氧化還原電位安全下限值（推理補齊） |
| orp_max | REAL | 氧化還原電位安全上限值（推理補齊） |

---

## 3  TimesFM預測與資料品質模組

<p align="center"><b>表 C-5：predicted_data（AI 預測資料表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 預測唯一識別碼（Primary Key） |
| pool_id | INTEGER | 對應之養殖池塘編號 |
| forecast_time | TEXT | 預期目標時間點（未來預測時間） |
| temp_pred | REAL | TimesFM 模型預測水溫值 |
| ph_pred | REAL | TimesFM 模型預測 pH 值 |
| do_pred | REAL | TimesFM 模型預測溶氧量 |
| orp_pred | REAL | TimesFM 模型預測氧化還原電位 |
| psu_pred | REAL | TimesFM 模型預測鹽度值 |

<br>

<p align="center"><b>表 C-6：data_issues（資料品質檢測表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 檢測紀錄唯一識別碼（Primary Key） |
| pool_id | INTEGER | 對應之養殖池塘編號 |
| timestamp | TEXT | 資料品質檢測執行時間點 |
| fetch_failed | INTEGER | 邊緣端資料抓取失敗旗標（0: 正常, 1: 失敗） |
| history_missing | INTEGER | 歷程歷史資料缺失旗標（0: 完備, 1: 遺失） |
| prediction_missing | INTEGER | AI 預測排程中斷或缺失旗標（0: 正常, 1: 缺失） |
| anomaly | INTEGER | 整體資料流是否判定為嚴重異常 |

---

## 4 警報與通知模組

<p align="center"><b>表 C-7：alerts（警報資料表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 警報唯一識別碼（Primary Key） |
| pool_id | INTEGER | 發生異常之養殖池塘編號 |
| sensor_type | TEXT | 觸發警報之感測器指標類型（如 temp, ph, do） |
| value | REAL | 觸發當時之實際感測數值 |
| status | TEXT | 處理狀態（如：'unhandled' 未處理, 'resolved' 已處理） |
| is_notified | INTEGER | 外部推播狀態（0: 未通知, 1: 已成功傳送 Discord） |
| consecutive_count | INTEGER | 該指標連續超過閾值之次數（用於防誤報機制） |
| timestamp | TEXT | 警報首次觸發時間點 |

<br>

<p align="center"><b>表 C-8：system_alert_logs（系統警報紀錄表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 日誌唯一識別碼（Primary Key） |
| event_type | TEXT | 系統事件類型（如：'Hardware_Disconnect', 'Database_Error'） |
| description | TEXT | 錯誤日誌詳細描述資訊 |
| start_time | TEXT | 系統異常事件起始時間 |
| end_time | TEXT | 系統異常事件恢復/結束時間（未結束則為空） |

<br>

<p align="center"><b>表 C-9：notification_settings（通知設定表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 設定識別碼（Primary Key） |
| user_id | INTEGER | 關聯之管理員 ID（Foreign Key） |
| enabled | INTEGER | 全域通知開關（0: 停用, 1: 啟用） |
| cooldown_minutes | INTEGER | 警報重複發送之冷卻時間（分鐘，避免通知轟炸） |

---

## 5 行為分析模組

<p align="center"><b>表 C-10：activity（活動分析表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 分析唯一識別碼（Primary Key） |
| pool_id | INTEGER | 觀測之養殖池塘編號 |
| timestamp | TEXT | 行為特徵分析時間戳記 |
| active_ratio | REAL | 畫面中活動蝦隻數量之比例（0.0 ~ 1.0） |
| p50_speed | REAL | 蝦群移動速度之中位數（50百分位數，像素/秒） |
| p90_speed | REAL | 蝦群高速行為指標（90百分位數，用於判別活躍度） |
| burst_events | INTEGER | 統計時間區間內觸發之「暴衝行為」總次數 |

<br>

<p align="center"><b>表 C-11：density（密度分析表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 密度分析唯一識別碼（Primary Key） |
| pool_id | INTEGER | 觀測之養殖池塘編號 |
| timestamp | TEXT | 密度量測時間戳記 |
| mean_occ | REAL | 蝦隻實例分割遮罩之平均畫面佔有率（%） |
| max_occ | REAL | 統計區間內最大畫面佔有率（%） |
| total_unique_ids | INTEGER | 經軌跡追蹤校正後之視野內個體總隻數（Count） |

<br>

<p align="center"><b>表 C-12：length_hist（長度分析表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 長度量測唯一識別碼（Primary Key） |
| pool_id | INTEGER | 觀測之養殖池塘編號 |
| timestamp | TEXT | 長度估算時間戳記 |
| min_len | REAL | 經骨架細化量測出之最小個體長度（cm） |
| max_len | REAL | 經骨架細化量測出之最大個體長度（cm） |
| counts_json | TEXT | 體長區間分布統計（JSON 格式，如 `{"1-2cm": 5, "2-3cm": 12}`） |

<br>

<p align="center"><b>表 C-13：metrics（綜合分析表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 綜合分析流水號（Primary Key） |
| pool_id | INTEGER | 觀測之養殖池塘編號 |
| timestamp | TEXT | 資料統整與報表生成時間點 |
| avg_active_ratio | REAL | 整合週期內之平均活動比例 |
| avg_speed | REAL | 整合週期內蝦隻平均移動速度 |
| total_bursts | INTEGER | 整合週期內累積之暴衝次數總計 |
| final_shrimp_count | INTEGER | 當前週期校正後之最終個體總數 |
| estimated_biomass | REAL | 基於平均體長與隻數推估之總生物量估計值 |

---

## 6 系統設定模組

<p align="center"><b>表 C-14：system_settings（系統設定表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| key | TEXT | 全域系統設定項目名稱（Unique Index，如 'api_url'） |
| value | TEXT | 設定項目對應之具體數值或配置文字 |

<br>

<p align="center"><b>表 C-15：pool_settings（池塘設定表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| pool_id | INTEGER | 養殖池塘編號（Primary Key） |
| settings | TEXT | 該池專屬之進階硬體與排程參數（JSON 結構文字） |

<br>

<p align="center"><b>表 C-16：perf_metrics（效能監控表）</b></p>

| 欄位名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| id | INTEGER | 效能流水號（Primary Key） |
| metric_name | TEXT | 系統資源指標名稱（如 'CPU_Usage', 'DB_Query_Time'） |
| value | REAL | 效能觀測實測數值 |
| timestamp | TEXT | 效能監控資料記錄時間 |

---

