# CLAUDE.md

> AI 助理協作本專案時的指引與規範。

## 專案概述

**台中家庭旅行 2026** — 一個靜態單頁網站，用於規劃三天兩夜的台中家庭旅遊（2026/2/20-22）。行程以海線觀光為主，成員為 2 位大人與 2 位小孩（4 歲、7 歲）。

- **線上網站：** https://taichung-family-trip.netlify.app
- **GitHub 儲存庫：** https://github.com/sniperHK/taichung-family-trip

## 技術棧

| 層級 | 技術 |
|------|------|
| 前端 | 純 HTML5、CSS3、原生 JavaScript（無框架、無打包工具） |
| 字型 | Google Fonts — Noto Sans TC、Noto Serif TC、DM Serif Display |
| 客戶端儲存 | `localStorage`（行李清單持久化，key：`taichung-trip-packing`） |
| 地圖 | Google Maps 路線規劃（外部連結） |
| 部署 | Netlify（push 至 `main` 自動部署） |
| 加密 | `age` 加密環境變數 |

## 儲存庫結構

```
.
├── index.html            # 主網頁（HTML、CSS、JS 全部合一）
├── 行程.md               # 行程資料 — 唯一資料來源（Single Source of Truth）
├── 交通路線.md            # 12 條駕車路線，含距離、車程、停車資訊
├── .shared-ai-context.md  # 專案上下文、變更日誌、Git 歷史同步
├── CLAUDE.md             # AI 助理指引（本檔案）
├── netlify.toml           # Netlify 設定（publish dir: /）
├── .env.example           # 環境變數範本
├── .env.age               # age 加密後的環境變數（已提交至 git）
├── .gitignore             # 忽略明文 .env、age-key.txt
├── .claude/
│   └── setup.sh           # Cloud VM 啟動時自動解密 .env.age
├── secrets/
│   └── env.keys           # 環境變數白名單
└── messageImage_*.jpg     # 旅行參考圖片
```

## 關鍵規則：資料同步

**`行程.md` 是所有行程資料的唯一資料來源（Single Source of Truth）。**

任何內容變更必須遵循以下流程：

1. **先**更新 `行程.md`
2. 將變更同步至 `index.html`（時間軸卡片、表格、詳情區塊）
3. 若涉及交通路線，一併更新 `交通路線.md`
4. 將所有修改的檔案一起 commit
5. Push 至 `main` 後 Netlify 會自動部署

**絕對不要**在未先更新 `行程.md` 的情況下修改 `index.html` 的行程內容。

## index.html 架構

整個網站存放在單一 `index.html` 檔案中（約 2000 行），CSS 與 JS 皆內嵌。

### CSS（約 900 行）

- 海洋度假風主題，使用 CSS 自訂屬性管理色彩
- 主要色彩 token：`--deep-navy`、`--ocean-blue`、`--sunset-coral`、`--warm-sand`、`--seafoam`
- 響應式設計：使用 `clamp()` 做流動字體大小、Flexbox 與 CSS Grid 佈局
- Mobile-first，搭配 media query 處理大螢幕

### HTML 區段（依序）

1. **導航列** — 固定置頂、backdrop blur 效果、捲動時高亮當前區段
2. **Hero 區** — 全螢幕動畫場景（雲朵、太陽、漸層天空）
3. **住宿資訊** — 飯店詳細資料
4. **第一天時間軸** — 抵達 + 高美濕地夕陽
5. **第二天時間軸** — 海洋館、漁港、自行車道、夜市
6. **第三天時間軸** — 科博館、太陽餅 DIY、回程
7. **交通路線** — 路線規劃器（含 Google Maps 連結）
8. **實用資訊** — 天氣、行李建議、緊急聯絡方式
9. **行李清單** — 互動式勾選清單（localStorage 持久化）

### JavaScript（約 140 行）

- 捲動監聽式導航高亮
- `IntersectionObserver` 實作捲動顯現動畫
- 景點資訊展開/收合（使用 `data-*` 屬性的可展開卡片）
- 行李清單的 `localStorage` 讀寫
- 路線規劃器：起終點選擇器、交換按鈕、Google Maps 連結產生

## 開發工作流程

### 無需建置步驟

本專案為純靜態網站 — 直接編輯檔案、用瀏覽器開啟 `index.html` 即可預覽。沒有 `package.json`、沒有 linter、沒有測試框架、沒有打包工具。

### 部署方式

- **自動部署：** Push 至 GitHub `main` 分支即觸發 Netlify 部署
- **備援部署：** 使用 `NETLIFY_BUILD_HOOK_URL`（存於 `.env.age`）透過 webhook 觸發部署，適用於無 GitHub PAT 的情境

### 加密與 Secrets 管理

環境變數使用 `age` 加密：

- **公鑰：** `age1sd7r54gpapjuvt67x4g4ednmzus7s5xqx93ltcv57axcrs6w7yyqktrceh`
- **加密檔：** `.env.age`（已提交至 git）
- **解密方式：** `.claude/setup.sh` 在 Cloud VM 啟動時自動執行，需設定 `AGE_SECRET_KEY` 環境變數
- **禁止提交**明文 `.env` 或 `age-key.txt`

### 分支策略

- `main` — 正式環境分支，push 即觸發 Netlify 部署
- `master` — 舊版預設分支（不用於部署）
- Feature 分支命名規則：`claude/<描述>-<id>`

## Commit 慣例

使用 conventional commit 前綴：

- `feat:` — 新功能或內容新增
- `fix:` — 錯誤修正或內容更正
- `docs:` — 僅文件變更
- `chore:` — 設定、工具、secrets 管理

Commit 訊息可使用中文或英文，依變更情境而定。保持簡潔。

## 程式碼慣例

- **檔案命名：** 內容檔案使用中文（`行程.md`、`交通路線.md`），設定與程式檔案使用英文
- **CSS：** 使用既有的 CSS 自訂屬性；維持海洋主題一致性
- **HTML：** 使用語意化元素；外部連結加上 `target="_blank"` 與 `rel="noopener"`
- **JS：** 僅使用原生 JavaScript — 不引入外部函式庫或框架
- **ID/Class 命名：** 英文、kebab-case（如 `route-planner`、`packing-list`）
- **Data 屬性：** 行李清單項目使用 `data-key` 對應 localStorage

## 更新 .shared-ai-context.md

每次完成工作後，請更新 `.shared-ai-context.md`：

1. 執行 `git log --oneline --graph -15`，將輸出貼入 Git 歷史區段
2. 新增變更日誌，格式：`[YYYY-MM-DD] [Agent 名稱]: 變更描述`
3. 更新不再準確的段落

## 重要內容細節

- **飯店：** 震大金鬱金香酒店（台中市梧棲區）
- **旅行日期：** 2026 年 2 月 20-22 日（週五至週日）
- **旅行成員：** 2 位大人 + 2 位小孩（4 歲、7 歲）
- **主要區域：** 台中海線（梧棲、高美、清水）
- **海洋館門票：** 已購票（訂單 #PN5727310576）— 全票 ×1 + 優待票 ×2 = NT$1,300，場次 2（10:00-11:00）

## 禁止事項

- 不要加入 npm/yarn 依賴或建置系統
- 不要將 `index.html` 拆分為多個檔案（單檔設計是刻意的）
- 不要更改海洋度假風視覺主題
- 不要移除行李清單的 localStorage 持久化功能
- 不要提交明文 `.env` 或私鑰
- 不要在未確認 `行程.md` 與 `index.html` 同步的情況下直接 push 至 `main`
