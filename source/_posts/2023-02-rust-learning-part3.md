---
title: Rust 學習筆記第三章 - 所有權（下）
date: 2023-02-16 09:40:33
categories:
- Learning
tags:
- Rust
---
## 前情提要
接續上篇，我們想要讓函式使用變數的數值，同時不希望它取得所有權，
也嘗試透過回傳元組解決，但這樣還是太繁瑣，很不 Rust（o
到底該如何同時達到乾淨而且不失去變數所有權呢？

這就是今天要討論的第一件事：**引用與借用**。

## 引用（Reference）
在前面的例子，我們會失去所有權是因為把變數傳遞給函式，
但若是只提供數值呢？

這就是引用的作用：追蹤、存取特定記憶體位址的資訊，同時該位址仍被其他變數擁有。
另外，Rust 的引用保證所指向的特定型別數值一定有效（避免如 C 可能發生空指標的狀況）。

我們可以更改上篇的例子：
```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("'{}' 的長度為 {}。", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```
我們會注意到元組都被拿掉了；
同時，我們會傳遞 `&s1` 給函式，這裡的 `&` 就是引用，
引用允許我們不用獲取所有權也能使用數值。

`&s1` 這個語法建立一個指向 `s1` 數值的引用，
但是因為沒有所有權，所以引用不再被使用後 `s1` 並不會被丟棄。
我們稱呼這樣的動作為**借用（borrowing）**，因為與現實的借用行為非常接近。

一般而言借用只能引用資料，並不可以修改資料，否則會出現錯誤，
這被稱為不可變引用；
但是跟變數一樣，若是想變更借用的數值，可以使用**可變引用**：
```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```
首先變數本身需要是可變的，才能在呼叫時建立可變引用 `&mut`。
可變引用最大的限制，在於如果對一個數值有可變引用，
便無法再對該數值有其他任何引用，也就是不能建立兩個以上的可變引用。
學到這裡確實好奇，其他大多數語言都允許直接更改，但為什麼 Rust 不行？

回到最前面，Rust 非常強調資料的安全性，所以在引用行為也設下幾道限制，
避免在編譯時發生**資料競爭（data races）**：
- 同時有兩個以上的指標存取同一資料
- 至少有一個指標在寫入資料
- 沒有針對資料的同步存取機制
資料競爭容易引發未定義行為（undefined behavior），學過的人應該都知道這是非常不安全的。

那如果真的想要有多個可變引用該怎麼做？
同樣很簡單，利用 `{}` 建立新的作用域就好了：
```rust
fn main() {
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 離開作用域，所以建立新的引用也不會有問題

    let r2 = &mut s;
}
```
如上，只要不是同時擁有，就不會發生資料競爭。

同樣地，在可變引用與不可變引用的組合中也存在類似規則，
如以下錯誤的程式碼：
```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // 沒問題
    let r2 = &s; // 沒問題
    let r3 = &mut s; // 很有問題！

    println!("{}, {}, and {}", r1, r2, r3);
}
```
很明顯，Rust **不允許**擁有不可變引用的同時擁有可變引用，
因為不可變引用的擁有者應該不會希望有人能夠擅自改變數值www；
不過同時有多個不可變引用就沒有問題，因為沒有任何人能夠改變資料。

注意，引用的作用域始於宣告，終於最後一次被使用；
所以其實能夠允許不可變引用與可變引用出現在同一份程式碼內，但兩者作用域必須分開。

### 迷途引用
前面提過，有別於其他語言可能發生釋放資源但指標仍然存在，導致迷途指標（dangling pointer）發生，
Rust 的編譯器會保證引用始終有效，也就是說若是有引用，編譯器會確保資料不會提前離開作用域。

我們可以透過以下程式碼理解 Rust 如何阻止迷途引用：
```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```
若是直接執行會看到錯誤訊息：
```sh
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime
  |
5 | fn dangle() -> &'static String {
  |                 +++++++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `ownership` due to previous error
```
其中出現一個陌生的名詞（也可能已經在其他語言看過）：生命週期（lifetime），
可以先略過（同樣後面有專門討論的主題）；
重點在於這句：
`this function's return type contains a borrowed value, but there is no value for it to be borrowed from`。
簡單來說，`s` 是在 `dangle` 內產生，當 `dangle` 作用域結束，`s` 會被釋放，
此時我們卻嘗試回傳這個目標被釋放的引用，這時引用指向的是已經無效的 `String`，
在其他語言可能還有辦法編譯，但 Rust 的檢查機制不會允許，也算是一層安全保障。

若是要解決上面的問題，改為直接回傳 `String` 就可以了：
```rust
fn main() {
    let string = no_dangle();
}

fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```
這會使所有權被轉移出去，不會釋放任何數值。

### 引用規則
先總結一下前面討論的引用規則：
- 同一時間只能有**一個可變引用**，或是**任意數量的不可變引用**
- 引用必須**永遠有效**
這是引用的基本原則。

