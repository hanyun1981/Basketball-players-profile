# Player Performance Portfolio — Firebase + GitHub Pages 部署指南

這份應用使用 **Firebase**（Authentication + Firestore）做雲端儲存，部署在 **GitHub Pages** 上。每個 Google 帳號的資料完全隔離，只有本人能讀寫自己的資料。

部署完成後你會得到一個像 `https://你的帳號.github.io/player-portfolio/` 的網址，從任何裝置打開、用 Google 登入，就能看到你的所有球隊資料。

---

## 第一部分：建立 Firebase 專案（一次性，約 5 分鐘）

### 1. 建立專案
1. 進入 [Firebase Console](https://console.firebase.google.com/)
2. 點 **「新增專案」**，取個名字（例如 `player-portfolio`）
3. Google Analytics 可以不啟用（這個應用不需要）

### 2. 啟用 Firestore Database
1. 左側選單 → **「Build」 → 「Firestore Database」**
2. 點 **「建立資料庫」**
3. **位置**：選離你最近的（例如 `asia-east1`，台灣使用者用這個）
4. **模式**：先選 **「以正式版模式啟動」**（待會手動設定權限）
5. 建立完成

### 3. 設定 Firestore 安全規則（重要！）
1. 進入 **Firestore Database → 「規則」** 分頁
2. 把整個內容換成下面這段，然後點 **「發布」**：

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 每個使用者只能讀寫自己的資料文件
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

> 這條規則確保：必須登入 + 只能存取自己 uid 對應的文件，其他人完全無法看到你的資料。

### 4. 啟用 Google 登入
1. 左側選單 → **「Build」 → 「Authentication」**
2. 點 **「開始使用」**
3. **「Sign-in method」** 分頁 → 找到 **Google** → 點開
4. 切換為 **「啟用」**
5. 填入 **專案的支援電子郵件**（用你自己的 Gmail 就好）
6. 點 **「儲存」**

### 5. 取得網頁 App 設定
1. 左上 **齒輪圖示 → 「專案設定」**
2. 往下捲到 **「您的應用程式」**
3. 點 **`</>` 圖示**（網頁）新增應用程式
4. 取個暱稱（例如 `web`），**不需要**勾選 Firebase Hosting，點 **「註冊應用程式」**
5. 會出現一段像這樣的設定，**複製整個 `firebaseConfig` 物件**：

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "你的專案.firebaseapp.com",
  projectId: "你的專案",
  storageBucket: "你的專案.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123..."
};
```

> 不用擔心 `apiKey` 公開：Firebase 的 API Key 不是密碼，配合 Firestore 規則才是真正的安全機制。

---

## 第二部分：把設定填入 `index.html`

1. 打開 `index.html`
2. 用編輯器搜尋 `YOUR_API_KEY_HERE`
3. 找到這一段：

```javascript
const FIREBASE_CONFIG = {
  apiKey: "YOUR_API_KEY_HERE",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  ...
};
```

4. 把 6 個欄位換成你剛才複製的值，存檔

---

## 第三部分：部署到 GitHub Pages

### 1. 建立 GitHub Repository
1. 進入 [github.com/new](https://github.com/new)
2. **Repository name**：例如 `player-portfolio`
3. 設為 **Public**（GitHub Pages 免費版需要）
4. **不要**勾任何初始化選項，點 **「Create repository」**

### 2. 推送檔案
最簡單方式：在 repo 頁面點 **「uploading an existing file」**，把 `index.html` 拖進去，commit。

或用命令列：
```bash
git init
git add index.html
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/你的帳號/player-portfolio.git
git push -u origin main
```

### 3. 啟用 GitHub Pages
1. Repo 頁面 → **「Settings」**
2. 左側 → **「Pages」**
3. **Source**：選 **「Deploy from a branch」**
4. **Branch**：選 `main`，資料夾留 `/ (root)`
5. **儲存**

等大約 1-2 分鐘，頁面上方會出現綠色框框：
> Your site is live at `https://你的帳號.github.io/player-portfolio/`

### 4. 把網址加入 Firebase 授權清單（**很重要，否則登入會失敗**）
1. 回到 Firebase Console
2. **Authentication → 「Settings」 → 「Authorized domains」**
3. 點 **「Add domain」**
4. 輸入：`你的帳號.github.io`（**只填網域，不要加路徑或 https**）
5. 確認

---

## 完成！測試一下

1. 打開 `https://你的帳號.github.io/player-portfolio/`
2. 應該會看到登入畫面
3. 點 **Sign in with Google**
4. 第一次登入會自動帶入範例資料
5. 編輯任何欄位 → 觀察右上角同步指示燈：
   - 🟡 **Syncing…**（800ms debounce 後寫入）
   - 🟢 **Synced**
   - 🔴 **Offline**（沒網路也能繼續編輯，連回網路自動同步）

換另一台裝置／瀏覽器登入同一個 Google 帳號 → 看得到一樣的資料 ✓

---

## 常見問題

**Q: 別人會看到我的資料嗎？**
A: 不會。Firestore 規則限制每個人只能讀寫自己的 `users/{自己的uid}` 文件，連 Firebase Console 都看不到別人的資料（除了 Firebase 專案擁有者）。

**Q: 我可以給隊友權限編輯同一份資料嗎？**
A: 目前的版本每個 Google 帳號各自獨立。如果需要共用，可以擴充規則允許特定 uid 也能存取，或建立一個「教練群組」模式。需要的話再告訴我。

**Q: Firebase 免費額度夠用嗎？**
A: 完全夠。Firebase 免費方案（Spark）每天 50,000 次讀取、20,000 次寫入、5,000 次刪除。即使你每天頻繁編輯，也用不到 1%。

**Q: 我把 apiKey 推到公開的 GitHub 上會被駭嗎？**
A: 不會。Firebase 的 apiKey 只是專案識別符，類似網址。真正的保護來自 **Firestore 規則** 和 **Authentication**。但你應該：
- 在 Firebase Console → 「Authentication → Settings → Authorized domains」只列出你信任的網域
- 確保 Firestore 規則正確（按照本指南設定）

**Q: 怎麼備份資料？**
A: 應用內 **Edit Data → Backup & Restore → Export JSON** 隨時可以下載備份檔。

**Q: 我想換 Firebase 專案怎麼辦？**
A: 在舊環境 Export JSON → 切換新的 `FIREBASE_CONFIG` → 重新部署 → 用新帳號登入 → Import JSON。

---

## 想再進一步？

- **自訂網域**：在 GitHub Pages 設定加 CNAME 即可（記得也要加到 Firebase 授權網域）
- **顯示更多統計**：可以擴充儀表板，加上球員間比較、歷史趨勢線等
- **多教練協作**：改寫資料結構，從「per-user」變成「per-team-with-shared-access」
