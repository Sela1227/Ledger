# Ledger

個人資產快照工具 · 單檔 HTML · 純前端 · 部署於 GitHub Pages

## 用法
開啟 `index.html` 即可。資料儲存於瀏覽器 localStorage。

每月 1 號:
1. 點「新增 YYYY/MM 資產表」
2. 輸入 QYLD 當日年化、各帳戶餘額
3. 雲象輸入後欠饅頭自動算(=-雲象/15)
4. 儲存
5. 設定 → 匯出 JSON 備份

## 編輯歷史月份
歷史頁點 ✎ 編輯,公式帳戶不會主動覆寫值,
但會顯示「公式期望值」當對帳參考(綠✓ 吻合 / 紅⚠ 不符)

## 檔案
- `index.html` 主程式
- `import_excel.py` Excel→JSON 匯入工具
- `ledger-history.json` 歷史備份
- `CLAUDE.md` 開發脈絡
- `DEPLOY.md` 部署指南
