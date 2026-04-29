# 資產札記 v4 — GitHub Pages 部署指南

## 你拿到的檔案

- `asset-tracker.html` — 主程式(已內嵌密碼登入 + 6 年歷史資料)
- `import_excel.py` — Excel 匯入腳本(以後 Excel 還更新時用)
- `ledger-history.json` — 已轉好的歷史 JSON 備份
- `DEPLOY.md` — 這份文件

---

## 部署前重要說明

### 關於密碼保護

你設定的密碼是 **Sela1227**(僅你和我之間提到,在這份文件裡也只提這一次)。

程式裡只存了密碼的 SHA-256 hash,**不是明文**。但要老實告訴你:

- 一般人按 F12 看原始碼,只會看到 hash 不會看到密碼 ✅
- **稍微懂技術的人**可以用 hash 去比對常見密碼字典,如果你的密碼夠特別(像 Sela1227 這種有點隨機性的),很難被破 ✅
- **真正懂的人**可以拿走 hash 自己跑暴力破解,雖然 SHA-256 很慢但理論上做得到 ⚠

**結論**:這個密碼擋一般人(家人朋友、隨機路人)綽綽有餘,但不擋專業人士。如果未來你想要更強的保護,就改用 Cloudflare Pages + Access(我前一份 DEPLOY 寫的 B 方案)。

### 關於財務資料

你的所有金額**只存在你自己手機/電腦的瀏覽器 localStorage**,從來不會上傳到 GitHub 或任何伺服器。所以即使 repo 公開,別人能看到的只有:
- 程式碼(包括帳戶名稱清單,如玉山、Dora)
- 內嵌的歷史快照(因為你選擇了「歷史資料寫在程式裡」,**這部分會被一起部署上去公開**)

⚠️ **這點很重要**:你選的「歷史資料寫在程式裡」意味著程式碼裡確實有 57 筆真實金額。GitHub Pages 公開 repo 後**任何人下載 HTML 都能看到**。

如果不能接受,有三個解法,你可以再決定:
1. 把 repo 設為 **private**(需要 GitHub Pro $4/月才能搭配 Pages)
2. 改用 **Cloudflare Pages + Access**(免費,可加 email 登入)
3. **不把歷史寫進程式**,改成首次使用時手動匯入 `ledger-history.json` 一次(更隱密)

我把選項 3 也做進去了——你可以在程式碼裡刪掉 `EMBEDDED_HISTORY` 那一段,然後手動匯入 JSON 即可。

---

## GitHub Pages 部署步驟

### 1. 建立 GitHub repo

```bash
# 如果還沒有,建立資料夾
mkdir asset-tracker && cd asset-tracker

# 把 asset-tracker.html 複製過來,並改名為 index.html
cp ~/Downloads/asset-tracker.html ./index.html

# 初始化 git
git init
git branch -M main
git add index.html
git commit -m "init: asset tracker v4"
```

### 2. 在 GitHub 建 repo

到 https://github.com/new 建立新的 repo:
- **Repository name**: 隨便取,建議用看不出意思的名字如 `notes`、`utils`、`my-tools`
- **Public** 或 **Private** 看你的選擇(Public 才免費用 Pages)
- **不要**勾「Add a README」「Add .gitignore」(會跟你 local 衝突)

建立後把 local 推上去:

```bash
git remote add origin git@github.com:YOUR_NAME/YOUR_REPO.git
# 如果不用 SSH,改用 HTTPS:
# git remote add origin https://github.com/YOUR_NAME/YOUR_REPO.git

git push -u origin main
```

### 3. 啟用 GitHub Pages

1. 到你的 repo 頁面
2. 點 **Settings**(右上方)
3. 左側選單找 **Pages**
4. **Source**: 選 `Deploy from a branch`
5. **Branch**: 選 `main` / `(root)`
6. 點 **Save**
7. 等 1-2 分鐘,刷新頁面會看到 `Your site is live at https://YOUR_NAME.github.io/YOUR_REPO/`

### 4. 第一次開啟

