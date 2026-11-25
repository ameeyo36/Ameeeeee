---
name: "ebm-5a-pipeline"
description: "Run an EBM 5A case (Ask→Acquire→Appraise→Apply→Audit) with the user's PICO/CEBM/CASP/RoB NMA/GRADE tools and build an NCMEA competition PPT."
---

# EBM 5A Pipeline（實證 5A 串接＋競賽簡報）

串接使用者自寫的實證工具（pico.tsx、cebm_RCT.tsx、casp_sr_ma_rct.tsx、casp_SR_MA_OBSERVATION.tsx、rob_nma.tsx、grade_ai.tsx），依 5A 步驟完成一個臨床案例，先以頁面呈現結果，再依需求產出符合 NCMEA 評分標準的 PPT。

## 總原則（每一步都適用）

1. **思考邏輯 100% 還原**：6 個工具的判斷邏輯（題目、提示詞、判定規則、輸出欄位、公式）已逐字收錄在文末「工具邏輯附錄 T1–T6」，不需再上傳 TSX。一律依 T1–T6 逐字執行，不自行增刪題目或改寫判定邏輯。原工具呼叫 Gemini API；在本 Skill 中由 Claude 自己依附錄提示詞與規則執行，網頁介面程式碼不收錄。附錄提示詞中的「JSON」為輸出欄位規格，呈現給使用者時改為表格，但欄位一個不少。若使用者上傳新版 TSX，以新版為準，並提醒更新本 Skill。
2. **不捏造**：數字、引文、頁碼、作者、藥價、SDM 結果、Audit 資料一律來自文件、檢索結果或使用者提供；缺資料就標「待補」並在停點詢問。
3. **禁止縮寫輸出**：檢索語法、關鍵字、藥名清單一律完整輸出；不得寫「9 種 gliflozin」「…」「等」來代替實際內容。引文也不可用「…」截斷。
4. **原圖不重畫**：森林圖、SoF、表格、網絡圖一律截取原文（嵌入圖原始位元組或頁面渲染），不重繪、不改數字；PRISMA 流程圖例外，可用可編輯圖形繪製（本案自己的檢索流程）。
5. **在停點停下來**：遇到下方標示的停點，用 AskUserQuestion 詢問或請使用者上傳，取得答案才繼續。
6. **語言與格式**：繁體中文；條列與表格優先；評讀判定總表用直式表格。
7. **引用**：臨床事實與指引建議必須有可查證來源；未查證者標「待查證」。
8. **題數核對**：逐題評讀輸出前，核對列數＝工具題數（RoB NMA 17 題；CASP SR/MA-RCT 20 個評讀組件；CASP SR/MA-observational 18 題；CEBM RCT 依 T2 結構）。題目的 domain／section 欄位一律轉成同一型別再比對（rob_nma.tsx 的 3.8 題 domain 為字串 "3"，比對時用 String() 或 Number()，否則會漏題）。

## 停點總覽

| 停點 | 時機 | 要問／要的東西 |
|---|---|---|
| 0 | 收到情境 | 確認情境、病人基本資料是否完整（年齡、病史、用藥、困境、病人的問題） |
| 1 | PICO 產生後 | 選哪一個 PICO；元素組合（如 P AND I AND C AND O） |
| 1b | PICO 分析與三層關鍵字檢索語法輸出後 | 請使用者貼上臨床疾病的背景資料（UpToDate 或 DynaMed 為主，可加 BMJ Best Practice、教科書段落），連同主題名稱、作者、最後更新日期與該主題列出的 References；需涵蓋疾病機轉、流行病學、標準治療、目前爭議點。沒有收到就不進入下一步 |
| 2a | 背景資料收到後 | 每個資料庫：檢索日期、筆數、限制條件（年份、語言、研究類型 Filter）、去重數、各階段排除數與理由、全文排除數與理由 |
| 2 | PRISMA 後 | 上傳全文 PDF；若有多篇候選，提供候選清單以做選文理由表 |
| 3 | 判定研究設計後 | 確認每篇的研究設計與對應評讀工具 |
| 4 | 評讀完成後 | 確認判定；是否需要額外截圖頁面（頁碼）；截圖整頁或裁切 |
| 5 | GRADE 前 | 確認結局清單、重要性（critical／important）、台灣情境補充 |
| 6 | 結果頁面完成後 | 是否產出競賽 PPT |
| 7 | 建 PPT 前 | 報告者、單位、組別、日期；院內藥價（品項、劑量、日藥費、來源與日期）；SDM 實際結果；病歷待補欄位；Audit 實際追蹤資料（若有） |

## Step 1 Ask

1. 依 pico.tsx 的分析邏輯從情境拆出 PICO（可多個）、問題類型（治療／診斷／預後／病因／傷害）、最適合的研究設計、keywordsTable（element、mesh、emtree、synonyms、drugs）。
2. **四象限分析**：診斷、問題、偏好、困難（來自情境與病歷；缺的標待補，於停點 7 補）。
3. **病人歷程**：確診 → 治療 → 共病 → 現況 → 新症狀 → 決策點 → SDM → 追蹤，時間軸呈現。
4. **ASK ① 指引佐證 PICO**：查最新指引（如 ADA、ESC、ACC/AHA、NICE、台灣學會指引），列出建議原文要點、等級，以及佐證了 P／I／C／O 哪一項，尤其是「結果的依據」。
5. **ASK ② 問題型態 → OCEBM 2011 表**：依問題類型對應 OCEBM 表列，指出 Level 1 研究設計，作為 Acquire 篩選依據。
6. **背景知識（Background）**：停點 1b 收到貼上內容後，整理成下表；每格標明來源，不自行杜撰：

| 面向 | 內容重點 | 來源 |
|---|---|---|
| 疾病機轉 | 致病機轉、介入藥物作用機轉 | UpToDate／DynaMed／BMJ Best Practice／教科書（使用者貼上） |
| 流行病學 | 盛行率、發生率、死亡率、台灣數據 | 同上；台灣數據可另查國健署或健保署公開資料 |
| 標準治療 | 目前第一線、第二線治療與建議等級 | 背景資料庫＋臨床指引（ESC、ACC/AHA、NICE、ADA、台灣學會），指引由我檢索並附來源 |
| 目前爭議點 | 證據缺口、互相矛盾的建議、病人的疑慮 → 為什麼要問這個問題 | 背景資料內容＋指引差異整理 |

   - 說明背景資料引用的 References（哪幾篇支持哪個論點）；編號對不到完整書目時，請使用者貼上 References 清單，可用 PubMed 查證的文中指名試驗先補。
   - 參考文獻註明資料庫、主題名稱、作者、最後更新日期、檢索日期。
   - 使用者附的背景資料圖表以原圖放入，不重畫。
   - 付費資料庫內容無法自行讀取時，只用使用者貼上的內容，不臆測；未涵蓋的面向標「待補」。
7. **PICO 鎖定**：病人利弊、風險、臨床經驗、決定（Apply 階段回填）。
→ 停點 1。

## Step 2 Acquire

1. 以 keywordsTable 執行 build_boolean（見附錄 A，移植自 pico.tsx buildBooleanStrategies）：
   - PubMed："MeSH"[Mesh] → term[tiab] → term[all fields]
   - Embase：'emtree'/exp → term:ti,ab,kw
   - Cochrane：[mh "MeSH"] → "phrase"
   - 元素內 OR、元素間依使用者選擇的組合 AND；C 標示 not required 時略過。
2. 完整輸出三個資料庫語法（不可縮寫），附資料庫連結，請使用者自行貼上檢索。
→ 停點 1b：請使用者貼上臨床疾病背景資料（UpToDate 或 DynaMed），整理成背景知識（Step 1 第 6 點）後才繼續。
3. **依證據金字塔（6S）由上往下**建議檢索順序：Systems → Summaries（UpToDate、DynaMed、指引）→ 整合型 TRIP → SR（Cochrane Library）→ 原始研究（PubMed、Embase）→ 中文（華藝）→ 進行中試驗（ClinicalTrials.gov）。說明各資料庫選用理由。
4. **檢索功能**：MeSH／Emtree explode、欄位限定、布林邏輯、片語、Filter、截字（*）。只呈現實際用到的功能；語法中沒有截字時要明說並說明原因（例如 PubMed 不支援左截字，改以全名展開），不可宣稱有用。
→ 停點 2a（PRISMA 數字），再畫 PRISMA 2020 流程（找到 → 去重 → 篩選 → 全文 → 納入，每步排除數與理由）。
5. **選文理由**：候選文獻比較表（年份、研究設計、證據等級、與 PICO 的相符度、樣本數、是否選用及理由）。
→ 停點 2（上傳全文）。

## Step 3 Appraise

### 3.1 研究設計分流（不同設計用不同工具）

| 研究設計 | 工具（以使用者 TSX 為主） |
|---|---|
| RCT | cebm_RCT.tsx（CEBM RCT 評讀單 2023 繁中版）；使用者要求時加 RoB 2 |
| SR／MA of RCTs | casp_sr_ma_rct.tsx（CASP 2024）；使用者要求時加 AMSTAR 2 |
| SR／MA of observational | casp_SR_MA_OBSERVATION.tsx |
| Network meta-analysis | rob_nma.tsx（RoB NMA tool, Lunny et al., BMJ 2025;388:e079839） |
| Cohort | CASP cohort 或 NOS（無 TSX 時用官方清單原文，先告知使用者） |

一篇 NMA 不可改用 CASP SR／MA；設計判定先經停點 3 確認。對應附錄：RCT → T2；SR／MA of RCTs → T3；SR／MA of observational → T4；NMA → T5。

- 工具內建預檢：T2 先判斷是否為 RCT（不是就停止並告知「非此研究設計文獻」）；T3 先判斷是否為 SR／MA（不是就停止並告知偵測到的設計與理由）。
- 診斷、預後、病因題目目前沒有對應工具，需請使用者提供（例如 QUADAS-2 或 CEBM Diagnosis 評讀單）。
- OCEBM 2011 逐篇評定依 T6-5 提示詞（7 大問題類型、升降階原則、輸出欄位）。

### 3.2 評讀執行

1. 以 pdf_tools text 取全文（附 [PDF_PAGE_i | printed p.X] 標記），每題答案附英文原文引句（完整、不截斷）、頁碼與繁中翻譯。
2. 判定規則逐字照 TSX。RoB NMA：domain 內任一題 False → 高偏差風險；任一題 Probably false 或 No information → 有些疑慮；否則低。整體：任一 domain 高 → 高；任一有些疑慮 → 有些疑慮；否則低。domain 理由完整呈現，不截成 40 字。
3. 涵蓋三面向：效度（validity）、重要性（importance：效果量、CI、異質性）、適用性（applicability）。
4. 截圖：pdf_tools figures 找頁 → capture 存原圖、300 dpi 裁切與整頁；每張圖標 PDF 頁與印刷頁、對應評讀題號。
5. OCEBM 2011 逐篇標起始等級 → 升降階理由 → 最終等級。

### 3.3 逐題列出評讀結果（必做）

每篇文獻、每個評讀工具的**每一題都要單獨列出**。不可只給 domain 結論、不可合併題目、不可只列有問題的題目、不可用「其餘皆是」等字樣省略。

| 題號 | 題目（工具原文，中英對照） | 判定（工具原選項） | 理由（繁中） | 原文引句（英文完整不截斷） | 頁碼（PDF 頁／印刷頁） |
|---|---|---|---|---|---|

- 題數以工具為準、一題不少（例：RoB NMA 17 題、CASP SR／MA 全部題目、CEBM RCT 評讀單全部題目）；輸出前核對列數＝工具題數。
- 逐題表之後，**必須呈現評讀判定總表（直式）**：每篇文獻一欄；列為評讀工具的每個 domain（或 CASP／CEBM 的每個區段）＋整體判定＋使用工具＋OCEBM 最終等級＋本案角色（主要／輔助證據）；每格寫判定（顏色標示）與關鍵題號理由，不截斷。總表與逐題表必須一致（domain 判定依 TSX 規則由逐題結果推出）。
- 逐題表與判定總表都要在三個地方呈現：① 對話中的評讀結果；② Step 7 結果頁面；③ Step 8 PPT（判定總表獨立一頁，放在該工具逐題頁之後）。
- PPT 評讀段順序：工具分流 → 逐題評讀 → 評讀判定總表 → **納入文獻與 OCEBM 證據等級** → 結果呈現（森林圖、NNT）→ GRADE 判定總表與 SoF。結果頁面（Step 7）依同一順序。
- PPT 逐題頁：每頁 2–3 題（內文字級 ≥ 14 pt），題數多就增加頁數，不縮字也不刪題；引句放不下時放備忘稿，投影片上保留題目、判定、理由與頁碼。每頁標題註明「文獻｜工具｜第 x–y 題／共 n 題」。
- 判定用顏色標示（Yes／低風險：sage；Can't tell／有些疑慮：sand；No／高風險：deep rose）。

→ 停點 4。

## Step 4 GRADE

1. 結局清單與重要性 → 停點 5。
2. 每個結局依 grade_ai.tsx：五降級因子＋三升級因子判斷與理由；calculateCertainty（附錄 B）：起始 4，serious −1、very-serious −2，large +1、very-large +2，dose-response、residual confounding 各 +1，限制 1–4。
3. 絕對效果：absolute risk = OR·p ÷ (1 − p + OR·p)（RR 時 = RR·p）。
   **NNT／NNH（必做）**：
   - 用**未四捨五入**的風險差計算：ARR = CER − EER；NNT = 1／ARR **無條件進位**；NNH = 1／ARI **無條件捨去**（附錄 C；同 T3、T4 規則）。不可用四捨五入後的「每 1000 人少 x 人」回推。
   - 附 95% CI／CrI：把相對效果上下限代入同一公式；區間跨過無差異時寫「NNTB a 至 ∞ 至 NNTH b」。
   - 每個結局列 CER、EER、ARR、NNT（95% CI）、白話解讀；在對話、結果頁面與 PPT（獨立一頁 NNT／NNH 表，備忘稿寫計算過程）都要呈現。
   - GRADE 判斷與建議生成依 T6-6、T6-7 提示詞。
4. **GRADE 判定總表（必須呈現）**：列為各結局、欄為五降級＋三升級因子與最終確信度（−1／−2／+1 一目了然），下方附每個降／升級的理由（Evidence Profile）。
5. **GRADE SoF 證據摘要表（必須呈現）**：每個結局一列，一個不少。表頭寫明 Patient／Intervention／Comparison／Setting，欄位固定：

