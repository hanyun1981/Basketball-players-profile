# Player Performance Portfolio — Firebase + GitHub Pages 部署指南

這份應用使用 **Firebase**（Authentication + Firestore）做雲端儲存，部署在 **GitHub Pages** 上。

**角色權限**：所有登入者共用同一份球隊資料。
- **Editor（可編輯）**：能新增／修改球隊、球員、成績，並能指派其他 Editor。
- **Viewer（僅瀏覽）**：只能查看雷達圖、趨勢、排行，無法修改任何資料。

「誰是 Editor」由 **Owner（擁有者，也就是你）** 決定。你永遠是 Editor，可以在網頁內把其他教練的 Google email 加進 Editor 名單。其他登入的人預設都是 Viewer。

---

## 第一部分：建立 Firebase 專案（一次性）

> 你已經有 `player-tracking-data` 專案的設定了，若已建立可直接跳到「啟用服務」步驟。

### 1. 建立專案
1. 進入 [Firebase Console](https://console.firebase.google.com/)
2. 點 **「新增專案」**，Google Analytics 可不啟用

### 2. 啟用 Firestore Database
1. 左側 → **Build → Firestore Database** → **建立資料庫**
2. 位置選 `asia-east1`（台灣最近）
3. 模式選 **「以正式版模式啟動」**

### 3. 啟用 Google 登入
1. 左側 → **Build → Authentication** → **開始使用**
2. **Sign-in method** → **Google** → 啟用 → 填支援信箱 → 儲存

---

## 第二部分：設定權限（角色制重點）

### A. 內建 Editor 帳號（已設定好）

以下兩個帳號已被設為**內建 Editor**，登入後永遠擁有編輯權限，無法在網頁中被移除：
- `hanyun@ibsh.tw`
- `jess@ibsh.tw`

這份名單寫在兩個地方且必須一致（已幫你設定好）：
- `index.html` 的 `const BASE_EDITORS = ["hanyun@ibsh.tw", "jess@ibsh.tw"];`
- `firestore.rules` 的 `isBaseEditor()` 函式

> ⚠️ **重要前提**：`@ibsh.tw` 必須是 **Google Workspace（Google 帳號）** 網域，這兩個信箱才能用「Google 登入」。如果學校的信箱不是 Google 帳號，Google 登入會失敗 —— 這種情況請告訴我，我改用 email/密碼登入方式。

之後若要新增其他 Editor，這兩位內建 Editor 任一人登入後，到 **Edit Data → 🔑 Manage Access** 輸入對方 Gmail 即可（不必再改程式碼）。

### B. 發布 Firestore 安全規則
1. Firebase Console → **Firestore Database → 「規則」** 分頁
2. 貼上改好的 `firestore.rules` 全部內容 → **發布**

這套規則確保：
- 所有登入者都能**讀取**球隊資料（`appData/main`）
- 只有 **Owner 或 Editor 名單內的人**能**寫入**資料
- 只有 Owner / Editor 能修改 Editor 名單（`config/roles`）

### C. 取得網頁 App 設定（若 index.html 尚未填）
你提供的設定已經寫入 `index.html` 了。若要重新取得：
**齒輪 → 專案設定 → 您的應用程式 → `</>` → 複製 firebaseConfig**

---

## 第三部分：部署到 GitHub Pages

### 1. 建立 Public Repository
[github.com/new](https://github.com/new) → 命名（如 `player-portfolio`）→ Public → Create

### 2. 上傳檔案
把 **`index.html`** 上傳到 repo（README 與 firestore.rules 是給你看的，不影響網站運作，但一起放也沒問題）。

命令列：
```bash
git init
git add index.html
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/你的帳號/player-portfolio.git
git push -u origin main
```

### 3. 啟用 GitHub Pages
Repo → **Settings → Pages** → Source 選 **Deploy from a branch** → Branch `main` / `(root)` → Save

1-2 分鐘後會出現網址：`https://你的帳號.github.io/player-portfolio/`

### 4. 加入授權網域（漏了會登入失敗）
Firebase Console → **Authentication → Settings → Authorized domains → Add domain**
輸入：`你的帳號.github.io`（只填網域，不加 https 或路徑）

---

## 完成！運作流程

1. 打開網址 → 用 Google 登入
2. `hanyun@ibsh.tw` 或 `jess@ibsh.tw` 登入後會看到綠色 **「✏️ Editor」** 徽章
3. 第一次沒有資料時，點 **「+ Add Team」** 或 **「Load Sample Data」** 開始
4. 需要更多 Editor 時：**Edit Data → 🔑 Manage Access** → 輸入 Gmail → **+ Add Editor**
5. 其他人登入：
   - 是內建 Editor 或被加入名單 → 看到 **「✏️ Editor」**，可編輯
   - 其他帳號 → 看到灰色 **「👁 View only」**，只能瀏覽

右上角同步燈：🟡 Syncing → 🟢 Synced（🔴 Offline 時仍可瀏覽，連線後自動同步）。

---

## 常見問題

**Q: 為什麼要分共用資料？以前不是每人各自獨立嗎？**
A: 因為「可編輯 vs 僅瀏覽」代表大家看的是同一份資料 —— 教練輸入、其他人瀏覽。所以資料從「每人一份」改成「全隊共用一份」（存在 `appData/main`）。

**Q: 我把某人從 Editor 移除後，他還能編輯嗎？**
A: 不能。移除後他下次操作會被 Firestore 規則擋下（前端也會即時切回 Viewer）。

**Q: Viewer 看得到 Manage Access 嗎？**
A: 看不到。Viewer 連 Edit Data 分頁都沒有，只有 Dashboard。

**Q: hanyun / jess 登入後卻是 View only？**
A: 確認 (1) 登入用的 email 完全等於 `hanyun@ibsh.tw` / `jess@ibsh.tw`，(2) `firestore.rules` 已在 Console「發布」，(3) `@ibsh.tw` 確實是 Google 帳號。三者其一不符就會變 Viewer。

**Q: apiKey 公開在 GitHub 安全嗎？**
A: 安全。真正的保護是 Firestore 規則 + Authentication 授權網域，apiKey 只是專案識別符。

**Q: 免費額度夠嗎？**
A: 非常夠。Spark 免費方案每天 5 萬次讀取、2 萬次寫入，一般使用用不到 1%。

**Q: 怎麼備份？**
A: Editor 在 **Edit Data → Backup & Restore → Export JSON** 下載備份；Import JSON 可還原。

---

## 資料結構（技術參考）

```
Firestore
├── appData/main          ← 共用球隊資料（teams, players, scores）
│                            所有登入者可讀，只有 Editor 可寫
└── config/roles          ← { editors: ["a@gmail.com", "b@gmail.com"] }
                             所有登入者可讀，只有 Editor 可寫
```

內建 Editor（`hanyun@ibsh.tw`、`jess@ibsh.tw`）由 `index.html` 的 `BASE_EDITORS` 與 rules 的 `isBaseEditor()` 共同認定，永遠是 Editor。
