# Logo 素材庫專案 — 交接文件

> 這份文件是給另一個 Claude 對話（不同帳號）用的完整脈絡。開新對話時把這份文件貼給 Claude，就能無縫接續。

## 專案是什麼

巨思文化媒體創意中心內部的品牌 logo 自動化蒐集/歸檔系統。目的：影音/圖表製作時需要用到公司 logo，不用每次重新上網搜尋。

## 網站與 Repo

- **網站**：https://linda0809.github.io/logo-library/
- **Repo**：https://github.com/linda0809/logo-library（GitHub 帳號 `linda0809`，**Public** repo）
- 用 GitHub Pages host，Public 是因為 GitHub 免費方案 Private repo 不支援 Pages

## 架構

```
logo-library/
├── index.json   → 資料索引，AI查詢/新增用
├── index.html   → 搜尋+批次下載頁面（人力使用）
├── README.md    → 命名規則/流程說明
└── assets/      → 實際logo檔案
```

### index.json schema（重要：每家公司支援多版本 variants）

```json
{
  "companies": [
    {
      "中文名稱": "台積電",
      "英文名稱": "TSMC",
      "別名": ["台灣積體電路", "Taiwan Semiconductor"],
      "variants": [
        {
          "標籤": "完整版（含晶圓網格圖案）",
          "檔案": "assets/台積電_TSMC_logo_完整版.svg",
          "來源網址": "...",
          "版權備註": "..."
        }
      ],
      "新增日期": "2026-09-18"
    }
  ]
}
```

- 找不到 logo 的公司：`variants: []` + `_狀態: "待補"` 欄位
- 命名規則：`中文名_英文名_logo_[版本標籤].副檔名`（單一版本可省略標籤）
- **只有確認官方真的有提供該版本才新增 variant**，不會用同一張圖假造不同排版

## 找 logo 的優先順序（重要教訓）

1. **Wikimedia Commons**（`commons.wikimedia.org`）—自由版權素材
2. **⚠️ 英文維基百科自己的 File 空間**（`en.wikipedia.org`，不是 commons！）—很多公司官方註冊商標是用 fair-use 方式放在這裡，**不會出現在 Commons 搜尋結果裡**。查法：抓該公司英文維基條目 infobox 原始碼裡的 `| logo = xxx.svg` 參數，再用 API 查 `en.wikipedia.org` 的 imageinfo 拿到 `upload.wikimedia.org/wikipedia/en/...` 的真實網址
3. **官方網站首頁** grep `<img>` 找 logo（class/id 含 "logo"）
4. 都找不到 → 標記「待補」，不硬湊

技術細節：
- 下載用 `curl -sL -A "Mozilla/5.0" "https://commons.wikimedia.org/wiki/Special:FilePath/檔名"`（會自動 redirect 到真實檔案）
- `logo.clearbit.com` 這類第三方API在這個環境連不上（DNS resolve失敗），別浪費時間試
- SVG 預覽驗證用 `cairosvg`（`pip install cairosvg --break-system-packages`）

## 目前 23 家公司狀態

**完整（21家）**：台積電(2版本:完整版+純文字版)、美光、環球晶、英特爾、輝達(2版本:直式+橫式)、中美晶、台塑集團、SUMCO、台勝科、Siltronic、昇陽半導體、蘋果、三星、鎧俠、SK海力士、旺宏、群聯、晶豪科、宇瞻、威剛、創見

**待補（2家）**：
- 合晶 Wafer Works：官網 logo 是文字+CSS做的，非圖檔
- 嘉晶 Episil-Precision：官網連線失敗(503)

## 網站功能

- 搜尋（中/英文名、別名）
- 每家公司卡片顯示所有版本，可個別勾選
- 全選/清除選取
- 打包下載成 zip（用 CDN 的 JSZip，`cdnjs.cloudflare.com/ajax/libs/jszip`）

## 完整工作流程（給AI的操作SOP）

1. 使用者丟 Google Sheet/Doc 連結，說看哪一欄（通常是「旁白」）
2. 用 `web_fetch` 讀取內容（前提：該檔案要設定「知道連結者可檢視」，否則會 401）
3. 從該欄位文字中辨識出現的公司/品牌名稱，列出清單給使用者看
4. **使用者說「要」之後，不用再問，直接跑完整流程**（查重複→找來源→下載→命名→更新index.json→git push）
5. 查 `index.json` 是否已有該公司，避免重複
6. 依「找 logo 優先順序」搜尋，下載、命名、更新 index.json
7. `git add . && git commit -m "..." && git push`
8. 用 `gh api repos/linda0809/logo-library/pages` 或直接 curl 網站確認部署完成（GitHub Pages有 `cache-control: max-age=600` 快取，剛更新完馬上看可能要強制重新整理 `Ctrl/Cmd+Shift+R`）

## 環境限制備忘

- GitHub CLI 登入：`gh auth login --hostname github.com --git-protocol https --web`（背景執行+讀log拿一次性驗證碼），登入後要跑 `gh auth setup-git` 才能 `git push`
- **每個新對話/新環境的 container 都要重新登入一次**，token 不會保留
- 沒有 Google Sheets/Docs/Drive 的官方連接器，只能用分享連結+`web_fetch`讀取（若 401 就是分享權限沒開）

## 待辦事項（尚未實作）

- 網頁上傳功能：目前是純靜態 GitHub Pages 沒有後端，沒辦法真正做到「網頁上傳自動存檔」。討論過的選項：
  - 方案B：接 Cloudflare Worker 當後端，用 GitHub API 自動 commit（需另外架設，token 存在 Worker 後端）
  - 方案C：網頁上傳後打包好給使用者，使用者再貼給 Claude 處理
  - 使用者目前選擇「先不用，維持現有流程」
- 去背功能：目前流程沒做（使用者要求拿掉）
- 合晶、嘉晶兩家 logo 待美術手動補
