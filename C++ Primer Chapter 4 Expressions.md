---
tags: C++
---

# C++ Primer Chapter 4 Expressions
* **An** expression is composed of one or more **operands** and **yields a result** when it is evaluated. 
* More complicated expressions are formed from an operator and one or more operands.
## 4.1 Fundamentals
### 4.1.1 Basic Concepts
* unary/binary operators
* one ternary operator
* call operator, takes unlimited operands
* Some symbol is both unary and binary operator, depending on context
    * it can be helpful to think of them as two different symbols.
#### Grouping Operators and Operands
* operator 彼此之間還有 **優先序(precedence)** 跟 **結合律(associativity)** 以及的概念 **求值順序(order of evaluation)** 概念
#### Operand Conversions
* 當你某個 operator 期待他的 operand 是某些型態，但實際上 operand 卻是別種型態時，operand 會被做轉型(如果可以的話)
* promote 的概念：
    * 小型別轉大型別
        * bool, char to int, long long
        * 到底怎麼 promote 其實有定義，之後會講
#### Overloaded Operators
* 語言已經定義好對那些 operator 跟 built-in type 作用時的行為
* We can also define what most operators mean when applied to class types.
* 因為這樣就像是在原本的 symbol 上加上另一種意義，所以叫做 overloaded operators
* 例子：
    * >>, << 作用在 string 上跟作用在 built-in type 上有不同的意義跟行為
* 不過 C++ 拿來當作 operator 的那幾個 symbols 的運算子數目，優先序跟結合律都不能被改變
#### Lvalues and Rvalues
* Every expression in C++ is either an rvalue or an lvalue
    * lvalues could stand on the left-hand side of an assignment whereas rvalues could not.
* Roughly speaking, when we use an object as an rvalue, we use the object’s value (its contents). When we use an object as an **lvalue**, we use the object’s **identity** (its location in memory).
* Operators **differ** as to **whether they require lvalue or rvalue operands and as to whether they return lvalues or rvalues.**
* 舉幾個已經學到會用到 lvalue 的例子
    * Assignment(=) return left-hand operand as lvalue
    * The address-of operator(&) **requires** an **lvalue operand** and **returns a pointer to its operand as an rvalue.**
    * The built-in dereference(*) and subscript operators([]) and the iterator dereference and string and vector subscript operators all yield lvalues.
    * The built-in and iterator increment and decrement(++, --) operators **require lvalue operands** and the prefix versions (which are the ones we have used so far) **also yield lvalues.**
* 以後要使用某個 operator 除了看 operand type 之外還要看 operand 是否為 l/rvalue，有沒有符合 operator 所需
* 幹，同樣 type，l/r 不同，用在 decltype 也會不一樣
    * 媽的他真的好用嗎= =
    * 如果 operand 是 lvalue，那 type 就會推斷為 reference
    ```cpp
    int i = 0;
    int *p = i;
    decltype(*p) i_ref = i; // *p is lvalue，decltype(*p) is int&
    decltype(&p) i_pp = nullptr; // &p is rvalue, decltype(&p) is int**
    ```
### 4.1.2 Precedence and Associativity
* **compound expression**
    *  An expression with two or more operators
    * Evaluating a compound expression involves grouping the operands to the operators.
    * 計算 compound expression 就會需要根據一套規則把部分的 operands 分配給 operators
        * 規則就是優先權，結合律，還有計算順序
        * 不過我們可以用 () 來無視這些規則
### 4.1.3 Order of Evaluation
* 雖然說優先權跟結合律決定了哪個 operand 要被哪個 operator 運算，**但是優先權跟結合律完全沒有指定一個 expression 內，哪個 operator 會先運算**
# 很重要
# 很重要
# 很重要
* 例子：
    * int i = f1() * f2();
    * 我們知道 f1() 跟 f2() 一定會在做 * 之前被算完，但是 f1() 跟 f2() 哪個會先算是沒有被定義的
    * For operators that do not specify evaluation order, **it is an error for an expression to refer to and change thesameobject.**
    * **Expressions that do so have undefined behavior (§ 2.1.2, p. 36).**
    ```cpp
    int i = 0; cout << i /*refer to*/<< " " << ++i/*and change*/ << endl; // undefined
    ```
