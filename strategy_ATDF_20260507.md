# ATDF 策略報告：台指期貨自適應趨勢追蹤策略

**策略縮寫**：ATDF（Adaptive Trend-Following Dual EMA）  
**資產標的**：台指期貨（TAIEX Futures，TAIFEX）/ 代理指數 ^TWII  
**報告日期**：2026-05-07  
**策略類型**：趨勢追蹤（Trend-Following）、多頭限制（Long-Only）  

---

## 一、策略概述

ATDF 策略是一個專為台指期貨設計的趨勢追蹤策略，結合雙均線（Dual EMA）交叉訊號與 ADX 趨勢強度過濾器，僅在市場呈現明確上升趨勢時持有多頭部位，其餘時間保持空倉。策略核心理念是「順勢而為、逆勢觀望」，透過濾除盤整期的雜訊訊號來減少不必要的交易損耗。

**核心優勢**：
- 回測期間（2015–2026）CAGR 15.9%，顯著優於大盤買進持有的 13.5%
- Sharpe Ratio 1.294，超越指數買持的 0.844
- 最大回撤僅 -18.7%，大幅低於指數的 -31.6%
- Calmar Ratio 0.851，幾乎是指數 0.426 的兩倍

---

## 二、理論基礎

### 2.1 市場假說

本策略建立於以下兩個核心市場假說：

**假說一：動量效應（Momentum Effect）**
> 資產價格具有時間序列動量特性——近期表現強勢的資產，未來一段時間仍傾向繼續強勢。此效應在跨資產類別及跨市場均有學術實證支持，尤其在月度至季度的持有週期下最為顯著。

**假說二：趨勢持續性（Trend Persistence）**
> 當總體經濟或產業結構性因素推動資產進入趨勢行情時，該趨勢往往具有慣性，不會在短期內立即反轉。移動平均線交叉作為趨勢濾波器，能有效辨識此類持續性趨勢。

**台灣市場特性補充**：台灣加權指數長期受科技股（尤其台積電）主導，結構性多頭趨勢明顯，使得長多策略在歷史上優於長空或雙向交易。

---

### 2.2 數學公式化

**符號定義**：

| 符號 | 定義 |
|------|------|
| $P_t$ | 第 $t$ 日收盤價 |
| $\text{EMA}(P, n)_t$ | 以 $n$ 日指數加權移動平均，平滑因子 $\alpha = \frac{2}{n+1}$ |
| $\text{ATR}_t$ | Wilder 平滑真實波幅，週期 $n=14$ |
| $\text{ADX}_t$ | 平均方向指數，週期 14 |
| $\text{DI}^+_t, \text{DI}^-_t$ | 正/負方向指標 |
| $r_t$ | 第 $t$ 日收益率 $= (P_t - P_{t-1}) / P_{t-1}$ |
| $S_t$ | 第 $t$ 日策略部位，$S_t \in \{0, 1\}$（多頭或空倉）|

**指標計算**：

$$\text{EMA}(P, n)_t = \alpha \cdot P_t + (1-\alpha) \cdot \text{EMA}(P, n)_{t-1}, \quad \alpha = \frac{2}{n+1}$$

$$\text{TR}_t = \max(H_t - L_t, |H_t - P_{t-1}|, |L_t - P_{t-1}|)$$

$$\text{ATR}_t = \frac{(n-1) \cdot \text{ATR}_{t-1} + \text{TR}_t}{n}$$

$$\text{DI}^+_t = 100 \cdot \frac{\tilde{+DM}_t}{\text{ATR}_t}, \quad \text{DI}^-_t = 100 \cdot \frac{\widetilde{-DM}_t}{\text{ATR}_t}$$

$$\text{DX}_t = 100 \cdot \frac{|\text{DI}^+_t - \text{DI}^-_t|}{\text{DI}^+_t + \text{DI}^-_t}, \quad \text{ADX}_t = \text{Wilder}(\text{DX}, 14)_t$$

**進場條件（多頭訊號）**：

$$\text{Signal}_t = \mathbb{1}\bigl[\text{EMA}(P, 10)_t > \text{EMA}(P, 200)_t\bigr] \cdot \mathbb{1}\bigl[\text{ADX}_t \geq 15\bigr]$$

**部位決策**（執行於次日，避免 look-ahead bias）：

$$S_t = \text{Signal}_{t-1}$$

**策略日收益率**：

$$R_t = S_t \cdot r_t - \delta \cdot |\Delta S_t|$$

其中 $\delta = 0.001$（10 bps 單邊交易成本），$|\Delta S_t| = |S_t - S_{t-1}|$

---

### 2.3 Pseudo-code

