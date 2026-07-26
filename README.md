# Eternal Shift｜依貝美學牙醫診所排班系統

依貝美學牙醫診所客製化的牙醫排班系統。提供可離線使用的完整單檔 HTML，以及可由 GitHub Pages 直接發布的 `gh-pages` 分支；不需安裝 npm、後端或資料庫。

## 立即使用

- [下載正式版單檔 HTML v1.1.1](https://github.com/JustinFang1019/eternal-dental-shift/releases/download/v1.1.1/Eternal_Shift_v1.1.1.html)
- [查看 v1.1.1 Release](https://github.com/JustinFang1019/eternal-dental-shift/releases/tag/v1.1.1)
- [Repository 內的完整單檔 HTML](Eternal_Shift_v1.1.1.html)
- GitHub Pages：`https://justinfang1019.github.io/eternal-dental-shift/`

下載 HTML 後直接用 Chrome、Edge、Safari 或 Firefox 開啟即可。所有排班資料會儲存在目前瀏覽器的 LocalStorage。

## 完整功能

- 營運總覽與週排班表
- 依門診班別、人員、診間切換
- 班次新增、編輯、刪除與桌面拖曳調班
- 人員新增、編輯、停用、每週工時上限與可排星期
- 休假申請、核准、駁回與既有班次影響提示
- 同人重疊、核准休假、每週超時與最低人力檢查
- 自動補足人力缺口與複製上週
- 工時、警示與班別覆蓋報表
- CSV 匯出、JSON 完整備份／還原與列印
- LocalStorage 自動保存
- 桌機與手機響應式介面

## 使用方式

1. 下載 Release 附件 `Eternal_Shift_v1.1.1.html`，或開啟已啟用的 GitHub Pages 網址。
2. 第一次開啟會載入展示資料。
3. 所有修改會自動儲存在目前瀏覽器。
4. 請定期從系統匯出 JSON 備份；更換電腦或瀏覽器時再匯入。

## GitHub Pages 發布

`gh-pages` 分支已放置可直接發布的完整單檔版本：

- `index.html`
- `404.html`
- `Eternal_Shift_v1.1.1.html`
- `.nojekyll`

首次啟用請進入 **Settings → Pages**，設定：

1. **Source**：`Deploy from a branch`
2. **Branch**：`gh-pages`
3. **Folder**：`/ (root)`
4. 按下 **Save**

之後 GitHub Pages 會直接從 `gh-pages` 分支發布，不依賴自訂 GitHub Actions workflow。

## 資料邊界

這是完整的單機前端版。單一 HTML 前端無法提供多人即時同步、帳號權限、伺服器備份、操作稽核與 LINE／Email 通知；上述能力需要後端服務。官網公開醫師資料與系統展示人力已在介面內分開標示。

## 版本與驗證

目前版本：**v1.1.1**。

- [Release Notes](RELEASE_NOTES.md)
- [Validation Report](VALIDATION_REPORT.md)
- 原始 HTML：99,990 bytes
- SHA-256：`5c6d1bf7997a771298353506a570fb89333518cbba81d2b113f6d5b7054765f7`