* 不過雖然大部分的 operator 都沒有指定運算順序，但有四個有指定，分別是 &&(logic AND), ||(logic OR), ? : (ternary),  跟 ,(comma operator)
### 結論是 Order of operand evaluation is independent of precedence and associativity.
```cpp
 f() + g() * h() + j();
```
* 我們只知道 * 一定會在 g() 跟 h() 做完之後才被運算，其他兩個 + 亦然
* **可是這四個 function call 誰會先算沒人知道。**
* 所以你要想如果你不同的 function call 會影響到同一個物件(包括 I/O)，那你這樣寫就是 UB

* ADVICE:MANAGING COMPOUND EXPRESSIONS
    * When you write compound expressions, two rules of thumb can be helpful:
    1. When in doubt, parenthesize expressions to force the grouping that the logic of your program requires.
    2. **If you change the value of an operand, don’t use that operand elsewhere in the same expresion.**
        * 但是第二點有例外：
            ```cpp
            *++iter
            ```
        * 這樣寫的話 iter 會被 ++ 先改變，之後 * 又運作在被改變的 iter 上。這樣符合第二點(eleswhere in the same exp)，但是因為你這樣寫，你一定會先算 ++ 再算 *，所以不會造成什麼問題，這感覺有點類似 f1() * f2() 一定會把 f1() 跟 f2() 都算完再算 *，不過稍微有點不同

