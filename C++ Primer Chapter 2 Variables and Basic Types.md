---
tags: C++
---

# C++ Primer Chapter 2 Variables and Basic Types
此章節介紹圍繞在 type(型態) 的幾個重要概念，以及介紹一些 built-in type

## 2.1 Primitive Built-in Types
### 2.1.1 Arithmetic Types
C++ 定義了幾種基本型別，這幾種基本型別又可以分為兩大類：arithmetic types(int, float, etc) 跟 void
* 阿就介紹幾種 type，標準怎麼定義他們的大小，以及 **先有形態才有value** 的概念
  再來介紹，除了 bool，intergral types 分成 signed 跟 unsigned types
  * 不過 char 又有分三種，分別是 char, signed char 跟 unsigned char。
      * 歷史包袱、歷史包袱...
      * **In particular, char is not the same type as signed char.**
      * 但更ㄎㄧㄤ的是，雖然有三種 char types，但電腦只有兩種表示方式阿！那到底在工三小？
      * 換句話說 char 這個 type **實際上** 可以自由挑選到底要用 signed char 的表示方式或者 unsigned char 的表達方式，取決於 compiler，**再換句話說，你不可以假設你的 char 是其中一種表達方式**。
* 還有標準其實也沒有規定 signed 到底是要用什麼表示方式實作，標準也只有規定說負數跟正數的範圍要一樣大，比放說 8bits 可以表示 -127~+127 之類的；
    * 實際上現在架構都用 2補數，這樣負數的範圍會比正數多 1。

### 2.1.2 Type Conversions
* \型態轉換/
* Type conversions happen **automatically** when we use an object of one type where an object of another type is expected. 
    * **HOW?**
    * **YOU NEED TO UNDRSTAND!**
    * 第四章回有詳細介紹，不過這裡先提一點
* What happens depends on the range of the values that the types permit:
    * 詳細內容請看 Primer 的解釋
* 最後給了一個 advice：**AVOID UNDEFINED AND IMPLEMENTATION-DEFINED BEHAVIOR**
    * 請自己去了解 UB 跟 IB 的細微差別，你可能可以對「標準怎麼定的」這件事更有感覺
    * UB 跟 ID 的差別：
        * 一個是標準完全沒有給定義
        * 一個是標準有給定義，但是他是"給一個(最小)範圍"，compiler 實作在這個標準範圍都可以，而你使用的 compiler(也就是 implementation, 實作)必須要完整給出這些標準定義的 IB 的行為
    * **CAUTION:DON’T MIX SIGNED AND UNSIGNED TYPES**

### 2.1.3 Literals
https://en.cppreference.com/w/cpp/language/integer_literal
*  Every literal has a type. 
*  Integer literals
    *  有8進位，10進位，16進位的寫法
        *  然後到底一個 integer literal 的**型態是什麼取決於他的 value 跟表示方式。**
            *  你可以打兩個 range 在 int 跟 range 在 long 的整數，然後在 VScode 上面滑鼠移過去看他們的型別，你會發現一個是 int 一個是 long
            *  可是如果你用 8進位或 16進位表示，有可能會變成 unsigned...
            *  但是你可以強制在 literal 加上 suffix 來明確指定 literal 的型別
            *  Primer：技術上來說，decimal literal 永遠不會是負的，如果你寫 -42 這樣的東西，negative sign 並不是 literal 的一部分。negative sign 在這個時候是一個 operator。

* Floating point literals
    * 有小數點跟科學記號兩種表示方式
    * By default 是 double 型別
    * 一樣可以用 suffix 改成 float 型別
