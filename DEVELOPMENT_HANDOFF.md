# Eternal Shift｜牙醫診所排班系統完整開發交接文件

> 文件用途：將本檔交給其他工程師、外包團隊、Cursor、Claude、Codex 或其他開發環境，即可在不遺失產品背景與既有規則的前提下接續開發。  
> 專案性質：**院內人員排班系統，不是病患預約掛號系統。**  
> 客製診所：依貝美學牙醫診所（Eternal Dental Clinic）  
> 目前版本：`v1.1.1`  
> 文件更新日：`2026-07-26`  
> GitHub：`https://github.com/JustinFang1019/eternal-dental-shift`  
> 診所官網：`https://eternal-dental.com/`

---

## 0. 可直接貼給其他 AI／開發者的任務指令

```text
請接續開發 Eternal Shift 牙醫診所「院內人員排班系統」。

重要定位：
1. 這是醫師、牙助、櫃檯、衛教師、牙技師、行政的排班系統。
2. 不是病患掛號、病歷、療程或診間預約系統。
3. UI 與文件一律使用繁體中文。
4. 必須保留「官網公開資料／系統範例資料／人工調整資料」的來源標示。
5. 不得把範例牙助與櫃檯姓名描述成診所真實員工。
6. 現有單檔 HTML 的功能不可退化；改寫框架前要先建立自動測試與資料遷移。
7. 所有日期以 Asia/Taipei 的診所本地日曆理解，資料庫日期欄位應使用 DATE，
   班別時間使用 local time，不要以 UTC timestamp 取代純日期／純時間。
8. 不得加入病患個資；正式上線前要有登入、權限、稽核、備份及多人同步。
9. 修改資料格式時必須提供 migration，不能只更換 LocalStorage key 造成舊資料消失。
10. 請先閱讀本文件的資料模型、商業規則、已知缺口與驗收案例，再開始修改。
```

---

## 1. 專案摘要

Eternal Shift 是依貝美學牙醫診所客製的院內排班工具。現行版本是零相依、可離線執行的單檔 HTML，包含 CSS、JavaScript、Logo 與預載範例資料，使用瀏覽器 `LocalStorage` 保存資料。

### 1.1 產品目標

- 將醫師公開門診表與院內人力排班放在同一個週視圖管理。
- 讓排班管理者快速辨識人力缺口、重疊班次、核准休假衝突與工時超限。
- 支援單機操作、備份、匯出與列印，作為正式後端版之前的可操作原型。
- 保留資料來源，避免公開醫師門診與虛構展示人員混淆。

### 1.2 非目標

目前版本不處理以下項目：

- 病患掛號、病歷、療程、收費、健保申報。
- 病患與醫師的診間預約。
- 薪資、加班費、出勤打卡與法定工時計薪。
- 多人即時共同編輯。
- 帳號登入、角色權限、伺服器稽核與雲端備份。
- LINE／Email／App 推播。
- 完整勞動法規判斷。

### 1.3 主要使用者

- **排班管理者／行政**：建立班次、處理休假、查看缺口與匯出。
- **主管／院長**：核准休假、檢視人力與工時風險。
- **一般員工（未來）**：查看自己的班表、提出休假或換班。
- **系統管理員（未來）**：管理帳號、權限、分院、稽核與備份。

---

## 2. 現行版本快照

| 項目 | 內容 |
|---|---|
| 版本 | `1.1.1` |
| 原始檔大小 | `99,990` bytes |
| 原始 HTML SHA-256 | `5c6d1bf7997a771298353506a570fb89333518cbba81d2b113f6d5b7054765f7` |
| LocalStorage key | `eternal_shift_v1_1_1` |
| 技術 | HTML5、CSS3、Vanilla JavaScript |
| 外部相依 | 無 |
| 外部 CDN | 無 |
| 後端 | 無 |
| 資料庫 | 無 |
| 預設時區語意 | Asia/Taipei／瀏覽器本地時間 |
| 週起始日 | 星期一 |
| 預載公開門診日期 | 26 天 |
| 預載公開醫師班次 | 193 筆 |
| 預載展示人力班次 | 207 筆 |
| 預設總班次 | 400 筆 |
| 預設人員 | 21 人：10 位醫師、11 位展示人員 |
| 預設班別 | 早診、午診、晚診 |
| 預設休假 | 1 筆待審展示資料 |

### 2.1 建議的原始碼來源

在其他環境開始開發時，優先使用：

1. Repository 內的 `Eternal_Shift_v1.1.1.html`，它是完整單檔版本。
2. 或 Release 附件 `Eternal_Shift_v1.1.1.html`。
3. 若從 `gh-pages` 分支取得，根目錄 `index.html` 應為同一份完整應用。

不要只拿到載入器版本的 `index.html` 而漏掉 `payload/`；最安全的基準是完整的 `Eternal_Shift_v1.1.1.html`。

---

## 3. 功能總覽

| 模組 | 內容 | 狀態 |
|---|---|---|
| 營運總覽 | 本週總工時、缺口班別、排班警示、待審休假、覆蓋摘要 | 已完成 |
| 週排班表 | 班別／人員／診間三種視角、週切換、搜尋、職務篩選、拖曳 | 已完成 |
| 班次管理 | 新增、編輯、刪除、複製上週、支援／待命標記 | 已完成 |
| 人員管理 | 新增、編輯、停用、職務、工時上限、可排日、專長 | 已完成 |
| 休假管理 | 申請、編輯、核准、駁回、刪除、影響班次提示 | 已完成 |
| 規則與缺口 | 重疊、核准休假、超時、最低人力、自動補缺 | 已完成 |
| 報表 | 工時使用率、衝突、班別覆蓋、CSV | 已完成 |
| 資料管理 | LocalStorage、JSON 備份／還原、重設範例 | 已完成 |
| 正式營運能力 | 登入、權限、多人同步、後端稽核、通知 | 尚未實作 |

---

## 4. 頁面與功能規格

### 4.1 營運總覽

顯示目前選定週的：

- 排定總工時與班次數。
- 人力缺口班別數。
- 排班警示數。
- 待審休假數。
- 每日每班別的最低人力達標狀態。
- 快速進入排班、人員、休假、報表。

#### 驗收重點

- 所有數字必須跟目前週一致。
- 總工時為所有角色班次工時合計。
- 缺口依「開診日 × 該日適用班別 × 職務最低需求」計算。
- 排班警示包含重疊、核准休假衝突與每週工時超限。

### 4.2 週排班表

支援三種視角：

1. **按門診班別**：列為早診、午診、晚診；格內顯示所有人員。
2. **按人員**：列為人員；格內顯示該人員當日所有班次與核准休假。
3. **按診間**：列為診間／工作區；格內顯示該空間的班次。