1. 用手機 Safari/Chrome 打開那個 URL
2. 看到登入頁,輸入你的密碼
3. 登入後 30 天內免重輸(localStorage 記住)
4. 看到空白的 app
5. 到「設定」→「載入合併」→ 確認 → 看到 6 年歷史
6. iPhone:Safari 點分享 → 加入主畫面,變成 app 圖示
7. **馬上**到設定 → 匯出 JSON 備份一份到 iCloud Drive

### 5. 之後更新程式

```bash
# 換新版的 asset-tracker.html(假設你之後有調整)
cp ~/Downloads/new-version.html ./index.html
git add index.html
git commit -m "update: ..."
git push

# GitHub 會在 1 分鐘內自動重新部署
```

---

## 常見問題

### Q1: 為什麼程式打開後沒有歷史資料,要按按鈕才載入?

因為你要求「歷史資料直接寫在程式裡的載入歷史資料」——意思是資料**內建**但**不自動載入**,需要手動觸發。這樣的好處是:
- 你日常使用時不會被覆蓋(localStorage 裡的最新資料優先)
- 換手機/清資料後,可以一鍵恢復內建歷史
- 想看歷史時也可以手動再次載入(合併不會重複)

### Q2: 我輸入了 2025/10 快照後,如果再按「載入合併」會發生什麼?

不會覆蓋你的 2025/10。合併邏輯是「**以日期為鍵,新資料蓋舊資料**」,內嵌歷史最晚到 2026/03(預估的攤還資料)。如果你的 2025/10 是後輸入的、跟內嵌沒衝突,就會被保留。

### Q3: 匯出後再匯入,會發生什麼?

也是合併,同日期會以匯入檔的版本為準。所以**「匯出 → 編輯 JSON → 匯入」可以當成手動修資料的方法**。

### Q4: 換手機怎麼辦?

新手機開啟同一個 URL → 輸密碼 → 設定頁載入內嵌歷史 → 完成。如果你有最新的 JSON 備份,改成匯入該檔案,資料更新。

### Q5: 我忘記密碼了

把程式碼裡的 `PASSWORD_HASH` 那一行整段註解掉(`// const PASSWORD_HASH = ...`)再加 `const PASSWORD_HASH = '';` 就能空密碼進入。重新部署後進去把密碼改了即可。

### Q6: 我要更換密碼

1. 用 Python 算新密碼的 SHA-256:
   ```python
   import hashlib
   print(hashlib.sha256("新密碼".encode()).hexdigest())
   ```
2. 在 HTML 裡找 `PASSWORD_HASH = '...'`,把 hash 替換掉
3. git push 即可

或更簡單:打開 https://emn178.github.io/online-tools/sha256.html 輸入新密碼,複製 hash 換進程式。

---

## 隱私強化選項(未來想做的話)

### 選項 A:把歷史資料抽出程式

如果你不想讓內嵌歷史曝光,把 HTML 裡 `const EMBEDDED_HISTORY = {...}` 整段改成:

```javascript
const EMBEDDED_HISTORY = { snapshots: [], accounts: [] };
```

然後每次新手機要用,先手動匯入 `ledger-history.json` 一次。代價:多一個步驟。

### 選項 B:升級到 Cloudflare Pages

完全免費 + 加 email 驗證碼登入。詳見前一份 DEPLOY.md。設定一次,之後跟 GitHub Pages 一樣 git push 即可。

---

## 維護備忘

每月 1 號流程:

1. 開 app(localStorage 已記得登入,不用重輸密碼)
2. 點「新增 YYYY/MM 快照」
3. 逐欄輸入,沒變的點「沿用上月」
4. 儲存
5. 設定 → 匯出 JSON → 覆蓋 iCloud/Dropbox 上的備份檔

季度檢查(可選):

- 看「歷史」分頁切換不同區間,確認資產走勢正常
- 比對 Excel(如果還在維護)看數字一致

每年:

- 更新「退休目標總資產」「年化報酬率」(設定頁可改)
