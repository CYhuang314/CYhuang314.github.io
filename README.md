# CYhuang's Blog

使用 [Hexo 8](https://hexo.io/) + [Fluid](https://hexo.fluid-dev.com/) 主題建立的個人部落格，
透過 GitHub Actions 自動建置並部署到 GitHub Pages。

- 線上網址：<https://cyhuang314.github.io/>
- 原始碼倉庫：<https://github.com/CYhuang314/CYhuang314.github.io>

---

## 目錄

- [環境需求](#環境需求)
- [第一次使用（或換一台電腦）](#第一次使用或換一台電腦)
- [寫文章的完整流程](#寫文章的完整流程)
- [Front matter 欄位說明](#front-matter-欄位說明)
- [草稿功能](#草稿功能)
- [插入圖片](#插入圖片)
- [專案結構](#專案結構)
- [部署機制](#部署機制)
- [常見問題與解法](#常見問題與解法)
- [常用指令速查表](#常用指令速查表)

---

## 環境需求

| 項目 | 版本 | 備註 |
| --- | --- | --- |
| Node.js | **>= 20.19.0** | Hexo 8 的硬性要求，本專案用 22 |
| npm | 隨 Node 附帶 | |
| Git | 任意近期版本 | |

本專案根目錄有 `.nvmrc`（內容是 `22`），在專案資料夾執行以下指令即可切到正確版本：

```bash
nvm use
```

> ⚠️ 如果你的系統 Node 是 18，`hexo` 會出現 `Unsupported engine` 警告甚至直接失敗。
> **每次開始工作前先執行 `nvm use`。**

---

## 第一次使用（或換一台電腦）

```bash
git clone git@github.com:CYhuang314/CYhuang314.github.io.git
cd CYhuang314.github.io

# 草稿放在獨立的私人倉庫，clone 進 source/_drafts/
git clone git@github.com:CYhuang314/blog-drafts.git source/_drafts

nvm use              # 切換到 Node 22
npm install          # 安裝 hexo、主題與所有外掛
npx hexo server      # 打開 http://localhost:4000 確認可以跑
```

`node_modules/`、`public/`、`db.json` 都不會進版控，clone 下來後一定要 `npm install`。
草稿倉庫的部分請看下面的[草稿功能](#草稿功能)。

---

## 寫文章的完整流程

### 1. 建立新文章

```bash
npx hexo new "我的第一篇文章"
```

會依照 `scaffolds/post.md` 範本，在 `source/_posts/我的第一篇文章.md` 建立檔案。

> 💡 **標題含空白時務必加引號**，否則 Hexo 只會取第一個字當標題。

### 2. 編輯內容

打開 `source/_posts/我的第一篇文章.md`，補上 front matter 與正文：

```markdown
---
title: 我的第一篇文章
date: 2026-08-26 14:30:00
tags:
  - Hexo
  - 筆記
categories:
  - 技術
excerpt: 這段文字會顯示在首頁的文章摘要區。
---

這裡是正文開頭。

<!-- more -->

這行以後的內容只有點進文章才看得到。
```

### 3. 本機預覽

```bash
npx hexo server
```

瀏覽器打開 <http://localhost:4000>。
Hexo server 會即時偵測 Markdown 變更，**改完存檔重新整理就看得到**，不需要重啟。

> 但是修改 `_config.yml` 或 `_config.fluid.yml` **不會**自動生效，必須 `Ctrl + C` 停掉再重開。

### 4. 發佈上線

```bash
git add .
git commit -m "post: 新增 我的第一篇文章"
git push
```

**push 到 `main` 就會自動觸發 GitHub Actions 建置與部署**，
大約 1～2 分鐘後網站就會更新，你不需要手動執行 `hexo deploy`。

查看部署進度：

```bash
gh run list --limit 5          # 列出最近的 workflow 執行狀況
gh run watch                   # 即時盯著最新那一次跑完
```

或直接看網頁：<https://github.com/CYhuang314/CYhuang314.github.io/actions>

---

## Front matter 欄位說明

寫在文章最上方 `---` 包起來的區塊：

| 欄位 | 必填 | 說明 |
| --- | --- | --- |
| `title` | ✅ | 文章標題（網址中的 slug 是**檔名**，不是標題） |
| `date` | ✅ | 發佈時間，格式 `YYYY-MM-DD HH:mm:ss`，決定排序與網址中的日期。**一定要寫**：缺少時會用檔案建立時間，而 CI 每次部署都是新檔案，日期和網址會跟著每次部署改變 |
| `updated` | | 更新時間，不填則用檔案的修改時間（在 CI 上等於部署時間） |
| `tags` | | 標籤，可多個 |
| `categories` | | 分類，可多個（有階層關係） |
| `excerpt` | | 首頁顯示的摘要文字 |
| `index_img` | | 首頁列表縮圖（Fluid 主題專用） |
| `banner_img` | | 文章頁的頭圖（Fluid 主題專用） |
| `permalink` | | 自訂網址，覆蓋預設規則 |
| `sticky` | | 數字，數字越大越置頂 |

多個標籤的兩種寫法：

```yaml
tags: [Hexo, 前端, 筆記]
```

```yaml
tags:
  - Hexo
  - 前端
  - 筆記
```

---

## 草稿功能

還沒寫完、不想上線的文章可以放草稿匣：

```bash
npx hexo new draft "還在寫的文章"      # 建立於 source/_drafts/
npx hexo server --draft                # 預覽時才會顯示草稿
npx hexo publish "還在寫的文章"        # 寫完了，移到 source/_posts/
```

草稿不會被 `hexo generate` 產出（`_config.yml` 的 `render_drafts: false`），所以不會出現在網站上。

### ⚠️ 草稿存放在獨立的私人倉庫

本倉庫是**公開**的，所以 `source/_drafts/` 已經加進 `.gitignore`，
草稿改放在私人倉庫 <https://github.com/CYhuang314/blog-drafts> 備份。

也就是說 `source/_drafts/` 這個資料夾**本身是另一個 git repo**：

| | 公開 repo（本專案） | 私人 repo（blog-drafts） |
| --- | --- | --- |
| 位置 | 專案根目錄 | `source/_drafts/` |
| 內容 | 正式文章、設定、主題 | 未完成的草稿 |
| 誰看得到 | 所有人 | 只有你 |

**備份草稿**（在 `source/_drafts/` 資料夾裡執行）：

```bash
cd source/_drafts
git add . && git commit -m "draft: 更新草稿" && git push
cd ../..
```

**草稿寫完要發佈**時，兩個倉庫都要 commit：

```bash
npx hexo publish "還在寫的文章"        # 檔案從 _drafts 移到 _posts

git add . && git commit -m "post: 新增 還在寫的文章" && git push   # 公開 repo

cd source/_drafts
git add -A && git commit -m "draft: 發佈 還在寫的文章" && git push  # 私人 repo
cd ../..
```

**確認草稿真的沒外流**（在專案根目錄執行，輸出應該是空的）：

```bash
git status --short | grep _drafts
```

---

## 插入圖片

最簡單的做法是把圖片放在 `source/img/` 底下：

```bash
mkdir -p source/img
cp ~/Pictures/screenshot.png source/img/
```

在文章中用**絕對路徑**引用：

```markdown
![截圖說明](/img/screenshot.png)
```

> `source/` 底下的檔案在建置時會原封不動複製到網站根目錄，
> 所以 `source/img/screenshot.png` 對應到網址 `/img/screenshot.png`。

---

## 專案結構

```
.
├── _config.yml              # Hexo 站台設定（標題、網址、永久連結規則）
├── _config.fluid.yml        # Fluid 主題設定（選單、頭圖、深色模式、留言板）
├── .nvmrc                   # 指定 Node 版本 22
├── package.json             # 相依套件（主題也是套件之一）
├── .github/workflows/
│   └── deploy.yml           # GitHub Actions 自動部署設定
├── scaffolds/               # hexo new 使用的範本
│   ├── post.md
│   ├── draft.md
│   └── page.md
├── source/
│   ├── _posts/              # ★ 你的文章都放這裡
│   ├── _drafts/             # 草稿（獨立的私人 git repo，已 gitignore）
│   ├── about/index.md       # 關於頁
│   ├── tags/index.md        # 標籤頁
│   └── categories/index.md  # 分類頁
├── public/                  # 建置產物（自動產生，已 gitignore）
└── db.json                  # 建置快取（自動產生，已 gitignore）
```

**主題設定只改 `_config.fluid.yml`，絕對不要改 `node_modules/hexo-theme-fluid/` 裡的檔案**，
那裡的內容在 `npm install` 時會被覆蓋掉。

---

## 部署機制

```
本機寫文章 ──git push──> GitHub main 分支
                              │
                              ▼
                    GitHub Actions (deploy.yml)
                    1. npm ci      安裝相依套件
                    2. hexo clean  清掉舊快取
                    3. hexo generate  產生 public/
                    4. 上傳 public/ 當作 Pages artifact
                              │
                              ▼
                    https://cyhuang314.github.io/
```

重點：

- **你不需要執行 `hexo deploy`**，`_config.yml` 裡的 `deploy` 區塊是空的、也用不到。
- **`public/` 不進版控**，網站內容由 Actions 在雲端重新建置。
- 主題是 npm 套件（`hexo-theme-fluid`），**不是 git submodule**，所以 Actions 不需要處理 submodule。
- 也可以到 Actions 頁籤按 **Run workflow** 手動觸發重新部署。

---

## 常見問題與解法

### Q1. 改了設定 / 主題，但網頁完全沒變化

Hexo 會把產出的結果快取在 `db.json`，最常見的原因就是讀到舊快取。

```bash
npx hexo clean && npx hexo server
```

**只要碰過 `_config.yml`、`_config.fluid.yml`、`scaffolds/` 或換主題，就先 `hexo clean`。**

---

### Q2. 出現 `Unsupported engine` 或 hexo 指令直接報錯

Node 版本太舊（Hexo 8 需要 >= 20.19.0）。

```bash
node -v        # 確認目前版本
nvm use        # 讀取 .nvmrc 切到 22
node -v        # 再確認一次
```

如果顯示 `N/A: version "22" is not yet installed`：

```bash
nvm install 22
```

---

### Q3. 寫好的文章在網站上看不到

依序檢查：

```bash
ls source/_posts/          # 1. 檔案是不是真的在 _posts（不是 _drafts）？
head -8 source/_posts/你的文章.md   # 2. front matter 的 --- 有沒有寫對？
```

常見原因：

| 原因 | 解法 |
| --- | --- |
| 檔案還在 `source/_drafts/` | `npx hexo publish "檔名"` |
| front matter 的 `---` 少一行或有多餘空格 | 修正格式，冒號後面一定要有空格 |
| `title` 或 `date` 冒號後面沒空格 | `title:我的文章` ❌ → `title: 我的文章` ✅ |
| 快取沒清 | `npx hexo clean && npx hexo generate` |

---

### Q4. 標題有冒號、引號，YAML 解析失敗

front matter 是 YAML，特殊字元要用引號包起來：

```yaml
title: "Hexo：從零開始的部落格"     # ✅ 有冒號一定要加引號
title: Hexo：從零開始的部落格       # ❌ 會解析失敗
```

---

### Q5. 中文標題產生的網址很醜 / 有亂碼

預設永久連結是 `:year/:month/:day/:title/`，其中 `:title` 是**檔名**而不是 front matter 的標題。
`hexo new "中文標題"` 會產生中文檔名，網址就會被 URL 編碼。

建立時用 `--slug` 指定英文檔名：

```bash
npx hexo new "我的第一篇文章" --slug my-first-post
```

或在該篇文章的 front matter 指定英文網址：

```yaml
---
title: 我的第一篇文章
permalink: my-first-post/
---
```

---

### Q6. `npx hexo server` 說連接埠被佔用

```bash
npx hexo server -p 5000          # 換一個埠號
```

或找出佔用 4000 的程序砍掉：

```bash
lsof -i :4000
kill -9 <PID>
```

---

### Q7. GitHub Actions 建置失敗，錯誤訊息是 `npm ci` 相關

`npm ci` 要求 `package.json` 與 `package-lock.json` 完全一致。
如果你手動改過 `package.json` 或裝過套件卻沒有把 lockfile 一起 commit，就會失敗。

```bash
npm install                      # 重新產生 package-lock.json
git add package.json package-lock.json
git commit -m "chore: 更新相依套件"
git push
```

---

### Q8. Actions 綠燈跑完了，但網站沒更新

1. 先確認 Pages 的來源設定是 **GitHub Actions** 而不是 Deploy from a branch：

   ```bash
   gh api repos/CYhuang314/CYhuang314.github.io/pages --jq '.build_type'
   ```

   應該要輸出 `workflow`。如果輸出 `legacy`，執行：

   ```bash
   gh api -X PUT repos/CYhuang314/CYhuang314.github.io/pages -f build_type=workflow
   ```

2. 如果設定正確，多半只是**瀏覽器快取**，用無痕視窗或 `Ctrl + Shift + R` 強制重新整理。

---

### Q9. 想調整主題外觀（選單、頭圖、深色模式、留言板）

全部都在 `_config.fluid.yml`，改完記得清快取重開：

```bash
npx hexo clean && npx hexo server
```

常用位置：

| 想改什麼 | 找哪個區塊 |
| --- | --- |
| 導覽列選單項目 | `navbar.menu` |
| 首頁大圖 | `index.banner_img` |
| 首頁副標題打字動畫 | `index.slogan.text` |
| 關於頁的頭像與自我介紹 | `about.avatar` / `about.name` / `about.intro` |
| 深色模式 | `dark_mode` |
| 留言板（Giscus / Waline 等） | `post.comments` 與對應的區塊 |
| 頁尾文字 | `footer.content` |

完整選項請參考 [Fluid 官方文件](https://hexo.fluid-dev.com/docs/)。

---

### Q10. 標籤頁 / 分類頁 / 關於頁顯示 404

這三頁需要實際的頁面檔案，且 `layout` 要指定正確：

```bash
npx hexo new page tags        # 然後在 index.md 加上 layout: tags
npx hexo new page categories  # layout: categories
npx hexo new page about       # layout: about
```

本專案已經建好了，檔案在 `source/tags/index.md` 等位置。

---

### Q11. 想升級 Hexo 或主題

```bash
npm outdated                  # 看看有哪些套件可以更新
npm update                    # 更新到 package.json 允許的最新版本
npx hexo clean && npx hexo server    # 本機確認沒壞掉再 push
```

升級主題後，`_config.fluid.yml` 可能缺少新版本的設定項，
可以跟 `node_modules/hexo-theme-fluid/_config.yml` 比對差異：

```bash
diff _config.fluid.yml node_modules/hexo-theme-fluid/_config.yml
```

---

### Q12. 不小心把 `public/` 或 `node_modules/` commit 進去了

```bash
git rm -r --cached public node_modules
git commit -m "chore: 移除不該進版控的目錄"
git push
```

`.gitignore` 已經設定好，正常情況不會發生。

---

## 常用指令速查表

| 目的 | 指令 |
| --- | --- |
| 切換 Node 版本 | `nvm use` |
| 新增文章 | `npx hexo new "標題"` |
| 新增草稿 | `npx hexo new draft "標題"` |
| 草稿轉正式文章 | `npx hexo publish "標題"` |
| 備份草稿到私人倉庫 | `cd source/_drafts && git add . && git commit -m "訊息" && git push && cd ../..` |
| 新增獨立頁面 | `npx hexo new page "頁面名"` |
| 本機預覽 | `npx hexo server` |
| 本機預覽（含草稿） | `npx hexo server --draft` |
| 換連接埠預覽 | `npx hexo server -p 5000` |
| 清除快取 | `npx hexo clean` |
| 產生靜態檔 | `npx hexo generate` |
| 清快取後重新產生 | `npx hexo clean && npx hexo generate` |
| 發佈上線 | `git add . && git commit -m "訊息" && git push` |
| 查看部署狀態 | `gh run list --limit 5` |
| 盯著部署跑完 | `gh run watch` |

> `package.json` 也有對應的捷徑：`npm run server`、`npm run build`、`npm run clean`。
