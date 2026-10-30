# 整理電腦與 NAS 檔案：流程

路徑一律給完整路徑：`C:\Users\名字\Downloads`、NAS 用 `\\伺服器\分享\資料夾` 或對應的磁碟機（例如 `Z:\行程照片`）。不知道 NAS 在哪時先 `list_locations`（會列出對應磁碟機與它的 `\\伺服器\分享`）。

## 找檔案（1 次呼叫）

`search_files`：`path` ＋ `query`（多個詞都要出現，或用 `*`／`?` 樣式）＋ 需要時加 `extensions`、`min_size`、`modified_after`、`modified_before`、`sort`。找文字內容用 `mode: "content"`。第一次搜一個大資料夾（例如整個 NAS 分享）會先建索引，可能回 `task_id`：直接 `task_status`（`wait_seconds: 60`）。之後同一處的搜尋幾乎即時，檔案有變動 so 會自動更新索引。

## 整理一個資料夾（典型：下載資料夾）

1. `organize_directory`：`path`，`by` 選 `type`（圖片／影片／音訊／文件／試算表／簡報／PDF／壓縮檔／安裝程式）、`date`（年／月）或 `extension`。只整理該層的檔案；要含子資料夾加 `recursive: true`。這一步**只產生計畫**。
2. 把摘要給同事：「要把 1,000 個檔案分到 9 個資料夾：圖片 320、文件 210…；跳過 12 個（不認得的類型）。要開始嗎？」有 `warnings` 一定要說（雲端同步、跨磁碟較慢）。
3. 同事說好 → `apply_plan`（`plan_id`、`confirmed_by_user: true`）。20 個以下可在第 1 步直接帶 `apply: true` 一次完成。
4. 回報；同事反悔 → `undo_plan`（同一個 `plan_id`），全部放回原處。

## 改名

`plan_moves`：`path` ＋ `rename` 範本，例如 `"{date:yyyyMMdd}_{name}"`、`"京都_{n:000}"`（`{name}` 原檔名、`{ext}` 副檔名、`{date:格式}` 修改日期、`{n}` 流水號、`{parent}` 所在資料夾名）。可加 `extensions` 只改某類。之後同上：給同事看 → `apply_plan`。

## 指定清單搬移

`plan_moves`：`moves: [{source, destination}]`（`destination` 是新完整路徑，或已存在的資料夾＝搬進去）。

## 找重複、清理

1. `find_duplicates`：`paths`（可多個，可含 NAS）。通常要幾秒到幾分鐘，會回 `task_id` → `task_status`。
2. 每組保留一份（預設保留最早的那份；同事有偏好就照同事的），其餘用 `plan_moves`：`moves` 只給 `source`、`action: "trash"`。
3. 給同事看要移走哪些、可省多少空間 → 同意後 `apply_plan`（`confirmed_by_user: true`）。

清舊檔：`plan_moves`：`path`、`action: "trash"`、`older_than_days: 365`（可加 `extensions`）→ 同上。

## 備份到 NAS／外接碟

`copy_verified`：`source`、`destination`（目的資料夾）。so 先檢查空間，逐檔 SHA-256 驗證；中斷後再呼叫一次會從沒完成的地方繼續。回覆 `all_verified: true` 才能說「備份完成」。原檔不會被刪；同事要騰空間，另外用 `trash`。

## 批次轉檔

`convert_images`：`paths`（檔案或資料夾）、`format`（`jpg`／`png`）、需要時加 `max_dimension`（長邊像素）與 `quality`。輸出到旁邊的 `converted` 資料夾（或 `destination`），原檔不動。HEIC 失敗時請同事從 Microsoft Store 安裝「HEIF 影像延伸模組」。

## 回報範本

完成：
> ✅ 下載資料夾整理好了：1,000 個檔案分到 9 個資料夾（圖片 320、文件 210…）。有 12 個不認得的檔案留在原處。要復原跟我說一聲就好。

需要確認：
> 📋 我先排好了整理計畫（還沒動任何檔案）：…。要照這樣整理嗎？

重複檔：
> 🔍 找到 214 組重複檔，共可省 3.2 GB。我建議每組留最早的那份，其餘移到 SKYE 回收區（可還原）。要這樣做嗎？

移到回收區：
> 🗑️ 已把 120 個檔案移到 SKYE 回收區（沒有真的刪除，隨時可以還原）。

備份：
> 💾 已把「2026 京都」複製到 NAS，共 1,532 個檔案、48 GB，每個檔案都驗證過一模一樣。原檔還在。
