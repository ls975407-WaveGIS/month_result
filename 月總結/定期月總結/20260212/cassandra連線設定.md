[back](../../20260212定期月總結.md)

+ Cassandra 的 Session == 連線池（connections pool）
+ (管理了連線池（connection pool），負責向不同節點分配連線和請求。)
+ 一條連線可同時處理多個 request
+ (驅動支援多路複用（multiplexing），一條 TCP 連線可以同時承載多個請求)


### 併發能力

```
nodes x max_connections_per_host × max_requests_per_connection
節點數 × 每節點最大連線數 × 每條連線最大請求數
```

- `nodes`：連線的 Cassandra 節點數 (會自己找有同步的DB)
- `max_connections_per_host`：每個節點的最大連線數
- `max_requests_per_connection`：每條連線最大同時請求數

### 設定

```yaml
cassandra:
  connection:
    # 每個 host 的最小連線數（保底）
    pool:
      local:
        size: 2            # core_connections_per_host：最少保持 2 條連線
        max: 8             # max_connections_per_host：最多 8 條連線

    # 每條連線最多 256 個 concurrent requests
    max-requests-per-connection: 256  
```


## 總結

|特色|Cassandra|Postgres + HikariCP + PgBouncer|
|---|---|---|
|Session 對應|連線池管理多條連線|一條連線 = Session|
|Multiplexing|支援，多個 request 可共用一條連線|不支援，一條連線只能一個 request|
|程式端連線池|每個節點一個 pool|HikariCP 連線池|
|服務端連線池|無，需要程式端管理|PgBouncer 可做共用連線池，減少 DB 負載|
|理論併發能力|nodes × max_connections × max_requests|max_pool_size × 1|