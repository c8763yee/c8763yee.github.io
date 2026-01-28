---
title: 帶你深入淺出 Git 版本控制實戰
tags: [NUTC-IMAC教學, Git]

---

# 帶你深入淺出 Git 版本控制實戰
課程連結: ![image](https://hackmd.io/_uploads/HJCgML9PA.png =100x)

## 1. 版本控制是什麼？爲什麼要做版本控制
版本控制是一種管理軟件開發過程中對code變更的技術。它允許開發者跟蹤和管理代碼的歷史變更，協作開發，並且可以在需要時恢復到先前的版本。

很多人作版本控制的方法是把檔案複製到另一個目錄（如果他們夠聰明的話，他們還會幫資料夾加上時間）。 這種做法很常見，因為這樣做很簡單，但是卻也非常容易產生離譜的錯誤。 這種做法非常容易搞混資料夾，意外寫錯檔案或複製覆蓋到不想要的檔案。

而透過「版本控制系統」可以清楚的記錄每個檔案是誰在什麼時候加進來、什麼時候被修改或刪除。Git 就是一種版本控制系統，也是目前業界最流行的版本控制系統，沒有之一。

出社會工作，有 Git 幫你保留這些歷史紀錄跟證據，萬一出事的時候你就能知道是從什麼時候開始就有問題，以及知道該找誰負責，再也不用自己背黑鍋了！
## 2. 事前準備
1. 一台電腦
2. 註冊Github帳號
3. 檢查電腦是否已安裝Git
```sh
# Windows可以使用where來確定指令是否可用
where git

# Linux與macOS則可以使用which或command -v來檢查
which git
# 或者
command -v git
```
## 3. 安裝Git
- 進入[Git官網的下載頁面(https://git-scm.com/downloads)](https://git-scm.com/downloads)後選擇自己的作業系統
![image](https://hackmd.io/_uploads/H1kvGvKUR.png)
### Windows
![image](https://hackmd.io/_uploads/B1NrzDYUA.png)
- 下載完打開安裝檔後一直按Next就行
> 看到這個頁面就代表安裝成功了![image](https://hackmd.io/_uploads/Bksfg3Jv0.png)


### Linux/macOS
- macOS與Linux則需要使用指令安裝
```sh
# macOS
brew install git

# Linux
# 針對各種系統發行版與套件管理器有不同的安裝方式

# apt(Ubuntu/Debian/Linux mint)
sudo apt install git -y 

# pacman(Arch Linux)
sudo pacman -S git --noconfirm

# Fedora
sudo yum install git -y
# fedora 22之後需使用下面指令
sudo dnf install git 
```
> ```
> # 快速查看自己使用的Linux系統發行版: 
> # cat /etc/os-release | grep "^PRETTY_NAME="| cut -d'=' -f2
> ```
## 4. 基礎概念
![](https://hackmd.io/_uploads/ryY3kSSuh.png)
Git 主要分為 4 個區塊及 4 種狀態
### Git區塊
- **Working Directory (在此簡稱: W)**: 現在存在於資料夾內的、編輯器可以打開的位置。
- **Staging Area (在此簡稱: S)**: 暫存狀態。
- **Local Repository (在此簡稱: L)**: 本地(自己的電腦上的)倉庫。
- **Remote Repository (在此簡稱: R)**: 遠端(不在自己的電腦上的)倉庫。

### Git狀態
- **Untracked**: 未被追蹤。
- **Unmodified**: 追蹤中，未修改。
- **Modified**: 追蹤中，已修改。
- **Staged**: 追蹤中，待提交。

<!-- Working Directory 和 Remote Repository 分別代表手上電腦編輯器開啟的檔案及遠端 GitHub 頁面上看到的檔案。

Staging Area 是本地端暫存的檔案，檔案需要先被加入追蹤，才有資格進入 Staging Area ，而沒被加入的檔案就是 Untracked 狀態。

Staging Area 內的檔案狀態與 Working Directory 的狀態一致（沒有編輯過）時，為 Unmodified 狀態；若狀態不一致（編輯過），則為 Modified 狀態。已存在於 Staging Area 檔案的狀態為 Staged 狀態。

Local Repository 就是在本地端的倉庫。可以理解為存在於本地的 GitHub。 -->

## 開始使用
### 本地設定
#### 相關指令
+ `git -h`: 顯示Git的幫助訊息
    - `git [指令] -h`: 顯示特定Git指令的幫助訊息

+ `git init`:在當前目錄下建立一個`.git`的隱藏資料夾用來儲存Git的相關資訊
    - 其他選項
    ```sh
    git init <資料夾名稱>    # 在指定的資料夾下建立Git倉庫
    git init -b <分支名稱>    # 初始化時同時將預設分支設定為指定的分支名稱
    ```
    
    :::warning
    <font size=8>注意\!\!\!\!\!</font>
    在初始化Git後，我們需要先設定使用者名稱與email，這樣除了可以讓Git知道是誰在使用，也可以讓其他人知道是誰做了這個commit。
    ```sh
    # 這邊的`--global`代表全域設定，也就是說這個設定會套用到所有的Git專案
    # 如果只想要套用到特定的專案，可以將`--global`拿掉。
    git config --global user.name <你的名稱>
    git config --global user.email <你的email>
    ```
    設定完之後，就可以真正開始使用Git了。
    :::
+ `git add <檔案名稱>`: 將檔案加入Staging Area
    - 一次可以加入多個檔案，也可以使用`git add .`將目前資料夾下的所有檔案加入
    - 其他選項
    ```sh
    git add -A          # 將所有檔案加入
    git add -u          # 將已經被追蹤的檔案加入
    git add -f          # 強制加入檔案
    ```

+ `git commit -m <訊息>`: 將Staging Area的檔案提交到Local Repository
    - `-m`後面的訊息是這次提交的訊息，可以讓其他人知道這次提交的目的
    - 也可以不加`-m`，這樣會進入一個編輯器讓你輸入訊息
    - 其他選項
    ```sh
    git commit -a -m <訊息>  # 將所有已經被追蹤的檔案加入Staging Area並提交
    git commit --amend -m <訊息> # 修改上一次的commit訊息
    ```
+ `git status`: 查看目前Local Repository與Staging Area的狀態
    - 會顯示目前有哪些檔案在Untracked、Modified、Staged等狀態 
+ `git diff`: 查看與上一版本之差異。    
+ `git reset <檔案名稱>`: 將檔案從Staging Area移除
    - 預設會將指定的檔案從Staging Area移除，但不會影響Working Directory
    - 其他選項
    ```sh
    git reset HEAD           # 將所有在Staging Area的檔案移除
    git reset --hard         # 將所有在Staging Area的檔案移除並且捨棄Working Directory的修改
    ```

## 遠端操作
在講解遠端操作前，我們需要先設定好遠端倉庫的SSH Key，這樣才能夠透過SSH的方式連接到遠端倉庫。
### 建立SSH Key
```sh
ssh-keygen -t ed25519 -C "<你的email>"
# 或者，如果你的電腦不支援ed25519
ssh-keygen -t rsa -b 4096 -C "<你的email>"

# 輸入 keys 檔案要放的位置與名稱，沒有要改就 Enter
> Enter file in which to save the key (/home/<user>/.ssh/id_ed25519):

# 設定通行密碼，沒有要改就 Enter
> Enter passphrase

# 再輸入一次通行密碼，沒有一樣 Enter 就好
> Enter same passphrase again:
```
- 接著輸入指令或用記事本打開公鑰。
```sh
cat ~/.ssh/id_ed25519.pub
# 或者
notepad ~/.ssh/id_ed25519.pub
```
- 複製公鑰內容，並且將公鑰加入到遠端倉庫的SSH Key設定中(以Github為例)
    - 進入Github的設定頁面
    ![image](https://hackmd.io/_uploads/H197l6cU0.png)
    - 選擇`SSH and GPG keys`
    ![image](https://hackmd.io/_uploads/SJbdxT5LC.png)
    - 按下`New SSH Key`
    ![image](https://hackmd.io/_uploads/rJF9x65UC.png)
    - 在`Title`中輸入自己取的SSH Key名稱，然後在`key`輸入剛才的SSH公鑰。最後按下`Add SSH Key`就完成了
    ![image](https://hackmd.io/_uploads/HkOogaq8C.png)

### 遠端相關指令
+ `git remote`: 查看目前有哪些遠端倉庫
	- 顯示目前有哪些遠端倉庫，並且會標示預設的遠端倉庫
	- 其他選項
    ```sh
    git remote add <遠端倉庫名稱> <遠端倉庫網址>  # 新增一個遠端倉庫
    git remote remove <遠端倉庫名稱>  # 移除一個遠端倉庫
    git remote set-url <遠端倉庫名稱> <遠端倉庫網址>  # 修改遠端倉庫的網址
    git remote -v  # 顯示遠端倉庫與網址
    git remote rename <舊遠端倉庫名稱> <新遠端倉庫名稱>  # 修改遠端倉庫的名稱
    ```
+ `git branch`: 查看目前有哪些分支
	- 會顯示目前有哪些分支，並且會標示目前所在的分支
	- 其他選項
    ```sh
    git branch <分支名稱>    # 建立一個新的分支
    git branch -d <分支名稱> # 刪除一個分支
    ```
+ `git checkout <分支名稱>`: 切換到指定的分支
    - 其他選項
    ```sh
    git checkout -b <分支名稱> # 建立一個新的分支(如果該分支不存在)並切換到該分支
    git checkout -- <檔案名稱> # 捨棄對檔案的修改
    ```
+ `git push <遠端倉庫名稱> <分支名稱>`: 將Local Repository的分支推送到遠端倉庫
    - 將Local Repository的分支推送到遠端倉庫
    - 其他選項
    ```sh
    git push -u <遠端倉庫名稱> <分支名稱>  # 設定預設推送的遠端倉庫與分支
    ```
+ `git clone <遠端倉庫網址>`: 複製一份遠端倉庫到本地
    - 在當前目錄下建立一個資料夾，資料夾名稱就是遠端倉庫的名稱
    - 其他選項
    ```sh
    git clone <遠端倉庫網址> <目標資料夾路徑>  # 指定複製到指定位置
    ```

    
+ `git pull <遠端倉庫名稱> <分支名稱>`: 從遠端倉庫拉取分支
	- 將遠端倉庫的分支拉取到Local Repository
	- 其他選項
    ```sh
    git pull  # 從預設遠端倉庫拉取分支
    git pull --rebase  # 使用rebase方式拉取分支
    ```
# 5. 協作流程
## Git Flow
![](https://hackmd.io/_uploads/Sy8SIXQKn.png)
+ 特點：
    + 不同分支，不同功能。
+ 優點：
  - 流程清晰可控
+ 缺點：
  - 需要同時維護 main/master 分支與 delevop 分支兩個長期分支
  - 對於中小型專案來說過於複雜且管理分支對初學者或新團隊來說會提高負擔

## Github Flow
![](https://hackmd.io/_uploads/B1m58Q7F2.png)
- 特點：
    - 簡化版的Git Flow
    - Pull Request(PR)
+ 優點：
    - 只有一個長期分支 - main/master 分支
+ 缺點：
    - 正式環境更容易出問題
    - 在沒有適當測試和健全的CI/CD 、代碼審核……等過程下更容易發布問題到生產環境。
    
    
## GitLab Flow
1.持續部署
![](https://hackmd.io/_uploads/rySiIm7Fn.png)
2.版本發布
![](https://hackmd.io/_uploads/B1G6I77Y2.png)

特點：
- upstream first(只有 main/master 作為主要分支)
## 有興趣自己可以去了解的知識
### 指令類
- git blame
- git rebase
- git cherry-pick
- git merge
### 其他
- commit message規範
- 如何發pull request
# 作業
![image](https://hackmd.io/_uploads/B1b-A75wA.png)

1. [複製（Fork）這個repo(https://github.com/c8763yee/git-lec)](https://github.com/c8763yee/git-tutorial.git)到自己的Github，然後Clone到自己的電腦
2. 在shared資料夾將以下的code儲存並命名為自己的學號後push到自己剛才Fork的Github Repo
```sh
#!/bin/bash
echo 'helloworld'
```
3. （可做可不做）發一個Pull Request到上面的repo並且在留言區中寫上自己的名字與學號
# 表單連結
![image](https://hackmd.io/_uploads/H1F1f49PA.png)
