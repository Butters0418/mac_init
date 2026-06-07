# macOS New Machine Setup

這個 repo 用來在新 Mac 上快速還原工作環境。預期流程是先安裝 Homebrew，再用 `Brewfile` 安裝常用工具，最後手動完成 Rime、iTerm2、Oh My Zsh、GitHub 帳號與 VS Code Copilot 設定。

## 會安裝的項目

- Rime / 鼠鬚管：Homebrew cask `squirrel-app`
- VS Code：Homebrew cask `visual-studio-code`
- GitHub Copilot：VS Code extension `github.copilot`（會自動帶入 Copilot Chat）
- iTerm2：Homebrew cask `iterm2`
- Oh My Zsh 相關：`powerlevel10k`、Meslo Powerlevel10k 字型、`zsh-autosuggestions`、`zsh-syntax-highlighting`
- Git：Homebrew formula `git`
- GitHub CLI：Homebrew formula `gh`
- Rectangle：Homebrew cask `rectangle`
- Node.js 版本管理：nvm（用官方安裝腳本，非 Homebrew）

## 1. 安裝 Homebrew

(教學)[https://www.josean.com/posts/terminal-setup]
先安裝 Xcode Command Line Tools：

```sh
xcode-select --install
```

再安裝 Homebrew：

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安裝完成後依照 Homebrew 終端機提示把 `brew` 加到 shell path。Apple Silicon 通常是：

```sh
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Intel Mac 通常是：

```sh
echo 'eval "$(/usr/local/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/usr/local/bin/brew shellenv)"
```

## 2. 用 Brewfile 安裝工具

在這個 repo 根目錄執行：

```sh
brew bundle --file ./Brewfile
```

安裝完可以檢查：

```sh
brew bundle check --file ./Brewfile
```

如果 VS Code extension 安裝失敗，通常是 `code` CLI 還沒進 PATH。開啟 VS Code 後執行 Command Palette，選 `Shell Command: Install 'code' command in PATH`，再重跑：

```sh
brew bundle --file ./Brewfile
```

## 3. 還原 Rime 設定

Homebrew 安裝完 `squirrel-app` 後，先到 macOS：

```text
System Settings > Keyboard > Text Input > Input Sources
```

加入 `Squirrel` / `鼠鬚管`。

接著把 repo 裡的 Rime 設定同步到使用者目錄：

```sh
mkdir -p ~/Library/Rime
rsync -av \
  --exclude='.DS_Store' \
  --exclude='build/' \
  --exclude='*.userdb/' \
  --exclude='installation.yaml' \
  --exclude='user.yaml' \
  --exclude='*.txt' \
  ./Rime/ ~/Library/Rime/
```

同步完成後，從 macOS 選單列的鼠鬚管選單執行 `Deploy` / `重新部署`。也可以重新啟動 Squirrel：

```sh
open -a Squirrel
```

### Rime repo 檢查結果

目前這包舊機設定裡有幾類檔案不建議放上 GitHub：

- `Rime/build/`：Rime 重新部署後會產生的編譯輸出。
- `Rime/*.userdb/`：個人輸入學習資料庫，可能包含輸入習慣或私有詞。
- `Rime/installation.yaml`：包含本機安裝 ID 與安裝時間。
- `Rime/user.yaml`：包含上次部署時間、最近使用方案等本機狀態。
- `Rime/.DS_Store`：macOS Finder 產物。
- `Rime/liur.txt`：舊機匯出的表格資料，曾包含個人自訂詞。

這些已經寫進 `.gitignore`。初始化 git 後，可以用下面指令確認哪些檔案會被忽略：

```sh
git init
git check-ignore -v Rime/build/default.yaml Rime/luna_pinyin.userdb/LOG Rime/installation.yaml Rime/liur.txt
```

如果確認沒有問題，再執行：

```sh
git add .
git status --short
```

## 4. 設定 iTerm2、Oh My Zsh、Powerlevel10k

安裝 Oh My Zsh：

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

確認 iTerm2 字型，避免 Powerlevel10k icon 跑版：

```text
iTerm2 > Settings > Profiles > Text > Font
```

選：

```text
MesloLGS NF
```

把下面內容加到 `~/.zshrc`。如果檔案裡已有 Oh My Zsh 區塊，保留原本 `source $ZSH/oh-my-zsh.sh`，再把 Powerlevel10k 和 plugin source 放在後面即可。

```sh
plugins=(git)

source $ZSH/oh-my-zsh.sh

source "$(brew --prefix)/share/powerlevel10k/powerlevel10k.zsh-theme"
source "$(brew --prefix)/share/zsh-autosuggestions/zsh-autosuggestions.zsh"
source "$(brew --prefix)/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh"

[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
```

重新載入：

```sh
exec zsh
```

第一次載入 Powerlevel10k 時通常會進入設定精靈。若沒有出現，可手動執行：

```sh
p10k configure
```

## 5. 安裝 nvm 與 Node.js

nvm 官方不建議用 Homebrew 安裝，改用官方安裝腳本（版本號到 <https://github.com/nvm-sh/nvm/releases> 查最新的，套進下面網址）：

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

腳本通常會自動把載入設定加到 `~/.zshrc`。若沒有，手動補上（放在前面 Powerlevel10k / plugin 那幾行之後即可）：

```sh
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

重開終端機或 `exec zsh` 後，安裝 LTS 版 Node 並設為預設：

```sh
nvm install --lts
nvm alias default 'lts/*'
node -v
```

## 6. Git 公司帳號與 Copilot 個人帳號

這台是公司筆電，預設情況很單純：**git 全部用公司帳號，Copilot 用個人帳號**。兩者獨立，登個人帳號做 Copilot 補全，不會影響 commit 身分，也不會影響 push。

設定全域 Git identity 為公司帳號：

```sh
git config --global user.name "Your Company Name"
git config --global user.email "your.company@example.com"
git config --global init.defaultBranch main
```

登入 GitHub CLI 時使用公司 GitHub 帳號：

```sh
gh auth login
gh auth status
```

VS Code Copilot 則在 VS Code 裡用個人 GitHub 帳號登入：

```text
VS Code > Accounts > Sign in with GitHub
```

登入後確認 Copilot extension 已啟用，並在 VS Code 裡檢查：

```text
GitHub Copilot: Sign In
GitHub Copilot: Check Status
```

`Check Status` 會顯示目前吃哪個帳號的 Copilot 訂閱，確認是個人那個即可。若公司帳號也被加進某個 Copilot 組織，這裡可能會選錯，重新用 `Sign In` 切回個人帳號。

到這裡日常開發就設定完了：公司帳號推 code、個人帳號補全。下面是選用的，平常用不到。

---

### 補充（選用）：哪天要用個人帳號推 repo 時

> 預設只用公司帳號推送，**不需要**做這段。只有當你之後想在這台筆電同時推公司和個人 repo 時，才照下面設定，讓兩個 GitHub 帳號各自用不同 SSH key。

建立兩組 key：

```sh
ssh-keygen -t ed25519 -C "your.company@example.com" -f ~/.ssh/id_ed25519_company
ssh-keygen -t ed25519 -C "your.personal@example.com" -f ~/.ssh/id_ed25519_personal
```

編輯 `~/.ssh/config`：

```sshconfig
Host github-company
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_company
  IdentitiesOnly yes

Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
  IdentitiesOnly yes
```

公司 repo 的 remote 使用：

```sh
git remote set-url origin git@github-company:ORG/REPO.git
```

個人 repo 的 remote 使用：

```sh
git remote set-url origin git@github-personal:USER/REPO.git
```

由於全域 Git identity 是公司帳號，個人 repo 還要單獨把該 repo 的 commit 身分改成個人，避免 commit 掛到公司 email（在該 repo 目錄底下執行，不加 `--global`）：

```sh
git config user.name "Your Name"
git config user.email "your.personal@example.com"
```

## 7. 推上 GitHub 前檢查

推上 GitHub 前建議跑：

```sh
git status --short
git check-ignore -v Rime/build/default.yaml Rime/luna_pinyin.userdb/LOG Rime/installation.yaml Rime/liur.txt
```

再掃一次疑似敏感字串：

```sh
grep -rEn "token|password|passwd|secret|private|BEGIN|github_pat|ghp_|ssh-rsa|ed25519|@" . --exclude-dir=.git
```

`@` 會找到很多公開作者 email 或 Rime 規則，重點是確認沒有自己的公司或個人帳號、token、SSH private key。
