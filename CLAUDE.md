# CLAUDE.md — 資產札記 Ledger

> 給下次接手的 Claude:讀完這份就能直接動手,不用問 Sela 已經決定過的事。
> 這個專案是 Sela 個人用的資產追蹤工具,單檔 HTML,部署在 GitHub Pages。

---

> **⚠ 給同時拿到 SELA-Starter-Kit 的 Claude:**
> 這是**已對齊 Kit V1.8.0 的成熟專案**(V0.8.0 起),不是新專案。
>
> 衝突仲裁規則:
> 1. **以本專案 CLAUDE.md 為主、Kit 為輔**
> 2. 本專案刻意不對齊 Kit 的部分:
>    - **配色保留「黑底金字」**(`--bg: #0e0d0b` / `--gold: #d4b074`),非 Kit 預設「SELA 主題橘 #F36825」也非「北歐霧藍 #5A7A8B」 — Sela 的設計選擇,理由是長時間看資產想要沉浸式深色降低焦慮感,已驗證 7 個版本
>    - **`theme-color` meta 用 `#0e0d0b`**(配合 PWA 啟動畫面),非 Kit logo/CLAUDE.md 4.1 預設的 `#F36825`
>    - **CLAUDE.md 章節結構保留**(累積 28+ 條坑、業務事實、版本歷程),不依 templates/CLAUDE-template.md 重排 — 結構已符合 conventions/CLAUDE-MD-章法.md 規範
>    - **帳戶 ID 中拼混合**(yunxiang / mantou / yushan_loan / taishin_car_loan)— 對映 Excel 既有業務術語,不對齊「全英」風格
> 3. **不要為對齊 Kit 而動既有設計** — 已驗證的就是事實標準
> 4. SELA logo 鐵律照走:橘 + 白,不改色、不重畫、保留 ®(Kit logo/CLAUDE.md §3 鐵律)
> 5. 版號規則照 Kit:三位數、空格分隔、部署版無後綴

---

## ⚠️ V0.9.0 升版必讀

V0.9.0 是**業務增量大版**:加「持股」整個新 tab 追蹤美股(嘉信 + 第一 + 永豐三家),加再平衡建議、加權年化報酬計算、Yahoo + 兩層反向代理股價 API、設定頁可編輯股數/現金/預定%/資金流。完整設計文件見 `V0.9.0-DESIGN.md`(已實作完成)。

主要動作:
- `DATA` 加 `holdings` 子物件:`brokers`、`positions`、`target_allocation_schwab`、`cash_by_broker`、`fxrate`、`fund_flows`、`quotes`
- `td_us` 帳戶 name 從「TD 美股」改「美國券商」(id 不動,DEFAULT_DATA + EMBEDDED_HISTORY 兩處同步改)
- 加 nav tab「持股」+ page slot + `renderHoldings` / `renderHoldingsAll` / `renderHoldingsBroker` / `renderPositionGroup` / `renderRebalance` / `renderFundFlowSummary`
- 加演算法:`computeBrokerStats`(加權年化)、`computeBrokerMarketValue`、`computeAllHoldingsStats`、`computeRebalance`、`computeLedgerAlignment`
- 加 API:`fetchQuote`(Yahoo 直連 → r.jina.ai → corsproxy.io 三層 fallback)、`refreshAllQuotes`(並行抓、失敗保留舊 quote)
- 設定頁加「持股設定」整段:匯率/三家現金/三家持股清單(可改股數/tier/刪除/新增)/嘉信預定%/資金流加單筆/全清
- `loadData()` 加 holdings 補齊邏輯:深拷貝 DEFAULT_DATA.holdings(避免引用穿透)
- 加 `import_holdings.py`:Excel fund_flows → JSON,匯入大量歷史資金流用
- 看坑 #31

未動的東西:
- Excel 12 條公式對齊不變(V0.6 退休試算)
- 凍結機制不變(V0.7 computed)
- 配色 / 章節結構 / 帳戶 ID 不變(V0.8 對齊 Kit 後決定)
- 月度資產表(snapshots)不變,持股**不存 snapshot**(只存 DATA.holdings,即時的)

**部署選項**:
- 選項 A(推薦):清空目錄,推 V0.9.0.zip → 用戶 localStorage 自動補齊 holdings(loadData 深拷貝 DEFAULT_DATA.holdings)
- 選項 B(機密考量):先匯出 JSON 備份,再升版,再匯入

**動工驗算對齊**(V0.9.0 已驗證):
- 嘉信合成等價資金流 → 8.40% 年化(目標 8.42%,差 0.02pp 是設計文件 profit 數字本身的精度差)
- 永豐合成等價資金流 → 4.67% 年化(完全吻合)
- 算法用 USD 加權平均(以 USD 金額 × 天數加權),(1+ratio)^(365/weighted_days) - 1

**Sela 後續要補的**(不擋功能):
- 嘉信 VPL / VWO / O 股數(目前 0,顯示「0%/預定 12%」)
- 嘉信「個股」其他 ~6 檔、「投機」其他 ~5 檔(設定頁手動加)
- 全部 fund_flows(嘉信 17 + 永豐 72 筆)從 Excel 用 `import_holdings.py` 匯入

---

## ⚠️ V0.8.0 升版必讀

