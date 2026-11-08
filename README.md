# 弦光 Stringlight

手機橫向遊玩的節奏遊戲。音符沿著會移動、旋轉、分岔的「光弦」落下，碰到光弦時用手指打擊。

整個遊戲只用 HTML、CSS 和 JavaScript 寫成，不需要安裝或建置，放上任何網頁伺服器就能玩。

## 怎麼玩

1. 把手機轉成橫向，點一下標題畫面。
2. 第一次玩建議先選「新手教學」。
3. iPhone 要關掉靜音鍵才會有聲音，建議戴耳機。

| 音符 | 怎麼打 | 判定 |
| --- | --- | --- |
| Tap（藍） | 音符碰到光弦時點一下 | Perfect ±75ms、Good ±150ms、Bad ±180ms |
| Drag（黃） | 手指放在光弦上，音符經過就算 | Perfect 或 Miss |
| Hold（藍、長條） | 點下去並按住，最後 1/4 可以放開 | 頭判 Perfect／Good；尾判 Perfect 或 Miss |
| Flick（紅） | 在光弦上往任意方向快速一滑 | Perfect 或 Miss |

- 判定只看手指在光弦方向上的位置，離光弦多遠都可以。
- 分數滿分 1,000,000：準度 900,000（Good 算 65%）＋連擊 100,000。
- 評級：S+ ≥ 980,000、S ≥ 950,000、A ≥ 900,000、B ≥ 800,000、C ≥ 700,000，其餘為 D。

## 歌曲

| 歌曲 | 難度 | 說明 |
| --- | --- | --- |
| 新手教學 First Light | 1 | 依序教四種音符、會移動的光弦和分線，畫面上有提示文字 |
| 流光迴路 Lumen Circuit | 3 / 7 / 11 | 150 BPM 原創電子曲，用 Web Audio 即時合成，三種難度都是手工編排 |
| 心動的每一天 Everyday Heartbeat | 2 / 6 / 9 | 收錄的音檔，譜面由起音分析自動產生 |

## 設定

- **遊玩**：流速、音符大小（只改外觀，不改判定範圍）、判定偏移、延遲校準、自動演示
- **畫面**：背景亮度、FC/AP 指示器（全 Perfect 時光弦變金色、Full Combo 時變藍色）、多押提示、早／晚提示、低解析度模式
- **聲音**：音樂、打擊音效、介面音效

藍牙耳機通常有延遲，建議先用「延遲校準」：跟著「噠」聲點圓圈，會算出建議的偏移值。

## 在電腦上執行

```bash
python3 -m http.server 8000
# 打開 http://localhost:8000/
```

收錄的音檔要透過網頁伺服器讀取，直接雙擊 `index.html` 開啟時會讀不到。

### 用 GitHub Pages 發布

到 repo 的 **Settings → Pages**，Source 選 **Deploy from a branch**，分支選 `main`、資料夾選 `/ (root)`，存檔後約一分鐘就能在
`https://<帳號>.github.io/stringlight/` 遊玩。iPhone 用 Safari 開啟後「分享 → 加入主畫面」，就能全螢幕遊玩。

## 程式結構

| 檔案 | 內容 |
| --- | --- |
| `index.html`、`css/style.css` | 畫面與樣式 |
| `js/app.js` | 介面流程：標題、選曲、設定、延遲校準、暫停、結算、紀錄 |
| `js/game.js` | 遊戲引擎：時鐘、多點觸控、判定、計分、繪圖、開場與結尾動畫 |
| `js/chart.js` | 譜面模型：關鍵影格軌道、緩動函式、流速積分、寫譜工具 `ChartBuilder` |
| `js/synth.js` | 即時合成器：鼓、貝斯、墊音、主旋律等音色與排程播放器 |
| `js/songs/*.js` | 每首歌的樂譜與譜面 |
| `tools/analyze-onsets.js` | 開發工具：分析音檔的起音，產生 `js/songs/theme-onsets.js` |

### 寫譜

譜面用程式寫，單位是「拍」：

```js
const ch = new Chart.ChartBuilder({ bpm: 150, offset: 0.3 });
const A = ch.line({ x: 0.5, y: 0.72 });          // 判定線：位置是舞台寬高的比例
A.move(16, 4, { x: 0.3, angle: 10 }, "inOutSine"); // 第 16 拍起花 4 拍移動、旋轉
A.speed(32, 1.3);                                 // 第 32 拍起流速 1.3 倍
A.tap(8, -0.2);                                   // 第 8 拍、沿著線偏左的位置
A.drag(9, 0).flick(10, 0.2).hold(12, 2, 0);       // Drag、Flick、長 2 拍的 Hold
const chart = ch.build();
```

判定線的屬性有 `x`、`y`、`angle`、`alpha`、`len`（半長）。音符可以加上 `{ side: -1 }`，讓它從線的另一側過來。

### 重新分析音檔

```bash
node tools/analyze-onsets.js assets/audio/theme.mp3 > js/songs/theme-onsets.js
```

需要先安裝 ffmpeg。