## 4.2 Arithmetic Operators
![](https://i.imgur.com/B8meck2.png)
* ALL left associative
* Unless noted otherwise, the arithmetic operators may be applied to any of the arithmetic types (§ 2.1.1, p. 32) or to any type that can be converted to an arithmetic type.
* results are rvalue
* small integral types are **promoted to a larger integral type**, and all operands may be **converted to a common type** as part of evaluating these operators.
* unary + and arith + and - may also be applied to pointers
* 你只要對某個 operand 用了 operator，你都要注意 operand 有可能會被 promoted，之後會講。
```cpp
int i = 1024; int k = -i; // i is -1024
bool b = true;
bool b2 = -b; // b2is true!
```
1. b 是 bool，被 unary - 做運算會先 promote 成 int，阿 true 變 int 值就是 1(int)；
2. 這時 - 再做運算，結果是 -1(int)；
3. 這時要把 int 存回 bool，所以要轉型，-1(int) 轉成 bool 因為非 0，所以是 true；
4. 最後把 true 存到 b2，所以 b2 是 true!

* 其他就加減乘除mod
* 記得可能會 overflow
* 介紹一下該死的 / 跟 %
    * / 如果你的被除數跟除數同 sign，那 result 是正的，否則就是負的
        * 但是負的值到底是多少？
        * 早期 C++版本允許負的商可以往 0 round 或者往負無限大 round，C++11 規定一律往 0 的方向 round(其實就是 truncated)
    *  m % n 的定義是如果 n 非 0，那 (m/n)*n + m%n = m，
        *  換句話說，如果 m%n 非 0，那他的 sign 跟 m 一樣。
            * (100/3)*3 + 100%3 = 100，由 / 的定義知道 + 左邊的值是 99，所以 100%3 是 1，100 跟 1 同 sign
            * (-100)/3*3 + (-100)%3 = -100，由 / 的定義知道 + 左邊的值是 -99，所以 (-100)%3 是 -1，-100 跟 -1 同 sign
    * 還有，除了一些ㄎㄧㄤ例子例如 -m overflow 之類的，(-m)/n 跟 m/(-n) 總是跟 -(m/n) 相同，m%(-n) 跟 m%n 相同，(-m)%n 跟 -(m%n) 相同
        ```cpp
        21 % 6; /* result is 3 */      21 / 6; /* result is 3 */ 
        21 % 7; /* result is 0 */      21 / 7; /* result is 3 */  
        -21 % -8; /* result is -5 */  -21 / -8; /* result is 2 */
        /* (-21)/(-8)*(-8) + (-21)%(-8) = -21*/
        /* (-21)/(-8) = -(21/(-8)) = -(-(21/8)) = 21/8 */
        /*由 / 定義得知 + 左邊的值是 -16，所以 (-21)%(-8) = -5*/
        21 % -5; /* result is 1 */     21 / -5; /* result is -4 */
        /* 21/(-5) = -(21/5) = -4*/
        /* (21)/(-5)*(-5) + 21%(-5) = 21
        /* 由 / 定義得知 + 左邊的值是 20，所以 (21)%(-5) = 1
        ```
### 4.3 Logical and Relational Operators
![](https://i.imgur.com/IdNhCoW.png)

# evaluate: 求值，這翻法滿讚ㄉ
# 原來 ?: 右邊兩個 operands 要是同一類型或者可以被轉成同一類型ㄟ；return lvalue，如果 : 左右都是 lvalue 或者可以被轉成 lvalue，否則 return rvalue


### 4.3 Logical and Relational Operators
* operands: rvalue
* result: rvalue
#### logic AND OR
* short-circuit evaluation
    * 可以用這個特性來做一些安全性檢查的效果
        * index != s.size() && !isspace(s[index])
        * We’re guaranteed that the right operand won’t be evaluated unless index is in range.
    * Primer 142 頁還有個例子，自己看看

#### Logical NOT Operator
* pass
#### The Relational Operators
* 請記得他們都是 binary operator
* 不要寫出什麼 i < j < k 之類的鬼東西
#### Equality Tests and the bool Literals
* 如果要看一個 object 的 truth value，最好直接解
    * if(obj)
* 不要寫
    * if(obj == true)
        * 這種寫法，只要 obj 不是 bool，bool literal(也就是這邊的 true) 就會被 promote
    -> if(obj == 1)
    * 只要 obj 不是 1 就會 false!
    * 如果你真的要看一個 obj 的 value 是不是 1，請直接寫 == 1 不要寫 == true
* 結論，bool literal 用在 comparison 時除非比較對象也是 bool，不然建議不要用
## 4.4 Assignment Operators
* 注意，宣告時用的 = 是 initialization，不是 assignment
* lhs must be **modifiable lvalue**
    ```cpp
    int i=0, j=0, k= 0; // initializations, not assignment
    const int ci = i; // initialization, not assignment
    1024 = k; // error: literals are rvalues
    i + j = k; // error: arithmetic expressions are rvalues
    ci = k; // error: ciis a const(nonmodifiable) lvalue
    ```
* result of assignment is lhs, which is an lvale
* type is type of lhs
    * if type of rhs and lhs differ, rhs is converted to type of lfs
    ```cpp
    k = 0; // result: type int,value 0
    k = 3.14159; // result: type int,value 3
    ```
    * 如果用新標準的 initializer list，如果 rhs 的型別要被轉到 lhs 的型別會 narrowing down 的話，就會噴 error。
```cpp
k = {3.14};  // error: "narrowing conversion"
vector<int> vi; // initially empty
vi = {0,1,2,3,4,5,6,7,8,9}; // vi now has ten elements, values 0 through 9
```
#### Assignment Is Right Associative
    學過ㄌ
#### Assignment Has Low Precedence
* assignment 的優先權很低，常常需要括號，尤其你用很 C/C++ 的把 assignment 放在 condition 的寫法的時候。
#### Compound Assignment Operators
```cpp
+= -= *=/=%= <<= >>= &= ^= |=
// arithmetic operators
// bitwise operators; see § 4.8 (p. 152)
```
* 注意他跟分開寫的反本的區別(e.g. i = i + 1 和 i += 1)
    * i = i + 1, i 會被求值(evaluate)兩次!
    * i += 1, i 只會被求值一次
* 除了效能上的差異以外"幾乎"沒有差別
    * 可是在組語上左邊的 i 不是都不用算嗎...
## 4.5 Increment and Decrement Operators
* There are two forms of these operators: prefix and postfix.
* require lvalue operand
* prefix version's result is lvalue
* **postfix version's result is rvalue**
#### Combining Dereference and Increment in a Single Expression
* 這是常用技巧!
    ```cpp
    *ptr++;
    ```
    * postfix increment 優先權比 dereference 高，所以 ++ 會先作用在 ptr 上
    * *(ptr++)
    * 這樣 dereference 還是作用在舊的 value，可是 ptr 還是會被 increment
    * This usage relies on the fact that postfix increment returns a copy of its original, unincremented operand. 
#### **Remember That Operands Can Be Evaluated in Any Order**
* Because the increment and decrement operators change their operands, it is easy to misuse these operators in compound expressions.
* increment operator 就是屬於那種會改變 operand value 的 operator，所以在 exp 內用它們的時候要小心不要有其他的 subexp 也用到它們作用到的 operand。
    ```cpp
    string s = "9487_I_LOVE_hsilu"
    for (auto s_it = s.begin(); s_it != s.end(); s_it++)
        *s_it = toupper(*s_it);
    ```
    * 這裡把 s_it++ 跟 loop body 內的 dereference 拆開，因為你 body 內的 exp 用了兩次 s_it，你只要對其中一個寫了 s_it++，那這個 exp 就會是 UB
        * *s_it = toupper(*s_it++);
## 4.6 The Member Access Operators
* dot (.) and arrow(->)
* (*p).size();
    . 的優先權比 * 高，所以要括號
* -> 需要一個 pointer 當 operand，然後 result 是 lvalue
    * p->size;
        * p->size(); 是先把某個 object 的 size 拿出來, which is lvalue，再把 () 用在這個 lvalue 上
* . 的 result 是 l 還 r depends on 你拿到的 member 是 l 還是 r。

## 4.7 The Conditional Operator
* cond ? expr1 : expr2;
* lets us embed simple if-else logic inside an expression
* 請不要 nest 這噁心的東西
* 注意， expr1 跟 expr2 要是同 type(或者可以被 convert 成同 type)
    * 你可以試試不同而且不能轉的 type，會噴 error
    * 每個 exp 本來就要有一個 value，而沒有 type 就沒有 value，**然後你的 exp 如果可能有兩種 type，你怎麼決定 value?**
* 如果 cond 是 true，expr1 會被求值，反之 expr2 會被求值
    * 不管怎樣只有其中一個會被求值
* **That result of the conditional operator is an lvalue if both expressions are lvalues or if they convert to a common lvalue type. Otherwise the result is an rvalue.**
* 但基本上不會把 ?: 放在 lvalue 可以放的地方吧... 你寫這種 code 會讓讀的人很想死
* ? : 的優先權非常低，比 << 跟 >> 還低，請無腦的直接在 ? : 兩邊用括號
    ```cpp

    int main() {
      int grade = 87;
      cout << ((grade < 60) ? "fail" : "pass"); // prints passor fail
      cout << (grade < 60) ? "fail" : "pass";   // prints 1or 0!
      cout << grade < 60 ? "fail" : "pass";     // error: compares coutto 60
    }
    ```
    * 請看 Primer P.152 的解釋
## 4.8 The Bitwise Operators
![](https://i.imgur.com/MqGFg5l.png)
* 用這些 operator 時，如果 operand 是 small integral，會先被 promote 成大的 integral
* **不要對 signed integral 做 >>，是 IB(implementation defined)!** 
* shift 的位數不可以超過那個 type 的 bit 數，否則也是 UB
* << inserts bit ""0"" on the right
* >> 如果 lhs 是 unsigned，那就 insert bit "0 on the left
    * 如果是 signed，那會是 implementation defined
* ~ 把 operand 的 bit 反轉
    * small integral 會先 promote
* bitwise AND OR XOR
    * 如果是 small integral，那會先做完運算之後在 promote，而且都填 0
#### Shift Operators (aka IO Operators) Are Left Associative
* 他其實跟 overloaded 版本(也就是 cin cout 用的 >> <<)有著一樣的結合律跟優先權
    * 換句話說，該括號的就要括號，而且他優先權也不高
    * The shift operators have midlevel precedence: lower than the arithmetic operators but **higher than the relational, assignment, and conditional operators.**

## 4.9 The sizeof Operator
* right associative
    * 拜託你用括號吧...
* **result is const expression**
    * 可以拿來諸如宣告陣列之類的
* The sizeof operator is unusual in that **it does not evaluate its operand**:
    * 他不會在 runtime 時真的算，畢竟他可以回傳 constexpr
    ```cpp
    int main() {
      int a = 94, b = 87;
      int c = sizeof(a + b);

      Sales_data data, *p;
      sizeof(Sales_data); // size required to hold an object of type Sales_data
      sizeof data;        // size ofdata’s type, i.e., sizeof(Sales_data)
      sizeof p;           // size of a pointer
      sizeof *p; // size ofthe type to which p points, i.e., sizeof(Sales_data)
      sizeof data.revenue; // size of the type of Sales_data’s revenue member
      sizeof Sales_data::revenue; // alternative way to get the size ofrevenue, C++11
    }
    ```
    * 注意，sizeof(pointer)，不管 pointer 是否為 valid 都可以，因為 runtime 本來就不會跑這句；同理，sizeof(*pointer) 也不用管 pointer 是否為 valid
* C++11
    * Under the new standard, **we can use the scope operator to ask for the size of a member** of a class type.
    * We don’t need to supply an object, because **sizeof does not need to fetch the member to know its size.**
* 還有幾點要注意
    1. sizeof(char) == 1
    2. sizeof(&ref) == ref 的對象的大小
    3. sizeof(pointer) == pointer 所需要的大小
    4. sizeof(*pointer) == pointer 指向的物件的大小，pointer 不需要合法
    5. sizeof(array) == array_len * sizeof(element)
        * 這時候用 sizeof array 並不會被 decay 成 pointer
    6. sizeof(string) 或 sizeof(vec_int) 並不會包含到字串或者 element 的內容，畢竟那些東西在 heap，而且要在 runtime 才會知道。
## 4.10 Comma Operator
* 少數有規定運算順序的 operator，先算左再算右
* result 是右邊的 value
* r 或 l 取決於右邊是 r 或 l

## 4.11 Type Conversions
* Some types are related to each other
    * There is a conversion between them
    ```cpp
    int ival = 3.541 + 3; // the compiler might warn about loss ofprecision
    ```
    1. 3 先被轉成 double
    2. 做這個 3.541(double) + 3.0(double) 兩 double 相加的運算
    3. 把結果的 double 再轉回 int 給 ival
* 在 C++，對於 built-in type 的運算都會要求 operator 兩邊 operands 的 type 要一樣，否則會先做轉換再做運算。
    * 怎麼轉換後面會敘述一套規則
* Rather than attempt to add values of the two different types, C++ defines a set of conversions to transform the operands to a common type.
    * **implicit conversions**
* The implicit conversions among the arithmetic types are **defined to preserve precision, if possible.** 

* When Implicit Conversions Occur
    * The compiler automatically converts operands in the following circumstances:
    * In most expressions, values of integral types smaller than int are first promoted to an appropriate larger integral type.
    * In conditions, nonbool expressions are converted to bool.
    * In initializations, the initializer is converted to the type of the variable; in assignments, the right-hand operand is converted to the type of the left-hand.
    * In arithmetic and relational expressions with operands of mixed types, the types are converted to a common type.
    * As we’ll see in Chapter 6, conversions also happen during function calls.
### 4.11.1 The Arithmetic Conversions
* The rules define a hierarchy of type conversions in whichoperands toanoperator are converted to the widest type.
#### Integral Promotions
我覺得這個部分真的就是要用到的時候再來查書了..
* convert the small integral types to a larger integral type.
#### Operands of Unsigned Type
* 如果 operand 有一個是 unsigned，那到底會轉成什麼還要看 operands 各自在機器上的大小...
## 然後你就會很想死，best practice:
* 不要把 signed 跟 unsigned 混用在一個 exp
#### Understanding the Arithmetic Conversions
* 剩下的看書ㄅ

### 4.11.2 Other Implicit Conversions
* 這還比較有看頭...
* 上一張的很機掰，可是不能錯
* 這張的比較有概念在裡面
* **Array to Pointer Conversions**: In most expressions, when we use an array, the **array is automatically converted to a pointer to the first element in that array:**
* 什麼時候 array name 當 operand 不會被轉成 pointer?
    1. decltype(arr)
    2. &arr (address-of)
    3. sizeof(arr)
    4. typeid(arr) // 19.2章，超後面的主題
    5. array_type &arr_ref = arr;
* array 在 function call 時也常被轉換成 pointer

* 指標間的轉換
    * 0 跟 nullptr 可以被轉成任何型態的指標
    * pointer to non-const type 可以轉成 void*
    * pointer to any type 可以轉成 const void*
    * 還有噁心的跟繼承(inheritance)有關的指標型態轉換...

* whatever 轉成 bool
    * 講過ㄌ...
    * 如果是自定義的 class 要自己實作轉換方式
### 4.11.3 Explicit Conversions
* cast-name<type>(expression); // </type>
* If type is a reference, then the result is an lvalue.
* The cast-name may be one of **static_cast, dynamic_cast, const_cast,and reinterpret_cast.**

* dynamic_cast 19章才會講
    * 哇靠當年 YYP為什麼會講..
#### static_cast
* Any well-defined type conversion, other than those involving low-level const, can be requested using a static_cast. 
    * 如果你想要的型態可以用既有型態轉過來可是沒有辦法用 implicit conversions，那你就要用 static_cast
    ```cpp
    // cast used to force floating-point division 
    double slope = static_cast<double>(j) / i;
    ```
    * 如果你沒加 cast，原本的 / 會用 int 的方式除，這樣小數會被丟掉。加了之後就會用浮點數的方法除了。

* A static_cast is often useful when a larger arithmetic type is assigned to a smaller type.
* informs both the reader of the program and the compiler
* A static_cast is also useful to perform a conversion that the compiler will not generate automatically.
    ```cpp
    void* p = &d; // ok: address ofany nonconstobject can be stored in a void* // ok: converts void*back to the original pointer type
    double *dp = static_cast<double*>(p);
    ```
    * void* 的確可以轉為任何的 pointer to non-const type
    * 可是這裡不會自動轉，所以要上 static_cast
    * 如果你不加就會噴 error
    * 傳統的 C 就是用 (double *)，其實 C++ 就是把這種噁心的 power 轉成四種類型而已。
    * When we store a pointer in a void* and then use a static_cast to cast the pointer back to its original type, we are guaranteed that the pointer value is preserved.
        * 把一個 ptr 丟給 void* 然後在 cast 回來會保證 value 一樣
        * 如果你 cast 回不同的 type，是 UB。
#### const_cast
* change low-level const
    ```cpp
    const char *pc;
    char *p = const_cast<char*>(pc); // ok: but writing through p is undefined
    ```
    * 如果要把 top-level 的 const 也幹掉，要用 reinterpret_cast, 夭壽
    * "casts away the const."
*  If the object was originally not a const, using a cast to obtain write access is legal. **However, using a const_cast in order to write to a const object is undefined.**
* const_cast 只能把 pointer 或 reference 的 low-level const 拿掉而已，用他來做其他的轉換會噴 error
    * **同樣的你也不能用其他種 cast 來轉換 expression 的 `const`ness
    * **`const char* p; static_cast<char*>(p)` 會噴 error**
* A `const_cast` is most useful in the context of overloaded functions, which we’ll describe in § 6.4 (p. 232).

#### reinterpret_cast
* 噁
* A reinterpret_cast generally **performs a low-level reinterpretation of the bit pattern** of its operands.
* 跟 const_cast 一樣，只能轉 pointer 或 reference

##### 題外話，class 要實作轉 type 要寫 operator type() 這個 function
##### Every time you write a cast, you should think hard about whether you can achieve the same result in a different way.

#### Old-Style Casts
* type(expr); // function-style cast notation
* (type)expr; // C-language-style cast notation
* 如果在某個用了 old-style casts 的 context 下，其實用 static_cast 或 const_cast 是合法的，那 old-style casts 就等價於用它們
* 如果都不合法，那就等於用 reinterpret_cast


#### static_cast: An explicit request for a well defined type conversion.
其實傳統 C 的 cast 也只是去使用這些 well defined conversion 而已，如果不存在這種 conversion，例如
int i = 0;
string s = (string)i;
還是會噴 error 的
不過 pointer to blabla，就真的可以亂轉了，例如
int *ip = &i;
double *dp = (double*)&i;
是可以的，不過這種用 C++ 版本的 cast 就只能用 reinterpret_cast 了，不能用 static_cast，因為根本不存在 well-defined 的轉法。
但是你現在對 dp 做 dereference 就會得到一塊實際上內容是 int 的物件惹
