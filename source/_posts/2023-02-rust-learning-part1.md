---
title: Rust 學習筆記第一章 - 基礎概念篇
date: 2023-02-06 23:51:42
categories:
- Learning
tags:
- Rust
---
## 前情提要
上次我們學習如何安裝 Rust，以及利用 Cargo 管理 Rust 專案，
今天則是要介紹 Rust 的變數概念、基本資料型別、函式以及控制流程，也就是常見的程式設計概念。

## 變數與可變性
相信大家對「**變數（variable）**」已經有基本概念，今天特別要說明的是在 Rust 中變數的特性。

### 變數
首先建立一個專案叫做 *variables*，並且編輯 `src/main.rs`：
```rust
fn main() {
    let x = 5;
    println!("x is：{x}");
    x = 6;
    println!("x is：{x}");
}
```
然後 `cargo run`，應該會得到以下輸出：
```sh
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("x 的數值為：{x}");
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

For more information about this error, try `rustc --explain E0384`.
error: could not compile `variables` due to previous error
```
錯誤訊息告訴我們，「cannot assign twice to **immutable** variable」。

**是的，在 Rust 中變數預設是不可變的。**

Rust 預設當我們給予變數一個數值，這個數值是不會被改變的，以確保程式進行的穩定性。
但變數的可變性一向是很重要的，所以當我們需要讓一個變數的數值是可變的，
可以在宣告時加上 `mut` 關鍵字，例如下列程式碼：
```rust
fn main() {
    let mut x = 5;
    println!("x 的數值為：{x}");
    x = 6;
    println!("x 的數值為：{x}");
}
```
再次執行，便能得到正確的輸出。

### 常數
常數也是一個常見的程式設計概念，在 Rust 中，常數與不可變變數的差異如下：
1. 常數是「**永遠不可變**」，所以宣告時不可以使用 `mut`，並且若是使用 `const` 宣告，需要指明型別（在下一段會提及）
2. 常數可以被定義在任何有效範圍，包含全域
3. 常數只能透過常數表達式設置

常數常用於將會擴散到所有程式碼的數值，在未來修改程式時，可以知道需要修改的部份。

### 遮蔽（Shadowing）
我們可以**使用之前的變數再次告新的變數**，而這個動作在 Rust 被稱為**遮蔽**，
代表編譯時編譯器會看到第二個變數的值，並佔據變數名稱的使用權直到該變數生命週期結束（在 Rust 中稱作「離開作用域」）。

下面是一個遮蔽的例子：
```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("x 在內部範圍的數值為：{x}");
    }

    println!("x 的數值為：{x}");
}
```
首先將 `x` 給予 `5`，然後重複使用 `let x = ` 建立新變數 `x`，以 `6`（5 + 1）取代原本數值；
接著在 `{}` 內第三次出現的 `let` 陳述遮蔽了 `x`，所以輸出 x = 12；
離開 `{}` 後，內部的 `let` 造成的遮蔽結束，故 `x` 回到原本的 `6`。

遮蔽與 `mut` 不同之處，在於 `mut` 是將變數本身設為可變，而遮蔽是透過再次宣告改變內部儲存的資料；
另外，可以透過遮蔽，在產生新的變數同時更改型別，例如：
```rust
fn main() {
    let spaces = "   ";
    let spaces = spaces.len();
}
```
在第一次宣告時 `space` 是一個字串，但遮蔽後變成數字；
而 `mut` 是無法做到的。

簡而言之，遮蔽就是對一個變數的「重新宣告」。

## 基本資料型別
在 Rust 中，每個數值都有一個型別，可以告訴 Rust 資料指定的格式，以便 Rust 處理。
需要注意的是，Rust 是一個「**靜態型別**」語言，代表 Rust 在編譯時需要知道所有變數的型別，
儘管與現在多數程式語言一樣，Rust 能依據數值與使用方式推導變數的型別，
但遇上多種可能的型別時，還是需要明確指定。

### 純量型別
純量（Scalar），即是指單一數值，在 Rust 中包含**整數、浮點數、布林與字元**。

