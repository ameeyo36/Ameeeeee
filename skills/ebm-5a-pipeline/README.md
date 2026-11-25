# EBM 5A Pipeline：實證 5A 串接＋NCMEA 競賽簡報

依 Ask → Acquire → Appraise → Apply → Audit 完成一個臨床案例，串接 PICO、CEBM RCT、CASP SR/MA（RCT、觀察性研究）、RoB NMA 與 GRADE 評讀邏輯（已逐字收錄於 SKILL.md 附錄 T1–T6），先輸出結果頁面，再產出符合 NCMEA 評分標準的 PPT。

- 共 10 個停點（0、1、1b、2a、2–7），在需要使用者資料或確認時停下來詢問
- 評讀 CHECKLIST 逐題列出並回答，附評讀判定總表、OCEBM、GRADE 判定總表、SoF 與 NNT
- PPT：頁碼右上角、評分標籤在標題上方、內文 ≥ 14 pt、不放附錄、參考文獻採 AMA 格式
- 目前支援治療、預防與傷害題型；診斷、預後、病因題型需另行提供評讀工具

## Claude Code

下載下方 ZIP 並解壓縮，把整個 ebm-5a-pipeline 資料夾放到個人 `~/.claude/skills/`，或目前專案的 `.claude/skills/`。

完整路徑應為 `~/.claude/skills/ebm-5a-pipeline/SKILL.md`。開啟 Claude Code 後輸入：

```text
/ebm-5a-pipeline 貼上你的臨床情境（病人基本資料、病史、用藥、臨床困境、病人提出的問題）
```

## Claude 網頁版

1. 下載 [技能 ZIP](./ebm-5a-pipeline.zip)，不必解壓縮。
2. 在 Claude 啟用 Code execution and file creation。
3. 前往 **Customize → Skills → + → Create skill → Upload a skill**，上傳 ZIP 並啟用。
4. 開新對話，輸入 `/ebm-5a-pipeline` 並貼上臨床情境。
