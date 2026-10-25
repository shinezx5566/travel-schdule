# travel-schdule：旅行行程網頁框架

一個在 claude.ai 上以 artifact 執行的多人旅行行程網頁。這個 repo 只放**框架**：網頁程式、說明文件和一份範例內容檔。每趟旅行的內容（日期、人員、行程、分攤表……）不在 repo 裡，而是存成「內容檔」，隨時匯入框架使用。

| 檔案 | 說明 |
|---|---|
| `trip.html` | 網頁本體（單一檔案，HTML＋CSS＋JS） |
| `examples/example-content.json` | 範例內容檔，示範內容檔格式 |
| `README.md` | 本文件：分頁、功能、邏輯與使用方式 |

---

## 1. 快速開始

1. 用 Claude 把 `trip.html` 發布成 claude.ai artifact，並宣告下面「權限與執行能力」一節的 capabilities。
2. 打開網頁：沒有設定時會顯示內建範例（「我的旅行（範例）」，3 天、4 位旅伴）。
3. 擁有者到「花費」分頁最下面的「匯入／匯出內容」，按「匯入內容檔」選擇內容檔，再按一次「確定匯入」。
4. 地圖：若有該趟的 `mapdata.json`，請 Claude 以 artifact 檔案的方式一併發布（網頁讀同目錄的 `mapdata.json`）；沒有的話地圖區塊自動隱藏。
5. 分享：在 artifact 的分享選單邀請同行者。要讓他們寫筆記、使用攜帶清單，需給「Editor」權限。

---

## 2. 角色與權限

| 角色 | 對應 | 可以做的事 |
|---|---|---|
| 擁有者 | artifact 的建立者 | 全部：編輯行程、花費、分攤、待辦、資訊、匯入匯出，切換方案 |
| 編輯者 | 分享選單的 Editor | 看行程；寫自己的筆記（可改自己寫的）；自己的攜帶清單 |
| 檢視者 | Viewer／Commenter | 看行程與筆記 |

- 花費、資訊分頁只有擁有者看得到；其他人底部只會看到「行程、筆記、待辦」三顆按鈕。
- 擁有者右上角有「檢視者畫面」，可預覽家人看到的樣子（不寫入任何資料）。

---

## 3. 分頁與功能

### 3.1 行程（所有人）
- **日期列**：每天一顆，下方色點代表地區；往下捲時縮小並固定在畫面頂端。依日本時間自動跳到「今天」。
- **地圖**：Leaflet 繪製當天景點編號、開車／電車路線；正在移動中的路段會閃爍（依日本時間判斷）。可展開、可開 Google 地圖導航。
- **卡片時間軸**：每張卡片含時間、類別、備註、地址、資訊表（推薦、營業、公休、付款、停車、預約、吸菸、電話）、連結預覽、路線示意圖、PDF；卡片間顯示移動方式與空檔。
- **自動完成**：時間過了的卡片自動打勾（之後手動取消會保留）。
- **方案 A／B**：同一時段標 `plan: "A"`／`"B"` 的兩張卡片互為備案，中間顯示「或」。擁有者按「改選這個方案」切換；未選的方案變淡，不計花費、不參與「現在」標示與移動中路段；若 `mapdata.json` 有該天的 `plans`，地圖路線也跟著換。
- **路線示意圖／PDF**：卡片可帶一張圖（點了放大）與 PDF（轉成頁面圖片在網頁內檢視，可點擊放大）。
- 擁有者可新增、編輯、刪除卡片，金額只有擁有者看得到。

### 3.2 花費（擁有者）
- **分攤表**：每筆帳一張卡片，依「待填／已填完」分區、各自依日期排序。
  - 右上角是墊付人與墊付總額；點擊可換人（含「無墊付」）、改幣別與金額。
  - 每個人一個方塊，依分組排成列：白＝尚欠、黃＝已結、虛線＝未記、淡色＝不分攤。點方塊輸入金額、幣別、尚欠／已結。
  - 上方顯示各組消費與尚欠合計，「結清」按兩下把該組白色方塊一次轉黃。
  - 每張卡片可上傳收據照片。