#### 整數
不含小數點的數字，依據使用位元大小分為以下數種：
| 長度 | 帶號 | 非帶號 |
|---|---|---|
| 8 bits | `i8` | `u8` |
| 16 bits | `i16` | `u16` |
| 32 bits | `i32` | `u32` |
| 64 bits | `i64` | `u64` |
| 128 bits | `i128` | `u128` |
| architecture | `isize` | `usize` |

Rust 預設使用的整數型別是 `i32`，至於位元長度與可儲存的範圍，
（這個應該是在計概就說過的東西，此處不再提及，真的有需要以後再寫一篇www）；
`isize` 與 `usize` 則是依據系統架構決定（32 位元與 64 位元）。

在 Rust 中，可以透過「**字面值（literals）**」以指定型別，並且可以加上底線 `_` 方便閱讀：
| 字面值 | 範例 |
|---|---|
| 十進制 | 98_222 |
| 十六進制 | 0xff |
| 八進制 | 0o77 |
| 二進制 | 0b1111_0000 |
| 位元組（限 `u8`） | b'A' |

**注意**：若是在變數名稱前加上 `_`，則是使變數成為不可被外部存取（類似 Java `private`）。

#### 浮點數
有小數點的數字被稱為浮點數，在 Rust 中有兩種基本型別：`f32` 與 `f64`（依據 IEEE-754 定義浮點數之精度），
其中預設使用 `f64`，並且所有浮點數都是帶號數。

以上數值皆可以使用基本數值運算之運算子（`+`、`-`、`*`、`/`、`%`）。

#### 布林
在 Rust 也有布林值 `true` 與 `false`，若要使用布林型別則是使用 `bool`，最常用於 `if` 表達式。

#### 字元
Rust 有一個字元型別 `char`，使用方式如下述範例：
```rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // 明確標註型別的寫法
    let heart_eyed_cat = '😻';
}
```
這邊需要提醒，`char` 使用**單引號**賦值（雙引號是**字串**），同時大小為四個位元組並且是 Unicode 純量數值（所以可以儲存亞洲文字甚至 emoji！）。
**特別注意**：一個「字元」並非真正的 Unicode 概念，關於這點往後會有討論。

### 複合型別
複合型別是組合數個數值為一個型別，在 Rust 有兩種基本複合型別：**元組（tuple）與陣列（array）**。

#### 元組（Tuple）
元組有以下特點：
1. 組合不同型別的數值
2. 擁有固定長度，一旦宣告完成便無法增長或縮減

在 Rust 中，元組以 `()` 宣告，每個值以逗號分開；可以進行**模式配對（pattern matching）**以解構元組的數值，取得每個獨立數值。
例如：
```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("y 的數值為：{y}");
}
```

若是需要取值，可以用 `.` 加上索引取得元素。例如：
```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

另外，沒有任何數值的元組被稱為**單元型別（unit）**，數值與型別都寫作 `()`，通常代表空數值或回傳型別；
一個表達式若無任何回傳數值會隱式回傳單元型別。

#### 陣列
陣列就不用多說，我們都很熟悉（特別是資訊大一必學 C，陣列絕對是老朋友了），
不過 Rust 的陣列不同於其他語言，是**固定長度**的，這代表我們可以讓資料安全的分配在堆疊（stack）內；
而之後會提到一個特殊型別————**向量（vector）**，與陣列相似但是允許變更長度。

當確認元素多寡不會改變時，使用陣列是非常好的選擇，例如月份：
```rust
fn main() {
let months = ["一月", "二月", "三月", "四月", "五月", "六月", "七月",
              "八月", "九月", "十月", "十一月", "十二月"];
}
```
在宣告陣列時可以寫出型別，如下列格式：
```rust
fn main() {
let a: [i32; 5] = [1, 2, 3, 4, 5];
}
```
或是將所有元素數值設為相同：
```rust
fn main() {
let a = [3; 5];   // 會產生與 let a = [3, 3, 3, 3, 3]; 相同的結果
}
```

至於獲取元素，跟陣列本身一樣，我們已經非常熟悉，就不再多提。
注意：Rust 會進行索引值檢查以保障記憶體安全，這是相較其他語言，比較特殊的功能。

## 函式
函式是非常普遍的程式概念，不論是程序式或者物件導向都很常見到，Rust 自然也不例外。
在 Rust 中，宣告函式需要使用關鍵字 `fn`，並且與變數相同，使用 `snake case` 命名。

下面是一個包含定義的範例：
```rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("另一支函式。");
}
```
注意，Rust 可以在**作用域的任何地方**定義函式。

### 參數
如同多數程式語言，Rust 函式可以擁有**參數（parameters）**，並且定義在**函式簽名（signatures）**中，
使用函式時就可以傳遞確切數值作為**引數（arguments）**；
我們可以改寫上面的範例：
```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("x 的數值為：{x}");
}
```
注意，在 Rust 中，必須要在函式簽名宣告每個參數的**型別**，這是 Rust 刻意為之的設計，
避免編譯器需要額外花時間尋找資訊以確認使用的型別。

### 陳述式與表達式
函式是由一系列**陳述式（statements）**組成，最後可選擇加上**表達式（expression）**；
而由於 Rust 是**基於表達式（expression-based）**的語言，所以需要特別注意兩者的差異：
- **陳述式**是進行一些動作，並不回傳任何數值
- **表達式**是計算並產生數值

可以看以下範例：
```rust
fn main() {
    let x = (let y = 6);
}
```
若是執行會看到錯誤訊息：
```sh
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error: expected expression, found `let` statement
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^

