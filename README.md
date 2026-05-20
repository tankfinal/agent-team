# agent-team

Tank 個人的 Claude Code Agent Teams 模板。提供兩個預先設計好的 team，可一鍵啟動 / 重建 / 連接，並用 tmux split pane 即時觀察所有 teammate 的工作與互相對話。

## 兩個 Team

### develop team（開發任務）
- **Teammates**：`pm`、`be`、`fe`
- **你的對口**：`pm`（產品經理 / Tech Lead）
  - pm 跟你討論釐清需求、給 trade-off 讓你拍板
  - 拍板後 pm 拆 task、派工給 be / fe、驗收
  - **pm 不直接寫 code**（沒 Write/Edit/Bash），刻意設計逼他派工
- **跳過 pm**：用 `@be` / `@fe` 直接點名工程師問技術問題
- **適用情境**：從零做網站 / app、需求模糊但想動工、產品開發迭代

### consult team（跨領域討論）
- **Teammates**：`be-manager`、`blockchain-expert`、`ai-researcher`
- **使用者角色**：給題目、聽辯論，必要時 `@點名` 追問
- **三位互動**：彼此可直接 SendMessage 對話（口語化中文，像群組辯論），不用透過 Lead 路由每句話
- **適用情境**：技術選型、跨領域 brainstorm、撞論點、找盲點

> 同時間只能跑其中一個 team（Claude Code 限制：1 個 lead session = 1 個 team），透過獨立 tmux session 做切換。

---

## 快速開始

### 前置需求

| 項目 | 版本 / 說明 |
|------|------|
| Claude Code | ≥ 2.1.32（Agent Teams 需要） |
| tmux | 任意版（推薦 ≥ 3.x） |
| 環境變數 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | repo 內 `.claude/settings.json` 已開啟 |
| `~/.tmux.conf` | 建議套用本 repo 的 `docs/tmux.conf`（滑鼠模式 + prefix 視覺指示） |

詳細的 tmux 安裝、設定與快捷鍵說明：見 [`docs/tmux-guide.md`](docs/tmux-guide.md)。

### 啟動 develop team

```bash
./scripts/dev-rebuild.sh   # 重建（kill 舊的 + 起新的）+ sanity check 列出載入的 md
./scripts/dev-attach.sh    # 連到 tmux session 看 lead
```

rebuild 會把 `prompts/dev-lead.md` 直接塞給 claude 當 initial prompt，Lead 自動跑 `TeamCreate` + 三個 `Agent(team_name="dev")` spawn pm / be / fe。attach 進去三個 pane 已開好，直接對 pm 描述需求，例：

```
我想做一個個人記帳網站，可以匯入信用卡 CSV
```

pm 會開始問你 1–3 個關鍵問題，釐清完才派工。

### 啟動 consult team

```bash
./scripts/consult-rebuild.sh
./scripts/consult-attach.sh
```

同樣自動帶 `prompts/consult-lead.md` 啟動，attach 進去三個 pane 已開好。直接給題目即可：

```
我們該不該自建 LLM？
```

---

## @mention 點名

兩個 team 的 Lead 都認得 `@<agent-name>` convention（**注意**：Claude Code 原生 `@` 是引用檔案，這裡是 Lead 自訂解析）。

### dev team
| 你的訊息 | 路由 |
|---|---|
| 一般訊息 | → `pm`（預設） |
| `@pm ...` | → `pm` |
| `@be ...` / `@fe ...` | 跳過 pm 直送工程師 |
| `@be @fe ...` | 同 turn 兩個 SendMessage 平行 |

### consult team
| 你的訊息 | 路由 |
|---|---|
| 一般訊息 | → 三位（廣播） |
| `@be-manager ...` | → 該位 |
| 點多位 | 同 turn 多個 SendMessage |
| 「收尾」/「彙總」 | Lead 請三位給最終結論並彙總 |

---

## 腳本一覽

| 腳本 | 用途 |
|------|------|
| `scripts/dev-rebuild.sh` | 砍掉 `tank-dev` tmux session，重建 + 自動把 `prompts/dev-lead.md` 餵給 claude |
| `scripts/dev-attach.sh` | 連到 `tank-dev` |
| `scripts/consult-rebuild.sh` | 砍掉 `tank-consult`，重建 + 自動把 `prompts/consult-lead.md` 餵給 claude |
| `scripts/consult-attach.sh` | 連到 `tank-consult` |

`*-rebuild.sh` 跑起來會印出實際在 `.claude/agents/*.md` 找到的清單，驗證該 team 預期的角色都在位（缺的會 ⚠️），然後 spawn `claude <prompt>` 讓 Lead 一啟動就直接 `TeamCreate` + 開三個 pane。修改 agent md 或 lead prompt 後一定要 rebuild 才會生效，正在跑的 session 不會 hot reload。

---

## 目錄結構

```
agent-team/
├── README.md                    ← 本文件
├── CLAUDE.md                    ← repo 規範（Claude Code 啟動時自動載入）
├── .claude/
│   ├── agents/                  ← 六位角色定義
│   │   ├── pm.md                ← develop team 的使用者對口
│   │   ├── be.md
│   │   ├── fe.md
│   │   ├── be-manager.md
│   │   ├── blockchain-expert.md
│   │   └── ai-researcher.md
│   └── settings.json            ← repo 層級設定（teammateMode: tmux + 實驗 flag）
├── prompts/                     ← Lead 的初始 prompt（rebuild 腳本會餵給 claude）
│   ├── dev-lead.md
│   └── consult-lead.md
├── scripts/                     ← 一鍵啟動 / 重建 / 連接
└── docs/
    ├── tmux-guide.md            ← tmux 安裝、教學、客製化
    └── tmux.conf                ← 推薦的 ~/.tmux.conf
```

---

## 更新 / 迭代角色

想改某位 teammate 的人格 / 專業 / 工具許可權：

1. 編輯 `.claude/agents/<name>.md`
2. 跑 `./scripts/<team>-rebuild.sh`（rebuild 完 Lead 自動啟動，pane 直接出現）
3. 重新 attach 進去看新版

> 已經在跑的 team **不會** pick up 修改 —— 必須 rebuild 才會吃到新版。

新增角色：在 `.claude/agents/` 放 `<name>.md`，然後在對應 team 的 `prompts/<team>-lead.md` 把該角色加進 spawn 步驟，並更新 `scripts/<team>-rebuild.sh` 的 `EXPECTED_AGENTS=()`。

---

## Caveats（已知限制）

- **Split pane 需要 `team_name` 參數**：team mode 是用 `TeamCreate` + `Agent(team_name=..., name=...)` 觸發，缺 `team_name` 的話 Agent 只會 spawn 普通 subagent，不會 split pane。兩個 `prompts/*-lead.md` 已經寫死這個流程。
- **Lead 不可轉移**：建立 team 的那個 Claude Code 進程才是 lead。重開機後 tmux server 死掉，team 就要重建。
- **Subagent frontmatter `skills` / `mcpServers` 不繼承為 teammate**：但**全域 `~/.claude/settings.json` 配的 MCP 仍可用**。be-manager 用 atlassian MCP 沒問題（前提：你 user-level 有設）。
- **同一 lead session 一次只能管一個 team**：要切換時務必 clean up team 後才能建另一個（或直接 kill tmux session 讓腳本重建）。
- **`/resume` 無法還原 in-process teammate**：本 repo 統一用 tmux split pane，detach 後可 reattach，但別 `exit`。
- **idle teammate 是正常狀態**：spawn 後 teammate 完成每輪會 idle，等下一個 SendMessage 喚醒。不要當錯誤。

---

## License

Personal use. No license attached.
