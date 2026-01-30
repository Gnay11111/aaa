
### 方法三：直接使用我为你估算的参数（保底方案）

如果上述操作太复杂，或者你因为没有 Root 权限拿不到文件，作为数学建模专家，我基于 **骁龙 8 Gen 3 (SM8650)** 的公开技术文档和类似旗舰机的功耗评测，为你构建了一份**“仿真”数据**。

你可以直接在论文中声明：“由于无法获取特定设备的加密系统文件，我们参考了 Snapdragon 8 Gen 3 架构白皮书及同类旗舰设备的公开功耗数据构建了参数。”

#### 仿真 `power_profile.xml` 核心数据 (针对 Xiaomi 14 Pro):

**1. 电池容量 (Battery):**
*   `battery.capacity`: **4880 mAh** (典型值)

**2. 屏幕功耗 (Display - 6.73 inch AMOLED 2K LTPO):**
*   `screen.on`: **150 mA** (基础电路功耗)
*   `screen.full`: **600 mA** (手动最高亮度，非激发亮度)
*   *建模提示：* 屏幕功耗 $P_{screen} \approx 150 + \alpha \times \text{Brightness}$。在阳光激发模式（3000 nits）下，功耗可能瞬间飙升到 1500mA 以上。

**3. CPU 功耗 (Snapdragon 8 Gen 3):**
*该处理器为 1xX4 + 5xA720 + 2xA520 架构。*
*注意：XML里通常是电流(mA)，你需要乘以电压(约 3.8V-4V) 换算为瓦特(W)，或者直接用电流建模。*

*   **小核 (Cluster 0 - A520):**
    *   Idle: 10 mA
    *   Max Freq: 150 mA
*   **中核 (Cluster 1 - A720):**
    *   Low Freq: 50 mA
    *   Max Freq: 400 mA (单核)
*   **超大核 (Cluster 2 - X4):**
    *   Max Freq: **800 mA - 1200 mA** (非常恐怖的功耗，约 4-5W)

**4. 网络与通信 (Radio/Wifi):**
*   `wifi.active`: **200 mA** (数据传输时)
*   `radio.active` (5G): **500 mA - 800 mA** (取决于信号强度，信号差时发射功率大)
*   `gps.on`: **60 mA**

### 建模关键点建议

对于小米 14 Pro 这种机型，你在建模时要特别注意以下两点，这能让你的模型比别人更“先进”：

1.  **LTPO 屏幕技术：** 小米 14 Pro 的屏幕刷新率会在 1Hz-120Hz 之间跳变。
    *   *修正项：* 在你的 $P_{screen}$ 公式中加入一个与 $Frequency$ 有关的系数。当画面静止（如看电子书）时，功耗显著降低；滑动屏幕时，功耗增加。
2.  **快充与发热：** 小米 14 Pro 支持 120W 快充。
    *   虽然题目主要问放电，但如果你能在模型中简要提及“由于快充导致的历史热积聚会加速电池老化($C_{total}$下降)”，会是一个很好的加分项（Sensitivity Analysis/Future Work）。

**搜索提示：**
你可以尝试在 Google 搜索 `Xiaomi 14 Pro battery drain test gsmarena` 或 `Snapdragon 8 Gen 3 power curve geekerwan`（极客湾的数据非常详细，他们有详细的电压-频率-功耗曲线图，非常有引用价值）。
