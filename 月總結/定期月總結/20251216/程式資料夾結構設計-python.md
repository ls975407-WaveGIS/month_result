
[back](../../20251216定期月總結.md)

```
main.py
app/
  ├── fastapi_app.py
  ├── api/
  ├── model/po
  ├── model/vo
  ├── service/
  └── util/
alembic/
```

```shell
D:.
├── main.py # 本地測試用
├── app/
│   ├── fastapi_app.py      # FastAPI 實例
│   ├── api/                # 路由模組
│   │   ├── central_api.py
│   │   └── flood_api.py
│   ├── model/              
│   │   ├── po/             # Persistence Object (DB ORM)
│   │   │   └── general_1st.py
│   │   └── vo/             # Value Object (DTO)
│   │       └── general_1st.py
│   ├── service/            # 業務邏輯
│   │   └── receive_data.py
│   └── util/               # 工具函數、例外、設定
│       ├── config.py
│       ├── exception.py
│       └── tool.py
├── alembic/                # DB migration
│   └── versions/
│       └── a9deb3ef3815_create_general_1st_table.py
```

## 專案持續增長

**API 分模組**
- 當路由增多，可以按照業務模組拆分子資料夾：
```
app/api/
    ├── flood/
    │   ├── central_api.py
    │   └── flood_api.py
    ├── device/
    │   └── device_api.py
    └── user/
        └── user_api.py
```
**Service 分模組**
- 對應 API 拆分 Service：
```
app/service/
    ├── flood/
    │   └── flood_service.py
    ├── device/
    │   └── device_service.py
    └── user/
        └── user_service.py
```
**Model 擴展**

- PO / VO 數量增加，可能按照業務模組拆分：
```
app/model/
    ├── po/
    │   ├── flood/
    │   └── device/
    └── vo/
        ├── flood/
        └── device/
```

**工具和共用函數**
- `util/` 可以拆分成：
```
util/
    ├── config.py
    ├── exception.py
    ├── http.py
    └── helper.py
```

## 案例 1 

[./model/po/general_1st.py](https://github.com/WaveGIS-Co/prepare-data/blob/master/app/model/po/general_1st.py)

```
app/
 └── model/
     └── po/
         ├── general_1st_po.py      # ORM 類別
         └── general_1st_dao.py     # CRUD 操作
```


## 案例 2

[./service/data_ai/general_1st.py](https://github.com/WaveGIS-Co/prepare-data/blob/master/app/service/data_ai/general_1st.py)

屬於特殊設計 (通用表)，可視為高度客製化的工具
可選名子: 
- infra
- storage
- generic_table
- meta_table
- general_table

```
app/
├── generic_table/
│   └── general_1st/
│       ├── operator.py        # OperationUnit, 
│       │      # OperationalDict, make_crud_functions
│       ├── refresher.py       # General1stPORefresher
│       ├── registry.py        # TYPE -> VO mapping
│
├── adapter/
│   └── web/
│       └── general_1st.py     # create_crud_route, get_router
│
├── model/ # operator會使用到
│   ├── po/
│   └── vo/
│   └── general_1st/          # 如果vo持續增長
│        ├── base.py          # General1stVO
│        ├── relative_station_ai.py
│        ├── flood_alert.py
└── util/
    └── infra/                 # singleton, db, scheduler
```

如果是一般商業邏輯所對應的表，可以放在service或是domain，
原則是 "這個模組如果拿去別的專案，還有意義嗎？"



## 案例 3

[./util/tool.py](https://github.com/WaveGIS-Co/prepare-data/blob/master/app/util/tool.py)

屬於典型tool.py功能過多

拆分策略

- HTTP 請求工具 → http_tool.py
- async / coroutine / singleton 工具 → async_tool.py
- 排程工具 → scheduler_tool.py
- 背景任務管理** → background_task.py
- 鎖管理 → lock_tool.py
- Redis 消費者 → redis_consumer.py

## 細節說明

### controller 和 adapter/web

Java controller 已經同時是 adapter ( annotation的形式)

```
@Controller   ← adapter
   ↓
@Service      ← use case
   ↓
@Repository   ← infra
```

```java
@RestController
@RequestMapping("/flood")
public class FloodController {

    @GetMapping("/list")
    public List<FloodVO> list(...) {
        return floodService.list(...);
    }
}
```

+ Python 需要刻意分層，但如果邏輯簡單，controller會合併到service 
+ 在 Python 專案中，controller / use-case 通常是被「省略到 service」

adapter/web <=> controller <=> service
adapter/web <=> service

```python
class UserService:
	# ....
    def register_user(self, ...):
        validate(...)
        create_user(...)
        send_email(...)
```

這個 `register_user`：
- 控制流程
- 組合多個能力
- 對外是一個動作

👉 **它語意上其實是 use-case / controller**

只是被命名成 service。

### controller 和 service 的差異

> Controller / Use-case：負責「一次請求的流程」  
> Service：負責「可被重複使用的業務能力」


1️⃣ Controller / Use-case 在做什麼？

核心特徵

- ❌ 不可重用（只為某個流程存在）
- ✅ 決定「順序、分支、組合」
- ✅ 一次 request / job / command 的**完整流程**
典型內容

```
1. 驗證輸入
2. 查資料 A
3. 根據結果決定要不要做 B
4. 更新 C
5. 觸發 D（event / refresh / task）
```

2️⃣ Service 在做什麼？

核心特徵

- ✅ 可重用
- ❌ 不掌控整個流程
- ❌ 不該知道「誰先誰後」
典型內容

```
class General1stService:
    def select(...)
    def update(...)
    def delete(...)

def calculate_rain_freq(...)
def validate_station(...)
```

👉 **Service = 能力，不是流程**

|問題|屬於|
|---|---|
|「我要不要先做 A 再做 B？」|Controller / Use-case|
|「怎麼做 A？」|Service|
|「這段邏輯會不會被別人拿去用？」|Service|
|「這段邏輯只為這條 API 存在嗎？」|Controller|

### 資料夾 api

分成兩種

./api:  
封裝外部 API / 整理資料

./service/XXX/api
對應java的controller




