# SELA-handoff.md — 資產札記 Ledger

> 這份是 Ledger **首次對齊 SELA-Starter-Kit V1.8.0** 的紀錄(V0.8.0 產出)。
> 依 Kit `templates/SELA-handoff-template.md` 規範,給未來 Kit Claude 升 Kit 時參考。

---

## 〇、專案速覽

- **專案名稱:** 資產札記 Ledger
- **專案類型:** 純靜態網頁 / 單檔 HTML(GitHub Pages 部署)
- **技術棧:** 純 HTML + JS + CSS,無 build step,localStorage 儲存,內嵌 SHA-256 密碼登入
- **規模:** 1 個主檔 `index.html` ~3155 行(含 CSS + JS + 內嵌歷史 JSON)
- **使用 Kit 版本:** V1.8.0(首次對齊)
- **完成版本:** V0.8.0(對齊版,業務邏輯零變動)
- **完成日期:** 2026-05-08
- **歷史:** Ledger 從 V0.1.0 跑到 V0.7.1 共 12 個版本後才接 Kit

---

## 一、用 Kit 的整體感受

### 預期外的順利

- **#40「對齊既有專案 SOP」非常好用** — 四級分類法(🔴/🟡/🟢/✗)清楚可操作,Ledger 用了不到 30 分鐘就盤點完所有候選改動
- **「鐵律最小對齊清單」六項清楚** — 一眼看出 Ledger 缺 favicon、缺 SELA logo banner、`.gitignore` 不全
- **Kit 衝突仲裁區塊範例**(claude-init.md §二)直接可套用,放進 Ledger CLAUDE.md 開頭就解決「下次 Claude 會不會又想改回 Kit 預設」的擔憂
- **Logo CLAUDE.md §4.1 標準片段**直接 copy-paste,不用想 favicon 怎麼引用

### 預期外的卡住

- **Logo CLAUDE.md §4.1 寫 `theme-color: #F36825`,但 Ledger 配色不是 SELA 主題** — Kit 預設 theme-color 跟 logo 鐵律(永遠橘+白)綁在一起,但實際上 theme-color 是 PWA 啟動畫面色,跟 logo 顏色不必然一致。Ledger 黑底金字 app 用 SELA 橘做啟動畫面會很違和。**目前用「✗ 不做」+ 明寫理由**處理,但這算 Kit 的隱性假設(沒明寫但實際會踩)
- **Kit 預設 `site.webmanifest` name = "SELA"** — 但實際每個專案應該客製成自己的名稱。Kit 沒明說「這個 webmanifest 要客製 name/short_name」。我自己改了,但下個 Claude 可能會直接用預設不改
- **「對齊既有專案」的版本號該怎麼跳沒明寫** — Ledger 從 V0.7.1 跳 V0.8.0(我判斷「加 favicon + handoff 算加功能」),但 cross-project-pitfalls #40 沒給範例。同類型情況「Kit 對齊算 b+1 還是 c+1」可以加範例

### 對 Kit 的整體評價

