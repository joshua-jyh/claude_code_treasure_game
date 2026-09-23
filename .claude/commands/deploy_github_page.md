## description: 檢查 GitHub 環境、建立 Repository，並將目前專案部署到 GitHub Pages
allowed-tools: Bash, Read, Write, Edit, Glob, Grep

# Deploy GitHub Page

請將目前所在的專案完整部署到 GitHub Repository 與 GitHub Pages。

這個指令的目標是讓使用者只需要執行：

```text
/deploy_github_page
```

就可以完成：

1. 檢查 Git 專案
1. 檢查 GitHub CLI
1. 檢查 GitHub 登入狀態
1. 檢查或建立 GitHub Repository
1. 判斷專案類型
1. 處理 GitHub Pages 路徑
1. Build 專案
1. 建立 GitHub Actions workflow
1. Commit
1. Push
1. 啟用 GitHub Pages
1. 確認 Deployment
1. 回報 Repository 與 Pages 網址

請實際執行需要的指令，而不是只告訴使用者如何操作。

# 1. 檢查 Git 專案

先執行：

```bash
git rev-parse --is-inside-work-tree
```

如果成功並回傳：

```text
true
```

代表目前已經是 Git Repository，繼續下一步。

如果不是 Git Repository：

1. 告知使用者目前專案尚未初始化 Git。
2. 執行：

```bash
git init
```

3. 再次確認：

```bash
git status
```

4. 繼續後續流程。

---

# 2. 檢查 GitHub CLI

執行：

```bash
gh --version
```

如果可以取得版本資訊，繼續下一步。

如果找不到 `gh`：

1. 告知使用者尚未安裝 GitHub CLI。
2. 根據目前作業系統提供 GitHub CLI 官方安裝方式。
3. Windows 可以引導使用者下載 GitHub CLI 官方 MSI 安裝程式。
4. 安裝完成後要求使用者重新開啟 Terminal。
5. 再執行：

```bash
gh --version
```

在 GitHub CLI 可以正常執行以前，不要繼續 Repository 建立與 Pages 部署。

---

# 3. 檢查 GitHub 登入狀態

執行：

```bash
gh auth status
```

如果已經登入 GitHub，記錄目前登入的 GitHub username，繼續下一步。

如果尚未登入：

1. 告知使用者需要登入 GitHub。
2. 執行或引導執行：

```bash
gh auth login
```

建議選擇：

```text
GitHub.com
HTTPS
Login with a web browser
```

登入完成之後再次執行：

```bash
gh auth status
```

只有確認登入成功後才可以繼續。

可以使用：

```bash
gh api user --jq .login
```

取得目前 GitHub username。

---

# 4. 檢查 GitHub Repository

執行：

```bash
git remote -v
```

## 已存在 origin

如果已經存在名為 `origin` 的 GitHub remote：

1. 取得 remote URL：

```bash
git remote get-url origin
```

2. 確認該 Repository 屬於目前登入的 GitHub 使用者，或至少確認使用者有 push 權限。
3. 不要重複建立 Repository。
4. 從 remote URL 取得 Repository 名稱。
5. 繼續部署。

## 不存在 origin

如果沒有 `origin`：

1. 取得目前資料夾名稱，作為 Repository 名稱的預設值。
2. 告知使用者目前沒有 GitHub Repository。
3. 如果使用者沒有指定其他 Repository 名稱，直接使用目前資料夾名稱。
4. Repository 預設建立為 public。
5. 執行：

```bash
gh repo create <REPOSITORY_NAME> --public --source=. --remote=origin
```

6. 再執行：

```bash
git remote -v
```

確認 `origin` 已經成功建立。

如果 Repository 名稱已經存在：

1. 不要直接覆蓋。
2. 檢查該 Repository 是否屬於目前使用者。
3. 如果可以使用，就設定為 origin。
4. 如果有名稱衝突，告知使用者並要求或產生另一個合理名稱。

---

# 5. 檢查專案類型

檢查專案中的檔案，例如：

```text
package.json
package-lock.json
yarn.lock
pnpm-lock.yaml
vite.config.js
vite.config.ts
src/
public/
index.html
```

如果存在 `package.json`，必須讀取內容。

判斷：

* framework
* build tool
* package manager
* build script
* build output directory

例如：

如果 `package.json` 包含：

```json
"scripts": {
  "build": "vite build"
}
```

則判斷為 Vite 專案。

如果使用：

```text
@vitejs/plugin-react
```

或：

```text
@vitejs/plugin-react-swc
```

則判斷為 React + Vite。

---

# 6. 判斷 Build Output Directory

不要直接假設所有 Vite 專案都輸出到 `dist`。

如果存在：

```text
vite.config.ts
```

或：

```text
vite.config.js
```

必須讀取設定。

