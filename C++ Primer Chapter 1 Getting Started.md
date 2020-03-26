---
tags: C++
---

# C++ Primer Chapter 1 Getting Started

## 1.1
* 要你寫一個簡單小程式，告訴你什麼是 function，**function definition** 的精準定義，以及 main function 是整個程式最開始執行的地方
* 怎麼編譯程式
* 除此之外還介紹型別(type)
## 1.2
* 粗略介紹 C++ 的 I/O 方式，C++，首先 C++ 並不存在一種 statement 來處理 I/O，而是透過呼叫 library function 的方式來實現 I/O
* 基本的 I/O 運作會用到 library 提供的兩種型別的物件，istream 跟 ostream
* stream 就是一串從 I/O 裝置讀取或寫入的字元，而且這些字元會**循序的**被讀入或寫出 ( A stream is a sequence of characters read from or written to an IO device. The term stream is intended to suggest that the characters are generated, or consumed, sequentially over time.)
* 而實際上使用的 istream/ostream 的物件就是那鼎鼎有名的 cin, cout, cerr, clog
    * 一般來說，cin 處理標準輸入，cout 處理標準輸出，cerr 輸出錯誤訊息，clog 則是輸出程式在執行時的狀態的資訊



* 接下來用一個簡單例子說明 C++ I/O 的使用：
```cpp
#include <iostream> 
int main() {
    std::cout << "Enter two numbers:" << std::endl;
    //上面的這個 statement "執行"了一個 expression
    //expression 的定義：是由一個或以上的「運算元」(operand)和「運算子」(operator)所組成，運算元被運算子做計算之後會得到一個值(value)
    //上面的 statement 裡面的 expression 就是
    //std::cout << "Enter two numbers:" << std::endl
    int v1 = 0, v2 = 0; //初始化是種好的信仰
    std::cin >> v1 >> v2; 
    std::cout << "The sum of " << v1 << " and " << v2 
              << " is " << v1 + v2 << std::endl;
    return 0;
}
```
* 書中在講這段程式碼的重點就是，每一個 statement 裡面會有一個 expression(exp)，exp 是由一堆 operator(運算元) 跟 operand(運算子) 所組成，exp 被計算完之後會有 value；
* 舉個例子，std::cout << "Enter two numbers:" << std::endl; 這個 statement 裡面有 std::cout << "Enter two numbers:" << std::endl 這個 exp。
* 你可能會問說馬的就少了個分號而已是有什麼差別，別急，之後好好把 Primer 看下去，就會懂其中的差異
* 而 exp 內其實也會有 exp，例如上面整個 exp 最左邊的 std::cout << "Enter two numbers:" 就是一個子 exp，這個 exp 是由 << 這個 "output operator"，跟 << 左邊的 std::cout 這個 ostream 物件，還有 << 右邊的 string literal 組成的；而這個 exp 計算之後會把 << 左邊的 operand 當作這個 exp 的 value；
* 重點來了，這個 value 一樣是個 ostream，他就可以繼續當作後面第二個 << 左邊的 operand。
* 意思就是 std::cout << "Enter two numbers:" << std::endl; 會這樣計算：
    * (std::cout << "Enter two numbers:") << std::endl;
    * 等價於：
        std::cout << "Enter two numbers:";
        std::cout << std::endl;
        
        
        
* 接下來是說 namespace 的概念，不過之後的章節才會詳細解釋
    * 注意看 std::cout 的 std::，這個 prefix 代表說，cout 是定義在 std 這個 namespace 裡面的。這樣可以防止自己定義的 name 跟 standard library 的 name coliision(namespace 到底要怎麼用之後章節會介紹)。
    * ch3.1會講怎麼使用更簡單的方法來用 cout。

* 再來換介紹 cin，跟 cout 有 87%像，不過他是 istream，>>(input operator)左邊要擺 istream object，>> 回傳左邊的 istream object。
        
## 1.3
* comment 粉重要， They are typically used to summarize an algorithm, identify the purpose of a variable, or clarify an otherwise obscure segment of code. 
* An incorrect comment is worse than no comment at all because it may mislead the reader.
* 一般用 C++ 的 comment 大 guy4 這樣:
    * ![](https://i.imgur.com/2KXAfCr.png)

## 1.4
介紹一些 flow control statement
 * while 是由一個 condition 跟 statement 組成
     ```cpp 
         while (condition)
            statement
    ```
     *  A condition is an expression that yields a result that is either true or false.
     *  注意 block statement，也就是 { statements }，也是 statement。

* for
    * for 是由一個 header 跟 statement 組成
    ```cpp
        for(header)
            statement
    ```
    * The header itself consists of three parts: an init-statement,a condition,andan expression.
        * 注意 init-statement, condition, expression 他們各自的精確定義

* 接下來介紹使用 loop 來讀取未知數量的 input
    ```cpp
        while (std::cin >> value)
    ```
    * 94這樣啦!
    * 好啦，如果執行上面這個 while 的 condition，就會計算 std::cin >> value 這個 exp
    * 計算這個 exp 時，除了會把標準輸入的資料導入 value 以外，還會產生一個 exp 的 value
    * 如同前面介紹的，exp 的 value 會是 >> 左邊的 istream
    * 所以這個 condition 變成在看 istream 的 value
    * 重點來了，當我們把一個 istream 當成 condition 時，他真正的效果是測試 istream 的狀態(state)，看狀態是否為有效(valid)
    * 一個 istream 的狀態會在沒有東西可以讀取(End of file) 或者讀取的資料不符合預期時變成 invalid
    * valid|invalid 就會計算成 true|false

* if statement
    ```cpp
    if(condition)
        statement_1
    else if(condition)
        statement_2
    ...
    else
        statement_i
    ```
    *
    
    
## 1.5
* Define our own data types!
* Primer 定義了一個 class 叫做 Sales_item，然後把詳細定義放在 Sales_item.h 這個檔案
* 接下來說明這個 class 有哪些 API 可以用
    * isbn 這個 function 可以取得一個 Sales_item 的 ISBN
    * 定義了 >>, <<, =, +, += 在 Sales_item 上的行為

## 1.6
    實際上用用看 Sales_item 這個 class