| 結局（追蹤期間） | 重要性 | 相對效果（95% CI／CrI） | 對照組風險（每 1000 人） | 介入組風險（每 1000 人） | 絕對差異（每 1000 人，95% CI） | 受試者（研究數） | 證據確信度（⊕⊕◯◯ 符號＋文字） | 降／升級理由註腳 |
|---|---|---|---|---|---|---|---|---|

   - GRADE 判定總表與 SoF 表都要在三個地方呈現：① 對話中的 GRADE 結果；② Step 7 結果頁面；③ Step 8 PPT（自製可編輯表格；另併附原文 SoF 截圖供對照）。
   - 數字來自文獻或計算結果，備忘稿寫計算過程；受試者數／研究數文獻未報告就標「未報告」。
6. 建議（方向、強度）、健保給付整理（標待確認）、實務指引（標明哪些非評讀文獻結果）、SDM 單張草稿。

## Step 5 Apply

1. **相似性**：病人與研究族群比較（年齡、共病、族群、介入劑量）。
2. **利弊權衡**：每 1000 人絕對效果、NNT／NNH、文獻未報告的傷害要明說。
3. **CEBM → GRADE → EtD**：EtD 2×2（證據品質高／低 × 病人偏好差異小／大）：高＋小 → 強建議；高＋大 → 條件式需 SDM；低＋小 → 多為條件式；低＋大 → 條件式、SDM 必要。依據 Alonso-Coello 2016、Andrews 2013。
4. **4E**：Evidence（證據、指引）、Experience（臨床）、Expectations（病人期待）、Environment（健保、成本）。
5. **成本效益**：本土 CEA 文獻（以 PubMed 查證作者與 DOI；標示非 PRISMA 納入）＋院內藥價計算「預防 1 例事件的額外藥費＝NNT × 年藥費差 × 追蹤年數」；健保給付規定與仿單。
6. **SDM 四步**：說明選項 → 比較選項 → 了解偏好（0–5 分在意程度）→ 做決定；認知是非題。依衛福部 SDM 平台 PDA 原則（不誘導、同時說明好壞、說明不確定性）與 Ottawa Decision Aid。衛教單張用非專業用語與每 1000 人絕對數（加分項）。
7. 最後以非專業術語寫給病人的決策輔助說明。

## Step 6 Audit

1. **病人層面**：實際追蹤結果（使用者提供；沒有就標「預計」並列指標：過程、結果、安全、時程）。
2. **自我評估**：5A 每一步執行品質自評表（例：問題是否可回答、檢索是否完整、工具是否適當、證據是否整合、應用是否考量病人）。

## Step 7 結果頁面

產出單一 HTML 報告（PICO、背景知識、語法、PRISMA、逐題評讀表與評讀判定總表、截圖、GRADE 判定總表與 SoF 證據摘要表、建議、SDM），以 artifact 發布，然後停點 6。

## Step 8 競賽 PPT（.pptx）

先讀 pptx skill，以 pptxgenjs 建構；頁數不設上限，以完整為原則（約 30–45 頁，評讀逐題頁數另計）。

### 必要內容（依序）

| 項目 | 應包含的要素 |
|---|---|
| 封面 | 題目（含 P、I、C）、單位、報告者、日期、組別 |
| 臨床情境 | 病人基本資料、病史、用藥、臨床困境、病人提出的問題；病人歷程時間軸；四象限 |
| 背景知識 | 疾病機轉、流行病學、標準治療、爭議點（為什麼要問這個問題）；說明背景資料引用的 References；至少 2 頁（① 機轉與流行病學；② 標準治療與爭議點）。來源：UpToDate、DynaMed、BMJ Best Practice；指引 ESC、ACC/AHA、NICE；教科書 |
| 提出問題 Ask | PICO 表、問題類型、最適合的研究設計；指引佐證 PICO；OCEBM 表 |
| 關鍵字轉換 | 同義詞、MeSH／Emtree、截字（*）、布林邏輯（關鍵字表完整列出） |
| 搜尋證據 Acquire | 依證據金字塔由上往下搜；每個資料庫的檢索式、筆數、檢索日期、限制條件；資料庫選擇理由。投影片放精簡版檢索式；完整檢索語法（不可縮寫）放該頁備忘稿，不另設附錄頁 |
| 文獻篩選流程 | PRISMA 2020 流程圖（可編輯圖形）與排除理由 |
| 選文理由 | 候選文獻比較表（年份、證據等級、與 PICO 相符度） |
| 嚴格評讀 | 效度、重要性、適用性；工具分流說明；**評讀 CHECKLIST 必須逐題呈現並回答，每題都要列出**（格式見 3.3；每篇、每題一列，題數多就加頁）；**之後必放評讀判定總表（直式，每篇一欄、每 domain 一列＋整體判定）** |
| 納入文獻與 OCEBM 證據等級 | **順序固定：放在逐題評讀（含評讀判定總表）之後、GRADE 之前**，不放在嚴格評讀開頭。內容：每篇納入文獻（作者、年份、期刊、研究設計、研究數／受試者數、本案角色）＋標明分級系統 OCEBM 2011＋逐篇起始等級 → 升降階理由（可引用前面的評讀結果）→ 最終等級 |
| 結果呈現 | 主要與次要結局的效果量、CI、異質性；原文森林圖截圖；結果彙整表；NNT／NNH 表 |
| 證據確信度 GRADE | **GRADE 判定總表**（各結局 × 升降級因子）與 Evidence Profile；**GRADE SoF 證據摘要表必須呈現**（格式見 Step 4 第 5 點，每個結局一列） |
| 臨床應用 Apply | 族群相似性、利弊權衡、成本（CEA＋院內藥價）、病人價值偏好、指引建議、EtD 2×2、4E、PICO 鎖定 |
| 醫病共享決策 SDM | 說明選項、比較選項、了解偏好、做決定；SDM 實際結果；衛教單張 |
| 效果評估 Audit | ① 病人層面實際追蹤；② 5A 每步自我評估表 |
| 結論 | 一句話回答臨床問題、take-home message |
| 參考文獻 | 統一格式完整書目：**AMA Manual of Style 第 11 版**；依內文**首次出現順序**編號（1, 2, 3…），內文用上標數字（如 SGLT2i 降低心衰竭住院³ 或 ¹⁻³、¹˒⁴），同一文獻再次引用沿用原編號；作者 ≤ 6 位全列，> 6 位列前 3 位加 et al（姓＋名縮寫不加句點，如 Kanie T, Mizuno A, Takaoka Y, et al）；期刊名用 NLM 縮寫並斜體；格式：作者. 篇名. *期刊縮寫*. 年;卷(期):起訖頁. doi:10.xxxx；線上資料庫主題（如 UpToDate）：作者. 主題名稱. UpToDate. 更新日期. 檢索日期. 網址；書目資料以 PubMed 核對 |

**PPT 不放附錄頁**：簡報到參考文獻結束。原本附錄的內容（如完整檢索語法）改放對應頁的備忘稿。

### NCMEA 評分對應（附件 1-7，實證新人組及臨床組，總分 102）

每頁**標題正上方**放一行評分標籤，只寫「題號＋評分項目」，不加「評分對應｜」字樣（例：`PICO 1-1 清楚描述病人及疾病`）。對應的評分項目如下：
- PICO 的質與量：1-1 病人及疾病、1-2 介入／診斷／暴露、1-3 結果的依據
- 文獻搜尋：2-1 關鍵字、2-2 資料庫理由與檢索策略、2-3 檢索功能、2-4 挑選歷程與排除理由、能否回答臨床提問
- 嚴格評讀：3-1 文獻的介入、3-2 評讀工具、3-3 效度、3-4 結果的重要性、3-5 整合與評定證據品質
- 臨床應用：4-1 能否應用、4-2 風險與利益、4-3 病人觀點、4-4 成本效益、4-5 由證據到建議
- 現場表現：5-1 系統分明、5-2 圖表清晰
- 加分：呈現手法創新（SDM 衛教單張等）

### 版面規範

- LAYOUT_WIDE；莫蘭迪色系（例：slate 4F6672、sage 7E9C93、dusty blue 8A9DB3、dusty rose C59D93、sand CDBE9E、mauve 9B8BA6、deep rose A8696F），白底，醫療科技風。
- 字型：中文 Microsoft JhengHei、英文 Calibri（theme 字型 latin=Calibri；建檔後把 theme 的 ea typeface 改成 Microsoft JhengHei）。
- 頁首配置：**頁碼放右上角**（最外側）；5A 進度條（Ask／Acquire／Appraise／Apply／Audit，目前步驟高亮）放在頁碼左側；評分標籤放在標題正上方。頁尾不放頁碼也不放評分標籤。
- **字級**（依此比例放大，版面也跟著放大，不可為了塞內容縮字）：
  - 頁標題 30–32 pt；封面主標 40–44 pt；章節分隔頁 36–40 pt。
  - 卡片標題、小標 18–20 pt。
  - **內文、條列、表格內容最小 14 pt**；重點數字 40–60 pt。
  - 評分標籤、頁碼、5A 進度條、圖說、來源註記 12–14 pt。
  - 放不下時拆成兩頁或把細節移到備忘稿，不得低於上述下限。
- 圖片用原文截圖並附說明（圖號、PDF 頁／印刷頁）；橫式頁面保持原方向，不旋轉。
- 待補欄位用斜體、灰藍色標「待補」。
- 每頁備忘稿寫資料來源與計算過程。
- 驗收：validate.py 通過；轉 PDF → 圖片逐頁檢查溢字、重疊、邊界。

## 附錄 A：build_boolean.py（移植自 pico.tsx）

```python
import json, re, sys
def _list(v):
    if isinstance(v, list): return [x for x in v if x]
    return [v] if v else []
def strip_dq(m): return re.sub(r'^"|"$', "", m)
def build(kt):
    if not isinstance(kt, list): return {"pubmed": "", "embase": "", "cochrane": ""}
    pm_b, em_b, co_b = [], [], []
    for row in kt:
        element = (row.get("element") or "").upper()
        mesh, emtree = _list(row.get("mesh")), _list(row.get("emtree"))
        if isinstance(row.get("synonyms"), list): syn = [s for s in row["synonyms"] if s]
        elif row.get("keywords"): syn = [s.strip() for s in re.split(r"[,;]", row["keywords"]) if s.strip()]
        else: syn = []
        drugs = [d for d in row.get("drugs") or [] if d] if isinstance(row.get("drugs"), list) else []
        combined = " ".join(mesh + syn).lower()
        if element.startswith("C") and ("not required" in combined or "not applicable" in combined): continue
        seen, terms = set(), []
        for t in syn + drugs:
            if t not in seen: seen.add(t); terms.append(t)
        terms = [t for t in terms if "not clearly stated" not in t.lower() and "not required" not in t.lower()]
        clean = [re.sub(r"\*+$", "", t).strip() for t in terms]
        pm = ['"' + strip_dq(m) + '"[Mesh]' for m in mesh if "not " not in m.lower()]
        pm += [f"{t}[tiab]" for t in clean] + [f"{t}[all fields]" for t in clean]
        if pm: pm_b.append("(\n  " + "\n  ".join(l if i == 0 else f"OR {l}" for i, l in enumerate(pm)) + "\n)")
        em = ["'" + re.sub(r"^'|'/exp$|/exp$", "", e).lower() + "'/exp" for e in emtree if "not " not in e.lower()]
        em += [f"{t}:ti,ab,kw" for t in clean]
        if em: em_b.append("(\n  " + "\n  ".join(l if i == 0 else f"OR {l}" for i, l in enumerate(em)) + "\n)")
        co = ['[mh "' + strip_dq(m) + '"]' for m in mesh if "not " not in m.lower()]
        co += [f'"{t}"' if " " in t else t for t in clean]
        if co: co_b.append("(\n  " + "\n  ".join(l if i == 0 else f"OR {l}" for i, l in enumerate(co)) + "\n)")
    return {"pubmed": "\nAND\n".join(pm_b), "embase": "\nAND\n".join(em_b), "cochrane": "\nAND\n".join(co_b)}
if __name__ == "__main__":
    data = json.load(open(sys.argv[1], encoding="utf-8"))
    for p in data["picos"]:
        st = build(p.get("keywordsTable"))
        for db in ("pubmed", "embase", "cochrane"):
            print(f"===== {p['id']} | {db} ====="); print(st[db] + "\n")
```

## 附錄 B：grade_certainty.py（移植自 grade_ai.tsx calculateCertainty）

```python
import json, sys
DOWN = ["riskOfBias", "inconsistency", "indirectness", "imprecision", "publicationBias"]
def certainty(j):
    s = 4
    for f in DOWN:
        r = j[f]["rating"]
        if r == "serious": s -= 1
        elif r == "very-serious": s -= 2
    r = j["largeEffect"]["rating"]
    if r == "large": s += 1
    elif r == "very-large": s += 2
    if j["doseResponse"]["rating"] == "yes": s += 1
    if j["residualConfounding"]["rating"] == "yes": s += 1
    s = max(1, min(4, s))
    return {4: "High", 3: "Moderate", 2: "Low", 1: "Very Low"}[s]
```

## 附錄 C：nnt.py

```python
import math, sys
cer, eer = float(sys.argv[1]), float(sys.argv[2]); d = abs(cer - eer)
if d == 0: print("NNT/NNH cannot be calculated"); sys.exit()
rr = eer / cer
print(f"RR={rr:.4f} RRR={1-rr:.4f}")
if eer < cer: print(f"ARR={d:.6f} NNT={math.ceil(1/d)}")   # 療效從寬：進位
else: print(f"ARI={d:.6f} NNH={math.floor(1/d)}")          # 安全從嚴：捨去
```

## 附錄 D：pdf_tools.py（pip install --break-system-packages pymupdf）

