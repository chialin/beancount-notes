---
title: Beancount 常用 CLI 工具速查
meta_title: bean-check / bean-query / bean-format / fava
description: 整理 beancount 安裝後內建的幾個 CLI 工具與 fava，包含驗證、查詢、格式對齊與網頁介面的常用指令。
date: 2026-04-28
tags: [基礎, 工具]
categories: [notes]
authors: [chialin]
draft: false
---

裝好 beancount 之後（`pip install beancount`），會多出一組 `bean-` 開頭的命令列工具。這篇整理日常記帳會用到的四個。

## bean-check — 驗證帳本

最常用、也最重要的工具。會把你的 `.beancount` 檔讀進去，檢查有沒有語法錯誤、帳戶是否都有 `open`、複式記帳是否平衡。

```bash
bean-check main.beancount
```

順利的話，不會有任何錯誤訊息顯示。
出錯時會把檔案路徑、行號、原因一條一條列出來，可以根據那一個交易來進行確認錯誤原因。

我自己的節奏是寫完一小段就 `bean-check` 一下，剛改的內容還在腦袋裡，看到報錯比較好對應到剛才動了什麼。等到累積二三十筆再一次驗，常常會卡在「到底是哪一邊的錯」，常常會找半天。

## bean-query — 用 SQL 查帳本

beancount 內建一個類 SQL 的查詢介面，叫 BQL（Beancount Query Language）。可以互動式進入：

```bash
bean-query main.beancount
```

或一次性執行單一查詢：

```bash
bean-query main.beancount "SELECT account, sum(position) WHERE account ~ 'Expenses:Food' GROUP BY account"
```

常用情境：

- 看某類別的總支出
- 篩選特定商家（例如所有「全聯」的消費）的所有交易
- 依月份統計收支

進到互動模式後輸入 `help` 可以看可用語法。完整文件：[Beancount Query Language](https://beancount.github.io/docs/beancount_query_language.html)。

## bean-format — 對齊欄位

手寫帳本時數字與貨幣很容易亂跑，看起來沒有對齊。`bean-format` 會把每一行的金額靠右對齊：

```bash
bean-format main.beancount > main.formatted.beancount
```

或直接覆蓋原檔（先 commit 過再做）：

```bash
bean-format -o main.beancount main.beancount
```

很多編輯器（VS Code、Vim、Emacs）都有 beancount plugin，存檔時自動跑 format，不一定要手動。

## fava — 網頁介面

`fava` 是另外裝的套件，提供網頁介面總覽資產的相關資訊：

```bash
pip install fava
fava main.beancount
```

預設開在 [http://localhost:5000](http://localhost:5000)，提供：

- 收支報表、淨值變化圖
- 餘額試算表（Trial Balance）、損益表（Income Statement）、資產負債表（Balance Sheet）
- 各帳戶的明細與圖表
- 互動式 BQL 查詢介面

## 整理

| 工具          | 用途     | 何時用       |
| ------------- | -------- | ------------ |
| `bean-check`  | 驗證     | 每次寫完一段 |
| `bean-query`  | SQL 查詢 | 寫客製化報表 |
| `bean-format` | 對齊欄位 | 整理格式     |
| `fava`        | 網頁報表 | 看圖、明細   |
