# Eternal Shift｜依貝美學牙醫診所排班系統

依貝美學牙醫診所客製化的牙醫排班系統。開啟 GitHub Pages 後即可直接使用，不需安裝 npm、後端或資料庫。

## 線上入口

- GitHub Pages：`https://justinfang1019.github.io/eternal-dental-shift/`
- Repository：`https://github.com/JustinFang1019/eternal-dental-shift`

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

1. 開啟 GitHub Pages 網址。
2. 第一次開啟會載入展示資料。
3. 所有修改會自動儲存在目前瀏覽器。
4. 請定期從系統匯出 JSON 備份；更換電腦或瀏覽器時再匯入。

## GitHub Pages 發布

Repository 內含 `.github/workflows/pages.yml`。在 Repository 的 **Settings → Pages** 將 **Source** 設為 **GitHub Actions** 後，推送至 `main` 即會自動發布。

## 資料邊界

這是完整的單機前端版。單一 HTML 前端無法提供多人即時同步、帳號權限、伺服器備份、操作稽核與 LINE／Email 通知；上述能力需要後端服務。官網公開醫師資料與系統展示人力已在介面內分開標示。

## 版本

目前版本：**v1.1.1**。詳細內容請見 [RELEASE_NOTES.md](RELEASE_NOTES.md)，驗證結果請見 [VALIDATION_REPORT.md](VALIDATION_REPORT.md)。