檢查是否有：

```javascript
build: {
  outDir: '...'
}
```

如果有設定 `outDir`，使用該目錄作為 GitHub Pages artifact。

例如：

```javascript
build: {
  outDir: 'build'
}
```

則：

```text
BUILD_OUTPUT=build
```

如果 Vite config 沒有設定 `outDir`，才使用 Vite 預設：

```text
dist
```

如果不是 Vite 專案，根據使用的 framework 或 build tool 判斷正確的輸出目錄。

---

# 7. 設定 GitHub Pages Base Path

先取得 Repository 名稱。

可以透過：

```bash
git remote get-url origin
```

或：

```bash
gh repo view --json name --jq .name
```

取得。

一般 GitHub Project Pages 網址格式為：

```text
https://<USERNAME>.github.io/<REPOSITORY_NAME>/
```

因此 Vite 專案需要正確的 `base`。

如果是 Vite 專案，檢查：

```text
vite.config.ts
```

或：

```text
vite.config.js
```

是否已設定：

```javascript
base: '/<REPOSITORY_NAME>/'
```

例如 Repository 名稱為：

```text
treasure_game
```

則：

```javascript
base: '/treasure_game/'
```

如果沒有 `base`：

1. 修改 Vite config。
2. 加入正確的 Repository base path。
3. 保留原本所有其他設定。
4. 不要破壞 plugins、resolve、alias、build、server 等既有設定。

例如原本：

```javascript
export default defineConfig({
  plugins: [react()],
})
```

應調整成：

```javascript
export default defineConfig({
  base: '/treasure_game/',
  plugins: [react()],
})
```

如果 Repository 是特殊的 User Pages Repository：

```text
<USERNAME>.github.io
```

則 base 應為：

```javascript
base: '/'
```

---

# 8. 檢查 Package Manager

根據 lock file 判斷：

如果存在：

```text
package-lock.json
```

使用 npm。

如果存在：

```text
pnpm-lock.yaml
```

使用 pnpm。

如果存在：

```text
yarn.lock
```

使用 yarn。

不要在已經明確存在 package manager lock file 的情況下任意切換 package manager。

---

# 9. 安裝 Dependencies 與 Build

如果是 npm 專案：

優先執行：

```bash
npm install
npm run build
```

如果適合且 lock file 完整，也可以使用：

```bash
npm ci
npm run build
```

如果是 pnpm：

```bash
pnpm install
pnpm build
```

如果是 yarn：

```bash
yarn install
yarn build
```

Build 完成後確認輸出目錄存在。

例如：

```bash
ls build
```

或：

```bash
ls dist
```

依照前面判斷出的 build output directory 為準。

如果 build 失敗：

1. 閱讀錯誤訊息。
2. 找出與 build 或部署直接相關的問題。
3. 修正問題。
4. 再次 build。
5. 確認成功後才可以繼續。

絕對不可以在 build 失敗的情況下宣稱部署完成。

---

# 10. 建立 GitHub Pages Workflow

建立：

```text
.github/workflows/deploy-pages.yml
```

如果資料夾不存在，建立：

```text
.github/workflows/
```

Workflow 必須依照目前專案實際狀況產生。

需要取得目前 branch：

```bash
git branch --show-current
```

使用目前 branch 作為 GitHub Actions trigger。

例如目前 branch 是：

```text
initial
```

則 workflow：

```yaml
name: Deploy GitHub Pages

on:
  push:
    branches:
      - initial
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v6

      - name: Setup Node
        uses: actions/setup-node@v6
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v4
        with:
          path: ./<BUILD_OUTPUT>

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

其中：

```text
<BUILD_OUTPUT>
```

必須替換成實際的 build output directory。

例如 Vite config 是：

```javascript
build: {
  outDir: 'build'
}
```

則 workflow 必須使用：

```yaml
with:
  path: ./build
```

不可以寫成 `./dist`。

如果不是 npm 專案，調整 Setup Node / install / build 指令。

---

# 11. 檢查敏感資料

Commit 前執行：

```bash
git status
```

檢查準備提交的內容。

確認不會加入：

```text
node_modules
.env
.env.local
.env.production
API keys
access tokens
passwords
private keys
credentials
```

檢查 `.gitignore`。

Node.js 專案至少應避免提交：

```text
node_modules/
```

Build output 是否加入 Git，可以依專案需求判斷。

使用 GitHub Actions Pages deployment 時，通常不需要把 build output commit 到 Repository。

---

# 12. Commit

先執行：

```bash
git status
```

然後：

```bash
git add .
```

再次查看：

```bash
git status
```

確認內容合理之後 commit：

```bash
git commit -m "Deploy project to GitHub Pages"
```

如果 Git 回報：

```text
nothing to commit
```

代表目前沒有新的變更，不要因此中止部署流程。

---

# 13. Push 到 GitHub

取得目前 branch：

```bash
git branch --show-current
```

例如：

```text
initial
```

則執行：

```bash
git push -u origin initial
```

不要假設 branch 一定是 `main`。

Push 後確認：

```bash
gh repo view --web
```

或：

```bash
gh repo view
```

確認 GitHub Repository 已存在而且程式碼已成功上傳。

---

# 14. 啟用 GitHub Pages

GitHub Pages 必須使用 GitHub Actions 作為 build/deployment source。

先確認 Pages 狀態。

可以使用：

```bash
gh api repos/{owner}/{repo}/pages
```

如果 GitHub Pages 尚未建立，可以嘗試：

```bash
gh api \
  --method POST \
  repos/{owner}/{repo}/pages \
  -f build_type=workflow
