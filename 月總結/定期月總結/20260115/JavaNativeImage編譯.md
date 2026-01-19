[back](../../20260115定期月總結.md)

> 將java服務編譯成二進位檔，執行不依賴JVM

## 限制

有嘗試用spring框架的程式碼編譯，但是設定繁瑣且沒有成功
需要改使用專門的框架，下面是用  micronaut 框架測試

## 編譯

```shell
docker run  --rm -it \
  --entrypoint /bin/bash \
  -v .:/app \
  -w /app \
  ghcr.io/graalvm/native-image-community:21

microdnf install -y findutils
chmod +x gradlew
./gradlew clean nativeCompile
chmod +x /app/build/native/nativeCompile/XXXXX
./app/build/native/nativeCompile/XXXXX
```

## 實驗

```shell
# Native Image
# 最簡容器
docker run  --rm -it \
  --name native-image-with-small-container \
  --entrypoint ./device-siren-simple \
  -p 8501:8080 \
  -v .:/app \
  -w /app \
  gcr.io/distroless/cc-debian12:latest

# Native Image
# 有jvm容器
docker run  --rm -it \
  --name native-image \
  --entrypoint ./device-siren-simple \
  -p 8502:8080 \
  -v .:/app \
  -w /app \
  ghcr.io/graalvm/jdk-community:21

  
# jar file
# 有jvm容器
docker run --rm -it \
  --name jar-file \
  -v .:/app \
  -p 8503:8080 \
  -w /app \
  ghcr.io/graalvm/jdk-community:21 \
  java -jar device-siren-simple2-all.jar
```

## 實驗結果

|容器名稱|記憶體使用 (MEM USAGE)|區塊 I/O (BLOCK I/O)|執行緒數 (PIDS)|
|---|---|---|---|
|native-image-small|13.83 MiB|0B / 0B|4|
|native-image|13.82 MiB|0B / 0B|4|
|jar-file (標準 JVM)|91.09 MiB|0B / 61.4kB|50|

- **記憶體大幅節省**：使用 Native Image 後，記憶體消耗極低（約 13-14 MiB），這對於在高密度環境（如 Kubernetes）中運行微服務非常有優勢。
    
- **執行緒 (PIDS) 減少**：標準 JVM 需要大量的背景執行緒進行 GC 與 JIT 編譯，而 Native Image 將這些工作提前到了編譯期，因此執行時非常輕量（僅 4 個 PIDS）。
    
- **容器大小影響**：可以看到 `native-image` (基於完整 OS) 與 `native-image-with-small-container` (Distroless) 在執行時的記憶體差異極小，這說明了 Distroless 主要優勢在於**硬碟佔用（Image Size）**與**安全性**，而非執行時的記憶體。

## 甚麼是 Micronaut

Micronaut 是一個專為現代微服務和無伺服器（Serverless）架構設計的現代 JVM 框架。

---

### 1. 為何 Micronaut 這麼快？（核心原理）

傳統的框架（如 Spring Boot）在啟動時會進行大量的 **「執行期反射 (Runtime Reflection)」** 和 **「動態代理」**。

- **Spring 的作法**：啟動時掃描所有類別，找 `@Autowired`、`@Controller`，這會導致啟動慢且吃記憶體。
    
- **Micronaut 的作法**：在**編譯期**就決定好所有的依賴注入（Dependency Injection）。當你的 Java 程式編譯成 `.class` 時，Micronaut 就已經把注入邏輯寫好了。
    

> **結果**：啟動速度與記憶體消耗不會隨著 Bean 的數量增加而線性暴漲。

---

### 2. Micronaut 的關鍵特性

|**特性**|**說明**|
|---|---|
|**原生支援 GraalVM**|它是市場上與 Native Image 整合度最好的框架之一，不需要寫繁雜的反射設定檔。|
|**無反射 (Reflection-Free)**|完全不依賴反射或動態類別載入，這也是它極其節省記憶體的主因。|
|**雲端原生 (Cloud Native)**|內建支援服務發現（Consul/Eureka）、配置中心、斷路器（Circuit Breaker）等功能。|
|**反應式程式設計**|基於 Netty 開發，原生支援非阻塞 I/O，處理高並發能力強。|

---

### 3. 簡單代碼範例

Micronaut 的語法跟 Spring 非常像，轉換成本很低：

Java

```
@Controller("/hello")
public class HelloController {
    @Get("/")
    public String index() {
        return "Hello World";
    }
}
```

- **Micronaut Data**：學會如何不寫 SQL 就能操作資料庫（JDBC）。
- **Validation**：學會如何驗證輸入資料（Bean Validation）。

