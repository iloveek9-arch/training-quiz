教育訓練測驗網站

內部教育訓練線上測驗（每頁 1 題、完卷顯示成績與是否及格、可選擇上傳成績至 Google 試算表）。
專案位址：https://github.com/iloveek9-arch/training-quiz
GitHub Pages（網站首頁）：https://iloveek9-arch.github.io/training-quiz/


📁 專案內容（重點檔案）
training-quiz/
├─ index.html                         # 首頁（列出所有測驗；依日期由近到遠排列）
├─ 11412滅火器介紹及使用操作.html      # 測驗檔（滅火器介紹與使用操作，114 年 12 月）
└─ 11411防範電器火災.html             # 測驗檔（防範電器火災，114 年 11 月）

首頁（index.html）

進站後可選擇進入哪一份測驗。
顯示順序：11412 在前、11411 在後（依日期新→舊）。
連結使用簡化後的中文檔名：

11412滅火器介紹及使用操作.html
11411防範電器火災.html



每份測驗檔

進入測驗前必須輸入「單位」與「姓名」。
每頁只出 1 題，最後一題按「提交交卷」。
完卷顯示「分數（滿分 100）」與「是否及格（60 分）」。
作答過程與分數儲存在瀏覽器 localStorage。
可選擇上傳成績至 Google 試算表（見下方串接說明）。


🌐 GitHub Pages 發佈（啟用網站）

開啟 Repo → 進入 Settings。
左側選單點 Pages。
Source 選擇 Deploy from a branch。
Branch 選 main；Folder 選 /（root）。
按 Save，等待約 30 秒。
網站會出現在：
https://iloveek9-arch.github.io/training-quiz/


其中 index.html 會自動成為首頁。



測驗頁直接連結（佈署後）：
https://iloveek9-arch.github.io/training-quiz/11412滅火器介紹及使用操作.html
https://iloveek9-arch.github.io/training-quiz/11411防範電器火災.html


如果中文檔名在特定環境出現 404，請改用英文檔名並同步更新 index.html 內的連結。


📤 成績上傳（Google 試算表串接）
1) 建立/確認 Google Apps Script Web App

你已提供 Web App URL 並嵌入 11412 測驗檔；若 11411 也要啟用，請同樣嵌入端點即可。


在目標試算表的 擴充功能 → Apps Script 建立以下程式（簡化版）：

JavaScriptconst SHEET_NAME = '成績';function ensureHeader_(sh) {  const header = ['測驗日期時間', '測驗項目', '單位', '姓名', '成績', '是否及格'];  const range = sh.getRange(1, 1, 1, header.length);  const values = range.getValues();  const hasHeader = values[0].some(v => v && String(v).trim() !== '');  if (!hasHeader) range.setValues([header]);}function appendRow_(payload) {  const ss = SpreadsheetApp.getActive();  let sh = ss.getSheetByName(SHEET_NAME);  if (!sh) sh = ss.insertSheet(SHEET_NAME);  ensureHeader_(sh);  const row = [    payload.timestamp || '',    payload.item || '',    payload.unit || '',    payload.name || '',    payload.score || '',    payload.pass || ''  ];  sh.appendRow(row);}function doPost(e) {  try {    const p = e.parameter || {};    appendRow_(p);    return ContentService      .createTextOutput(JSON.stringify({ ok: true }))      .setMimeType(ContentService.MimeType.JSON);  } catch (err) {    return ContentService      .createTextOutput(JSON.stringify({ ok: false, error: String(err) }))      .setMimeType(ContentService.MimeType.JSON);  }}顯示更多行

部署：點 部署 → 新部署 → 類型選「網路應用程式」

執行身分：本人
存取權：任何擁有連結的人
取得 URL（形如 https://script.google.com/macros/s/…/exec）



2) 在測驗檔中設定 Web App URL

打開測驗檔（HTML），找到：
JavaScriptconst SHEETS_ENDPOINT = 'https://script.google.com/macros/s/xxxx/exec';顯示更多行

替換為你實際的網址（11412 已替換；11411 視需要同樣替換）。

3) 上傳欄位（前端送出格式）

timestamp：台北時區、24 小時制時間字串（若不支援時區，退回 YYYY-MM-DD HH:mm:ss）
item：自動由檔名轉換成測驗項目（去除括號/副檔名/後綴）
unit：單位（使用者填寫）
name：姓名（使用者填寫）
score：整份測驗得分（0–100）
pass：是否及格（及格/不及格；及格線 60 分）


🧪 本機測試
方法一：直接雙擊 index.html 在瀏覽器開啟（不涉及外部資源可正常使用）。
方法二：啟動簡易靜態伺服器（可避開某些瀏覽器的本機限制）：
Shell# Python 3python -m http.server 8080# 然後在瀏覽器開 http://localhost:8080/顯示更多行

🔐 隱私與安全

作答過程與分數僅存於瀏覽器 localStorage；若啟用上傳，僅於交卷時傳送至你設定的 Google Web App。
Web App 請設定為「任何擁有連結的人」可存取，並放置於公司控管的 Google 帳號下；必要時可改在後端加上來源檢查（白名單）或 Token 驗證。


🧭 常見問題（FAQ）


GitHub Pages 開啟後頁面空白或資源載入不到？
請確認 Pages 設定為 main / root，且變更後等待 30–60 秒生效；若使用公司網路，檢查是否有快取或防火牆阻擋。


中文檔名連結 404？
少數情況下 URI 編碼可能導致路徑不相符。建議將檔名改為英文，再同步更新 index.html 連結。


成績沒寫入試算表？

檢查 SHEETS_ENDPOINT 是否為 /exec（不是 /dev）
Web App 權限是否「任何擁有連結的人」
Apps Script → 執行記錄有無錯誤訊息
試算表是否有「成績」工作表（或讓程式自動建立）




🗓️ 維運建議

新增新月份測驗：複製任一現有測驗檔 → 修改標題/題庫/檔名（建議格式 114MM_英文關鍵字.html 或中文簡名） → 在 index.html 新增卡片並依日期排序。
統一版型：若你想在首頁自動掃描資料夾並依年月排序列出所有測驗，我可以幫你改成自動化首頁（免手動維護清單）。


📝 版權與授權

題目文字與圖片屬於公司內部教育訓練用途。
此網站程式碼可作為內部專案範本使用；若需對外公開或二次授權，請先徵得相關單位同意。