```python
# ATDF Strategy — Pseudo-code
# 初始化參數
EMA_FAST = 10       # 短期均線
EMA_SLOW = 200      # 長期均線
ADX_PERIOD = 14     # ADX 計算週期
ADX_THRESH = 15     # 趨勢強度門檻
COST_BPS   = 0.001  # 10 bps 單邊成本

# 每日信號計算（當日收盤後執行）
for t in trading_days:
    ema_fast[t] = EMA(close, EMA_FAST)[t]
    ema_slow[t] = EMA(close, EMA_SLOW)[t]
    adx[t]      = ADX(high, low, close, ADX_PERIOD)[t]

    # 趨勢上升且趨勢強度足夠 → 明日進場多頭
    if ema_fast[t] > ema_slow[t] and adx[t] >= ADX_THRESH:
        raw_signal[t] = LONG   # 1
    else:
        raw_signal[t] = FLAT   # 0

# 次日執行（避免 look-ahead bias）
for t in trading_days:
    position[t]     = raw_signal[t-1]   # 訊號延遲一天執行
    turnover[t]     = abs(position[t] - position[t-1])
    cost[t]         = turnover[t] * COST_BPS
    daily_return[t] = position[t] * price_return[t] - cost[t]

# 績效計算
equity_curve = cumprod(1 + daily_return) * initial_capital
```

---

### 2.4 風險因子分析

| 風險類型 | 說明 | 緩解措施 |
|---------|------|---------|
| **趨勢反轉風險** | 多頭趨勢突然反轉（如 2022 年升息衝擊），策略因持有多頭而蒙受損失 | ADX 過濾器減少進入盤整期 |
| **鞭打效應（Whipsaw）** | 假突破導致頻繁進出場，交易成本累積 | EMA 200 長期均線降低交叉頻率 |
| **結構性偏差** | 策略基於歷史多頭市場優化，若台股進入長期空頭則失效 | 可加入熊市避險機制 |
| **流動性風險** | 台指期近月合約於劇烈波動時可能滑價擴大 | 實際操作需考慮下單規模限制 |
| **利率環境** | 高利率環境導致台股本益比收縮，策略多頭暴露期長 | ADX 門檻可動態調整 |
| **單一標的集中** | 策略僅交易台指期一個標的，無分散效果 | 結合其他資產類別 |

**策略可能失效情境**：
- 長期橫盤整理市場（ADX 持續低於 15）
- 多次急跌急彈但 EMA 未形成死叉（如 2020 年 COVID 崩盤後快速反彈）
- 台股進入如日本失落十年式的長期空頭

---

## 三、文獻佐證

```
Jegadeesh, N., & Titman, S. (1993). Returns to Buying Winners and Selling Losers:
Implications for Stock Market Efficiency. Journal of Finance, 48(1), 65–91.
核心發現：美股股票在 3–12 個月持有期下，過去表現強的標的繼續強勢，
         動量效應在統計上顯著（月均超額報酬 ~1%）。與本策略關聯：
         支撐以移動平均線捕捉持續性動量趨勢的核心假說。
```

```
Moskowitz, T. J., Ooi, Y. H., & Pedersen, L. H. (2012). Time Series Momentum.
Journal of Financial Economics, 104(2), 228–250.
核心發現：跨 58 種流動性期貨（股指、商品、外匯、利率），過去 12 個月
         正報酬資產未來一個月繼續正報酬，策略 Sharpe Ratio 約 1.28。
         與本策略關聯：直接支持期貨市場趨勢追蹤策略的跨市場有效性，
         且台股期貨屬於文獻所涵蓋的股指期貨類別。
```

```
Wilder, J. W. (1978). New Concepts in Technical Trading Systems.
Trend Research (Greensboro, NC). (廣泛引用於量化文獻)
核心發現：ADX 指標量化趨勢強度，ADX > 25 代表強趨勢，ADX < 20
         代表盤整。文獻中提出結合方向指標（+DI/-DI）與 ADX 門檻的
         完整交易系統框架，與本策略的 ADX 過濾設計直接對應。
```

```
Faber, M. T. (2007). A Quantitative Approach to Tactical Asset Allocation.
Journal of Wealth Management, 9(4), 69–79.
核心發現：以 10 個月（約 200 日）移動平均線作為進出場條件，對美股、
         商品、REIT 等資產進行回測，相較買持可降低 50% 以上最大回撤，
         Sharpe Ratio 顯著改善。與本策略關聯：支持 EMA(200) 作為長期
         趨勢濾波器的有效性，與本策略的多空切換機制高度一致。
```

---

## 四、回測結果

### 4.1 績效摘要

| 指標 | ATDF 策略 | 買進持有 ^TWII | 策略優勢 |
|------|-----------|--------------|---------|
| 回測期間 | 2015-01-23 ~ 2026-04-29 | 同左 | — |
| **CAGR** | **15.90%** | 13.47% | **+2.43%** |
| 年化波動度 | 12.45% | 17.49% | 更低波動 |
| **Sharpe Ratio** | **1.294** | 0.844 | **+53%** |
| **最大回撤** | **-18.69%** | -31.63% | **少損 -12.9%** |
| **Calmar Ratio** | **0.851** | 0.426 | **+100%** |
| 交易次數（多頭段） | 40 次 | — | — |
| 平均持倉天數 | 64 天 | — | — |
| 持倉比例 | 63.3% | 100% | 空倉期無風險 |
| 單邊交易成本 | 10 bps | 0 bps | 含成本後仍優 |

### 4.2 年度報酬比較

