# Cloudflare Pages Custom Domain — 兩條 add 路徑與正確選法

> **記錄日期：** 2026-04-28
> **情境：** 把 `beancount-notes.chialin.me` 接到 Cloudflare Pages 專案；DNS 主管在 Porkbun，不想搬整個 `chialin.me` zone 到 Cloudflare。

---

## TL;DR

- Cloudflare 有**兩個入口**都叫「加自訂網域」，但行為差很多。
- **錯誤入口**：Cloudflare 帳號頂部 `Domains` 列表頁 → `Add domain` → 進入「Add a site」zone 精靈 → 逼你改 registrar 的 nameservers → 影響整個 zone。
- **正確入口**：Pages 專案內 → `Custom domains` 分頁 → `Set up a custom domain` → 選 **`My DNS provider` → `Begin CNAME setup`** → 只動一個子網域，不碰其他既有服務。

---

## 兩個入口的位置

### A. Domains 列表頁的 `Add domain`（誤入）

路徑：Cloudflare Dashboard 頂層 → `Domains`（左側選單）→ 右上 `Add domain`

行為：開啟 **Add a site / zone-onboarding 精靈**，預設假設你要把**整個 domain 的 DNS 管理權**搬到 Cloudflare。最後一步要求你到 registrar 把 nameservers 改成 Cloudflare 提供的兩台。

副作用：

- 影響整個 zone 的所有 records（其他子網域 / email MX / TXT / 第三方服務 CNAME 全部都會因為 NS 切換改由 Cloudflare 回應）
- 必須事先逐筆比對 import 結果完整、無遺漏
- DNS propagation 1–24 小時，期間可能新舊 NS 並行回應

如果不小心進到這個流程但還沒按下 `Save` 改 NS，**安全做法是直接刪除這個 zone**（在 `Domains` 列表 → 點 `chialin.me` → Overview 頁底部 `Remove Site` / 三點選單 `Remove`）。Porkbun 那邊的 NS 沒動就不會出事。

### B. Pages 專案內 `Custom domains` 分頁（正確）

路徑：`Workers & Pages` → 點目標 Pages 專案 → 上方分頁切到 `Custom domains` → `Set up a custom domain`

輸入要綁的子網域（例：`beancount-notes.chialin.me`）→ Continue 後會出現 **Setup Method** 選擇畫面：

```
┌─────────────────────────┐  ┌─────────────────────────┐
│   Cloudflare DNS        │  │   My DNS provider       │
│                         │  │                         │
│  Transfer your DNS to   │  │  Use your current DNS   │
│  Cloudflare. Once the   │  │  provider. Set up a     │
│  transfer is complete,  │  │  CNAME record to point  │
│  you'll be able to add  │  │  your domain to         │
│  this Custom Domain to  │  │  Cloudflare.            │
│  your Pages project.    │  │                         │
│                         │  │                         │
│  [Begin DNS transfer]   │  │  [Begin CNAME setup]    │
└─────────────────────────┘  └─────────────────────────┘
```

| 選項 | 等同於 | 結果 |
|---|---|---|
| **Cloudflare DNS** → `Begin DNS transfer` | 等同於 A 入口的 zone 搬移流程 | 整個 zone 由 Cloudflare 接管，要改 registrar 的 NS |
| **My DNS provider** → `Begin CNAME setup` | 用你既有的 DNS provider（Porkbun / GoDaddy / etc.）的 CNAME | 只動一個子網域，其餘不影響 |

---

## 什麼時候選哪個

| 場景 | 建議選擇 |
|---|---|
| 只想把一個子網域接到 Cloudflare Pages，其他既有服務（blog、email、第三方）都不想動 | **My DNS provider** |
| 想把整個 zone 統一到 Cloudflare（速度、分析、Cloudflare proxy 功能） | Cloudflare DNS |
| 已經有第三方 DNS 在管（如 Vercel CNAME、AWS Route 53 部分子網域） | **My DNS provider**（避免雙重管理衝突） |
| 個人 / 小站，registrar 自帶 DNS 已夠用 | **My DNS provider** |

對 `chialin.me` 來說，因為已有 `blog.chialin.me`（Vercel）、`beancount.chialin.me`（既有 Pages）、email MX 等，整個搬過來代價遠大於收益 — 選 **My DNS provider**。

---

## 完整操作順序（已驗證走通）

1. **registrar 端（Porkbun）** 加 CNAME：
   - Type: `CNAME`
   - Host: `beancount-notes`（即子網域）
   - Answer: `<pages-project-name>.pages.dev`
   - TTL: `600`

2. `dig beancount-notes.chialin.me CNAME +short` 確認 propagate（通常幾十秒）

3. Cloudflare Pages 專案 → `Custom domains` → `Set up a custom domain`

4. 輸入 `beancount-notes.chialin.me` → Continue

5. Setup Method 選 **`My DNS provider`** → `Begin CNAME setup`

6. Cloudflare 偵測既有 CNAME 通過 → 進 `Verifying` → `Active`（1–5 分鐘核發 TLS）

7. `curl -I https://beancount-notes.chialin.me` 預期 `HTTP/2 200` + Cloudflare headers

---

## 為什麼 Workers 沒這條路

對應驗證：本案最初用 Cloudflare **Workers Builds**（含 `wrangler.jsonc`）部署，加 custom domain 時報錯：

> `Only domains active on your Cloudflare account can be added.`

Workers 的 Custom Domain / Workers Routes 都**強制要求 zone 在 Cloudflare DNS 上**，不接受外部 DNS 的 CNAME。所以一開始的 Workers 流程到 custom domain 這步就卡住，必須改用 Pages 重做。

→ **若需要外部 DNS 的 custom domain，部署平台只能選 Cloudflare Pages，不能選 Workers。**

---

## 善後 / 排錯

- 誤建的 zone 在 `Domains` 列表會顯示 **`Invalid nameservers`** 狀態（因為沒真的改 NS）。安全刪除即可，不影響 registrar 端 DNS。
- 若 Custom Domain 卡在 `Verifying` 超過 10 分鐘 → 多半是 CNAME 還沒 propagate，或 Answer 寫錯成 Worker URL 而非 Pages URL。`dig` 一下對照 Pages 專案首頁顯示的 `*.pages.dev` 網址。
- 同個 repo 同時開 Workers + Pages 兩個專案會兩邊都自動 build，浪費資源。決定走 Pages 後，記得到 Workers & Pages 列表把舊的 Worker 刪掉。
