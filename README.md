# WORKFLOW HUB · AI 協作平台

> 人提供資訊，AI 編譯知識；要做決定時，答案找得到、也查得到。

WORKFLOW HUB 是一套給製造業內部使用的 **AI 協作平台**：把每日工作日誌、專案任務、品質異常、追料、議價、市場情報、會議與公告集中在同一個入口，並以 **AI 知識編譯器（AKC）** 把散落在各模組的資訊整理成可追溯、可治理的組織知識。

![儀表板](docs/screenshots/dashboard.jpg)

> 📷 本文所有畫面皆為系統實際介面，資料為示範用虛構資料。

---

## 目錄

- [核心特色](#核心特色)
- [功能模組](#功能模組)
- [AI 知識編譯器（AKC）](#ai-知識編譯器akc)
- [系統架構](#系統架構)
- [技術棧](#技術棧)
- [專案結構](#專案結構)
- [安裝與啟動](#安裝與啟動)
- [安全與治理](#安全與治理)
- [Roadmap](#roadmap)

---

## 核心特色

| | |
|---|---|
| 🧩 **一個入口、多個模組** | 工作、品質、船務、採購、開發、業務、知識，共用帳號、權限、通知與稽核 |
| 🧠 **AI 知識編譯** | 9 階段編譯管線把原始資料變成 17 種知識物件，決策問答附引用出處 |
| 🔌 **AI 供應商可切換** | Claude / Gemini / OpenAI / 本機 Ollama，於畫面上設定，模組程式碼不需修改 |
| 🔐 **權限與稽核** | 7 動作 × 4 範圍權限、Token 身分驗證、三層稽核日誌 |
| 🏭 **接 ERP / APS** | 直接讀取 ERP 與 APS 報表資料，追料與議價不需重複輸入 |

---

## 功能模組

### 工作

| 登入 | 任務看板 |
|---|---|
| ![登入](docs/screenshots/login.jpg) | ![任務](docs/screenshots/tasks.jpg) |

- **工作日誌 / 儀表板**：每日紀錄、專案進度、待辦與通知一頁掌握
- **專案與任務**：專案 → 任務 → 工作紀錄，支援指派、狀態、附件

### 品質

| 品質異常 | AI 根因分析 |
|---|---|
| ![品質異常](docs/screenshots/qc.jpg) | ![AI RCA](docs/screenshots/qc-ai-rca.jpg) |

- **品質異常（QC）**：異常登錄、8D 流程、AI 協助根因分析（RCA）
- **客訴管理**：客訴案件追蹤與回覆

![客訴](docs/screenshots/complaint.jpg)

### 船務

![進出口](docs/screenshots/import-export.jpg)

- **進出口管理**：報單、往來對象與文件管理

### 採購

| 追料管理 | 議價戰情室 |
|---|---|
| ![追料](docs/screenshots/material-chase.jpg) | ![議價清單](docs/screenshots/nego-list.jpg) |

![議價明細](docs/screenshots/nego-detail.jpg)

- **追料管理**：串接 ERP / APS，自動列出缺料、交期風險與催料紀錄
- **議價戰情室**：原物料指數、匯率、新聞與歷史價格，協助準備議價論點

### 開發

| 開發領航台 | 流程知識庫 |
|---|---|
| ![領航台](docs/screenshots/navigator.jpg) | ![知識庫](docs/screenshots/kmnote.jpg) |

- **開發領航台**：AI 訪談 → 需求審核 → 方案（含不寫程式的選項）→ 一步一提詞、一步一驗證
- **流程知識庫**：流程 / SOP 與知識點樹狀管理，修改後自動退回「待確認」

### 業務

| 市場情報 | 市場聲量 |
|---|---|
| ![市場情報](docs/screenshots/market-intel.jpg) | ![市場聲量](docs/screenshots/market-voice.jpg) |

- **市場情報**：競品、專利、新聞自動收集與 AI 摘要

### 協作

| 會議 | 公告 |
|---|---|
| ![會議](docs/screenshots/meeting.jpg) | ![公告](docs/screenshots/announcement.jpg) |

- **會議**：議程、語音轉文字（瀏覽器或 Whisper）、AI 會議紀錄與待辦
- **公告**：公告發布、已讀追蹤

---

## AI 知識編譯器（AKC）

AKC 把各模組產生的原始資料（日誌、異常、會議、文件…）編譯成結構化知識。

![AKC 儀表板](docs/screenshots/akc-dashboard.jpg)

**9 階段編譯管線**

```
理解 → 分類 → 拆分 → 實體 → 去重 → 關係 → 評分 → 衝突 → 治理
```

![AKC 編譯](docs/screenshots/akc-compile.jpg)

- **17 種知識物件、6 個層級**：從事實、規則到決策經驗
- **決策問答（D1–D6）**：回答附引用編號（`KO-xxxx`），可點回原始知識
- **知識缺口與主題頁**：系統主動指出「缺什麼、該問誰」
- **防幻覺**：使用者原文以 `<user_input>` 隔離包覆；輸出經 JSON Schema 驗證，並做字串 / 數字比對（C1–C7），不合格整筆不落庫

![AKC 決策問答](docs/screenshots/akc-decide.jpg)

---

## 系統架構

```mermaid
flowchart LR
    U[使用者瀏覽器<br/>React 18 + Vite] -->|HTTPS + Token| F[Flask app.py]
    F --> BP[Blueprints<br/>*_routes.py]
    BP --> SEC[wj_security<br/>Token / POLICIES]
    BP --> AI[_v16_ai_call<br/>統一 AI 入口]
    AI --> C[Claude]
    AI --> G[Gemini]
    AI --> O[OpenAI / Whisper]
    AI --> L[Ollama 本機]
    BP --> DB[(SQL Server<br/>WorkJournalDB)]
    BP --> ERP[(ERP)]
    BP --> APS[(APS 報表)]
    BP --> AUD[(稽核日誌)]
```

**後端**：`app.py` 為核心，各模組以 Blueprint 形式掛載，透過依賴注入取得共用服務：

```python
init_xxx(get_conn, ai_call, perm_check, audit)
```

每個 Blueprint 以 `try/except` 掛載，單一模組載入失敗不影響整個平台。

**前端**：單一外殼 `work-journal-v22.07.jsx` 負責登入、導覽與共用元件，各功能以 `*-module.jsx` 獨立載入。導覽分組：工作 / 模組（品質、船務、採購、開發、業務）/ 知識 / 設定 / 管理。

### AI 設定

![AI 設定](docs/screenshots/ai-settings.jpg)

所有模組的 AI 呼叫都透過同一個入口；管理者在畫面上選擇供應商與模型，「回答」與「搜尋（embedding，預設 bge-m3）」模型分開設定，敏感資料可改走本機 Ollama。

---

## 技術棧

| 層 | 技術 |
|---|---|
| 前端 | React 18、Vite |
| 後端 | Python、Flask（Blueprint 架構） |
| 資料庫 | Microsoft SQL Server |
| AI | Anthropic Claude、Google Gemini、OpenAI（含 Whisper）、Ollama |
| 向量 | bge-m3 embedding |
| 外部系統 | ERP、APS 報表 |

---

## 專案結構

```
.
├── backend/
│   ├── app.py                      # Flask 核心、AI 統一入口、Blueprint 掛載
│   ├── wj_security.py              # Token 驗證與路由權限 POLICIES
│   ├── workjournal_routes.py       # 工作日誌
│   ├── projects_routes.py / task_routes.py
│   ├── qc_routes.py / complaint_routes.py
│   ├── material_chase_routes.py    # 追料
│   ├── nego_routes.py              # 議價（含 nego_*_fetch.py 資料抓取）
│   ├── mi_routes.py                # 市場情報（含 mi_fetch*.py）
│   ├── navigator_routes.py         # 開發領航台
│   ├── km_routes.py / kmnote_routes.py
│   ├── meeting_routes.py / notifications_routes.py
│   ├── impexp_routes.py / impexp_party_routes.py
│   ├── akc_routes.py / akc_batch_routes.py   # AI 知識編譯器
│   └── sql/schema_v*.sql           # 資料庫結構（依版本遞增）
├── frontend/
│   └── src/
│       ├── main.jsx
│       ├── work-journal-v22.07.jsx # 主外殼
│       └── *-module.jsx            # 各功能模組
└── docs/
    ├── AKC_規格書_v1.0.md
    └── screenshots/
```

> 實際檔名與目錄請依你的 repo 調整。

---

## 安裝與啟動

### 1. 資料庫

```sql
-- 於 SQL Server 建立 WorkJournalDB，依版本順序執行
schema_v*.sql
```

### 2. 後端

```bash
cd backend
pip install -r requirements.txt
# 建立 db_config.ini（連線字串、AI API Key，勿提交至 Git）
python app.py
```

`db_config.ini` 範例：

```ini
[database]
server   = YOUR_SQL_SERVER
database = WorkJournalDB
username = YOUR_USER
password = YOUR_PASSWORD
```

### 3. 前端

```bash
cd frontend
npm install
npm run dev        # 開發
npm run build      # 正式建置
```

> ⚠️ 請將 `db_config.ini`、API Key 與任何含公司資料的檔案加入 `.gitignore`。

---

## 安全與治理

- **權限 7 動作 × 4 範圍**：檢視、新增、修改、刪除與三種附件權限，各分「全公司 / 同部門 / 僅自己 / 不可」；判定失敗一律拒絕（fail-closed）
- **身分只信伺服器 Token**：不接受前端傳來的使用者 ID；SQL 全部參數化；API Key 只存在後端
- **三層稽核**：平台操作日誌、知識治理日誌（只增不改）、AI 呼叫追溯（提詞雜湊、原始回應、耗時）
- **防幻覺不靠 AI 自律**：輸入隔離、Schema 驗證、後處理比對，不合格不落庫