```python
import json, os, re, sys
import pymupdf as fitz
FOREST = re.compile(r"forest plot|favou?rs|risk ratio|odds ratio|hazard ratio|mean difference|heterogeneity|I2|I²|tau|weight|M-H|IV, random|IV, fixed|test for overall effect", re.I)
def printed_page(page):
    h = page.rect.height; cands = []
    for b in page.get_text("blocks"):
        x0, y0, x1, y1, t = b[:5]; t = t.strip()
        if y1 < h * 0.08 or y0 > h * 0.88:
            for line in t.splitlines():
                if re.fullmatch(r"[A-Za-z]?\d{1,5}", line.strip()): cands.append(line.strip()); break
        m = re.search(r"(?:^|\s)(?:p\.|page)\s*(\d{1,5})\b", t, re.I) if (y1 < h*0.08 or y0 > h*0.92) else None
        if m: cands.append(m.group(1))
    return cands[0] if cands else "?"
def text(pdf, out):
    doc = fitz.open(pdf)
    with open(out, "w", encoding="utf-8") as f:
        for i, p in enumerate(doc, 1): f.write(f"[PDF_PAGE_{i} | printed p.{printed_page(p)}]\n{p.get_text()}\n")
def figures(pdf):
    doc, res = fitz.open(pdf), []
    for i, p in enumerate(doc, 1):
        t = p.get_text(); hits = len(FOREST.findall(t))
        caps = re.findall(r"(?:Figure|Fig\.)\s*\d+[A-Za-z]?[.:][^\n]{0,120}", t)
        n_img = len({r.get("xref") for r in p.get_image_info(xrefs=True) if r.get("xref")}); n_draw = len(p.get_drawings())
        if hits >= 3 or caps or n_img:
            res.append({"pdf_page": i, "printed_page": printed_page(p), "forest_keyword_hits": hits, "captions": caps[:4], "embedded_images": n_img, "vector_paths": n_draw, "likely_forest_plot": hits >= 3 and (n_draw > 40 or n_img > 0)})
    print(json.dumps(res, ensure_ascii=False, indent=2))
def clip(pdf, pno, outdir, frac):
    os.makedirs(outdir, exist_ok=True); doc = fitz.open(pdf); p = doc[pno - 1]; W, H = p.rect.width, p.rect.height
    x0, y0, x1, y1 = [float(v) for v in frac.split(",")]
    out = os.path.join(outdir, f"p{pno}_clip_{x0:.2f}_{y0:.2f}.png")
    p.get_pixmap(dpi=300, clip=fitz.Rect(x0*W, y0*H, x1*W, y1*H)).save(out); print(out)
def capture(pdf, pno, outdir):
    os.makedirs(outdir, exist_ok=True); doc = fitz.open(pdf); p = doc[pno - 1]; stem = f"p{pno}"; saved = []
    fp = os.path.join(outdir, f"{stem}_fullpage.png"); p.get_pixmap(dpi=200).save(fp); saved.append(fp)
    shown = []
    for r in p.get_image_info(xrefs=True):          # only images drawn on THIS page (skips shared logos elsewhere)
        if r.get("xref") and r["xref"] not in shown: shown.append(r["xref"])
    for k, xref in enumerate(shown, 1):
        info = doc.extract_image(xref)
        if info["width"] < 150 or info["height"] < 100: continue
        ip = os.path.join(outdir, f"{stem}_img{k}_original.{info['ext']}"); open(ip, "wb").write(info["image"]); saved.append(ip)
    rects = [fitz.Rect(d["rect"]) for d in p.get_drawings()] + [fitz.Rect(r["bbox"]) for r in p.get_image_info()]
    rects = [fitz.Rect(r.x0 - 1, r.y0 - 1, r.x1 + 1, r.y1 + 1) for r in rects if r.width > 2 or r.height > 2]
    if rects:
        box = fitz.Rect(rects[0])
        for r in rects[1:]: box |= r
        for b in p.get_text("blocks"):
            br = fitz.Rect(b[:4])
            if br.intersects(box + (-20, -70, 20, 60)): box |= br
        box = (box + (-6, -6, 6, 6)) & p.rect
        if box.width * box.height < p.rect.width * p.rect.height * 0.92:
            cp = os.path.join(outdir, f"{stem}_figure_crop.png"); p.get_pixmap(dpi=300, clip=box).save(cp); saved.append(cp)
    print(json.dumps({"pdf_page": pno, "printed_page": printed_page(p), "files": saved}, ensure_ascii=False, indent=2))
if __name__ == "__main__":
    c = sys.argv[1]
    if c == "text": text(sys.argv[2], sys.argv[3])
    elif c == "figures": figures(sys.argv[2])
    elif c == "capture": capture(sys.argv[2], int(sys.argv[3]), sys.argv[4])
    elif c == "clip": clip(sys.argv[2], int(sys.argv[3]), sys.argv[4], sys.argv[5])
```

# 工具邏輯附錄 T1–T6（自使用者 TSX 逐字擷取，僅去除介面程式碼）

以下程式碼區塊內容一律逐字照用，不得改寫。`${...}` 為原工具的變數，執行時代入實際內容。附錄 A、B 的 Python 版本為 T1-2、T6-1 的等價移植；兩者不一致時以本附錄為準。

## T1 PICO 工具（pico.tsx）

臨床情境 → PICO 拆解、關鍵字表、三層 Boolean 語法。

T1-1 分析提示詞（輸入＝臨床情境原文；temperature 0.1）

```text
    const systemPrompt = `你是一位實證醫學 (EBM) 與醫學資訊檢索專家。請分析使用者提供的臨床情境。

任務要求：
1. 判斷問題類型 (Therapy, Diagnosis, Prognosis, Etiology/Harm, Prevention, Screening, Qualitative)。
2. 若情境包含多個決策問題，必須拆成 PICO-1, PICO-2... 等獨立區塊輸出，不可合併。
3. 依照 PICO 架構拆解。若資訊不足，請標示 "Not clearly stated; inferred as..."。
4. 針對不同類型使用專屬問句模板：
   - Therapy: 「在 [P] 中，[I] 相較於 [C]，是否能改善 / 降低 / 增加 [O]？」
   - Diagnosis: 「在 [P] 中，[I] 相較於 [C]，是否具有較佳的 diagnostic accuracy、sensitivity、specificity？」
   - Prognosis: 「在 [P] 中，[I/prognostic factor] 是否與 [O] 相關？」

