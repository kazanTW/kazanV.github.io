---
title: Rust 學習筆記第二章 - 所有權（上）
date: 2023-02-11 14:56:18
categories:
- Learning
tags:
- Rust
---
## 前情提要
上次學到了 Rust 的基本概念，這次則進入 Rust 最大的特色————**所有權**。
因為比較難理解，我這次打算分上下篇來紀錄，~~絕對不是寫不完拖更~~。

**注意：這個概念一定要理解，才能更有效的利用 Rust！**

## 所有權是什麼？
我們在學習任何語言時，都會需要知道它們如何使用與管理記憶體的資源，以免造成浪費或者更糟糕的情形。
一般而言會有下列兩種流派：
- 垃圾回收機制（garbage collector），程式在執行時會尋找不再被使用的記憶體並釋放，例如 Java、Python
- 開發者親自分配，在程式碼中明確指出何時分配、何時釋放，例如 C（~~我相信對很多人來說 `malloc` 是個無法跨過的夢魘~~）

但是 Rust 選擇了第三種方式：由**所有權系統**管理記憶體資源，
同時在編譯時檢查規則，違反規則會無法編譯，且規則不影響執行速度。

#### 這就是 Rust 最大的特點：所有權（Ownership）。

所以簡單來說，所有權是 Rust 中用以**管理程式記憶體的一系列規則**。

