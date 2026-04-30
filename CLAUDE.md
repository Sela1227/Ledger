# CLAUDE.md — 資產札記 Ledger

> 給下次接手的 Claude:讀完這份就能直接動手,不用問 Sela 已經決定過的事。
> 這個專案是 Sela 個人用的資產追蹤工具,單檔 HTML,部署在 GitHub Pages。

---

## ⚠️ V0.5.1 升版必讀

V0.5.1 是 V0.5 的**規則修正版**,改動很小但語意重要 —— **公式從「強制套用」降級為「對帳驗證」**。看坑 #15。

新規則一覽:

| 場景 | formula 帳戶行為 |
|---|---|
| 新增資產表(下個月) | readonly + 自動算 + 寫入 |
| 編輯舊月份 | 可手改 + 顯示「期望 vs 實際」對帳(綠 ✓ / 紅 ⚠) |
| 匯入歷史 / 載入內嵌 | 完全不介入,既定事實優先 |
| 歷史頁查看 | 不介入 |

V0.5.0 的破壞性改動:退休目標自動算、每快照存 yld、show_inactive toggle、按鈕「快照→資產表」,V0.5.1 全部繼承。

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
  settings: { retirement_target_wan, qyld_yield, monthly_expense_wan },
  accounts: [{ id, name, category, side, liquid, owner, active, auto? }],
  snapshots: [{ date: 'YYYY-MM-DD', balances: { acc_id: number_in_wan } }]
}
```

**金額單位永遠是「萬」**(跟 Sela Excel 一致)。負債用負數。Snapshots 不要存衍生欄位(總資產、負債比都用算的)。

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

下面這些是 Sela 個人真實情況,接手後**不要當成資料異常去處理**:

- **2023 整年沒資料** — Sela 那年沒記錄,Excel 從 2022/10 直接跳到 2024/01。圖表的 X 軸要能容忍中間斷檔,**不要寫程式去「補插值」或「警告缺漏」**。
- **未來月份(2026/01–03)是預估** — Excel 居然有未來資料,因為 Sela 拿攤還公式預先填了「估算未來」。**保留**,別主動提醒「這是未來資料」,Sela 自己知道。
- **「玉山賃款」是筆誤** — Excel 2025/04 之後出現「玉山賃款」(賃 ≠ 貸),是 Sela 打錯沒改回來。匯入腳本已對應到 `yushan_loan`,**UI 永遠顯示「玉山貸款」**,不要顯示「賃款」。
- **2025/05 負債從 -2942 跳到 -3407** — 不是資料錯誤,是 Sela 那月新增了一筆貸款。
- **2026/01 負債多 158 萬** — 也不是錯誤,是台新車貸出現。
- **雲象是私募股權投資,饅頭出資 1/15 與 Sela 共持** — `mantou = -yunxiang/15` 是業務勾稽,不是借貸關係。雲象漲 → 饅頭那份也漲 → 欠饅頭數字變大。**淨資產收益 = 雲象 × 14/15**(扣除饅頭那份)。V0.5 起 mantou 用 `auto.formula` 自動算,不可手改。
- **雲象從 2025/07 起在 Excel 的 C21 欄位**(不是原本 A 欄位置)— 匯入腳本已處理,讀 A 欄找不到雲象不要當 bug,要去抓 C21。
- **QYLD 年化報酬率每月在變動** — Sela 每月 1 號實際去查 QYLD 近 5 年含息年化,寫進 Excel 的 D19。app 端也是每快照記一次,別假設它是固定值。

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
| **改設定頁** | `function renderSettings(` |
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

- **V0.5.1** — 公式語意修正:編輯舊月份不主動套用公式,改顯示「期望 vs 實際」對帳資訊(✓/⚠);新增模式維持 readonly 自動算
- **V0.5.0** — 退休目標改自動算(月支出×12÷yld)、每快照存當月 yld、mantou 改 formula 自動算、加 show_inactive_accounts toggle、按鈕「快照」改「資產表」、匯入腳本支援 C21 雲象 + D19 yld
- **V0.4.0** — 加密碼登入(SHA-256)、內嵌歷史改顯式載入、匯入改合併、清空保留帳戶結構、設定頁加登出按鈕、Help modal 重寫
- **V0.3.0** — Excel→JSON 匯入腳本完成,57 筆歷史正確匯入,內嵌進 HTML,加 PWA manifest
- **V0.2.0** — 字體換 Noto Sans TC + JetBrains Mono、加歷史頁(區間/指標切換)、加 SVG 走勢圖、加代管頁三人疊加圖、加 Help modal、加快照編輯
- **V0.1.0** — MVP:卡片式輸入頁、dashboard、代管分頁、設定、localStorage 儲存、攤還自動扣

---

## 八、下版候選工作

按優先序:

1. **正式部署到 GitHub Pages 並在手機驗證 PWA 加主畫面** — 上線前必備:Sela 還沒實際在 iPhone Safari 上測過「加入主畫面」流程
2. 寫 Service Worker 做離線快取
3. 圖表加 tooltip(點/hover 顯示該月詳細數字)
4. 雲象的「淨資產收益率」獨立指標(雲象 × 14/15 的累積走勢)
5. 加「年度報表」分頁:每年資產增減、報酬率、配息估算
6. 攤還參數可在 UI 改
7. 匯出 CSV
8. 多語系

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

V0.5.1 公式語意修正版:Sela 確認「過去資料是既定事實,公式只是內部驗證用」,所以編輯歷史月份時 formula 不主動覆寫,改顯示「期望 vs 實際」對帳(吻合綠✓、不符紅⚠);新增資產表時公式仍是 readonly 自動算。下版第一優先仍然是把 PWA 在 iPhone 實機跑一次驗證加主畫面流程。