- ✓ **#40 對齊 SOP** + **claude-init.md §二** 是 V1.8.0 的關鍵新增,Ledger 是第一個受惠的專案
- ✓ **章法手冊**(`CLAUDE-MD-章法.md`)規範了 Ledger 從 V0.1 自然演化出的寫法,證明這套章法不只我一個人用得通
- ✗ **Logo theme-color 跟主題色綁定的隱性假設**沒寫清楚,建議補強(見二、強烈建議加坑 #1)

---

## 二、發現的「跨專案通用坑」(建議進 Kit)

### 提案前的檢查

**檢查 1 — grep Kit 結果**:
```bash
grep -rn "theme.color\|theme_color" SELA-Starter-Kit/conventions/ SELA-Starter-Kit/logo/
```

結果:Kit logo/CLAUDE.md §4.1 + §4.8 + colors.md 都寫 `theme-color: #F36825`(SELA 橘),沒任何地方提「PWA 啟動畫面色 vs Logo 顏色不必然一致」。

**檢查 2 — 踩坑 vs 速查表**:這是「事後反思」(我先用 SELA 橘 → 發現啟動畫面違和 → 改成 app 主題色),所以進坑庫格式。

### 強烈建議加坑

#### 1. PWA `theme-color` ≠ Logo 主色

- **症狀**:照 Kit logo/CLAUDE.md §4.1 設 `<meta name="theme-color" content="#F36825">`,結果非 SELA 主題色的 app(如 Ledger 黑底金字)在 iPhone Safari「加入主畫面」啟動時,白底→橘底→app 黑色背景的閃跳體驗很糟
- **原因**:Kit 把「logo 永遠橘+白」(品牌鐵律)跟「`theme-color` 用橘」(隱性綁定)寫在一起,沒區分「品牌色」和「PWA 啟動畫面色」是兩個概念
- **做法**:
  - **Kit 規範改成**:`theme-color` 用 **app 主題色**(也就是 `--bg` 對應色),而非永遠用 SELA 橘
  - SELA 主題的 app:`theme-color: #F36825` 沒問題
  - 北歐霧藍主題的 app:`theme-color: #5A7A8B`
  - 自訂主題的 app:用該 app 自己的背景色
  - **`<meta name="theme-color">` 跟 `site.webmanifest` 的 `theme_color` 都要用 app 主題色,保持一致**
- **影響範圍**:所有非 SELA 主題色的 PWA(目前只有 Ledger 黑底金字算這類,但未來會有更多 — 任何「沉浸式深色 app」都會踩)
- **證據**:Ledger CLAUDE.md 衝突仲裁第 2 條 + V0.8.0 對齊紀錄
- **檢查 1 結果**:Kit 已有 logo/CLAUDE.md §4.1 寫死 `#F36825`,建議補強(不要全砍,要加分支判斷)

#### 2. 對齊既有專案的版本號跳法沒範例

- **症狀**:V0.7.1 跳 V0.8.0(對齊 Kit V1.8.0)時,我自己判斷「加 favicon + handoff 算加功能」走 b+1。但 #40「對齊既有專案 SOP」沒給範例,下個專案 Claude 可能跳得不一樣
- **原因**:#40 寫了「對齊」做什麼,但沒寫「對齊」算哪種升版
- **做法**:在 cross-project-pitfalls #40「對齊流程」第 7 步加範例:
  - **純鐵律對齊**(只補 favicon、`.gitignore`)→ c+1
  - **加 SELA-handoff、衝突仲裁區塊、文件結構動較多**→ b+1
  - **跟著對齊也順手做業務變更**→ 看業務變更等級決定
- **影響範圍**:每個將來首次接 Kit 的既有專案
- **證據**:Ledger V0.8.0 的決策過程

### 可加但等更多證據確認

- **首次接 Kit 的專案,localStorage 升級風險** — Ledger 從 V0.7 升 V0.8 不動資料模型,但理論上「首次接 Kit」可能伴隨資料模型大改(對齊 Kit 範式),這時 localStorage 遷移需要小心。Ledger V0.7.1 已踩過一次坑(#27 mergeSettings),Kit 可以考慮加「對齊既有專案時,localStorage 遷移檢查清單」。**等下個有資料的既有專案接 Kit 再決定**

---

## 三、發現的「跨專案設計模式」

### 1. 「凍結機制」處理「歷史不被未來公式影響」

- **本案發生情境**:Ledger V0.7 加凍結機制 — 最新月份的衍生數字即時算,非最新月份儲存時凍結進 `snapshot.computed`,改設定不再影響歷史
- **可推廣的原則**:任何「**有時間序列、會累積歷史紀錄、業務規則會演進**」的 app 都會踩同樣需求(歷史報表 / 評估紀錄 / 計分系統等)。設計模式:
  - 衍生欄位分兩層:**即時算**(最新)+ **凍結存**(非最新)
  - 寫一個 `getStats(snapshot)` 包裝函式統一這個判斷
  - 加「強制重新凍結所有歷史」逃生口(不應隨便用,但要有)
- **代價 / 取捨**:
  - ✗ 失去「以後加新指標」的彈性(歷史月份沒有新欄位的數字)
  - ✗ 找到 bug 時也不會回頭修(舊月份停留在舊邏輯)
  - ✓ 但業主要的就是「歷史不變」,所以這個代價是必要的
- **建議寫入**:`sela-philosophy.md`(設計原則)或新增 `conventions/historical-data-pattern.md`(設計模式)

### 2. 「業務對齊驗證」當開發測試用

- **本案發生情境**:Ledger 開發過程中,每次改公式都用「Excel 2026/03 數字 vs 程式算出數字」做對照,12 條公式全部對齊到小數點後 4 位。這比寫 unit test 還快 + 更貼近業務
- **可推廣的原則**:當業務有「外部既存事實」(Excel / 紙本表格 / 別的系統)時,把該事實當「真值表」,程式算出的數字必須對齊。比 unit test 更直接(直接驗證業務正確,不只驗證程式正確)
- **代價 / 取捨**:
  - ✗ 只能驗證「外部既存事實涵蓋的範圍」,新功能還是要寫 unit test
  - ✓ 對「移植既有 Excel/紙本流程」的場景特別有效
- **建議寫入**:`tech-stack-lessons.md` 加「移植既有業務流程的測試模式」一節,或進 #40 對齊既有專案 SOP 的延伸

---

## 四、Kit 該瘦身或調整的地方

### Kit 規範修改建議

#### 1. logo/CLAUDE.md §4.1 — `theme-color` 寫法

- **現狀**:`<meta name="theme-color" content="#F36825">`(寫死 SELA 橘)
- **建議改成**:加一句「**`theme-color` 應為 app 主題色,非 SELA 主題的 app 用該 app 自己的 `--bg` 對應色**」+ 三個範例(SELA 主題 / 北歐霧藍 / 自訂深色)
- **理由**:見二、強烈建議加坑 #1

#### 2. cross-project-pitfalls.md #40 — 對齊版本號

- **現狀**:沒寫對齊算哪種升版
- **建議改成**:對齊流程第 7 步加範例(c+1 vs b+1 的判斷)
- **理由**:見二、強烈建議加坑 #2

### Kit 結構性建議

無。Ledger 是首次對齊,還沒體會到結構性問題。

---

## 五、留在這個專案、**不要回流 Kit** 的東西

> 這節避免 Kit Claude 把 Ledger 業務邏輯誤收進 Kit。

- **Excel 12 條公式對齊清單**(D1/D4/D6/D7/F2/F5/I9~I18) — Sela 個人 Excel 結構,不通用
- **「雲象 / 欠饅頭 = -雲象/15」業務規則** — 私募股權投資協議,Sela 個人
- **「2029/12/27 是 Sela 50 歲生日」** — 個人事實
- **帳戶 ID 中拼混合**(yunxiang/mantou/yushan_loan/taishin_car_loan) — Sela 個人 Excel 對映,不通用
- **凍結機制的具體實作**(`freezeSnapshot()` / `getStats()` 寫法) — 程式碼留專案。但**設計原則**可進 Kit(見三、設計模式 #1)
- **「黑底金字」配色實際 hex 值** — Sela 個人選擇
- **「菜石差額 = Dora - Leo」「雲象在台股」**等業務勾稽 — 個人家庭/投資結構
- **PWA 密碼用 SHA-256 hash 寫死在程式裡** — Ledger 接受公開 repo + 擋家人朋友的妥協,不通用(其他專案要看個資/規模)

---

## 六、Kit Claude 的建議行動清單

### 建議升 Kit 版本

- 主要建議是新增坑(#42、#43)+ 補強現有 logo/CLAUDE.md
- 不算大改,**建議走 V1.8.1**(b 不變,c+1)

### 必做

1. logo/CLAUDE.md §4.1 加「theme-color 用 app 主題色」分支判斷 + 三個範例
2. cross-project-pitfalls.md 加坑 #42(theme-color ≠ logo 色)
3. cross-project-pitfalls.md 加坑 #43(對齊版本號跳法)

### 暫緩

- 「凍結機制」設計模式(三 #1)— 等下個時間序列 app 接 Kit 再決定要不要進 sela-philosophy
- 「業務對齊驗證測試模式」(三 #2)— 等下個移植專案再決定

### 不做

- 把 Ledger 配色/業務術語/帳戶結構納入 Kit — 都是 Sela 個人事實,不通用

---

## 七、附:Ledger V0.8.0 對齊操作摘要

完整對齊清單(走 #40 SOP):

| 等級 | 動作 | 結果 |
|---|---|---|
| 🔴 鐵律 | 加 `favicon/` 整套(svg/多尺寸 png/ico/manifest) | ✓ 從 logo/favicon/ + logo/svg/ 複製 |
| 🔴 鐵律 | `index.html <head>` 改用 SELA 標準 favicon 引用 | ✓ 移除舊的 inline `data:` URL favicon |
| 🔴 鐵律 | `.gitignore` 補齊 Kit 標準項目 | ✓ 12 行 → 49 行,加 Flet/node/build/*.egg-info |
| 🔴 鐵律 | README 加 SELA logo banner | ✓ 重寫整個 README |
| 🔴 鐵律 | 產出 SELA-handoff.md | ✓ 本檔 |
| 🟡 建議 | CLAUDE.md 開頭加 Kit 衝突仲裁區塊 | ✓ 在開頭明列「不對齊 Kit 的 4 項 + 理由」 |
| 🟡 建議 | CLAUDE.md 加 V0.8.0 升版必讀章節 | ✓ |
| 🟡 建議 | CLAUDE.md 加坑 #29 紀錄這次對齊 | ✓ |
| 🟢 順便 | topbar 版本號 v0.7.1 → v0.8.0 | ✓ |
| 🟢 順便 | PWA manifest name 從「資產札記」改「資產札記 Ledger」 | ✓ |
| ✗ 不做 | 配色不對齊 Kit 預設(SELA 主題橘 / 北歐霧藍) | 理由:已驗證 7 版的個人選擇 |
| ✗ 不做 | `theme-color` 不用 #F36825 | 理由:配合 PWA 啟動畫面深色 |
| ✗ 不做 | CLAUDE.md 章節結構不重排 | 理由:已符合章法手冊 |
| ✗ 不做 | 帳戶 ID 不全英化 | 理由:對映 Excel 業務術語 |

**業務邏輯零變動**:Excel 12 條公式對齊不變、凍結機制不變、所有功能不變 — 只動了品牌資產、文件結構、版本號顯示。

---

> Made by **Sela** + Claude · 2026-05-08 · Ledger V0.8.0 首次對齊 SELA-Starter-Kit V1.8.0
