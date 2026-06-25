# HANDOFF — 手機旗艦對戰小遊戲

> 記錄 HTML 檔本身看不出來的東西:部署、設計取捨、待辦。
> 遊戲玩法/卡牌資料看 `phone-battle.html`(自我說明,有中文註解)。

## 🌐 正式網址(公開,已上線)
**https://mansonliu.github.io/phone-battle/** — 任何網路、任何裝置都能玩,不用開電腦、不用同 Wi-Fi。
- 頁面內「📲 掃碼分享」鈕會跳出 QR(向量 SVG,指向上面這個公開網址)。

## 怎麼改版 / 重新部署
1. 改 `~/git/phone-battle-site/index.html`(這是部署用的檔,等同 `phone-battle.html`)。
2. `cd ~/git/phone-battle-site && git add -A && git commit -m "..." && git push`
3. 約 30–40 秒後 GitHub Pages 自動更新上線。
- repo:`https://github.com/mansonliu/phone-battle`(public,main 分支,Pages source = main /)
- git 推送認證走 `gh`(已 `gh auth login` 過,帳號 mansonliu,HTTPS token 存 keyring)。

## 檔案位置
| 檔案 | 用途 |
|---|---|
| `~/git/phone-battle-site/index.html` | **部署用主檔**(改這個 → push) |
| `~/git/phone-battle-site/qr.png` | 產 QR 的暫存(頁面內已改用 inline SVG,不依賴此檔) |
| `~/claude/phone-battle.html` | 開發原始檔,與 index.html 同步(已統一成公開網址版) |

## 已退役(不要再用)
- 家裡 Mac mini 的 LaunchAgent 區網伺服器(`com.manson.phonebattle.plist` :8765)**已停止並移除**。
- 舊的本機 QR(`~/claude/qr.html`、`phone-battle-qr.png`)指向已失效的 `LiudeMac-mini.local:8765`,**已刪除**。
- 之後若我又提「連 LiudeMac-mini.local:8765」是過時資訊 → 一律用上面的 github.io 公開網址。

## 玩法 / 規格(已定案)
- 單一 HTML 網頁,響應式。**2026-06 大改版**:從深色霓虹電競風改成**明亮寶可夢卡牌對戰風**(受眾是小學生,玩法刻意不加深,只把畫面做得像線上遊戲)。
- **雙卡 PK**:挑兩支,四屬性(效能/相機/螢幕/電池,各 0–100)比總分(滿分 400)。玩法本身沒變,只是包裝。
- **選機 = 選角格子牆**(取代舊版下拉選單):依品牌分區的手機圖格子,點一格進「發光的那一格」(我方🔵/對手🔴),點完自動換邊;點空卡片可切換要填哪格。`activeSide` 控制目前要填的一側,`buildRoster()`/`pickTile()`/`updateActiveUI()`。骰子/隨機照舊。
- **手機圖**:用 emoji/向量風,實際是 `phoneSVG(p)` 即時畫的 SVG(手機背面+鏡頭模組,**鏡頭數量隨相機分數 1~4 顆**,品牌色+首字母 logo)。選了 emoji/SVG 版而非 AI 生成插畫或真實照片,維持單檔可攜、零版權。
- **美術風格**:亮色天空+綠擂台地面+飄浮光點背景;白卡+品牌色粗邊框+右上角「戰力 HP」+中央擂台窗格放手機圖+四屬性做成「招式列」(⚡效能金/📷相機紫/🖥️螢幕青/🔋電池綠,各有屬性色),招式下方顯示真實規格(`p.sp`,跟舊版一樣)。
- **對戰演出**(`battle()`):全螢幕回合橫幅(`roundBanner()`:準備…→各屬性對決→VICTORY!)→ 開場**兩顆火焰流星拳 🤜 從左右拖火焰尾巴水平對衝相撞**(`#clash` 的 swordL/swordR,emoji 已從劍→📱→🔥→👊→側面🤜,`::before` 拉火焰殘影)→ 逐項出招:贏方射屬性光彈 `projectile()` 打對手、跳傷害數字 `popNum()`、差距≥20 跳「超強!」、戰力槽累加 → 勝方 👑 + ⭐星星評價(`setStars()`,領先 ≥80/≥30/其他 = 3/2/1 星)+ 彩帶 + 號角。
- 音效全 Web Audio 即時合成(無音檔):metalClang / explosion / zap(出招咻聲) / fanfare / stampThud + 背景音樂(~140 BPM 循環)。**背景音樂改預設關**(不再首戰自動播);分頁切到背景時 `visibilitychange` 暫停排程省電。加了 `prefers-reduced-motion` 降級。空白鍵/Enter 可開戰。
- **CSS 3D(A 方案,純 CSS 無函式庫)**:卡片滑鼠/陀螺儀立體傾斜視差 + 反光、選卡翻牌登場、衝撞時兩卡 3D 轉身互撞、勝者前浮敗者後傾。實作用 CSS 變數組合單一 transform(`--tiltX/--tiltY` 傾斜、`--lift/--scale` 勝負、`--lungeY/--lungeZ` 互撞)互不覆蓋;結構:`#cardX` = `.card-stage`(perspective 容器),內層 `.card` 由 renderCard 注入;陀螺儀在首次對戰於手勢內 `enableGyro()` 請求權限。

## 分數是怎麼來的(重要)
- **手工相對分級,非實測跑分。** 依 SoC 世代、感光元件/變焦、解析度/更新率/亮度、電池容量/快充判定。要改直接改 `PHONES` 陣列數字。

## 卡池(63 張,2006–2026)
- 完整年度旗艦:Apple / Samsung / Google / Xiaomi / ASUS(含 ROG 電競)。
- HTC:2006–2018 完整旗艦史(TyTN→U12+),2018 後淡出故止於 U12+。
- 退出市場、刻意只放真實舊機(分數低、懷舊向):**Acer**(2016 止)、**BenQ**(BenQ-Siemens 2007 解散 + 2015 中階 F5)。
- 其他:Sony / OnePlus / Huawei / vivo / OPPO 各少數代表作。

## 待辦 / 之後可做
- [ ] 補其他品牌早期經典機(初代 iPhone 2007、Galaxy S 2010、Nokia 等)—— 使用者目前說「先不用補其他機型」。
- [ ] 2026 機型(S26 Ultra 等)規格定案後補。
- [ ] 屬性加權(目前四項等權)。
- [ ] 進階玩法:人機對戰 / Top Trumps 單屬性比拼(當初討論過,選了雙卡 PK)。
- [ ] Acer Predator 6(2015 發表未量產)—— 使用者若不介意可補。
- [ ] 3D 升級到 Three.js 真 3D(B 方案:立體厚度卡牌、空間運鏡)—— 使用者目前選了 A 方案 CSS 3D,「先這樣」;要更炫再評估。
- 微調備忘:傾斜幅度目前 ±14°(pointermove handler 內);各 3D 效果是獨立 CSS 變數,可單獨拆掉。對戰節奏靠 `battle()` 裡的 `sleep()` 數值調;火焰拳/光彈顏色在 `#clash .sword` 與 `STATCOL`。
