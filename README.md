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

## 電子寵物（v3）
- 每個孩子各有一台蛋形寵物機（小四藍白、小六粉紫），液晶畫面裡是像素寵物。造型是原創的，沒有使用任何品牌的名稱或角色。
- 第一次使用按 B 鍵（或「領養」）：從 星喵、星兔、星熊、星龍、星鯨 選一隻，再幫牠取名字。
- 選單有 🍙 餵食、🎾 玩耍、🧽 清潔、💡 睡覺／開燈、📊 狀態、🎁 送禮。A 鍵移動游標或換食物，B 鍵確認，C 鍵取消；直接點圖示也可以。側邊旋鈕可以跟寵物打招呼。
- 選擇題答對的 ⭐ 會存進自己的錢包，用來買食物：🍎 1 顆（成長 +1）、🍙 2 顆（+2）、🍰 3 顆（+3，開心 +25）。
- 狀態有三項：飽足、開心、學習力。答對會提升學習力。每過一天沒照顧，飽足 −25、開心 −20、學習力 −15，最低降到 10，寵物不會死。另外每天會多一坨便便（最多 3 坨）。每餵 3 次也會出現便便。
- 玩耍一天最多 3 次。
- 進化看成長值：蛋 → 寶寶（5）→ 少年（15）→ 成體（30）。孵化和進化都有動畫。成體會依照得到最多星星的科目分支成數學博士、自然探險家、地球旅行家、文字魔法師、英語小達人。
- 互助：送 1 ⭐ 給對方的寵物（對方成長 +1、開心 +15）；合體關教學經家長 PIN 確認後，兩隻寵物都會加成長值。每次都會讓「🤝 互助」計數 +1。
- v2 → v3 遷移：原本共用星寶的成長值（= `teamStars`），兩隻寵物都完整繼承，另外各送 3 ⭐ 見面禮。`teamStars` 保留，繼續用來兌換家庭特權，不會扣除。

## 家庭雲端同步（v4，可選）
- 沒有加入家庭時完全單機，不載入 Firebase、不連網。
- 第一台平板：存檔 → ☁️ 家庭同步 → 選「這台是誰的」→ 建立家庭（要家長 PIN），會產生 24 碼家庭碼（每 4 碼一組顯示），可以複製或分享加入連結 `#join=家庭碼`。
- 第二台：打開加入連結，或輸入家庭碼 → 加入家庭 → 選角色。打錯碼會顯示「找不到這個家庭」，不會開出空家庭。
- Firebase 專案 `shuangbao-family`：匿名登入，Firestore 路徑 `families/{家庭碼}`、`families/{家庭碼}/kids/{g4|g6}`、`families/{家庭碼}/shared/main`（合體星星、互助、家長 PIN 雜湊）。安全規則在 `firestore.rules`（家長要在 Firebase 主控台發布）。規則還沒發布時，畫面顯示「🔒 同步尚未啟用」，遊戲照常玩，資料存在本機。
- 合併規則：成長值、錢包、合體星星用「每台裝置各自累加」的計數器（G-counter），合併時每台取最大值再加總，不會重複也不會遺失；寵物狀態以最後修改的那台為準；答題紀錄、遊玩日期取聯集；其他數字取最大值。禮物和合體加成用 Firestore `increment` 寫進對方的 `inbox`，斷線時也只會送一次。
- 離線照玩（localStorage + Firestore 離線快取），連線後自動同步。右上角狀態燈：💻 單機／☁️ 已同步／⏳ 同步中／📴 離線／🔒 同步尚未啟用。
- v3 → v4 遷移：舊的成長值、錢包和合體星星放進計數器的 `legacy` 欄位，數值不變。兩台在同步前各自玩過時，`legacy` 取較大的那台（不會相加）。

## 存檔格式
- 存在 `localStorage["shuangbao-save"]`，內容是 JSON。
- 升級前和匯入前，會把舊存檔複製到 `localStorage["shuangbao-save-prev"]`。

```jsonc
{
  "schemaVersion": 4,
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
                             "cards": { "<cardId>": { "g4": {"right":0,"retry":0,"wrong":0}, "g6": {…} } },
                             "pets": { "g4": {"fed":0,"played":0,"cleaned":0,"gifts":0}, "g6": {…} } } },   // pets 為 v3
  "settings": { "petName": "星寶", "names": { "g4": "小四", "g6": "小六" }, "parentPin": null },   // PIN 只存雜湊值
  "pets": { "g4": { "species":"dino", "name":"小恐龍", "adoptedOn":"YYYY-MM-DD", "xp":0, "wallet":0,
                   "fullness":70, "happiness":70, "learn":40, "poop":0, "lightsOff":false, "lastTickOn":"YYYY-MM-DD",
                   "fedCount":0, "play":{"date":null,"n":0}, "giftsGiven":0, "giftsReceived":0, "subjectXP":{"數學":0},
                   "stageSeen":0, "inheritedXp":0, "migratedV3":true }, "g6": { … } },        // v3
  "help": { "count":0, "gifts":0, "teach":0 },                                             // v3 互助
  // v4：pets.*.xpC / earnC / spendC = {裝置id: 累計}（舊值在 "legacy"），xp = Σxp + inbox.xp，wallet = Σearn − Σspend
  //     pets.*.stateT（寵物狀態最後修改時間）、pets.*.inbox {xp,happy,count}、pets.*.inboxApplied
  //     teamC = {裝置id: 合體星星}、teamBase；teamStars = teamBase + kidStars.g4 + kidStars.g6 + Σ teamC；settings.pinT
  //     settings.parentPin = {alg:"sha256", salt, hash}（加鹽雜湊）；裝置設定另存在 localStorage["shuangbao-device"] {id, role, family}
  "history": [ { "at": "ISO", "from": 0, "to": 1, "app": "1.0.0" } ]   // 升級紀錄
}
```

- 連續天數不存，每次都從 `playDates` 算：從台灣時間的今天（今天還沒玩就從昨天）往回數連續的日期。
- 「該複習」也不存，每次重新算：有 `wrongPending` 的題目，隔天就該複習；另外從 `firstClearedOn` 和 `playedOn` 算第 1、3、7、14 天的複習。
- 版本紀錄：v3 → v4 加入同步用的計數器、`inbox`、`stateT`、`teamC`、`teamBase`，PIN 改成加鹽雜湊（舊 PIN 第一次輸入時自動升級）。
- 版本紀錄：v2 → v3 加入 `pets`、`help` 和每日的 `pets`。
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