接下來我們可能會遇到一個問題：
我想要取得 / 描述一個字串的其中一部份，那我該怎麼做？
這就是下面要討論的特殊引用方式：**切片**。

## 切片（Slice）
切片是一個可以引用一串集合中的元素序列，而不需要引用整個集合的方法，
與引用相同，切片並**不會取得所有權**。

我們先思考剛剛的問題，如果我們接收到一串用空格分開的字串，要求寫一個函式取得第一個單字，
我們該怎麼做？

先寫出函式簽名，我們才能理解該怎麼解決問題：
```rust
fn first_word(s: &String) -> ?
```
可以看到，確定會有一個參數 `&String`，因為我們已經知道不需要取得字串的所有權，這很合理；
但是我們還不知道要回傳什麼，所以回傳型別仍然是個問號。
我們先嘗試把單字最後一個索引回傳並與空格 `' '` 做比較：
```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```
這裡出現了幾個不熟悉的方法：`as_bytes()`、`iter()` 與 `enumerate()`，
其中 `as_bytes()` 是將 `String` 轉為位元組陣列，因為我們需要遍歷字串的每個元素；
`iter()` 在非常後面才會提及，這邊只要知道它可以回傳集合中每一個元素；
`enumerate()` 則是將 `iter()` 的結果轉為如 `(index, &element)` 的元組回傳，
其中 `index` 是索引，`&element` 是元素的引用。
如此，我們便能省去計算索引的時間。

接著，我們利用模式配對解構元組來獲取數值，
並使用字串字面值的語法比對空格，如果有空格就回傳位置，否則回傳字串長度。

現在我們可以找到第一個單字的結尾索引，
但需要注意的是，上面回傳的是一個 `usize`，需要套用在 `&String` 上才有意義。
簡單的說，無法保證這個回傳值在未來是否有效：
```rust
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word 取得數值 5

    s.clear(); // 這會清空 String，這就等於 ""

    // word 仍然是數值 5 ，但是我們已經沒有相等意義的字串了
    // 擁有 5 的變數 word 現在完全沒意義！
}
```
雖然這段程式碼可以編譯執行，但仍然有程式錯誤：
在 `s.clear()` 之後 `word` 已經失去相等意義的字串。
如果接著繼續使用 `s` 取得單字，就會發生錯誤。

若是要避免這樣的資料脫勾，以目前學到的方法，會製造出更多沒有直接相關的計算結果，還需要保持同步，
這太容易發生錯誤，所以 Rust 提供一個相對安全的方式，也就是這邊要討論的切片。

### 字串切片
字串切片是引用一部分的 `String`，如下：
```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```
其中 `s[0..5]` 代表取用的範圍，
`[起始索引..結束索引]` 這樣的指定範圍語法代表從起始索引開始到結束索引停止，且不包含結束索引。
切片與字串相同內部擁有一個結構：指向**起始位置的指標**以及**長度（結束索引減去起始索引）**。

指定範圍若是從索引 0 開始，此時語法可以如 `[..2]`，省略 `0`；
而若是包含最後一位元，同樣能省略最後一個數值：`[3..]`；
而如果要整個字串，甚至可以這樣表達：`[..]`。
（注意：這樣的切片索引範圍必須是有效的 UTF-8 字元。）

利用切片，我們可以重寫前面的 `first_word`：
```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```
同樣是判斷第一個空格，但我們可以利用找到的索引產生字串切片並回傳，這樣乾淨太多了；
並且，因為切片會使編譯器確保引用有效，所以前面發生的問題程式碼就會被編譯器擋下。

### 字串字面值作為切片
先回憶一下字串字面值如何存在執行檔，那麼現在從切片我們可以更加理解：
```rust
let s = "Hello, world!";
```
在這裡 `s` 是 `&str`，因為它是指向執行檔某部份的切片，
同時也說明字串字面值不可變的原因，是因為 `&str` 是個**不可變引用**。

### 字串切片作為參數
我們可以再次改善 `first_word` 的簽名：
```rust
fn first_word(s: &str) -> &str 
```
這是比較有經驗的人的寫法，因為這樣允許函式同時接受 `&String` 與 `&str`，
在我們有切片時直接傳遞，而有 `String` 時傳遞整個切片或是引用，
具有非常大的彈性。（這裡用到強制解引用，在非常後面會提及）

### 其他切片
切片並非只有字串可以使用，如陣列也可以使用：
```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```
這裡的切片為 `&[i32]`，如同字串切片一樣儲存第一個元素與長度。
以後會有更多使用這類切片的機會，在不遠的將來會討論這些場合。

## 總結
所有權與借用、切片是非常重要的 Rust 概念，能夠讓我們寫出符合 Rust 期待的安全程式碼；
同時所有權系統也能夠減少我們花在除錯的時間。

不過這概念非常困難，我自己都不能保證這筆記寫得很精確……
但也希望多少能讓自己更有印象，因為這會影響後面很多的執行，比如下一篇要討論的自訂型別概念————**結構體（Structure）**。

希望大家在學習路上都能開開心心。

火山 / Kazan
2023.02.16