
[back](../../20251117定期月總結.md)


## Hive 風格

**大數據存儲系統**（例如 Hive 或 Spark）中，經常會使用 **目錄結構來分區資料**

/data/table_name/
    country=US/year=2023/month=10/
    country=CN/year=2023/month=10/
每個分區的目錄名稱遵循 key=value 格式。
分區鍵 (key) 可以是多個，例如 country, year, month。
自動從目錄結構中識別分區信息，而無需在資料中存儲這些字段。

## 資料切割

```txt
data/
 ├── water_st/
 │   ├── station_id=TCRD001/
 │   │   ├── date=2025-10/               # 歷史月檔，已合併完成
 │   │   │   └── TCRD001_2025-10.parquet
 │   │   ├── date=2025-11/               # 當月合併檔（可更新）
 │   │   │   └── TCRD001_2025-11.parquet
 │   ├── station_id=TCRD002/
 │   │   └── ...
```

```sql
SELECT *
FROM 'data/water_st/station_id=TCRD001/date=*/ST*.parquet'
WHERE date >= substr('2025-11-01',1,7)
  AND date <= substr('2025-11-17',1,7)
  AND datatime >= '2025-11-01 00:00:00'
  AND datatime <= '2025-11-17 23:59:59'
```


## 資料小於十萬筆以下無腦選擇SQLite

諸如關連表、設定檔，即使需要客制欄位，也可以用json格式，資料筆數在這種量級，兩種資料庫速度一樣

特殊情況需要升級

+ 多人同時頻繁寫入 => PostgreSQL
+ 高併發 API 查詢  => PostgreSQL
+ 單筆資料很大(MB JSON) => DuckDB 或 PostgreSQL JSONB

只有在資料筆數很多發現用SQLite很慢，才需要考慮其他種資料庫