5. 【藥理學同類藥物全展開規則】：
   - 若 PICO 元素涉及藥物大類（例如 SGLT2i、DPP-4i、GLP-1 RA、Statin、ARNI、DOAC 等），**必須**將該藥物大類所有已上市的「單方藥品學名 (Single-Ingredient generic names)」完整列出在 \`drugs\` 陣列中！
   - 範例（SGLT2i）：\`["Empagliflozin", "Dapagliflozin", "Canagliflozin", "Ertugliflozin", "Bexagliflozin", "Sotagliflozin", "Ipragliflozin", "Tofogliflozin", "Luseogliflozin"]\`。
   - 若該元素非藥物大類，則 \`drugs\` 設為空陣列 \`[]\`。

6. 【控制詞彙真實性校驗】：
   - \`mesh\` 陣列中的詞彙必須是 NCBI MeSH Database 真實存在的官方詞彙。
   - \`emtree\` 陣列中的詞彙必須是 Embase Emtree 真實存在的官方詞彙（全部小寫，無連字號）。
   - \`synonyms\` 陣列請列出該概念的自由文字關鍵字、同義詞與縮寫（不含藥物單方學名，藥物學名請放進 \`drugs\`）。

輸出請嚴格遵守 JSON 格式（不要 Markdown 標記）：
{
  "picos": [
    {
      "id": "PICO-1",
      "type": "Therapy",
      "analysis": {
        "p": { "content": "...", "explanation": "..." },
        "i": { "content": "...", "explanation": "..." },
        "c": { "content": "...", "explanation": "..." },
        "o": { "content": "...", "explanation": "..." }
      },
      "question": "...",
      "keywordsTable": [
        { 
          "element": "P", 
          "mesh": ["Diabetes Mellitus, Type 2"], 
          "emtree": ["non insulin dependent diabetes mellitus"], 
          "synonyms": ["type 2 diabetes", "T2DM", "non-insulin-dependent diabetes"],
          "drugs": []
        },
        { 
          "element": "I", 
          "mesh": ["Sodium-Glucose Transporter 2 Inhibitors"], 
          "emtree": ["sodium glucose cotransporter 2 inhibitor"], 
          "synonyms": ["SGLT2 inhibitor", "SGLT2i", "gliflozin"],
          "drugs": ["Empagliflozin", "Dapagliflozin", "Canagliflozin", "Ertugliflozin", "Bexagliflozin", "Sotagliflozin", "Ipragliflozin", "Tofogliflozin", "Luseogliflozin"]
        }
      ],
      "studyDesign": "..."
    }
  ]
}`;
```

T1-2 buildBooleanStrategies（逐字 JS，可存成 .js 以 node 執行）

```javascript
// 程式化自動組裝 Boolean 搜尋策略：確保第 3 部分的控制詞彙、同義詞與同類藥物學名 100% 完整同步至第 4 部分，絕無遺漏！
const buildBooleanStrategies = (keywordsTable) => {
  if (!keywordsTable || !Array.isArray(keywordsTable)) {
    return { pubmed: '', embase: '', cochrane: '' };
  }

  const pubmedBlocks = [];
  const embaseBlocks = [];
  const cochraneBlocks = [];

  keywordsTable.forEach(row => {
    const element = (row.element || '').toUpperCase();
    const meshList = Array.isArray(row.mesh) ? row.mesh.filter(Boolean) : (row.mesh ? [row.mesh] : []);
    const emtreeList = Array.isArray(row.emtree) ? row.emtree.filter(Boolean) : (row.emtree ? [row.emtree] : []);
    const synList = Array.isArray(row.synonyms) ? row.synonyms.filter(Boolean) : (row.keywords ? row.keywords.split(/[,;]/).map(s => s.trim()).filter(Boolean) : []);
    const drugList = Array.isArray(row.drugs) ? row.drugs.filter(Boolean) : [];

    // 若為 Prognosis 的 C 元素且標示 Not required，則跳過 C 區塊
    const combinedCheck = [...meshList, ...synList].join(' ').toLowerCase();
    if (element.startsWith('C') && (combinedCheck.includes('not required') || combinedCheck.includes('not applicable'))) {
      return;
    }

    // 合併所有自由關鍵字與完整藥物單方學名（去重）
    const allTextTerms = Array.from(new Set([...synList, ...drugList])).filter(
      t => !t.toLowerCase().includes('not clearly stated') && !t.toLowerCase().includes('not required')
    );

    // 1. 組裝 PubMed 區塊 (MeSH -> [tiab] -> [all fields])
    const pmLines = [];
    meshList.forEach(m => {
      if (!m.toLowerCase().includes('not ')) {
        pmLines.push(`"${m.replace(/^"|"$/g, '')}"[Mesh]`);
      }
    });
    allTextTerms.forEach(term => {
      const cleanTerm = term.replace(/\*+$/g, '').trim();
      pmLines.push(`${cleanTerm}[tiab]`);
    });
    allTextTerms.forEach(term => {
      const cleanTerm = term.replace(/\*+$/g, '').trim();
      pmLines.push(`${cleanTerm}[all fields]`);
    });
    if (pmLines.length > 0) {
      pubmedBlocks.push(`(\n  ${pmLines.map((l, i) => (i === 0 ? l : `OR ${l}`)).join('\n  ')}\n)`);
    }

    // 2. 組裝 Embase 區塊 ('emtree'/exp -> :ti,ab,kw)
    const emLines = [];
    emtreeList.forEach(e => {
      if (!e.toLowerCase().includes('not ')) {
        const cleanEmtree = e.replace(/^'|'\/exp$|\/exp$/g, '').toLowerCase();
        emLines.push(`'${cleanEmtree}'/exp`);
      }
    });
    allTextTerms.forEach(term => {
      const cleanTerm = term.replace(/\*+$/g, '').trim();
      emLines.push(`${cleanTerm}:ti,ab,kw`);
    });
    if (emLines.length > 0) {
      embaseBlocks.push(`(\n  ${emLines.map((l, i) => (i === 0 ? l : `OR ${l}`)).join('\n  ')}\n)`);
    }

    // 3. 組裝 Cochrane 區塊 ([mh "MeSH"] -> "keyword")
    const coLines = [];
    meshList.forEach(m => {
      if (!m.toLowerCase().includes('not ')) {
        coLines.push(`[mh "${m.replace(/^"|"$/g, '')}"]`);
      }
    });
    allTextTerms.forEach(term => {
      const cleanTerm = term.replace(/\*+$/g, '').trim();
      const formatted = cleanTerm.includes(' ') ? `"${cleanTerm}"` : cleanTerm;
      coLines.push(formatted);
    });
    if (coLines.length > 0) {
      cochraneBlocks.push(`(\n  ${coLines.map((l, i) => (i === 0 ? l : `OR ${l}`)).join('\n  ')}\n)`);
    }
  });

  return {
    pubmed: pubmedBlocks.join('\nAND\n'),
    embase: embaseBlocks.join('\nAND\n'),
    cochrane: cochraneBlocks.join('\nAND\n')
  };
};
```

T1-3 Embase.com 平台語法說明（輸出 Embase 語法時一併附上）

```text
⚑ Embase.com 平台語法說明：Embase.com 介面的有效欄位代碼為 :ti、:ab、:kw（可合併為 :ti,ab,kw），並無「all fields」對應代碼，故不使用 :mp（該代碼僅存在於 Ovid Embase，寫法為句點語法 .mp. 而非冒號）。若您使用 Ovid 平台，請將本頁 :ti,ab,kw 改為 .ti,ab,kw.，並可視需要另加 .mp. 做多欄位搜尋。
```

T1-4 資料庫連結

```text
MeSH 驗證：https://www.ncbi.nlm.nih.gov/mesh/
Emtree 驗證：https://www.embase.com/emtree
Cochrane MeSH 驗證：https://www.cochranelibrary.com/advanced-search/mesh
PubMed Advanced：https://pubmed.ncbi.nlm.nih.gov/advanced/
Embase Search：https://www.embase.com/search/quick
Cochrane Search：https://www.cochranelibrary.com/advanced-search/search-manager
```

## T2 CEBM RCT 評讀（cebm_RCT.tsx）

原工具讀取 PDF 前 12 頁文字（[PDF_PAGE_i] 標記）；本 Skill 以 pdf_tools 取全文。先執行 RCT 預檢：is_rct 為 false 時停止並告知「非此研究設計文獻：經預檢分析，此文獻不符合隨機對照試驗 (RCT) 研究設計」。

T2-1 評讀提示詞

```text
const systemPrompt = `
  You are an Evidence-Based Medicine (EBM) expert. Your role is to critically appraise RCTs using the Oxford CEBM framework.

  CRITICAL PRE-CHECK:
  - Analyze the Title, Abstract, and Methods. 
  - Determine if this study is an RCT. If it is NOT an RCT, return ONLY: {"is_rct": false}.

  APPRAISAL RULES (If is_rct: true):
  1. TERMINOLOGY: Strictly use "AppraisalSheets_RCT_zh-tw-2023" terminology.
  2. STRUCTURE: 1:1 correspondence with the checklist.
  3. BILINGUAL CONTENT: 
     - Left: Verbatim English quotes. 
     - Right: Precise 2023 Trad. Chinese Medical translation.
     - Citation: Must include "Section, Paragraph, and Printed Page Number".

  SECTION REQUIREMENTS:
  - Step 1 Internal Validity: Include items 1a, 1b, 2a, 2b, 3.
  - Step 2 Results Part 2 (Precision): Must explain point estimates and 95% CI using the specific phrasing: "我們不知道整體族群發生結果的真實風險為何... 透過審視點估計值的信賴區間 (CI)...".
  - Step 3 Applicability: Must address the 3 core questions regarding patient difference, feasibility, and benefit vs harm.

  JSON STRUCTURE:
  {
    "is_rct": true,
    "pico": [{"label_en": "Patients", "label_ch": "病人 P", "evidence_en": "...", "evidence_ch": "...", "source": "..."}],
    "validity": {"items": [{"id": "1a", "q_en": "...", "q_ch": "...", "judgment": "...", "evidence_en": "...", "evidence_ch": "...", "source": "..."}]},
    "results": {
      "data": {"rr": {"val": "...", "formula": "..."}, "arr": {"val": "...", "formula": "..."}, "rrr": {"val": "...", "formula": "..."}, "nnt": {"val": "...", "formula": "..."}},
      "precision_en": "...", "precision_ch": "...", "source": "..."
    },
    "applicability": {"items": [{"q_en": "...", "q_ch": "...", "judgment": "...", "evidence_en": "...", "evidence_ch": "...", "source": "..."}]},
    "ocebm": {"initial_level": "...", "final_level": "...", "summary": "...", "downgrade": [{"item": "...", "status": "...", "detail": "..."}], "upgrade": []}
  }
`;
```

## T3 CASP SR/MA-RCT（casp_sr_ma_rct.tsx）

先執行 SR/MA 預檢（temperature 0.0）：is_srma 為 false 時停止並告知偵測到的研究設計與理由（「本系統僅適用於 Systematic Review / Meta-Analysis (SR/MA) 文獻評讀」）。通過後執行評讀提示詞，20 個評讀組件全數依序輸出（含 appraisal_summary 與 summary）。

T3-1 SR/MA 預檢提示詞

```text
const checkPrompt = `請快速判斷這份 PDF 文獻是否為 Systematic Review 或 Meta-Analysis (SR/MA) 研究文章。
請回傳以下 JSON 格式：
{
  "is_srma": true 或 false,
  "detected_type": "偵測到的實際研究設計類型 (例如：Single RCT, Cohort Study, Narrative Review, Case Report, Guideline, Systematic Review and Meta-Analysis 等)",
  "reason": "簡短繁體中文說明判斷依據"
}`;
```

T3-2 評讀提示詞

```text
  const systemPrompt = `你是一位資深實證醫學教授。請閱讀附件 PDF，嚴格按照 CASP Checklist: For systematic reviews with meta-analysis of randomised controlled trials (RCTs) (2024) 與 CASP_SR_MA_RCT_2024_bilingual 進行雙語評讀。

【核心強化規則】：
1. 文獻原始句盡可能詳細擷取 (Verbatim Excerpt)：
   - 在 e_en 中，必須優先以雙引號 "..." 完整摘錄文獻中的原始連續英文句子或數據段落（愈詳細、愈完整愈好，不可過度摘要省略），隨後再附上英文判定分析。
   - 在 e_zh 中，必須針對左側擷取的英文原文提供精確、流暢的繁體中文醫學翻譯，並附上完整的中文判定說明。
2. 交代清楚 CONSIDER 判斷原因 (Explicit Rationale based on CONSIDER)：
   - 每一題都必須嚴格對照該題 CONSIDER 條列項目與【判斷建議】的每一項條件（例如：P/I/C/O/T/S 要素是否齊全、資料庫數量與檢索詞、雙人獨立篩選與第三人仲裁、PRISMA 數據、RoB 工具、異質性 I² 與模型選擇、敏感度分析、發表偏誤、交互作用檢定等）。
   - 必須在 e_en 與 e_zh 中明確寫出「【CONSIDER 判斷依據 / Rationale】」，逐條交代哪些條件有達成、哪些未達成或未報告，並說明為何最終判定為 Yes、No 或 Can't tell（絕不自行假設研究者做了未報告的步驟）。
3. 具體章節、段落與頁碼標註：
   - citation 欄位必須精確標註文獻的具體章節名稱、子標題、段落順序（如第幾段）以及原始印刷頁碼（Journal Page Number，例如：Methods - Search Strategy and Selection Criteria, Paragraph 2, Page 3）。
4. 數據計算 (Impact of Effect)：搜尋文獻中的 Event Rates，若數據充足，請在 Q8 詳列 Event Rate (Exp vs Ctrl)、ARR/ARI 以及 NNT (1/ARR 無條件進位) 與 NNH (1/ARI 無條件捨去) 的計算過程與結果。

　20 個評讀組件完整清單（必須全數依序輸出，100% 保留雙語原文標題與題目）】：
1. Section A:
   - title: "Section A: Is the basic study design valid for a systematic review? / Section A：study design 對 systematic review 而言是否有效？"
   - Q1:
     * id: "1"
     * q: "1. Did the systematic review address a clearly formulated research question? / 1. systematic review 是否提出明確的 research question？"
     * consider: "Did the researchers state a research question and a null hypothesis? For a systematic review of RCTs, a research question can be 'formulated' in terms of the PICOT(S) framework:\n• Population\n• Intervention\n• Comparator\n• Outcome/s and Outcome measures\n• Time, e.g., study timeframe, or follow-up intervals\n• Setting\n\n研究者是否陳述 research question 及 null hypothesis？RCT 的 systematic review 可用 PICOT(S) 架構「形成」research question：\n• P：Population（族群）\n• I：Intervention（介入）\n• C：Comparator（對照）\n• O：Outcome/s and Outcome measures（結果與結果測量）\n• T：Time, e.g., study timeframe, or follow-up intervals（時間，如研究期間或追蹤間隔）\n• S：Setting（場域）\n\n【判斷建議】：Yes：P、I、C、O 皆明確，最好還有 T 和 S；No：問題模糊，或缺少 I／C／O；Can't tell：只有籠統的研究目的敘述。"
   - Q2:
     * id: "2"
     * q: "2. Did the researchers search for appropriate study design(s) to answer the research question? / 2. 研究者是否搜尋適當的 study design 來回答 research question？"
     * consider: "If the research question is concerned with the efficacy of an intervention, the RCT is the appropriate study design for a systematic review. The most common type of RCT is the parallel RCT in which individuals are randomised to study groups; other methods of randomisation, however, could be relevant depending on the research question.\n\n若 research question 關注 intervention 的療效，RCT 是 systematic review 的適當 study design。最常見的是平行 RCT（個體隨機分派至各組）；依 research question，其他隨機方式也可能適用。\n\n【判斷建議】：Yes：問療效，而且納入的是 RCT（cluster、crossover 等依問題可接受）；No：問療效卻混入觀察性研究，又沒有分開分析；Can't tell：沒寫清楚納入哪些設計。"

2. Section B:
   - title: "Section B: Is the systematic review methodologically sound? / Section B：systematic review 的方法學是否嚴謹？"
   - Q3 (主項總結):
     * id: "3"
     * q: "3. Were all the relevant primary research studies likely to have been included in the systematic review? / 3. 相關的 primary research studies 是否可能都已納入 systematic review？"
     * consider: "綜合評估 3a (Searching)、3b (Screening)、3c (Selecting) 與 3d (Summarising) 之執行嚴謹度。\n\n【判斷建議】：a–d 大多為 Yes 才給 Yes；任一項嚴重不足（如只搜單一資料庫）可能漏掉研究，進而改變合併 effect size。"
   - Q3a:
     * id: "3a"
     * q: "a) Searching for primary research studies / a) 搜尋 primary research studies"
     * consider: "• Was the search strategy comprehensive and clearly reported?\n• Did the search include 1 or more of the major bibliographic databases, e.g., MEDLINE/PubMed, and Embase?\n• Did the researchers provide MESH terms for MEDLINE, or their equivalent for other databases?\n• Were relevant subject-specific bibliographic databases searched?\n• Did the search include non-English language studies?\n• Did the researchers undertake citation searching, including hand-searching of reference lists from primary research studies included in the systematic review?\n• Did the search include unpublished studies? For instance, did the search include registers of ongoing trials or preprint repositories?\n• Did the researchers consult experts in the field about potential primary research studies or ongoing trials that could be included?\n\n• search strategy 是否完整且清楚報告？\n• 是否搜尋 1 個以上 major bibliographic databases，如 MEDLINE/PubMed、Embase？\n• 是否提供 MEDLINE 的 MESH terms 或其他資料庫的對應詞彙？\n• 是否搜尋相關的 subject-specific bibliographic databases？\n• 是否納入 non-English language studies？\n• 是否進行 citation searching，包括 hand-searching 已納入研究的參考文獻？\n• 是否納入 unpublished studies，如 registers of ongoing trials 或 preprint repositories？\n• 是否諮詢領域專家可能納入的 primary research studies 或 ongoing trials？\n\n【判斷建議】：Yes：≥2 個 major bibliographic databases＋search strategy 完整可重現＋有 unpublished studies 或 registers of ongoing trials；No：只用單一資料庫，或限定英文且未說明理由；Can't tell：只列資料庫名稱，未附 search strategy。"
   - Q3b:
     * id: "3b"
     * q: "b) Screening primary research studies from the search / b) 篩選搜尋到的 primary research studies"
     * consider: "• Did the researchers define appropriate eligibility or inclusion and exclusion criteria?\n• Did the researchers design and implement a robust process to screen the primary research studies? For instance, two researchers working independently with a third independent researcher to resolve any disagreements.\n• Was screening based on the title and abstract of primary research studies found during the search?\n• Did the researchers adhere to the eligibility criteria?\n\n• 是否定義適當的 inclusion and exclusion criteria？\n• 是否設計並執行穩健的篩選流程？例如兩位研究者獨立篩選，由第三位獨立研究者解決分歧。\n• 是否依搜尋所得研究的標題與摘要篩選？\n• 是否遵守 eligibility criteria？\n\n【判斷建議】：Yes：條件明確，且兩人獨立篩選＋第三人仲裁；Can't tell：未說明篩選人數或流程。"
   - Q3c:
     * id: "3c"
     * q: "c) Selecting primary research studies to include in the systematic review / c) 選擇納入 systematic review 的 primary research studies"
     * consider: "• Did the researchers design and implement a robust process to select the primary research studies according to the eligibility criteria? For instance, two researchers working independently with a third independent researcher to resolve any disagreements.\n• Were decisions to include or exclude primary research studies based on full-text analysis?\n• Did the researchers adhere to the eligibility criteria?\n• Was the level of agreement between the researchers responsible for selecting the primary research studies calculated and reported? For instance, by calculating the kappa statistic of inter-rate reliability.\n\n• 是否設計並執行穩健流程，依 eligibility criteria 選擇 primary research studies？例如兩位研究者獨立進行，由第三位解決分歧。\n• 是否依全文分析決定納入或排除？\n• 是否遵守 eligibility criteria？\n• 是否計算並報告選文者之間的一致性？例如 kappa statistic of inter-rate reliability。\n\n【判斷建議】：Yes：全文由兩人獨立判讀＋有 reasons for exclusion；kappa 屬加分項，非必要條件。"
   - Q3d:
     * id: "3d"
     * q: "d) Summarising the search and its outputs / d) 彙整搜尋過程及其結果"
     * consider: "Did the researchers present a PRISMA-type flow diagram, including the numbers of primary research studies that were:\n• Duplicates?\n• Screened out?\n• Excluded, with the reasons for exclusion?\n• Included in the systematic review?\n• Included in the meta-analysis (data may not have been complete in some of the primary research studies)?\n\n是否呈現 PRISMA-type flow diagram，並列出以下 primary research studies 數量：\n• Duplicates?\n• 篩除 Screened out?\n• 排除（附 reasons for exclusion）Excluded, with the reasons for exclusion?\n• 納入 systematic review Included in the systematic review?\n• 納入 meta-analysis（部分研究資料可能不完整）Included in the meta-analysis (data may not have been complete in some of the primary research studies)?\n\n【判斷建議】：Yes：flow diagram 各階段數字完整，且 reasons for exclusion 明確。"
   - Q4:
     * id: "4"
     * q: "4. Did the researchers assess the validity or methodological rigour of the primary research studies included in the systematic review? / 4. 研究者是否評估納入 primary research studies 的 validity or methodological rigour？"
     * consider: "Lack of methodological rigour in the individual primary research studies can affect the validity and interpretation of the findings of the systematic review with meta-analysis.\n• Did the researchers use a validated tool to assess the methodological rigour of the primary research studies included in the systematic review?\n• Was the tool appropriate to assess the type(s) of study design(s) included in the systematic review? For example, the Cochrane Risk of Bias tool specifically for RCTs or the McMaster EPHPP tool for any quantitative study design, including RCTs.\n• Did the researchers present the findings from their quality assessment, and interpret them accurately?\n\n個別 primary research studies 缺乏 methodological rigour，會影響 systematic review with meta-analysis 結果的 validity 與解讀。\n• 是否使用 validated tool 評估 primary research studies 的 methodological rigour？\n• 工具是否適用於所納入的 study design？例如專用於 RCT 的 Cochrane Risk of Bias tool，或適用各種量性設計（含 RCT）的 McMaster EPHPP tool。\n• 是否呈現 quality assessment 結果並正確解讀？\n\n【判斷建議】：Yes：使用 Cochrane Risk of Bias tool，逐篇或逐 domain 呈現，並在討論中說明對結果的影響；No：未做 quality assessment；Can't tell：宣稱有做但未呈現結果。"
   - Q5 (主項總結):
     * id: "5"
     * q: "5. Did the researchers extract, and present information from the individual primary research studies appropriately and transparently? / 5. 研究者是否恰當且透明地 extract 並呈現各 primary research studies 的資訊？"
     * consider: "綜合評估 5(a) 資料擷取與 5(b) 資料呈現之透明度與完整性。"
   - Q5a:
     * id: "5a"
     * q: "(a) Extraction of data"
     * consider: "• Did the researchers design and implement a robust process for the extraction of data from the individual primary research studies?\n• Did the researchers follow guidance on data extraction?\n• Did the researchers use a standardised form or software programme to record the data to ensure completeness and accuracy?\n• Did the researchers extract the relevant data for the study-level characteristics and the results of each primary research study?\n\n• 是否設計並執行穩健的 process for the extraction of data？\n• 是否依循 guidance on data extraction？\n• 是否使用 standardised form or software programme 記錄資料，以確保完整與正確？\n• 是否 extract 各研究的 study-level characteristics 與結果等相關資料？\n\n【判斷建議】：Yes：兩人獨立 extract＋使用 standardised form。"
   - Q5b:
     * id: "5b"
     * q: "(b) Presentation of data"
     * consider: "• Did the researchers present the key characteristics of the individual primary research studies, e.g., in a table? For instance, the number of participants, the profile of participants (age, sex), the intervention, the comparator, the outcome/s evaluated, and the study timeframe.\n• Did the researchers present the results of the individual primary research studies in a Forest plot or combination of table and Forest plot? For instance, the effect size/s, the confidence-interval ranges, and the P values. NB: The Forest plot should also show the overall result from the meta-analysis.\n\n• 是否以表格等方式呈現各 primary research studies 的關鍵特徵？例如受試者人數、特徵（年齡、性別）、intervention、comparator、評估的結果與研究期間。\n• 是否以 Forest plot 或表格加 Forest plot 呈現各研究結果？例如 effect size、confidence interval 與 P values。註：Forest plot 也應顯示 meta-analysis 的整體結果。\n\n【判斷建議】：Yes：研究特徵表與 Forest plot 都有，且 Forest plot 含 meta-analysis 的 overall result。"

3. Section C：
   - title: "Section C: Are the results of the systematic review trustworthy? / Section C：systematic review 的結果是否可信？"
   - Q6:
     * id: "6"
     * q: "6. Did the researchers analyse the pooled results of the individual primary research studies appropriately? / 6. 研究者是否恰當地分析各 primary research studies 的 pooled results？"
     * consider: "• Did the researchers undertake a power calculation and sample-size estimation during the design and planning of the systematic review?\n• Did the number of participants whose outcomes were entered into the analysis meet the power calculation, i.e., was the meta-analysis sufficiently powered to detect any effect on the outcomes of interest?\n• Did the researchers use an appropriate effect measure?\n• Did the researchers provide confidence-interval ranges for the effect estimates in the systematic review?\n• Did the researchers provide p values for the effect estimates in the systematic review?\n• Did the researchers provide a minimal important difference, that is the smallest possible difference that a person would perceive as a beneficial effect of intervention?\n• Did the researchers assess the level of statistical heterogeneity (variability) among the primary research studies? For example, using the I² statistic.\n• Did the researchers use an appropriate model of meta-analysis for the level of heterogeneity among the primary research studies (a random-effects model if there was heterogeneity or a fixed-effects model if the primary research studies were all investigating the same underlying effect)?\n• Did the researchers perform any sensitivity analyses?\n• Did the researchers analyse the reasons for heterogeneity using subgroup analysis or meta-regression? For subgroup analysis, see Question 6.1, and for meta-regression see Question 6.2.\n• Did the researchers investigate the small-study effect, and assess the potential for publication bias in the systematic review (e.g., using a funnel plot)?\n\n• 設計規劃階段是否進行 power calculation and sample-size estimation？\n• 納入分析的受試者人數是否達到 power 要求，亦即 meta-analysis 是否有足夠 power 偵測關注結果的效果？\n• 是否使用適當的 effect measure？\n• 是否提供 effect estimate 的 confidence interval？\n• 是否提供 effect estimate 的 p values？\n• 是否提供 minimal important difference，即個人會認為 intervention 有益的最小差異？\n• 是否評估 primary research studies 間的 statistical heterogeneity，例如使用 I² statistic？\n• 是否依 heterogeneity 程度選擇適當的 model of meta-analysis？有 heterogeneity 時用 random-effects model；primary research studies 皆探討同一個真實效果時用 fixed-effects model。\n• 是否進行 sensitivity analyses？\n• 是否以 subgroup analysis 或 meta-regression 分析 heterogeneity 的原因？subgroup analysis 見 6.1，meta-regression 見 6.2。\n• 是否探討 small-study effect，並評估 publication bias 的可能性（如使用 funnel plot）？\n\n【判斷建議】：Yes：effect measure 恰當＋有 confidence interval＋有 I²＋模型選擇合理＋有 sensitivity analyses＋有評估 publication bias（實務上很少做 power calculation，缺此項通常不直接判 No，但記為 limitations；Cochrane 建議 ≥10 篇研究才適合做 funnel plot）。"
   - Q6.1:
     * id: "6.1"
     * q: "6.1 Subgroup analysis"
     * consider: "Were the characteristics or effect modifiers for investigation:\n• Specified in the study protocol, with the direction of effect, and statistical tests to be used?\n• Clearly defined, with a rationale for selection?\n• Not closely related to other characteristics, i.e., differentiation is possible?\n• Analysed in relation to the primary outcome?\nIf continuous data were allocated to categories, were the thresholds or cut-off points specified in the study protocol together with a rationale?\nIf a large number of characteristics or effect modifiers were investigated, or subgroup analyses conducted, did the researchers adjust for multiple testing?\nWas a test for interaction undertaken to determine whether any subgroup effects were statistically significant?\nWas the analysis of effect modification based on comparison within rather than between studies?\n\n所探討的 characteristics or effect modifiers 是否：\n• 在 study protocol 中事先設定，並註明預期效果方向及所用統計檢定？\n• 定義清楚，並有選擇理由？\n• 與其他特徵不過度相關，亦即可以區分？\n• 針對主要結果分析？\n連續資料若切分為類別，thresholds or cut-off points 是否在計畫書中事先設定並附理由？\n若探討大量 characteristics or effect modifiers，或進行多個 subgroup analysis，是否校正 multiple testing？\n是否進行 test for interaction，以判斷 subgroup effects 是否 statistically significant？\nanalysis of effect modification 是基於研究「內」而非研究「間」的比較嗎？\n\n【判斷建議】：Yes：事先設定＋有 test for interaction＋數量有限或有校正（事後才做、只看各 subgroup p values，或僅有研究間比較：可信度低；若本研究未執行 subgroup analysis 請註明並判 Can't tell 或依實況說明）。"
   - Q6.2:
     * id: "6.2"
     * q: "6.2 Meta-regression"
     * consider: "Were the characteristics or effect modifiers for investigation:\n• Specified in the study protocol, with the direction of effect?\n• Continuous data? If continuous data were allocated to categories, were the thresholds or cut-off points specified in the study protocol with a rationale for selection?\nIf a large number of characteristics or effect modifiers were investigated, or meta-regression analyses performed, did the researchers adjust for multiple testing?\nWas a test for interaction undertaken to determine whether any effects were statistically significant?\nWas a random-effects model used for the meta-regression analyses?\nWas the analysis of effect modification based on comparison within rather than between studies?\n\n所探討的 characteristics or effect modifiers 是否：\n• 在 study protocol 中事先設定，並註明預期效果方向？\n• 連續資料：若切分為類別，thresholds or cut-off points 是否在計畫書中事先設定並附選擇理由？\n若探討大量 characteristics or effect modifiers，或進行多個 meta-regression 分析，是否校正 multiple testing？\n是否進行 test for interaction，以判斷效果是否 statistically significant？\nMeta-regression 是否使用 random-effects model？\nanalysis of effect modification 是基於研究「內」而非研究「間」的比較嗎？\n\n【判斷建議】：同 6.1；另需確認使用 random-effects meta-regression，且研究數量足夠（若本研究未執行 meta-regression 請註明並判 Can't tell 或依實況說明）。"
   - Q7:
     * id: "7"
     * q: "7. Did the researchers report any limitations of the systematic review and, if so, do the limitations discussed cover all the issues you identified during critical appraisal? / 7. 研究者是否報告 systematic review 的 limitations？若有，討論的 limitations 是否涵蓋你在 critical appraisal 中發現的所有問題？"
     * consider: "• Did the researchers discuss whether the meta-analysis was sufficiently powered to detect an effect of intervention?\n• Did the researchers consider the appropriateness of the effect measure or measures they used?\n• Did the researchers reflect on the precision of the effect estimate, i.e., the confidence-interval range? The smaller the range, the narrower the confidence intervals, meaning the result is more precise, and closer to the true effect size.\n• If relevant, did the researchers note whether the confidence-interval range included the \"line of no effect\" (0 for a difference, 1 for a ratio, where the null hypothesis holds true), or whether the lower limit of the confidence-interval range was close to the \"line of no effect\", and discuss the implications for the results of the meta-analysis?\n• If the results were statistically significant (i.e., they were less likely to be due to chance), did the researchers discuss whether the results would be important or meaningful for the outcomes experienced by individuals and/or populations using a minimal important difference specific to the research question? Did the researchers consider whether relevant primary research studies could have been missed?\n• Did the researchers mention any systematic bias identified during the risk-of-bias/quality assessment of the primary research studies, and explain how it might have influenced the effect estimate in the meta-analysis?\n• Did the researchers mention any potential sources of confounding that could have influenced the effect estimate in the meta-analysis?\n• Did the researchers discuss the implications of any sensitivity analyses?\n• Did the researchers discuss the impact of the level of heterogeneity on the results of the meta-analysis?\n• Did the researchers investigate the reasons for any heterogeneity across the primary research studies and discuss the implications? For subgroup analysis, see Question 7.1, and for meta-regression, see Question 7.2.\n• Did the researchers discuss the effect of any publication bias on the results of the meta-analysis?\n\n• 是否討論 meta-analysis 有無足夠 power 偵測 intervention 效果？\n• 是否考量所用 effect measure 的適當性？\n• 是否反思 effect estimate 的 precision（confidence interval 範圍）？\n• 若相關，是否指出 confidence interval 是否包含 \"line of no effect\"，或下限是否接近，並討論意涵？\n• 若結果 statistically significant，是否依 minimal important difference 討論臨床意義？是否考慮可能遺漏相關 primary research studies？\n• 是否提及 risk-of-bias/quality assessment 中發現的 systematic bias，並說明如何影響 effect estimate？\n• 是否提及潛在 sources of confounding？\n• 是否討論 sensitivity analyses 的意涵？\n• 是否討論 heterogeneity 程度對結果的影響？\n• 是否探討 heterogeneity 的原因並討論意涵（subgroup analysis 見 7.1，meta-regression 見 7.2）？\n• 是否討論 publication bias 對結果的影響？\n\n【判斷建議】：Q6 看分析有沒有「做」，Q7 看作者有沒有「討論」。Yes：作者討論的 limitations 涵蓋你在 Q3–6 找到的主要問題；No：你找到重要缺陷（如高 risk of bias、高 heterogeneity），作者卻沒提。"
   - Q7.1:
     * id: "7.1"
     * q: "7.1 Subgroup-analysis"
     * consider: "• If characteristics or effect modifiers were not pre-specified, did the researchers address whether bias was introduced into the analysis?\n• Did the researchers reflect on whether the characteristics or effect modifiers selected were well-defined to ensure clarity about the effect being investigated?\n• If no rationale was given for the selection of specific characteristics or effect modifiers, or the rationale was not supported by evidence or a plausible argument of meaningfulness, did the researchers discuss whether this affected the validity or relevance of the subgroup analysis?\n• If characteristics or effect modifiers were closely related to other characteristics, did the researchers mention the potential for confounding?\n• Did the researchers outline whether the subgroup analyses were sufficiently powered to detect an effect on the primary outcome?\n• If continuous data were allocated to categories, did the researchers address whether the thresholds or cut-off points could have introduced bias into the subgroup analysis or were not meaningful either clinically or in terms of public or population health? If more than three characteristics or effect modifiers were investigated, or subgroup analyses performed, did the researchers adjust for multiple testing and consider the potential to generate Type I errors?\n• Did the researchers explain the results of any tests for interaction and whether they were statistically significant?\n• Did the researchers discuss the implications of whether the results of tests for interaction were quantitative or qualitative?\n• If the analysis of effect modification was based on a comparison between studies, did the researchers reflect on whether the number of studies in the smallest subgroups was large enough for the results to be credible?\n\n• 若未事先設定，是否討論是否引入 bias？\n• 是否反思所選因子定義是否明確？\n• 若缺乏選擇理由或證據支持，是否討論是否影響 validity 或相關性？\n• 若因子與其他特徵高度相關，是否提及 potential for confounding？\n• 是否說明 subgroup analysis 有無足夠 power？\n• 若連續資料切分為類別，是否討論 thresholds or cut-off points 可能引入 bias 或不具意義？探討超過三個因子或多個分析時，是否校正 multiple testing 並考慮 Type I errors？\n• 是否解釋 test for interaction 的結果及是否 statistically significant？\n• 是否討論 interaction 屬於 quantitative 或 qualitative 差異及其意涵？\n• 若基於研究間比較，是否反思 smallest subgroups 研究數是否足以使結果可信？"
   - Q7.2:
     * id: "7.2"
     * q: "7.2 Meta-regression"
     * consider: "• If characteristics or effect modifiers were not pre-specified, did the researchers address whether bias was introduced into the analysis?\n• If continuous data were allocated into categories, did the researchers address whether any thresholds or cut-off points for categorisation were arbitrary and could have introduced bias into the meta-regression or were not meaningful clinically or in terms of public or population health?\n• If more than three characteristics or effect modifiers were investigated, or meta-regression analyses performed, did the researchers adjust for multiple testing and consider the potential to generate Type I errors?\n• Did the researchers discuss the implications of any tests for interaction and whether they were statistically significant?\n• If a random-effects model was not used to account for residual heterogeneity and/or mixed effects, which would have allowed for both within-study and between-study variation, did the researchers outline the implications for the results?\n• If the analysis of effect modification was based on a between-study comparison, did the researchers reflect on whether the number of primary research studies in the meta-regression was sufficient for the results to be credible?\n\n• 若未事先設定，是否討論是否引入 bias？\n• 若連續資料切分為類別，是否討論 thresholds or cut-off points 是否任意、引入 bias 或不具臨床意義？\n• 若探討超過三個因子或多個分析，是否校正 multiple testing 並考慮 Type I errors？\n• 是否討論 test for interaction 的意涵及是否 statistically significant？\n• 若未使用 random-effects model 處理 residual heterogeneity 及／或混合效應，是否說明對結果的影響？\n• 若基於研究間比較，是否反思納入 meta-regression 的研究數是否足以使結果可信？"
   - Q8:
     * id: "8"
     * q: "8. Would the benefits of intervention outweigh any potential disadvantages, harms and/or additional demand for resources associated with acting on the results? / 8. intervention 的 benefits 是否大於依結果行動可能帶來的 disadvantages、harms 及／或額外 demand for resources？"
     * consider: "• Are you clear about the likely benefits of the intervention, bearing in mind the potential impacts of any study limitations?\n• Did the researchers identify any potential disadvantages or harms associated with the intervention?\n• If so, did the researchers assess any benefits of the intervention against the disadvantages or harms, and discuss the overall balance between benefit and harm?\n• Did the researchers report any information on the potential demand for resources (e.g. cost, workforce, time, skills level/skills mix, training needs, data collection and analysis, IT requirements) that might be associated with acting on the results of the systematic review?\n\n• 考量研究 limitations 的可能影響後，你是否清楚 intervention 可能帶來的 benefits？\n• 研究者是否找出 intervention 可能的 disadvantages 或 harms？\n• 若有，是否權衡 intervention 的 benefits 與 disadvantages 或 harms，並討論整體利弊平衡？\n• 是否報告依結果行動可能衍生的 demand for resources（例如成本、人力、時間、skills level/skills mix、training needs、資料收集與分析、IT requirements）？\n\n【判斷建議】：Yes：有報告不良事件，並權衡淨效益；No／Can't tell：只報有效性，沒有安全性或資源資料。"

4. Section D：
   - title: "Section D: Are the results of the systematic review relevant locally? / Section D：systematic review 的結果在本地是否適用？"
   - Q9:
     * id: "9"
     * q: "9. Can the results of the systematic review be applied to your local population/in your local setting or context? / 9. systematic review 的結果能否應用於你的 local population／local setting 或情境？"
     * consider: "• Are there differences between your local population and the participants in the primary research studies in the systematic review that would influence whether you would act upon the results?\n• Are there differences between your local setting and the settings or contexts in the primary research studies in the systematic review that would influence whether you would act upon the results?\n• Are there any outcomes or other factors that the researchers could have studied that would have been useful to you bearing in mind the needs of your local population and/or setting?\n\n• 你的 local population 與 primary research studies 受試者之間，是否有會影響你是否依結果行動的差異？\n• 你的 local setting 與 primary research studies 的 setting 或情境之間，是否有會影響你是否依結果行動的差異？\n• 考量 local population 及／或 setting 的需求，是否有研究者原本可以研究、且對你有用的結果或其他因素？\n\n【判斷建議】：屬評讀者依本地情境的判斷，可逐項比對：populations 特徵（年齡、共病、種族）、醫療體系與給付、藥品可近性、劑型劑量、結果指標是否符合本地需求。"

5. Section E：
   - title: "Section E: Will the implementation of the results represent greater value for your service users or population? / Section E：執行結果是否能為你的服務對象或 populations 帶來更大 value？"
   - Q10:
     * id: "10"
     * q: "10. If actioned, would the findings from the systematic review represent greater or additional value for the individuals or populations for whom you are responsible? / 10. 若付諸行動，systematic review 的結果能否為你負責的個人或 populations 帶來更大或額外的 value？"
     * consider: "Value equals the Outcome/s (Benefit minus Harm) divided by the Resources required for implementation.\n• What resources would be needed to take action on the findings of the systematic review? Take account of various types of resource, not only costs, but also time, skills mix, skills development or training needs, IT requirements, and other material resources.\n• If necessary, are you able to disinvest resources from other activities to be able to re-invest in actioning the findings from the systematic review?\n\nvalue ＝ Outcome/s（Benefit − Harm）÷ Resources required for implementation。\n• 依 systematic review 結果採取行動需要哪些資源？除成本外，也要考量時間、skills mix、skills development 或 training needs、IT requirements 及其他物資。\n• 必要時，能否從其他活動 disinvest resources，再投入執行本研究結果？"

請產出如下 JSON：
{
  "title": "文獻標題 (Bilingual)",
  "sections": [
    { 
      "title": "上述指定的 100% 雙語原文 Section Title", 
      "questions": [
        { 
          "id": "題號 (1, 2, 3, 3a, 3b, 3c, 3d, 4, 5, 5a, 5b, 6, 6.1, 6.2, 7, 7.1, 7.2, 8, 9, 10)", 
          "q": "上述指定的 100% 雙語原文題目或子標題", 
          "consider": "上述指定的 100% 雙語原文 CONSIDER 與判斷建議", 
          "a": "Yes / No / Can't tell", 
          "e_en": "【Verbatim Excerpt】盡可能完整詳細之文獻英文原文段落摘錄 + 【CONSIDER Rationale】逐條對應 CONSIDER 條目之英文判定理由分析 (含 Q8 之 NNT/NNH 計算)",
          "e_zh": "【文獻原文翻譯】對照左側英文原文之精確繁體中文醫學翻譯 + 【CONSIDER 判斷依據】逐條交代為何符合或不符合 CONSIDER 條件之中文詳細說明 (含 Q8 之 NNT/NNH 計算)",
          "citation": "具體章節名稱、子標題、段落位置與文獻內文原始印刷頁碼 (例如：Methods - Study Selection, Paragraph 2, Page 4)"
        }
      ]
    }
  ],
  "appraisal_summary": {
    "positive": "優點／方法學嚴謹 (Positive/Methodologically sound) 描述",
    "negative": "缺點／方法學較弱 (Negative/Relatively poor methodology) 描述",
    "unknowns": "未知 (Unknowns) 描述",
    "overall_judgement": "整體風險判定與結論 (Conclusion: What is your conclusion about the systematic review – can it be used to support evidence-based decision-making?)"
  },
  "summary": "臨床應用決策建議總結 (Bilingual)"
}`;
```

## T4 CASP SR/MA-observational（casp_SR_MA_OBSERVATION.tsx）

18 題（題目清單見提示詞）；temperature 0.1。輸出含逐題、Section notes、appraisal_summary（positive／negative／unknowns）、clinical_summary、final_conclusion，全部呈現。

T4-1 評讀提示詞

```text
const systemPrompt = `
  🧠 ROLE: You are an expert in medical evidence-based medicine (EBM), clinical epidemiology, and biostatistics.
  🎯 TASK: Analyze the article strictly following the CASP Checklist: For systematic reviews with meta-analysis of observational studies (2024).

  📌 MANDATORY WORDING FOR SECTIONS, ITEMS & SUB-ITEMS (100% EXACT VERBATIM, NO ABBREVIATIONS OR ALTERATIONS):

  Section A:
  - title_en: "Section A: Is the basic study design valid for a systematic review?"
  - title_tw: "Section A：study design 對 systematic review 而言是否有效？"
  - notes_en: "Notes to support interpretation of Section A, Questions 1 and 2: If you answered 'No' to both these questions: It is likely that the researchers did not clearly formulate the fundamental aspects of the research question, and the most appropriate way of answering it. If this is the case, it is likely other problems will arise during the conduct of the systematic review. Consider whether it would be useful to continue with the critical appraisal process."
  - notes_tw: "Section A 第 1、2 題解讀說明：若兩題皆答「No」：研究者很可能沒有清楚形成 research question 的基本要素，也沒有選擇最適當的回答方式；若是如此，systematic review 執行過程很可能還會出現其他問題。考慮是否值得繼續 critical appraisal。"
  - Questions in Section A:
    1. item_id: "1"
       q_en: "1. Did the systematic review address a clearly formulated research question?"
       q_tw: "1. systematic review 是否提出明確的 research question？"
    2. item_id: "2"
       q_en: "2. Did the researchers search for appropriate study design(s) to answer the research question?"
       q_tw: "2. 研究者是否搜尋適當的 study design(s) 來回答 research question？"

  Section B:
  - title_en: "Section B: Is the systematic review methodologically sound?"
  - title_tw: "Section B：systematic review 的方法學是否嚴謹？"
  - notes_en: "Notes to support interpretation of Section B, Questions 3-5: If you answered 'No' to these questions, it is likely that there is a lack of methodological rigour in the conduct of the systematic review, which means it is best to interpret the results with caution, and to assess how those aspects of poor methodology will have an impact on the results of the systematic review. For Question 3, a 'No' response indicates that this systematic review may have missed primary research studies that could have contributed to answering the research question. For Question 4, a 'No' response indicates that the researchers did not identify any systematic bias or confounding factors in the primary research studies that could have affected the results. For Question 5, a 'No' response indicates that the researchers did not organise the data from the primary research studies in a coherent way such that it could analysed appropriately. If you answered 'No' to all three questions in Section B, consider whether it would be useful to continue with the critical appraisal process."
  - notes_tw: "Section B 第 3–5 題解讀說明：若這些題目答「No」，systematic review 的執行很可能缺乏 methodological rigour，結果宜審慎解讀，並評估這些方法缺陷會如何影響結果。第 3 題答「No」：可能漏掉有助回答 research question 的 primary research studies；在 meta-analysis 中，遺漏研究的結果可能改變 effect estimate。第 4 題答「No」：研究者未找出 primary research studies 中可能影響結果的 systematic bias 或 confounding factors。第 5 題答「No」：研究者未有條理地整理 primary research studies 資料。若 Section B 三題皆答「No」，考慮是否值得繼續 critical appraisal。"
  - Questions in Section B (Include parent question context in sub-items so nothing is omitted):
    3a. item_id: "3a"
        q_en: "3. Were all the relevant primary research studies likely to have been included in the systematic review? — a) Searching for primary research studies"
        q_tw: "3. 相關的 primary research studies 是否可能都已納入 systematic review？ — a) 搜尋 primary research studies"
    3b. item_id: "3b"
        q_en: "3. Were all the relevant primary research studies likely to have been included in the systematic review? — b) Screening primary research studies from the search"
        q_tw: "3. 相關的 primary research studies 是否可能都已納入 systematic review？ — b) 篩選搜尋到的 primary research studies"
    3c. item_id: "3c"
        q_en: "3. Were all the relevant primary research studies likely to have been included in the systematic review? — c) Selecting primary research studies to include in the systematic review"
        q_tw: "3. 相關的 primary research studies 是否可能都已納入 systematic review？ — c) 選擇納入 systematic review 的 primary research studies"
    3d. item_id: "3d"
        q_en: "3. Were all the relevant primary research studies likely to have been included in the systematic review? — d) Summarising the search and its outputs"
        q_tw: "3. 相關的 primary research studies 是否可能都已納入 systematic review？ — d) 彙整搜尋過程及其結果"
    4.  item_id: "4"
        q_en: "4. Did the researchers assess the validity or methodological rigour of the primary research studies included in the systematic review?"
        q_tw: "4. 研究者是否評估納入 primary research studies 的 validity or methodological rigour？"
    5a. item_id: "5a"
        q_en: "5. Did the researchers extract, and present information from the individual primary research studies appropriately and transparently? — (a) Extraction of data"
        q_tw: "5. 研究者是否恰當且透明地 extract 並呈現各 primary research studies 的資訊？ — (a) Extraction of data"
    5b. item_id: "5b"
        q_en: "5. Did the researchers extract, and present information from the individual primary research studies appropriately and transparently? — (b) Presentation of data"
        q_tw: "5. 研究者是否恰當且透明地 extract 並呈現各 primary research studies 的資訊？ — (b) Presentation of data"

  Section C:
  - title_en: "Section C: Are the results of the systematic review trustworthy?"
  - title_tw: "Section C：systematic review 的結果是否可信？"
  - notes_en: "Notes to support interpretation of Section C, Questions 6, 7 & 8: If you answered 'No' to these questions, it is likely that the researchers did not analyse and interpret the information from the primary research studies appropriately, nor did they discuss the limitations of the systematic review as fully as possible so it is not possible for you to assess the trustworthiness (validity and credibility) of the results of the systematic review. Finally, if there is no information on the likely resource demands of intervention, it is not possible for you to judge whether you have the resource capacity to act upon the results. If you answered 'No' to all three questions in Section C, consider whether it would be useful to continue with the critical appraisal process."
  - notes_tw: "Section C 第 6、7、8 題解讀說明：若這些題目答「No」，研究者很可能沒有恰當分析與解讀 primary research studies 的資訊，也未充分討論 systematic review 的 limitations，因此無法判斷結果的 trustworthiness（validity and credibility）。此外，若缺乏 intervention 可能需要的 resource demands 資訊，便無法判斷你是否有 resource capacity 依結果行動。若 Section C 三題皆答「No」，考慮是否值得繼續 critical appraisal。"
  - Questions in Section C:
    6.   item_id: "6"
         q_en: "6. Did the researchers analyse the pooled results of the individual primary research studies appropriately?"
         q_tw: "6. 研究者是否恰當地分析各 primary research studies 的 pooled results？"
    6.1. item_id: "6.1"
         q_en: "6.1 Subgroup analysis"
         q_tw: "6.1 Subgroup analysis"
    6.2. item_id: "6.2"
         q_en: "6.2 Meta-regression"
         q_tw: "6.2 Meta-regression"
    7.   item_id: "7"
         q_en: "7. Did the researchers report any limitations of the systematic review and, if so, do the limitations discussed cover all the issues you have identified during critical appraisal?"
         q_tw: "7. 研究者是否報告 systematic review 的 limitations？若有，討論的 limitations 是否涵蓋你在 critical appraisal 中發現的所有問題？"
    7.1. item_id: "7.1"
         q_en: "7.1 Subgroup analysis"
         q_tw: "7.1 Subgroup analysis"
    7.2. item_id: "7.2"
         q_en: "7.2 Meta-regression"
         q_tw: "7.2 Meta-regression"
    8.   item_id: "8"
         q_en: "8. Would the benefits of acting upon the results outweigh any potential disadvantages, harms and/or additional demand for resources associated with acting on the results?"
         q_tw: "8. 依結果採取行動的 benefits，是否大於可能的 disadvantages、harms 及／或額外的 demand for resources？"

  Section D:
  - title_en: "Section D: Are the results of the systematic review relevant locally?"
  - title_tw: "Section D：systematic review 的結果在本地是否適用？"
  - notes_en: "Notes to support interpretation of Section D, Question 9: If you answered 'No' to this question, it is not necessary to answer Question 10 because, irrespective of a systematic review's methodological rigour, the results are not applicable to the individuals or populations for whom you are responsible. If you answered 'Yes' to Question 9, answer Question 10."
  - notes_tw: "Section D 第 9 題解讀說明：若答「No」，不需回答第 10 題，因為無論 systematic review 的 methodological rigour 如何，結果都不適用於你負責的 individuals or populations。若答「Yes」，請回答第 10 題。"
  - Questions in Section D:
    9. item_id: "9"
       q_en: "9. Can the results of the systematic review be applied to your local population/in your local setting or context?"
       q_tw: "9. systematic review 的結果能否應用於你的 local population／local setting or context？"

  Section E:
  - title_en: "Section E: Will the implementation of the results represent greater value for your service users or population?"
  - title_tw: "Section E：執行結果是否能為你的 service users or population 帶來更大 value？"
  - notes_en: "Notes to support interpretation of Section E, Question 10: If you answered 'No' to this question, it is likely that the findings of the systematic review will not confer greater or additional benefit or value on the individuals and/or populations for whom you are responsible, despite the systematic review's applicability to your local setting. If you answered 'Yes' to the question, it is likely that the findings of the systematic review will confer greater or additional benefit or value on the individuals and/or populations for whom you are responsible, and you need to discuss with colleagues whether it would be appropriate to implement the findings in your local setting."
  - notes_tw: "Section E 第 10 題解讀說明：若答「No」：儘管 systematic review 適用於你的 local setting，其 findings 可能仍無法為你負責的 individuals and/or populations 帶來更大或額外的 benefit or value。若答「Yes」：findings 可能帶來更大或額外的 benefit or value，需與同事討論是否適合在 local setting 實施。"
  - Questions in Section E:
    10. item_id: "10"
        q_en: "10. If actioned, would the findings from the systematic review represent greater or additional value for the individuals or populations for whom you are responsible?"
        q_tw: "10. 若付諸行動，systematic review 的 findings 能否為你負責的 individuals or populations 帶來更大或額外的 value？"

  📌 OPTIMIZED CONSIDER & JUDGMENT CRITERIA (FROM CASP_SR_MA_Observational_2024_bilingual):
  For EVERY question, populate "consider_en" and "consider_tw" thoroughly with the exact CASP 2024 CONSIDER prompts AND the specific appraisal judgment guidelines (判斷建議):
  - Never make assumptions: If not explicitly reported, select "Can't tell".
  - Item 1: Check research question, null hypothesis, and PECOT(S) framework (P: Population, E: Exposure/Risk factor, Detection of beneficial/harmful effect, C: Comparator/Controls, O: Outcome/s or Event/s, T: Time, S: Setting). Yes requires P, E, C, O clear (preferably T & S).
  - Item 2: Check if observational study designs (Cohort, Case-control, Cross-sectional) match the research question (e.g., cohort/case-control for risk factors/exposures; cross-sectional for prevalence; avoid cross-sectional for causal temporality or unseparated mixed designs).
  - Item 3a: Comprehensive search strategy, >=2 major databases (MEDLINE/PubMed, Embase), MESH terms/equivalents, subject-specific databases, non-English studies, citation/hand-searching, unpublished studies (registers/preprints), expert consultation. Note higher publication bias risk in observational studies due to lack of prospective registration.
  - Item 3b: Clear eligibility/inclusion-exclusion criteria (acceptable study designs & exposure definitions), robust screening (two independent researchers + third resolver) on title/abstract.
  - Item 3c: Robust selection (two independent researchers + third resolver) via full-text analysis, adherence to eligibility criteria, inter-rater agreement (kappa statistic).
  - Item 3d: PRISMA-type flowchart showing numbers for duplicates, screened out, excluded with reasons, included in SR, included in MA.
  - Item 4: Validated tool appropriate for observational designs (Newcastle-Ottawa Scale or ROBINS-E tool), detailed presentation (per study or domain: confounding, selection, exposure/outcome measurement, missing data) & accurate interpretation.
  - Item 5a: Robust data extraction (two independent extractors), guidance followed, standardised form/software, study-level characteristics & results extracted (verify adjusted vs. unadjusted effect estimates and adjusted confounders).
  - Item 5b: Study characteristics table (participants count, age, sex, intervention/exposure, comparator, outcomes, study timeframe, design, confounders) + Forest plot showing individual study effect sizes, CIs, P values, and overall meta-analysis result.
  - Item 6: Sample-size estimation, appropriate effect measure (check if OR/RR/HR or adjusted/unadjusted are improperly mixed), CIs, p values, minimal important difference (MID), statistical heterogeneity (I²), appropriate model (random-effects vs. fixed-effects), sensitivity analyses, subgroup/meta-regression for heterogeneity, small-study-effect / publication bias (funnel plot, ideally >=10 studies).
  - Item 6.1: Pre-specified effect modifiers in protocol with direction & statistical tests, clear definitions & rationale, non-collinearity, primary outcome focus, pre-specified continuous cut-offs, multiple testing adjustment, test for interaction, within-study rather than between-study comparison.
  - Item 6.2: Pre-specified continuous/categorical modifiers with cut-off rationale, multiple testing adjustment, test for interaction, random-effects model used, within-study comparison.
  - Item 7: Limitations discussed covering all issues found in Q3-Q6: sample size adequacy, effect measure appropriateness, precision (CI width & line of no effect 0 or 1), clinical importance vs. MID, missed primary studies, systematic bias from RoB/quality assessment, potential/residual confounding & association vs. causation, sensitivity analysis implications, heterogeneity impact & causes, publication bias impact.
  - Item 7.1: Subgroup limitations: non-pre-specified bias, well-defined modifiers, rationale validity, confounding between correlated modifiers, subgroup sample size adequacy, arbitrary continuous cut-offs bias, multiple testing (>3 modifiers) Type I error, quantitative vs. qualitative interaction tests, smallest subgroup study count in between-study comparisons.
  - Item 7.2: Meta-regression limitations: non-pre-specified bias, arbitrary cut-offs bias, multiple testing (>3 modifiers) Type I error, interaction test implications, lack of random-effects/mixed-effects model impact, study count adequacy in between-study comparisons.
  - Item 8: Likely benefits bearing in mind limitations vs. disadvantages/unwanted outcomes/harms, overall benefit-harm balance, and demand for resources (cost, workforce, time, skills mix, training, data/IT).
  - Item 9: Local population differences (age, comorbidities, ethnicity, exposure prevalence), local setting/context differences, missing locally relevant outcomes.
  - Item 10: Value = Outcome/s (Benefit - Harm) / Resources required (costs, time, skills mix, training, IT, material resources), feasibility of disinvesting/re-investing resources.

  📌 AUTOMATIC NNT/NNH CALCULATION (MANDATORY FOR ITEM 8):
  For Section C Item 8, you MUST search for Event Rates (CER - Control Group Event Rate and EER - Experimental Group Event Rate) in the article.
  If numerical data is found, perform the following in the "evidence_zh" section:
  - Formula: ARR = |CER − EER|
  - Formula: NNT = 1 / ARR
  - Formula: NNH = 1 / ARR
  - Rounding Rule: NNT MUST be rounded UP (進位) to the nearest integer (療效從寬).
  - Rounding Rule: NNH MUST be rounded DOWN (捨去/無條件捨去) to the nearest integer (安全從嚴).
  - Content: Show the full formula, the substituted values from the article, and the final integer result.
  If calculation is impossible, state: “NNT/NNH cannot be calculated from available data.”

  📌 RATIONALE & EVIDENCE (VERBATIM EXCERPT & ACCURATE MEDICAL TRANSLATION):
  - evidence_en: Extract continuous, complete, verbatim English sentences directly from the article (do NOT paraphrase or truncate).
  - evidence_zh: Provide an accurate Traditional Chinese (Taiwan medical terminology) medical translation of the verbatim excerpt, followed by a clear rationale explaining how it supports the answer (Yes / No / Can't tell).
  - location: Specify exact Section name, Paragraph number/description, and Printed Page number ("section", "paragraph", "printed_page").

  📌 OUTPUT JSON STRUCTURE:
  {
    "paper_metadata": { "title": "...", "authors": "..." },
    "sections": [
      {
        "id": "A",
        "title_en": "...",
        "title_tw": "...",
        "questions": [
          {
            "item_id": "1",
            "q_en": "...",
            "q_tw": "...",
            "a": "Yes / No / Can't tell",
            "consider_en": "Detailed CASP 2024 CONSIDER criteria...",
            "consider_tw": "極詳盡繁中 CONSIDER 判定標準與判斷建議...",
            "evidence_en": "Continuous verbatim English sentences from the article...",
            "evidence_zh": "精確的繁體中文醫學翻譯與證據理由說明 (Item 8 需含 NNT/NNH 計算)...",
            "location": { "section": "...", "paragraph": "...", "printed_page": "..." }
          }
        ],
        "notes_en": "...",
        "notes_tw": "..."
      }
    ],
    "appraisal_summary": { "positive": "...", "negative": "...", "unknowns": "..." },
    "clinical_summary": { "key_findings": "...", "implication": "...", "applicability": "...", "recommendation": "..." },
    "final_conclusion": "Summary sentence"
  }
`;
```

## T5 RoB NMA（rob_nma.tsx）

17 題逐題執行（每題一次，依 T5-2 提示詞）；judgment 代碼：T／PT／PF／F／NI／NA。domain 名稱：Domain 1 Interventions & Geometry（介入措施與網絡結構）、Domain 2 Effect Modifiers（效果修飾因子）、Domain 3 Statistical Synthesis（統計整合）。

T5-1 17 題題目

```javascript
const ROB_ITEMS = [
  { id: "1.1", domain: 1, en: "All interventions and their comparators included in the NMA are reasonable alternatives for the whole target population", zh: "所有納入 NMA 之介入措施及其比較對象，對整體目標族群而言皆為合理的替代方案" },
  { id: "1.2", domain: 1, en: "All eligible interventions were included in the network", zh: "所有符合條件之介入措施皆已納入網絡" },
  { id: "1.3", domain: 1, en: "Interventions were appropriately grouped into nodes in the network", zh: "介入措施在網絡中之分群（節點）方式是否適當" },
  { id: "1.4", domain: 1, en: "All compared interventions were connected through a suitable chain of within study comparisons", zh: "所有比較之介入是否透過研究內比較形成適當連結" },
  { id: "2.1", domain: 2, en: "Outcome definitions and timepoints were similar across direct comparisons in the network", zh: "網絡中不同直接比較之結果定義與評估時間點是否一致" },
  { id: "2.2", domain: 2, en: "Effect-modifying participant characteristics were similar across direct comparisons in the network", zh: "不同直接比較中之受試者效果修飾特徵是否相似" },
  { id: "2.3", domain: 2, en: "Effect-modifying study characteristics were similar across direct comparisons in the network", zh: "不同直接比較中之研究層級效果修飾特徵是否相似" },
  { id: "2.4", domain: 2, en: "The analysis appropriately addressed the differences in effect modifiers across the network", zh: "分析是否已適當處理效果修飾因子差異" },
  { id: "3.1", domain: 3, en: "The analysis respected within-study randomisation", zh: "分析是否尊重研究內隨機分派" },
  { id: "3.2", domain: 3, en: "No publication bias or other selective non-reporting biases were suspected", zh: "無發表偏差或選擇性報告偏差" },
  { id: "3.3", domain: 3, en: "All pre-defined analyses, and only those analyses, were reported, or discrepancies were explained", zh: "是否完整報告所有預先規劃之分析，或對差異提供合理解釋" },
  { id: "3.4", domain: 3, en: "Biases in primary studies were minimal or addressed in the synthesis", zh: "原始研究之偏差是否已最小化或於整合分析中加以處理" },
  { id: "3.5", domain: 3, en: "Appropriate methods were used to handle multi-arm studies", zh: "是否使用適當方法處理多臂試驗" },
  { id: "3.6", domain: 3, en: "Appropriate assumptions were made about homogeneity or heterogeneity of effects within comparisons", zh: "是否對比較內之同質性或異質性作出適當假設" },
  { id: "3.7", domain: 3, en: "There was no evidence of conflict between direct and indirect estimates of the same effect", zh: "直接證據與間接證據之間無不一致（inconsistency）" },
  { id: "3.8", domain: "3", en: "Conflicting results between direct and indirect evidence were adequately addressed", zh: "是否適當處理直接與間接證據之衝突" },
  { id: "3.9", domain: 3, en: "If a Bayesian analysis was performed, the choice of prior distributions was appropriate", zh: "若使用貝葉斯分析，其先驗分布是否適當" }
];
```

T5-2 逐題評讀提示詞

```text
const systemPrompt = `You are an expert Systematic Review and NMA Methodologist. Your task is to perform an automated appraisal using the RoB NMA tool.

CRITICAL RULE: 
1. Read the FULL TEXT of the provided article.
2. Extract EXACT evidence (Quotes) and provide precise medical translations in Traditional Chinese.
3. Make rigorous judgments based ONLY on article content and the RoB NMA judgment criteria.
4. Maintain a professional, reviewer-level tone.

FOR ITEM ${item.id}: ${item.en} (${item.zh})

RETURN JSON ONLY:
{
  "quoteEN": "The exact verbatim English quote from the article (including context).",
  "quoteZH": "精確的繁體中文醫學翻譯。",
  "page": "Article PRINTED page number (e.g., p.12) and section.",
  "rationaleEN": "Detailed rationale in English explaining the judgment.",
  "rationaleZH": "判斷理由的繁體中文翻譯（包含如何對應判定標準）。",
  "howToAssess": "How to assess this statement (Original English definitions).",
  "howToAssessZH": "如何評估此陳述（中文翻譯）。",
  "judgmentCriteria": "Judgment criteria (Original English definitions for the chosen judgment).",
  "judgmentCriteriaZH": "判斷標準（中文翻譯）。",
  "judgment": "T/PT/PF/F/NI/NA"
}`;
```

T5-3 domain／整體判定規則（getDomainReason 的 40 字截斷不採用，理由完整呈現）

```javascript
const getDomainRisk = (domainId) => {
  const items = results.filter(r => String(r.domain) === String(domainId));
  if (items.some(r => r.judgment === 'F')) return { label: 'High Risk', zh: '高偏差風險', color: 'text-red-600', bg: 'bg-red-50', icon: '🔴' };
  if (items.some(r => r.judgment === 'PF' || r.judgment === 'NI')) return { label: 'Some Concerns', zh: '有些疑慮', color: 'text-amber-600', bg: 'bg-amber-50', icon: '🟡' };
  return { label: 'Low Risk', zh: '低偏差風險', color: 'text-emerald-600', bg: 'bg-emerald-50', icon: '🟢' };
};

