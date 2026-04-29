# beancount-notes

Beancount 記帳手記 — 個人知識庫，公開於 [beancount-notes.chialin.me](https://beancount-notes.chialin.me)。

## 架構

- **Astro 6 + [Bookworm Light](https://github.com/themefisher/bookworm-light-astro) theme**：靜態網站
- **[Pages CMS](https://pagescms.org)**：[app.pagescms.org](https://app.pagescms.org) 線上編輯介面（手機友善）
- **Cloudflare Pages**：`main` push 自動 build & deploy
- **DNS**：Porkbun（CNAME `beancount-notes` → `beancount-notes.pages.dev`）

## 寫作介面

| 介面 | 用途 |
|------|------|
| Pages CMS | 主要寫作介面，特別是手機 |
| Obsidian | 桌面查閱與編輯，本機 clone repo 當 vault |
| 直接編輯 markdown | 任何文字編輯器都行 |

## 內容結構

- `src/content/posts/` — 文章（檔名 `YYYY-MM-DD-{slug}.md`）
- `src/content/authors/` — 作者
- `src/content/about/` `src/content/pages/` — 靜態頁面
- `public/images/` — 圖片（CMS 上傳目的地，frontmatter 用 `/images/...` 引用）

## 路由

- `/` — 首頁，列出最新文章
- `/blog/` — 文章列表
- `/blog/{post-id}/` — 單篇文章（`post-id` 即檔名去除副檔名）
- `/tags/` `/categories/` `/authors/` — 分類索引

## 本地開發

```bash
yarn install
yarn dev      # http://localhost:4321
yarn build    # 產出 dist/
```

需要 Node.js 20+。專案 lockfile 為 `yarn.lock`（`package-lock.json` 已加入 `.gitignore`）。

## 設計與計劃文件

- Spec：[docs/superpowers/specs/2026-04-28-beancount-blog-design.md](docs/superpowers/specs/2026-04-28-beancount-blog-design.md)
- Implementation plan：[docs/superpowers/plans/2026-04-28-beancount-blog.md](docs/superpowers/plans/2026-04-28-beancount-blog.md)

## License

內容（`src/content/`）為作者個人著作，保留所有權利。

主題程式碼基於 [Bookworm Light Astro](https://github.com/themefisher/bookworm-light-astro)（MIT License），詳見 [LICENSE](LICENSE)。
