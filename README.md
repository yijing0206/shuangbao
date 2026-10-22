# 雙寶複習冒險隊（單檔網頁版）

- 永久網址：https://yijing0206.github.io/shuangbao/ （GitHub Pages，從 `main` 分支發布）
- 檔案：`index.html`，單一檔案，不需要任何外部資源
- 更新方式：改好 `index.html` 後 `git commit` + `git push`，網址不變，存檔也會接續
- localStorage 跟著網址走，網址不同就是不同的存檔，請一律用上面這個網址。舊的 deed.page 網址已停用。

## 選擇題（v2）
- 10 張任務卡，每張卡小四 2 題、小六 2 題，共 40 題（小四 20 題、小六 20 題），每題 4 個選項，每次都重新洗牌。
- 作答前要先選信心。選對第一次 +1 星；選錯會顯示提示和那個選項錯在哪，可以再選一次，答對 +0.5 星；第二次也錯就公布答案和解說。
- 同一題一天只拿一次星星，之後是練習模式。
- 答錯的題目會標記 `wrongPending`，隔天地圖上會出現「⏰ 該複習」；之後哪天第一次就答對，標記就解除。
- 合體關、出題反轉卡、兌換特權維持開放題，星星要由家長輸入 4 位數 PIN 確認（第一次使用時設定，確認後 10 分鐘內免輸入；忘記的話，在存檔頁長按 5 秒可清除）。
- 正確答案、提示和解說經過輕度加密（XOR＋base64），存在 `MCQ` 的 `s` 欄位；作答後才解開，作答前不會出現在 DOM。
- 題庫原始檔不放在公開 repo，存在機器上的 `/workspace/shuangbao-src/questions.json`。改完執行 `node /workspace/shuangbao-src/build.mjs index.html`，會把題庫寫進 `/*__MCQ__*/…/*__END_MCQ__*/` 區塊。如果原始檔不見了，可以用頁面裡的 `secret()` 和 `MCQ_KEY` 解回來。
- 題目 id（例如 `math-a-g4-1`）和卡片 id 一樣，發布後永遠不改。

## 存檔格式
- 存在 `localStorage["shuangbao-save"]`，內容是 JSON。
- 升級前和匯入前，會把舊存檔複製到 `localStorage["shuangbao-save-prev"]`。

```jsonc
{
  "schemaVersion": 2,
  "createdAt": "ISO", "updatedAt": "ISO", "appVersionLastSeen": "1.0.0",
  "teamStars": 0,                      // 星寶的總星星數（可以有 .5），決定進化階段
  "kidStars": { "g4": 0, "g6": 0, "team": 0 },
  "redeemed": 0,                       // 已兌換的家庭特權數（每 15 星一個）
  "cards": { "<cardId>": {             // 用穩定的卡片 id 當 key，例如 "math-a"
      "cleared": false, "firstClearedOn": "YYYY-MM-DD", "lastPlayedOn": "YYYY-MM-DD",
      "playedOn": ["YYYY-MM-DD"], "stars": {"g4":0,"g6":0,"team":0}, "misses": {"g4":0,"g6":0}, "opened": 0,
      "mc": { "<questionId>": { "right":0, "retry":0, "wrong":0, "lastOn":"YYYY-MM-DD", "lastResult":"right|retry|wrong",
              "starOn":"YYYY-MM-DD", "wrongPending":false, "wrongOn":"YYYY-MM-DD", "everRight":false, "confLog":[["日期",信心,"結果"]] } } } },   // v2
  "flips": { "<flipId>": { "drawn": 0, "good": 0 } },
  "playDates": ["YYYY-MM-DD"],         // 台灣時間（Asia/Taipei）的遊玩日期
  "daily": { "YYYY-MM-DD": { "g4": {"stars":0,"actions":0,"misses":0,"right":0,"retry":0,"wrong":0},   // right/retry/wrong 為 v2
                             "g6": {"stars":0,"actions":0,"misses":0,"right":0,"retry":0,"wrong":0},
                             "team": {"stars":0,"actions":0},
                             "cards": { "<cardId>": { "g4": {"right":0,"retry":0,"wrong":0}, "g6": {…} } } } },
  "settings": { "petName": "星寶", "names": { "g4": "小四", "g6": "小六" }, "parentPin": null },   // PIN 只存雜湊值
  "history": [ { "at": "ISO", "from": 0, "to": 1, "app": "1.0.0" } ]   // 升級紀錄
}
```

- 連續天數不存，每次都從 `playDates` 算：從台灣時間的今天（今天還沒玩就從昨天）往回數連續的日期。
- 「該複習」也不存，每次重新算：有 `wrongPending` 的題目，隔天就該複習；另外從 `firstClearedOn` 和 `playedOn` 算第 1、3、7、14 天的複習。
- 版本紀錄：v1 → v2 加入 `cards.*.mc`、每日的 `right/retry/wrong/cards` 和 `settings.parentPin`，舊欄位和星星全部保留。
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