共同能力：

- 上一週、下一週、回到本週。
- 搜尋人員、專長、班別或診間。
- 依職務篩選。
- 從空白格直接新增班次。
- 點擊班次編輯或刪除。
- 桌面瀏覽器拖曳調班。
- 列印 A4 橫向排班表。
- 複製上週。
- 自動補足最低人力缺口。
- 匯出 CSV。

#### 班次卡顯示

- 開始與結束時間。
- 視角相對應的人員、班別、職務與診間。
- `支援／待命` 標記。
- `官網`、`範例` 或無標籤的人工資料來源。
- 發生衝突時以警示樣式顯示。

### 4.3 人員管理

欄位：

- 姓名。
- 職務：牙醫師、牙助、櫃檯、衛教師、牙技師、行政。
- 聘用型態／職稱。
- 每週工時上限，1–80 小時，可用 0.5 小時。
- 電話。
- 識別色。
- 專長／備註，以逗號分隔。
- 可排星期。
- 啟用狀態。
- 是否為系統範例人員。

行為：

- 新增與編輯。
- 有班次或休假參照時，「刪除」改為停用。
- 無任何參照時可實體刪除。
- 停用人員不參與自動補缺；既有歷史班次仍保留。
- 搜尋姓名與專長，並依職務、啟用狀態篩選。

### 4.4 休假管理

欄位：

- 人員。
- 開始日期與結束日期。
- 類型：特休、病假、事假、公假、教育訓練、家庭照顧。
- 狀態：待審核、已核准、已駁回。
- 備註。

行為：

- 新增、編輯、刪除。
- 待審項目可直接核准或駁回。
- 核准時若期間已有班次，顯示影響數量並要求確認。
- 核准後不自動刪除既有班次，而是在衝突報表中顯示。
- 核准休假日期內禁止新增或拖入新班次。
- 目前只支援整日日期範圍，不支援半日或時段假。

### 4.5 工時與人力缺口

包含：

- 本週排定總工時。
- 超過工時上限的人員數。
- 排班警示數。
- 缺口班別數。
- 每位啟用人員的工時、上限、使用率與狀態。
- 重疊、休假衝突、超時的明細。
- 每日每班別、每職務的實際人數／最低需求。

### 4.6 班別與系統設定

診所設定：

- 診所名稱。
- 電話。
- 地址。
- 診間／工作區，一行一個。
- 固定每週營業日。

班別設定：

- 班別名稱。
- 識別色。
- 開始／結束時間。
- 適用星期。
- 各職務最低人力需求。

資料管理：

- 匯出 JSON。
- 匯入 JSON。
- 重設為依貝範例資料。

### 4.7 Release Notes

應呈現版本、日期、修正、資料假設、已知限制與後續規劃。每次功能改動都要同步更新：

- 應用內 Release Notes。
- Repository 的 `RELEASE_NOTES.md`。
- 版本號與 LocalStorage schema migration。
- 測試報告。

---

## 5. 核心商業規則

### 5.1 日期與星期

- 日期字串格式固定為 `YYYY-MM-DD`。
- 星期使用 JavaScript `Date.getDay()`：
  - `0` 週日
  - `1` 週一
  - `2` 週二
  - `3` 週三
  - `4` 週四
  - `5` 週五
  - `6` 週六
- 週視圖從星期一開始，到星期日結束。
- 目前沒有國定假日、臨時休診或特殊營業日覆寫模型。

### 5.2 時間與工時

- 班次以同日 `HH:mm` 儲存。
- 現行版本要求 `end > start`，因此不支援跨午夜班次。
- 工時 = `(結束分鐘 - 開始分鐘) / 60`。
- 顯示與警示四捨五入至小數一位。
- `支援／待命` 現在與一般班次同樣計入工時及最低人力。

### 5.3 重疊判斷

使用半開區間：

```text
existing.start < candidate.end AND candidate.start < existing.end
```

因此：

- 09:00–12:00 與 12:00–15:00 不算重疊。
- 09:00–12:00 與 11:59–15:00 算重疊。
- 只檢查同一人員、同一日期。
- 現行版本**沒有檢查診間同時被多人占用**。

### 5.4 休假規則

- 只有 `approved` 休假會阻擋新排班。
- `pending` 與 `rejected` 不阻擋。
- 休假日期範圍包含起訖日。
- 核准休假不刪除既有班次，而是建立可見衝突。

### 5.5 每週工時

- 人員具有 `weeklyMax`。
- 手動新增／編輯若超時，顯示確認對話框，使用者仍可強制儲存。
- 自動補缺不得讓人員超過 `weeklyMax`。
- 拖曳調班目前沒有重新檢查週工時上限，後續應補上。

### 5.6 最低人力覆蓋

對每個：

```text
開診日期 × 該星期適用的班別 × 每個職務
```

計算：

```text
actual = 當日且 typeId 相同之班次人數
missing = max(required - actual, 0)
```

注意：

- 目前以 `typeId` 計數，不會再比對班次實際開始／結束時間。
- 目前每筆班次都計為 1 人，不處理 FTE 比例。
- 同一人同一班別若有兩筆不重疊班次，可能被計數兩次；正式版應定義唯一性。
- `onCall` 現在也會算入最低人力，正式版應確認是否合理。

### 5.7 自動補缺演算法

對每一個缺口角色，候選人必須同時符合：

1. `active === true`
2. `role` 相同
3. `days` 包含該星期
4. 當日沒有核准休假
5. 與現有班次及本次待新增班次不重疊
6. 新增後不超過 `weeklyMax`

排序：

1. 本週目前工時較少者優先。
2. 同工時時依繁體中文姓名排序。

新增班次：

- 使用班別起訖時間。
- 使用職務預設診間。
- 備註為「系統自動補缺」。
- 範例人員的來源為 `demo`；其他人員為 `manual`。

這是啟發式補缺，不是最佳化排班，也未處理偏好、連續工時、公平輪值、技能需求、休息時間或成本。

### 5.8 複製上週

- 取目前週前 7 天的所有班次。
- 日期全部加 7 天。
- 新 ID、新 `createdAt`。
- 官網來源 `official` 複製後改為 `manual`。
- 範例來源維持 `demo`。
- 跳過完全相同的人員、日期、開始與結束時間。
- 跳過目標日期已有核准休假者。
- 不會重新檢查可排星期、營業日、班別適用日、診間衝突或工時。

### 5.9 拖曳調班

視角造成的修改：

- 人員視角：可改日期與人員。
- 診間視角：可改日期與診間。
- 班別視角：可改日期與班別；改班別時時間改為該班別預設時間。

