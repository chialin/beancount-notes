# Beancount 學習 Blog — 設計文件

**日期：** 2026-04-28
**作者：** chialin
**狀態：** Draft（待 review）

---

## 1. 目標與背景

### 為什麼做這件事

- 想記錄學習 beancount 的過程與整理相關知識
- 既有 blog（`blog.chialin.me`，Next.js + Contentlayer + MDX）難更新：
  - 寫作環境門檻高（要 IDE + Yarn + MDX）
  - 發布流程繁瑣（git commit / push / 等 build）
  - 手機完全不能編輯
  - 圖片處理麻煩
- 既有 blog 主題定位是「生活觀察」，與 beancount 技術筆記性質差異大

### 目標

打造一個**新的、獨立的** beancount 學習 blog，要求：

1. **能在電腦或手機上編輯發布**（IG-like 體驗）
2. **發布流程零摩擦**（不用手動 commit / push）
3. **檔案同步進 Obsidian**（作為查閱與離線編輯的第二介面）
4. **以個人筆記為主**，對外公開但不刻意追求讀者
5. **長期可維護**（避免重蹈 Contentlayer 停更的覆轍）

### 非目標（YAGNI）

- ❌ 不做留言系統
- ❌ 不做 newsletter / email 訂閱
- ❌ 不做 analytics（user 明確選 N）
- ❌ 不做複雜的多階層分類
- ❌ **不在此 spec 處理舊 blog 遷移**（見 Section 11 Future Work）

---

## 2. 架構

### 資料流

```
        ┌───────────────────┐
手機 ──▶ │   Pages CMS       │ ── auto commit ──▶ ┐
         │  (網頁編輯器)      │                    │
         └───────────────────┘                    │
                                                  ▼
                                        ┌──────────────────┐
桌面 Obsidian ── push/pull ───────────▶ │  GitHub Repo     │
(Obsidian Git plugin 自動同步)           │ beancount-notes  │
                                        └────────┬─────────┘
                                                 │ webhook
                                                 ▼
                                        ┌──────────────────┐
                                        │ Cloudflare       │
                                        │ Pages (build)    │ ── deploy ──▶ beancount.chialin.me
                                        │ Astro 編譯        │
                                        └──────────────────┘
```

### 核心原則

**三個介面（Pages CMS、Obsidian、靜態網站）共讀同一個 GitHub repo 的 markdown 檔案。沒有「同步」這件事，只有一份 source of truth。**

---

## 3. 技術選型

