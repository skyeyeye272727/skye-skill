# so 回傳的錯誤 → 你該怎麼做

錯誤代碼只給你看；對同事一律用 [report-templates.md](report-templates.md) 的白話。

| 錯誤 | 意思 | 你要做的 |
| --- | --- | --- |
| `LOGIN_REQUIRED` | 網站要登入，so 已開登入視窗 | 請同事在跳出的視窗登入，說「好了」再繼續；**不要反覆重試** |
| `LOGIN_REQUIRED_AFTER_WRITE` | 寫入送出後頁面要求登入，結果不確定 | 同事登入後**先回讀**確認有沒有寫進去，再決定是否重做；不要直接重送 |
| `TIMEOUT` | 超過時間 | 讀取可以再試一次；**寫入要先回讀**再決定 |
| `REOPEN_REQUIRED` | 這個分頁是同事手動用過的，有即時連線 | `browser_open` 帶同一個 `tab_id` 重開頁面，再做一次 |
| `TAB_BUSY` | 同一分頁還有動作在跑 | 等它結束再做 |
| `TAB_NOT_FOUND` | 分頁已關（同事關掉、按了瀏覽器上方偵錯提示的「取消」、閒置關閉） | 重新 `browser_open` |
| `TAB_LIMIT` | 同一設定檔開了 12 個分頁 | 用 `browser_close` 關掉用完的分頁 |
| `ELEMENT_NOT_FOUND`、`invalid_selector` | 找不到欄位 | 先 `browser_read`（`form`／`elements`）看清楚再改選擇器 |
| `SCRIPT_ERROR` | 你的 JavaScript 出錯 | 修正腳本；資料放 `args` |
| `NAVIGATION_FAILED` | 網路或網址錯 | 確認網址；網站掛了就回報 |
| `CONFIRMATION_DENIED` | 同事沒有通過 Windows Hello | 停止，不要再要求 |
| `CONFIRMATION_UNAVAILABLE` | 這台電腦不能用 Windows Hello | 停止，請同事自己在後台操作 |
| `not_found` | 上傳的檔案不存在 | 跟同事確認路徑 |
| `reserved_control_state` | 想上傳 so 自己的資料 | 不可以，換同事提供的檔案 |
| `DOWNLOAD_FAILED` | 下載被取消 | 再試一次；仍失敗就回報 |
| `EXTENSION_NOT_CONNECTED` | 同事的 Chrome／Edge 沒開，或沒裝 SKYE 擴充功能 | 請同事打開瀏覽器（擴充功能會自己連上），再試一次；仍不行就請同事開 SKYE 的「連接瀏覽器」看狀態 |
| `HOST_NOT_ALLOWED` | 這個網站不在 SKYE 的公司網站清單 | 不要換方法硬做；回報同事，網站清單只有 SKYE（老闆）能加 |
| `blocked_requests` 裡有 `host_not_allowed` | SKYE 的分頁被導到清單外的網站 | 停止，回到公司網站重新 `browser_open` |
| `BROWSER_NOT_FOUND` | （備援專用瀏覽器模式）電腦沒有 Edge／Chrome | 請同事安裝 Edge |
| `BROWSER_LAUNCH_FAILED`、`BROWSER_EXITED`、`BROWSER_PROTOCOL_ERROR` | 瀏覽器出問題 | 重新 `browser_open`；連續失敗就回報 |
| `SITES_FILE_INVALID` | so 的網站設定檔壞了，所有寫入被擋 | 停止，請同事聯絡老闆 Skye |
| `LICENSE_REQUIRED`、`LICENSE_OFFLINE_EXPIRED`、`ACCOUNT_DISABLED`、`LICENSE_ENDED` | so 授權狀態 | 停止。未啟用時請同事打開 SKYE 按「啟用」，用公司登記的 Google 帳號完成；顯示「已停用」時請同事聯絡老闆 Skye |
| `EXPECTATION_FAILED`、`TEMPLATE_UNRESOLVED`、`invalid_plan`、`PLAN_NOT_FOUND` | `run_task` 計畫問題 | 用 `task_status` 看哪一步，修正計畫 |
| `blocked_requests` 裡有 `undeclared_write` | 用 `read` 卻送出寫入 | 確認真的要寫，再用 `effect: "write"` |
| `blocked_requests` 裡有 `irreversible_requires_confirmation` | 碰到公開／刪除類網址 | **停下來**；只有同事明確要求公開時才用 `irreversible` |

