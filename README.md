# Ledger

個人資產快照工具 · 單檔 HTML · 純前端 · 部署於 GitHub Pages

## 用法

開啟 `index.html` 即可。資料儲存於瀏覽器 localStorage,不上傳。

每月 1 號:
1. 點「新增 YYYY/MM 快照」
2. 輸入各帳戶餘額(沒變動的點「沿用上月」)
3. 儲存
4. 設定 → 匯出 JSON 備份

## 檔案

- `index.html` — 主程式
- `import_excel.py` — Excel → JSON 匯入工具
- `ledger-history.json` — 歷史資料備份
- `CLAUDE.md` — 開發脈絡(給接手 AI 看的)
- `DEPLOY.md` — 部署指南