| 年份 | ATDF 策略 | 買進持有 | 相對 α |
|------|-----------|---------|--------|
| 2015 | **+0.7%** | -11.0% | **+11.7%** |
| 2016 | +5.6% | +11.0% | -5.4% |
| 2017 | **+17.2%** | +15.0% | +2.2% |
| 2018 | **+6.9%** | -8.6% | **+15.5%** |
| 2019 | +12.4% | +23.3% | -10.9% |
| 2020 | +17.9% | +22.8% | -4.9% |
| 2021 | +18.2% | +23.7% | -5.5% |
| 2022 | **-3.6%** | -22.4% | **+18.8%** |
| 2023 | +22.2% | +26.8% | -4.6% |
| 2024 | **+31.1%** | +28.5% | +2.6% |
| 2025 | +20.4% | +25.7% | -5.3% |
| 2026* | +35.7% | +35.7% | +0.0% |

> *2026 年為截至 4 月底的累積報酬（非全年），數字放大為正常現象。

**觀察**：策略在熊市年份（2015、2018、2022）大幅跑贏指數，空倉機制有效保護資本；在牛市年份（2019–2021）略微落後，為策略的合理代價。

### 4.3 回測圖表

> 圖表已儲存至：`atdf_backtest_chart.png`

圖表包含：
1. 淨值曲線 vs 買進持有（1M NTD 起始資金）
2. 策略最大回撤 vs 指數回撤對比
3. 各年度報酬長條圖
4. 台指收盤價 + EMA(10)/EMA(200) + 持倉區間標示
5. ADX / +DI / -DI 指標走勢

---

## 五、策略限制

1. **資料代理問題**：回測使用現貨指數 ^TWII 作為期貨代理，未考慮期貨轉倉成本（Roll Cost）、保證金利息收益（Carry）及基差風險（Basis Risk）。實際台指期一年約換倉 4 次，轉倉成本需另行計算。

2. **參數過度配適風險**：最佳參數組合（EMA 10/200 + ADX≥15）係由網格搜尋得出，可能存在樣本內過度配適。建議進行走期分析（Walk-Forward Analysis）驗證穩健性。

3. **執行假設過於理想**：回測假設以收盤價執行（信號為前日收盤後計算，隔日執行），實際執行需面對缺口開盤、流動性不足等問題。

4. **未含保證金與槓桿**：台指期每口保證金約為合約價值 10-15%，槓桿效果會放大波動度與回撤，本策略以不使用槓桿方式呈現。

5. **長多偏差（Long Bias）**：僅做多、不做空的設計依賴台股長期多頭趨勢。若進入日本式長期停滯，策略效益將大幅下降。

---

## 六、改進方向

| 方向 | 具體建議 | 預期效益 |
|------|---------|---------|
| **多空雙向** | 在空頭市場加入小台指放空訊號（需強化風控） | 提升熊市捕捉能力 |
| **部位規模化** | 依 ATR/VIX 調整部位大小（風險平價） | 降低波動度，提升 Sharpe |
| **機制性止損** | 加入 2x ATR 追蹤停損，縮短持虧時間 | 降低最大回撤 |
| **多標的分散** | 延伸至電子期、金融期、黃金期等 | 降低單一標的風險 |
| **機器學習強化** | 以 XGBoost / LSTM 預測 ADX 是否有效並動態調整門檻 | 提升訊號品質 |
| **樣本外驗證** | 使用 2010–2014 作為訓練集，2015–2026 作為測試集 | 驗證策略穩健性 |
| **Walk-Forward 分析** | 滾動 3 年訓練 + 1 年測試，逐期驗證 | 量化過配適程度 |

---

## 七、參考文獻

1. Jegadeesh, N., & Titman, S. (1993). Returns to Buying Winners and Selling Losers: Implications for Stock Market Efficiency. *Journal of Finance*, 48(1), 65–91.

2. Moskowitz, T. J., Ooi, Y. H., & Pedersen, L. H. (2012). Time Series Momentum. *Journal of Financial Economics*, 104(2), 228–250.

3. Wilder, J. W. (1978). *New Concepts in Technical Trading Systems*. Trend Research, Greensboro, NC.

4. Faber, M. T. (2007). A Quantitative Approach to Tactical Asset Allocation. *Journal of Wealth Management*, 9(4), 69–79.

5. Brock, W., Lakonishok, J., & LeBaron, B. (1992). Simple Technical Trading Rules and the Stochastic Properties of Stock Returns. *Journal of Finance*, 47(5), 1731–1764.

---

## 附錄：策略代碼檔案

| 檔案 | 說明 |
|------|------|
| `backtest_atdf_final.py` | 最終生產版回測腳本（向量化實作） |
| `backtest_atdf_v3.py` | 版本三參數搜尋版本 |
| `atdf_metrics.json` | 量化績效指標 JSON 輸出 |
| `atdf_backtest_chart.png` | 5 格回測圖表（淨值/回撤/年報酬/均線/ADX） |

---

*本報告由量化策略研究系統自動生成，僅供學術研究與策略探索使用，不構成任何投資建議。*  
*所有回測結果均為歷史模擬，不代表未來實際績效。期貨交易具有高度風險，需充分了解相關規則後方可操作。*