- **零星花費**：分攤表以外的支出（停車、門票……），可附收據。
- **現金區塊**：來自私人設定的 HTML（例如誰要帶多少現金）。
- **總計**：全團總計（換算台幣）、每人平均、日幣、台幣、依類別、依天數。
- **匯入／匯出內容**：見第 6 節。

### 3.3 筆記（所有人可看，編輯者以上可寫）
- 多則筆記，可標日期、附照片；標示作者，只能改自己寫的（擁有者可改全部）。

### 3.4 待辦
- **注意事項**（設定的 `tips`）：可收合的區塊，`alert: true` 標題為紅字。
- **我的攜帶清單**：每人一份、只有本人看得到；第一次打開自動放入預設項目，可勾選、新增、刪除。
- **行前待辦**：只有擁有者看得到。

### 3.5 資訊（擁有者）
- 私人設定的 `infoHtml`（航班、住宿、租車……）。

---

## 4. 設定（config）

網頁本身不含任何旅行內容，全部來自資料庫的兩份設定；兩份都沒有時使用內建範例。

### 4.1 `meta/config`（所有人可讀，擁有者可寫）
```jsonc
{
  "version": 1,
  "title": "我的旅行",                 // 頁首與瀏覽器標題
  "start": "2027-04-01",              // 第 1 天（日本時間）
  "regions": { "a": { "name": "城市 A", "tone": 1 } },   // tone 1–6：地區色點
  "days": [ { "region": "a", "theme": "抵達、市區散步" } ], // 天數＝陣列長度
  "people": [ { "key": "p1", "name": "我" } ],
  "groups": [ { "id": "A", "name": "A 組", "members": ["p1"] } ], // 分攤表的分組
  "owner": "p1",                      // 預設墊付人（通常是擁有者本人）
  "rate": 0.2,                        // 日幣→台幣匯率
  "costNote": "不含個人購物。",          // 花費頁說明的補充
  "twdLabel": "台幣部分",               // 台幣總計的標籤
  "tips": [ { "title": "搭機注意事項", "alert": true, "html": "<ol class=\"tips\">…</ol>" } ],
  "pack": { "items": ["護照"], "ownerOverrides": { "0": "擁有者版本的第 1 項" } }
}
```

### 4.2 `private/config`（只有擁有者可讀寫）
```jsonc
{ "version": 1, "infoTitle": "航班與住宿", "infoHtml": "<div class=\"box\">…</div>", "cashHtml": "<div class=\"box\">…</div>" }
```

`tips[].html`、`infoHtml`、`cashHtml` 會以 HTML 顯示，可用的樣式：`box`、`list`、`li`、`fine`、`tips`（編號清單）、`hl`（黃色粗體）、`adv-tot`（合計列）。

---

## 5. 資料模型（資料庫集合）

| 路徑 | 讀／寫 | 內容 |
|---|---|---|
| `items/{id}` | 所有人／擁有者 | 行程卡片 |
| `split/{id}` | 擁有者 | 分攤表 |
| `misc/{id}` | 擁有者 | 零星花費 |
| `todos/{id}` | 擁有者 | 行前待辦 |
| `meta/config`、`meta/maps`、`meta/plans` | 所有人／擁有者 | 設定、每天的 Google 地圖連結與備用路線圖、各天選定方案 |
| `private/config` | 擁有者 | 私人設定 |
| `notes/{uid}/items/{id}` | 所有人／本人（擁有者可改全部） | 筆記 |
| `data/users/{uid}/{id}` | 只有本人 | 攜帶清單 |