* Character and Character String Literals
    * ’a’ // character literal 
        * A character enclosed within single quotes is a literal of type char
    * "Hello World!" // string literal
        * The type of a string literal is array of constant chars，array 之後會講(?(XD
        * The compiler appends a null character (’\0’) to every string literal.
        * String literal 可以 concat
            * std::cout << "a really, really long string literal " "that spans two lines" << std::endl;
        * Escape Sequences
            * newline \n
            * vertical tab \v
            * ...etc.
* Boolean and Pointer Literals
    ```cpp
    true false nullptr
    ```
    
## 2.2 Variables
* variable 就是有名稱的儲存空間(named object)
* 每一個 variable 都有型別
* variable 的型別決定的 variable 使用的 memory 的 layout(擺設) 以及 size(大小)，variable 可以儲存的 value 的範圍，還有可以對這個 variable 所進行的操作(operations)
* 有時候會把 variables 就稱作 objects
### 2.2.1 Variable Definitions
```cpp
    typespecifier variable_name [= initial_value] [, variable_name [= initial_value] ...];
```
* Initializers
    * 嚇人的東西...
    * Initialization in C++ is a surprisingly complicated topic and one we will return to again and again. 
    * Initialization 看起來跟 assignment 很像，**可是在 C++ 這是不一樣的 operation**，Primer 非常強調這件事，到底怎麼個不一樣，請慢慢看下去...

* List Initialization
    * Initialization 很複雜的其中一個原因，就是光是語法就有很多種:
    ```cpp
    int units_sold = 0; 
    int units_sold = {0}; //C++11
    int units_sold{0}; //C++11
    int units_sold(0);
    ```
    * 以上全部都是法 units_sold 的值初始化成 0
    * 這種寫法有一些特性:
        1. 如果你用 range 比較大的型別來當 list initialization 的 initializer，會噴 error
            * e.g.
            ```cpp
                long double ld = 3.1415926536; 
                int a{ld}, b = {ld}; // error: narrowing conversion required
                int c(ld), d = ld; // ok: but value will be truncated 
            ```
            * 你可能會覺得上面的例子太ㄎㄧㄤ，不過 Primer 說到 16章時會好好解釋這種由大型別轉到小型別的狀況可能會不小心發生(寫糞code?)
    * 這些很ㄎㄧㄤ的 initialization 方式到第三章才會詳細說明
* Default Initialization
    * 當你在宣告變數時沒有指定 initializer，那個變數會使用那個型態定義的 defualt value 做初始化
        * 額外補充，如果你變數的型別是 C++ 標準物件，那都有一些很好的 default value，但如果你的型別是 built-in type，那就要看**你是在哪個地方宣告這個變數的**，如果在全域，預設是 0，如果在 local，那 C++ 不幫你初始化，變數內容是 undefined

### 2.2.2 Variable Declarations and Definitions
* 這也是重要到靠北的概念
* 他的其中一個目的是可以讓我們把 code 分成有結構的一堆檔案，然後**可以獨立編譯**
* 但是要把 code 分開，就需要有 share code 的機制：
    * 讓某檔案的 code 可以用另一個檔案的變數
        * 舉個最簡單的例子，std::cin 不是你寫的，他在別的檔案裡，可是你可以用它
    * etc.
* **C++ 利用將宣告(declarations) 跟定義(definitions) 分開，來做到上述說的 share code 的機制，**
* A **declaration makes a name known to the program.** A file that wants to use a name defined elsewhere includes a declaration for that name.
* **A definition creates the associated entity.**
    * A variable **declaration specifies the type and name** of a variable.
    * A variable **definition is a declaration**. In addition to specifying the name and type, a **definition also allocates storage and may provide the variable with an initial value.**
    * 用 extern 這個 keyword 可以做到**宣告**某個變數但是**不定義**某個變數
        ```cpp
            extern int i; // declares but does not define i 
            int j; // declares and defines j
        ```
    * Any declaration that includes an explicit initializer is a definition
        * 一個宣告如果有帶 initializar，那他就是 definition
        * 你也可以寫 extern int i = 100;，只是這樣就會把 extern 的效用蓋掉(overrides)
            * https://stackoverflow.com/questions/17090354/why-does-initializing-an-extern-variable-inside-a-function-give-an-error 看到這篇
        * 如果在 function 內用 extern，代表 extern 後面接的名稱是定義在 function 外頭的；那這是時就很理所當然地不能在 function 內有 extern 的宣告內加上 initializer，不然會噴 error:
            ```cpp
            int i;
            void foo()
            {
                extern int i = 10;
            }
            void bar()
            {
                extern int i = 11;
            }
            ```
        * 為什麼不行？其實看上面的例子就明白了，如果可以的話，請問 i 到底要在初始化的時候給 10 還是 11 呢？
    * Variables must be defined exactly once but can be declared many times.
        * 變數可以宣告很多次，但(在同個 scope 下)只能定義一次。
### 2.2.3 Identifiers
    老生常談...
    
### 2.2.4 Scope of a Name
* 這也是超重要議題
    * 我怎麼覺得每個都很重要...
    * Scope 就是在整個程式的程式碼的某些部分，某個 name 有特別的意義
    * 在不同 Scope，同樣的 name 會有不同意義
    * 一個 name 在他被 create 時，一直到程式執行到屬於這個 name 的 scope 結束的地方為止，都是可見的(visible) (非常他媽饒口)
    * 用下面這個例子來說明 scope:
        ```cpp
        #include <iostream>
        int main() {
          int sum = 0; // sum values from 1 through 10 inclusive
          for (int val = 1; val <= 10; ++val)
            sum += val; // equivalent to sum= sum+val
          std::cout << "Sum of 1 to 10 inclusive is " << sum << std::endl;
          return 0;
        }
        ```
        * 這個程式使用到了五個 names，main sum val cin cout，其中 cin cout 是標準定義的 name。
        * main 在 global scope，global scope 就是在任何 {} 之外的 scope
            * 只要在 global scope **宣告** 一個變數名稱，那到程式結束為止之前這個 name 都是 visible
        * sum 有 block scope
        * val 也有 block scope，不過他的 scope 在 sum 的 scope(也就是 main function) 的裡面
* Nested Scope
    * Scope 可以包含其他 Scope
        * 被包含的叫 inner scope，包含人的叫 outer scope
        * inner scope 可以使用 outer scope 已經宣告的 name
        * 在 outer scope 定義過的 name 可以在 inner scope 重新定義(這時 inner scope 的這個 name 是指向不同的 entity)
        * 但是你如果真的重用名字通常都是糞 code 啦...

## 2.3 Compound Types
* A compound type is a type that is defined in terms of another type.
* 又是一個對新手很殘忍的定義
* 這章會講 pointer 跟 reference 這兩種 compound types
* 前面說過，變數宣告是由一個型別名稱加上一堆的變數名稱形成的
    *  More generally, a declaration is a **base type** followed by a list of **declarators**. 
    *   Each declarator names a variable and gives the variable a type that is **related** to the base type.
        ```cpp
            type_name var_name;
        ```
*   在前面的例子，我們的 declarator 就僅僅只是變數名稱而已；這樣的 declarator 給予變數名稱的型別就直接是 base type(上面的 `type_name`)
*   但之後會介紹到，我們可以使用更複雜的 declarator，讓我們的變數名稱的型別變成 compound type，而這個 compound type 是建構在 base type (`type_name`) 上的
### 2.3.1 References
**先提醒這是 lvalue reference，跟 C++11 新定義的 rvalue reference 無關**
* A reference defines **an alternative name for an object.**
* A reference type **“refers to” another type.**
* 語法：
    ```cpp
        type_name &var_name;
    ```
* 多加一個 & 在變數名稱前面
    * base type 是 `type_name`, declarator 是 `&var_name`
* 注意，reference type 一定要初始化!
    ```cpp
    int ival = 1024; 
    int &refVal = ival; // refVal refers to (is another name for) ival 
    int &refVal2; // error: a reference must be initialized
    ```
* 一般型別在初始化時，都是把 initializer 的 value copy 到 var_name 裡，可是 reference 的初始化是說，把 `var_name` **bind** 到他的 initializer。
* 而且 reference 被 bound 之後就不能再被 bound 其他 object。
* 因為不可能在宣告 reference 之後把 reference bind 到其他 object，所以 reference **一定要在宣告的時候做初始化**。
* 一個可能比較好懂的解釋：
    * A reference is not an object. Instead, a reference is *just another name for an already existing object.*
* 當 reference 被 bound 到某物件之後，你對 reference 做的任何操作就會真的發生在 reference bind 的物件上
* 然後我們不能定義一種 type 叫做 reference to reference to what，因為(第二個) reference 不是物件不能被 bound；之後要說的 pointer 是可以的。

### 2.3.2 Pointers
這是 C 姊姊的陰毛，可以不要打筆記ㄇ
提一個，沒有 pointer to reference。
還是有專屬於 C++ 的東西，例如 nullptr，這是 C++11 新定義的常量，跟 C 的 NULL 很像，新標準建議用 nullptr
* nullptr is a literal that has a special type that can be converted (§ 2.1.2, p. 35) to any other pointer type
請 initialize 所有的 pointers

### 2.3.3 Understanding Compound Type Declarations
講怎麼宣告變數，以及如果同時宣告多個變數，每個變數的型別會因為 declarator 的樣子而有所差別

## 2.4 const Qualifier
* 一定要初始化
* 初始化給的 exp 就跟一般變數一樣可以是任意 exp
* **initialization 又分為 compile time 跟 run time**
* A const type can use most but not all of the same operations as its nonconst version.
* 這還會牽扯到之後會講的 `constexpr`... 恐怖

### 2.4.1 References to const
之前有說過，宣告的時候，declarator 的型態一定要跟 type_name 一致(? 不是 related 就好嗎?)，不過有例外，其中一個例外就是 const reference(reference to const)

### 2.4.2 Pointers and const
跟 references to const 有 87%像，不過 pointers 是物件，換句話說真的有 const pointer 這種東西

### 2.4.3 Top-Level const
* 我們把 pointer 自己是否為 const 跟他指向的東西是否為 const 分別稱為 Top-Level const 跟 Low-Level const。
* 而一般型別(不包括 reference)，只有 Top-Level const，因為 Top-Level const 換句話說就是在說明物件本身(object itself)是不是 const 物件；
* 而 reference 型別則只有 Low-Level const，因為 reference 本身不是物件，不是物件根本沒有是否為 const 的問題；但他一定得 bind 在某個物件上，所以他有 Low-Level const
* When we copy an object, top-level consts are ignored
    * 換句話說你把一個物件 A copy 到物件 B 時，你根本不用管 A 是不是 const
* 但是 Low-Level const 就不會被無視惹
    * 沒 (low level const) 的可以丟給 (low level) const，可是有 (low level) const 的不可以丟給沒 (low level) const 的(?

### 2.4.4 constexpr and Constant Expressions
大．魔．王
* A **constant expression** is an expression whose **value cannot change** and that **can be evaluated at compile time.**
    * can be evaluated at compile time.
    * can be evaluated at compile time.
    * can be evaluated at compile time.
* Literal 就是
* A **const object** that is **initialized from a constant expression** is **also a constant expression.**
    * 請仔細鑽研這句話，很純
    * 你 depends on 的 expression 全部都可在 compile time 就算出來，你當然也可以在 compile time 就算出來!
* 一個 object 是不是 const expression，**同時取決於** 他宣告時的 type 以及 initializer。
* Example
    ```cpp
    int main() {
      const int max_files = 20;        // max_files is a constant expression
      const int limit = max_files + 1; // limitis a constant expression
      int staff_size = 27;             // staff_size is not a constant expression
      const int sz = get_size();       // sz is "not" a constant expression
      /*
       * Although staff_size is initialized from a literal,
       * it is not a constant expression because it is a plain int,
       * not a const int.On the other hand, even though sz is a const,
       * the value of its initializer is not known until run time.Hence,
       * sz is "not" a constant expression.
       */
    }
    ```
    * 上面第三個跟第四個例子，尤其是第四個，好好理解之後會對 `constexpr` 更有感覺
#### constexpr functions
* 第六章會講
* **BEST PRACTICE**
    * Generally, it is a good idea to use `constexpr` for variables that you intend to use as constant expressions.
* Literal Types
    * `constexpr` 只能用在 litereal types，而 literal types 有 arithmetic，reference，pointer 還有之後章節(7.5.6 and 19.3)會介紹的幾種。
        * 其他的都不是！例如 IO 或者 string
        * **提示，這是取決於 `constexpr` 在 class 上的規則**
    * reference 跟 pointer 要能成為 `constexpr`，只能去 bind 或指向某種 variables。
        * 哪種？ global object  還有 function 的 static variables 的 address 都是 `constexpr`，其他 object 的 address 都不是，所以 `constexpr` reference 跟 pointer 只能 bind 或指向這種 variables
        * 不過 `constexpr` pointer 還可以直接指向 nullptr 或者給一個 literal address 就是，但這通常沒有三小路用
        * 指標還有一點要注意：
        ```cpp
        constexpr int *q = nullptr;
        ```
        * `constexpr` 是作用在 Top-level(也就是指標上)的，而不是 Low-level(也就是說指標指向的型別)。
## 2.5 Dealing with Types
### 2.5.1 Type Aliases
* 傳統的 typedef
    ```cpp
    typedef double wages; // wages is a synonym for double 
    typedef wages base, *p; // base is a synonym for double, p for double*
    ```
    * The keyword `typedef` may appear as **part of the base type of a declaration**
        * `typedef double wages;`, 其中 `typedef double` 是這個宣告內的 base type
    * Declarations that include `typedef` define type aliases rather than variables.
        * 如果你的宣告的 base type 裡面有 `typedef`，這樣就是在定義 type aliases，而不是一個新的 object
        * 這時候 declarator 一樣可以包含 type modifier(*, & 之類的)
            * 則這時 declarator 裡面的 name 就可以當作一個型態，其形態為 base type + 它的 modifier
* C++11 的新方式，叫做 alias declaration
    ```cpp
    using SI = Sales_item; // SI is a synonym for Sales_item
    SI item; // same as Sales_itemitem
    using func = void(*)(int);  // 開掛先講函數指標，這樣有沒有比 typedef 好懂w?
    ```
### 2.5.2 The auto Type Specifier
* C++11 的 auto
    * 讚！
    * let the compiler figure out the type for us by using the auto type specifier. 
    * **deduce the type from the initializer**
    * 換句話說，你要用 auto 宣告某個變數，你一定要給 initializer！
    * 注意：The type that the compiler infers for auto is **not always exactly the same as** the initializer’s type. 
        * Instead, the compiler adjusts the type to conform to normal initialization rules.
        * auto 會 ignore initializer 的 Top-level constant
        * 但 Low-level constant 一樣會保留
        * 而且如果你給的 initializer 實際上是 referece 的話，auto 會用 referece bind 的 object 來推斷型別
        ```cpp
        int i=0, &r= i; auto a = r; // a is an int (r is an alias for i, which has type int)
        const int ci = i, &cr = ci; 
        auto b = ci; // b is an int(top-level const in ci is dropped) 
        auto c = cr; // c is an int(cr is an alias for ci whose const is top-level) 
        auto d = &i; // d is an int*(& of an int object is int*)
        auto e = &ci; // e is const int*(& of a const object is low-level const)
        ```
        * 例如上面的 `auto c = cr;`，`cr` 是 reference to const int，那就會轉成去看 const int；而 top-level constant 又會 ignore，所以 `c` 的型別就是 int
    * 如果你希望 `auto` 可以有 top-level constant，你要明確寫出 `const auto var_name = obj;`，這樣 `var_name` 就會是 const obj's type
    * 如果你希望你用 `auto` 宣告的變數是一個 reference，你要寫 `auto &var_name = obj;`，這時 `var_name` 就會是 reference to obj's type, 而且 bind 在 obj。
    * 注意：When we ask for a reference to an auto-deduced type(也就是 `auto &var_name = obj;`), **top-level consts in the initializer(`obj`) are not ignored.**
        * initializer 的 top-level 之後就會變成 reference 的 low-level
### 2.5.3 The `decltype` Type Specifier
另一種類似 auto 的東西
使用方式:
* decltype(operand) var_name = initializer;
* Compiler 會去分析 operand 的 type，**但是不會計算(evaluate) operand**
* **The way `decltype` handles top-level const and references differs subtly from the way auto does.**
* decltype 的 operand 如果是 variable，他回傳的型態會包含 variable 的 top-level constant；如果 variable 是 reference，他回傳的型態就會包含 reference。
```cpp
    const int ci = 0, &cj = ci;
    decltype(ci) x = 0; // x has type const int
    decltype(cj) y = x; // y has type const int& and is bound to x
    decltype(cj) z;    // error: z is a reference to const int and must be initialized
```
#### decltype and References
* 那如果 operand 不是 variable 呢?
    *  we get the type that that expression yields.
        *  不過有些 exp 還是會推導出 reference type(就是可以當 lvalue 的 exp)
        ```cpp
        // decltype of an expression can be a reference type 
        int i = 42, *p = &i, &r = i; 
        decltype(r + 0) b; // ok: addition yields an int; b is an (uninitialized) int
        decltype(*p) c; // error: c is int& and must be initialized
        ```
        * 看上面的 decltype(*p) c;，*p 可以當 lvalue，這樣 c 的 type 就是 int&
* 還有一個雷點：
    * Another important difference between decltype and auto is that the deduction done by decltype depends on the form of its given expression.
        * `*p` 跟 `(*p)` ㄏㄏㄏㄏ

## 2.6 Defining Our Own Data Structures
Primer 的例子是定義很基本的 Sales_data 這個 struct(注意不是 Sales_item，他沒有實作 operation，所以先叫他 Sales_data)

### 2.6.1 Defining the Sales_data Type

```cpp
struct Sales_data { 
    std::string bookNo; 
    unsigned units_sold = 0; 
    double revenue = 0.0;
};
```

其中每個 member 如果有提供 initializer 的話，member 會在 class 的 object 被 created 時按照 initialzer 作初始化。如果沒有 initializer 就作 default initialization。這種 initializer 又叫做 **in-class initializer**，是 C++11 new feature(居然!)

### 2.6.2 Using the Sales_data Class

### 2.6.3 Writing Our Own Header Files
* There may be only one definition of "that class"(正確來說，whatever) in any given source file.
* In order to ensure that the class definition is the same in each file, classes are usually defined in header files.
* Header guards