[back](../../20251015定期月總結.md)


## 控制目的

+ 讓系統達到期望輸出
+ 公式與實際的誤差
+ 快速達到目標狀態減少震盪


## PID 控制器：

$$ u(t) = K_p e(t) + K_i \int_0^t e(\tau) d\tau + K_d \frac{de(t)}{dt} $$

|控制項|在學習中的意義|類比對應|
|---|---|---|
|Kp|根據當前誤差立即修正|梯度方向|
|Ki|修正長期偏差|動量（Momentum）或 Adam 中的平均梯度|
|Kd​|抑制快速變化|學習率衰減或正則化|


```python
# --- PID 參數 ---
Kp, Ki, Kd = 0.5, 0.05, 0.2
# 過去的誤差
e_prev = 0
# 積分項
I = 0

# === PID 控制 ===
e_pid = target_t - y_pid
I += e_pid
D = e_pid - e_prev
u_pid = Kp * e_pid + Ki * I + Kd * D
e_prev = e_pid
```