重新檢查：

- 人員重疊。
- 核准休假。

目前未重新檢查：

- 工時上限。
- 人員可排星期。
- 診所營業日。
- 班別適用星期。
- 診間重疊。
- 技能資格。

### 5.10 資料來源

`Shift.source`：

- `official`：由診所官網公開門診表匯入。
- `demo`：功能展示用人力班次。
- `manual`：人工新增、人工修改、複製或移動後的非展示班次。

現行規則：

- 編輯或移動 `official` 班次後改為 `manual`。
- 複製 `official` 班次後改為 `manual`。
- `demo` 編輯或移動後仍維持 `demo`。

---

## 6. 完整資料模型

建議以以下 TypeScript 介面作為重構時的基準。

```ts
type Role =
  | "牙醫師"
  | "牙助"
  | "櫃檯"
  | "衛教師"
  | "牙技師"
  | "行政";

type LeaveStatus = "pending" | "approved" | "rejected";
type ShiftSource = "official" | "demo" | "manual";

interface AppState {
  version: string;
  clinic: Clinic;
  staff: Staff[];
  types: ShiftType[];
  shifts: Shift[];
  leaves: Leave[];
}

interface Clinic {
  name: string;
  english?: string;
  phone?: string;
  address?: string;
  rooms: string[];
  openDays: number[]; // 0=週日，1=週一 ... 6=週六
}

interface Staff {
  id: string;
  name: string;
  role: Role;
  employment: string;
  weeklyMax: number;
  phone: string;
  color: string;      // #RRGGBB
  skills: string[];
  days: number[];     // 可排星期
  active: boolean;
  demo: boolean;
}

interface ShiftType {
  id: string;
  name: string;
  start: string;      // HH:mm
  end: string;        // HH:mm
  color: string;      // #RRGGBB
  days: number[];
  req: Record<Role, number>;
}

interface Shift {
  id: string;
  date: string;       // YYYY-MM-DD
  personId: string;
  typeId: string;
  start: string;      // HH:mm
  end: string;        // HH:mm
  room: string;
  note: string;
  onCall: boolean;
  source: ShiftSource;
  createdAt: string;  // ISO datetime
}

interface Leave {
  id: string;
  personId: string;
  start: string;      // YYYY-MM-DD
  end: string;        // YYYY-MM-DD
  kind: string;
  status: LeaveStatus;
  note: string;
  createdAt: string;  // ISO datetime
}
```

### 6.1 完整狀態範例

```json
{
  "version": "1.1.1",
  "clinic": {
    "name": "依貝美學牙醫診所",
    "english": "Eternal Dental Clinic",
    "phone": "03-5510999",
    "address": "新竹縣竹北市環北路五段408號",
    "rooms": ["診間 1", "顯微根管室", "櫃檯"],
    "openDays": [1, 2, 3, 4, 5, 6]
  },
  "staff": [
    {
      "id": "doc_wang",
      "name": "王俊智",
      "role": "牙醫師",
      "employment": "院長",
      "weeklyMax": 48,
      "phone": "",
      "color": "#76566f",
      "skills": ["一日假牙", "微創植牙"],
      "days": [1, 2, 3, 4, 5, 6],
      "active": true,
      "demo": false
    }
  ],
  "types": [
    {
      "id": "m",
      "name": "早診",
      "start": "09:30",
      "end": "12:30",
      "color": "#b0877d",
      "days": [1, 2, 3, 4, 6],
      "req": {
        "牙醫師": 1,
        "牙助": 2,
        "櫃檯": 1,
        "衛教師": 0,
        "牙技師": 0,
        "行政": 0
      }
    }
  ],
  "shifts": [
    {
      "id": "sh_example",
      "date": "2026-08-03",
      "personId": "doc_wang",
      "typeId": "m",
      "start": "09:30",
      "end": "12:30",
      "room": "診間 1",
      "note": "依官網門診表匯入",
      "onCall": false,
      "source": "official",
      "createdAt": "2026-07-26T00:00:00.000Z"
    }
  ],
  "leaves": [
    {
      "id": "lv_example",
      "personId": "ast_5",
      "start": "2026-09-02",
      "end": "2026-09-02",
      "kind": "特休",
      "status": "pending",
      "note": "範例休假申請",
      "createdAt": "2026-07-26T00:00:00.000Z"
    }
  ]
}
```

### 6.2 ID 產生方式

現行單機版使用：

```text
prefix + "_" + Date.now().toString(36) + "_" + random(5 chars)
```

正式多人版必須改用 UUID／ULID，由後端或可信任資料層建立，不能依賴瀏覽器時間與隨機短字串。

---

## 7. 預設診所設定

### 7.1 基本資料

| 欄位 | 值 |
|---|---|
| 名稱 | 依貝美學牙醫診所 |
| 英文 | Eternal Dental Clinic |
| 電話 | 03-5510999 |
| 地址 | 新竹縣竹北市環北路五段 408 號 |
| 固定營業日 | 週一至週六 |
| 週日 | 預設休診 |

### 7.2 預設職務

- 牙醫師
- 牙助
- 櫃檯
- 衛教師
- 牙技師
- 行政

新增職務目前需要改程式常數 `ROLES`；正式版應改成可設定資料表，但必須處理既有報表與最低人力欄位遷移。

### 7.3 預設診間／工作區

1. 診間 1
2. 診間 2
3. 診間 3
4. 診間 4
5. 兒童診療室
6. 顯微根管室
7. 矯正診間
8. 手術室
9. X 光室
10. 牙技室
11. 櫃檯
12. 行政區

### 7.4 預設班別

| ID | 名稱 | 示範時間 | 適用星期 | 最低人力 |
|---|---|---|---|---|
| m | 早診 | 09:30–12:30 | 一、二、三、四、六 | 牙醫師 1、牙助 2、櫃檯 1 |
| a | 午診 | 14:00–17:30 | 一、二、三、四、五、六 | 牙醫師 1、牙助 2、櫃檯 1 |
| e | 晚診 | 18:00–20:30 | 一、二、三、四、五 | 牙醫師 1、牙助 2、櫃檯 1 |

> 上述時間、適用星期與最低人力均為示範設定，正式導入前須由診所管理者確認。

### 7.5 預設醫師