```

如果 Pages 已存在但不是 workflow 模式，可以嘗試：

```bash
gh api \
  --method PUT \
  repos/{owner}/{repo}/pages \
  -f build_type=workflow
```

如果第一次呼叫 Pages API 因為 Repository 尚未 push 或 workflow 尚未存在而失敗：

1. 先確認程式碼已經 push。
2. 確認 `.github/workflows/deploy-pages.yml` 已在遠端 Repository。
3. 再次設定 Pages。

如果 GitHub API 回應代表 Pages 已正確設定為 workflow，不要重複建立。

---

# 15. 檢查 GitHub Actions

執行：

```bash
gh run list
```

找出：

```text
Deploy GitHub Pages
```

相關 workflow。

如果需要取得 workflow：

```bash
gh workflow list
```

如果 workflow 已開始執行，使用：

```bash
gh run watch
```

或指定 run id：

```bash
gh run watch <RUN_ID>
```

等待 deployment 結果。

---

# 16. Deployment 失敗時

如果 GitHub Actions 失敗：

執行：

```bash
gh run view <RUN_ID>
```

必要時：

```bash
gh run view <RUN_ID> --log-failed
```

找出失敗原因。

常見問題包括：

* build output 路徑錯誤
* npm dependencies 安裝失敗
* Vite base 設定錯誤
* GitHub Pages 未設定為 GitHub Actions
* workflow permissions 不足
* branch trigger 錯誤
* Node.js 版本問題

修正之後：

```bash
git add .
git commit -m "Fix GitHub Pages deployment"
git push
```

再重新確認 workflow。

不可以在 workflow 為 failure 時回報部署成功。

---

# 17. 驗證 GitHub Repository

取得目前 Repository：

```bash
gh repo view --json url --jq .url
```

確認可以取得：

```text
https://github.com/<USERNAME>/<REPOSITORY_NAME>
```

這個網址必須可以看到目前專案的程式碼。

---

# 18. 驗證 GitHub Pages URL

取得 username：

```bash
gh api user --jq .login
```

取得 Repository 名稱：

```bash
gh repo view --json name --jq .name
```

一般 Project Pages URL 為：

```text
https://<USERNAME>.github.io/<REPOSITORY_NAME>/
```

也可以透過 GitHub Pages API 或 deployment workflow output 取得實際 Pages URL。

確認 deployment workflow 成功後，回報該網址。

---

# 19. 最終結果

只有在以下條件都成立時才可以宣稱部署完成：

* GitHub 登入成功
* GitHub Repository 存在
* origin 設定完成
* 專案 build 成功
* GitHub Actions workflow 已建立
* 程式碼已 push
* GitHub Pages 已設定
* Deployment workflow 成功

成功後輸出：

```text
部署完成！

GitHub Repository:
https://github.com/<USERNAME>/<REPOSITORY_NAME>

GitHub Pages:
https://<USERNAME>.github.io/<REPOSITORY_NAME>/
```

另外簡短列出：

```text
GitHub 登入：成功
Repository：成功
Build：成功
Push：成功
GitHub Pages：成功
```

提醒使用者開啟 GitHub Pages URL，確認網站畫面正常顯示。

---

# 執行原則

執行 `/deploy_github_page` 時：

1. 優先自行檢查環境，不要一開始就要求使用者提供已經可以透過指令取得的資訊。
2. 可以自動完成的步驟就直接執行。
3. 涉及 GitHub 登入等需要使用者操作的步驟，再清楚引導使用者。
4. 不要假設 Repository 已經存在。
5. 不要假設 branch 一定叫 `main`。
6. 不要假設 Vite build output 一定是 `dist`。
7. 不要假設 Repository 名稱。
8. 不要破壞既有 Vite 設定。
9. 不要提交敏感資訊。
10. 不要在 build 或 deployment 失敗時宣稱成功。
11. 最終目標必須是讓 GitHub Repository 看得到專案，並且 GitHub Pages URL 可以正常開啟網站。