const getDomainReason = (domainId) => {
  const items = results.filter(r => String(r.domain) === String(domainId));
  // Find the item with the "worst" judgment to extract a key reason
  const order = { 'F': 5, 'PF': 4, 'NI': 3, 'PT': 2, 'T': 1, 'NA': 0 };
  const sorted = [...items].sort((a, b) => (order[b.judgment] || 0) - (order[a.judgment] || 0));
  const worst = sorted[0];
  if (!worst || worst.judgment === 'T') return "無明顯偏差疑慮";

  // Extract a short meaningful snippet from rationaleZH
  const reason = worst.rationaleZH || "";
  return reason.length > 40 ? reason.substring(0, 40) + "..." : reason;
};

const getOverallRisk = () => {
  const risks = [1, 2, 3].map(id => getDomainRisk(id).label);
  if (risks.includes('High Risk')) return { label: 'High Risk of Bias', zh: '高偏差風險', color: 'bg-red-600 text-white', icon: '🔴' };
  if (risks.includes('Some Concerns')) return { label: 'Some Concerns', zh: '有些疑慮', color: 'bg-amber-500 text-white', icon: '🟡' };
  return { label: 'Low Risk of Bias', zh: '低偏差風險', color: 'bg-emerald-600 text-white', icon: '🟢' };
};
```

## T6 GRADE-AI（grade_ai.tsx）

PICO 與結局萃取 → 各文獻 OCEBM 2011 等級 → 各結局 GRADE（2011＋2025 Core GRADE）→ SoF → 台灣在地建議與 SDM。

T6-1 calculateCertainty

```javascript
const calculateCertainty = (judgments) => {
  let score = 4; // High (4) -> Moderate (3) -> Low (2) -> Very Low (1)

  // Downgrades
  if (judgments.riskOfBias.rating === 'serious') score -= 1;
  else if (judgments.riskOfBias.rating === 'very-serious') score -= 2;

  if (judgments.inconsistency.rating === 'serious') score -= 1;
  else if (judgments.inconsistency.rating === 'very-serious') score -= 2;

  if (judgments.indirectness.rating === 'serious') score -= 1;
  else if (judgments.indirectness.rating === 'very-serious') score -= 2;

  if (judgments.imprecision.rating === 'serious') score -= 1;
  else if (judgments.imprecision.rating === 'very-serious') score -= 2;

  if (judgments.publicationBias.rating === 'serious') score -= 1;
  else if (judgments.publicationBias.rating === 'very-serious') score -= 2;

  // Upgrades
  if (judgments.largeEffect.rating === 'large') score += 1;
  else if (judgments.largeEffect.rating === 'very-large') score += 2;

  if (judgments.doseResponse.rating === 'yes') score += 1;
  if (judgments.residualConfounding.rating === 'yes') score += 1;

  if (score > 4) score = 4;
  if (score < 1) score = 1;

  const labels = { 4: 'High', 3: 'Moderate', 2: 'Low', 1: 'Very Low' };
  return labels[score];
};
```

T6-2 PICO 與結局萃取：輸入組裝

```javascript
const documentTextSummary = documents.map((d, index) => 
  `[文獻 #${index + 1}]\n標題: ${d.title}\n作者: ${d.author} (${d.year})\n內文數據: ${d.content.substring(0, 5000)}` // Slice to avoid exceeding token limit
).join('\n\n');

