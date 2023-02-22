---
title: Rust 學習筆記第四章 - 結構體
date: 2023-02-20 04:13:00
categories:
- Learning
tags:
- Rust
---
## 前情提要
上次我們已經大略討論完 Rust 最重要的概念：所有權，
這次要討論的則是**結構體（structure）**，是很常見的資料結構。
如果學過 C/C++ 大概會很有印象。

## 結構體
結構體與元組類似，兩者都可以擁有多種數值，
但結構體的每個資料部份都需要被命名以表達意義；
這同時允許我們不用依賴資料的順序存取結構體實例的值。

如果要定義結構體，使用 `struct` 關鍵字並命名，
然後在 `{}` 內定義**欄位（fields）**，也就是每個資料部份的名稱與型別。
以下是一個簡單的例子：
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```
定義完結構體，接著就可以在其他地方建立擁有實際數值的**實例（instance）**：
```rust
fn main() {
    let user1 = User {
        active: true,
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```
如上，我們對每個欄位指定實際的數值以建立一個實例；
若是實例可變，可以用 `.` 存取指定的欄位並賦值更改。
注意，整個實例可變才能更改單一欄位，Rust 不允許一個實例中僅有特定欄位被標記為可變。

另外如同表達式，函式本體最後的表達式可以回傳一個新的結構體實例：
```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        email: email,
        username: username,
        sign_in_count: 1,
    }
}
```
不過這段的寫法有一個問題：函式參數名稱與結構體欄位名稱相同，重複太多了；
如果有更方便更簡潔的語法就更好了……

放心，Rust 會告訴你，「我當然有更好的方法囉。」

### 欄位初始化簡寫
當函式的參數名稱與結構體欄位相同，Rust 提供一種語法：**欄位初始化簡寫（field init shorthand）**，
能達到同樣的效果但不需要重複寫出相同的名稱：
```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        email,
        username,
        sign_in_count: 1,
    }
}
```
這樣就可以省略重複的部份了，確實非常簡潔明瞭w

### 結構體更新語法
有時候我們也會希望從現有的實例產生新的實例，並只修改一部分欄位，
這時候就可以使用**結構體更新語法（struct update syntax）**：
```rust
fn main() {
    // --省略--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```
其中 `..user1` 是代表除了上一行指名的 `enail` 欄位以外，其他欄位使用與 `user1` 同樣的值。
你可能會注意到 `user2` 更新語法使用的是賦值的 `=`，
因為更新語法同樣會發生**資料轉移**（見[所有權篇](../rust-learning-part2#移動（Move）、複製（Clone）、拷貝（Copy）
)），這也代表 `user2` 建立之後，`user1` 便會失效。

### 無名稱欄位元組結構體
Rust 支援讓結構體長的像元組，稱之**元組結構體（tuple structs）**。
元組結構體的特徵是欄位**沒有名稱、只有型別**，
多用於命名不同型別的元組、或是不需要對結構體欄位命名時。

元組結構體同樣使用 `struct`：
```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```
我們會注意到儘管 `black` 跟 `origin` 的元素型別組成相同，但這兩者仍然是不同型別，
因為它們是不同型別的元組結構體實例；
另外元組結構體的使用方式與元組類似，可以解構成獨立的部份並透過索引存取。

### 類單元結構體
元組結構體是沒有欄位名稱，那如果連欄位都沒有還算是結構體嗎？
當然！Rust 有一種結構體是沒有任何欄位的，我們稱作**類單元結構體（unit-like structs）**。
類單元結構體的行為與單元型別類似，都是沒有任何除存在型別中的資料，
這樣的結構體通常用在實作**特徵（trait，會在後面討論）**：
```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```
（注意：結構體資料仍然有所有權的概念，所以若是要在結構體內儲存資料引用，會需要**生命週期（lifetime）**的概念。）

學到這裡，我們可以透過一個簡單的範例專案理解結構體的使用。

## 透過範例理解
首先建立一個新專案 rectangles，這個專案會接收矩形的長寬，並計算面積。

### 僅使用變數的狀況
先按照下面的程式碼撰寫主程式：
```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "長方形的面積為 {} 平方像素。",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```
接著編譯執行（`cargo run`），應該能得到正確的輸出；
這是一個邏輯正確的程式，但顯然不夠乾淨與明確，
首先從函式簽名就可以看到問題：
```rust
fn area(width: u32, height: u32) -> u32 {
    width * height
}
```
`area` 應該計算矩形面積，但參數名稱與矩形的關聯卻非常不明顯。
或許我們可以把長、寬組合起來？

### 使用元組重構的情況
我們先用元組重構：
```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "長方形的面積為 {} 平方像素。",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```
我們會發現元組只需要傳遞一個引數，而且引數非常明確是想要計算的矩形；
但同時你會發現閱讀性反而更差了，因為函式內部使用了元組的元素，
而元組有個弱點：**元組無法對元素命名，只有索引**。

我曾經在 [Coding Style](../coding-style) 的文章談過，
一段好的程式碼是讓別人也看得懂，才有辦法進行協作與維護；
若是整個程式碼充斥著無意義的變數名稱與索引，那閱讀起來就非常吃力。

所以我們還有什麼方法可以讓函數的參數擁有意義、內部的函式本體也能建立關聯？
這就是結構體的用處了。

### 使用結構體重構
現在讓我們用剛剛學到的結構體語法重構函式：
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "長方形的面積為 {} 平方像素。",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```
我們定義了一個 `Rectangle` 結構體，
另外有一個 `area` 函式需要參數 `rectangle`，型別是上述結構體的不可變借用；
使用借用是因為我們不希望取走實例的所有權，讓 `main` 可以繼續使用。

如此一來，我們便有一個清楚表達各數值關係的程式了。

### 使用推導特徵實現更多功能
一般我們在 debug 的時候會利用標準輸出的方式去檢視數值，
但是在 Rust 自定義的結構體並不能這麼做：
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {}", rect1);
}
```
上述程式在編譯時便會出錯：
```sh
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
```
因為要能被 `println!` 印出必須擁有 `Display` 特徵。

為了正常輸出，我們可以指定 `Debug` 特徵輸出格式如 `println!("rect1 is {:?}", rect1);`，
不過這樣仍然會得到錯誤，因為我們的結構體沒有顯式實作 `Debug` 特徵，
所以需要加上屬性來修改：
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);
}
```
如此便可以編譯並執行：
```sh
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/rectangles`
rect1 is Rectangle { width: 30, height: 50 }
```
但這樣輸出不好看，若是結構體成長起來會很麻煩，
所以我們會這樣修改 `println!`：`println!("rect1 is {:#?}", rect1);`
便能得到以下輸出：
```sh
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/rectangles`
rect1 is Rectangle {
    width: 30,
    height: 50,
}
```

或者，我們也可以使用 `dbg!` 這個巨集，
但會拿走表達式的所有權（`println!` 只使用引用）並且將訊息顯示到 `stderr`，
印出的是呼叫的檔案與行數：
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```
其中 `30 * scale` 我們也使用 `dbg!` 是因為 `dbg!` 會回傳所有權，
所以我們可以取得相同的數值；
另外我們不希望 `dbg!` 取走 `rect1` 的所有權，所以呼叫時加上引用，
會得到這樣的輸出：
```sh
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/rectangles`
[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
```
如此，我們發現 `dbg!` 是非常實用的一種輸出方式！

我們完成了 `area` 函式，並且確認它只會計算矩形面積，
那如果我們能把這樣的行為與結構體綁定呢？

## 方法語法
關於上面的問題，有一個很好的方案可以解決：**方法（method）**。

方法與函式類似，也是透過 `fn` 宣告，但方法是特別針對結構體定義的。
我們先來嘗試修改 `area` 作為 `Rectangle` 的方法：
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "長方形的面積為 {} 平方像素。",
        rect1.area()
    );
}
```
這裡有幾個需要注意的部份：
#### 1. `impl Rectangle`
這是要實作結構體方法的宣告方式，`impl` 代表 **implementation**，
在這個區塊內所有的方法都會與指定的結構體有關。

#### 2. `fn area(&self)`
注意到參數是 `&self`，如果寫過 Java、Python 的物件會很熟悉，
這個東西是 `self: &Self` 的簡寫，指的是呼叫的結構體實例。
**所有結構體方法的第一個參數都應該是 `self`**；
另外，這裡使用借用是因為我們不希望失去所有權，這在前面討論所有權時已經說明過了。

除了另外定義，我們也可以把結構體的欄位當作方法的名稱：
```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("長方形的寬度不為零，而是 {}", rect1.width);
    }
}
```
這邊我們把 `width` 同時作為方法，判斷是否大於 0；
在 `main` 中當我們呼叫 `width` 時，若是後面跟隨 `()` 就是呼叫方法，反之則是呼叫欄位。

不過一般而言，我們使用欄位名稱作為方法時，通常希望回傳欄位的數值，
這被稱作 **getter**，並且常常將欄位隱藏起來，這跟 public 與 private有關。

### 關聯函式
在 `impl` 區塊內的所有方法都是**關聯函式（associated funtion）**，因為會與結構體相關；
但若是不需要型別實例的話，可以定義沒有 `self` 作為第一個參數的函式，
這種關聯函式就不是方法（因為沒有方法實例）。

沒有方法實例的關聯函式常用於作為建構子產生新實例，
在 Java 有指定的關鍵字 `new`，但在 Rust 並沒有這樣的內建關鍵字，
所以可以自行指定建構子函式的名稱。

呼叫關聯函式的方式是在結構體名稱後使用 `::` 呼叫函式，
這時結構體名稱會被作為**命名空間**。（命名空間會在後續討論）

### 多重 `impl`
每個結構體都被允許擁有多個 `impl` 區塊，一般多半用於**泛型**（會在後面討論）：
```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

## 總結
結構體在 Rust 中是非常實用的語法，能自己定義需要且有意義的型別，
也能夠定義實用的方法、關聯函式，指定結構體可以有什麼行為。

這次結構體相對很容易理解，加上如果有寫物件導向的經驗，會更容易上手。
下一篇我們就要來談談另一個自訂型別的好方法————**枚舉**。

火山 / Kazan
2023.02.22