[back](../../20260115定期月總結.md)
 
> 該問題在java 25已經有很大程度的優化

## Virtual Thread（虛擬線程）

- **概念**：  
    Virtual Thread 是 Java 21 中引入的輕量級線程，它不同於傳統的 OS Thread（Platform Thread）。虛擬線程的目標是**數量可以非常大**，適合高併發 I/O 場景。
    
- **特點**：
    1. **輕量**：一個虛擬線程只是一個 Java 對象，並不直接綁定操作系統線程。
    2. **非阻塞調度**：當虛擬線程遇到阻塞（如 I/O 或 Thread.sleep），會自動掛起並釋放底層 Carrier Thread。
    3. **大量併發**：理論上可以支持上百萬個虛擬線程，而傳統 Thread 數量通常只有幾千個。
        

---

## Carrier Thread（承載線程）

- **概念**：  
    Carrier Thread 是實際由操作系統管理的線程（Platform Thread），它**承載虛擬線程的執行**。可以把它理解為虛擬線程的「跑步機」。
- **關係**：
    - 每個虛擬線程在執行 Java 代碼時，必須借用一個 Carrier Thread。
    - 當虛擬線程被阻塞（如等待網路 I/O），Carrier Thread 可以釋放去執行其他虛擬線程。
    - Carrier Thread 數量通常比虛擬線程少很多，可以大幅降低線程上下文切換成本。

---

## Pinned Thread（綁定線程）

- **概念**：  
    Pinned Thread 是一種特殊情況，虛擬線程**被固定綁定到 Carrier Thread**，無法被動態調度到其他 Carrier Thread。
        
- **問題與影響**：
    
    1. **阻塞 Carrier Thread**：Pinned Thread 一旦執行，Carrier Thread 不能切換給其他虛擬線程，可能造成性能下降。
        
    2. **高併發受限**：如果大量虛擬線程變成 Pinned Thread，就會喪失虛擬線程的「輕量可擴展」優勢。
        
    3. **需要注意調度**：在使用 JNI 或本地資源時，需要小心設計，避免虛擬線程被不必要地 Pinned。

---

## synchronized 會造成 Pinned Thread

- Virtual Thread 的特點是 **可以在遇到阻塞時掛起**，Carrier Thread 可以去跑其他虛擬線程。
    
- 但是，`synchronized` 是 **Java 層的內置鎖**，底層是 **Monitor**（操作系統 mutex 封裝）。
    
- JVM 必須保證：
    
    - 同一把鎖在同一時刻只被一個線程持有。
    - 所以當虛擬線程持有鎖或試圖進入被鎖住的區塊時，它**必須固定在一個 Carrier Thread 上**來保證安全。
        
- 這就導致 **虛擬線程被 Pinned**，Carrier Thread 無法再切換給其他虛擬線程，直到鎖釋放。
    

換句話說：

> `synchronized` 這種「依賴於線程身份和調度的鎖」會破壞虛擬線程的可遷移性，所以 JVM 會自動把虛擬線程固定到 Carrier Thread。


```java
import java.util.concurrent.Executors;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.TimeUnit;

public class VirtualThreadPinnedDemo {

  private static final Object lock = new Object();

  public static void main(String[] args) throws Exception {
    // 建立虛擬線程執行器
    ExecutorService executor = 
      Executors.newVirtualThreadPerTaskExecutor();

    // 建立多個虛擬線程，每個都要搶同一個鎖
    for (int i = 0; i < 7; i++) {
      int threadNum = i;
      executor.submit(() -> {
        System.out.println(
	      Thread.currentThread() + " 嘗試進入 synchronized 鎖"
        );
        synchronized (lock) {
          System.out.println(
            Thread.currentThread() + " 已持有鎖，開始工作..."
          );
          try {
            Thread.sleep(2000); // 模擬工作
          } catch (InterruptedException e) {
            throw new RuntimeException(e);
          }
          System.out.println(
            Thread.currentThread() + " 完成工作，釋放鎖"
          );
        }
      });
    }

    executor.shutdown();
    executor.awaitTermination(100000, TimeUnit.SECONDS);
    System.out.println("Main thread exiting");
  }
}
```

## java 21

![[./定期月總結/20260115/Carrier Thread java 21.png]]

## java 25

![[./定期月總結/20260115/Carrier Thread java 25.png]]