const promptText = `
【臨床基本提問】: ${picoQuery || "未提供特定偏好，請從文獻中自動歸納"}
【所有文獻資料】:
${documentTextSummary}
`;
```

T6-3 PICO 與結局萃取：提示詞

```text
const systemPrompt = `你是一個資深的實證醫學與 GRADE 方法學家。請詳細閱讀多篇文獻內容，並完成以下任務：
1. 自動定義此臨床主題的 PICO 架構。
2. 針對「台灣本土健保給付與臨床環境」，推估此主題可能面臨的台灣本土限制與給付考量背景。
3. 自動推薦並建立 2 到 4 個對於這項臨床主題最重要的結果指標 (Outcomes)，包含一個「預防性/主要療效指標」(如預防事件)與「安全性指標」(如出血、不良反應)，並註明哪些是關鍵 (Critical) 哪些是重要 (Important)。

請以繁體中文(台灣常用醫學指引口吻)回覆，並嚴格遵循以下 JSON 格式以便程式解析：
{
  "title": "最精準、高水準的指引名稱主題",
  "population": "細緻、具體的患者族群定義 (P)",
  "intervention": "治療介入方式定義 (I)",
  "comparator": "對照組療法定義 (C)",
  "taiwanContext": "融入台灣現況(健保、申報、自費考量)的分析說明",
  "recommendedOutcomes": [
    {
      "name": "指標名稱 (如：主要心血管不良事件 MACE)",
      "type": "critical 或 important"
    },
    {
      "name": "安全性指標名稱 (如：嚴重副作用發生率)",
      "type": "critical 或 important"
    }
  ]
}`;
```

T6-4 OCEBM 評定：輸入組裝

```javascript
const docsInput = documents.map((d, idx) => (
  `[文獻 ID: ${d.id}] (#${idx + 1})\n標題: ${d.title}\n作者與年份: ${d.author} (${d.year})\n內文摘要與方法學:\n${d.content.substring(0, 4500)}`
)).join("\n\n---\n\n");

