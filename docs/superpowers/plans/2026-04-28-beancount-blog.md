# Beancount 學習 Blog 實作計劃

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 建立一個 IG-like 編輯體驗的 beancount 學習 blog，從手機或桌面都能無摩擦寫作發布，內容同時可在 Obsidian 中查閱編輯。

**Architecture:** GitHub repo 為唯一 source of truth；Pages CMS（雲端版）負責網頁編輯介面、Cloudflare Pages 自動 build & deploy Astro 站、Obsidian 把同一個 repo 當 vault 開讀寫。

**Tech Stack:** Astro v6 + Bookworm Light theme + Pages CMS + Cloudflare Pages + Porkbun (DNS)

**Spec:** [docs/superpowers/specs/2026-04-28-beancount-blog-design.md](../specs/2026-04-28-beancount-blog-design.md)

---

## 前置需求（Pre-flight）

開始前確認下列項目（不需要動 accounting repo，所有變更發生在新 repo `beancount-notes`）：

- [ ] **GitHub** 已登入 `gh` CLI：`gh auth status` 應顯示 `Logged in to github.com as chialin`
- [ ] **Node.js 20+** 已安裝：`node --version` 應 >= v20
- [ ] **Cloudflare 帳號** 已開（如未開：[dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)）
- [ ] **Porkbun** 帳號已能管理 chialin.me（登入 [porkbun.com](https://porkbun.com)）
- [ ] **Pages CMS** 帳號（會在 Task 10 用 GitHub OAuth 第一次登入時建立）
- [ ] 工作目錄：本機開新資料夾放 repo，**不要在 accounting repo 裡**。建議放在 `~/Documents/2-Areas/beancount-notes/`

---

## Task 1: 建 GitHub repo

**External:** GitHub
**Local working dir:** `~/Documents/2-Areas/`

- [ ] **Step 1.1：建 repo（public，無 README、無 .gitignore）**

```bash
gh repo create chialin/beancount-notes \
  --public \
  --description "Beancount 學習筆記 — 個人知識庫"
```

預期輸出：`✓ Created repository chialin/beancount-notes on GitHub`

- [ ] **Step 1.2：驗證 repo 存在**

```bash
gh repo view chialin/beancount-notes --json name,url,visibility
```

預期看到 `"visibility":"PUBLIC"` 且 `url` 是 `https://github.com/chialin/beancount-notes`

---

## Task 2: 用 Bookworm Light template 初始化

**External:** GitHub (clone source) → 本機新資料夾
**Files created:** 整個 `beancount-notes/` 結構（從 template clone 來）

- [ ] **Step 2.1：clone Bookworm Light template 到本機**

```bash
cd ~/Documents/2-Areas/
git clone https://github.com/themefisher/bookworm-light-astro.git beancount-notes
cd beancount-notes
```

預期：建立 `~/Documents/2-Areas/beancount-notes/` 目錄，內含完整主題檔案

- [ ] **Step 2.2：移除 template 的 git 歷史，重新 init**

```bash
rm -rf .git
git init -b main
git remote add origin https://github.com/chialin/beancount-notes.git
```

預期：`git remote -v` 顯示 origin 指向 `chialin/beancount-notes.git`

- [ ] **Step 2.3：安裝相依套件**

```bash
npm install
```

預期：建立 `node_modules/`、產生 `package-lock.json`，無 error

> 註：template 用 yarn，但用 npm install 會自動產 package-lock.json，相容。後續一律用 npm。

- [ ] **Step 2.4：驗證 dev server 可跑**

```bash
npm run dev
```

預期輸出包含 `Local: http://localhost:4321/`，瀏覽器打開應看到 Bookworm Light 預設首頁

確認後 Ctrl+C 停止 dev server。

---

## Task 3: 客製化站台基本資訊

**Files modified:**
- `src/config/config.json` — 站名、URL、語言
- `src/config/menu.json` — 導覽選單
- `src/config/social.json` — 社群連結（清空）

- [x] **Step 3.1：編輯 `src/config/config.json`**

把 `site.title`、`site.base_url`、`site.description`、`site.author`、`metadata` 等改成符合本站：

```json
{
  "site": {
    "title": "Beancount 學習筆記",
    "base_url": "https://beancount-notes.chialin.me",
    "base_path": "/",
    "trailing_slash": false,
    "favicon": "images/favicon.png",
    "logo": "images/logo.png",
    "logo_darkmode": "images/logo-darkmode.png",
    "logo_width": "160px",
    "logo_height": "32px",
    "logo_text": "Beancount Notes"
  },
  "settings": {
    "search": true,
    "sticky_header": true,
    "theme_switcher": true,
    "default_theme": "system",
    "pagination": 8,
    "summary_length": 200,
    "blog_folder": "posts"
  },
  "metadata": {
    "meta_author": "chialin",
    "meta_description": "我學習 beancount 與複式記帳的筆記與整理。",
    "meta_image": "images/og-image.png"
  },
  "params": {
    "footer_content": "© 2026 chialin · 用 Astro + Bookworm Light 打造"
  },
  "navigation_button": {
    "enable": false,
    "label": "",
    "link": ""
  }
}
```

> 註：保留 template 中其他既有欄位（如 `disqus`、`google_tag_manager` 等），把 `enable` 都設為 `false` 即可關閉

- [x] **Step 3.2：編輯 `src/config/menu.json` 簡化導覽**

```json
{
  "main": [
    { "name": "首頁", "url": "/" },
    { "name": "文章", "url": "/posts" },
    { "name": "標籤", "url": "/tags" },
    { "name": "關於", "url": "/about" }
  ],
  "footer": [
    { "name": "About", "url": "/about" },
    { "name": "RSS", "url": "/rss.xml" },
    { "name": "我的另一個 Blog", "url": "https://blog.chialin.me" }
  ]
}
```

- [x] **Step 3.3：清空 `src/config/social.json` 的範例值**

把預設的 facebook/twitter 等改成只留你實際使用的（或全留空）：

```json
{
  "github": "https://github.com/chialin",
  "rss": "/rss.xml"
}
```

> 主題會 iterate 此檔案產出 footer icons；保留 key 但 value 可以是空字串或實際 URL

- [x] **Step 3.4：再跑一次 dev server 確認改完沒壞**

> **實作備註**：
> - `[方向調整]` social.json 原計畫的扁平 `{github, rss}` 結構不符 Bookworm theme 的 schema（footer 用 `social.main` 陣列搭 name/icon/link），改為陣列形式並用 `FaGithub` / `FaRss` icon。
> - `[技術障礙]` 用 build 取代 dev server 驗證時，發現兩個 config 缺漏：(1) `params.copyright` 被誤寫成 `footer_content`（Footer.astro 讀的是 `copyright`）；(2) `contactinfo.{address,email,phone}` 被遺漏（contact.astro 解構這個欄位）。兩者皆已補上 stub。
> - `[後續依賴]` Task 4 清理範例內容時可一併考慮是否刪除 `/contact` 頁與對應的 `src/content/contact/-index.md`；若刪除則 `config.contactinfo` stub 也可移除。

```bash
npm run dev
```

預期：localhost:4321 看到「Beancount 學習筆記」標題與新導覽選單

Ctrl+C 停止。

---

## Task 4: 建立作者頁、about 頁、清掉範例內容

**Files modified:**
- `src/content/authors/chialin.md` — 新增（取代範例作者）
- `src/content/posts/post-1.md` ~ `post-7.md` — 刪除
- `src/content/posts/-index.md` — 保留（這是文章列表頁的設定）
- `src/content/about/-index.md` — 改寫
- `src/content/authors/` 下其他範例作者 — 刪除

- [x] **Step 4.1：列出現有 authors 資料夾**

```bash
ls -la src/content/authors/
```

預期看到幾個範例 .md 檔（如 `john-doe.md`）。

- [x] **Step 4.2：建立自己的作者檔，刪除範例**

先建立 `src/content/authors/chialin.md`：

```markdown
---
title: chialin
meta_title: "關於 chialin"
description: "把學 beancount 的過程寫下來，給自己也順便給別人看。"
image: "/images/authors/chialin.png"
social:
  github: "https://github.com/chialin"
---

我是 chialin，從 2024 年開始用 beancount 記帳。

這個 blog 是學習過程中的筆記與心得整理，主題包含：

- Beancount 語法與雙式記帳概念
- 多幣別與投資紀錄
- 加密貨幣記帳
- 自動化工具
```

刪除其他範例作者：

```bash
# 先確認列表，再刪掉非 chialin 的
ls src/content/authors/
# 例：rm src/content/authors/john-doe.md src/content/authors/...
```

> 動手前列一遍清單，避免誤刪到 `-index.md` 之類的設定檔

- [x] **Step 4.3：刪除範例文章**

```bash
ls src/content/posts/
# 預期看到 -index.md, post-1.md, ..., post-7.md
rm src/content/posts/post-*.md
ls src/content/posts/
# 預期只剩 -index.md
```

- [x] **Step 4.4：改寫 `src/content/about/-index.md`**

打開檔案，把內文改寫為：

```markdown
---
title: "關於這個 Blog"
meta_title: "關於 — Beancount 學習筆記"
description: "為什麼有這個 blog、寫些什麼、誰在寫"
image: ""
draft: false

what_i_do:
  title: "關於這裡"
  items:
    - title: "Beancount 學習筆記"
      description: "把學雙式記帳、beancount 語法、自動化工具的過程寫下來。"
    - title: "個人知識庫"
      description: "以個人筆記為主，偶爾整理成可分享的文章。"
    - title: "繁體中文"
      description: "全站用繁體中文撰寫。"
---

這個 blog 是 chialin 的 beancount 學習筆記。

更個人化的生活紀錄請見 [blog.chialin.me](https://blog.chialin.me)。
```

> 註：`what_i_do` 物件結構是 Bookworm Light about schema 要求的（Section 4 spec 沒詳細列，這裡依 template 的 schema）

- [x] **Step 4.5：跑一次 build 確認沒壞**

```bash
npm run build
```

預期：`Server is built` / `Generated dist/` 訊息，無 error。

> 若 build error 提示 schema 缺欄位，依錯誤訊息補上對應欄位（通常是 about/contact 等需要必填的 metadata）

- [x] **Step 4.6：commit 階段成果**

```bash
git add -A
git commit -m "init: bootstrap Bookworm Light template + customize site metadata + clean sample content"
```

---

## Task 5: 加入 `.pages.yml` 設定 Pages CMS

**Files created:** `.pages.yml`（repo 根目錄）

- [x] **Step 5.1：在 repo 根建立 `.pages.yml`**

完整內容：

```yaml
content:
  - name: posts
    label: 文章
    type: collection
    path: src/content/posts
    filename: '{year}-{month}-{day}-{primary}.md'
    view:
      fields: [title, date, tags, draft]
      sort: [date desc]
    fields:
      - { name: title, label: 標題, type: string, required: true }
      - { name: meta_title, label: SEO 標題, type: string }
      - { name: description, label: 摘要, type: text }
      - { name: date, label: 發布日期, type: date, required: true }
      - { name: image, label: 封面圖, type: image }
      - name: tags
        label: 標籤
        type: string
        list: true
        options:
          creatable: true
          values: [記帳, 基礎]
      - { name: categories, label: 分類, type: string, list: true, default: [notes], hidden: true }
      - { name: authors, label: 作者, type: string, list: true, default: [chialin], hidden: true }
      - { name: draft, label: 草稿, type: boolean, default: false }
      - { name: body, label: 內文, type: rich-text }

  - name: pages
    label: 靜態頁面
    type: collection
    path: src/content/pages
    fields:
      - { name: title, type: string, required: true }
      - { name: body, type: rich-text }

media:
  input: src/assets
  output: /src/assets
```

- [x] **Step 5.2：建立 `src/assets/` 與 `src/content/pages/` 資料夾（不存在則建）**

> **實作備註**：`src/content/pages/` 已存在且有 `elements.mdx`、`privacy-policy.md`，不需 `.gitkeep`；只在空的 `src/assets/` 放一份。

```bash
mkdir -p src/assets src/content/pages
touch src/assets/.gitkeep src/content/pages/.gitkeep
```

> Pages CMS 需要 `media.input` 路徑與 `pages` collection path 都實際存在

- [x] **Step 5.3：commit**

```bash
git add .pages.yml src/assets/.gitkeep src/content/pages/.gitkeep
git commit -m "feat: add Pages CMS config"
```

---

## Task 6: 第一次推到 GitHub

**External:** GitHub

- [x] **Step 6.1：再跑一次 build 最終確認**

```bash
npm run build
```

預期：無 error。

- [x] **Step 6.2：push 到 origin/main**

```bash
git push -u origin main
```

預期：成功推送，輸出 `branch 'main' set up to track 'origin/main'`

- [x] **Step 6.3：在 GitHub 上目視確認**

```bash
gh repo view chialin/beancount-notes --web
```

預期：瀏覽器打開 GitHub 頁面，看到 README、`src/`、`.pages.yml` 等檔案

---

## Task 7: 設定 Cloudflare Pages

**External:** Cloudflare Dashboard ([dash.cloudflare.com](https://dash.cloudflare.com))

> **[方向調整] Pages → Workers Builds**：2026 年 Cloudflare 介面已主推 Workers Builds（搭 Static Assets）取代傳統 Pages 流程。實際採用後者：
> - 走 `Compute (Workers)` → `Create` → `Connect to Git` 入口
> - 設定欄位多了一個 `Deploy command`，預設 `npx wrangler deploy`，需要 repo 內有 `wrangler.jsonc`
> - 模板 (Bookworm Light) 已內建 `wrangler.jsonc` 與 `package.json` 中的 `wrangler` v4 + `deploy:cf-workers` 等 scripts，僅需把 `name` 從 `bookworm-light-astro` 改為 `beancount-notes`
> - 部署後預設網址是 `beancount-notes.<account-subdomain>.workers.dev`（不是 `*.pages.dev`），Task 8.1 的 CNAME `Answer` 改用此網址
> - 其餘自訂網域、TLS 流程同 Pages

- [x] **Step 7.1：登入 Cloudflare → Compute (Workers) → Create → Import a repository**

導覽路徑：左側選單 `Compute` → `Workers & Pages`（或新版 `Workers`）→ 點 `Create` → 選 `Import a repository`（Connect to Git）

> 若找不到入口，直接 deep link：`https://dash.cloudflare.com/?to=/:account/workers-and-pages`

- [x] **Step 7.2：授權 GitHub 整合（首次設定才需要）**

- 點 `Connect GitHub`
- 在 GitHub 授權頁，選 `Only select repositories`，加入 `chialin/beancount-notes`
- 回到 Cloudflare，repo 列表應出現 `beancount-notes`

- [x] **Step 7.3：選 repo 並設 build configuration**

| 欄位 | 值 |
|------|------|
| Project name | `beancount-notes`（要與 `wrangler.jsonc` 內 `name` 一致；workers.dev 子網域會是 `beancount-notes.<account-subdomain>.workers.dev`） |
| Production branch | `main` |
| Build command | `npm run build` |
| Deploy command | `npx wrangler deploy`（保持預設） |
| Root directory | `/`（保持預設） |
| Environment variables | 無 |

> Workers Builds 不再有 `Framework preset` 與 `Build output directory` 欄位 — 後者由 `wrangler.jsonc` 的 `assets.directory` (`./dist`) 指定。

- [x] **Step 7.4：點 `Save and Deploy`，等待第一次 build 完成**

預期：build log 顯示 `npm install` → `npm run build` → `npx wrangler deploy` → `Success! Your project is deployed`，總耗時約 1–2 分鐘

- [x] **Step 7.5：驗證 temporary URL 可訪問**

打開 `https://beancount-notes.chialin-shr.workers.dev`（已驗證：HTTP/2 200，`cf-cache-status: HIT`，首頁顯示「Beancount 學習筆記」）

預期：看到 Bookworm Light 樣式 + 「Beancount 學習筆記」站名 + 空白的文章列表（因為剛清空範例）

> 若 build fail：到 Deployments 頁看 log，常見錯誤是 Node 版本太舊 → 在 Settings → Variables and Secrets 加 `NODE_VERSION=20`

---

## Task 8: 設定 DNS（Porkbun + Cloudflare 自訂網域）

**External:** Porkbun ([porkbun.com/account/domains](https://porkbun.com/account/domains)) + Cloudflare Pages

> **[方向調整] 子網域 `beancount` → `beancount-notes`**：實際設 CNAME 時 Porkbun 報錯 `Another record type already exists for that host`，dig 確認 `beancount.chialin.me` 已 CNAME 指向 `export-beancount-expenses.pages.dev`（既有匯出工具，仍在用，不能覆蓋）。改用 `beancount-notes.chialin.me` 作為本站網域（與 repo / Worker 同名，明確）；同步更新 `src/config/config.json` 的 `base_url`、spec、plan、README 等所有引用。

- [ ] **Step 8.1：在 Porkbun 加 CNAME 記錄**

- 登入 Porkbun → Domains → `chialin.me` → 點 `DNS Records`
- 點 `+ Add` 新增記錄：

| 欄位 | 值 |
|------|------|
| Type | `CNAME` |
| Host | `beancount` |
| Answer | `beancount-notes.chialin-shr.workers.dev`（從 Task 7.5 來） |
| TTL | `600` |

點 `Add Record`。

- [ ] **Step 8.2：等 DNS propagation（30–300 秒）**

```bash
dig beancount-notes.chialin.me CNAME +short
```

預期：回傳 `beancount-notes.chialin-shr.workers.dev.`（前面可能還有別的 CNAME 中繼）

> 若沒回傳：等 1–5 分鐘再試。Porkbun 通常很快。

- [ ] **Step 8.3：在 Cloudflare Pages 加 custom domain**

- Cloudflare Dashboard → Pages → 點 `beancount-notes` project → `Custom domains` 分頁
- 點 `Set up a custom domain`
- 輸入 `beancount-notes.chialin.me` → `Continue`
- Cloudflare 會自動偵測 CNAME → 點 `Activate domain`

- [ ] **Step 8.4：等 TLS 憑證核發（1–5 分鐘）**

Status 從 `Verifying` → `Active`。

- [ ] **Step 8.5：驗證 https 可訪問**

```bash
curl -I https://beancount-notes.chialin.me
```

預期：HTTP/2 200，header 含 `cf-cache-status` 等 Cloudflare 標記

瀏覽器打開 `https://beancount-notes.chialin.me` 應看到站台。

---

## Task 9: 連接 Pages CMS 到 GitHub repo

**External:** Pages CMS ([app.pagescms.org](https://app.pagescms.org))

- [ ] **Step 9.1：登入 Pages CMS（GitHub OAuth）**

- 打開 [app.pagescms.org](https://app.pagescms.org)
- 點 `Sign in with GitHub` → 授權 Pages CMS 讀寫 repo（首次才需要）
- 授權範圍：選 `Only select repositories` → 加入 `chialin/beancount-notes`

- [ ] **Step 9.2：在 Pages CMS 加入專案**

- Dashboard → `Add a project`
- 選 `chialin/beancount-notes` → 選 branch `main` → `Add`

- [ ] **Step 9.3：驗證 CMS 讀到 `.pages.yml`**

- 進入 project 後，左側應看到「文章」與「靜態頁面」兩個 collection
- 點「文章」→ 應看到空列表 + 右上有 `+ New entry` 按鈕

- [ ] **Step 9.4：驗證 hidden 欄位行為（spec 要驗證的點）**

- 點 `+ New entry`
- 觀察表單：是否有「分類」與「作者」欄位？
  - **若沒有**（hidden 生效）：✅ 完美，繼續
  - **若有顯示**：把 `.pages.yml` 中的 `hidden: true` 改為 `readonly: true` 或加 `view: hidden`，重新試一次。文件參考：[pagescms.org/docs](https://pagescms.org/docs)
- 確認後關掉表單（不要儲存）

> 若兩種都不支援，最終 fallback：把 categories/authors 從 CMS schema 移除，靠 Astro 預設值處理（schema 已 default）

---

## Task 10: 寫第一篇 hello world（端到端測試）

**External:** Pages CMS 介面 → GitHub commit → Cloudflare Pages build → live site

- [ ] **Step 10.1：在 Pages CMS 寫第一篇文章**

- 進「文章」collection → 點 `+ New entry`
- 填入：

| 欄位 | 值 |
|------|------|
| 標題 | `Hello, Beancount Notes` |
| SEO 標題 | （留空） |
| 摘要 | `第一篇文章，測試整個發布流程是否打通。` |
| 發布日期 | 今天 |
| 封面圖 | （留空） |
| 標籤 | 選 `記帳` |
| 草稿 | 關閉（false） |
| 內文 | 寫一段 markdown，包含 `# 標題`、一段 paragraph、一個 `**粗體**` 測試 |

- 點 `Save`

- [ ] **Step 10.2：驗證 commit 出現在 GitHub**

```bash
gh api repos/chialin/beancount-notes/commits --jq '.[0] | {sha, message, date: .commit.author.date}'
```

預期：最新 commit 訊息類似 `Create posts/2026-04-28-hello-beancount-notes.md`，date 是剛剛

- [ ] **Step 10.3：等 Cloudflare Pages 自動 build（~30 秒）**

到 Cloudflare Pages → `beancount-notes` → `Deployments` 看新 build 進度（從 `Initializing` → `Building` → `Success`）

- [ ] **Step 10.4：驗證文章上線**

打開 `https://beancount-notes.chialin.me`

預期：首頁出現「Hello, Beancount Notes」這篇文章卡片

點進去確認內文渲染正確（標題、粗體、tag 顯示）。

- [ ] **Step 10.5：測試圖片上傳流程**

- 回 Pages CMS → 編輯剛寫的文章
- 在「封面圖」欄位 → 點上傳 → 選一張本機圖
- 觀察 CMS 是否能上傳到 `src/assets/`：
  - 上傳成功 → 看實際路徑（應是 `src/assets/<filename>` 或 `src/assets/<date>/<filename>`，依 CMS 行為）
  - 確認後 Save → 等 build → 驗證 live 站圖片正常顯示

> 若 CMS 上傳後路徑不在預期資料夾，記下實際行為，調整 Obsidian 端的 attachment 路徑配合（兩端要一致）

---

## Task 11: 本機 Obsidian vault 設定

**External:** Obsidian 應用程式
**Local:** `~/Documents/2-Areas/beancount-notes/`（已是 Task 2 clone 的本機 repo）

- [ ] **Step 11.1：先把遠端最新拉下來（Task 10 在 CMS 寫的內容）**

```bash
cd ~/Documents/2-Areas/beancount-notes/
git pull
```

預期：拉下 hello world post 的 commit

- [ ] **Step 11.2：Obsidian 開啟此資料夾為 vault**

- 開啟 Obsidian → `Open another vault` → `Open folder as vault`
- 選 `~/Documents/2-Areas/beancount-notes/`
- 信任作者（首次提示）

- [ ] **Step 11.3：設定 attachment 路徑**

- Obsidian → Settings → Files & Links → `Default location for new attachments`
- 選 `In subfolder under current folder`，並設子資料夾路徑為 `../../assets/{{date:YYYY-MM-DD}}`
- （若 Obsidian 不支援這個語法，退而求其次設成固定資料夾 `src/assets`，每次貼圖手動移）

> 註：Obsidian 的 `{{date}}` 模板會用「今天」日期；spec 預期文章寫作日 = 文章發布日，多數時候對得上

- [ ] **Step 11.4：隱藏無關資料夾不出現在側邊欄**

- Settings → Files & Links → `Excluded files`
- 加入：`node_modules/`、`dist/`、`.astro/`、`.cloudflare/`、`public/`

- [ ] **Step 11.5：驗證 — 在 Obsidian 看到 hello world 文章**

- Obsidian 側邊欄 → `src/content/posts/` → 應看到 `2026-04-28-hello-beancount-notes.md`
- 打開檔案，內容與 CMS 寫的一致

- [ ] **Step 11.6：在 Obsidian 改一段文字、手動 push 驗證雙向**

- 在 Obsidian 編輯該檔案，加一行「（在 Obsidian 補的內容）」
- 終端機：

```bash
cd ~/Documents/2-Areas/beancount-notes/
git add -A
git commit -m "test: edit from obsidian"
git push
```

- 等 Cloudflare Pages build 完，驗證 live 站有更新

> 同步機制（Obsidian Git plugin / launchd / 手動）依使用感受之後決定，目前手動 push 已足以驗證 pipeline

---

## Task 12: 手機端到端測試（驗收）

**External:** 手機瀏覽器 + Pages CMS

- [ ] **Step 12.1：手機瀏覽器登入 Pages CMS**

- 手機打開 [app.pagescms.org](https://app.pagescms.org)
- GitHub OAuth 登入（同帳號）
- 進入 `beancount-notes` project

- [ ] **Step 12.2：手機新增一篇含圖文章（IG-like 體驗測試）**

- 點 `+ New entry`
- 填入標題：`從手機寫的測試`
- 上傳一張手機相簿的圖到「封面圖」
- 寫一兩句內文
- 設「草稿」為關閉
- 點 `Save`

> 計時：從打開 CMS 到按 Save 應該在 30–60 秒內

- [ ] **Step 12.3：驗證上線**

- 等 30–60 秒
- 手機開 `https://beancount-notes.chialin.me`
- 應看到剛剛從手機寫的新文章卡片，圖片正常顯示

- [ ] **Step 12.4：完成驗收清單（從 spec Section 12）**

逐項打勾：

- [ ] 在手機瀏覽器打開 app.pagescms.org，30 秒內貼圖 + 寫一段文字 + 發布
- [ ] 30–60 秒後，內容出現在 https://beancount-notes.chialin.me
- [ ] 桌面打開 Obsidian，看到剛剛從手機寫的文章與圖片（先 `git pull`）
- [ ] Obsidian 改了一段文字，手動 push 後 5 分鐘內同步到網站
- [ ] `bean-check` 等既有 accounting 工作流不受影響

---

## Task 13: README 與專案文件

**Files created:**
- `README.md` — 給未來的自己看，記錄這個 repo 怎麼玩

- [ ] **Step 13.1：寫 README.md**

```markdown
# beancount-notes

Beancount 學習筆記 — 個人知識庫，公開於 [beancount-notes.chialin.me](https://beancount-notes.chialin.me)。

## 架構

- **Astro v6 + Bookworm Light theme**：靜態網站
- **Pages CMS**：[app.pagescms.org](https://app.pagescms.org) 線上編輯
- **Cloudflare Pages**：自動 build & deploy（main 分支 push 觸發）
- **DNS**：Porkbun（CNAME → `beancount-notes.pages.dev`）

## 寫作介面

| 介面 | 用途 |
|------|------|
| Pages CMS | 主要寫作介面，特別是手機 |
| Obsidian | 桌面查閱與編輯，本機 clone repo 當 vault |
| 直接編輯 markdown | 任何文字編輯器都行 |

## 本地開發

\`\`\`bash
npm install
npm run dev      # http://localhost:4321
npm run build    # 產出 dist/
\`\`\`

## 設計與計劃文件

- Design spec: 維護於 [accounting repo](https://github.com/chialin/accounting/blob/main/docs/superpowers/specs/2026-04-28-beancount-blog-design.md)
- Implementation plan: 同上 plans 目錄
```

- [ ] **Step 13.2：commit & push**

```bash
git add README.md
git commit -m "docs: add README"
git push
```

---

## 完成檢查

全部 task 跑完後，最終狀態應該是：

- ✅ GitHub repo `chialin/beancount-notes` 存在且公開
- ✅ `https://beancount-notes.chialin.me` 可訪問且有內容
- ✅ `https://app.pagescms.org` 連接 repo，能新增/編輯文章
- ✅ 本機有一份 clone repo，Obsidian 可開啟為 vault
- ✅ 手機可在 60 秒內發布一篇含圖文章

未完成項目（spec 已標 Future Work，本 plan 不處理）：
- 舊 blog 遷移
- 同步機制最終決定（Obsidian Git plugin / launchd / 手動）
- 圖片優化 / 外部圖床
- i18n
- 進階 SEO

---

**End of plan.**
