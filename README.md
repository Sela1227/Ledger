<div align="center">
  <img src="favicon/sela.svg" width="120" alt="SELA"/>
  <h1>資產札記 Ledger</h1>
  <p>Sela 的個人資產追蹤工具 · 單檔 HTML · 純前端 · 部署於 GitHub Pages</p>
</div>

---

## 簡介

每月一張資產表,自動算總資產、月支出、退休試算。
資料只存自己瀏覽器的 localStorage,不上傳任何伺服器。
12 條 Excel 公式對齊驗證,V0.7 起加入凍結機制讓歷史月份不被未來公式變動影響。

## 啟動

開啟 `index.html` 即可。或部署到 GitHub Pages 在手機 Safari「加入主畫面」變 PWA。

## 用法

每月 1 號:
1. 點「新增 YYYY/MM 資產表」
2. 輸入 QYLD 當日年化、一菜雙石儲備、特斯拉車貸月繳
3. 輸入各帳戶餘額(沒變的點「沿用」)
4. 雲象輸入後欠饅頭自動算(=-雲象/15)
5. 儲存
6. 設定 → 匯出 JSON 備份

## 目錄結構

```
資產札記/
├── index.html              主程式(單檔)
├── ledger-history.json     內嵌歷史備份(57 筆 2019/11~2026/03)
├── import_excel.py         Excel→JSON 匯入腳本
├── favicon/                SELA 品牌資產 + favicon 套組
│   ├── sela.svg
│   ├── favicon.ico
│   ├── favicon-16x16.png
│   ├── favicon-32x32.png
│   ├── apple-touch-icon.png
│   ├── android-chrome-192x192.png
│   ├── android-chrome-512x512.png
│   └── site.webmanifest
├── README.md               本檔
├── CLAUDE.md               給下次接手 Claude 的工作上下文
├── DEPLOY.md               GitHub Pages 部署指南
├── SELA-handoff.md         首次對齊 SELA-Starter-Kit V1.8.0 紀錄
└── .gitignore
```

## 版本歷程

- **V0.8.1** — 資料更新到 2026/05;修 import_excel.py 漏抓 F4(monthly_kid_wan)和 I2(tesla_pmt_wan)的 bug
- **V0.8.0** — 對齊 SELA-Starter-Kit V1.8.0(加 favicon、加 SELA logo banner、加 SELA-handoff、補齊 .gitignore;配色保留現有黑底金字)
- **V0.7.1** — 修升級 bug:loadData 加 mergeSettings 補齊缺漏設定;貸款月繳全可編輯
- **V0.7.0** — 凍結機制:儲存下一張時凍結上一張,改設定不再影響歷史
- **V0.6.2** — 手機 UX 修正:貸款月繳改卡片式 + oninput + inputmode
- **V0.6.1** — 首頁摺疊群組:預設加總、點開看細項
- **V0.6.0** — 退休試算重構:對齊 Excel 12 條公式
- **V0.5.x** — 公式語意修正、退休目標自動算、雲象 reference side
- **V0.4.0** — 加密碼登入、內嵌歷史、匯入合併
- **V0.3.0** — Excel→JSON 匯入腳本、PWA manifest
- **V0.2.0** — 字體換 Noto Sans TC + JetBrains Mono、加歷史頁、SVG 走勢圖
- **V0.1.0** — MVP

## 配色說明

Ledger 用「黑底金字」(沉浸式深色配 JetBrains Mono 數字),不是 SELA 主題橘。
這是 Sela 的個人選擇,理由是長時間看資產降低焦慮感。
SELA logo 永遠是橘 + 白(在 favicon 與 README banner 看到)。

## 隱私

- 部署到 GitHub Pages **公開 repo**(用混淆 repo 名 + JS SHA-256 密碼遮蔽)
- 真實金額存 localStorage,不上 Git
- 密碼內建在程式裡,擋家人朋友夠用,擋資安專業不夠
- 詳見 CLAUDE.md「業務事實」章節

---

> Made by **Sela** · V0.8.1 · 對齊 SELA-Starter-Kit V1.8.0
