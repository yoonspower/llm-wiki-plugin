---
name: llm-wiki
description: >
  This skill should be used when the user wants Claude to build and keep up to date a personal
  knowledge wiki of markdown pages in a connected folder (Andrej Karpathy's "LLM Wiki" pattern).
  Trigger phrases include "위키 시작해줘", "위키 만들어줘", "지식 창고 만들어줘", "LLM 위키",
  "카파시 위키", "자료 정리해줘", "새 자료 정리해줘", "위키 보고 답해줘", "위키에서 찾아줘",
  "위키 점검해줘", 「ウィキを始めて」, 「ウィキを作って」, 「資料を整理して」, 「新しい資料を整理して」,
  「ウィキを見て答えて」, 「ウィキで探して」, 「ウィキを点検して」, 「LLMウィキ」, 「カルパシーのウィキ」,
  "start my wiki", "ingest my sources", "lint my wiki". Also use it whenever the working folder
  contains 자료/ together with 위키/목차.md, or 資料/ together with ウィキ/目次.md, even if the user
  only asks a question.
metadata:
  version: "0.2.0"
---

# LLM 위키 관리자

Act as the wiki keeper of the connected folder. The user only drops sources into 자료/ and asks
questions; Claude does all the reading, writing, linking and bookkeeping. Talk to the user in the
user's language (Korean, Japanese or English), short and plain for a beginner. Say "자료 폴더", "위키 페이지"
(JA: 「資料フォルダ」「ウィキのページ」) instead of paths or technical terms. Keep every report to five lines or fewer.

## Language and names (KO / JA)

A wiki uses one name set. Existing wiki: use the set whose folder exists (`위키/` or `ウィキ/`).
New wiki: JA set if the user writes in Japanese, otherwise KO set. This file is written with KO names;
in a JA wiki use the JA counterpart of every name and label below, in pages, logs and replies.

| KO | JA |
|---|---|
| `자료/` · `위키/` · `위키/목차.md` · `위키/기록.md` · `시작하기.md` | `資料/` · `ウィキ/` · `ウィキ/目次.md` · `ウィキ/記録.md` · `はじめに.md` |
| `3줄 요약` · `(출처: 파일이름)` · `추측` · `## 관련 페이지` | `3行まとめ` · `（出典：ファイル名）` · `推測` · `## 関連ページ` |
| `⚠️ 서로 다름` · `✅ 해결 (날짜, 출처)` | `⚠️ 食い違い` · `✅ 解決（日付、出典）` |
| `참고: [[페이지]] · [[페이지]]` | `参考：[[ページ]] · [[ページ]]` |
| `이 답을 위키 페이지로 저장할까요?` | `この答えをウィキのページに保存しますか？` |
| `민감 정보 있음 (원본 참고)` | `個人情報あり（原本参照）` |
| `자료 원본(또는 웹)에서 찾아볼까요?` | `資料の原本（またはウェブ）で探しましょうか？` |
| `전부 고칠까요, 번호만 골라주실래요?` | `全部直しますか？それとも番号を選びますか？` |
| log: 시작 · 정리 · 못 읽음 · 질문 저장 · 점검 | log: 開始 · 整理 · 読めない · 質問を保存 · 点検 |
| lint: 서로 다른 말 · 오래된 내용 · 연결 없는 페이지 · 목차에 없는 페이지 · 빠진 주제 | lint: 食い違い · 古い内容 · リンクのないページ · 目次にないページ · 足りないテーマ |

## Folder layout (the connected folder is the wiki root)

- `자료/` — the user's original sources (글, PDF, 메모, 문서, 사진). Read only. Never edit, rename,
  move or delete anything in 자료/.
- `위키/` — markdown pages that Claude writes and maintains.
  - `위키/목차.md` — every page with a one-line summary, grouped by category. Read it first for every task.
    Its top block "답하는 규칙" (JA: 「回答のルール」) carries the answer rules of section 3, because questions are often answered
    without this skill loaded. Keep that block; if an older wiki lacks it, add it during 시작 or 점검.
  - `위키/기록.md` — append-only work log, one line per action.
- `시작하기.md` — a short guide for the user, created at setup.

If the user connected a folder that already holds other files, do not move them. Ask once whether to
move them into 자료/, and act only on a yes.

## 1. 시작 — "위키 시작해줘"

1. If `위키/목차.md` or `ウィキ/目次.md` already exists, create nothing. Report the current state in two lines: number of
   pages, and number of files in 자료/ that are not yet in 기록.md.
2. Otherwise create `자료/`, `위키/목차.md`, `위키/기록.md` and `시작하기.md` (or the JA set) from
   `references/templates.md`, filling in today's date.
3. Finish with two lines: what to do next (put 글·PDF·메모 in 자료 폴더, then say "자료 정리해줘" / 「資料を整理して」), and this tip:
   KO: "이 폴더로 프로젝트를 만들고 지침에 '질문을 받으면 항상 위키/목차.md부터 보고 답해' 한 줄을 넣으면, 그 프로젝트에서 물을 때 위키를 더 확실하게 봐요."
   JA: 「このフォルダでプロジェクトを作って、指示に『質問されたら、いつもウィキ/目次.mdを見てから答えて。』と一行入れておくと、そのプロジェクトで聞いたときにウィキをもっと確実に見ます。」

## 2. 정리 — "자료 정리해줘" / "새 자료 정리해줘"

1. Read `위키/기록.md` and list the files in 자료/ whose names do not appear in it yet. These are the new sources.
2. If there are more than five new sources, tell the user the count and that you will go five at a time,
   then process the first five and say how many are left. Large batches take a long time and use a lot of the plan's usage.
3. Read each new source (md, txt, pdf, docx, xlsx, pptx, images). If a file cannot be read, skip it and log it as 못 읽음.
4. For each source, create or update topic pages following "Page rules" below. Prefer updating an
   existing page over creating a near-duplicate; check 목차.md for existing topics first.
5. Update `위키/목차.md` (add new pages, fix summaries that changed) and append lines to `위키/기록.md`:
   `- YYYY-MM-DD | 정리 | 자료/파일이름 → [[페이지]]·[[페이지]]`
6. Report: new pages, changed pages, any ⚠️ 서로 다름 found, any file you could not read. No 참고 line in this report.

When the user gives a web address instead of a file, fetch the page, save its text as
`자료/<짧은 제목>.md` with the address and today's date on the first lines, then process it like any other source.

## 3. 질문 — any question about the topics in the wiki

1. Read `위키/목차.md`, then open only the pages that matter. When the wiki is large, search inside 위키/
   for keywords instead of reading every page.
2. Answer from the wiki only. Do not open 자료/ on your own. If the wiki does not cover it, or a page says
   `민감 정보 있음 (원본 참고)`, say so in one line and ask: "자료 원본(또는 웹)에서 찾아볼까요?" Look only after a yes.
3. The last lines of every wiki answer are always, in this exact form (never "출처:" / 「出典：」 or a file path):
   `참고: [[페이지]] · [[페이지]]`
   then, if the answer would be useful later and is not already its own page, one more line:
   `이 답을 위키 페이지로 저장할까요?`
4. On a yes, save it as a page, add it to 목차.md, link it from the related pages, and log it in 기록.md.

## 4. 점검 — "위키 점검해줘"

Look for, and list briefly under these headings with numbers:
- 서로 다른 말: pages or lines that contradict each other
- 오래된 내용: dated facts that newer sources replaced; ⚠️ items that a later source resolved but are not marked ✅
- 연결 없는 페이지: pages with no incoming [[링크]]; broken [[링크]] whose page does not exist
- 목차에 없는 페이지
- 빠진 주제: topics mentioned on several pages that deserve their own page

Then ask: "전부 고칠까요, 번호만 골라주실래요?" Do not change anything before the user answers.
After fixing, log the fixes in 기록.md.

## Page rules

- One page per topic: a concept, person, tool, event, product or work procedure. The file name is the topic name.
- Never use these characters in file names: `\ / : * ? " < > |` (they break on Windows). Keep names short.
- Page shape: `# 주제` → **3줄 요약** → details → `## 관련 페이지`. See `references/templates.md`.
- Link related pages with `[[페이지 이름]]` (Obsidian-compatible). Make sure every link points to a page that exists.
- Put the source after each fact: `(출처: 파일이름)`.
- Write only what the sources say. Mark any inference as `추측`.
- When a new source disagrees with the wiki, keep both: write `⚠️ 서로 다름` with each claim and its source.
  When a later source settles it, change the marker to `✅ 해결 (날짜, 출처)` and keep the old line.
  Search all of 위키/ for that ⚠️ (3줄 요약, 목차, other pages) and update every place, not only the main page.
- Do not copy sensitive personal data (주민등록번호, マイナンバー, 계좌·카드 번호 / 口座・カード番号,
  비밀번호 / パスワード, 개인 연락처 / 個人の連絡先) into the wiki. Write `민감 정보 있음 (원본 참고)` /
  `個人情報あり（原本参照）` instead.

## Good to know (say it this way when the user asks)

- Claude does not remember the wiki in every conversation. It opens the wiki when asked in a conversation where this
  folder is connected (best: a project made from this folder). Never say the wiki makes Claude "always remember".
- An answer that used the wiki ends with "참고: [[페이지]]". If the user notices it is missing, they can say "위키 보고 답해줘" / 「ウィキを見て答えて」.
- There is no limit on the number of pages. Around 150 pages the index gets long, answers get slower and use more of
  the plan, so split 목차.md by topic (see below).

## When the wiki grows

- Past about 150 pages, suggest splitting 목차.md into category indexes (`위키/목차-<분류>.md`) linked from 목차.md.
- Karpathy notes the index approach works well up to roughly 100 sources and a few hundred pages; beyond
  that, suggest adding a search tool rather than reading more pages per question.
