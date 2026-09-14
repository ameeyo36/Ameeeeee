# Figure → 可編輯 PowerPoint：Codex / Claude 共用技能

將 figure、AI 圖片、海報或截圖拆成文字、圖片、icon、形狀與表格，重建可編輯 PPTX，並檢查重疊與版面差異。

共用同一份 SKILL.md；agents/openai.yaml 僅為可選的 Codex 顯示資訊。執行環境仍需具備圖片檢視、檔案產生及 PPTX 製作能力。

## Codex

在 Codex 貼上以下安裝指令：

```text
使用 $skill-installer，從以下 GitHub 網址下載並安裝這個技能：
https://github.com/ameeyo36/Ameeeeee/tree/main/skills/figure-to-editable-ppt-reconstruction
```

若尚未顯示，重新啟動 Codex。上傳圖片後輸入：

```text
使用 $figure-to-editable-ppt-reconstruction，將這張圖還原為可編輯 PPTX。
保留原文、比例、配色與布局，提供預覽與重建報告。
```

## Claude Code

下載下方 ZIP 並解壓縮，把整個 figure-to-editable-ppt-reconstruction 資料夾放到個人 `~/.claude/skills/`，或目前專案的 `.claude/skills/`。

完整路徑應為 `~/.claude/skills/figure-to-editable-ppt-reconstruction/SKILL.md`。Windows 的 `~` 是使用者家目錄。開啟 Claude Code，提供圖片路徑並輸入：

```text
/figure-to-editable-ppt-reconstruction 將這張圖還原為可編輯 PPTX，保留原文與布局，提供預覽與重建報告。圖片路徑：你的圖片路徑
```

## Claude 網頁版

1. 下載 [技能 ZIP](./figure-to-editable-ppt-reconstruction.zip)，不必解壓縮。
2. 在 Claude 啟用 Code execution and file creation。
3. 前往 **Customize → Skills → + → Create skill → Upload a skill**，上傳 ZIP 並啟用。
4. 在聊天中上傳原圖，輸入：

```text
請使用 figure-to-editable-ppt-reconstruction 技能，將這張圖還原為可編輯 PPTX。
保留原文、比例、配色與布局，提供預覽與重建報告。
```

Claude 網頁版使用技能名稱的自然語言提示，不需要 Codex 的 $skill-installer。組織帳號可能需要管理員啟用技能功能。

ZIP 結構：

```text
figure-to-editable-ppt-reconstruction/
├── SKILL.md
└── agents/
    └── openai.yaml
```

## 輸出與限制

輸出 PPTX、實際渲染預覽、疊圖、差異圖與 JSON 報告；缺少渲染器時標示尚未完成視覺驗證。文字與簡單幾何優先原生重建，複雜插畫可能保留為獨立圖片。單張圖無法保證恢復原字型、被遮住的像素及所有圖層，不承諾 100% 相同或全部像素可編輯。

已檢查共用格式與 ZIP 結構；尚未在 Claude 帳號內實際上傳及轉檔驗證。各平台工具與字型可能造成成品差異。

官方文件：[Codex Skills](https://learn.chatgpt.com/docs/build-skills)、[Claude Code Skills](https://code.claude.com/docs/en/skills)、[Claude 技能使用](https://support.claude.com/en/articles/12512180-use-skills-in-claude)、[Claude ZIP 封裝](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills)。
