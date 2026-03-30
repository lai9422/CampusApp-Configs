
# 🏫 課務通 APP · 萬用校園配置開源庫

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com) 
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

本專案旨在透過社群的力量，收集全台灣各大大專院校的校園資訊系統配置檔 (`config.json`)。

## 📲 下載 課務通 APP

| 平台 | 連結 |
|------|------|
| **iOS**（App Store） | [https://apps.apple.com/tw/app/%E8%AA%B2%E5%8B%99%E9%80%9A/id6761055559](https://apps.apple.com/tw/app/%E8%AA%B2%E5%8B%99%E9%80%9A/id6761055559) |
| **Android**（APK） | [https://uni-campus-app.pages.dev/apk/v1.0.1.apk](https://uni-campus-app.pages.dev/apk/v1.0.1.apk) |

你不必懂原生 iOS / Android 開發，**只要會基礎的 JavaScript (DOM 操作或 API Fetch)**，就能透過配置 JSON，讓 **課務通 APP** 完美支援你的學校，提供包含：**自動登入、課表匯入、校園公告、待領信件、常用捷徑** 等強大功能！

---

## 🚀 如何為你的學校新增支援？ (貢獻指南)

我們非常歡迎你為自己的學校提交配置檔！請遵循以下步驟：

1. **下載並打開「課務通 APP」**：進入「設置」>「配置精靈」。
2. **填寫與測試**：依序完成基礎、登入、課表、公告、信件的設定，並務必在 **課務通 APP** 內的 **「沙盒環境 」** 測試通過。
3. **匯出配置碼**：測試無誤後，在最後一步點擊「複製分享配置碼 (JSON)」。
4. **Fork 本專案**：點擊右上角的 Fork 將本儲存庫複製到你的 GitHub。
5. **建立資料夾**：在儲存庫**根目錄**下，以**你學校的網域**為資料夾名稱 (例如：`ncue.edu.tw/`)，與 `config.json` 內 `schoolId` 一致。
6. **上傳檔案**：將剛剛複製的 JSON 存成 `config.json`。
7. **提交 Pull Request (PR)**：發送 PR 給我們，審核通過後，你的學校就會出現在 **課務通 APP** 的支援清單中！🎉

---

## ⚙️ 核心運作邏輯與開發必讀

**課務通 APP** 的底層核心依賴 **WebView 輪詢注入** 與 **API 請求**，請在撰寫腳本時務必遵守以下規範：

### 1. 輪詢頻率 (Polling) 與防連點鎖 (Lock)
課務通 APP 在背景執行任務（如自動登入、爬取課表）時，會 **每 500 毫秒 (500ms)** 將你的腳本注入 WebView 執行一次。
> ⚠️ **極度重要**：請務必在腳本最外層使用鎖定變數（如 `window._isProcessing = true` 或 `sessionStorage`），防止按鈕被瘋狂連點或資料被重複送出！

### 2. 與課務通 APP 通訊 (`postMessage`)
所有的腳本（API 解析 或 WebView 爬蟲），最終都必須透過 `postMessage` 將特定格式的資料回傳給課務通 APP。
**基本語法：**
```javascript
window.mobileApp.postMessage({ 
  detail: { 
    type: '對應的成功訊號', 
    payload: 陣列資料 // 依各模組規定
  } 
});
```
支援的訊號 Type：
- `AUTH_SUCCESS`：登入成功
- `DATA_SUCCESS`：課表解析成功
- `NEWS_SUCCESS`：公告解析成功
- `MAIL_SUCCESS`：信件解析成功
- `SESSION_EXPIRED`：憑證過期 (請課務通 APP 重新導向登入頁)
- `ERROR`：發生錯誤

---

## 📖 `config.json` 完整規格與撰寫範例

一份完整的 `config.json` 包含以下頂層結構：
```json
{
  "schoolId": "學校網域 (如 ncue.edu.tw)",
  "schoolName": "彰化師範大學",
  "logoPath": "https://.../logo.png",
  "calendar": { "type": "pdf|image|web", "url": "https://..." },
  "auth": { ... },
  "schedule": { ... },
  "news": { ... },
  "mail": { ... },
  "links": [ ... ]
}
```

### 本庫實際配置檔（範例來源）

本儲存庫根目錄已收錄三校之 `config.json`，以下模組範例皆從中摘錄；完整內容請直接打開對應檔案。

| 學校 | `schoolId` | 檔案路徑 |
|------|------------|----------|
| 國立高雄科技大學 | `nkust.edu.tw` | [`nkust.edu.tw/config.json`](nkust.edu.tw/config.json) |
| 彰化師範大學 | `ncue.edu.tw` | [`ncue.edu.tw/config.json`](ncue.edu.tw/config.json) |
| 國立臺中科技大學 | `nutc.edu.tw` | [`nutc.edu.tw/config.json`](nutc.edu.tw/config.json) |

- **高科大**：課表為 **Kendo + `#detail-table` 表格**；公告／信件為 **Webview**；信件腳本含 **多頁合併**。
- **彰師大**：**NAM SSO**；課表為 **學年學期下拉 + `table.table-timetable`**；公告為 **API 模式**（`doc_board_ajax.php`）；信件為 **Webview** 表格列。
- **臺中科大**：登入後 **導向 AIS**（`verifyScript`）；課表讀取 **`g_ClsTime`** 全域變數；無需切換學期時 `switchScript` 可為空函式。

以下依模組說明欄位。**純設定網址、不含注入腳本者**（如 `calendar`、`links`）僅列**一組**格式範例即可；**各校網頁框架差異大者**（`verifyScript`、`switchScript`、`parseScript`、公告／信件之 `webviewScript`、`apiRequestScript` 等）則分列**多則腳本範例**，對應檔案見上表。

**注意**：`verifyScript`、`switchScript`、`parseScript`、`webviewScript` 等在 `config.json` 內須為**單行字串**（含 `\n` 轉義）。本文件中的 `javascript` 區塊為**可讀格式**，撰寫完成後請以課務通 APP「配置精靈」匯出之 JSON 為準，或自行壓成單行。

### 📆 校曆 (`calendar`)

首頁顯示之行事曆來源。

* **`type`**：`pdf`、`image` 或 `web`。
* **`url`**：PDF／圖片網址，或行事曆網頁連結。

**欄位格式範例（各校僅替換 `url`）：**

```json
"calendar": {
  "type": "pdf",
  "url": "https://example.edu.tw/path/to/calendar.pdf"
}
```

### 🔐 1. 登入模組 (`auth`)

負責處理入口系統的自動登入與驗證。

* **`entryUrl`**：登入頁或導向登入之入口完整網址。
* **`loginIndicators` / `successIndicators`**：陣列。網址特徵 `{ "type": "contains|startsWith|exact", "value": "關鍵字" }`，用於判斷是否在登入頁或已登入。
* **`usernameSelector` / `passwordSelector` / `submitSelector`**：帳、密、送出鈕的 CSS 選擇器。
* **`autoFill` / `autoSubmit`**：是否自動填帳密、是否自動送出表單。
* **`verifyScript`**：(選填) 補強用；例如偵測已出現「登出」且密碼欄已消失時回報 `AUTH_SUCCESS`。

**JSON 欄位結構範例（一組即可；`entryUrl`、選擇器與 indicators 請依貴校修改）：**

```json
"auth": {
  "entryUrl": "https://...",
  "successIndicators": [ { "type": "contains", "value": "..." } ],
  "loginIndicators": [ { "type": "contains", "value": "..." } ],
  "usernameSelector": "#user",
  "passwordSelector": "#pass",
  "submitSelector": "#loginBtn",
  "autoFill": true,
  "autoSubmit": false,
  "verifyScript": "<見下方腳本範例，或留空字串 \"\">"
}
```

#### `verifyScript` 範例（各校頁面不同，擇一或自行改寫）

**範例 A — DOM 判斷已登入（摘自 `nkust.edu.tw`，偵測登出連結且密碼欄已消失）：**

```javascript
(function () {
  var logoutBtn = document.querySelector('a[href="/student/Account/Logout"]');
  var pwdInput = document.querySelector('input[name="Password"]');
  if (logoutBtn && !pwdInput && window.mobileApp) {
    window.mobileApp.postMessage({ detail: { type: 'AUTH_SUCCESS' }});
  }
})();
```

**範例 B — 可不寫腳本（摘自 `ncue.edu.tw` 思路）：** 若 `successIndicators` / `loginIndicators` 已能可靠區分登入頁與已登入，則設 `"verifyScript": ""` 即可。

**範例 C — 中介頁再導向目標系統後回報（摘自 `nutc.edu.tw`，點「學生管理系統」連結後再在 AIS 首頁 `AUTH_SUCCESS`）：**

```javascript
(function () {
  try {
    if (location.href.indexOf('MyArea.aspx') !== -1) {
      if (window._nutcRedirecting) return;
      var links = document.querySelectorAll('a');
      for (var i = 0; i < links.length; i++) {
        if (links[i].textContent.trim() === '學生管理系統' && links[i].href.indexOf('Ticket=') !== -1) {
          window._nutcRedirecting = true;
          location.href = links[i].href;
          return;
        }
      }
    }
    if (location.href.indexOf('ais.nutc.edu.tw/student/home.aspx') !== -1) {
      if (window.mobileApp) window.mobileApp.postMessage({ detail: { type: 'AUTH_SUCCESS' }});
    }
  } catch (e) {}
})();
```

### 📅 2. 課表模組 (`schedule`)

處理學生課表爬取，支援自動切換學期。

* **`syncMode`**：例如 `auto` 與課務通 APP 同步行為（依版本定義）。
* **`entryUrl`**：課表功能入口網址。
* **`targetUrlIndicators`**：符合後才執行 `switchScript` / `parseScript`。
* **`sessionExpiredIndicators`**：符合時視為登入逾時，可導回登入。
* **`timeSlots`**：`[{ "s": "節次代號", "t": "HH:mm~HH:mm" }]`，須與學校節次一致。
* **`switchScript`**：(選填) 切換學年學期；可用占位符 `{{TARGET_YEAR}}`、`{{TARGET_SEM}}`。
* **`parseScript`**：解析 DOM，成功後以 `DATA_SUCCESS` 回傳。
* **`manualParseScript`**：手動解析用（若課務通 APP 支援）。

> ⚠️ **Payload 強制規格**：`payload` 陣列長度**必須等於** `timeSlots` 數量。每筆為 `{ "slot": "代號", "days": [ 週一～週日共 7 格 ] }`。空堂為 `{}`，有課為 `{ "name", "room", "teacher" }`。

**JSON 欄位結構範例（一組即可；`entryUrl`、indicators、`timeSlots` 請依貴校補齊）：**

```json
"schedule": {
  "syncMode": "auto",
  "entryUrl": "https://...",
  "targetUrlIndicators": [ { "type": "contains", "value": "..." } ],
  "sessionExpiredIndicators": [ { "type": "contains", "value": "..." } ],
  "timeSlots": [
    { "s": "1", "t": "08:10~09:00" },
    { "s": "2", "t": "09:10~10:00" }
  ],
  "switchScript": "<見下方範例；無需切換學期可為空函式>",
  "parseScript": "<見下方範例>",
  "manualParseScript": ""
}
```

（實際配置須補齊該校**全部**節次之 `timeSlots`，列數須與 `parseScript` 產出之 `payload` 一致。）

#### `switchScript` / `parseScript` 範例（不同學校系統差異大，下列三則摘自本庫）

**① 高科大（`nkust.edu.tw`）— Kendo 學期下拉 + jQuery、`#detail-table` 列解析**

**`switchScript`（可讀）：**

```javascript
(function(){
  if (typeof jQuery === 'undefined') return;
  var targetVal = '{{TARGET_YEAR}}-{{TARGET_SEM}}';

  if (window.nkustQueryLock && (Date.now() - window.nkustQueryLock < 3000)) return;

  try {
    var $ddl = jQuery('#SchoolYearSms').data('kendoDropDownList');
    if (!$ddl) return;

    var currentVal = $ddl.value();
    var table = document.querySelector('#detail-table');

    if (currentVal !== targetVal || !table) {
      window.nkustQueryLock = Date.now();

      if (currentVal !== targetVal) {
        $ddl.value(targetVal);
        $ddl.trigger('change');
      }

      setTimeout(function() {
        if (typeof window.query === 'function') {
          window.query();
        } else {
          var qBtn = document.querySelector('#query');
          if (qBtn) qBtn.click();
        }
      }, 800);
    }
  } catch(e) {}
})();
```

**`parseScript`（可讀）：**

```javascript
(function() {
  if (window.isProcessing) return;
  var table = document.querySelector('#detail-table');
  if (!table) return;
  var rows = table.querySelectorAll('tbody tr');
  if (rows.length === 0) return;

  var hasContent = Array.from(rows).some(function(r){ return r.innerText.trim().length > 10; });
  if (!hasContent) return;

  window.isProcessing = true;
  var result = [];

  Array.from(rows).forEach(function(tr) {
    var tds = tr.querySelectorAll('td');
    if (tds.length < 8) return;

    var slot = tds[0].innerText.trim().split('\n')[0].trim();
    var days = [];

    for (var d = 1; d <= 7; d++) {
      var cell = tds[d];
      var a = cell.querySelector('a');
      if (a) {
        var courseName = a.innerText.trim();
        var textNodes = Array.from(cell.childNodes).filter(function(n) {
            return n.nodeType === 3 && n.textContent.trim().length > 0;
        });
        var teacher = textNodes.length > 0 ? textNodes[0].textContent.trim() : '';
        var room = textNodes.length > 1 ? textNodes[1].textContent.trim() : '';

        days.push({
          name: courseName,
          teacher: teacher,
          room: room
        });
      } else {
        days.push({});
      }
    }
    result.push({ slot: slot, days: days });
  });

  if (result.length > 0) {
    if (window.mobileApp) window.mobileApp.postMessage({ detail: { type: 'DATA_SUCCESS', payload: result }});
  } else {
    window.isProcessing = false;
  }
})();
```

**② 臺中科大（`nutc.edu.tw`）— 頁面全域變數 `g_ClsTime`、無需學期切換**

AIS 週課表會注入 `g_ClsTime`（鍵為 `星期-節次`，值為 tab 分隔欄位）。**`switchScript` 可為空流程**：

```javascript
(function(){
  // 無需學期切換
})();
```

**`parseScript`（可讀）：**

```javascript
(function() {
  try {
    if (window._hasSentData) return;
    if (typeof g_ClsTime === 'undefined') return;

    var result = [];
    var slotMap = ['1','2','3','4','5','6','7','8','9','10','11','12','13','14'];
    for(var k=0; k<slotMap.length; k++) {
      result.push({ slot: slotMap[k], days: [{},{},{},{},{},{},{}] });
    }

    var hasData = false;
    for (var key in g_ClsTime) {
      var kParts = key.split('-');
      var dayIdx = parseInt(kParts[0]) - 1;
      var pIdx = parseInt(kParts[1]) - 1;
      if(dayIdx >= 0 && dayIdx < 7 && pIdx >= 0 && pIdx < slotMap.length) {
        var cData = g_ClsTime[key].split('\t');
        var cName = cData[1] ? cData[1].replace(/<[^>]+>/g, '').trim() : '';
        result[pIdx].days[dayIdx] = {
          name: cName,
          teacher: cData[2] ? cData[2].trim() : '',
          room: cData[3] ? cData[3].trim() : ''
        };
        hasData = true;
      }
    }

    if (hasData) {
      window._hasSentData = true;
      if(window.mobileApp) {
        window.mobileApp.postMessage({ detail: { type: 'DATA_SUCCESS', payload: result }});
      }
    }
  } catch(e) {}
})();
```

**③ 彰師大（`ncue.edu.tw`）— 原生 `<select>` 學年學期 + `table.table-timetable`**

**`switchScript`（可讀）：**

```javascript
(function() {
  if (window._isSwitching) return;
  var modals = document.querySelectorAll('.modal.show');
  if (modals.length > 0) {
    modals.forEach(function(m) { m.classList.remove('show'); m.style.display = 'none'; });
    document.querySelectorAll('.modal-backdrop').forEach(function(b) { b.remove(); });
    document.body.classList.remove('modal-open');
  }
  var yearSelect = document.getElementById('ddl_yms_year');
  var semSelect = document.getElementById('ddl_yms_smester');
  var btn = document.querySelector('button[name="PGcmd"][value="查詢"]');
  if(yearSelect && semSelect && btn) {
    var currentY = yearSelect.value;
    var currentS = semSelect.value;
    if(currentY !== '{{TARGET_YEAR}}' || currentS !== '{{TARGET_SEM}}') {
      window._isSwitching = true;
      yearSelect.value = '{{TARGET_YEAR}}';
      semSelect.value = '{{TARGET_SEM}}';
      yearSelect.dispatchEvent(new Event('change', { bubbles: true }));
      semSelect.dispatchEvent(new Event('change', { bubbles: true }));
      setTimeout(function() { btn.click(); }, 300);
    }
  }
})();
```

**`parseScript`（可讀）：**

```javascript
(function() {
  if (window._hasSentData) return;
  if (document.querySelector('.modal.show')) return;
  var yearSelect = document.getElementById('ddl_yms_year');
  var semSelect = document.getElementById('ddl_yms_smester');
  if (yearSelect && semSelect) {
    if (yearSelect.value !== '{{TARGET_YEAR}}' || semSelect.value !== '{{TARGET_SEM}}') return;
  }
  var titleSpan = document.querySelector('.card-header.bg-primary span.fw-bold');
  if (titleSpan) {
    var titleText = titleSpan.innerText;
    if (titleText.indexOf('{{TARGET_YEAR}}') === -1 || titleText.indexOf('第 {{TARGET_SEM}}') === -1) return;
  }
  var table = document.querySelector('table.table-timetable');
  if (!table) return;
  window._hasSentData = true;
  try {
    var result = [];
    var slotMap = ['1','2','3','4','14','5','6','7','8','9','10','11','12','13'];
    for (var k = 0; k < slotMap.length; k++) {
      result.push({ slot: slotMap[k], days: [{},{},{},{},{},{},{}] });
    }
    var rows = table.querySelectorAll('tbody tr');
    var dataIndex = 0;
    for (var i = 0; i < rows.length; i++) {
      var tr = rows[i];
      var cols = tr.querySelectorAll('td.course-cell');
      if (cols.length >= 7 && dataIndex < result.length) {
        var currentSlot = result[dataIndex];
        for (var d = 0; d < 7; d++) {
          var cell = cols[d];
          var div = cell.querySelector('.course-text');
          var text = div ? div.innerText.trim() : cell.innerText.trim();
          if (text.length > 0) {
            var parts = text.split('/');
            var courseObj = {};
            if (parts.length >= 3) {
              courseObj = {
                id: parts[0].trim(),
                name: parts[1] ? parts[1].trim().replace(/^[A-Z]/, '') : '',
                room: (parts[3] && parts[3].includes('教室')) ? parts[3].replace('教室','').trim() : (parts[3] || '').trim(),
                teacher: parts[4] ? parts[4].trim() : ''
              };
            } else {
              courseObj = { name: text, room: '', teacher: '' };
            }
            currentSlot.days[d] = courseObj;
          }
        }
        dataIndex++;
      }
    }
    if (window.mobileApp) {
      window.mobileApp.postMessage({ detail: { type: 'DATA_SUCCESS', payload: result }});
    } else {
      window.location.href = 'school-data://' + encodeURIComponent(JSON.stringify({ type: 'DATA_SUCCESS', payload: result }));
    }
  } catch (e) {
    if (window.mobileApp) {
      window.mobileApp.postMessage({ detail: { type: 'ERROR', message: '解析失敗: ' + e.message }});
    }
  }
})();
```

**課表 `payload` 格式示意（教學用，節次數請與貴校 `timeSlots` 一致）：**

```javascript
var result = [
  {
    slot: '1',
    days: [
      {},
      { name: '微積分', room: 'A101', teacher: '王大明' },
      {}, {}, {}, {}, {}
    ]
  }
];
window.mobileApp.postMessage({ detail: { type: 'DATA_SUCCESS', payload: result } });
```

### 📢 3. 公告模組 (`news`) 與信件模組 (`mail`)

兩者皆支援 **`mode`: `api`** 與 **`mode`: `webview`**。

**共通欄位（節錄）：**

* **`enabled`**：是否啟用。
* **`mode`**：`api` 或 `webview`。
* **API 模式**：`apiUrl`、`apiMethod`、`apiHeaders`、`apiBody`、`apiRequestScript`、`parseScript`（回傳陣列）。
* **Webview 模式**：`webviewUrl`、`targetUrlIndicators`、`webviewScript`（於頁面內 `postMessage`）。

#### 模式 A：API 請求（通用）

透過底層直接發送 HTTP 請求。

* **`apiUrl`, `apiHeaders`, `apiBody`**：可含 `{{KEY}}`，由 `apiRequestScript` 回傳之物件替換。
* **`apiRequestScript`**：回傳 `{ KEY: '值' }` 供替換用。
* **`parseScript`**：處理 `responseData` 後 `return` 陣列（公告欄位與 Webview 相同；信件為 `name, department, date, campus`）。

```javascript
var list = [];
(responseData.items || []).forEach(function(n) {
  list.push({
    title: n.title,
    date: n.date,
    department: n.dept,
    link: n.url
  });
});
return list;
```

**模式 A（實例）— 彰師大 API 公告（`ncue.edu.tw`）**  
於 `news` 設 `mode: "api"`，設定 `apiUrl`、`apiMethod`、`apiBody`（內含 `{{TODAY}}`、`{{P3D}}` 等占位），其餘空欄位與共通欄位說明一致；完整鍵值組合見倉庫內 `ncue.edu.tw/config.json`。

**`apiRequestScript`（可讀）：**

```javascript
function formatDate(date) {
  var y = date.getFullYear();
  var m = String(date.getMonth() + 1).padStart(2, '0');
  var d = String(date.getDate()).padStart(2, '0');
  return y + '/' + m + '/' + d;
}
var today = new Date();
var threeDaysAgo = new Date();
threeDaysAgo.setDate(today.getDate() - 3);

return {
  TODAY: formatDate(today),
  P3D: formatDate(threeDaysAgo)
};
```

**`parseScript`（可讀）：**

```javascript
var parsedList = [];
if (responseData && responseData.data) {
  responseData.data.forEach(function(item) {
    if (Array.isArray(item) && item.length > 7) {
      var cleanTitle = (item[0] || '').replace(/<[^>]+>/g, "").trim().replace(/&amp;/g, "&");
      parsedList.push({
        title: cleanTitle,
        department: item[1],
        date: item[3].trim(),
        link: 'https://aps.ncue.edu.tw/odedi/doc_page.php?id=' + item[7]
      });
    }
  });
  parsedList.sort(function(a, b) { return b.date.localeCompare(a.date); });
}
return parsedList;
```

#### 模式 B：Webview 爬蟲

`webviewUrl` 指向公告列表頁，於 **`webviewScript`** 內組出 `NEWS_SUCCESS` 與 `payload: { title, date, link, department }[]`（欄位格式見共通說明）。

##### 公告 Webview 範例 ① — 高科大（`nkust.edu.tw`，列表 `#pageptlist .d-item`）

**`webviewScript`（可讀）：**

```javascript
(function() {
  if (window._hasParsedNews) return;

  var items = document.querySelectorAll('#pageptlist .d-item');
  if (items.length === 0) return;

  window._hasParsedNews = true;
  var result = [];

  for (var i = 0; i < items.length; i++) {
    var a = items[i].querySelector('.mtitle a');
    var dateEl = items[i].querySelector('.mdate');
    if (a) {
      result.push({
        title: a.textContent.trim(),
        date: dateEl ? dateEl.textContent.trim() : '',
        link: a.href,
        department: '校園公告'
      });
    }
  }

  if (window.mobileApp) {
    window.mobileApp.postMessage({
      detail: {
        type: 'NEWS_SUCCESS',
        payload: result
      }
    });
  }
})();
```

##### 公告 Webview 範例 ② — 臺中科大（`nutc.edu.tw`，表格 `.listTB`、`data-th` 欄位）

**`webviewScript`（可讀）：**

```javascript
(function () {
  // 全域鎖：防止重複執行回傳
  if (window._hasParsedNews) return;

  var rows = document.querySelectorAll('.listTB tbody tr');
  if (rows.length === 0) return; // 等待表格載入完成

  window._hasParsedNews = true;
  var result = [];

  for (var i = 0; i < rows.length; i++) {
    var tr = rows[i];
    var titleA = tr.querySelector('td[data-th="標題"] a');
    var dateTd = tr.querySelector('td[data-th="日期"]');
    var deptTd = tr.querySelector('td[data-th="發佈單位"]');

    if (titleA) {
      var cleanDate = dateTd ? dateTd.textContent.replace(/[\s\[\]]/g, '') : '';

      result.push({
        title: titleA.textContent.trim(),
        date: cleanDate,
        link: titleA.href,
        department: deptTd ? deptTd.textContent.trim() : '校方公告'
      });
    }
  }

  if (window.mobileApp) {
    window.mobileApp.postMessage({
      detail: {
        type: 'NEWS_SUCCESS',
        payload: result
      }
    });
  }
})();
```

#### 信件（`mail`）

`webviewUrl` 指向待掛號／信件列表頁；**`webviewScript`** 內回傳 `MAIL_SUCCESS`，`payload` 為 `{ name, department, date, campus }[]`。

##### 信件 Webview 範例 ① — 高科大（`nkust.edu.tw`，ASP.NET GridView、查詢鈕、多頁合併）

**`webviewScript`（可讀）：**

```javascript
(function() {
  if (window._isHandlingMail) return;
  window._isHandlingMail = true;

  var grid = document.getElementById('ctl00_ContentPlaceHolder1_GridView3');
  var btn = document.getElementById('ctl00_ContentPlaceHolder1_qry_btn1');

  if (!grid) {
    if (!sessionStorage.getItem('mail_init_query')) {
      sessionStorage.setItem('mail_init_query', 'true');
      sessionStorage.removeItem('mail_all_data');
      sessionStorage.removeItem('mail_parsed_page');
      sessionStorage.removeItem('mail_done');
      if (btn) btn.click();
    }
    window._isHandlingMail = false;
    return;
  }

  if (sessionStorage.getItem('mail_done') === 'true') {
    window._isHandlingMail = false;
    return;
  }

  var curPageStr = document.getElementById('ctl00_ContentPlaceHolder1_GridView3_ctl01_lblCurrentPage');
  var curPage = curPageStr ? parseInt(curPageStr.textContent.trim()) : 1;

  var parsedPage = sessionStorage.getItem('mail_parsed_page');
  if (parsedPage && parseInt(parsedPage) === curPage) {
    window._isHandlingMail = false;
    return;
  }

  var rows = document.querySelectorAll('.gv-row');
  var pageData = [];
  for (var i = 0; i < rows.length; i++) {
    var tds = rows[i].querySelectorAll('td');
    if (tds.length >= 7) {
      var dateRaw = tds[0].textContent.trim();
      var cls = tds[1].textContent.trim();
      var name = tds[2].textContent.trim();
      var type = tds[5].textContent.trim();
      var location = tds[6].textContent.trim().replace(/\s+/g, ' ');

      var formattedDate = dateRaw;
      var dateParts = dateRaw.split('-');
      if (dateParts.length === 3) {
        var rocYear = parseInt(dateParts[0]) - 1911;
        formattedDate = rocYear.toString() + dateParts[1] + dateParts[2];
      }

      pageData.push({
        name: name,
        department: cls + (type ? ' (' + type + ')' : ''),
        date: formattedDate,
        campus: location
      });
    }
  }

  var allData = JSON.parse(sessionStorage.getItem('mail_all_data') || '[]');
  allData = allData.concat(pageData);
  sessionStorage.setItem('mail_all_data', JSON.stringify(allData));
  sessionStorage.setItem('mail_parsed_page', curPage.toString());

  var nextBtn = document.getElementById('ctl00_ContentPlaceHolder1_GridView3_ctl01_lnkBtnNext');
  var hasNext = nextBtn && !nextBtn.getAttribute('disabled');

  if (curPage < 6 && hasNext) {
     nextBtn.click();
  } else {
    sessionStorage.setItem('mail_done', 'true');
    if (window.mobileApp) {
      window.mobileApp.postMessage({ detail: { type: 'MAIL_SUCCESS', payload: allData } });
    }
  }

  window._isHandlingMail = false;
})();
```

##### 信件 Webview 範例 ② — 彰師大（`ncue.edu.tw`，一般 `<table>` 列）

**`webviewScript`（可讀）：**

```javascript
(function() {
  if (window._hasParsedMail) return;
  var rows = document.querySelectorAll('table tr');
  if (rows.length === 0) return;

  window._hasParsedMail = true;
  var result = [];

  for (var i = 0; i < rows.length; i++) {
    var cols = rows[i].querySelectorAll('td');
    if (cols.length >= 4) {
      var dept = cols[0].innerText.trim();
      var name = cols[1].innerText.trim();
      var date = cols[2].innerText.trim();
      var campus = cols[3].innerText.trim();

      if (name.length > 0 && name !== '姓名') {
        result.push({
          department: dept,
          name: name,
          date: date,
          campus: campus
        });
      }
    }
  }

  if (window.mobileApp) {
    window.mobileApp.postMessage({ detail: { type: 'MAIL_SUCCESS', payload: result }});
  }
})();
```

### 🔗 4. 捷徑連結模組 (`links`)

首頁常用服務，可多群組。

* **`title`**：群組標題。
* **`style`**：`pill`（橢圓標籤）或 `list`（條列）。
* **`items`**：`title`、`url`、`icon`。

**欄位格式範例（無腳本，僅網址與群組；`icon` 可省略）：**

```json
"links": [
  {
    "title": "常用服務",
    "style": "pill",
    "items": [
      { "title": "教務系統", "url": "https://example.edu.tw/student/", "icon": "book" },
      { "title": "學生信箱", "url": "https://mail.example.edu.tw/", "icon": "mail" }
    ]
  },
  {
    "title": "其他",
    "style": "list",
    "items": [
      { "title": "圖書館", "url": "https://lib.example.edu.tw/" }
    ]
  }
]
```

> 各校實際連結請見倉庫內對應 `schoolId/config.json`。

---

## 🤝 貢獻規範 (Code of Conduct)

1. **不包含惡意腳本**：所有提交的 `webviewScript` 都會經過人工審核，嚴禁任何竊取使用者帳密、Cookie 或發送至第三方伺服器的行為。
2. **盡量使用 Vanilla JS**：因為是注入到各校不同的網頁中，請盡量使用原生 JavaScript (`document.querySelector`)，不要過度依賴 jQuery，以免舊版網頁不相容。

感謝您的熱心參與！因為有你，開源校園生態系才能更加繁榮！