| 角色 | 選擇 | 理由 |
|------|------|------|
| **靜態網站產生器** | **Astro v6** | 商業公司 backed、活躍維護、Content Collections 成熟 |
| **主題** | **Bookworm Light Astro**（[github.com/themefisher/bookworm-light-astro](https://github.com/themefisher/bookworm-light-astro)） | MIT 授權免費、2026-04-23 仍在更新、用 Content Collections、frontmatter 命名清楚（`description` 而非怪名字） |
| **CMS** | **Pages CMS**（[pagescms.org](https://pagescms.org)） | 100% 免費、零部署（用雲端版）、手機 UX 為設計核心、GitHub OAuth 整合 |
| **部署** | **Cloudflare Pages** | 免費額度寬裕、自動從 GitHub 拉新 commit 重 build、HTTPS 自動 |
| **網域** | **`beancount.chialin.me`**（DNS 在 Porkbun，加 CNAME 指向 Cloudflare Pages） | 子網域明確、不動到既有 chialin.me 的 nameserver |
| **Obsidian 整合** | **本機 clone repo + Obsidian Git plugin** | 桌面 Obsidian 把 repo 當 vault 開，plugin 自動 pull/push |

### 故意不選的方案（決策紀錄）

- ❌ **既有 chialin-blog repo + Pages CMS**（混入 beancount 內容或新增 collection）
  - 理由：Contentlayer2 是單人維護的 fork，最後 release 在 2025-05，存在 stack 風險。決定一次到位用 Astro
- ❌ **Notion + Super.so**：Notion ↔ Obsidian 同步差，違反需求 3
- ❌ **Obsidian Publish**：手機 Obsidian 體驗差，違反需求 1
- ❌ **Sveltia / Decap CMS**：要自己部署 admin 介面，多一個維護面

---

## 4. Repo 結構與內容模型

### Repo 長相

```
beancount-notes/                    ← GitHub repo（同時是 Obsidian vault）
├── src/
│   ├── content/
│   │   ├── posts/                  ← 文章 markdown（Obsidian / CMS / Astro 共讀）
│   │   │   ├── 2026-04-28-hello.md
│   │   │   └── 2026-05-01-double-entry-basics.md
│   │   ├── authors/
│   │   │   └── chialin.md          ← 單一作者頁
│   │   ├── about/
│   │   │   └── -index.md
│   │   └── pages/                  ← 其他靜態頁
│   ├── assets/                     ← 圖片放這（Bookworm Light 慣例）
│   │   └── 2026-05-01/             ← 以文章日期命名，方便對應
│   │       └── cover.webp
│   ├── content.config.ts           ← 主題定義的 collection schema
│   ├── layouts/, components/, pages/   ← 主題程式碼（少改）
│   └── styles/
├── .pages.yml                      ← Pages CMS 設定
├── astro.config.mjs
├── package.json
└── README.md
```

### Posts frontmatter（依 Bookworm Light schema）

```yaml
---
title: "複式記帳的基本概念"
meta_title: "複式記帳是什麼？— Beancount 入門"
description: "為什麼一筆交易要寫兩行？"
date: 2026-05-01
image: "/src/assets/2026-05-01/cover.webp"
categories: [notes]                       # 固定填 [notes]，不做分類
authors: [chialin]
tags: [基礎, 記帳]
draft: false
---

文章內容（純 markdown）...
```

**欄位設計決策：**

- **`description`**：用來放摘要（Bookworm Light 命名清楚，不像 Serene Ink 的 `frontmatter`）
- **`categories`**：固定填 `[notes]`（user 選 B：只用 tags，categories 用一個固定值）
- **`authors`**：固定填 `[chialin]`，搭配 `src/content/authors/chialin.md` 作者頁
- **`tags`**：初始 seed 為 `[基礎, 記帳]`，可在 CMS 介面自由新增
- **`image`**：可選，封面圖

### 圖片策略：Option A（全域資料夾，按日期分）

- 所有圖片放 `src/assets/YYYY-MM-DD/` 結構（**以文章日期命名，方便對應**）
- Markdown 引用路徑：`/src/assets/YYYY-MM-DD/filename.webp`
- **CMS 上傳**：Pages CMS `media.input` 設為 `src/assets`。實作時驗證是否支援 field-level 的 `path` 模板（例如自動依 `date` 欄位產生子資料夾）：
  - **若支援**：CMS 上傳自動進對應日期資料夾
  - **若不支援**：CMS 上傳到 `src/assets/`，使用者手動建立/選擇日期子資料夾（CMS 介面有資料夾 navigator）
- **Obsidian 寫文章**：設定 attachment 路徑為 `src/assets/{{date:YYYY-MM-DD}}`（用今天日期，多數情境下與文章日期一致）
- **不採用 page bundle**（每篇一資料夾、圖在同資料夾）的理由：
  - Pages CMS 不支援「media folder = 當前文章資料夾」
  - 用 page bundle 會讓 CMS 上傳的圖跑到全域資料夾，之後要手動移
  - 破壞 IG-like 體驗

---

## 5. Pages CMS 設定（`.pages.yml`）

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

**設計決策：**
- `categories` 與 `authors` 嘗試用 `hidden: true` 隱藏（自動填預設值）。**待實作時驗證 Pages CMS 是否支援此欄位選項**；若不支援，退回方案：以唯讀方式顯示但帶預設值
- `tags` 用 `creatable: true`，可在預設清單外自由新增
- 中文 label 顯示在 CMS 介面，但實際存到 markdown 的 key 仍是英文（符合主題 schema）

---

## 6. 部署與網域

### Cloudflare Pages 設定

| 項目 | 值 |
|------|------|
| Repo | `chialin/beancount-notes` |
| Production branch | `main` |
| Build command | `npm run build` |
| Output directory | `dist` |
| Node version | 20+ |
| 環境變數 | 無 |
| Preview deploy | 自動（Pull Request 產生預覽連結） |

**成本：** $0（在 Cloudflare 免費額度內）

### DNS 設定（Porkbun）

在 Porkbun DNS 管理頁面新增一筆 CNAME：

```
Type:  CNAME
Host:  beancount
Value: <project>.pages.dev    # 由 Cloudflare Pages 產生
TTL:   600
```

並在 Cloudflare Pages 後台「Custom domains」加入 `beancount.chialin.me`，Cloudflare 會自動驗證並核發 TLS 憑證。

**chialin.me 的 nameserver 不動，DNS 管理留在 Porkbun。**

### 舊 blog 處理

- **目前：保留現狀**（`blog.chialin.me` 不動）
- **未來：另開專案遷移到 Astro**（不在此 spec 範圍）
- 兩個 blog 透過 footer 互相連結（手動加 link）

---

## 7. Obsidian 整合

### 設定步驟

1. 本機 `git clone https://github.com/chialin/beancount-notes.git`
2. Obsidian 開啟此資料夾為 vault
3. 安裝 community plugin：**Obsidian Git**
4. 設定 Obsidian Git：
   - Auto pull interval：5 分鐘
   - Auto push（commit on save）：每 N 分鐘
5. 設定 Obsidian attachment 路徑：
   - Files & Links → Default location for new attachments → `src/assets/{{date:YYYY-MM-DD}}`
6. 設定 Obsidian 隱藏無關資料夾：
   - 在 `.obsidian/app.json` 隱藏 `node_modules/`、`dist/`、`.astro/` 等

### 同步策略：實作時決定

由於 GitHub Actions **無法主動推檔案到本機電腦**（網路安全限制），同步必須由本機主動拉。可選機制（**實作階段再決定採用哪個或組合**）：

| 機制 | 特性 |
|------|------|
| **Obsidian Git plugin** | 跑在 Obsidian 內，每 N 分鐘 auto pull/push。需開著 Obsidian。設定簡單 |
| **macOS launchd 排程** | 系統層 cron，定時跑 `git pull && git push`，不需開 Obsidian。較穩，但要寫 plist |
| **手動** | 想到再 `git pull` / `git push`。完全可控但容易忘 |

可單獨用一個，也可組合（例：Obsidian Git plugin 即時 + launchd 每小時保險）。**實作時依實際使用感受決定。**

### 工作流

- **手機寫**：開瀏覽器 → Pages CMS → 寫 → 存 → CMS 自動 commit → 30 秒上線
- **桌面寫**：開 Obsidian → 寫 → Obsidian Git 自動 push → 30 秒上線
- 兩邊改的都是同一份 `.md` 檔案，**沒有同步衝突問題**（除非同時編輯同一篇）

### 為什麼 GitHub Actions 不適合做這個同步？

- GitHub Actions 跑在 GitHub 雲端，**單向只能往 GitHub 推**
- 要把檔案從 GitHub 推到你的電腦，需要你的電腦對外開埠（不安全）或用 self-hosted runner（過度工程）
- 同步本來就是 pull-based 比較自然

---

## 8. 額外決定

| 項目 | 決定 |
|------|------|
| **RSS feed** | ✅ 開啟（Bookworm Light 內建） |
| **Sitemap** | ✅ 開啟（Astro 內建） |
| **Analytics** | ❌ 不裝 |
| **留言系統** | ❌ 不裝 |
| **OG image** | ✅ 開啟（Bookworm Light 內建） |

---

## 9. Setup Roadmap

實作會依序進行（每步詳細任務在 implementation plan）：

1. **Step 1**：建 GitHub repo `chialin/beancount-notes`
2. **Step 2**：用 Bookworm Light 主題初始化（fork or 用 starter 命令）
3. **Step 3**：客製化主題（站名、tag 預設、字體微調、刪掉用不到的範例內容）
4. **Step 4**：寫 `.pages.yml` 設定 Pages CMS
5. **Step 5**：在 Cloudflare Pages 連 repo、設定 build
6. **Step 6**：在 Porkbun 加 CNAME，Cloudflare Pages 加 custom domain
7. **Step 7**：在 [app.pagescms.org](https://app.pagescms.org) 連 GitHub repo、驗證設定
8. **Step 8**：寫第一篇「hello world」測整個 pipeline（CMS → commit → build → deploy）
9. **Step 9**：本機 clone repo、設定 Obsidian vault；同步機制（Obsidian Git plugin / launchd / 手動）實作時依使用感受決定
10. **Step 10**：手機開 Pages CMS、實際走一次「貼圖 → 寫文章 → 發布」流程

---

## 10. 風險與緩解

| 風險 | 機率 | 影響 | 緩解 |
|------|------|------|------|
| Bookworm Light 主題未來停更 | 低 | 中 | Astro markdown 可移植，換主題不影響內容 |
| Pages CMS 雲端版停運 | 低 | 中 | 可改自架（同 codebase，免費） |
| Obsidian Git plugin 衝突（手機 CMS 同時編輯同檔） | 低 | 低 | 工作流上避免同時編輯，git 衝突也可手動解 |
| Cloudflare Pages 免費額度超出 | 極低 | 低 | 個人 blog 流量遠低於上限 |
| 圖片 repo 越來越肥 | 中 | 中 | 超過 1GB 再考慮搬外部圖床（R2 / Cloudinary） |

---

## 11. Future Work / Out of Scope

以下**不在此 spec 範圍**，待新 blog 上線、流程穩定後再評估：

- **舊 blog（`blog.chialin.me`）遷移到 Astro**：獨立專案，需另開 spec
- **兩個 blog 整合到單一 Astro monorepo**：等兩邊都跑順再評估
- **Image optimization / 外部圖床**：等 repo 圖片量真的肥到不行再做
- **多語言（i18n）**：目前只寫繁體中文
- **進階 SEO**（結構化資料、Open Graph 細調）：基本款先用，之後再優化

---

## 12. 驗收標準

完成此 spec 對應的實作後，應能：

- [ ] 在手機瀏覽器打開 [app.pagescms.org](https://app.pagescms.org)，30 秒內貼圖 + 寫一段文字 + 發布
- [ ] 30–60 秒後，內容出現在 `https://beancount.chialin.me`
- [ ] 桌面打開 Obsidian，看到剛剛從手機寫的文章與圖片
- [ ] Obsidian 改了一段文字，5 分鐘內自動同步到網站
- [ ] `bean-check` 等既有工作流不受影響（新 repo 完全獨立）

---

**End of design.**
