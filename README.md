# 雙寶複習冒險隊（單檔網頁版）

- 網址（永久、固定）：https://shuangbao-558031f7.deed.page/
- 檔案：`index.html`，單一檔案，執行時不需要任何外部資源
- 更新：改好 `index.html` 後執行 `./publish.sh`，網址不變
- `.deed-token` 是更新這個網址的唯一鑰匙。沒設恢復信箱，弄丟就不能再更新，只能換新網址（再用匯出/匯入搬存檔）。請另外備份，不要公開。
- 不要讓孩子用 `v1--…deed.page` 這類版本網址玩。網址不同，存檔就不同。

## 存檔格式
- 存在 `localStorage["shuangbao-save"]`，內容是 JSON。
- 升級前和匯入前，會把舊存檔複製到 `localStorage["shuangbao-save-prev"]`。

```jsonc
{
  "schemaVersion": 1,
  "createdAt": "ISO", "updatedAt": "ISO", "appVersionLastSeen": "1.0.0",
  "teamStars": 0,                      // 星寶的總星星數，決定進化階段
  "kidStars": { "g4": 0, "g6": 0, "team": 0 },
  "redeemed": 0,                       // 已兌換的家庭特權數（每 15 星一個）
  "cards": { "<cardId>": {             // 用穩定的卡片 id 當 key，例如 "math-a"
      "cleared": false, "firstClearedOn": "YYYY-MM-DD", "lastPlayedOn": "YYYY-MM-DD",
      "playedOn": ["YYYY-MM-DD"], "stars": {"g4":0,"g6":0,"team":0}, "misses": {"g4":0,"g6":0}, "opened": 0 } },
  "flips": { "<flipId>": { "drawn": 0, "good": 0 } },
  "playDates": ["YYYY-MM-DD"],         // 台灣時間（Asia/Taipei）的遊玩日期
  "daily": { "YYYY-MM-DD": { "g4": {"stars":0,"actions":0,"misses":0},
                             "g6": {"stars":0,"actions":0,"misses":0},
                             "team": {"stars":0,"actions":0} } },
  "settings": { "petName": "星寶", "names": { "g4": "小四", "g6": "小六" } },
  "history": [ { "at": "ISO", "from": 0, "to": 1, "app": "1.0.0" } ]   // 升級紀錄
}
```

- 連續天數不存，每次都從 `playDates` 算：從台灣時間的今天（今天還沒玩就從昨天）往回數連續的日期。
- 「該複習」也不存，從 `firstClearedOn` 和 `playedOn` 算：第 1、3、7、14 天各複習一次。
- 內容資料（`CARDS`、`FLIPS`、`SCOPE` 等）寫在程式裡，不放進存檔。

## 存檔碼與檔案
- 存檔碼：`"SB1:" + base64(UTF-8 JSON)`。
- 下載的檔案是一般 JSON。
- 匯入時兩種都收，也收純 JSON 文字。匯入前會先執行 `migrate()`，再取代目前存檔；原本的存檔會先放到 `-prev`。

## 升級規則（以後每一版都要遵守）
1. `SAVE_KEY` 永遠是 `shuangbao-save`，網址也永遠是同一個。
2. 卡片 `id` 一旦發布就不能改、不能重複使用。要拿掉卡片，就從 `CARDS` 刪除，但存檔裡的進度要留著。新卡片用新的 id。
3. 存檔結構有變時：`SCHEMA_VERSION` +1，並在 `MIGRATIONS` 加一個步驟 `N: s => {...; return s;}`，把 v(N-1) 轉成 vN。
4. 只能新增欄位，不能刪除或改名，也不能改舊欄位的意義。需要新意義就開新欄位。新欄位的預設值寫在 `defaultSave()`，`fillDefaults()` 會自動補進舊存檔。
5. 存檔裡程式不認得的欄位要原樣保留，不能清掉。
6. 存檔版本比程式新時，不降級、不刪資料。
7. 發布前，先用前一版的存檔測一次：放進 localStorage，重新整理，確認星星、過關、連續天數都還在。
