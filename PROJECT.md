# 日文學習工具（kakumei）

## 網址
- 使用：https://kakumei.vercel.app
- 程式碼：https://github.com/summywang/kakumei

---

## ⚡ Claude 新 Session 必讀（最優先執行）

**每次新 Session 開始，Claude 應該：**
1. 從 GitHub 讀取最新 `PROJECT.md`（用 `git show origin/main:PROJECT.md` 或 fetch）
2. 確認 Cowork 選取的資料夾是 `~/Dev/nihongo/`（不是 `~/kakumei/`）
3. 詢問使用者想做什麼，**不要**主動要求使用者跑任何指令

**使用者不需要自己跑指令，除非：**
- 這台電腦是**第一次使用**（需要 clone，見下方「新電腦設定」）
- 需要重新在 Cowork 選取資料夾

---

## 開發工具設定

### Cowork 資料夾
```
~/Dev/nihongo/
```
> Cowork 必須選取這個資料夾，Claude 才能讀寫最新的 `index.html`

> ⚠️ 不要在 Google Drive 路徑下直接 git 操作，會因 sync 衝突產生 .lock 檔

### Claude Context 同步方式
- **claude.ai Project**「日語學習工具」已連結 GitHub repo（`summywang/kakumei`）
- 同步檔案：`PROJECT.md`、`index.html`
- 每次 `git push` 後，claude.ai Project 會自動更新 knowledge
- **Cowork** 直接讀本機 `~/Dev/nihongo/`，不需要額外設定

---

## 新電腦第一次設定（使用者自行執行）

> 只有這台電腦從來沒用過才需要做這段。

```bash
git clone https://github.com/summywang/kakumei.git ~/Dev/nihongo
git config --global user.name "Summy"
git config --global user.email "summy5677@gmail.com"
```

Clone 完之後，在 Cowork 重新選取資料夾 `~/Dev/nihongo/`，Claude 就能讀到最新檔案。

---

## 標準發版流程（每次開發完必做）

> ⚠️ 這段由 **Claude** 在 Cowork 執行，使用者不需要自己跑。

```bash
cd ~/Dev/nihongo
git add index.html
git commit -m "簡短描述這次改了什麼"
git push
# push 完：Vercel 自動部署（30 秒）、claude.ai Project knowledge 自動更新
```

> 💡 每次對話結束前，Claude 要確認有沒有跑 `git push`，否則 kakumei.vercel.app 不會更新，下個 session 也會讀到舊版。

---

## 如果遇到 .lock 檔錯誤

```bash
find ~/Dev/nihongo/.git -name "*.lock" -delete
```

---

## 技術架構
- 單一檔案：`index.html`（約 1900 行）
- 無框架，原生 HTML / CSS / JS
- API：Anthropic Claude Haiku（用戶自備 Key，存在 localStorage）
- TTS：ElevenLabs（Voice ID: `baaIB19ClFmB5qcPSImI`，木村拓哉日本男聲）
- 音檔 Cache：IndexedDB（`el_audio_cache`，key = `{voiceId}:v2:{text}`）
- 部署：Vercel（GitHub push → 自動部署，通常 30 秒內生效）
- 字體：Noto Serif JP、Shippori Mincho、Noto Sans TC、Material Icons Round

---

## 現有功能

### 首頁
- 你的內容（使用者儲存的貼文 / AI 搜尋結果）
- 情境對話（機場、餐廳預設三難度 + 自訂情境）
- 新增內容（貼文字 / AI 搜尋）
- ⚙ 設定（難度、Anthropic Key、ElevenLabs Key）

### 閱讀頁
- 振假名、羅馬音、中文翻譯切換顯示
- 詞性顯示切換
- 點整行朗讀（ElevenLabs 或 Web Speech API fallback）
  - 載入中：紫色脈衝指示條
  - 播放中：金色指示條
- 點單字出現泡泡（朗讀 + 查詢）
- 查詢 Sheet：詞意、語法結構分析、例句（附朗讀）、重新生成
- 長按整行編輯（編輯 / 插入 / 刪除，輸入中文 AI 翻日文）
- 全部朗讀（含預載下一行、loading 狀態）
- ⋯ more menu（編輯名稱、刪除內容）

### 情境對話
- 機場、餐廳預設三難度手寫內容
- 自訂情境：輸入描述，AI 同時生成三難度
- 生成新對話（可附對話要點）
- 儲存對話 / 放棄確認

### 全域設定
- 難度：N5-N4 簡單 / N3-N2 中等 / N1 進階 / N1+ 大師
- 影響查字解釋深度、例句複雜度
- Cache 按難度分開存

### AI 搜尋
- 輸入關鍵字，AI 整理成學習素材

---

## localStorage 結構

| Key | Value |
|-----|-------|
| `anthropic_api_key` | Anthropic API Key 字串 |
| `elevenlabs_api_key` | ElevenLabs API Key 字串 |
| `global_diff` | `'N5N4'` \| `'N3N2'` \| `'N1'` \| `'ADV'` |
| `sc_{情境}_{難度}` | `{ lines: [...] }` |
| `custom_scenarios` | `['藥局', ...]` |
| `saved_content` | `[{ title, lines, date }, ...]` |
| `word_{詞}_{難度}` | `{ meaning, grammar, examples }` |

## IndexedDB 結構

| DB | Store | Key 格式 | Value |
|----|-------|----------|-------|
| `el_audio_cache` | `audio` | `{voiceId}:v2:{text}` | Blob（mp3） |

---

## 已知技術決策

- **AudioContext 取代 new Audio()**：ElevenLabs fetch 是 async，若用 `new Audio().play()` 會因超出 user gesture 時間窗口被 autoplay policy block。現在在每個 click handler 中同步呼叫 `unlockAudioCtxSync()`，AudioContext 一旦 running 就不受 async 影響。
- **預載下一行**：全部朗讀時，播放第 n 行同時 background fetch 第 n+1 行，命中 IndexedDB cache 則第 n+1 行無等待時間。
- **speakELWithCallback**：單句朗讀用，fetch 完播放，播完執行 callback 移除 playing class。
- **Google Drive 不放 git repo**：Drive sync 會干擾 `.git/` 資料夾產生 lock 檔，repo 統一放 `~/Dev/nihongo/`。