error: expected expression, found statement (`let`)
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^^^^^^^
  |
  = note: variable declaration using `let` is a statement

error[E0658]: `let` expressions in this position are unstable
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^^^^^^^
  |
  = note: see issue #53667 <https://github.com/rust-lang/rust/issues/53667> for more information

warning: unnecessary parentheses around assigned value
 --> src/main.rs:2:13
  |
2 |     let x = (let y = 6);
  |             ^         ^
  |
  = note: `#[warn(unused_parens)]` on by default
help: remove these parentheses
  |
2 -     let x = (let y = 6);
2 +     let x = let y = 6;
  |

For more information about this error, try `rustc --explain E0658`.
warning: `functions` (bin "functions") generated 1 warning
error: could not compile `functions` due to 3 previous errors; 1 warning emitted
```
因為 `let y = 6` 是一個陳述式，並**不會回傳數值**，所以 `x` 無法得到任何數值。

而表達式會計算出一個數值：
```rust
fn main() {
    let x = 5;

    let y = {
        let x = 3;
        x + 1
    };

    println!("y 的數值為：{y}");
}
```
其中表達式
```rust
{
    let x = 3;
    x + 1
}
```
會回傳 `4` 給 `let y = ` 以進行賦值。
注意 `x + 1` 並沒有加上分號 `;`，
因為表達式結尾並不會加上分號，否則會導致表達式變成陳述式而不回傳數值；
另外在 Rust 中，**數字本身也是一種表達式**。

### 回傳值
在 Rust 中函式可以回傳數值給呼叫者，不過並不會為回傳值命名，而是在函式簽名上使用 `->` 宣告回傳值的型別；
通常回傳值會是函式本體的最後一個表達式，但也可以使用 `return` 提早回傳。

回傳的範例如下：
```rust
fn main() {
    let x = plus_one(5);

    println!("x 的數值為：{x}");
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

## 控制流程
多數程式語言都有「依據某項條件是否為 `true` 執行特定程式碼」，以及「依據某項條件是否為 `true` 重複執行特定程式碼」的功能，
在 Rust 中，能夠讓我們控制流程的常見方法是 `if` 表達式以及迴圈。

### `if` 表達式
`if` 是依據條件判斷產生**分支（arms）**，若是為真則執行後續的程式碼；
另外也可以加上 `else`，在條件不符時執行另一段程式碼：
```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("條件為真");
    } else {
        println!("條件為否");
    }
}
```
需要注意的是，判斷條件必須是 `bool`，否則會造成無法編譯。

#### `else if`：多重條件判斷
`else if` 能夠實現多重條件與分支。
Rust 的用法與其他語言沒有太大差異，就只提供範例：
```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("數字可以被 4 整除");
    } else if number % 3 == 0 {
        println!("數字可以被 3 整除");
    } else if number % 2 == 0 {
        println!("數字可以被 2 整除");
    } else {
        println!("數字無法被 4、3、2 整除");
    }
}
```
不過，上述程式碼只會執行第一個條件為 `true` 的區塊；
若是要實現同時成立一個以上，可以透過 `match` 結構，這個會在往後提到。

#### 在 `let` 陳述式中使用 `if`
因為 `if` 是一種表達式，所以可以用在 `let` 陳述式之中，並將結果賦值給變數：
```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("數字結果為：{number}");
}
```
上述程式碼會將 `if` 表達式運算的數值賦值給 `number`。
需要注意，可能成為結果的每個分支所回傳的值必須要是**相同型別**。

### 迴圈
迴圈跟 `if` 一樣是程式設計的老朋友，
在 Rust 中有三種迴圈：`loop`、`while` 與 `for`。

#### `loop`
`loop` 是不斷執行迴圈直到我們親自告訴迴圈停下來：
```rust
fn main() {
    loop {
        println!("再一次！");
    }
}
```
這段程式碼會一直印出 `再一次！`，也就是產生了**無限迴圈**，可以利用 Ctrl-C 或其他中斷方式強行切斷；
但是學過基本程式設計都知道，無限迴圈是一個很危險的東西，一不小心就會耗盡系統資源……blahblahblah。
不過跟其他語言類似，Rust 有提供中斷迴圈的關鍵字 `break`，以及跳過本次迴圈繼續執行的 `continue`。

##### 迴圈與回傳
`loop` 可以在停止的時候回傳數值，這可以讓我們檢查執行緒是不是完成任務：
```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("結果為：{result}");
}
```

##### 多重迴圈與迴圈標籤
有時候我們會在迴圈之中還有迴圈，這時候 `break` 與 `continue` 會用在最內層迴圈，
如果需要直接跳出外層迴圈， Rust 提供一個方式：迴圈標籤，可以直接針對迴圈進行操作：
```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```
其中 `break 'counting_up` 可以離開外層的迴圈。

#### `while`
`while` 則是「當條件為 `true` 時繼續執行迴圈」：
```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("升空！！！");
}
```
`while` 可以消除很多 `loop`、`if`、`else` 與 `break` 的結構，讓程式碼更容易閱讀。

#### `for`
`for` 多用於**遍歷**集合元素，儘管 `while` 也能做到，但使用 `for` 顯然更為簡潔。
以下是使用 `while` 的狀況：
```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("數值為：{}", a[index]);

        index += 1;
    }
}
```
而這是 `for` 的狀況：
```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("數值為：{element}");
    }
}
```
可以看到 `for` 更為簡潔，並且可以避免程式錯誤（例如超出陣列大小）、增加安全性。

這邊要提到一個運算子：`..` 範圍運算子。
`..` 前後可以放上起點與終點，例如 `1..4` 就是從 `1` 開始，遇到 `4` 停止（所以只會有 1 到 3）。

## 註解
啊好像還有一個東西沒說，但是這個不需要太多篇幅啦（而且寫到這邊我的原始檔已經 519 行了）。

在撰寫程式碼時，我們已經盡力讓程式碼更容易被閱讀，但有時免不了需要額外解釋，這就是**註解（comments）**的功能。
在 Rust 中，註解一律是使用 `//` 開頭，而 Rust 的註解格式，大多是位於說明目標的上一行：
```rust
fn main() {
    // 幸運 777！
    let lucky_number = 7;
}
```
另外 Rust 還有一種特殊的註解：**技術文件註解**，這在很後面提到發布 Crate 時才會用到。

## 總結
序章也才 260 行啊……怎麼這次直接翻倍了，害我說好平均三天一更馬上富奸……
這還只是基本概念而已欸……

這次把基本設計概念（大概就是大一上程式設計前半需要的概念）講完了，
平常可以多練習寫寫看，官方文件提供了幾個題目：
- 溫度轉換
- 產生費波那契數字
- 重複歌詞印出 *The Twelve Days of Christmas*
或是可以去找 LeetCode 題目練習（LeetCode 真的很好玩，雖然題目也很不好解就是了）。

下次要講到 Rust 最特殊的概念：**所有權**，會稍微複雜一點，希望在這之前我能自己讀的更懂（吐血

火山 / Kazan
2023.02.09