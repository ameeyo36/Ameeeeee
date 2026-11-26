# EBM 5A 實證案例串接：Codex / Claude 共用技能

依 Ask → Acquire → Appraise → Apply → Audit 完成一個臨床實證案例：PICO 拆解與同類藥物全展開、PubMed／Embase／Cochrane 三層檢索語法、PRISMA 2020、依研究設計分流的逐題評讀（CEBM RCT、CASP SR/MA、RoB NMA）、OCEBM 2011、GRADE 與 SoF、NNT／NNH、EtD、4E、成本效益、SDM 衛教單張與 Audit。所有結果在對話中呈現，最後輸出整合報告與 HTML 結果頁面。不產出 PPT。

共用同一份 SKILL.md；agents/openai.yaml 僅為可選的 Codex 顯示資訊。SKILL.md 內有「平台對應」表，說明 Claude 與 Codex 的工具差異。

## 使用前須知

- **停點為強制**：流程中有 10 個停點（0、1、1b、2a、2、3、4、5、6、7），需要使用者提供資料或確認後才繼續；只回「繼續」不會跳過。
- **檢索由使用者執行**：技能輸出完整檢索語法，使用者自行到資料庫檢索並回報筆數。
- **背景資料需自備**：UpToDate、DynaMed 等付費資料庫內容由使用者貼上。
- **全文 PDF 需自備**：評讀與截圖依使用者上傳的全文。
- PDF 截圖需要 PyMuPDF：`pip install pymupdf`。

## Codex

在 Codex 貼上以下安裝指令：

```text
使用 $skill-installer，從以下 GitHub 網址下載並安裝這個技能：
https://github.com/ameeyo36/Ameeeeee/tree/main/skills/ebm-5a
```

或手動把整個 `ebm-5a` 資料夾放到 `~/.agents/skills/`（Windows：`C:\Users\<帳號>\.agents\skills\`）。若尚未顯示，重新啟動 Codex。使用時輸入：

```text
使用 $ebm-5a，依 5A 步驟處理以下臨床情境，遇到停點先問我：
（貼上臨床情境）
```

## Claude Code

下載下方 ZIP 並解壓縮，把整個 ebm-5a 資料夾放到個人 `~/.claude/skills/`，或目前專案的 `.claude/skills/`。完整路徑應為 `~/.claude/skills/ebm-5a/SKILL.md`。使用時輸入：

```text
/ebm-5a （貼上臨床情境）
```

## Claude 網頁版

1. 下載 [技能 ZIP](./ebm-5a.zip)，不必解壓縮。
2. 在 Claude 啟用 Code execution and file creation。
3. 前往 **Customize → Skills → + → Create skill → Upload a skill**，上傳 ZIP 並啟用。
4. 在聊天中輸入：

```text
/ebm-5a （貼上臨床情境）
```

組織帳號可能需要管理員啟用技能功能。

ZIP 結構：

```text
ebm-5a/
├── SKILL.md
└── agents/
    └── openai.yaml
```

## 輸出與限制

- 輸出：對話中的完整 5A 結果、原文森林圖與 SoF 截圖、整合報告（AMA 第 11 版參考文獻），以及 HTML 結果頁面（Claude 發布為網頁；Codex 輸出 HTML 檔）。
- 技能不捏造數字、引文、藥價或 SDM 結果；缺資料一律標「待補」。
- 評讀判斷由執行的 AI 依附錄提示詞與規則產生，仍需臨床人員確認。
- 已在 Claude 實際執行一個案例至 Apply（停點 7）；Codex 版僅檢查共用格式與 ZIP 結構，尚未在 Codex 實際跑完整案例。