const promptText = `
【臨床情境 PICO】:
- 主題/疑問: ${targetPico.title || picoQuery || "未指定，請依文獻推斷"}
- P (族群): ${targetPico.population || "依文獻內容推斷"}
- I (介入/診斷/暴露): ${targetPico.intervention || "依文獻內容推斷"}
- C (對照): ${targetPico.comparator || "依文獻內容推斷"}

【待評估文獻列表 (共 ${documents.length} 篇)】:
${docsInput}
`;
```

T6-5 OCEBM 評定：提示詞

```text
const systemPrompt = `你是一個精通實證醫學與《2011 Oxford Centre for Evidence-Based Medicine (OCEBM) Levels of Evidence》的方法學專家。
請嚴格依據 2011 OCEBM 評估標準（包含 Introductory Document、Background Document 與 2011 Levels Table），結合使用者提供的【臨床情境 PICO】，逐一評估每一篇上傳文獻的證據等級（Level 1 至 Level 5）。

【2011 OCEBM 評估準則重點】：
1. 判斷臨床問題類型 (7大類)：
   - Prevalence (How common is the problem?): Level 1 Local random sample surveys/censuses; Level 2 SR of surveys; Level 3 Local non-random sample; Level 4 Case-series; Level 5 n/a.
   - Diagnosis (Is this diagnostic/monitoring test accurate?): Level 1 SR of cross-sectional studies with reference standard & blinding; Level 2 Individual cross-sectional study; Level 3 Non-consecutive or inconsistent reference standard; Level 4 Case-control or poor reference standard; Level 5 Mechanism-based reasoning.
   - Prognosis (What will happen if we do not add a therapy?): Level 1 SR of inception cohort; Level 2 Inception cohort; Level 3 Cohort or control arm of RCT; Level 4 Case-series/case-control/poor cohort; Level 5 n/a.
   - Treatment Benefits (Does this intervention help?): Level 1 SR of RCTs or n-of-1 trials; Level 2 RCT or observational study with dramatic effect; Level 3 Non-randomized controlled cohort/follow-up; Level 4 Case-series, case-control, or historically controlled; Level 5 Mechanism-based reasoning.
   - Common Harms (>20%): Level 1 SR of RCTs, SR of nested case-control, n-of-1 with patient, or observational study with dramatic effect; Level 2 Individual RCT or (exceptionally) observational study with dramatic effect; Level 3 Non-randomized controlled cohort/post-marketing surveillance with sufficient numbers/follow-up; Level 4 Case-series/case-control/historically controlled; Level 5 Mechanism-based reasoning.
   - Rare Harms: Level 1 SR of RCTs or n-of-1; Level 2 RCT or (exceptionally) observational study with dramatic effect; Level 3-5 同 Common Harms。
   - Screening: Level 1 SR of RCTs; Level 2 RCT; Level 3 Non-randomized controlled cohort; Level 4 Case-series/case-control/historically controlled; Level 5 Mechanism-based reasoning.
2. 升降階原則 (Grading Down / Grading Up，適用於 Level 1–4)：
   - 降階 (Grade Down) 原因：Study quality (研究品質/偏倚風險)、Imprecision (不精確性/樣本過小或信賴區間過寬)、Indirectness (間接性：文獻的 PICO 與使用者輸入的臨床情境 PICO 不符！)、Inconsistency (研究間不一致性)、Very small absolute effect size (絕對效應量極小)。
   - 升階 (Grade Up) 原因：Large or very large effect size (大效應量或極大效應量，例如觀察性研究呈現戲劇性療效可升階)。
3. 輸出要求：
   - 必須以 "Level 1"、"Level 2"、"Level 3"、"Level 4" 或 "Level 5" 呈現。
   - 清楚說明「初始等級 (initialLevel)」、「最終等級 (finalLevel)」、「升降階狀態 (adjustment)」，以及詳細的「升階或降階原因（若維持原等級亦需說明為何未升/降階，並比對文獻與臨床 PICO 的契合度）」。

請以繁體中文回覆，並嚴格輸出純 JSON 陣列（不要包含多餘文字），格式如下：
[
  {
    "id": "對應輸入的文獻 ID",
    "questionType": "對應之 OCEBM 臨床問題類型（如：Treatment Benefits 治療療效）",
    "studyDesign": "辨識出的研究設計（如：隨機對照試驗 RCT、系統性文獻回顧 SR、世代研究 Cohort Study 等）",
    "initialLevel": "Level 1 或 Level 2 或 Level 3 或 Level 4 或 Level 5",
    "finalLevel": "Level 1 或 Level 2 或 Level 3 或 Level 4 或 Level 5",
    "adjustment": "升階 (Upgraded) 或 降階 (Downgraded) 或 維持原等級 (No Change)",
    "reason": "詳細解釋依據 2011 OCEBM 標準判定初始等級的原因，並逐項檢視與臨床情境 PICO 的符合度(Indirectness)、研究品質(Study quality)、精確度(Imprecision)與效應量(Effect size)，具體說明升階、降階或不調整的理由。"
  }
]`;
```

T6-6 GRADE 結局評讀：輸入與提示詞

```text
const documentTextSummary = documents.map((d, index) => 
  `[文獻 #${index + 1}]\n標題: ${d.title}\n作者: ${d.author} (${d.year})\n數據與方法學內文: ${d.content.substring(0, 5000)}`
).join('\n\n');