## 檔案工具

| 錯誤 | 意思 | 你要做的 |
| --- | --- | --- |
| `CONFIRMATION_REQUIRED` | 計畫超過 20 個項目，還沒取得同事同意 | 把計畫摘要給同事看，同事說好才帶 `confirmed_by_user: true` 再呼叫 |
| `PROTECTED_PATH` | 系統資料夾、so 自己的資料、磁碟根目錄或 SKYE 回收區 | 不能動；換一個位置，或跟同事說明 |
| `NOT_FOUND` | 檔案或資料夾不在了 | 用 `search_files` 找新位置，或跟同事確認 |
| `FILE_IN_USE` | 檔案正被其他程式開著 | 請同事關掉那個程式再試 |
| `INSUFFICIENT_SPACE` | 目的地空間不夠，什麼都沒複製 | 回報需要多少空間，請同事清出空間或換目的地 |
| `VERIFY_FAILED` | 複製後內容不一致，沒有放上去 | 再試一次；連續失敗可能是磁碟或網路問題，回報 |
| `CROSS_VOLUME_FOLDER` | 想把整個資料夾搬到另一顆磁碟或 NAS | 改用 `copy_verified`，驗證完成再 `trash` 原資料夾 |
| `ALREADY_EXISTS` | 同名的檔案已存在（建資料夾時） | 換名稱 |
| `PLAN_ALREADY_APPLIED`、`PLAN_ALREADY_UNDONE`、`PLAN_NOT_APPLIED` | 計畫已套用／已還原／還沒套用 | 照狀態決定：要再整理就重新產生計畫 |
| `PLAN_NOT_FOUND` | 沒有這個計畫（保留 30 天） | 重新產生計畫 |
| `PLAN_BUSY` | 這個計畫正在套用或還原中 | 先用 `task_status` 等那個工作結束 |
| `ACCESS_DENIED` | 沒有權限或原檔唯讀，沒有搬動 | 請同事確認權限或取消唯讀 |
| `CONVERT_FAILED`、`UNSUPPORTED` | 圖片轉不了 | HEIC：請同事安裝「HEIF 影像延伸模組」；其他格式回報 |
| `TASK_NOT_FOUND` | 背景工作編號不存在（so 重新啟動過） | 重新呼叫原本的工具 |
| `RESULT_EXPIRED` | 分頁清單超過 30 分鐘 | 重新呼叫原本的工具 |


## 操作 App 與執行程式

| 錯誤 | 意思 | 你要做的 |
| --- | --- | --- |
| `PROCESSES_DISABLED` | 「允許 SKYE 執行程式」沒打開 | 請同事點工作列的 SKYE 圖示打開；不要催，也可以改用其他方法 |
| `CONFIRMATION_DENIED` | 同事按了「否」或 10 秒內沒回應，指令沒有執行 | 停下來，問同事要不要再試或換做法 |
| `CONFIRMATION_PENDING` | 確認框還開著，桌面動作先等 | 等同事回答後再做 |
| `PROTECTED_WINDOW` | SKYE 不操作自己的確認框、登入視窗、命令列、執行對話框、檔案總管網址列 | 不要繞過；改用檔案工具或 `start_process` |
| `NOT_A_DOCUMENT`、`USE_START_PROCESS` | 想用開檔／開 App 來執行程式或腳本 | 改用 `start_process`（同事打開並確認） |
| `USER_ACTIVE` | 同事正在用鍵盤滑鼠 | 稍等再試，或改用不動滑鼠的 `perform_ui_action` |
| `COVERED` | 目標被別的視窗蓋住 | 用 `perform_ui_action`，或請同事把 App 拉到前面 |
| `NEEDS_FOREGROUND` | 快捷鍵要 App 在前景 | 改用 `press_menu_item` |
| `APP_NOT_RESPONDING` | App 沒回應（忙碌或選單開著） | 稍後再試，或先 `close_menu` |
| `ELEMENT_GONE`、`SNAPSHOT_EXPIRED`、`ELEMENT_NOT_FOUND` | 畫面變了 | 重新 `get_accessibility_tree` |
| `MENU_ITEM_NOT_FOUND`、`MENU_ITEM_DISABLED` | 選單沒有這一項或現在是灰色 | 看錯誤裡列出的項目，或先完成前一步 |
| `FILE_CHANGED` | 檔案在你讀之後被改過 | 重新讀取，再決定怎麼改 |
| `CLIPBOARD_CHANGED` | 同事剛複製了別的東西 | 重新 `read_clipboard`，問同事要不要覆蓋 |
| `PASSWORD_FIELD` | 想填密碼欄位 | 請同事自己輸入 |

