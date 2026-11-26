# YouBike 2.0 即時資料定時爬蟲 (GitHub Actions 版)

本專案透過 Python 腳本抓取台北市 YouBike 2.0 即時開放資料，並利用 **GitHub Actions** 設定排程，每 5 分鐘自動執行爬蟲，將最新的站點車輛動態自動更新並保存至 SQLite 資料庫（`ubike.db`）。

---

## 專案核心架構

- **`get_ubike_data.py`**：主要爬蟲邏輯，負責向台北市政府開放資料 API 請求 JSON 資料，並寫入/更新 `ubike.db` 資料庫。
- **`.github/workflows/ubike_crawler.yml`**：GitHub Actions 自動化工作流程設定檔，每 5 分鐘觸發一次。
- **`ubike.db`**：SQLite 資料庫檔案，儲存歷史爬取資料（依站點編號與時間戳為主鍵，避免重複寫入）。

---

## 步驟一：GitHub 倉庫初始化與推送 (Git 指令)

請開啟終端機（PowerShell 或 Bash），切換至專案根目錄，依序執行下列 Git 指令：

### 1. 初始化本地 Git 儲存庫
```bash
git init
```

### 2. （可選）建立 `.gitignore` 排除大型或暫存檔案
建議在根目錄建立 `.gitignore`，避免大檔案或暫存檔上傳：
```bash
# 建立 .gitignore
echo "__pycache__/" >> .gitignore
echo "*.pyc" >> .gitignore
echo ".vscode/" >> .gitignore
```
> **注意**：本專案需要由 Actions 同步 `ubike.db`，請**不要**將 `ubike.db` 加入 `.gitignore`。

### 3. 加入所有檔案並進行第一次提交 (Commit)
```bash
git add .
git commit -m "feat: 建立 YouBike 爬蟲與 GitHub Actions 定時排程"
```

### 4. 將預設分支切換為 `main`
```bash
git branch -M main
```

### 5. 關聯到 GitHub 遠端儲存庫
> 請先至 [GitHub](https://github.com/new) 建立一個全新的 Repository（建議設為 Public 或 Private 均可）。
> 將下列 `<你的GitHub帳號>` 與 `<你的倉庫名稱>` 替換為實際網址：

```bash
git remote add origin https://github.com/<你的GitHub帳號>/<你的倉庫名稱>.git
```

### 6. 推送至 GitHub
```bash
git push -u origin main
```

---

## 步驟二：開啟 GitHub Actions 寫入權限 (關鍵設定)

預設情況下，GitHub Actions 的 GITHUB_TOKEN 只有唯讀權限。若未開啟寫入權限，Actions 執行完將無法把更新後的 `ubike.db` 推送回儲存庫。

請依照以下步驟設定：

1. 開啟您的 GitHub 專案頁面。
2. 點擊上方導覽列的 **Settings**（設定）。
3. 在左側選單點選 **Actions** -> **General**。
4. 向下滾動至 **Workflow permissions** 區塊。
5. 勾選 **「Read and write permissions」**（讀寫權限）。
6. 點擊 **Save**（儲存）。

```
Settings ➔ Actions ➔ General ➔ Workflow permissions ➔ Read and write permissions ➔ [Save]
```

---

## 步驟三：測試與驗證

### 1. 手動觸發測試 (Workflow Dispatch)
不用乾等 5 分鐘，可直接手動測試：
1. 進入 GitHub 專案頁面的 **Actions** 頁籤。
2. 左側點擊 **YouBike Data Crawler** 工作流程。
3. 右側會出現 **Run workflow** 按鈕，點擊並選擇 `Branch: main`，再點擊綠色 **Run workflow**。
4. 點進該次執行紀錄，確認所有步驟（Setup Python ➔ Install ➔ Run script ➔ Commit & Push）皆顯示綠色勾勾。

### 2. 檢視自動提交記錄
當資料更新成功後，回到專案的 **Code** 頁籤或查看 Commits，會看到類似以下的自動提交紀錄：
```text
Auto: Update ubike.db [2026-11-26 06:05:00 UTC] (by github-actions[bot])
```

---

## 步驟四：本地同步最新資料庫

當 GitHub Actions 持續在雲端收集資料並更新 `ubike.db` 後，若您要在本機分析或查詢最新資料，只需在本地終端機執行：

```bash
git pull
```

即可將雲端最新的 `ubike.db` 同步下載回本地電腦。

---

## 常見問題 (FAQ)

### Q1: 排程沒有準時在第 5 分鐘執行？
- **說明**：GitHub Actions 的 `cron` 排程在公共負載較高時可能會排隊延遲（約 3~10 分鐘不等），這是 GitHub 免費算力資源的正常機制，並非設定錯誤。

### Q2: 執行時報錯 `remote: Permission to ... denied to github-actions[bot]`？
- **解決方式**：請確認已完成 **步驟二**，將 **Workflow permissions** 改為 **Read and write permissions**。

### Q3: 本地修改程式碼後如何再次更新推送到 GitHub？
```bash
# 1. 建議先拉取遠端最新 db
git pull

# 2. 加入變更並提交
git add .
git commit -m "fix: 更新爬蟲邏輯"

# 3. 推送更新
git push
```