const systemPrompt = `你是一個資深的實證醫學方法學與臨床指引評析專家。
請分析使用者文獻資料庫池中所有文獻。你的任務是：
1. 針對臨床問題 (PICO):
   P: ${pico.population}
   I: ${pico.intervention}
   C: ${pico.comparator}
2. 從所有文獻中，精確交叉比對、整合、或萃取出與當前結果指標【${activeOutcome.name}】最符合的統計數據。
   包括：相對效應(RR/HR/OR, 95% CI)、對照組基線風險(Baseline risk)、介入治療組絕對風險與絕對效果差(Absolute Difference)。
3. 依據 2011 GRADE 與最新 2025 Core GRADE 指引，評量此指標之 5 大降級與 3 大升級因素：
   - 特別依照「2025 Core GRADE」決策臨界值(Decision Thresholds)評定不精確性(Imprecision)。
   - 考慮 Schmidt et al. (2011) 指出之抽樣設計缺失(survey design misspecification)對觀察性或特定調查研究點估計與標準誤的偏差風險。若文獻之統計分析方法有不合理之忽略，應於偏倚風險降級。
4. 用繁體中文(台灣常用醫學專門術語)撰寫評讀判定理由 (reason)，並在 reason 中精確進行學術引用 (例如 Connolly et al., 2009; Granger et al., 2011)。

輸出格式必須嚴格符合以下 JSON 結構：
{
  "relativeEffect": "相對效應，如 RR 0.80 (95% CI: 0.70 - 0.92)",
  "baselineRisk": "基線風險，例如 50 per 1000",
  "interventionRisk": "治療組絕對風險，例如 40 per 1000",
  "absoluteDifference": "絕對效應差，例如 10 fewer per 1000 (95% CI: 4 to 15 fewer)",
  "sourceStudies": "整合之來源文獻縮寫，如 Connolly et al., 2009; Granger et al., 2011",
  "judgments": {
    "riskOfBias": { "rating": "no 或 serious 或 very-serious", "reason": "偏倚風險評讀理由，需引用具體論文設計細節(APA格式)" },
    "inconsistency": { "rating": "no 或 serious 或 very-serious", "reason": "不一致性與統計異質性判定理由" },
    "indirectness": { "rating": "no 或 serious 或 very-serious", "reason": "間接性評估理由" },
    "imprecision": { "rating": "no 或 serious 或 very-serious", "reason": "不精確性(結合2025決策臨界值)理由" },
    "publicationBias": { "rating": "no 或 serious", "reason": "發表偏倚判定理由" },
    "largeEffect": { "rating": "no 或 large 或 very-large", "reason": "大效應升級理由" },
    "doseResponse": { "rating": "no 或 yes", "reason": "劑量效應升級理由" },
    "residualConfounding": { "rating": "no 或 yes", "reason": "殘餘混雜升級理由" }
  },
  "footnotes": "指標特別注意事項之簡明註腳與綜整結論。"
}`;
```

T6-7 建議生成：輸入與提示詞

```text
const outcomesSummary = outcomes.map(o => 
  `- 結果指標: ${o.name}\n  重要性: ${o.type}\n  相對效應: ${o.relativeEffect}\n  絕對效果: ${o.absoluteDifference}\n  證據確信度 (GRADE CoE): ${o.certainty}\n  專家評讀說明: ${o.footnotes}`
).join('\n\n');

const systemPrompt = `你是一個台灣臨床醫學指引發展專家小組的主席，熟稔台灣全民健康保險 (NHI) 的審查要點、醫院實務臨床常規以及醫病共享決策 (SDM)。
請依據以下 GRADE 證據評估彙整，以及台灣本土臨床、健保政策環境：

【臨床問題 (PICO)】：
P: ${pico.population}
I: ${pico.intervention}
C: ${pico.comparator}
台灣在地背景與痛點：${pico.taiwanContext}

【GRADE Summary of Findings (SoF) 證據彙整】：
${outcomesSummary}

請以繁體中文(台灣常用醫學指引文體)撰寫一份極高水準、可直接套用至醫院院內指引(Clinical Practice Guideline)的官方建議書。
在推導邏輯中，請必要時引用您上傳的核心文獻(如：Guyatt et al., 2011; Balshem et al., 2011; Schmidt et al., 2011; Guyatt et al., 2025)。
回覆格式必須嚴格符合以下 JSON 格式：
{
  "direction": "for 或 against",
  "strength": "strong 或 conditional",
  "statement": "高水準、結構化且具體、符合台灣臨床用藥實務的指引核心陳述 (以醫學繁體中文表現)",
  "nhisRatio": "針對台灣健保給付要點、事前審查、給付限制及排除條件等最詳細的實務申報與處方建議(極重要！)",
  "practiceGuidance": "具體可行且符合台灣醫院現況的臨床操作流程(分點詳述)",
  "sdmSheet": "為該指引設計的「醫病共享決策(SDM)輔助工具大綱」，清楚對比介入與對照之利弊、自費負擔、台灣病患價值偏好"
}`;
```

T6-8 參考文獻目錄（原工具內建）

```javascript
const [citationList, setCitationList] = useState([
  { id: 'c1', author: "Guyatt, G. H., Oxman, A. D., Kunz, R., et al.", year: "2011", title: "GRADE guidelines: 2. Framing the question and deciding on important outcomes.", journal: "Journal of Clinical Epidemiology", volume: "64", issue: "4", pages: "395-400" },
  { id: 'c2', author: "Balshem, H., Helfand, M., Schünemann, H. J., et al.", year: "2011", title: "GRADE guidelines: 3. Rating the quality of evidence.", journal: "Journal of Clinical Epidemiology", volume: "64", issue: "4", pages: "401-406" },
  { id: 'c3', author: "Guyatt, G. H., Oxman, A. D., Vist, G., et al.", year: "2011", title: "GRADE guidelines: 4. Rating the quality of evidence—study limitations (risk of bias).", journal: "Journal of Clinical Epidemiology", volume: "64", issue: "4", pages: "407-415" },
  { id: 'c4', author: "Schmidt, C. O., Alte, D., Völzke, H., et al.", year: "2011", title: "Partial misspecification of survey design features sufficed to severely bias estimates of health-related outcomes.", journal: "Journal of Clinical Epidemiology", volume: "64", issue: "4", pages: "416-423" },
  { id: 'c5', author: "Guyatt, G., Agoritsas, T., Brignardello-Petersen, R., et al.", year: "2025", title: "Core GRADE 1: overview of the Core GRADE approach.", journal: "BMJ", volume: "389", issue: "e081903", pages: "1-4" }
]);
```