| 姓名 | 職稱 | 示範週工時上限 | 示範可排日 | 專長／標籤 |
|---|---|---|---|---|
| 王俊智 | 院長 | 48 | 一、二、三、四、五、六 | 一日假牙、微創植牙、All-On-4/6、陶瓷貼片、智齒拔除、隱適美 |
| 陳郁 | 副院長 | 40 | 一、三、四、五 | 一日假牙、全瓷牙冠、牙周水雷射、隱適美 |
| 金宏亮 | 牙髓病專科醫師 | 24 | 二、四、五 | 顯微根管治療、活髓保存術、牙根再生術 |
| 林伊柔 | 牙髓病專科醫師 | 24 | 二、四、五 | 顯微根管治療、活髓保存術、牙根再生術 |
| 駱星任 | 兒童牙科專科醫師 | 24 | 二、四、六 | 兒童全口治療、兒童舒眠麻醉、肌功能矯正、兒童隱適美 |
| 詹景淳 | 兒童牙科專科醫師 | 24 | 一、四、五 | 兒童全口治療、兒童舒眠麻醉、肌功能矯正、兒童隱適美 |
| 彭心沛 | 齒顎矯正專科醫師 | 24 | 二、四、六 | 傳統齒顎矯正、自鎖式矯正、隱適美、正顎合併矯正 |
| 陳妍容 | 家庭牙科專任醫師 | 40 | 一、三、四、五 | 家庭牙科、一般根管治療、牙周治療、全瓷牙冠、牙齒美白 |
| 林子涵 | 家庭牙科專任醫師 | 32 | 二、六 | 家庭牙科、全瓷牙冠、3D 齒雕、牙周治療 |
| 林瑞元 | 家庭牙科專任醫師 | 24 | 一 | 家庭牙科、一般根管治療、牙周治療、全瓷牙冠、牙齒美白 |

> 姓名與專長來自診所公開資料；每週工時上限、可排星期與診間屬於系統示範設定，不代表院內真實規則。

### 7.6 展示人員

- 牙助 A01–A06，共 6 人。
- 櫃檯 F01–F02，共 2 人。
- 衛教師 H01，共 1 人。
- 牙技師 T01，共 1 人。
- 行政 O01，共 1 人。

全部名稱均含「（範例）」或以範例欄位標示，不得對外宣稱為真實員工。

---

## 8. 2026 年 8 月公開醫師門診資料

- 日期數：26。
- 班別格數：69。
- 醫師班次指派：193 筆。
- `m` = 早診、`a` = 午診、`e` = 晚診。
- 已確認修正：`2026-08-20 午診`為王俊智、陳妍容、金宏亮、彭心沛。

| 日期 | 早診 m | 午診 a | 晚診 e |
|---|---|---|---|
| 2026-08-01 | 林子涵、駱星任、彭心沛 | 林子涵、駱星任、彭心沛 | — |
| 2026-08-03 | 王俊智、陳郁、林瑞元 | 王俊智、陳妍容、林瑞元、詹景淳 | 王俊智、陳妍容、詹景淳 |
| 2026-08-04 | 林子涵 | 金宏亮、林子涵、駱星任、彭心沛 | 金宏亮、林子涵、駱星任、彭心沛 |
| 2026-08-05 | 王俊智、陳郁 | 王俊智、陳妍容 | 王俊智、陳妍容 |
| 2026-08-06 | 陳妍容 | 王俊智、陳郁、陳妍容、彭心沛 | 王俊智、陳郁、駱星任、彭心沛 |
| 2026-08-07 | — | 王俊智、陳妍容、金宏亮 | 王俊智、陳妍容、金宏亮 |
| 2026-08-08 | 林子涵、駱星任、彭心沛 | 林子涵、駱星任、彭心沛 | — |
| 2026-08-10 | 王俊智、陳郁、林瑞元 | 王俊智、林瑞元、詹景淳、陳妍容 | 王俊智、陳妍容、詹景淳 |
| 2026-08-11 | 林子涵 | 王俊智、林子涵 | 王俊智、駱星任、彭心沛、林子涵 |
| 2026-08-12 | 王俊智、陳郁 | 王俊智、陳妍容 | 王俊智、陳妍容 |
| 2026-08-13 | 陳妍容 | 王俊智、陳郁、陳妍容、彭心沛 | 王俊智、陳郁、駱星任、彭心沛 |
| 2026-08-14 | — | 陳妍容、詹景淳、金宏亮 | 陳妍容、詹景淳、金宏亮 |
| 2026-08-15 | 王俊智、林子涵、駱星任 | 王俊智、林子涵 | — |
| 2026-08-17 | 王俊智、陳郁、林瑞元 | 王俊智、陳妍容、林瑞元 | 王俊智、陳妍容、詹景淳 |
| 2026-08-18 | 林子涵 | 王俊智、林子涵、彭心沛 | 王俊智、林子涵、駱星任、彭心沛 |
| 2026-08-19 | 王俊智、陳郁 | 王俊智、陳妍容 | 王俊智、陳妍容 |
| 2026-08-20 | 陳妍容 | 王俊智、陳妍容、金宏亮、彭心沛 | 王俊智、金宏亮、駱星任、彭心沛 |
| 2026-08-21 | — | 王俊智、陳郁、陳妍容、金宏亮 | 王俊智、陳郁、陳妍容、金宏亮 |
| 2026-08-22 | 王俊智、林子涵、彭心沛 | 王俊智、林子涵、駱星任、彭心沛 | — |
| 2026-08-24 | 王俊智、陳郁、林瑞元 | 王俊智、陳妍容、林瑞元 | 王俊智、陳妍容 |
| 2026-08-25 | 林子涵 | 王俊智、林子涵、彭心沛 | 王俊智、林子涵、駱星任、彭心沛 |
| 2026-08-26 | 王俊智、陳郁 | 王俊智、陳妍容 | 王俊智、陳妍容 |
| 2026-08-27 | 陳妍容 | 王俊智、陳郁、陳妍容、彭心沛 | 王俊智、陳郁、駱星任、彭心沛 |
| 2026-08-28 | — | 陳妍容、詹景淳、金宏亮 | 陳妍容、詹景淳、金宏亮 |
| 2026-08-29 | 王俊智、林子涵、駱星任 | 王俊智、林子涵、駱星任 | — |
| 2026-08-31 | 王俊智、陳郁 | 王俊智、陳妍容、詹景淳 | 王俊智、陳妍容、詹景淳 |

---

## 9. 現行單檔 HTML 架構

### 9.1 全域常數

```js
const KEY = "eternal_shift_v1_1_1";
const ROLES = ["牙醫師", "牙助", "櫃檯", "衛教師", "牙技師", "行政"];
const DAYS = ["週日", "週一", "週二", "週三", "週四", "週五", "週六"];
const PAGE_META = { ... };
const PUBLIC_ROSTER = { ... };
```

### 9.2 全域狀態

```js
let state = load();
let ui = {
  page: "schedule",
  week: initialWeek(),
  view: "session",
  dragId: null
};
```

### 9.3 主要函式分群