### 行程卡片 `items`
| 欄位 | 說明 |
|---|---|
| `day`, `time`, `end` | 第幾天、開始與結束時間（`HH:MM`） |
| `cat` | `transport` `food` `sight` `stay` `activity` `shop` |
| `title`, `note` | 標題、備註 |
| `addr`, `addrQuery`, `geo` | 地址、地圖搜尋字、座標 `[lat, lng]`（有座標才上地圖） |
| `yen`, `ntd`, `perPerson` | 全團金額；`perPerson: false` 不顯示每人平均 |
| `move` | `{ mode: car/walk/train/taxi/plane/rest, text }`，到下一站的移動 |
| `info` | `rec` `hours` `closed` `pay` `parking` `reserve` `smoke` `tel` `src` |
| `url`, `preview` | 連結與預覽（`site` `title` `desc` `img`） |
| `plan` | `"A"` 或 `"B"`，見方案 A／B |
| `guide` | 路線示意圖（`/_blob/…`） |
| `pdf` | `{ name, url, pages: ["/_blob/…"] }` |
| `trains` | 電車班次 |
| `done`, `autoDone` | 是否完成、是否為自動打勾 |

### 分攤表 `split`
| 欄位 | 說明 |
|---|---|
| `label` | 以 `M/D` 開頭會歸到該日期，否則在「整趟」 |
| `pay`, `hint`, `time`, `order` | 付款方式、預算提示、排序 |
| `cur` | 預設幣別 `JPY`／`TWD` |
| `na` | 不需分攤的人（`people[].key`） |
| `cells`, `cellState`, `cellCur`, `cellHints` | 每人金額、`owed`／`paid`、幣別、預估提示 |
| `payer`, `paidAmt`, `payCur` | 墊付人（`none`＝無墊付）、墊付總額與幣別 |
| `receipts` | 收據圖片 id |

---

## 6. 內容檔（匯入／匯出）

擁有者在「花費 → 匯入／匯出內容」操作。

- **匯出**：把設定、私人設定與 `items`、`split`、`misc`、`todos`、`meta/maps`、`meta/plans` 存成 `trip-content-YYYY-MM-DD.json`。
- **匯入**：選檔後顯示摘要，再按一次「確定匯入」。檔案裡沒有的文件會被刪除，等於整份取代。
- **不包含**：每個人的攜帶清單與筆記（私人資料）；圖片與 PDF 只存 `/_blob/…` 連結，只在原本那個 artifact 有效，換到新的 artifact 需重新上傳。
- 格式：
```jsonc
{
  "format": "trip-content", "version": 1, "exportedAt": "…",
  "config": { … }, "private": { … },
  "data": { "items": { "id": { … } }, "split": { … }, "misc": { … }, "todos": { … },
            "meta": { "maps": { … }, "plans": { "3": "A" } } }
}
```

---

## 7. 地圖資料 `mapdata.json`

與網頁一起發布的靜態檔：`water`、`major`、`minor`（底圖）、`places`（地名）、`days`（每天的路線段 `{ k: "road"|"rail", c: [[lat,lng],…] }`）、`plans`（`{ "天": { "A": [...], "B": [...] } }`，方案路線）。路線段端點對應相鄰兩張有座標的卡片，用來判斷「移動中」。路線可用 OSRM 計算。

---

## 8. 權限與執行能力（發布時宣告）

```json
{
  "assets": {}, "downloads": true, "user": { "scopes": ["profile"] },
  "db": { "rules": [
    { "path": "", "read": "view", "write": "owner" },
    { "path": "split", "read": "owner", "write": "owner" },
    { "path": "misc", "read": "owner", "write": "owner" },
    { "path": "todos", "read": "owner", "write": "owner" },
    { "path": "private", "read": "owner", "write": "owner" },
    { "path": "notes", "read": "view", "write": "owner" },
    { "path": "notes/{self}", "write": "admin" },
    { "path": "data/users/{self}", "write": "admin" }
  ] }
}
```

---

## 9. 實作重點

- 單一 IIFE；`applyConfig()` 由設定推出 `START`、`DAYS`、`REGIONS`、`PEOPLE`、`BILL_GROUPS` 等，設定變更時重畫全頁。
- 時間一律用日本時間（`Asia/Tokyo`）判斷今天、現在進行中的卡片與移動中路段。
- iPhone 相容：數字輸入以 focusout／Enter 存檔（不依賴 change 事件）；鍵盤需在點擊當下 focus 才會出現；PDF 改以頁面圖片在網頁內顯示（App 內開新分頁會被擋）。
- 幣別目前固定日幣／台幣。