V0.8.0 是**首次對齊 SELA-Starter-Kit V1.8.0**(走 Kit cross-project-pitfalls #40 的「對齊既有專案 SOP」)。看坑 #29。

主要動作:
- 加 `favicon/` 整套(svg / 多尺寸 png / ico / site.webmanifest)
- `index.html <head>` 改用 SELA 標準 favicon 引用,移除舊的 `data:` URL inline favicon
- README.md 加 SELA logo banner + 重寫描述
- `.gitignore` 補齊 Kit 標準項目(原本 12 行 → 49 行,加 Flet / node / build/ / *.egg-info 等)
- 產出 `SELA-handoff.md`(首次對齊 Kit 是重大里程碑,Kit 規範強制要求)
- topbar 版本號 v0.7.1 → v0.8.0

未動的東西:
- 配色保留(見上方衝突仲裁第 2 條)
- CLAUDE.md 章節結構保留
- 12 條 Excel 公式對齊不變(全部測試重跑通過)
- 凍結機制不變(V0.7 的 computed / getStats / freezeSnapshot)

**部署選項**:
- 選項 A(推薦):清空目錄,推 V0.8.0.zip → 用戶 localStorage 保留(不會被清掉,因為瀏覽器層)
- 選項 B(機密考量):先匯出 JSON 備份,再升版,再匯入

---

## ⚠️ V0.7.1 升版必讀

V0.7.1 是 **V0.7 的修正補強版**:Sela 用手機開設定頁,**完全看不到任何貸款項目**。看坑 #27、#28。

主要修正:
- `loadData()` 加 `mergeSettings()`:升級時用 DEFAULT_DATA 補齊缺少的設定欄位(loan_pmt_wan / salary / birthday 等)
- 貸款月繳卡片改成**完全可編輯**:名稱可改、繳款日可改、可刪除、可新增(`addLoan` / `removeLoan` / `updateLoanField`)
- 卡片 top 區改用 `<input>` 而非 `<span>`,點下去可直接改名稱
- Topbar 加版本號顯示 `v0.7.1`(金色 pill,清楚知道用到哪版)

V0.7 凍結機制全部繼承,Excel 12 條公式對齊最新月份不變。

**部署選項(Sela 已選 A)**:

- 選項 A:**GitHub Pages 公開 + 內嵌歷史**(現況) — 程式碼公開、HTML 裡有 57 筆真實金額
- 選項 B:**GitHub Pages 公開 + 歷史抽出** — `EMBEDDED_HISTORY` 改成空 + 首次手動匯入 ledger-history.json
- 選項 C:**Cloudflare Pages + Access** — 真正的密碼保護,免費

⚠️ **Sela 接受了選項 A 的隱私風險**(她家人非技術背景,JS 密碼 + 看不出意義的 repo 名夠擋)。**不要主動建議改 B/C**,除非她明確問起。

---

## 一、資料模型(Single source of truth)

整個 app 三個 JS 物件就是全部:

| 物件 | 形態 | 何時被讀 | 何時被寫 |
|---|---|---|---|
| `DATA` | 執行期狀態(memory) | 每次 render 都讀 | 每次儲存快照、設定、合併載入 |
| `localStorage[ledger_v2]` | 持久化 | App 啟動時 `loadData()` | 每次 `saveData()` |
| `EMBEDDED_HISTORY` | 程式碼常數 | 只在使用者按「載入合併」時 | 從不寫,deploy 時更新 |

**`DATA` schema**:

```js
{
  version: 2,
  settings: { retirement_target_wan, qyld_yield, monthly_expense_wan, ... },
  accounts: [{ id, name, category, side, liquid, owner, active, auto? }],
  snapshots: [{ date: 'YYYY-MM-DD', balances: { acc_id: number_in_wan } }],
  holdings: {  // V0.9.0 加
    brokers: [{ id, name, ledger_id, currency, rebalance }],         // 3 家:嘉信/第一/永豐
    positions: [{ symbol, broker, shares, tier }],                    // 30 筆 - tier: core/satellite/stock/speculative
    target_allocation_schwab: { TICKER: pct },                        // 嘉信 7 檔目標 %(合計 66%,其餘自由)
    cash_by_broker: { schwab, firstrade, sinopac },                   // 各家現金 USD
    fxrate: number, fxrate_updated_at: 'YYYY-MM-DD',                  // 手動匯率(不抓 API)
    fund_flows: { schwab: [{date, amount_usd, fxrate}], ... },        // 給加權年化算的歷史匯入
    quotes: { TICKER: { price, at } },                                // refreshAllQuotes 寫入
  },
}
```

**金額單位永遠是「萬」**(跟 Sela Excel 一致),**只有 holdings 是 USD 為主**(對齊嘉信/永豐 Excel)。負債用負數。Snapshots 不要存衍生欄位(總資產、負債比都用算的)。**holdings 不進 snapshot**(即時的,跟月度快照分離)。

---

## 二、帳戶結構對映表

**改帳戶 = 改三個地方**:`ACCOUNT_DEFS`(import_excel.py)、`DEFAULT_DATA.accounts`(HTML 內)、`NAME_TO_ID`(import_excel.py 中文名對應)。漏改任一處,匯入腳本就會吐 ⚠ 警告或新帳戶不出現在輸入頁。

| Owner | Account ID | 中文名 | Category | Side | 備註 |
|---|---|---|---|---|---|
| me | td_us, sinopac_us | TD/永豐美股 | stock | asset | 已換算台幣輸入,不存美元原值 |
| me | crypto, tw_stock | 虛擬幣、台股 | stock/crypto | asset | |
| me | sinopac_bank, yushan_bank, ctbc_bank, taishin_bank, oway_bank | 五家銀行 | cash | asset | |
| me | primer, mega_zar, yunxiang | 普瑞默、兆豐南非幣、雲象 | other/cash | asset | yunxiang 在 2025/07 後消失,匯入腳本自動 active=false |
| me | labor_self, labor_perf | 勞退自提/績效 | pension | asset | liquid=false |
| me | house, land | 房子、土地 | real_estate | asset | liquid=false,固定 2600/200 |
| me | yushan_loan | 玉山貸款 | loan | liability | 有 `auto.amortize`,系統算每月扣 |
| me | mantou | 欠饅頭 | loan | liability | |
| me | fubon_loan, taishin_car_loan, xt_loan, cash_legacy | 富邦房貸/台新車貸/小彤借款/舊現金 | - | - | 歷史殘留,新月份不會出現 |
| dora/leo/aaron | {owner}_stock, {owner}_cash | 三小孩股票+現金 | - | asset | **代管,不算 me 總資產** |
| jinghan | jh_stock, jh_loan, jh_cash | 景翰(弟)股票/借券/現金 | - | asset | **代管,不算 me 總資產** |

**衍生欄位都從上表算**,不要寫死在程式裡:

- 總資產 = `me` 帳戶 asset 加總 + liability(負數)
- 動產 = 總資產 - real_estate
- 負債比 = |liab| / asset
- 退休年數 = (target - total) / (movable × yield)
- 菜石差額 = dora 總和 - (leo + aaron) / 2

---

## 三、業務事實(不是 bug,不要試圖修補)

- **雲象不計入總資產,且其市值已包含在「台股」帳戶裡** — V0.6 教訓:雲象是 Sela 的私募股權投資,但其市值在 Sela 的會計裡是放在「台股」帳戶下面(因為性質類似,而且 Sela 不想分開記)。Excel 那個「雲象」欄位只是參考股價,單純為了讓「欠饅頭 = -雲象/15」公式能跑。**`accounts[id=yunxiang].side = 'reference'`** 標記它,`computeStats()` 看到 reference 跳過。Excel 驗證:不含雲象 = D1 = 6695.58 完全對齊。
- **欠饅頭不算「正規貸款餘額」** — V0.6 教訓:Excel D4=SUM(B2:B3) 只算玉山貸款+台新車貸,**不含 mantou**。因為 mantou 是雲象連動的衍生計算,不是固定還款的貸款。`computeStats()` 分兩個欄位:`liab` 全部負債、`normal_liab` 排除 formula 類負債。退休試算 I17 用 `normal_liab`,負債比 D8 用 `liab`。
- **退休試算用美股當本金來源** — Excel D6 = 月支出×12÷yld - 美股 - 額外,只把「TD美股+永豐美股」當退休後配息來源(假設房子土地不會賣、勞退太晚拿)。
- **F2 月支出含貸款月繳** — Sela 把退休前的「月支出 = 21.81 萬日常 + 29.52 萬貸款 = 約 50 萬」算進去,所以退休前的「全部支出」要靠美股配息+其他。Excel F4(一菜雙石)是逐月變動的,F3(卡費 25 萬)固定。

下面這些是 Sela 個人真實情況,接手後**不要當成資料異常去處理**:

- **2023 整年沒資料** — Sela 那年沒記錄,Excel 從 2022/10 直接跳到 2024/01。圖表的 X 軸要能容忍中間斷檔,**不要寫程式去「補插值」或「警告缺漏」**。
- **未來月份(2026/01–03)是預估** — Excel 居然有未來資料,因為 Sela 拿攤還公式預先填了「估算未來」。**保留**,別主動提醒「這是未來資料」,Sela 自己知道。
- **「玉山賃款」是筆誤** — Excel 2025/04 之後出現「玉山賃款」(賃 ≠ 貸),是 Sela 打錯沒改回來。匯入腳本已對應到 `yushan_loan`,**UI 永遠顯示「玉山貸款」**,不要顯示「賃款」。
- **2025/05 負債從 -2942 跳到 -3407** — 不是資料錯誤,是 Sela 那月新增了一筆貸款。
- **2026/01 負債多 158 萬** — 也不是錯誤,是台新車貸出現。
- **雲象是私募股權投資,饅頭出資 1/15 與 Sela 共持** — `mantou = -yunxiang/15` 是業務勾稽,不是借貸關係。雲象漲 → 饅頭那份也漲 → 欠饅頭數字變大。**淨資產收益 = 雲象 × 14/15**(扣除饅頭那份)。V0.5 起 mantou 用 `auto.formula` 自動算,不可手改。
- **雲象從 2025/07 起在 Excel 的 C21 欄位**(不是原本 A 欄位置)— 匯入腳本已處理,讀 A 欄找不到雲象不要當 bug,要去抓 C21。
- **QYLD 年化報酬率每月在變動** — Sela 每月 1 號實際去查 QYLD 近 5 年含息年化,寫進 Excel 的 D19。app 端也是每快照記一次,別假設它是固定值。
- **玉山房貸名稱後括號是「繳款日」不是「年限」** — Excel I3 寫「玉山房貸(28)」是指每月 28 號要繳款,提醒 Sela 用。**不是貸款 28 年**。`settings.loan_pmt_wan[i].due_day` 存這個。
- **2029/12/27 是 Sela 50 歲生日** — 退休試算的 anchor。`settings.retire_birthday`。改不改是 Sela 的事(只能改提早或延後)。

下面是 V0.9.0 持股 Tab 相關的業務事實(設計文件 `V0.9.0-DESIGN.md` 動工時必看):

- **td_us 帳戶顯示名稱「美國券商」(V0.9.0 改),實際涵蓋嘉信(主)+ 第一證券(副)兩家**。`id` 維持 `td_us` 不動(改 id 會破壞 6 年快照),只改 `name`。「TD」是歷史包袱(TD Ameritrade 已 2020 併入嘉信)。
- **永豐 Excel R1 標題「TD Ameritrade 867238065」是抄錯**,實為永豐複委託。Sela 確認過,複製貼上沒清。下次匯入腳本看到不要當 bug。
- **永豐 QQQ 102.5 股 = 主追蹤 72.5 + 定期定額 30**(碎股因兩種買法產生)。Excel 在主清單 R11 + 投資對照 R19 分兩處顯示,程式合併成一筆 102.5 股。
- **永豐「投資對照」AMZN/QQQ/MSFT/UBER 是現在持有,不是想買清單** — Sela 確認過,Excel 主清單漏列了,下次匯入要把這 4 檔合併進永豐持股。
- **永豐「主追蹤 8 檔 + 雜部位 USD 24,307」 程式合併當一個清單** — Excel 結構亂(歷史遺留兩種展示),但實際是同一份持股,匯入時不拆兩層。
- **第一證券持股 = TLT 600 股 + LENZ 2000 股** + 現金 USD 2,319(V0.9.0 動工已確認)。Excel 沒明確分配,V0.8.2 起在 `DATA.holdings.positions[broker='firstrade']` 兩筆寫死。
- **嘉信 LENZ 是 4000 股**(V0.9.0 動工 Sela 確認)。第一也有 LENZ 2000 股、永豐也有 LENZ 2200 股 — 三家都持有同一檔,合計 8200 股。Excel R12=2000、R22=4000 是嘉信內部分批買的歷史紀錄,實際倉位 4000(不是 6000、不是 2000)。
- **嘉信再平衡只追 7 檔**(V0.9.0 動工 Sela 確認簡化):核心 4 檔 VTI 24% / VGK 18% / VPL 12% / VWO 6% + 能源 3 檔 VDE 3% / ICLN 2% / URNM 1%,合計 66%。**O / TLT / QYLD / 個股 / 投機**全部不設預定 %,視為自由部位(34%)。原設計文件 9 檔(含 O 5%、TLT 3.24%)已過時。
- **加權年化報酬演算法**:嘉信 8.42% / 永豐 4.67%(從 Excel 反推)。V0.9.0 用合成等價資金流驗算:嘉信 8.40%(差 0.02pp,在精度容差內)、永豐 4.67%(完全吻合)。公式 `(1+profit_ratio)^(365/weighted_days) - 1`,`weighted_days = Σ(amount_usd × days) / Σ(amount_usd)`。看 `index.html` `computeBrokerStats()`。
- **持股是即時的,不像資產表是月度快照** — 不存進 snapshot,只存 `DATA.holdings`。凍結機制不影響 holdings。
- **持股對齊 Ledger 帳戶不主動覆寫** — 嘉信 + 第一 持股總和對齊 td_us、永豐對齊 sinopac_us;差 > 1% 顯示警告,讓 Sela 自己決定改哪邊。延續 V0.7 凍結機制精神:既定事實優先。

---

## 四、踩過的坑(累積編號,永不重排)

1. **同名帳戶兩筆要區分** (V0.3 教訓)
   - 症狀:Excel 2021/01–2021/07 的「富邦房貸」是同名兩筆(房貸主+加貸)
   - 原因:openpyxl 讀進來會兩個 row 同名 key 互相覆蓋
   - 做法:`extract_balances_from_sheet()` 用 `name_count` 加 `__N` 後綴去區分,輸出時再合併

2. **「玉山賃款」是筆誤** (V0.3 教訓)
   - 症狀:Excel 2025/04 之後出現「玉山賃款」,2025/03 之前是「玉山貸款」
   - 原因:Sela 自己打錯字,沒改回來
   - 做法:`NAME_TO_ID` 把兩個都對到 `yushan_loan`,**不要在 UI 顯示「賃款」**

3. **歷史 schema 演化:玉山貸款拆四筆 → 合併一筆** (V0.3 教訓)
   - 症狀:2024/01–2025/03 是「玉山房貸 1/2 + 玉山信貸 1/2」共四筆,2025/04 起併成一筆
   - 原因:Sela 嫌每月輸四次太累,自己合併了
   - 做法:`MERGE_INTO_YUSHAN_LOAN` 把那四個 ID 全部加總到 `yushan_loan`,合併後對得起 Excel(2025/03 = -2955.12 ✓)

4. ~~(已移到「業務事實」章節)~~

5. ~~(已移到「業務事實」章節)~~

6. **localStorage 資料優先 vs 內嵌歷史** (V0.4.0 教訓)
   - 症狀:V0.3 時 `DEFAULT_DATA` 直接內嵌完整歷史,結果使用者輸入新快照後重新整理頁面就被覆蓋
   - 原因:`loadData()` 先檢查 localStorage,沒資料才用 `DEFAULT_DATA`,但 `DEFAULT_DATA` 包含歷史就會誤導
   - 做法:V0.4 把歷史抽到 `EMBEDDED_HISTORY` 常數,`DEFAULT_DATA.snapshots = []`,使用者必須手動「載入合併」才會看到歷史

7. **匯入應該是合併,不是覆蓋** (V0.4.0 教訓)
   - 症狀:V0.3 的「匯入 JSON」直接覆蓋整個 `DATA`,使用者匯入備份後手輸的快照消失
   - 原因:把匯入當「還原」處理,沒考慮「補資料」場景
   - 做法:V0.4 改用 `mergeData()`,以 `date` 為鍵,新檔覆蓋同日舊檔,其他保留。要全清要先按「清空」

8. **JS 密碼保護的安全等級** (V0.4.0 教訓)
   - 症狀:Sela 想要密碼但部署在公開 GitHub Pages
   - 原因:JS hash 寫在前端,任何人 F12 都看得到 hash 值
   - 做法:使用 SHA-256 hash(不是明文)+ 在登入頁底部寫「此處密碼僅用於介面遮蔽」+ DEPLOY 文件清楚說明風險邊界。**不要假裝這是真的安全**

9. **計算總資產時要過濾 owner** (V0.2 教訓)
   - 症狀:V0.1 時 dashboard 顯示的「總資產」把 Dora/Leo/Aaron/景翰 的也算進去
   - 原因:`computeStats()` 沒過濾 owner
   - 做法:`computeStats()` 永遠先 `accounts.filter(a => a.owner === 'me')`,代管帳戶獨立分頁顯示

10. **房子/土地不算動產** (V0.2 教訓)
    - 症狀:退休試算用「動產 × 報酬率」算年化,但 V0.1 把房子當動產也乘上去,結果退休年數爆減
    - 原因:`movable` 沒扣掉 real_estate
    - 做法:`movable = assets - real_estate`,`liquid` 旗標只是分類用,真正算動產用 category

11. **show_inactive 開關要影響「讀」和「寫」** (V0.5.0 教訓)
    - 症狀:V0.5 加 toggle 但只改 `getActiveAccounts()`,結果「歷史頁顯示」過濾了,「月輸入頁」卻沒過濾,行為不一致
    - 原因:多處呼叫 `DATA.accounts` 沒走 `getActiveAccounts()`
    - 做法:統一所有「列出帳戶顯示給 Sela 看」的地方都走 `getActiveAccounts()`,只有 `computeStats()` 例外(它必須跑全部 owner=me 帳戶,不管 active,因為歷史月份的舊帳戶餘額也要計入總資產)

12. **formula 類 auto 跟 amortize 類要分開處理** (V0.5.0 教訓)
    - 症狀:V0.5 把 mantou 改公式後,使用者輸入雲象想看到饅頭跟著變,但只在 goToSnapshot() 算了一次,不會即時更新
    - 原因:formula 依賴其他帳戶當前 balance,需要每次 input event 都重算
    - 做法:`renderSnapshot` 的 input listener 每次寫入後呼叫 `recomputeFormulas(SNAP_DRAFT.balances, SNAP_DRAFT.autoApplied)`,並更新所有 formula 帳戶的輸入欄 DOM 顯示

13. **退休目標自動算後,圖表會出現「rate 變」造成的線條跳動** (V0.5.0 觀察)
    - 症狀:不同月份的 yld 不同,所以「退休目標」每月不同,「離退休還差」也每月跳
    - 原因:這是物理事實,不是 bug
    - 做法:**接受跳動**,不要去做平滑/移動平均。Sela 想看的就是「當下若退休所需的真實本金」

14. **formula 的 auto 不可手改,但要保留歷史 mantou 的手輸值** (V0.5.0 邊界)
    - 症狀:從 Excel 匯入的 2024/03–2025/06 mantou 都是 Sela 手輸的(那時還沒搬到 C21,但欄位邏輯一致),數值跟 -yunxiang/15 一致誤差 < 0.001
    - 原因:歷史值已經對齊公式,不需要重算
    - 做法:**只在「新增 / 編輯」流程套用公式**,匯入歷史 JSON 的舊資料不要強制覆寫(避免浮點誤差造成歷史異動)

15. **編輯舊月份不可主動套用公式,但要做對帳驗證** (V0.5.1 教訓)
    - 症狀:V0.5.0 編輯歷史月份時,formula 帳戶仍 readonly + 跟著公式跑,結果 0.0003 萬的浮點誤差會「污染」歷史既定事實
    - 原因:V0.5.0 沒區分「新增模式」和「編輯模式」的 formula 行為
    - 做法:V0.5.1 把規則改清楚 ——
      - **新增模式**:formula readonly + 自動算 + 寫入 balances
      - **編輯模式**:formula 變可手改 + 顯示「期望 vs 實際」對帳資訊(綠 ✓ 吻合 / 紅 ⚠ 不符)
      - **匯入/載入歷史**:公式完全不介入
      - 公式的角色從「歷史改寫工具」降級為「對帳驗證工具」,既定事實永遠優先

16. **雲象重複計算總資產** (V0.6.0 教訓)
    - 症狀:V0.5 把雲象當 me 的資產,加總後 2025/09 的「總資產」算成 6861(Excel 是 6695),多了一個雲象 165 萬
    - 原因:雲象的市值已經包含在某個 stock 帳戶裡(Sela 自己知道是哪個),Excel 那個雲象欄位只是「公式參考用」
    - 做法:加新的 `side='reference'`,`computeStats()` 看到 reference 跳過。雲象保留在帳戶清單(這樣 mantou 公式才有依賴目標),但不計入 assets/liab

17. **欠饅頭跟正規貸款餘額分開算** (V0.6.0 教訓)
    - 症狀:V0.5 把 mantou 當 liab 算,結果 I17 退休試算公式的 D4 跟 Excel 差 11 萬
    - 原因:Excel D4=SUM(B2:B3) 只含玉山貸款+台新車貸,不含 mantou(因為 mantou 是雲象連動衍生計算,不是固定貸款)
    - 做法:`computeStats()` 算兩個欄位 —— `liab`(全部負債,給負債比和總資產用)、`normal_liab`(排除 `auto.type==='formula'` 的負債,給退休試算 I17 用)

18. **菜石差額 = Dora - Leo,不是 Dora - (Leo+Aaron)/2** (V0.6.0 教訓)
    - 症狀:V0.5 用「Dora vs Leo+Aaron 平均」算菜石差額,跟 Excel G13 = F13-F14 差距很大
    - 原因:「雙石」的「石」原來只指 Leo,Aaron 不在這個對比裡
    - 做法:`gap = doraTotal - leoTotal`,UI 文字改「Dora 比 Leo 多」

19. **退休試算的 TODAY 用「快照日期」而非真正今天** (V0.6.0 設計選擇)
    - 症狀:Excel I11 = (生日-TODAY)/365,生日固定 2029/12/27,但 TODAY 是「Excel 打開的那天」
    - 原因:對歷史月份來說,如果用「真正今天」算,所有歷史月份的 I11 會一樣,沒有歷史意義
    - 做法:`computeYearsToRetireDate(snapshot.date)` 用快照日期,這樣 2024/01 的 I11=5.99 / 2025/01 = 4.99 / 2026/01 = 3.99,自然遞減,反映「那時候算出來的退休還差是多少」

20. **首頁帳戶列表太長,要改用摺疊群組** (V0.6.1 教訓)
    - 症狀:V0.6 首頁有 17+ 個帳戶從上往下列,Sela 在手機上要滑半天才能看完
    - 原因:UX 預設「全展開」,但實際上 Sela 平常只想看加總,要看細項才會展開
    - 做法:V0.6.1 改用 `group-card` 摺疊,預設只顯示「分類加總 + 月變化」,點才展開細項
    - 細節:分組不只用 `category`,要用 Sela 的會計概念(例如台幣銀行排除南非幣和 cash_legacy,雲象獨立放「參考」群組)
    - 狀態:`EXPANDED_GROUPS = new Set()` 模組級變數,記錄哪些群組被展開,renderDashboard 重繪保留

21. **匯入腳本要自動偵測「歷史殘留帳戶」** (V0.6.1 教訓)
    - 症狀:V0.6 把 cash_legacy 留 active,結果它出現在「銀行存款」群組,但金額永遠 0
    - 原因:匯入腳本沒檢查「最近月份還在用嗎」
    - 做法:檢查最近 6 個月的 snapshots,如果某帳戶都沒值,自動 `active=false`
    - 已自動停用的:cash_legacy, fubon_loan, xt_loan, jh_loan

22. **手機數字輸入框三重 bug:擠、失焦、不觸發** (V0.6.2 教訓)
    - 症狀:Sela 在 iPhone 開設定頁改貸款月繳,輸入框被擠到 50px 寬看不清楚;打字打到一半輸入框會跳出來;有時改完數字沒反應
    - 原因(三個各自獨立):
      - **CSS**:`setting-row` flex 排版 + label 文字太長,輸入框被壓
      - **renderAll**:`oninput` 每次重繪整個設定頁,DOM 重建讓 input 失焦
      - **onchange**:手機數字鍵盤要按「完成」才觸發 onchange,使用者以為沒反應
    - 做法:
      - 貸款月繳改用 `loan-pmt-card`(獨立卡片,輸入框佔整行)
      - 用 `oninput` 即時觸發(每按一鍵就存)
      - 加 `inputmode="decimal"` 開數字鍵盤
      - 加 `onfocus="this.select()"` 自動全選方便覆寫
      - `updateSetting/updateLoanPmt` 移除 `renderAll()`,改靠 `switchPage` 切回 dashboard 時自動重繪
      - 例外:`updateLoanPmt` 還是更新一下「合計」顯示(用 `id="loan-pmt-sum"` 局部更新,不影響輸入框)

23. **設定的 loan_pmt 跟 snapshot.tesla_pmt_wan 要避免 double-count** (V0.6.2 教訓)
    - 症狀:V0.6.2 加台新車貸進 settings 後,2026/03 的「貸款月繳」可能變 31.66 萬(多算一次 Tesla)
    - 原因:V0.6 的 `computeTotalLoanPmt` 是「settings 全加 + snapshot.tesla 也加」
    - 做法:遍歷 settings.loan_pmt_wan,若該筆名稱是「台新車貸」且 snapshot 有 tesla_pmt_wan,**用 snapshot 值取代**(該月實際金額優先)
    - 結果:2026/03 = 29.52 ✓ / 2026/01 = 29.48 (snapshot=2.1 取代 settings=2.1429) ✓

24. **「凍結機制」要分清楚最新跟非最新** (V0.7.0 教訓)
    - 症狀:Sela 想要「按確定後就變成純數字記錄,不再被公式變動影響」
    - 原因:V0.6 所有衍生數字都用當前 settings/公式即時算,改 settings 會回頭影響所有歷史
    - 做法:加 `snapshot.computed` 區塊存凍結值,加 `getStats()` 包裝函式:最新月份即時算 / 非最新讀 computed / 沒 computed 就 fallback computeStats
    - 凍結時點:儲存「下一張」資產表時,凍結「上一張」(因為它再也不會變最新)
    - **不要改 `computeStats` 本身** — 它仍然是純算函式,給「活的最新月份」和「逃生口重凍」用

25. **autoFreezeOldSnapshots 用「當下 settings」凍結歷史 — 是副作用不是 bug** (V0.7.0 教訓)
    - 症狀:V0.7 第一次啟動,2025/09 月支出凍結成 57.52 萬(包含當下 settings 的台新車貸 2.1429),但 2025/09 那時根本沒有車貸,Excel 顯示 55.38 萬
    - 原因:啟動時用當前公式 + 當前 settings 算,所以「歷史月份用了還沒發生的車貸」
    - 做法:**接受這個副作用**,理由是修正成本太高(要從 Excel 逐月推算當時的 settings 列表),而且 Sela 的需求是「以後不變」而不是「已發生的不影響」
    - 邊界:Sela 若要修,可以編輯該月→改 frozen_settings 的快取→重凍,或用 forceRefreezeAll 在改設定後重凍(但會再次用當下 settings)

26. **編輯非最新月份不能跑公式重算** (V0.7.0 規則)
    - 症狀:V0.6.x 的 saveSnapshot 不管編輯誰都用 computeStats,結果改個小數字會把整個 computed 重算成「當前公式 ✕ 該月 balances」
    - 原因:V0.6 沒有「凍結」的概念
    - 做法:saveSnapshot 編輯模式分三類:
      - 編輯**最新**月份 → 不寫 computed(讓它保持活的)
      - 編輯**非最新已凍結** → 保留原 computed(只改 balances)
      - 編輯**非最新但還沒凍結**(舊資料) → 用當前公式凍結一次
    - 邊界:這代表編輯舊月份的 balances 不會反映在「離退休還差」之類的衍生欄位,Sela 知道這是 by design

27. **localStorage 升級時不會自動補上新欄位** (V0.7.1 教訓)
    - 症狀:Sela 從 V0.6 升 V0.7,手機開設定頁完全看不到任何貸款項目(預載的 6 筆消失)
    - 原因:`loadData()` 從 localStorage 拿到舊 settings(沒 loan_pmt_wan),直接 return,DEFAULT_DATA 的預載被忽略
    - 做法:V0.7.1 加 `mergeSettings(defaults, stored)`:用 stored 為主、defaults 補齊缺漏。空陣列也視為「缺漏」改用 defaults
    - 邊界:**不要直接覆蓋 stored 設定** — Sela 之前改過的值要保留(例如她把 qyld_yield 改成 0.06,升級後仍應是 0.06,不該被 reset 成 default 的 0.0739)
    - 副作用:升級後第一次打開,空的設定欄位會自動填上預設值,Sela 看到的不是「空白」而是「預載」

28. **貸款月繳要可增刪改** (V0.7.1 設計補充)
    - 症狀:V0.6/V0.7 的貸款月繳是寫死 6 筆,使用者無法增加/刪除
    - 原因:當時為了省事直接 hardcode
    - 做法:V0.7.1 改成完全可編輯
      - 名稱可改(`<input type="text">`)
      - 繳款日可改(1-31 數字輸入框)
      - 可刪除(每張卡片右上 ✕,confirm 後刪)
      - 可新增(底部「+ 新增貸款」虛線按鈕,點下加一筆「新貸款 0 萬/月,1 號」)
    - 注意:`computeTotalLoanPmt` 仍以「名稱=台新車貸」當判斷,**如果使用者改名,Tesla 智慧合併會失效**。可接受,因為使用者改名通常是有意為之

29. **首次對齊 SELA-Starter-Kit V1.8.0** (V0.8.0 教訓 + 紀錄)
    - 症狀:Ledger 跑了 7 個版本沒整合 SELA 品牌資產(沒 logo、沒 favicon),favicon 是用土法 inline `data:` URL 寫了個 serif「資產」中文字
    - 原因:Ledger 起源於 V0.1 MVP 時還沒接 Kit,Kit V1.8.0 才出 + Sela 才提供 starter kit zip
    - 做法:走 Kit cross-project-pitfalls #40 的「對齊既有專案 SOP」(四級分類法):
      - 🔴 鐵律:加 favicon 整套、`<head>` 改用 SELA 標準 favicon 引用、`.gitignore` 補齊 Kit 標準項目、README 加 SELA logo banner、產出 SELA-handoff.md
      - 🟡 建議:CLAUDE.md 開頭加 Kit 衝突仲裁區塊
      - 🟢 順便:topbar 版本號改 v0.8.0、PWA manifest name 從「資產札記」改「資產札記 Ledger」對齊 README
      - ✗ 不做:配色保留(個人選擇,已驗證 7 版)、CLAUDE.md 章節結構保留(已符合章法手冊)、帳戶 ID 中拼混合保留(對映 Excel 業務術語)
    - 重點教訓:**Kit 規範不是法律**,衝突時以本專案 CLAUDE.md 為主、Kit 為輔。但「鐵律」(zip 命名、必含 .gitignore、必含 SELA logo)違反就壞,沒得商量
    - 影響範圍:純對齊版,**Excel 12 條公式對齊不變**、凍結機制不變、所有業務邏輯不變 — 只動了品牌資產、文件結構、版本號顯示

30. **import_excel.py 漏抓 F4(monthly_kid_wan)和 I2(tesla_pmt_wan)** (V0.8.1 教訓)
    - 症狀:V0.8.1 重新匯入新版 Excel 時,所有 39 筆既有有 monthly_kid_wan 的快照都被清成 None,變相弄丟一菜雙石歷史資料
    - 原因:V0.6 把 `monthly_kid_wan` 加進資料模型時,只在前端 saveSnapshot 寫,但 import_excel.py 的 `extract_yunxiang_and_qyld()` 沒同步加 F4 / I2 讀取邏輯。**「資料模型 ↔ 匯入腳本」沒對齊**(類似 Kit 坑 #1 的三方對齊事故)
    - 做法:`extract_yunxiang_and_qyld()` 補上 F4(一菜雙石+儲備 → monthly_kid_wan)和 I2(特斯拉車貸,元 ÷ 10000 → tesla_pmt_wan),main() 把這兩個欄位寫入 snapshot
    - 邊界:`extract_yunxiang_and_qyld` 命名已不準確(現在抓 4 個欄位),但保留命名避免改太多。下版有空可重命名為 `extract_dynamic_fields()`
    - 對齊驗證:修補後重匯一次,跟 V0.8.0 既有 ledger-history.json 比對,**0 筆差異**(除了新月份 2026-05)

31. **holdings 升級補齊要深拷貝,否則 saveData 寫穿 DEFAULT_DATA** (V0.9.0 教訓)
    - 症狀:V0.8.x 升 V0.9.0 時,舊 localStorage 沒有 `holdings` 子物件,要從 `DEFAULT_DATA.holdings` 補進來
    - 原因:如果直接 `stored.holdings = DEFAULT_DATA.holdings`,兩者共用同一物件記憶體;之後 `saveData()` 把 stored 寫回 localStorage 雖然不影響 DEFAULT_DATA(常數已被序列化覆蓋),但**同一個 session 裡 stored.holdings 跟 DEFAULT_DATA.holdings 是同個 reference**,改一邊另一邊也變
    - 做法:`loadData()` 用 `JSON.parse(JSON.stringify(DEFAULT_DATA.holdings))` 深拷貝再賦值,徹底切斷 reference
    - 邊界:這個坑只在「新欄位首次出現」時會踩(每次 schema 加 root-level 物件都要深拷貝補齊)。坑 #27 mergeSettings 處理 settings 子欄位是同一精神
    - 同類保險:V0.9.x 之後若 holdings 又加新子欄位(例如 `holdings.alerts`),loadData 還要再加更細的子欄位補齊邏輯,不能假設「holdings 存在就什麼都有了」

32. **zip 命名要照 Kit 鐵律 #0:用專案名 + 空格分隔 + 點版號** (V0.9.0 教訓)
    - 症狀:V0.9.0 連續打錯兩次。第一次 `Ledger_V0_9_0.zip`(底線錯),Sela 指出「給檔格式沒有照規範」;第二次自作主張把專案名翻成中文 `資產札記 V0.9.0.zip`,Sela 再指「程式就叫 Ledger,不要任意改名」
    - 原因:Kit 鐵律 #0 寫「`<專案名稱> V<版本>.zip`」但沒明寫「專案名 = 給我的 zip 檔名前段」。我看到 README/index.html 的「資產札記 Ledger」雙語並陳,以為中文是「正式名」 — **錯**。專案名以 Sela 上傳的 zip 檔名為準(`Ledger_V0_8_x.zip` → 程式叫 Ledger),資料夾內容名(中文「資產札記」)是 UI 品牌、不是專案名
    - 做法:**本專案 zip 名 = `Ledger V<點版號>.zip`**(英文 Ledger、空格、V 大寫、版號用點)
    - 反例:`Ledger_V0_9_0.zip`(底線)、`Ledger-V0.9.0.zip`(連字號)、`資產札記 V0.9.0.zip`(改名)、`ledger v0.9.0.zip`(小寫)、`Ledger V0.9.zip`(版號漏一位)— 全錯
    - 打包指令:`zip -r "Ledger V$VER.zip" 資產札記` — 注意 zip 檔名是 Ledger、但壓縮的資料夾名是「資產札記」(這兩個不一樣,別混)
    - 對照 Kit `CLAUDE.md` §3「關於交付」:「✗ 不用 `<名稱>_V1.0.0.zip`(底線)或 `<名稱>-V1.0.0.zip`(連字號)——用空格分隔」

---

## 五、關鍵路徑(改 X 功能動哪幾個地方)

單檔 HTML,所有東西都在 `index.html`(部署時)/ `index_v4.html`(開發時)。用 `grep -n` 找以下標記快速定位:

| 改什麼功能 | 動哪幾段(用 grep 標題找) |
|---|---|
| **新增/改帳戶** | `DEFAULT_DATA.accounts`(HTML 內) + `ACCOUNT_DEFS`(import_excel.py) + `NAME_TO_ID`(import_excel.py) |
| **改總資產/負債比公式** | `function computeStats(` 一處,所有頁面都讀它 |
| **改密碼** | `const PASSWORD_HASH = '...'` + 重算 SHA-256(`hashlib.sha256("xxx".encode()).hexdigest()`) |
| **改登入時效** | `const LOGIN_EXPIRY_DAYS = 30` |
| **改匯入合併邏輯** | `function mergeData(` |
| **改 dashboard 卡片** | `function renderDashboard(` |
| **改歷史頁圖表** | `function renderHistory(` + `function renderLineChart(` |
| **改代管頁** | `function renderCustodial(` |
| **改月輸入流程** | `function renderSnapshot(` + `function goToSnapshot(` + `function saveSnapshot(` |
| **改攤還公式** | `function calcAmortizeNext(` |
| **改設定頁** | `function renderSettings(` + `function renderHoldingsSettings(` |
| **改持股 tab(V0.9.0)** | `function renderHoldings(` + `function renderHoldingsBroker(` + `function renderRebalance(` |
| **改加權年化演算法** | `function computeBrokerStats(` |
| **改股價 API fallback 順序** | `async function fetchQuote(` |
| **改持股對齊 Ledger 邏輯** | `function computeLedgerAlignment(` |
| **改幫助說明** | `function openHelpModal(` |

`renderAll()` 把四個 render 都跑一次,儲存後一律呼叫它。**不要嘗試做局部更新**,這專案規模不值得。

---

## 六、煙霧測試(每次改完必跑)

```bash
# 1. JS 語法檢查
python3 -c "
import re
with open('index_v4.html') as f: html = f.read()
m = re.findall(r'<script>(.*?)</script>', html, re.DOTALL)
open('/tmp/check.js','w').write(m[0])
" && node --check /tmp/check.js && echo "✅ JS OK"

# 2. 密碼 hash 對得起來
python3 -c "
import hashlib
print(hashlib.sha256('Sela1227'.encode()).hexdigest())
" # 應為 2203bc4858946646c27e6930d6dd95b50f3e072873a5afcade1153259ef9a384

# 3. 匯入腳本能跑出對的數
cd /home/claude/asset-tracker
python3 import_excel.py ../assets.xlsx /tmp/test.json
python3 -c "
import json
d = json.load(open('/tmp/test.json'))
me = {a['id']: a for a in d['accounts'] if a['owner']=='me'}
sep = next(s for s in d['snapshots'] if s['date']=='2025-09-01')
total = sum(sep['balances'].get(i,0) for i in me)
assert abs(total - 6695.58) < 0.01, f'2025/09 total={total}, want 6695.58'
print('✅ 2025/09 總資產對齊 Excel')
"

# 4. 手動驗證(瀏覽器):打開 index → 輸密碼 → 設定頁載入合併 → 歷史頁應該看到 2019/11 ~ 2026/03 完整圖表
```

---

## 七、版本歷程(只留近期)

- **V0.9.0** — 持股 Tab(美股配置追蹤+再平衡):加 `DATA.holdings`(brokers/positions/target_allocation_schwab/cash/fxrate/fund_flows/quotes)、加「持股」整個新 tab(全部/嘉信/第一/永豐 4 子 tab,核心/衛星/個股/投機 4 tier 摺疊群組,再平衡建議,對齊 Ledger 警示,加權年化報酬)、加 Yahoo + 兩層反向代理股價 API、設定頁可編輯股數/現金/匯率/預定%/資金流、加 `import_holdings.py` 從 Excel 匯入大量資金流。td_us 改名「美國券商」(嘉信+第一兩家合計)。再平衡簡化成 7 檔(原設計 9 檔)合計 66%。看升版必讀 + 坑 #31
- **V0.8.2** — 設計文件版:加 `V0.9.0-DESIGN.md`(完整持股 tab 設計,給下次對話接手 V0.9.0 用),不動程式。本版理由:V0.9.0 規模大(~+1200 行)且本對話 context 已用很多,選擇先把設計沉澱成文件,避免半成品;設計含三家券商架構、加權報酬演算法、股價 API 三層 fallback、Excel 兩份匯入規則、Sela 5 輪討論的決策追溯
- **V0.8.1** — 資料更新 + 修 import bug:加入 2026/05 月份(58 筆)、修 import_excel.py 沒抓 F4(`monthly_kid_wan`)和 I2(`tesla_pmt_wan`)的 bug,未來重匯 Excel 不會再清空這兩欄(看坑 #30)
- **V0.8.0** — 首次對齊 SELA-Starter-Kit V1.8.0:加 favicon 套組(svg/png/ico/manifest)、index.html `<head>` 改用 SELA 標準引用、README 加 SELA logo banner、`.gitignore` 補 Kit 標準項目、產出 SELA-handoff.md、CLAUDE.md 開頭加衝突仲裁區塊。配色/章節結構/業務邏輯全部保留(看坑 #29)
- **V0.7.1** — 修正 V0.7 升級 bug:`loadData` 加 `mergeSettings` 補齊缺漏設定;貸款月繳改全可編輯(名稱/繳款日/增/刪);Topbar 加版本號 pill
- **V0.7.0** — 凍結機制:加 `snapshot.computed` 存凍結值,`getStats()` 包裝(最新→即時/非最新→讀凍結),`saveSnapshot` 儲存下一張時凍結上一張,啟動時自動凍結舊資料,設定頁加「強制重新凍結」逃生口
- **V0.6.2** — 手機 UX 修正:貸款月繳改卡片式 UI、所有設定輸入改 `oninput` + `inputmode="decimal"`、`updateSetting` 不再 renderAll(避免輸入框失焦);settings 加台新車貸,snapshot.tesla_pmt_wan 覆蓋 settings 值
- **V0.6.1** — 首頁 UX 重構:帳戶列表改摺疊群組(預設加總、點開看細項)、自定義分組對齊 Sela 會計概念、自動停用 4 個歷史殘留帳戶
- **V0.6.0** — 退休試算重構:對齊 Excel 12 條公式;加 reference side(雲象)、normal_liab、settings.salary/card/loan_pmt_wan/birthday/retire_extension;快照加 monthly_kid_wan/tesla_pmt_wan
- **V0.5.1** — 公式語意修正:編輯舊月份不主動套用公式,改顯示「期望 vs 實際」對帳資訊

---

## 八、下版候選工作

按優先序:

1. **V0.9.x — Sela 補資料、實機測整套** — V0.9.0 上線後第一優先:Sela 補嘉信 VPL/VWO/O 股數、補嘉信「個股 / 投機」其他 ~10 檔、用 `import_holdings.py` 匯入 89 筆 fund_flows、實機看股價 API 哪一層 fallback 在用。動工前必問:股價有抓到嗎?Yahoo 直連被擋還是過了?
2. PWA 在 iPhone 實機驗證加主畫面流程(從 V0.8.x 起一直延後,V0.9.0 業務告一段落後該補)
3. 寫 Service Worker 做離線快取
4. 圖表加 tooltip(點/hover 顯示該月詳細數字)
5. 持股歷史走勢圖(每次 refreshAllQuotes 把 today_market_usd 寫進新表 `holdings_snapshots`,看走勢)
6. 雲象的「淨資產收益率」獨立指標(雲象 × 14/15 的累積走勢)
7. 加「年度報表」分頁:每年資產增減、報酬率、配息估算
8. 攤還參數可在 UI 改
9. 匯出 CSV
10. 多語系

### 不在優先序裡:隱私強化(Cloudflare Access / 抽歷史出程式)

**Sela 在 V0.4.0 已明確接受公開 GitHub Pages + 內嵌歷史的隱私風險**,理由是:家人朋友非技術背景、repo 名混淆、SHA-256 hash 擋一般人夠用。**所以不要主動建議升級**。

但下面三個情境出現任一個,**這項自動升到第 1 名**:

- Sela 主動提到「我發現有人看到我的網址了」或類似擔憂
- Sela 把 repo 改名成跟資產有關的名字(代表她不再在意混淆)
- Sela 在 commit/issue 裡提到「給朋友看了」「給某某用了」(代表分享範圍擴大)

升級時的兩條路:
- **快**:把 `EMBEDDED_HISTORY` 內容清空(改成 `{ snapshots: [], accounts: [] }`),Sela 改用手動匯入 `ledger-history.json`
- **正規**:遷移到 Cloudflare Pages + Access,部署流程改成連 Cloudflare 而非 GitHub Pages,DEPLOY_GITHUB.md 要改寫成 DEPLOY_CLOUDFLARE.md

---

## 九、一句話總結

V0.9.0 業務增量大版:加「持股」整個新 tab(嘉信+第一+永豐三家美股配置追蹤)、加權年化報酬演算法用合成資料反推 Excel 對齊(8.40% / 4.67%)、Yahoo + 兩層反向代理股價 API、嘉信再平衡建議簡化成 7 檔(VTI/VGK/VPL/VWO + VDE/ICLN/URNM)合計 66%、設定頁全可編輯股數/現金/匯率/預定%/資金流、加 `import_holdings.py` 從 Excel 匯入 89 筆歷史資金流。td_us 改名「美國券商」(實涵嘉信+第一)。加坑 #31:loadData 補齊 holdings 必須深拷貝 DEFAULT_DATA(否則 reference 穿透)。下版第 1 優先:Sela 實機看股價 API 哪一層 fallback 在用 + 補 VPL/VWO/O 股數和其他 ~10 檔個股/投機。