#### 初始化與儲存

- `makeDefault()`
- `seedRoster(state)`
- `load()`
- `save(message)`

#### 日期與時間

- `parseDate()`
- `iso()`
- `addDays()`
- `startWeek()`
- `weekDays()`
- `mins()`
- `hours()`
- `overlaps()`

#### 查詢與規則

- `person(id)`
- `type(id)`
- `shiftsWeek()`
- `staffHours()`
- `approvedLeave()`
- `directConflict()`
- `conflicts()`
- `coverage()`

#### 畫面

- `renderAll()`
- `renderDashboard()`
- `renderSchedule()`
- `renderStaff()`
- `renderLeave()`
- `renderReports()`
- `renderSettings()`

#### CRUD

- `submitShift()`／`deleteShift()`
- `submitStaff()`／`deleteStaff()`
- `submitLeave()`／`deleteLeave()`／`updateLeave()`
- `submitType()`／`deleteType()`

#### 排班操作

- `copyWeek()`
- `autoFill()`
- `moveShift()`

#### 資料交換

- `exportJson()`
- `exportCsv()`
- `importFile()`
- `download()`

### 9.4 重構建議

不要直接把 100 KB 單檔一次拆完。建議順序：

1. 先建立 Cypress／Playwright E2E，固定現行行為。
2. 將純函式抽出：
   - `date.ts`
   - `schedule-rules.ts`
   - `coverage.ts`
   - `state-migrations.ts`
3. 將資料存取抽象成 `Repository`：
   - `LocalStorageRepository`
   - `ApiRepository`
4. 再拆 UI component／page。
5. 最後才更換框架或加入後端。

---

## 10. LocalStorage、備份與資料遷移

### 10.1 現行保存方式

```js
localStorage.setItem("eternal_shift_v1_1_1", JSON.stringify(state));
```

重新整理後由相同 key 載入。若解析失敗，回到預設資料。

### 10.2 現行匯入驗證限制

目前只確認：

- `staff` 是陣列。
- `shifts` 是陣列。
- `types` 是陣列。

沒有完整 schema validation，也沒有欄位修復、版本 migration、外鍵完整性檢查、重複 ID 檢查。

### 10.3 必須新增的 migration 機制

建議保持固定 key，例如：

```text
eternal_shift_state
```

State 內使用數字 schema version：

```json
{ "schemaVersion": 2 }
```

啟動流程：

```ts
const raw = storage.read();
const migrated = migrateToLatest(raw);
validate(migrated);
storage.write(migrated);
```

每次 migration 必須：

- 可從上一正式版升級。
- 不丟失班次與休假。
- 有單元測試。
- 匯出前先使用最新 schema。
- 失敗時保留原始備份，不可靜默重設範例。

### 10.4 建議備份策略

單機版：

- 每日或每次重大調班後匯出 JSON。
- 匯入前自動建立目前狀態快照。
- 提供最近 10 版回復。

後端版：

- 資料庫 point-in-time recovery。
- 每日快照。
- 排班版本表。
- 操作稽核 log。
- 可依週／月份回復，而非整庫覆蓋。

---

## 11. 匯出格式

### 11.1 JSON

檔名：

```text
Eternal_Shift_backup_YYYY-MM-DD.json
```

內容為完整 `AppState`。

### 11.2 CSV

檔名：

```text
Eternal_Shift_schedule_YYYY-MM-DD.csv
```

欄位順序：

1. 日期
2. 星期
3. 人員
4. 職務
5. 職稱
6. 班別
7. 開始
8. 結束
9. 工時
10. 診間
11. 資料標示
12. 備註

CSV 前綴 UTF-8 BOM，文字含逗號、雙引號或換行時需正確 quote。

### 11.3 未來建議

- 月門診表 Excel／PDF。
- ICS 行事曆。
- 員工個人班表。
- 薪資／出勤系統交換格式。
- 官網公告圖輸出。
- API Webhook。

---

## 12. UI／UX 規格

### 12.1 視覺語言

主要 CSS 變數：

```css
--bg: #f7f3f0;
--surface: #fffdfb;
--surface2: #fbf7f4;
--text: #453d42;
--muted: #7c7177;
--line: #e7ddd8;
--primary: #7b6678;
--primary2: #5f4d5b;
--primarySoft: #eee6eb;
--danger: #b42318;
--warn: #b54708;
--ok: #027a48;
--info: #175cd3;
```

方向：

- 米杏底色、藕紫主色。
- 卡片化、圓角、低對比陰影。
- 官網、範例、警示有明確標籤，不只依賴顏色。
- 重要操作使用繁體中文，避免英文縮寫。

### 12.2 響應式斷點

- `1100px`：縮減雙欄／統計配置。
- `760px`：側欄改抽屜、表單單欄、Modal 底部彈出。
- `470px`：統計卡改單欄，資料來源提示改垂直。
- 排班表本身允許橫向捲動，不強行壓縮所有欄位。

### 12.3 無障礙改善清單

現行版本可用，但正式版應補：

- Modal focus trap。
- 開啟 Modal 後聚焦第一個欄位。
- 關閉後回到觸發按鈕。
- 鍵盤拖曳替代方案。
- `aria-label`、`aria-live` toast。
- 顏色對比檢測。
- 表格標題 scope。
- 錯誤訊息與欄位關聯。
- 避免只用 `confirm()` 作為主要流程。

### 12.4 列印

- A4 landscape。
- 隱藏側欄、頂部列、工具列、覆蓋摘要、操作按鈕。
- 排班頁完整顯示。
- 不列印其他功能頁。
- 班次卡避免跨頁切割。

---

## 13. 驗證、錯誤處理與安全邊界

### 13.1 現行輸入驗證

- 必填欄位。
- 班次與班別 `end > start`。
- 休假 `end >= start`。
- 人員重疊。
- 核准休假。
- 手動排班工時超限警告。
- 班別已有參照時禁止刪除。
- 人員有參照時改為停用。
- 顏色只接受 `#RRGGBB`。
- 顯示使用 `esc()` 避免一般文字直接形成 HTML。

### 13.2 正式版必須補強

- JSON Schema／Zod validation。
- 外鍵完整性。
- ID 唯一性。
- 診間重疊。
- 人員可排日。
- 班別適用日。
- 診所開診日。
- 技能／專科需求。
- 休息間隔與連續工時。
- 同班同人重複。
- 權限檢查必須在後端執行。
- 所有寫入使用 optimistic locking 或 transaction。
- 防止 CSRF、XSS、SQL injection。
- 敏感 log 遮罩。
- Rate limit。
- 備份及還原演練。

### 13.3 個資原則

