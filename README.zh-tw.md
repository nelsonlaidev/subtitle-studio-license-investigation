# Subtitle Studio 許可證違規調查

Articue Tech Limited 開發的 Subtitle Studio (subtitlestudio.ai) 中發現的許可證合規性問題文件。

## 目錄

- [執行摘要](#執行摘要)
- [時間軸](#時間軸)
- [技術發現](#技術發現)
  - [GPL-3.0 違規](#gpl-30-違規)
  - [MIT 許可證違規](#mit-許可證違規)
  - [模型驗證](#模型驗證)
- [調查方法](#調查方法)
- [開發者回應](#開發者回應)
- [法律分析](#法律分析)
- [影響](#影響)
- [資源](#資源)
- [貢獻](#貢獻)
- [許可證](#許可證)

## 執行摘要

這項調查記錄了 Subtitle Studio 中的多項開源許可證違規行為，這是一款商業 macOS 應用程式，被宣傳為「隱私優先的 AI 字幕生成器」，售價 69 美元。

**主要發現：**

- 透過在閉源商業軟體中捆綁 `ffmpeg-static` 套件，違反了 GPL-3.0
- 透過使用 whisper.cpp/OpenAI Whisper 模型而未經署名，違反了 MIT 許可證
- 誤導性行銷將免費開源 AI 模型宣傳為專有的「高級 AI」
- 對於購買違規版本的客戶缺乏問責制

**目前狀態：**

- 開發者已承認 GPL-3.0 問題，並聲稱已在 v1.5.0 中修復
- 截至上次驗證，MIT 許可證違規仍未解決
- 未向現有客戶提供退款、賠償或解釋

## 時間軸

| 日期       | 事件                                                                |
| ---------- | ------------------------------------------------------------------- |
| 2026-01-05 | 初次發現並公開披露 GPL-3.0 違規                                     |
| 2026-01-05 | 發現應用程式模型與免費 Whisper large-v2 之間 SHA256 雜湊相同        |
| 2026-01-06 | 開發者回應：「v1.5.3 已移除 FFmpeg-static，大家使用許可證時要小心」 |
| 2026-01-07 | 最終分析文件                                                        |

## 技術發現

### GPL-3.0 違規

**受影響版本：** v1.0.0 至 v1.4.0

**套件：** ffmpeg-static v5.3.0 (GPL-3.0-or-later)

**違規：** 在閉源商業應用程式中分發 GPL 許可軟體，但未：

- 提供原始碼
- 將整個應用程式根據 GPL-3.0 許可
- 在可訪問位置包含 GPL 許可證文本

**證據：**

```json
// package.json (v1.4.0)
{
  "name": "subtitle-studio",
  "version": "1.4.0",
  "dependencies": {
    "ffmpeg-static": "^5.3.0"
  }
}

// node_modules/ffmpeg-static/package.json
{
  "name": "ffmpeg-static",
  "version": "5.3.0",
  "license": "GPL-3.0-or-later"
}
```

**關於「隱藏許可證」的說明：** 儘管在壓縮的 ASAR 檔案中的 `node_modules/ffmpeg-static/` 中存在 LICENSE 文件，但這不符合 GPL 要求。終端用戶無法在沒有技術工具（ASAR 提取）的情況下訪問這些文件，這使得它們實際上無法按照 GPL 的「包含在所有副本中」條款的要求進行訪問。

### MIT 許可證違規

**受影響版本：** 所有版本 (v1.0.0 至當前)

**組件：** 透過 whisper.cpp 使用 Whisper large-v2 模型 (MIT 許可證)

**違規：** 使用 OpenAI Whisper 模型，但未：

- 包含 whisper.cpp 作者或 OpenAI 的版權聲明
- 包含 MIT 許可證文本
- 在應用程式或網站的任何地方進行署名
- 披露開源使用情況

**分發方法：**

- v1.0.0-1.1.0：捆綁在應用程式中
- v1.2.0+：要求用戶從 `https://assets.subtitlestudio.ai/models/model.bin` 下載

**重要提示：** 將下載移至他們自己的託管服務並不能解決違規問題。他們仍然在分發 MIT 許可的模型而未經署名，無論是捆綁還是單獨託管。

### 模型驗證

**SHA256 比較：**

### Subtitle Studio 的模型

```
文件：   model.bin
大小：   1,080,732,091 位元組
SHA256： 3a214837221e4530dbc1fe8d734f302af393eb30bd0ed046042ebf4baf70f6f2
```

### 官方 Whisper large-v2-q5_0

```
文件：   ggml-large-v2-q5_0.bin
大小：   1,080,732,091 位元組
SHA256： 3a214837221e4530dbc1fe8d734f302af393eb30bd0ed046042ebf4baf70f6f2
```

**結果**：逐位元組相同

### 行銷與現實

網站聲明：「由高級 AI 提供支援」

現實：未經修改的開源 Whisper large-v2 模型，可在 https://huggingface.co/ggerganov/whisper.cpp 免費獲取

## 調查方法

這項調查可以由任何具有基本技術知識的人獨立驗證。

### 先決條件

```bash
# 安裝 asar 以提取 Electron 應用程式
npm install -g @electron/asar

# 下載 v1.4.0 (違規版本)
# https://assets.subtitlestudio.ai/releases/Subtitle%20Studio-arm64-1.4.0.dmg
```

### 重現步驟

**1. 提取應用程式：**

```bash
cd /Applications/Subtitle\ Studio.app/Contents/Resources/
asar extract app.asar ~/extracted-app
cd ~/extracted-app
```

**2. 驗證 GPL 違規 (v1.4.0)：**

```bash
# 檢查依賴項
cat package.json | grep ffmpeg-static
# 輸出："ffmpeg-static": "^5.3.0"

# 驗證許可證
cat node_modules/ffmpeg-static/package.json | grep license
# 輸出："license": "GPL-3.0-or-later"

# 檢查可訪問的 GPL 披露
# (沒有 UI 元素，沒有文件，沒有網站提及)
```

**3. 驗證模型身份：**

```bash
# 計算其模型的雜湊
shasum -a 256 model.bin
# 3a214837221e4530dbc1fe8d734f302af393eb30bd0ed046042ebf4baf70f6f2

# 下載官方模型
curl -L https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-large-v2-q5_0.bin -o whisper-official.bin
shasum -a 256 whisper-official.bin
# 3a214837221e4530dbc1fe8d734f302af393eb30bd0ed046042ebf4baf70f6f2
```

**結果**：相同

**4. 驗證無署名：**

- 打開 Subtitle Studio 應用程式
- 檢查菜單/設置中是否有「許可證」或「鳴謝」部分（不存在）
- 訪問 https://subtitlestudio.ai 並搜索「whisper」或「open source」（沒有提及）

## 開發者回應

回應發佈在 [Threads](https://www.threads.com/@mtmcy_ig/post/DTI0OoakiAb) 於 2026-01-06 00:01:41 HKT：

**原文 (粵語)：**

> 「Update: 1.5.3 版本已經移除 FFmpeg-static package，感激大家嘅關心！大家 vibe code 個陣都要小心 d license 野！另外新版本嘅推出包含一個 Burn in 字幕功能，可以 Edit 字幕樣式。雖然暫時未有 export video 功能，但未來會整，順便比大家 preview 定多謝大家支持！」

**翻譯：**

> 「更新：版本 1.5.3 已移除 FFmpeg-static 套件，感謝大家的關心！大家在 vibe code 時也要小心許可證問題！此外，新版本包含一個燒錄字幕功能，可以編輯字幕樣式。雖然暫時沒有導出影片功能，但未來會添加。感謝您的支持！」

**回應分析：**

- 透過移除套件，間接承認 GPL 違規
- 未向受影響客戶明確道歉
- 以隨意的語言淡化嚴重性
- 未提及 MIT 許可證違規
- 未提供退款或賠償
- 立即轉向宣傳新功能

## 法律分析

### 為何違反 GPL-3.0

GPL-3.0 要求任何分發 GPL 許可軟體的人：

1. 向接收者提供完整的原始碼
2. 將整個組合作品根據 GPL-3.0 許可
3. 使許可證條款易於用戶訪問

Subtitle Studio 因在閉源商業產品中包含 GPL 許可的 `ffmpeg-static` 而未能滿足所有三個要求。

**「隱藏許可證」問題：** 在壓縮的 ASAR 檔案中的 `node_modules/` 中存在 LICENSE 文件不符合 GPL 要求。這些文件：

- 無法在沒有技術工具的情況下訪問
- 對終端用戶不可見
- 不符合 GPL 要求的「人類可讀」形式

行業標準要求許可證可透過應用程式 UI 訪問（例如，「關於」→「許可證」菜單）。

### 為何違反 MIT 許可證

MIT 許可證只有一個要求：

> 「上述版權聲明和本許可聲明應包含在軟體的所有副本或實質部分中。」

Subtitle Studio 違反了這一點，因為：

1. 未經任何版權聲明使用 Whisper 模型
2. 未包含 MIT 許可證文本
3. 未在任何地方署名 whisper.cpp 或 OpenAI
4. 將模型宣傳為專有技術

**仍在違規：** 即使在 v1.2.0+ 中，用戶單獨下載模型，違規行為仍在繼續，因為：

- 使用 whisper.cpp 的應用程式碼仍需要署名
- 在他們自己的伺服器 (`assets.subtitlestudio.ai`) 上託管模型意味著他們仍在分發它
- 他們的下載頁面或應用程式中沒有署名

### 合規性是什麼樣子

**GPL 合規性：**

- 透過書面報價或直接包含提供原始碼
- 將整個應用程式根據 GPL-3.0 許可，或者
- 移除 GPL 許可的組件

**MIT 合規性：**

- 在應用程式 UI 中添加「鳴謝」或「開源許可證」部分
- 包含：「此應用程式使用 whisper.cpp (版權所有 2023-2024 The ggml authors) 和 OpenAI Whisper (版權所有 OpenAI) 根據 MIT 許可證」
- 包含完整的 MIT 許可證文本
- 在網站上披露應用程式使用開源 Whisper 模型

## 影響

### 對客戶

- 支付 69 美元購買重新打包的免費開源軟體
- 購買了存在許可證違規的版本
- 未收到通知、退款或解釋
- 被誤導為「專有 AI」技術

### 對開源作者

- whisper.cpp 和 ggml 作者：其作品未獲署名
- FFmpeg 開發者：GPL 條款被違反
- OpenAI：Whisper 模型未獲署名

### 對軟體行業

- 展示了「快速行動，被抓到再修復」的許可證處理方式
- 顯示了自動許可證合規性檢查的必要性
- 突顯了寬鬆許可證與實際署名實踐之間的差距

## 資源

**存檔證據：**

- 網站 (2026-01-06 GMT)：https://web.archive.org/web/20260106175103/https://subtitlestudio.ai/
- 開發者回應：https://www.threads.com/@mtmcy_ig/post/DTI0OoakiAb

**官方來源：**

- Subtitle Studio：https://subtitlestudio.ai
- 開發者公司：https://articue.io
- 下載連結 (v1.4.0)：https://assets.subtitlestudio.ai/releases/Subtitle%20Studio-arm64-1.4.0.dmg

**開源專案：**

- whisper.cpp：https://github.com/ggml-org/whisper.cpp (MIT)
- OpenAI Whisper：https://github.com/openai/whisper (MIT)
- ffmpeg-static：https://github.com/eugeneware/ffmpeg-static (GPL-3.0-or-later)
- FFmpeg：https://ffmpeg.org (GPL-2.0+/LGPL-2.1+)

**許可證文本：**

- GPL-3.0：https://www.gnu.org/licenses/gpl-3.0.html
- MIT 許可證：https://opensource.org/license/mit

**工具：**

- asar：https://github.com/electron/asar
- Whisper 模型：https://huggingface.co/ggerganov/whisper.cpp

## 貢獻

發現其他資訊或錯誤？請提交問題或拉取請求。

**準則：**

- 提供可驗證的證據（連結、截圖、技術數據）
- 保持事實、尊重的語氣
- 不進行人身攻擊或騷擾

## 許可證

本文件根據 CC0 1.0 Universal (公共領域) 發布。詳情請參閱 [LICENSE](LICENSE) 文件。

---

**上次更新：** 2026-01-07  
**調查員：** [@nelsonlaidev](https://github.com/nelsonlaidev)  
**聯絡方式：** me@nelsonlai.dev

**免責聲明：** 本調查僅用於教育和透明度目的。作者不鼓勵騷擾。所有資訊均基於公開數據和技術分析。Subtitle Studio 和 Articue Tech Limited 是其各自所有者的商標。
