# 素材網站、上架自動找圖、一句話換圖、影音（Windows）

## 素材網站怎麼用

| 網站 | 怎麼取得 | 備註 |
| --- | --- | --- |
| Wikimedia Commons（`wikimedia`）、Openverse（`openverse`） | SKYE 直接搜尋與下載 | 免費授權，每張授權不同（CC BY、CC BY-SA…）：上架時依授權標示作者 |
| Pexels（`pexels`）、Shutterstock（`shutterstock`）、Adobe Stock（`adobe_stock`） | 公司填好 API 金鑰後由 SKYE 搜尋；還沒有金鑰時跟下一列一樣 | Adobe Stock 的授權要同事的 Adobe 登入，所以一律由同事按下載 |
| Envato Elements（`envato`）、Artlist（`artlist`）、Canva（`canva`） | 在同事自己已登入的瀏覽器搜尋；**同事本人按一次下載** | 這些網站的條款禁止程式自動下載，帳號可能被停權 |

流程：

1. `asset_search`（`site`、`query`、`kind` photo／video／music；上架用時加 `travel_date` 與 `country`，回覆會附當地季節）。
   - 有 API 的網站會回候選：小縮圖網址、標題、關鍵字、授權、作者、尺寸，以及網站有提供時的拍攝地點與日期。
   - 其他網站會回 `search_url`：用 `browser_open` 在同事的瀏覽器開，再 `browser_read` 看候選。
2. 依下面的三條件挑。
3. `asset_download`：
   - 有 API 的網站直接下載。
   - 其他網站要帶 `page_url`（那張素材的頁面）。SKYE 會打開那頁並在背景等待；請同事按網站的下載按鈕，按完 SKYE 自動歸檔、寫紀錄。用 `task_status` 等結果。
4. 下載的檔案放在「下載\素材\<網站>\」。每一筆都會寫進「素材紀錄.jsonl」：網站、素材 ID、來源、授權、扣點、檔案、SHA-256。
   - **同一個素材不會下載或付費第二次**，會直接回已經有的檔案。
   - 回報時說「這次新下載幾個、用掉幾點」，扣點看 `asset_record`。

## 挑圖三條件（都要符合）

| 條件 | 例 | 怎麼看 |
| --- | --- | --- |
| 地區 | 「清水寺」不能用中國的寺廟冒充 | 標題、關鍵字、拍攝地點、縮圖 |
| 季節 | 12 月紐西蘭是**夏天**；10 月京都是紅葉初期；赤道附近沒有四季 | `travel_date`＋`country` 算出的季節，再看縮圖（雪、紅葉、花） |
| 行為 | 「搭纜車」「品酒」「看極光」 | 標題、關鍵字、縮圖 |

找不到三條件都符合的素材時，跟同事說缺哪一項，不要硬放。

## 上架自動找圖

1. 先找公司自己的照片：`search_files` 搜 NAS 照片庫（國家、城市、景點名稱）。
2. 沒有合適的，再到素材網站找（流程同上）。
3. 每個位置用 `asset_place`：
   - 必填 `site`（例如 crowntw）、`page`（團號）、`slot`（例如 cover、day3.photo1）、`asset`。
     - `asset` 填 `asset_download` 回的 ref，或 NAS 照片的檔案路徑。
   - `basis` 寫三條件，每條附上對應的文案段落。
   - 網站要固定尺寸時，帶 `width`／`height`，SKYE 會置中裁切成剛好的大小。
4. `browser_upload` 上傳回覆裡的 `upload_file`，然後回讀頁面（照片出現、尺寸正確）。

## 一句話換圖、換回

- 同事說「第三天的照片換一張有雪的」：
  1. `asset_record` 帶 `page` 查各位置現在用的素材與理由。
  2. 找新候選。
  3. `asset_replace`，`items` 帶 site、page、slot、asset、basis。
- 多個位置可以一次換：`items` 放多筆。
- 換回原本的：`asset_replace` 帶 `revert: true`，`items` 只要 site、page、slot。原本的理由也會一起回來。
- 換完一樣要上傳並回讀。

## 影音

- `media_control`：
  - `action` 可用：status（正在播什麼、哪個 App）、play、pause、toggle、next、previous、stop、volume_up、volume_down、mute。
  - 不用把 App 叫到前面。
  - 做完看回覆裡的 `status` 確認；剛開始播放時可能要再按一次。
- `media_search_play`：
  - 播什麼由你挑。挑好後 `item` 帶 YouTube 影片 ID 或連結、`spotify:track:…` 或 open.spotify.com 連結，SKYE 會打開來播放。
  - 不帶 `item` 時，只打開搜尋頁讓同事自己選。
  - 電腦上或 NAS 的歌與影片用 `source: "local"`：先列出符合 `query` 的檔案，再用 `item` 帶路徑播放。
- `media_edit`：用 Windows 內建的轉檔功能，不用另外安裝軟體。
  - 剪片段：`start`、`end`，可寫秒數或 mm:ss。
  - 轉格式：`format` mp4、m4a、mp3、wav。
  - 壓縮：`target_mb`（例如 LINE 傳送上限）。
  - 結果是新檔，名稱加「_剪輯／_轉檔／_壓縮」，原檔不動；比較久時用 `task_status` 等。