目前不含病患資料。正式版若只做人員排班，仍可能包含：

- 員工電話。
- 休假原因。
- 登入帳號。
- 操作紀錄。

因此需要：

- 最小權限。
- 休假備註可見範圍。
- 加密傳輸與靜態加密。
- 稽核。
- 保存期限與刪除政策。
- 不要把 production JSON 放在公開 GitHub。

---

## 14. 已知缺口與技術債

### P0：正式使用前必修

1. **沒有診間重疊檢查**：同一時間可把多位人員排到同一診間。
2. **手動排班不檢查可排星期**。
3. **手動排班不檢查診所開診日與班別適用日**。
4. **拖曳不檢查週工時上限**。
5. **最低人力只看 typeId，不看實際時間**。
6. **待命班仍算一般最低人力**。
7. **沒有特殊休診日／國定假日模型**。
8. **只有整日休假，沒有時段假**。
9. **匯入資料沒有完整 schema validation／migration**。
10. **公開 GitHub Pages 不適合放正式員工個資**。

### P1：可用性與管理

- 月視圖。
- 一次批次建立多日班次。
- 重複規則。
- 換班申請與簽核。
- 排班鎖定／公布。
- 排班版本比較。
- 人員偏好與不可排時段。
- 診間設備與技能需求。
- 連續工作天、最短休息、夜班規則。
- 主管備註與員工可見備註分離。
- 休假餘額。

### P2：正式產品化

- 多分院。
- LINE／Email 通知。
- SSO。
- HR／打卡／薪資整合。
- BI 報表。
- 官網門診表同步。
- 行動版 PWA／推播。
- 最佳化排班求解器。

### 14.1 現行 Release Note 文字差異

應用內 Release Notes 的 `2026/08/20 午診`描述可能只寫「陳妍容醫師」，但正確完整名單是：

- 王俊智
- 陳妍容
- 金宏亮
- 彭心沛

下一版應同步修正文案，避免只看到一位醫師。

---

## 15. 測試與驗收

### 15.1 現行已驗證

- 首頁與預載排班。
- 2026/08/03 當週顯示 94 筆班次卡。
- 七個主要頁面導覽。
- 人員新增、儲存與搜尋。
- 班次新增並回寫週表。
- 同一人員重疊阻擋。
- 核准休假後阻擋同日排班。
- LocalStorage 保存。
- CSV 匯出。
- JSON 完整備份。
- 診所、診間與營業日設定。
- 390×844 手機響應式。
- JavaScript Page Error／Console Error：0。

### 15.2 最低驗收案例

| ID | 情境 | 驗收標準 |
|---|---|---|
| SCH-001 | 新增一般班次 | 必填完整且開始早於結束時可儲存，重新整理後仍存在 |
| SCH-002 | 阻擋人員重疊 | 同人、同日、時間區間重疊時不得儲存 |
| SCH-003 | 核准休假阻擋 | 核准休假日期內不得新增或拖入該人員班次 |
| SCH-004 | 超時處理 | 手動排班超時須警告並允許使用者取消；自動補缺不得超時 |
| SCH-005 | 複製上週 | 日期加 7 天、跳過完全重複與核准休假、官網來源轉人工 |
| SCH-006 | 拖曳調班 | 可依視角改日期／人員／診間／班別，並重新檢查重疊與休假 |
| SCH-007 | 最低人力 | 每個開診日及適用班別依職務統計 actual/required |
| SCH-008 | 自動補缺 | 只選啟用、職務相符、可排日、無休假、無重疊、未超時者 |
| STF-001 | 刪除有人員參照 | 若已有班次或休假，只能停用，不可實體刪除 |
| LV-001 | 核准既有班次休假 | 須顯示影響班次數並要求二次確認，核准後列為衝突 |
| TYP-001 | 刪除班別 | 已有班次參照時不得刪除 |
| DAT-001 | JSON 備份 | 匯出完整 state；匯入前確認，成功後覆蓋並重繪 |
| DAT-002 | CSV 匯出 | 欄位與 UTF-8 BOM 正確，可由 Excel 開啟繁中 |
| RWD-001 | 手機版 | 390×844 下可開啟側欄、表單改單欄、Modal 由底部顯示 |
| PRT-001 | 列印 | 只列印排班頁，A4 橫向，隱藏操作按鈕與導覽 |

### 15.3 建議測試分層

#### Unit

- 日期週起始與跨月。
- 時間重疊邊界。
- 工時計算。
- 覆蓋計算。
- 自動補缺候選排序。
- migration。
- CSV quote。
- 權限規則。

#### Integration

- 建立人員後可排班。
- 建立班別後計入覆蓋。
- 核准休假後產生衝突。
- 刪除參照資料行為。
- 匯出再匯入等價。
- 多人同時修改的版本衝突。

#### E2E

- 完整排班工作流。
- 休假審核。
- 複製週與自動補缺。
- 手機操作。
- 列印／匯出。
- 角色權限。
- 後端斷線與重試。

### 15.4 Definition of Done

每一個 feature 至少要：

- 有明確規格與驗收案例。
- 通過 lint、typecheck、unit、integration、E2E。
- 不破壞既有 JSON。
- 有 migration。
- UI 支援桌機與手機。
- 錯誤可理解且不丟資料。
- 更新 Release Notes。
- 更新部署與回復說明。
- 安全與權限由後端驗證。
- 重要寫入有 audit log。

---

## 16. 在其他地方快速啟動

### 16.1 最簡單：直接用單檔 HTML

```bash
git clone https://github.com/JustinFang1019/eternal-dental-shift.git
cd eternal-dental-shift
```

直接開啟：

```text
Eternal_Shift_v1.1.1.html
```

或啟動本機靜態伺服器：

```bash
python -m http.server 8080
```

然後開啟：

```text
http://localhost:8080/Eternal_Shift_v1.1.1.html
```

### 16.2 部署到任意靜態主機

只要主機可提供靜態檔案即可，例如：

- Cloudflare Pages
- Netlify
- Render Static Site
- GitHub Pages
- Nginx／Apache
- 公司內部靜態網站

將完整檔案命名為 `index.html` 放在發布根目錄即可。

### 16.3 GitHub Pages 現況

Repository 已有 `gh-pages` 分支與完整 `index.html`。若網址仍為 404，Repository 擁有者需在：

```text
Settings → Pages
Source: Deploy from a branch
Branch: gh-pages
Folder: / (root)
```

設定完成後網址預期為：

```text
https://justinfang1019.github.io/eternal-dental-shift/
```

### 16.4 不建議直接雙擊 loader

完整 HTML 可用 `file://` 開啟；若使用分割 payload 的 loader，瀏覽器可能因 `fetch()` 本機檔案限制而失敗，必須透過 HTTP server。外部開發請以完整單檔為基準。