（註：如果要更深入理解所有權的原理，會需要理解**堆疊（Stack）**與**堆積（Heap）**的運作，可以參考官方文件的[這篇](https://rust-lang.tw/book-tw/ch04-01-what-is-ownership.html#%E5%A0%86%E7%96%8Astack%E8%88%87%E5%A0%86%E7%A9%8Dheap)）

## 所有權基本規則
- #### Rust 中每個數值都有**擁有者（Owner）**
- #### 同一時間只能有一個擁有者
- #### 擁有者離開**作用域**時，數值會被丟棄

## 變數作用域
**作用域（Scope）**，指的是某個項目在**程式內的有效範圍**。

我們首先看到以下範例：
```rust
let s = "hello";
```
上述的陳述建立一個字串字面值 `s`，字串數值被寫死。
`s` 的有效範圍是從宣告開始，直到當前作用域結束：
```rust
{                       // s 無效，因為尚未宣告
    let s = "hello";    // s 開始有效

    // 使用 s
}                       // s 不再有效，因為作用域結束
```
上面說明了 `s` 有效的範圍；也就是說，有兩個重要時間點需要注意：
1. 當 `s` 進入作用域，它便開始有效
2. `s` 持續被視為有效，直到它離開作用域

對作用域有基本認識後，接著要以此為基礎認識 `String` 型別。

## `String` 型別
上面的字串 `s` 是一個字串字面值（string literals），代表數值被寫死在程式內。
這樣的作法是很方便，但因為不可變，在需要改變它的值時（例如收集使用者的輸入）就會變得很麻煩。

Rust 為此提供了一個方法：字串型別 `String`。
這個型別管理的是在堆積上的資料，所以能夠用來儲存編譯期間未知的文字。
延續上面的例子，我們可以利用字串字面值以及 `from` 函式建立一個 `String`：
```rust[label](https://www.maj-soul.com/#%2Fhome)
let s = String::from("hello");
```
如果熟悉 C++，這邊的 `::` 概念是一樣的，也就是將函式等置於命名空間下，後面還會再討論這個語法。
而透過這個方式建立的字串是可以被改變的：
```rust
fn main() {
    let mut s = String::from("hello");

    s.push_str(", world!"); // push_str() 將字面值加到字串後面

    println!("{}", s); // 這會印出 `hello, world!`
}
```
所以為什麼兩種方式建立了同樣的字串，但一個可變一個不行？這主要來自兩者對記憶體的操作方式。

## 記憶體與分配
字串字面值之所以效率高，是因為編譯時我們已經知道內容，能夠寫死在執行檔內；
但這樣的優勢也是來自不可變性，一旦遇上大小未知或可能改變的文字，這樣的優勢就會立刻消失。

而 `String` 型別為了支援可變大小，會在堆積上分配一塊未知大小的記憶體，實作上會是這樣子：
- 執行時需要請求記憶體
- 當不再需要這個 `String` 時，需要把記憶體歸還
在上面的例子，`String::from` 已經完成請求記憶體的部份，這跟許多其他語言類似；
但歸還的部份就有所不同了。

一般而言，如果有 GC，GC 會自動追蹤與清理不再被使用的記憶體；
沒有 GC 則需要自己手動釋放（例如 C 的 `free()`），
而這會是一個非常複雜而艱鉅的任務，必須精準的配對分配與釋放，否則會發生嚴重的錯誤。

但 Rust 仍然選擇與眾不同的方式：**記憶體在擁有者離開作用域會自動釋放**。
我們修改前面的例子：
```rust
{
    let s = String::from("hello"); // s 在此開始視為有效

    // 使用 s
}                                  // 此作用域結束
                                   // s 不再有效
```
前面有提過，`s` 的作用域在 `{}` 中，當離開該作用域，`s` 便不再有效。
而當 `s` 離開作用域時，Rust 會**自動呼叫 `drop` 將記憶體歸還**。
（如果你熟悉 C++ 的 RAII，那麼 `drop` 你就會很熟悉。）

接著我們要討論一些複雜的情形。

### 移動（Move）、複製（Clone）、拷貝（Copy）
數個變數可以有不同方式與**相同**的資料互動，先看看以下範例：
```rust
let x = 5;
let y = x;
```
上述的範例非常簡單，將數值 `5` 指定給變數 `x`，接著 copy 一份給 `y`，
並且在記憶體中，這兩個變數都會進入**堆疊**。

再看看下面的例子：
```rust
let s1 = String::from("hello");
let s2 = s1;
```
乍看之下可能會認為，上面做的與前面一模一樣，也就是 copy `s1` 的內容給 `s2`。
但在 Rust 中並非如此。

Rust 的 `String` 架構是由三個部份：指向儲存內容記憶體的指標 `ptr`、長度 `len`、容量 `capacity`，在此先不討論長度與容量的差異；
`String` 儲存的資訊是放在**堆疊**上，但是指向的資料內容則是在**堆積**上。
上面的 `s2 = s1`，確實是 copy 資料，但 copy 的部份是 `String` 儲存的資訊，也就是指標、長度與容量；
否則若是真的直接拷貝資料內容，會產生巨大的「花費」，使得堆積上的資料累積越來越多，這對效能會有非常明顯的影響。

另外，前面有提到當變數離開作用域，Rust 會自動呼叫 `drop` 釋放記憶體；
但若是有兩個指標同時指向同一記憶體，在釋放時便會發生「兩個變數都嘗試釋放同一塊記憶體」，
這會導致**雙重釋放（double free）**錯誤發生，進而可能造成記憶體損壞、產生安全漏洞。

為了預防這種情況，Rust 會再做一件事：在 `let s2 = s1;` 後，`s1` 便**不再有效**，
所以在 `s1` 離開作用域時就不會進行釋放。
我們可以透過下面的範例驗證這件事：
```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{}, world!", s1);
}
```
正常來說這段程式碼無法被編譯，並且會得到下列的錯誤：
```sh
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move
  |
  = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0382`.
error: could not compile `ownership` due to previous error
```

如果聽過**淺拷貝（shallow copy）**與**深拷貝（deep copy）**，會發現上面的行為與淺拷貝非常相像，
但是 Rust 在拷貝資訊的同時會**無效**第一個變數，所以在 Rust 中，這樣的行為稱為**移動（Move）**。

若是真的想要深拷貝堆積上的資料，Rust 仍然有提供方法：**複製或作克隆）** `clone`，
這是一個方法（method）：
```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
}
```
這就能正常執行了，因為 `s2` 是深拷貝了 `s1` 在堆積上的資料內容，或者可以說另外宣告了一個變數。
但 `clone` 很「昂貴」，請謹慎使用。

前面都在說堆積上的資料，那如果是在堆疊上的資料呢？

先來看以下的例子：
```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
}
```
照理來說，依據上面所學，因為沒有呼叫 `clone`，`x` 應該已經變為無效，但程式會**正常執行**。
可以先自己想想看為什麼？

正確的原因是，因為在編譯時，整數這樣的型別是已知大小，只會存在堆疊上，
故拷貝實際數值是很快的，也失去「無效化」的理由。
Rust 有種特殊標記：**`Copy` 特徵（trait，後續會提及）**，
如果有這個特徵，那麼變數賦值給其他變數後仍然保持有效；
反之，若是型別實作了 `Drop` 特徵，則不會被允許擁有 `Copy` 特徵，這是避免衝突與錯誤產生。
以下是實作了 `Copy` 特徵的舉例：
- 所有純量型別（整數、浮點數、布林、字元）
- 元組可以實作，但前提是包含的型別也都有實作（例如 `(i32, String)` 就不會有 `Copy`）

## 所有權與函式
與賦值給變數類似，傳遞變數給函式會是移動或拷貝；
下面的範例說明變數如何進入且離開作用域：
```rust
fn main() {
    let s = String::from("hello");  // s 進入作用域

    takes_ownership(s);             // s 的值進入函式
                                    // 所以 s 也在此無效

    let x = 5;                      // x 進入作用域

    makes_copy(x);                  // x 本該移動進函式裡
                                    // 但 i32 有 Copy，所以 x 可繼續使用

} // x 在此離開作用域，接著是 s。但因為 s 的值已經被移動了
  // 它不會有任何動作

