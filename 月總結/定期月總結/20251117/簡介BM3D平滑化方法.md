
[back](../../20251117定期月總結.md)

BM3D（Block-Matching and 3D Filtering）是一種強大的圖像去噪方法，其核心思想是 **將相似的小區塊（patch）聚集起來進行三維頻域處理**。以下是主要流程：

### 1️⃣ 取出圖像小區塊

- 將原始圖像 X×Y 切分成大小為 N×N 的小區塊（patch）。
- 以目前像素為中心，抽取 patch_k 作為參考 patch。

### 2️⃣ 找到相似 patch

- 計算 patch_k 與圖像中其他 patch_i 的相似性，通常用平方差或距離衡量。
- 選取與 patch_k 最相似的 k 個 patch（k=16、32、64）形成一組（group）。
- 這是一種非監督式分組，相似 patch 堆疊形成 **3D 結構** k×N×N

#### patch向量權重計算

$$
\mathbf{x}_t =
\begin{bmatrix}
x(t - k) \\
x(t - k + 1) \\
\vdots \\
x(t + k)
\end{bmatrix}
$$

$$
d^2(t_0, t)
= \|\mathbf{x}_{t_0} - \mathbf{x}_t\|^2
= \sum_{i=-k}^{k} [x(t_0+i) - x(t+i)]^2
$$

$$
w(t_0, t)
= \exp\!\left(-\frac{\|\mathbf{x}_{t_0} - \mathbf{x}_t\|^2}{2h^2}\right)
$$


### 3️⃣ 3D 頻域轉換

- 對每個 3D 結構進行 **3D 變換**，將資料轉換到頻率域。
    
- 標準 BM3D 做法：先對每個 patch 做 2D DCT，再沿 patch 堆疊方向做 1D DCT（等價於 3D DCT）。
    
- 其他變體可用 Wavelet 或濾波代替部分 DCT，視應用需求而定。

#### 偽3D DCT

  1. 2D DCT + 1D Wavelet(Haar)
  2. 2D Wavelet + 1D DCT
  3. 2D Wavelet + 1D 濾波
  4. 2D DCT + 1D 濾波



### 4️⃣ 頻率域去噪（Thresholding）

- **硬閾值 (Hard Thresholding)**：將低於閾值的頻率直接設為 0，保留高頻成分。
    
- **軟閾值 / Wiener 過濾 (Soft Thresholding)**：根據信號功率與噪聲功率對每個頻率加權，降低噪聲但保留更多細節。
    

### 5️⃣ 反變換回 3D 結構

- 將經閾值處理的頻率域資料反變換回原來的 3D patch 結構。
    

### 6️⃣ 聚合重建原圖（Weighted Aggregation）

- 將 3D 結構中的每個 patch 投影回原圖對應位置。
    
- 多個重疊 patch 的值會加權平均，得到每個像素的最終去噪值。






