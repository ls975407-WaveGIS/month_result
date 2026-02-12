[back](../../20260212定期月總結.md)

+ Postgres 的 Session == 一個連線 (TCP 連線)
+ 一條連線只能處理一個 SQL request
+ 為了支援多併發，通常會配合連線池使用 (HikariCP)


## 程式端 - 連線池(HikariCP) 連 postgres

+ 一個連線池 = 多個連線
+ 一條連線只能處理一個 request
+ 連線池可視情況動態調整連線數量

### 併發能力

```
maximum-pool-size
連線池最大連線數
```

### 設定

```yaml
hikari:
  minimum-idle: 2                    # 最少保持 2 條空閒連線
  maximum-pool-size: 10              # 最多 10 條連線
  idle-timeout: 30000                # 空閒連線超過 30 秒會被回收
```


## 服務端 - PgBouncer 連 postgres

```
App1 ----\
          \
           PgBouncer ---- Postgres
          /
App2 ----/
```

- 多個程式或多個服務共用同一個 PgBouncer
- PgBouncer 和 Postgres 之間維持固定數量的連線
- PgBouncer 對外接收大量 client 連線，內部只維持少量 DB 連線



