---
title: 良好的 Coding style
date: 2022-09-26 07:29:05
categories:
- Others
tags:
- coding
---
## 前情提要
上一篇聊到同學們 code 的排版造成我很嚴重的視力及心理傷害（無誤），
這篇正好來聊聊一個好的 coding style 是什麼吧！

***注意：本篇所有論點皆屬個人見解，若不認同請保持理性討論。***

## 什麼是 coding style？
Coding style 指的是「程式碼的『寫作』風格」，
如同一般的文章風格一樣，程式碼的寫作風格也會因人而異，
小至空格的使用、大括號 `{}` 的位置，大至函式的宣告、語法的選擇都會有不同的個人風格。

**那為什麼要強調「好的」coding style？**

如果初學程式設計的人，可能會在教材上看到大部分高階程式語言有個特性——「可讀性」，
也就是容易閱讀、容易理解，這也造就了現代的開發協作環境。
但我個人認為這樣的特性需要配合良好的撰寫習慣才能發揮到最大，
特別是在幫別人 debug 的時候，如果你看到一個雜亂無章、文字過度集中的程式碼，
真的會跟我一樣眼睛很痛（o

以下是我覺得很重要的幾個各語言都通用的 coding style：
### 空格與段落
首先給各位看個範例：
```C
int main(void)
{
    int n=0;
    printf("%d\n",n);    
    return 0;
}
```
這個看起來很正常的程式碼有什麼問題呢？

先不急著解答，我們來看看另一個人寫的：
```C
int main(void) {
    int n = 0;
    printf("%d\n", n);

    return 0;
}
```
有發現細微的差異了嗎？
沒錯，就是**「適當的空格與段落」**。

在絕大部分語言中，運算子、參數之間的空格，以及分隔段落的空行並非必要，
編譯 / 直譯器最後都會把這些空格 / 行無視掉，
那為什麼還要加這些東西增加行數？（關於程式碼行數這又是另一個故事了）

**因為雖然機器看得懂，但人也要看得懂跟舒服啊。**

上面兩個程式碼拿給任一個人看，大多數都會說後者看起來比較舒服，
也會比較有耐心（？）幫忙 debug，這說明**沒有適度的排版確實會影響閱讀體驗**；
簡單說，你今天要拜託人看程式碼<，結果因為你寫得很醜反而讓對方很困擾，
這不是造成人家更麻煩、你自己也更麻煩嗎？
如同寫字一樣，寫程式碼最基本的就是整齊易讀。

最後給各位看個非常極端的案例：
```C
#include <stdio.h>
#include <stdlib.h>
int binSearch(int arr[],int target,int left,int right);
int compare(int x,int y);
int main(void){
int arr_size,target;
scanf("%d",&arr_size);
int* list=malloc(sizeof *list*arr_size);
for(int i=0;i<arr_size;i++){
scanf("%d",(list+i));}
scanf("%d",&target);
int tar_index=binSearch(list,target,0,arr_size-1);
printf("%d\n",tar_index);
return 0;}
int compare(int x,int y){
if(x<y){
return -1;} 
else if(x==y){
return 0;} 
else{
return 1;}
}
int binSearch(int arr[],int target,int left,int right){
int middle=(left+right)/2;
printf("%d %d %d\n",left,middle,right);
printf("%d %d %d\n",arr[left],arr[middle],arr[right]);
if(left<=right){
switch(compare(arr[middle],target)){
case -1:
return binSearch(arr,target,middle+1,right);
case 0:
return middle;
case 1:
return binSearch(arr,target,left,middle-1);}}
return -1;
}
```
~~如果能堅持看完全部的人我佩服你~~

### 變數、函數、類別的命名
這是更多初學者會犯的錯誤——變數**胡亂命名**或是**命名無意義**。

常常會在一些小朋友寫的程式碼裡面看到諸如 `int a, b` 之類的命名滿街跑，
每次在 debug 都要問好多次「這變數是要存什麼」、「這個 function 是在做什麼」、「這類別是什麼東西」，
問到最後甚至會得到這樣的回應：「呃我忘了耶。」~~忘記了你還叫我 debug？我不會通靈啊ㄍ！~~

一個好的命名，是能夠讓人一目瞭然這東西的用途，進而可以更快的發現問題所在；
一個壞的命名，可能會讓你自己都搞不清楚這在幹嘛，結果就是程式 bug 一大堆，然後你也不知道從何處理。

所以什麼是好的命名方式？

#### 1. 有意義的名稱
舉個例子：
當你的程式需要儲存一筆「金額」的資料，一般人可能就 `int a` ，
最多用註解告訴大家這個 `a` 是放金額的變數，但誰知道 `a` 跟金額的關聯？！
~~如果在 debug 的時候絕對會聽到以下哀號：「這 a 是在幹嘛啊啊啊（無限慘叫）？」~~
所以用有意義的名稱命名是很重要的！
以上面的例子，命名為 `int mount` 會是比較好的做法；若是需要，也可以組合兩個甚至以上的「名詞」作為命名。
但也不是所有單字都可以用，盡量避免使用以下的字詞：
- 語言保留字（如 Python `input`）
- 單一動詞，通常也會是保留字
- 無意義的單字，容易造成干擾（`temp` 之類的）
- 無意義的前綴
另外也要避免命名過長，否則溝通上會有困難。

#### 2. 大小寫識別
有意義的名稱固然重要，但若是全部都大寫或全部都小寫，其實也是有缺點的，
因為仍然會造成閱讀及識別上的困難。
例如：`int arraysize` 這是個有意義的組合命名，但是不容易閱讀。
所以一般在命名時會建議如下：
- 變數、函式 / 方法：小寫開頭，以底線連結如 `first_name`（此即 snake case），或是第二個單字首字母大寫如 `lastName`（即「**駝峰式命名法**」）。
- 常數：全部大寫，如 `PI = 3.14`、`MAX_SIZE`
- 類別：大寫開頭，如 `Display`、`Phone`

#### 3. 命名一致
這裡的一致指的是「定義一致」，也就是對單一對象的定義；
舉例：定義使用者叫 `user`，那有關的其他命名也要以 `user` 為主，如使用者的存款可以叫 `userBalance`，
不要變成 `accountBalance` ，否則會造成一定程度的混亂。

#### 4. 術語正確
這沒什麼好說的，就是用正確的專業術語，否則會造成極大的混淆。

### 建立 coding convention
說了這麼多，最重要的是建立一個 coding 的慣例（convention），並且整份專案都要遵守這個慣例，
這在多人合作的專案中更為重要，否則大家都用自己的方式寫，
結果就是會造成每個人都不知道其他人在寫什麼，最後整合的時候就會大混亂。

## 結論
~~好的 coding style 帶你上天堂，壞的 coding style 讓你住套房~~

寫出一份漂亮的 code 不只對自己維護專案方便，也能讓別人在幫助你的時候更加輕鬆，
所以何不從現在開始練習，養成良好的 coding style 吧！

火山 / Kazan
2022.09.27