---

## 17. 建議的正式版架構

### 17.1 前端

可選：

- React + TypeScript + Vite
- Vue 3 + TypeScript + Vite
- SvelteKit
- 維持 Vanilla TypeScript

核心要求比框架更重要：

- domain rules 與 UI 分離。
- 所有 server state 透過 repository／query layer。
- 表單 schema validation。
- 日期使用 date-only library 或 Temporal polyfill。
- 排班表虛擬化，避免大量 DOM。
- 保留離線匯出／匯入。

### 17.2 後端

可選：

- FastAPI
- NestJS
- ASP.NET Core
- Spring Boot

基本要求：

- REST 或 GraphQL。
- PostgreSQL。
- Transaction。
- RBAC。
- Audit log。
- Optimistic concurrency。
- Background job。
- 備份。
- OpenAPI。

### 17.3 建議模組

```text
apps/
  web/
  api/
packages/
  domain/
    schedule-rules
    coverage
    conflicts
    migrations
  ui/
  shared-types/
infra/
  migrations/
  docker/
  deployment/
tests/
  unit/
  integration/
  e2e/
```

### 17.4 保留單機模式

正式版仍建議提供：

- JSON 匯出。
- 只讀離線班表。
- 網路中斷暫存。
- 災難回復用單檔報表。

但不要讓 LocalStorage 成為正式多人資料的 source of truth。

---

## 18. 建議資料庫模型

### 18.1 主要資料表

#### clinics

- `id uuid pk`
- `name`
- `english_name`
- `phone`
- `address`
- `timezone`
- `active`
- timestamps

#### clinic_rooms

- `id uuid pk`
- `clinic_id fk`
- `name`
- `room_type`
- `capacity`
- `active`
- unique `(clinic_id, name)`

#### users

- `id uuid pk`
- `email`
- `password_hash` 或外部 SSO ID
- `status`
- `last_login_at`
- timestamps

#### memberships

- `user_id`
- `clinic_id`
- `role`: owner/admin/scheduler/manager/staff/viewer
- unique `(user_id, clinic_id)`

#### staff

- `id uuid pk`
- `clinic_id`
- `user_id nullable`
- `display_name`
- `job_role_id`
- `employment_title`
- `weekly_max_minutes`
- `phone`
- `color`
- `active`
- `is_demo`
- timestamps
- row version

#### staff_skills

- `staff_id`
- `skill_id`

#### staff_availability

- `id`
- `staff_id`
- `weekday`
- `start_time nullable`
- `end_time nullable`
- `available`
- effective date range

#### shift_types

- `id`
- `clinic_id`
- `name`
- `start_time`
- `end_time`
- `color`
- `active`

#### shift_type_weekdays

- `shift_type_id`
- `weekday`

#### shift_type_requirements

- `shift_type_id`
- `job_role_id`
- `required_count`

#### shifts

- `id`
- `clinic_id`
- `schedule_version_id`
- `staff_id`
- `shift_type_id`
- `room_id`
- `work_date date`
- `start_time time`
- `end_time time`
- `on_call`
- `source`
- `note_private`
- `note_public`
- `status`: draft/published/cancelled
- timestamps
- row version

#### leaves

- `id`
- `clinic_id`
- `staff_id`
- `start_date`
- `end_date`
- `start_time nullable`
- `end_time nullable`
- `kind`
- `status`
- `reason_private`
- `reviewed_by`
- `reviewed_at`
- timestamps

#### clinic_closures

- `clinic_id`
- `date`
- `closed`
- `reason`
- 可選班別層級覆寫

#### schedule_versions

- `id`
- `clinic_id`
- `week_start`
- `version`
- `status`: draft/published/archived
- `published_by`
- `published_at`

#### audit_logs

- `id`
- `clinic_id`
- `actor_user_id`
- `entity_type`
- `entity_id`
- `action`
- `before_json`
- `after_json`
- `request_id`
- `created_at`

### 18.2 關鍵索引與限制

- `shifts (clinic_id, work_date)`
- `shifts (staff_id, work_date, start_time, end_time)`
- `shifts (room_id, work_date, start_time, end_time)`
- `leaves (staff_id, start_date, end_date)`
- `schedule_versions (clinic_id, week_start, version)`
- PostgreSQL exclusion constraint 或 transaction 內檢查人員及診間重疊。
- 所有寫入帶 `version`，避免後寫覆蓋先寫。

---

## 19. 建議 API

Base：

```text
/api/v1
```

### 19.1 Bootstrap

```http
GET /clinics/:clinicId/bootstrap
```

回傳診所、人員、房間、班別、指定日期範圍排班、休假與使用者權限。

### 19.2 Staff

```http
GET    /clinics/:clinicId/staff
POST   /clinics/:clinicId/staff
GET    /clinics/:clinicId/staff/:staffId
PATCH  /clinics/:clinicId/staff/:staffId
DELETE /clinics/:clinicId/staff/:staffId
```

有參照時 DELETE 回應衝突，前端改用停用。

### 19.3 Shifts

```http
GET    /clinics/:clinicId/shifts?from=YYYY-MM-DD&to=YYYY-MM-DD
POST   /clinics/:clinicId/shifts
PATCH  /clinics/:clinicId/shifts/:shiftId
DELETE /clinics/:clinicId/shifts/:shiftId
POST   /clinics/:clinicId/shifts/copy-week
POST   /clinics/:clinicId/shifts/auto-fill
POST   /clinics/:clinicId/shifts/validate
```

所有寫入由後端重新驗證重疊、休假、診間與權限。

### 19.4 Leaves

```http
GET    /clinics/:clinicId/leaves
POST   /clinics/:clinicId/leaves
PATCH  /clinics/:clinicId/leaves/:leaveId
POST   /clinics/:clinicId/leaves/:leaveId/approve
POST   /clinics/:clinicId/leaves/:leaveId/reject
DELETE /clinics/:clinicId/leaves/:leaveId
```

### 19.5 Settings

```http
GET   /clinics/:clinicId/settings
PATCH /clinics/:clinicId
CRUD  /clinics/:clinicId/rooms
CRUD  /clinics/:clinicId/shift-types
CRUD  /clinics/:clinicId/job-roles
```

### 19.6 Reports

```http
GET /clinics/:clinicId/reports/hours
GET /clinics/:clinicId/reports/conflicts
GET /clinics/:clinicId/reports/coverage
GET /clinics/:clinicId/reports/export.csv
```

### 19.7 發布與稽核