## 素材與影音

| 錯誤 | 意思 | 你要做的 |
| --- | --- | --- |
| `BASIS_INCOMPLETE`、`BASIS_REQUIRED` | 地區、季節、行為有一項不符合或沒寫文案依據 | 不放；跟同事說缺哪一項，或再找一張 |
| `DOWNLOAD_NOT_SEEN` | 等了 10 分鐘，同事還沒下載 | 問同事是否按了下載（有些網站會先問尺寸或授權） |
| `API_KEY_REJECTED` | 公司的 API 金鑰不對或沒權限 | 改用同事的瀏覽器（帶 `page_url`），並告訴同事金鑰要檢查 |
| `SITE_UNREACHABLE`、`SITE_ERROR`、`RATE_LIMITED` | 網站連不上、出錯或要求放慢 | 稍後再試，或換一個網站 |
| `ASSET_NOT_FOUND` | 網站上沒有這個素材 ID | 重新 `asset_search` |
| `SLOT_NOT_FOUND`、`NOTHING_TO_REVERT` | 這個位置還沒有放過素材，或沒有前一張 | 先用 `asset_place`；或跟同事確認是哪一個位置 |
| `NO_MEDIA_SESSION` | 現在沒有 App 在播放 | 先用 `media_search_play` 開始播放 |
| `TRANSCODE_FAILED`、`TARGET_TOO_SMALL` | Windows 轉不了這個檔，或要壓的大小太小 | 換格式；剪短一點或放寬大小 |

## 遠端

| 錯誤 | 意思 | 你要做的 |
| --- | --- | --- |
| `MACHINE_REQUIRED` | 有多台電腦，沒指定哪一台 | 先 `list_machines`，再帶 `machine` |
| `MACHINE_NOT_FOUND` | 這個人名下沒有這台電腦 | 用 `list_machines` 看名稱；不要嘗試別人的電腦 |
| `MACHINE_OFFLINE` | 那台電腦沒連上（關機、睡眠、遠端關閉） | 請同事開機或在系統匣打開遠端 |
| `MACHINE_TIMEOUT` | 電腦沒在時間內回應，結果不確定 | 先讀回目前狀態，再決定要不要重做 |
| `NOT_PAIRED`、`RELAY_NOT_SET` | 這台電腦還沒配對，或公司的遠端服務還沒設定 | 請同事在系統匣選「遠端控制」；服務沒設定時請等 SKYE 通知 |

## Gemini 與更新

| 錯誤 | 意思 | 你要做的 |
| --- | --- | --- |
| `MODEL_NOT_ARGON`、`MODEL_CHANGED` | Gemini 現在不是 Argon（或中途換掉） | 自己完成這件事，不要換別的模型 |
| `APP_NOT_INSTALLED`、`NOT_SIGNED_IN` | 這台電腦沒有 Gemini App，或還沒登入 | 自己完成；告訴同事可以安裝並自己登入 |
| `EXTENSION_OUTDATED` | 瀏覽器的 SKYE 擴充比 SKYE 舊 | 請同事把瀏覽器關掉再打開 |
| `SKYE_OUTDATED` | 電腦上的 SKYE 比擴充舊，正在自動更新 | 幾分鐘後再試 |
| `UPDATING` | SKYE 正在安裝更新，馬上會重新開啟 | 一分鐘後再試同一個呼叫 |
| `BUSY`（argon） | Gemini 還在回答別的問題 | 等它答完再叫一次 |
| `UNSUPPORTED`（argon 帶了 `mode: task`、`new_chat`、`temporary_chat` 或 `level`） | Windows 上還沒有這些選項 | 不帶這些參數再叫一次 |