fn takes_ownership(some_string: String) { // some_string 進入作用域
    println!("{}", some_string);
} // some_string 在此離開作用域並呼叫 `drop`
  // 佔用的記憶體被釋放

fn makes_copy(some_integer: i32) { // some_integer 進入作用域
    println!("{}", some_integer);
} // some_integer 在此離開作用域，沒有任何動作發生
```
若是嘗試在呼叫 `take_ownership` 後使用 `s`，會發生編譯錯誤。

## 回傳值與作用域
回傳值同樣可以轉移所有權，見範例：
```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership 移動它的回傳值給 s1

    let s2 = String::from("哈囉");     // s2 進入作用域

    let s3 = takes_and_gives_back(s2);  // s2 移入 takes_and_gives_back
                                        // 該函式又將其回傳值移到 s3
} // s3 在此離開作用域並釋放
  // s2 已被移走，所以沒有任何動作發生
  // s1 離開作用域並釋放

fn gives_ownership() -> String {             // gives_ownership 會將他的回傳值
                                             // 移動給呼叫它的函式

    let some_string = String::from("你的字串"); // some_string 進入作用域

    some_string                              // 回傳 some_string 並移動給
                                             // 呼叫它的函式
}

// 此函式會取得一個 String 然後回傳它
fn takes_and_gives_back(a_string: String) -> String { // a_string 進入作用域

    a_string  // 回傳 a_string 並移動給呼叫的函式
}
```
記住，變數所有權會遵從相同模式：**賦值給其他變數就會移動**。

若是想要讓函式使用數值卻不取得所有權，同時回傳它們自己產生的值呢？
Rust 可以利用元組回傳多個數值：
```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("'{}' 的長度為 {}。", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 回傳 String 的長度

    (s, length)
}
```
不過，這樣做雖然能達到要求，但還是太過繁瑣。
到底有沒有什麼方法可以乾淨的在不轉移所有權的同時使用數值？

這就是下一篇要提的：引用（reference）。

（待續）
2023.02.11