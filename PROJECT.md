# 日文學習工具（kakumei）

## 網址
- 使用：https://kakumei.vercel.app
- 程式碼：https://github.com/summywang/kakumei

## 技術架構
- 單一檔案：`index.html`（約 1600 行）
- 無框架，原生 HTML / CSS / JS
- API：Anthropic Claude Haiku（用戶自備 API Key，存在 localStorage）
- 部署：Vercel（GitHub push 自動部署）
- 字體：Noto Serif JP、Shippori Mincho、Noto Sans TC、Material Icons Round

## 更新流程
```bash
cd ~/Library/CloudStorage/GoogleDrive-summy5677@gmail.com/我的雲端硬碟/claude/nihongo
git add index.html && git commit -m "描述" && git push
```

---

## 現有功能

### 首頁
- 你的內容（預載革命道中 + 使用者儲存的內容）
- 情境對話（機場、餐廳預設三難度 + 自訂情境）
- 新增內容（貼文字 / AI 搜尋）
- ⚙ 設定（難度、API Key）

### 閱讀頁
- 振假名、羅馬音、中文翻譯切換顯示
- 語速調整（0.3× ～ 1.5×）
- 點整行朗讀
- 點單字出現泡泡（朗讀 + 查詢）
- 查詢：語法結構分析 + 例句 + 重新生成
- 長按整行編輯（編輯 / 插入 / 刪除，輸入中文 AI 翻日文）
- 全部朗讀
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
| `anthropic_api_key` | API Key 字串 |
| `global_diff` | `'N5N4'` \| `'N3N2'` \| `'N1'` \| `'ADV'` |
| `sc_{情境}_{難度}` | `{ lines: [...] }` |
| `custom_scenarios` | `['藥局', ...]` |
| `saved_content` | `[{ title, lines, date }, ...]` |
| `word_{詞}_{難度}` | `{ meaning, grammar, examples }` |
