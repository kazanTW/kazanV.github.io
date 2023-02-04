---
title: Rust 學習筆記 - 序篇
date: 2023-02-04 22:11:48
categories:
- Learning
tags:
- Rust
---
## 前情提要
因為 jserv 老師今年預計會在 **Linux 系統核心課程**教學 Rust，加上近幾年許多老牌 CLI 程式開始以 Rust 改寫，
「**用 Rust 開發系統程式**」似乎成為主流了，於是在傳統的 C 之外，我也決定開始學習 Rust，反正技多不壓身www
為了避免忘記學到的東西，並且分享學習心得，
我決定把學習的過程寫成文章放上來，便開始這系列的文章（~~同時也是逼自己更新，不然我真的太容易怠惰了~~）。

本系列文章皆參考自 [Rust 程式設計語言](https://rust-lang.tw/book-tw/)電子書，同時會加上我在學習過程中的心得與發現。

## Rust 語言簡介
Rust 是以「**賦權（empowerment）**」為重點的程式語言，「能賦予你更多能力，在更廣泛的領域中帶有自信地向前邁進」（來自電子書*前言*）

在過去，程式語言往往會在「上層的易讀易用性」與「底層的掌控性」之間難以取捨，
而 Rust 對此做出挑戰，嘗試讓開發者可以控制底層的實作細節，但又能免去以往的麻煩細節。

同時，Rust 更加強調**安全性**、**記憶體組態**與**並行處理**，
這讓 Rust 程式變得更適合用於設計大型網路伺服器與客戶端、以及更安全的系統程式。

## Rust 適合什麼樣的人？
先簡單總結：**幾乎全部**。

Rust 雖然重視底層實作，但同時也具備現代化的開發工具，提高開發者的生產力：
- Cargo：負責管理依賴函式庫與建構
- Rustfmt：確保專案開發遵循統一的程式碼風格（也就是 **Coding style**）
- Rust Language Server：提供 IDE 內的自動補全等功能

所以無論是一個完整的開發團隊、公司、開源開發者甚至學生，
如果需要重視**速度**與**穩定性**，Rust 或許是不錯的選擇。

當然 Rust 也跟其他程式語言一樣有其缺陷：**目前 Rust 並不適合用來開發 GUI 程式**
Rust 的長處在於 CLI 程式，GUI 尚在拓荒階段，並不推薦。

說到這裡，如果你也有興趣，那就跟著我開始練習吧！

## 開始 Rust 的旅程
從這裡開始，我會假設大家跟我一樣有程式設計的基礎，並且有實際撰寫程式碼的經驗，
往後就不會再多提及程式邏輯，而是注重在 Rust 的特性與應用方式。

學習一個語言，當然要先安裝編譯器與執行環境，
接著就是要說明如何安裝 Rust 環境在系統上，並認識 Rust 的開發環境。

## 安裝 Rust
使用及安裝 Rust 有很多方式，但新手（其實開發老手也是）強烈推薦透過 `rustup` 安裝，
這是一個負責管理 Rust 版本以及相關工具的 CLI 工具。

### 安裝 rustup
#### Linux 及 macOS
前往[安裝網頁](https://www.rust-lang.org/tools/install)，會偵測你在使用的 OS，並提供你推薦的安裝方式；
或者直接透過以下命令下載 `rustup` 的安裝腳本並開始安裝，接著開始安裝最新穩定版本的 Rust（過程可能需要密碼）；
```sh
curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
```
其中 `curl` 命令可以代換成 `wget` 或 `fetch`，不過通常使用 `curl` 就可以了（大多數 Linux 發行版跟 macOS 都會預裝）。
當出現以下訊息，代表安裝成功：
```
Rust is installed now. Great!
```

另外，Rust 在編譯資料輸出時，需要一個連結器（linker），通常會推薦使用 C 編譯器，若是遇到 linker 相關錯誤，就會需要安裝；
大部分的 Linux 發行版都有預裝 GCC compiler，或者依據使用習慣會是 Clang；若是沒有，請依據發行版文件安裝相關套件；
至於 macOS 使用者可以透過 Xcode 的工具安裝：
```sh
xocde-select --install
```

並且，若是有使用 IDE 或任何支援 LSP 的編輯器如 Neovim，推薦額外安裝 `rust-analyzer` 這個 LSP 套件，
能提供更多自動補全以及整合開發的功能，目前 Rust 團隊也正在積極透過這個套件提昇對 IDE 的支援。
我自己是使用 Neovim，所以當然有安裝www

#### Windows
雖然我整天噴 Windows 應該被丟進太平洋最深處封印起來，但由於這東西還是有太多使用者，
想了想還是寫一下 Windows 的部份好了（多不甘願

同樣前往安裝網頁，應該會有個「Install Windows」等著你，依照指示安裝即可，
並且在 VS Code 的 Extension 商店安裝 `rust-analyzer` 套件。
（前述電子書是使用 Visual Studio，但因其為付費軟體就不多討論）

#### 檢查安裝
Rust 的好處就是在各種 OS 指令動作都是一樣的，比較不需要特殊操作，
所以以下指令相容於 POSIX shell 與 cmd.exe、PowerShell。

要檢查是否正確安裝，在 shell 輸入以下指令：
```sh
rustc --version
```
理論上會看到版本編號、雜湊跟提交日期，代表安裝成功；
若是沒有，檢查環境變數 `PATH` 是否有 Rust：
cmd.exe：`echo %PATH%`
PS：`echo $env:PATH`
Linux、macOS：`echo $PATH`
若是有，但無法執行 Rust，就……~~STFG~~去社群求助吧！

### 更新與解除安裝
~~還沒開始寫 code 就先學怎麼解除安裝有點謎~~
更新 Rust 同樣透過 `rustup` 執行更新腳本：
```sh
rustup update
```
而解除安裝也是透過 `rustup` 執行解除安裝腳本：
```sh
rustup self install
```

### 技術文件
安裝 Rust 時，同時也會安裝一份本地文件，在遇上問題時可以離線閱讀；執行 `rustup doc` 即可。

都安裝好了，接著就該來寫 code 了！

## 第一支 Rust 程式
首先建議建立一個專門放 Rust 專案的目錄（每個語言都建議這麼做，避免混亂），
在其下建立一個 `hello_world` 目錄，在這個目錄下產生新的原始碼檔案並命名為 `main.rs`，
其中 `.rs` 是 Rust 的原始碼文件格式。

在這邊提一下， Rust 慣用的命名方式是 snake_case，或者比較正式的格式名稱是 `lowercase_separated_by_underscores`，
也就是諸如 `hello_world` 的形式。

開啟 `main.rs` 並輸入以下程式碼（部份編輯器 / IDE 會警告「不存在 `Cargo.toml`」，此時先略過沒關係）：
```rust
fn main() {
    println!("Hello, world!");
}
```
儲存後回到專案目錄，編譯：
```sh
rustc main.rs
./main      // Windows 上輸入 .\main.exe
```
理論上會輸出：
`Hello, world!`
到這邊如果都正常，代表你成功寫出了一支 Rust 程式！

### 分析
首先看到：
```rust
fn main() {

}
```
在 Rust 中，所有的函式都是以 `fn` 開頭，其中 `fn main()` 是所有 Rust 程式的進入點，
如同 C 的 `main()`、Java 的 `public static void main(String[] args)` 等；
並且函式本體被放在 `{}` 中，如同大多數程式語言。

接著看到：
```rust
    println!("Hello, world!");
```
這裡需要注意的細節是：
1. Rust 中排版風格並不是一個 tab，而是**四個空格**（但可以透過編輯器設定讓 tab 等於四個空格）。
2. `println!` 是一支巨集而非函式（函式則為 `println`），目前我們只需要知道呼叫巨集是使用 `!`，且規則不完全與函式一樣。
3. "Hello, world!" 是字串，作為引數被傳遞給 `println!` 然後顯示在螢幕（準確來說是終端機）上，。
4. 多數 Rust 程式碼使用分號（`;`）作為結尾（後面會學到什麼時候**不該**使用 `;`）。

### 編譯與執行
在稍早，我們首先透過 `rustc` 編譯了 `main.rs` 並執行，
這意味著 Rust 如同 C、Java 等，是編譯式語言，程式碼可以編譯後將執行檔釋出給沒有安裝 Rust 的人執行。

雖然簡單的程式可以透過 `rustc` 編譯，但若是專案很大，我們就需要管理選擇並使程式碼容易分享，
所以接著要介紹 Rust 的好朋友——Cargo。

## Hello, Cargo!
Cargo 是 Rust 建構與管理用的工具，具有以下用途：
1. 管理專案
2. 建構程式碼
3. 下載並安裝依賴函式庫
世界上絕大多數的開發者與專案都是使用 Cargo，所以作為初學者我們也必須熟悉 Cargo，往後也會預設使用 Cargo。
Cargo 在前面安裝 Rust 時應該也安裝好了（若使用官方 `rustup`），可以用以下指令檢查：
```sh
cargo --version
```
若是輸出版本號，代表安裝成功，接著就來學著如何用 Cargo 建立並管理專案吧！

### 使用 Cargo 建立專案
輸入以下指令：
```sh
cargo new hello_cargo
cd hello_cargo
```
Cargo 會產生一個專案目錄，若是進入目錄並檢視會發現：
```sh
$ ls -l
total 8
-rw-r--r-- 1 kazan kazan  174 Feb  5 00:20 Cargo.toml
drwxr-xr-x 2 kazan kazan 4096 Feb  5 00:20 src
```
Cargo 產生了一個目錄：`src`（其中包含一個 `main.rs` 檔案），以及一個 `Cargo.toml` 檔案，
同時會將專案目錄初始化為一個 Git repository，並且新增 `.gitignore` 檔案。（如果不想要使用 git 或任何 VCS，請參考文件）

用任何編輯器開啟 `Cargo.toml`，應該會看到以下內容：
```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```
Cargo 配置文件使用 TOML，想了解更多關於 TOML，請參考[官方網站](https://toml.io/)；
而其中 `[edition]` 是什麼，會在日後介紹；
至於 `[dependency]` 是列出專案使用哪些 dependency，在 Rust 中套件被稱為 **crates**，這在往後也會陸續介紹。

而 `main.rs` 被放在 `src` 下，是因為 Cargo 預期原始碼都放在 `src` 下，
至於根目錄是放 README、LICENSE 以及配置文件（`Cargo.toml`）等無關程式碼的檔案，讓一切井然有序。

### 建構並執行專案
使用 Cargo 建構程式：
```sh
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```
這指令會將執行檔輸出到 `target/debug` 目錄，因為 Cargo 預設是用 debug build，
接著可以仿照前面的方式執行，或是利用下面的指令一次編譯並執行：
```sh
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```
如果需要快速檢查程式碼是否能成功編譯但不想產生執行檔，可以執行：
```sh
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

### 建構發布版本（Release）
如果準備好要正式發布專案，可以利用 `cargo build --release`，
這指令會**最佳化**編譯結果，同時產生執行檔到 `target/release` 而非 `target/debug`；
最佳化編譯能夠使程式碼有更好、更快速的執行狀況，但同時需要更長的編譯時間，
這就是為什麼 Cargo 使用不同的設定檔：一個可以快速並經常重新建構，另一個可以產生最終釋出的版本。

### 請將 Cargo 視為常規
Cargo 在小型專案的優勢可能不那麼明顯，但當專案越來越大，Cargo 的用途將會越發明顯。
所以無論多大、多複雜的專案，都建議一律使用 Cargo 進行管理與建構。

另外，若是已存在的專案，仍然可以重新建構為一個 Cargo 專案：
```sh
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

## 總結
今天我們學到了：
1. 如何安裝、更新 Rust
2. 直接撰寫一支 Rust 程式
3. **使用 Cargo 建立、管理、執行 Rust 專案**

接著沒意外是一週兩更，但會視我的進度調整（~~簡單說就是先打拖更預防針~~），
也希望能夠用這系列文章陪伴各位到開學第一個月結束（？
希望大家在學習的路上都能有所收穫。

火山 / Kazan
2023.02.05