```http
POST /clinics/:clinicId/schedules/:weekStart/publish
GET  /clinics/:clinicId/schedules/:weekStart/versions
POST /clinics/:clinicId/schedules/:weekStart/restore/:versionId
GET  /clinics/:clinicId/audit-logs
```

### 19.8 錯誤格式

```json
{
  "error": {
    "code": "SHIFT_OVERLAP",
    "message": "此人員已有重疊班次",
    "details": {
      "conflictingShiftIds": ["..."]
    },
    "requestId": "..."
  }
}
```

前端不可只依 HTTP 文字判斷，要依 `code` 顯示對應訊息。

---

## 20. 權限矩陣建議

| 能力 | Owner | Admin | Scheduler | Manager | Staff | Viewer |
|---|---:|---:|---:|---:|---:|---:|
| 查看全部班表 | ✓ | ✓ | ✓ | ✓ | 視政策 | ✓ |
| 編輯班次 | ✓ | ✓ | ✓ | 視政策 | — | — |
| 發布班表 | ✓ | ✓ | ✓ | ✓ | — | — |
| 管理人員 | ✓ | ✓ | 視政策 | — | — | — |
| 提出休假 | ✓ | ✓ | ✓ | ✓ | 本人 | — |
| 核准休假 | ✓ | ✓ | 視政策 | ✓ | — | — |
| 查看休假原因 | ✓ | ✓ | 視政策 | 視政策 | 本人 | — |
| 系統設定 | ✓ | ✓ | — | — | — | — |
| 查看 audit log | ✓ | ✓ | — | 視政策 | — | — |
| 匯出完整資料 | ✓ | ✓ | 視政策 | 視政策 | — | — |

所有權限必須由後端執行；隱藏按鈕不等於授權。

---

## 21. 開發優先順序

### Milestone 0：基準保護

- 將 v1.1.1 完整 HTML 加入 fixture。
- 建立 E2E 覆蓋現行核心流程。
- 建立 JSON schema。
- 建立 migration framework。
- 將規則抽成純函式。
- 修正應用內 2026/08/20 Release Note 文案。

### Milestone 1：單機版 v1.2

- 診間重疊。
- 可排日／開診日／班別適用日檢查。
- 特殊休診日。
- 半日／時段休假。
- 月視圖。
- 批次建立與刪除。
- 自動備份與版本回復。
- 更完整匯入驗證。
- 單元與 E2E。

### Milestone 2：多人 MVP

- 後端 API、PostgreSQL。
- 登入與 RBAC。
- 多人同步與 optimistic locking。
- 稽核。
- 排班草稿／發布。
- 休假簽核。
- 備份。
- HTTPS 與 private deployment。

### Milestone 3：營運提升

- 換班流程。
- LINE／Email。
- 員工個人入口。
- 技能與診間設備需求。
- 公平輪值與最佳化排班。
- 多分院。
- 官網／公告輸出。

---

## 22. Coding Standards

### 22.1 TypeScript

- `strict: true`
- 不使用未定義的 `any`。
- domain model、DTO、view model 分離。
- 金額以整數最小單位；工時以整數分鐘。
- 日期使用 `YYYY-MM-DD`／資料庫 `DATE`。
- 時間使用 `HH:mm`／資料庫 `TIME`。
- 所有 enum 在 API 與 DB 有明確版本。

### 22.2 Domain Rules

- 規則函式不得讀 DOM。
- 所有規則輸入明確、輸出 deterministic。
- UI 只負責顯示規則結果。
- API 與前端共享測試案例，不共享不可信執行結果。
- 後端永遠重新驗證。

### 22.3 Git

- 分支：`feature/...`、`fix/...`。
- Conventional Commits。
- 每個 PR 有需求、畫面、資料 migration、測試與回復計畫。
- 不將 production JSON、員工個資、token 或 `.env` commit。
- Release 使用 semantic versioning。

### 22.4 版本

- Patch：修錯、文字、無 schema 變更。
- Minor：向後相容功能；如有 state 新欄位必須 migration。
- Major：不相容 API／schema／流程。

---

## 23. Release Notes

### v1.1.1 — 2026-07-26

- 校正 2026/08/20 午診醫師名單為王俊智、陳妍容、金宏亮、彭心沛。
- 完成 JavaScript 語法檢查。
- 完成 Chromium 桌機與手機響應式檢查。
- 驗證排班視角、人員、班次、休假、報表、CSV、JSON 與 LocalStorage。
- 補強官網資料與示範資料區隔。

### v1.1.0 — 2026-07-26

- 套用依貝美學名稱、Logo 與品牌視覺。
- 預載公開診所資訊、10 位醫師與主要專長。
- 匯入 2026 年 8 月公開門診表。
- 新增班別、人員、診間三種週排班視角。
- 完成班次、人員、休假、班別 CRUD。
- 完成人力缺口、自動補缺、複製上週、拖曳、報表與資料交換。

---

## 24. 接手開發檢查清單

開始前：

- [ ] 已確認這是人員排班，不是病患預約。
- [ ] 已取得 `Eternal_Shift_v1.1.1.html` 完整檔。
- [ ] 已核對 SHA-256：`5c6d1bf7997a771298353506a570fb89333518cbba81d2b113f6d5b7054765f7`。
- [ ] 已保存一份可操作的 v1.1.1 baseline。
- [ ] 已匯出並保存現有 JSON。
- [ ] 已建立 E2E baseline。

修改資料模型前：

- [ ] 已設計 schema version。
- [ ] 已提供 migration。
- [ ] 已測試舊 JSON 匯入。
- [ ] 已處理外鍵與重複 ID。
- [ ] 已提供 rollback。

正式上線前：

- [ ] 已移除／隔離範例人員。
- [ ] 已由診所確認真實班別、工時、可排日、診間及最低人力。
- [ ] 已建立私人部署，不把員工資料放公開 GitHub Pages。
- [ ] 已完成登入、RBAC、audit、backup。
- [ ] 已完成診間重疊與特殊休診日。
- [ ] 已完成負載、資安與還原測試。
- [ ] 已讓診所管理者完成 UAT 並簽核。

---

## 25. 最後結論

v1.1.1 已經是一個可操作的完整單機前端排班原型，核心 CRUD、衝突、休假、人力缺口、自動補缺、報表、備份與列印皆存在。接續開發的重點不是重畫畫面，而是：

1. 保護既有資料與行為。
2. 補足診間、可排日、特殊休診與完整工時規則。
3. 將 domain rules 從 DOM 抽離。
4. 建立 schema migration 與自動測試。
5. 加入後端、權限、多人同步、稽核與備份。
6. 保持「官網／範例／人工」資料來源可追溯。
7. 正式營運時使用私人部署，避免員工資料暴露於公開靜態網站。
