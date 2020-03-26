---
tags: C++
---

# C++ Primer Chapter 14 Oerloaded Operations and Conversions

* Operator overloading lets us define the meaning of an operator when applied to operand(s) of a class type.
* **Judicious use** of operator overloading can make our programs easier to write and easier to read.
    ```cpp
    cout << item1 + item2; // print the sum of two Sales_items
    print(cout, add(data1, data2)); // print the sum of two Sales_datas
    ```
    * 因為 `Sales_item` 有定義 `operator+/=/<<` 的行為所以可以這樣寫
    * 反之 `Sales_data` 就沒有，code 看起來就不明確

## 14.1 Basic Concepts
* Overloaded operators are **functions with special names**:
    * the keyword `operator` followed by the symbol for the operator being defined
    * 既然是 function，一樣有 return type, para list, 跟 function body

* 我們雖然可以定義 operator 的意義，不過 operand 的數量是不變的(除了 call operator(`operator()`))
    * unary, binary
    * binary 第一個參數就是 lhs，第二個是 rhs

* **除了 `operator()`，其他 operator 都不能有 default args**
    * 合理，否則會寫出像是 syntax error 的東西
* 另外如果一個 operator 定義成 class 的 member function，那**他的第一個 operand 就會綁到 `this`**
    * 所以參數數量會比這個 operator 所需要的 operand 數量少一個
    * 或者說比 nonmember 版本 function 的少一個

* 一個 operator function 一定要馬是 memeber function，要馬是 global function
    * 但是 para list 內至少有一個 parameter 是 class type:
    ```cpp
    // error: cannot redefine the built-in operator for ints 
    int operator+(int, int);
    ```
    * 這也意味著我們不能改變操作在 buil-in type 上的 operator 的意義(因為你沒辦法定義一個 operator function，其中所有參數都是 built-in type)

* table 14.1 列出可以跟不可以 overload 的 operator
    * 19.1 會講怎麼 overload `new` 跟 `delete`
* 我們也不能創造新的 operator symbol
    * 比方說定義 `operator**` 來實作 exponent 之類的

* 而有些 symbol(`operator+/-/*/&`) 同時是 unary/binary operator
    * 兩種都可以 overload
    * 定義他們時的參數數量決定你是定義 unary/binary operator
* 還有一點，**operator 就算被 overloading，彼此之間的運算優先級(precedence)以及結合律(associativity)也不會改變**(第四章)
    * `x== y+z;` 永遠等價於 `x== (y + z);`

![](https://i.imgur.com/hsFlfBj.png)
* `.*` 是 [member access through pointer to member](https://en.cppreference.com/w/cpp/language/operators#Restrictions)

#### Calling an Overloaded Operator Function Directly
* 你要直接 call operator function 還是寫成一般 expression 的形式都可以，這兩者是等價的
    ```cpp
    // equivalent calls to a nonmember operator function
    data1 + data2; // normal expression
    operator+(data1, data2); // equivalent function call
    ```
    * 兩個都會使用 nonmember function `operator+()`
* 而如果 operator function 是定義成 member function，這樣寫才等價:
    ```cpp
    data1 += data2; // expression-based "call"
    data1.operator+=(data2); // equivalent call to a member operator function
    ```
    * 其中如果 `data1` 是指標， member access operator 就要改成 `->`

#### **Some Operators Shouldn't Be Overloaded**
* 之前在第四章學過，有少部分的 operator 保證運算順序或者 short circuit evaluation
    * `&&`, `||`, `,`
* **但如果對 class type 定義這幾種 operator，則上述的保證全部不成立!**
    * 因為 operator function 本質上是 function call
    * 先前學過，function parameter 參數運算順序是未定義
    * 這意味著 operator function 的參數運算也是未定義
* 而 `&&` 跟 `||`，他們 overloading 也不會有 short circuit!!
    * **兩個 operands 都會運算**

* 總之這些 operator 最好不要 overload!
* 如果你的 class 放在 `&&` 或 `||` 運算看起來很自然的話，請定義 class to `bool` 的轉型，之後會講到

* 另外還有有些 operator 就算是使用到 class type 上也會有 built-in meaning
    ```cpp
    &obj; // pointer to obj's type
    a, b; // b
    ```
* 如果再重定義，會有可能反而會造成誤解
    * comma operator 我想不到什麼反例
    * 但 `operator&` 則可能可以用在 smart pointer
        * `&smart_ptr_obj` 應該要回傳 `pointer to underlying type`
        * 不過實際上標準沒有這麼做就是

:::info
* Best Practice: Ordinarily, the comma, address-of, logical AND, and logical OR operators should not be overloaded.
:::

#### **Use Definitions That Are Consistent with the Built-in Meaning**
* 設計 class 時，你應該要先想他提供什麼 operations
    * 也就是 **do what**(public API)
    * **how to do**(private utility)
* 在這些 operations 之中，如果有某些 operation 跟某些 operator 有蓋線上的對應關係(logical mapping)(比方說概念上都是加法)，則可以考慮使用 overloading(比方說用 operator+ 來實作加法)
    * 舉 `std::string` 的 `operator+`，對 programmer 來說他的實際行為(串接字串)跟符號(`+`)可以對得起來

* 更 general 的規則如下
    * 如果 class **支援 IO**，你應該要**定義 shift operators(`>>`, `<<`)**
    * 如果 class 會測試兩個物件是否相等，你應該要定義 `operator==`；
        * 如果定義 `operator==` 通常也要定義 `operator!=`
    * 如果 class 物件存在一個順序(natural ordering)的概念，你應該要定義 `operator<`
        * 如果定義 `operator<` 也要把其他 relational operators 都定義
    * operator function 的 return type 要跟使用在 built-in type 的情況一致
        * 算數運算子應該要回傳 class object 的 value(nonreference)
        * 邏輯運算子要回傳 bool
        * assignment 跟 compound assignment 要回傳 reference to lhs

#### Assignment and Compound Assignment Operators
* `operator=` 13章講過ㄌ，不過概念上必須讓 lhs 跟 rhs 的 value 一致
* 如果 class 有定義算術運算子跟 bitwise 運算子，最好也定義對應的 compound assignment
    * 而且 `operator?=` 應該就要先做 `lsh ? rhs` 再把這個結果 assign 給 lhs
* 下面看一下，簡短的說明該怎麼定義 operator overloading，以及什麼時候不該 overload
* ![](https://i.imgur.com/n9CcgjY.png)


#### Choosing Member or Nonmember Implementation
* 必須想我們要把 operator function 定義成 member 還是 global
* 有時候是都可以，有時候只能選一種
* 下面的 guideline 可以幫忙選:
    * The assignment (`=`), subscript (`[]`), call (`()`), and member access arrow (`->`) operators **must** be defined as members.
    * The compound-assignment operators ordinarily *ought* to be members. However, unlike assignment, they are not required to be members.
    * Operators that **change the state of their object or that are closely tied to their given type**—such as increment, decrement, and dereference—usually should be members.
    * *Symmetric* operators—those that might convert either operand, such as the arithmetic, equality, relational, and bitwise operators—usually should be **defined as ordinary nonmember functions.**
        ```cpp
        class_name operator+(clas_name, class_name);
        class_name operator+(clas_name, convertible_type);
        class_name operator+(convertible_type, class_name);
        ```
        
* 如果你把 operator function 定義成 member，之前說過，第一個 operand 會綁到 this；
    ```cpp
    string s = "world";
    string t = s + "!"; // ok: we can add a const char* to a string
    string u = "hi" + s; // would be an error if operator+ were a member of string
    ```
    * 如果 `operator+` 是定義成 member，那 `s + "!"` 就等於 `s.operator+("!");`
    * 而 `"hi" + s;` 就等於 `"hi".operator+(s);` 噴 error
* 因為 `std::string` 是把 `operator+()` 定義成 global，所以 `s + "!";` 等於 `operator+(s, "!");`，`"hi" + s;` 等於 `operator+("hi", s);`
        * `const char*` 都會被轉換成 `std::string` 再用 `operator(string, string);` 做運算
    * 總之 symmetric operaotr 定義成 global 的話，只要其中一個 operand 是 class type，function matching 即可找到

## 14.2 Input and Output Operators
### 14.2.1 Overloading the Output Operator <<
* first parameter of an output operator is a reference to a non`const` `std::ostream` object.
    * non`const` 因為 write to ostream 會改變 ostream 的 state
    * parameter 是 reference 是因為不能 copy
    * 這兩個之前都講過了
* The second parameter ordinarily should be a reference to const of the class type wewant to print.
    * 通常 output object "邏輯上"不會改變 object，所以用 const reference
        * reference 避免 copy
    * 如果 object 內容有實作上必須會改變的 member，請用 `mutable`
* return type 要跟標準一致，回傳 `std::ostream&`

#### The `Sales_data` Output Operator
* 拿 `Sales_data` 來舉例
    ```cpp
    ostream &operator<<(ostream &os, const Sales_data &item) {
        os << item.isbn() << " " << item.units_sold 
           << " " << item.revenue << " " << item.avg_price();
        return os;
    }
    ```
    * 其實做的事情就跟之前寫的 `print` 一樣

#### Output Operators Usually *Do Minimal Formatting*
* 這是一個 practice
    * 例如上面的 `operator<<()` 沒有印出 newline
* An output operator that does minimal formatting **lets users control the details of their output.**
    * 讓 class 的 user code 來做 formatting
    * 該 user code 可能就是 "formatter"
    * 而這個 formatter 可能又被某個 user code 使用

#### IO Operators Must Be Nonmember Functions
* 因為 lhs 一定要是 IO type，所以不可能是 member function
    * 這樣 `this` 會綁到 IO stream object
* 因為 `operator>>/<<` 要是 nonmember，然後 IO 通常又會讀 class 的 non`public` member
    * 所以要在 class 內宣告成 friend

### 14.2.2 Overloading the Input Operator >>
* 一樣 nonmember
* 第一個參數是 `std::istream&`
* 第二個參數是 non`const` reference
* 一樣回傳 `std::istream&`
    ```cpp
    istream &operator>>(istream &is, Sales_data &item) {
        double price; // no need to initialize; we’ll read into price before we use it
        is >> item.bookNo >> item.units_sold >> price;
        if (is) // check that the inputs succeeded
            item.revenue = item.units_sold * price;
        else
            item = Sales_data(); // input failed: give the object the default state
        return is;
    }
    ```
    * 跟 `operator<<()` 不同
    * 這裡還對 IO 做一些 error handling，自己看

:::info
* Note: Input operators **must deal with the possibility that the input might fail**; output operators generally don’t bother.
:::

#### Errors during Input
* 總之 input 就有可能 error
* 這裡有個小技巧，與其每個 member 都檢查有沒有讀取成功，不如一次讀近來然後檢查 istream，有 error 就把 object 設成某個狀態

* **Putting the object into a valid state is especially important if the object might have been partially changed before the error occurred.**

:::info
* Best Practice: Input operators should decide what, if anything, to do about error recovery.
* 反正一個 class 的 API，就是一個狀態圖的概念
* 理論上要讓 user code 不管怎麼操作 class 的 API(即使操作錯)， class object 還是要處於狀態圖的其中一個 state
* IO error 可以直接把 object reset(成 initial state)
:::

#### Indicating Errors
* 除了 error handling，你還要有一種方式來告知 user code 有 error
    * 第 8 章的 error bit
* 就算有成功讀取資料到 data member(type 正確)，更進階的狀況可能是 value 不符預期(超出 spec 範圍，比方說 Sales_data 賣 0 元之類的)
* 這時候可以利用 iostream 的那些 flag 來告知 user code
    * 但通常業務邏輯 fail 只會設定 `failbit`
    * 另外的 `eof` 跟 `badbit` 最好讓 `iostream` object 自己設定比較好
        * These errors are best left to the IO library itself to indicate.

## 14.3 Arithmetic and Relational Operators
* 再說一次，nonmember
* 一般來說給定的參數不會被改變，所以用 const reference
* 通常這些 operator 都是藉由 operands 計算出一個 **local** value，然後回傳 value
* 另外如果定義了某個 relational operator(s)，也會定義對應的 compound assignment
    * **並且 relational operator 通常會利用對應的 compound assignment 來實作**:
    ```cpp
    // assumes that both objects refer to the same book 
    Sales_data
    operator+(const Sales_data &lhs, const Sales_data &rhs) {
        Sales_data sum = lhs; // copy data members from lhs into sum 
        sum += rhs; // add rhs into sum
        return sum;
    }
    ```
    * 上面 `operator+()` 用 `operator+=()` 實作
    * 某整程度上有 code reuse
    * 而且也有效能考量
    * **如果用 `operator+()` 來實作 `operator+=()`，會多出一次 copy**

### 14.3.1 Equality Operators
* 如果 class object 概念上可比較是否相等，就可以定義 `operator==()`
    * 並且通常會一併定義 `operator!=()`
    ```cpp
    bool operator==(const Sales_data &lhs, const Sales_data &rhs) {
        return lhs.isbn() == rhs.isbn() 
        && lhs.units_sold == rhs.units_sold 
        && lhs.revenue == rhs.revenue;
    }
    bool operator!=(const Sales_data &lhs, const Sales_data &rhs) {
        return !(lhs == rhs);
    }
    ```
   * 注意又是一個用已定義的 operator(`operator==()`) 來實作另一個 operator(`operator!=()`) 的例子
 
:::info
* Design Principle:
    * 如果 class 有等價的概念，請定義 `operator==()`
        * 不要使用 named function 來當 interface，這樣 user code 還要額外記要怎麼用
        * 而且很多 STL API 都會用 `operator==()`，有定義的話，很多 algo 都可以用，不用額外再傳 callable object
    * 另外，定義出來的 `operator==()` 要 transitive:
        * `a == b` 且 `b == c`, 則 `a == c`
    * 定義了 `operator==()` 也請定義 `operator!=`
    * 還有上面的其中一個應該要由另一個來實作
:::

### 14.3.2 Relational Operators
* 想要定義 `operator<()` 的話必須滿足兩個要求
    1. strict weak ordering(詳情看 11.2.2, p.424)
    2. **如果已經為 class 定義了 `operator==`，那麼你由 strict weak ordering 組出來的 `operator==()`，亦即 `!(a<b) && !(b<a)`，要跟你已經定義的 `operator==` 的行為一致**

* 從前面 `Sales_data` 定義的 `operator==` 的例子可以發現，`Sales_data` 不該定義 `operator<()`
    * 首先他已經定義了 `operator==`()，*回傳結果是比較 `Sales_data` 三個 member 的值*
    * 這裡只能這樣定義 ==，因為如果有任何一個 member 不一樣，邏輯上(或說現實上，或者說業物邏輯上)這兩筆 data 都不會當成相同
* 那如果假設我們的 `operator<()` 是只有比較 `isbn` 的話，那會發生無法滿足第二個要求的情況
    * `isbn` 一樣，可是另外兩個 member 不一樣，這時候 `!(a<b) && !(b<a)` 為真，可是自定義的 `operaotr==()` 卻為假

* 你可能會覺得，不然把另外兩個 member 也拿來比較
    * 先比 `isbn` 再比 `unit_sold` 再 `revenue` 之類的，可以滿足第二點(因為三個 member 都比到，如果三個 member 真的都一樣，`operator<` 組出來的 ==(`!(a<b) && !(b<a)`) 跟 `operator==()` 的行為就會一致
    * 可是這樣子的比較本質上實際意義
        * 或者說會 ambiguous
    * 因為先比 `isbn` 再比 `revenue` 再 `unit_sold`(順序跟上面有差)也是一種比較方式阿?
        * 這樣 user code 要認為是哪一種?
        * **之前說過一個 operator 如果會有 ambiguous meaning，就不該 overloading**
* 結論就是 `Sales_data` 這個例子不適合定義 `operator<()`

## 14.4 Assignment Operators
* 除了 13章講的 `operator=`，還可以定義不同 type 的 rhs，沒有一定要 class type
* 例如 `std::vector` 定義吃 `std::initializer_list` 的:
    ```cpp
    vector<string> v;
    v = {"a", "an", "the"};
    ```
* 或額外為 `StrVec` 定義:
    ```cpp
    class StrVec {
    public:
        StrVec &operator=(std::initializer_list<std::string>);
        // other members as in § 13.5 (p. 526)
    };
    ```
* 記得都要 return rhs&
    ```cpp
    StrVec &StrVec::operator=(initializer_list<string> il) {
        // alloc_n_copy allocates space and copies elements from the given range 
        auto data = alloc_n_copy(il.begin(), il.end());
        free(); // destroy the elements in this object and free the space 
        elements = data.first; // update data members to point to the new space 
        first_free = cap = data.second;
        return *this;
    }
    ```
    * 有沒有發現少了 self-assignment check? 因為不可能同物件R
* 還有之前說過，`operator=()` 只能定義成 member
* 還有一點，就是你可以指定義 copy/move assignment，然後定義各種 吃特定行別的 ctor
    * 這樣 compiler 會自動把 rhs 轉成 class object
    * 跟直接定義特定的 `operator=(T)` 的差異就是多了一個 temp object

#### Compound-Assignment Operators
* 他跟 `operator=` 不一樣，不用一定是 member function
    * 但他本質上會 "change object state"
    * 不要自找麻煩，請定義成 member function
* 一樣 return lhs&
    ```cpp
    // member binary operator: left-hand operand is bound to the implicit this pointer 
    Sales_data& Sales_data::operator+=(const Sales_data &rhs) {
        units_sold += rhs.units_sold;
        revenue += rhs.revenue;
        return *this;
    }
    ```
    * assumes that both objects refer to the same book

## 14.5 Subscript Operator \[\]
* 如果你的 class 用起來像 STL container，通常會定義這個 operator
* 一定要是 member function
* return reference
    * 可以當 lvalue
* 最好做 `const` overload，這樣 `const` 物件就會呼叫 `const` 版本

* 用 `StrVec` 舉例:
    ```cpp
    class StrVec {
    public:
        std::string& operator[](std::size_t n) { return elements[n]; }
        const std::string& operator[](std::size_t n) const  { return elements[n]; }
        // other members as in § 13.5 (p. 526)
    private:
        std::string *elements; // pointer to the first element in the array
    };
    ```
    * 注意有 `const` overload
    * 另外 `operator[]` 的 `rhs` 沒有一定要是 integral type
    * 例如 `std::map` 就是 `std::map<K, V>::key_type`

* 可以這樣用:
    ```cpp
    // assume svec is a StrVec
    const StrVec cvec = svec; // copy elements from svec into cvec
    // if svec has any elements, run the string empty function on the first one 
    if (svec.size() && svec[0].empty()) {
        svec[0] = "zero"; // ok: subscript returns a reference to a string 
        cvec[0] = "Zip"; // error: subscripting cvec returns a reference to const
    }
    ```
    * 對 `const StrVec` 的 element assign 會噴 error

## 14.6 Increment and Decrement Operators
* 通常用起來行為像是 iterator 的 class 會實作
* 一樣不用是 member，但因為他會改變 object state，請定義成 member
* 有 prefix 跟 postfix 兩種
    * 那我們要怎麼分別定義他們呢?
    * 後面會講，*很噁心*
* 會先講怎麼定義 prefix 再講 postfix

#### Defining Prefix Increment/Decrement Operators
* 既然 iterator class 通常會定義，就來用 `StrBlobPtr` 當例子:
    ```cpp
    class StrBlobPtr {
    public:
        // increment and decrement
        StrBlobPtr& operator++();
        StrBlobPtr& operator--(); 
        // other members as before
    };
    ```
    * 注意這是 prefix 版本，return reference
* 實作:
    ```cpp
    // prefix: return a reference to the incremented/decremented object 
    StrBlobPtr& StrBlobPtr::operator++() {
        // if curr already points past the end of the container, can’t increment it
        check(curr, "increment past end of StrBlobPtr");
        ++curr; // advance the current state
        return *this;
    }
    StrBlobPtr& StrBlobPtr::operator--() {
        // if curr is zero, decrementing it will yield an invalid subscript
        --curr;
        // move the current state back one element
        check(curr, "decrement past begin of StrBlobPtr");
        return *this;
    }
    ```
    * 會用之前定義的 `check` 來看 boundary 是否超過
    * 注意 `--` 的版本，要先 `--curr` 再 `check`，如果原本 `curr` 是 0，`--` 之後會變最大正數(因為他是 `unsigned`)
        * 帶進 check 才會炸裂；

#### *Differentiating Prefix and Postfix Operators*
* Normal overloading cannot distinguish between these operators.
* 用很ㄎㄧㄤ的解法: 讓 postfix 版本多吃一個參數:
    ```cpp
    class StrBlobPtr { 
    public:
        // increment and decrement 
        StrBlobPtr operator++(int); // postfix operators   
        StrBlobPtr operator--(int); 
        // other members as before
    };
    ```
    * When we use a postfix operator, the compiler supplies 0 as the argument for this parameter.
        * *Its sole purpose is to distinguish a postfix function from the prefix version.*
    * 最重要的，return 的是 value

* 實作:
    ```cpp
    // postfix: increment/decrement the object but return the unchanged value
    StrBlobPtr StrBlobPtr::operator++(int) {
        // no check needed here; the call to prefix increment will do the check
        StrBlobPtr ret = *this; // save the current value 
        ++*this; // advance one element; ***prefix ++ checks the increment***
        return ret; // return the saved state
    }
    StrBlobPtr StrBlobPtr::operator--(int) {
        // no check needed here; the call to prefix decrement will do the check 
        StrBlobPtr ret = *this; // save the current value 
        --*this; // move backward one element; prefix -- checks the decrement 
        return ret; // return the saved state
    }
    ```
    * 又是一個用 operator 實作 operator 的例子:
        * 用 prefix 實作 postfix!
    * 而且會發現，postfix 會 create 一份 copy
#### Calling the Postfix Operators Explicitly
* 你用寫 expression 的方式呼叫 postfix 版本，可以不用提供那個 dummy `int`
    * 可是你用 function call 來呼叫就一定要提供那個 `int`
* 如果你沒提供的話也不會噴 error，可是會 call 到 prefix 版本的:
    ```cpp
    StrBlobPtr p(a1); // ppoints to the vector inside a1
    p.operator++(0); // call postfix operator++
    p.operator++(); // call prefix operator++
    ```
    
## 14.7 Member Access Operators
* **高能注意，尤其是 `operator->()`**
* dereference `operator*()` and arrow `operator->()`
* 常表示在 iterator 或 smart pointer class
    ```cpp
    class StrBlobPtr { 
    public:
        std::string& operator*() const { 
            auto p = check(curr, "dereference past end");
            return (*p)[curr]; // (*p)is the vector to which this object points
        }
        // ***注意下面的 return type 是 pointer***
        std::string* operator->() const {
            // delegate the real work to the dereference operator 
            return & this->operator*();
        } 
        // other members as before
    };
    ```
    * 注意兩個都宣告成 `const` function，因為不會更改 `StrBlobPtr` 的 state
        * 本質上 `StrBlobPtr` 若為 `const`，代表他不能再指向其他東西(top level `const`)，不代表不能透過他更改他指向的東西(low level `const`)
    * 這樣 `const` 跟非 `const`，`StrBlobPtr` 都可以呼叫
* `operator*()` 雖然可以做任何事，你可以回傳 42 或任何東西，但是最好還是回傳某個 referece
* **可是 `opetator->()` 就不一樣了，詳細說明如下**
* 當你寫一個 exp `A->B`，這時會等價於兩種情況
    * 如果 `A` 是指標，那 `A->B` 就等於 `(*A).B`
    * 可是如果 `A` 是一個有實作 `operator->()` 的 class 物件，則 `A->B` 就等於 `A.operaotr->()->B`，一個遞迴概念...
        ```cpp
        struct A { void foo(); };
        struct B { A* operator->(); };
        struct C { B operator->(); };
        struct D { C operator->(); };
        int main() {
            D d;
            d->foo();
           
            // expands to:
            // (*d.operator->().operator->().operator->()).foo();
            //   D            C            B           A*
        }
        ```
        * 
    * 注意**這邊 Primer p.570 頁第二個等價的 expression，格式看起來有誤**
        ```cpp
        (*point).mem; // point is a built-in pointer type
        
        // *** error ***
        point.operator()->mem; // point is an object of class type
        point.operator->().mem; // correct
        ```
* 總之你可以想成，如果某個物件他的 `operator->()` 回傳的不是指標，也就是回傳的是物件，他就會拿這個回傳物件的 `operator->` 來呼叫
    * **只有當回傳的型態是指標時才會使用指標內建的 -> 那個運算，也就是 `(*A).B`**
    * 如果回傳的是物件，但卻沒有 `operator->()` 可用，那就噴 error
* 這也是為什麼上面實作的 `operator->` 是回傳指標的原因

* 總結: The overloaded arrow operator **must return either a pointer** to a class type **or an object of a class type that defines its own operator arrow.**
* 感覺實務上不會讓 `operator->()` 回傳物件，太可怕了
    * 可能會生出上面那種遞迴 code

## 14.8 Function-Call Operator
* 可以讓 class object 用起來像 function
    * **callable object**
* Because such classes can also store state, they can be more flexible than ordinary functions.
    * 傳說中的 functor
* Toy example:
    ```cpp
    struct absInt {
        int operator()(int val) const
            { return val < 0 ? -val : val; }
    };
    ```
    * 用起來就很像 function
    ```cpp
    int i = -42;
    absInt absObj;
    // object that has a function-call operator
    int ui = absObj(i); // passes ito absObj.operator()
    ```
* `operator()` 一定要定義成 member
* class 一樣可以定義多個 `operator()`
    * 但是要跟第六章介紹的一樣，para list 數量/型態要不同

* 然後我們說 class 有定義 `operator()` 的話，class 的物件就叫做 **function objects**，"act like functions"

#### Function-Object Classes with State
* Function-object classes often contain data members that are used to customize the operations in the call operator.
    ```cpp
    class PrintString {
    public:
        PrintString(ostream &o = cout, char c = ' '):
            os(o), sep(c) { }
        void operator()(const string &s) const
            { os << s << sep; }
    private:
        ostream &os; // stream on which to write
        char sep;
    };
    ```
    * 你可以根據初始化時提供的參數，決定在使用 `operator()` 時會做的事情
    * 其實越看越像 lambda object 了
    ```cpp
    PrintString printer; // uses the defaults; prints to cout
    printer(s); // prints s followed by a space on cout
    PrintString errors(cerr, '\n');
    errors(s); // prints s followed by a newline on cerr
    ```
* function objects 最常用在當作 generic algorithm 的 predicate:
    ```cpp
    for_each(vs.begin(), vs.end(), PrintString(cerr, '\n'));
    ```
    
### 14.8.1 **Lambdas Are Function Objects**
* **When we write a lambda, the compiler translates that expression into an *unnamed object of an unnamed class* (§ 10.3.3, p. 392).**
* 一個沒有名字的 class 的沒有名字的 object
* **這個 class 包含了一個 `operator()` member!**
    ```cpp
    // sort words by size, but maintain alphabetical order for words of the same size
    stable_sort(words.begin(), words.end(),
        [](const string &a, const string &b)
            { return a.size() < b.size();});
    ```
    * 上面的 lambda 宛如:
    ```cpp
    class ShorterString {
    public:
        bool operator()(const string &s1, const string &s2) const
            { return s1.size() < s2.size(); }
    };
    ```
    * 至於為什麼上面這個 class 的 `operator()` 有 `const` 呢? 那是因為 lambda 預設不能跟改 capture list 內的值(10.3.3, p.395)，所以 `operator()` 就宣告成 `const`
    * 如果要可以改 capture list 的值必須把 lambda 宣告成 `mutable`(一樣 10.3.3, p.395)，這樣對應的 `operator()` 就不會是 `const`
    * 所以上面的 `stable_sort` 也可以改寫成:
    ```cpp
    stable_sort(words.begin(), words.end(), ShorterString());
    ```
* 以下是標題的三段論證
    * 因為 lambda 其實是 unnamed object of unnamed class which define call operator
    * 再根據前述，Objects of classes that define the call operator are referred to as function objects.
    * 所以 lambda 是 function object

#### Classes Representing Lambdas with Captures
* 那我們要怎麼用 class 實作 lambda 的 capture list 呢?
    * 其實就是 class 的 data member
    * 而根據你是 capture by value/reference，這個 data member 就是 value/reference

* 例如很久以前這樣寫的 code:
    ```cpp
    // get an iterator to the first element whose size() is >=sz
    auto wc = find_if(words.begin(), words.end(),
        [sz](const string &a) {return a.size() > sz;}
    ```
    * 上面的 lambda 生出來的 class 大概就會長這樣:
        ```cpp
        class SizeComp {
        public:
            SizeComp(size_t n): sz(n) { } // parameter for each captured variable 
            // call operator with the same return type, parameters, and body as the lambda 
            bool operator()(const string &s) const
                { return s.size() >= sz; }
        private:
            size_t sz; // a data member for each variable captured by value
        };
        ```
        * 在 C++11 之前，要寫出可存內部狀態的 callable object 都要實作類似這種 class，叫做 functor
    * 所以上面寫的 `find_if` 可以這樣改寫:
    ```cpp
    // get an iterator to the first element whose size()is >=sz
    auto wc = find_if(words.begin(), words.end(), SizeComp(sz));
    ```
    * 注意這是把一個 `SizeComp` 的(temp)物件丟給 `std::find_if`，只是用 `SizeComp(size_t)` 初始化
    
### 14.8.2 Library-Defined Function Objects
* 一堆有名子，可以 call 的 class template... 定義在 `<functional>`
* ![](https://i.imgur.com/983MjA0.png)
* 拿來呼叫的時候，他會使用你給定的 type 對應的 operation 來運算
    ```cpp
    plus<int> intAdd; // function object that can add two int values
    negate<int> intNegate; // function object that can negate an int value 
    // uses intAdd::operator(int, int)to add 10 and 20
    int sum = intAdd(10, 20); // equivalent to sum = 30
    sum = intNegate(intAdd(10, 20)); // equivalent to sum= -30
    // uses intNegate::operator(int) to generate -10 as the second parameter 
    // to intAdd::operator(int, int)
    sum = intAdd(10, intNegate(10)); // sum=0
    ```
    
#### Using a Library Function Object with the Algorithms
* 這裡舉例比較快，例如 `sort` `vector<string>` 會按照字點順序排列(也就是使用 string 的 `operator<` 來排列)；你想要倒著排可以這樣寫:
    ```cpp
    // passes a temporary function object that applies the > operator to two strings
    sort(svec.begin(), svec.end(), greater<string>());
    ```
    * 會改成呼叫 `std::string` 的 `operator>()`
    * 其實使用這個 lib func object 也是有好處，如果要定義這種簡單的 function 直接拿 lib 給的 template 生一個就好了，不用自定義一個 function，然後傳進來

* 這鬼東西還可以用在排序指標上
    * 還記得直接 **對指向不同 container 的指標用比大小是 UB**
    * 可是我們可以用 `less<T*>` 來比較指標，標準定義這樣的行為是可以的
    * *但好像很雷，這邊看看就好*
* 所以如果你真的有要排列指標的需求可以這樣寫:
    * 一樣，看看就好
    ```cpp
    vector<string *> nameTable; // vector of pointers
    // error: the pointers in nameTableare unrelated, so < is undefined 
    sort(nameTable.begin(), nameTable.end(), [](string *a, string *b)
                    { return a < b; });
    // ok: library guarantees that less on pointer types iswell defined
    sort(nameTable.begin(), nameTable.end(), less<string*>());
    ```
* 還有一點，**關聯容器其實不是直接拿 key 的 `operator<` 來做排序的，而是拿 less`<key_type>` 來排序**，這其實也代表了我們可以直接在 key 裡面用指標，可以定義 set of pointers 或 map，其中 key 是指標
    * *一樣，對指標做排序，看看就好*

### 14.8.3 Callable Objects and function
* **參考: type erasure**
    * https://shininglionking.blogspot.com/2017/01/stdfunction-c11.html
    * https://www.youtube.com/watch?v=tbUCHifyT24


* 之前介紹過，C++ 有這麼多種 callable objects:
    * functions
    * pointers to functions
    * lambdas (§ 10.3.2, p. 388)
    * objects created by `std::bind` (§ 10.3.4, p. 397)
    * and classes that overload the function-call operator

* 既然這些是 callable *object*，他就有 type
    * 只不過有些 type 沒有名字

* 但是 type 雖然可能不同，但是 callable objects 可能有相同的 **call signature**
    * 就是你很久很久以前學過的那個 signature，第一次可能叫做function signature，但是 C++ 有很多東西可以當 function 一樣來呼叫，所以叫 call signature
    * 例如 `int(int, int)`
* call signature 指定了 return type 跟 para list

#### Different Types Can Have the Same Call Signature
* Sometimes we want to **treat several callable objects that share a call signature as if they had the same type.**
    ```cpp
    // ordinary function
    int add(int i, int j) { return i + j; }
    // lambda, which generates an unnamed function-object class
    auto mod = [](int i, int j) { return i % j; };
    // function-object class
    struct divide {
        int operator()(int denominator, int divisor)
            { return denominator / divisor; }
    };
    ```
    * 上面三個 callable objects 互為三個不同 type
    * 不過 call signature 都一樣: `int(int, int)`

* 這裡先說實務上可能會有一種應用，就是一個 container(可能是 `std::map`)，裡面存一堆 function pointer
    * 可能在 C 很常見吧，不過 C 的 callable objects 只有 function/function pointer，而且還可以互轉
* 但是前面已經看過了，C++ 有一狗屁的 callable objects，我們沒辦法存一個 function pointer，然後指向各種不同的 callable objects
    * 畢竟他們 type 不一樣:
    * 假設這樣定義 `std::map`，且 `add` 跟 `mod` 都是上面定義的:
    ```cpp
    // maps an operator to a pointer to a function
    // taking two ints and returning an int 
    map<string, int(*)(int,int)> binops;
    ```
    * 則這樣合法:
    ```cpp
    // ok: add is a pointer to function of the appropriate type 
    binops.insert({"+", add}); // {"+", add} is a pair§ 11.2.3 (p. 426)
    ```
    * 但這樣不合法:
    ```cpp
    binops.insert({"%", mod}); // error: mod is not a pointer to function
    ```
    * 不能拿普通的函數指標去指 lambda 物件

#### The Library `std::function` Type
:::info
這裡有空可以回去看看 `shared_ptr`
因為實務上 `shared_ptr` 的 deleter 是類似用 `std::function` 這種東西封裝的
這樣才能達到 runtime 換(*不同型態*)的 deleter 的效果
詳情看 12 章以及 16.1.5 講 deleter 的部分
:::

* 所以 C++11 定義了 `std::function` ㄏㄏ
    * **基本上就是 callable object 的 wrapper**
* 一樣是 template
    * `<>` 內指定 call signature
    * 有這種 call signature 的物件都可以拿來初始化 `std::function<call_signature>`
        * 問: 為何任何 type 都可以傳給 `std::function` ctor?
        * **因為 `std::function` 本身是個 template 之外，他的 ctor 也是個 template!**
* 例如這樣定義，`function<int(int, int)>`
* 則上面三個函數都可以 assign:
    ```cpp
    function<int(int, int)> f1 = add; // function pointer
    function<int(int, int)> f2 = divide(); // object ofa function-object class
    function<int(int, int)> f3 = [](int i, int j) // lambda { return i * j; };
    cout << f1(4,2) << endl; // prints 6
    cout << f2(4,2) << endl; // prints 2
    cout << f3(4,2) << endl; // prints 8
    ```
    
* 然後我們的 container 就可以這樣定義:
    ```cpp
    // table of callable objects corresponding to each binary operator 
    // all the callables must take two ints and return an int 
    // ***an element can be a function pointer, function object, or lambda***
    map<string, function<int(int, int)>> binops;
    ```
    * `mapped_type` 吃 `std::function<call_signature>`
    * 於是所有有 `int(int, int)` call signature 的物件都可以當 value 了:
    ```cpp
    map<string, function<int(int, int)>> binops = {
        {"+", add}, // function pointer
        {"-", std::minus<int>()}, // library function object
        {"/", divide()}, // user-defined function object
        {"*", [](int i, int j) { return i * j; }}, // unnamed lambda
        {"%", mod} }; // named lambda object
    ```
    * 也可以這樣寫扣:
    ```cpp
    binops["+"](10, 5); // calls add(10, 5)
    binops["-"](10, 5); // uses the call operator of the minus<int>object 
    binops["/"](10, 5); // uses the call operator of the divide object 
    binops["*"](10, 5); // calls the lambda function object
    binops["%"](10, 5); // calls the lambda function object
    ```
    * `std::map["blabla"]` 回傳 `std::function`，對他使用 `operator()` 則會呼叫他內部存的 callable object 的 `operator()`

#### Overloaded Functions and function
* 如果我們有 overload functions，並且想要把其中一個 function assign 給 `std::function`，情況會有點複雜:
    ```cpp
    int add(int i, int j) { return i + j; } // add 1
    Sales_data add(const Sales_data&, const Sales_data&); // add 2
    map<string, function<int(int, int)>> binops;
    binops.insert( {"+", add} ); // error: which add?
    ```
    * 注意 `binops.insert( {"+", add} );` 這一行，如果我們只有一個 `add` 的話是不會噴 error 的
    * 可是我們現在 overload `add` 就是不行，雖然你會覺得很容易知道要選有 `int(int, int)` call signature 的 `add`
    * 真正的原因是因為 template 的機制，詳情 https://stackoverflow.com/questions/30393285/stdfunction-fails-to-distinguish-overloaded-functions
    * 看不懂的話等 16章，這裡先記起來
    * 本質上是因為 std::function ctor 是 template
    * 反而造成他什麼都可以吃
        * 兩種 add 都可以吃
        * 反而無法決定哪個 add 絕對不可能 viable

* 解法是用 function pointer 或 lambda:
    ```cpp
    int (*fp)(int,int) = add; // pointer to the version of add that takes two ints
    binops.insert( {"+", fp} ); // ok: fp points to the right version of add
    ```
    ```cpp
    // ok: use a lambda to disambiguate which version of add we want to use 
    binops.insert( {"+", [](int a, int b) {return add(a, b);} } );
    ```
* 題外話，其實在這種情境下，你寫 `auto p = add;` 也會錯

* 以下是 `function` 可用操作:
* ![](https://i.imgur.com/EeBxKFA.png)


## 14.9 Overloading, Conversions, and Operators
* 以前講過，non`explicit` 的只有一個參數的 cstr 等於定義了從 parameter type 到 class type 的轉換 -- converting constructors
* 我們其實也**可以定義從 class type 到某種 type 的轉換**
* **conversion operator**
* Converting constructors and conversion operators **define class-type conversions.** Such conversions are also **referred to as user-defined conversions.**

### 14.9.1 Conversion Operators
* 先警告，將 class type 轉型成 non`bool` type 通常不是好東西，很容易成為 bad design
* A conversion operator is a special kind of member function
    * that **converts a value of a class type to a value of some other type.**
    ```cpp
    operator type() const;
    ```
* Conversion operators can be defined for any type (other than `void`) that can be a function return type (§ 6.1, p. 204).
    * 要把 class object 轉成什麼型態都可以，除了 `void` 之外的 first class type 都可
    * function 不能 return array 或 function，所以這兩種也不行
* 一定要定義成 member function
* conversion operator 通常不改變物件本身，所以宣告成 `const`
* Toy example:
    ```cpp
    class SmallInt {
    public:
        SmallInt(int i = 0): val(i) {
            if (i < 0 || i > 255)
                throw std::out_of_range("Bad SmallInt value");
        }
        operator int() const { return val; }
    private:
        std::size_t val;
    };
    ```
    * `SmallInt` 同時定義了 conversion **from and to** `int`
    * 因為有 converting ctor 跟 conversion operator

* 然後可以寫這種 code:
    ```cpp
    SmallInt si;
    si = 4; // implicitly converts 4 to SmallInt then calls copy assignment for SmallInt
    si + 3; // implicitly converts si to int followed by integer addition
    ```

* 注意很久很久以前講的，user defined conversion 只會"連續隱式轉換"一次(4.11.2 p.162)
    * **但是這種轉換之前或之後還是可以搭配 built-in 轉換**
* 換句話說其實 `SmallInt` 已經定義了 arithmetic type 的轉換，因為其他 arithmetic type 可以先轉成 int 再轉成 `SmallInt`，也可以從 `SmallInt` 轉成 int 之後再轉成其他 arithmetic type:
    ```cpp
    // the double argument is converted to int using the built-in conversion     
    SmallInt si = 3.14; // calls the SmallInt(int) constructor
    // the SmallInt conversion operator converts si to int;
    si + 3.14; // that int is converted to double using the built-in conversion
    ```
* 不能對 conversion operator 傳參數，而 conversion operator 回傳的型別就是你指定要轉換過去的型別
* 雖然沒有指定回傳型別，但是你 function body 內一定要回傳同型別的 expression
    ```cpp
    class SmallInt;
    operator int(SmallInt&); // error: nonmember
    class SmallInt { 
    public:
        int operator int() const; // error: return type
        operator int(int = 0) const; // error: parameter list
        operator int*() const { return 42; } // error: 42 is not a pointer
    };
    ```
    
:::danger
#### CAUTION: AVOID OVERUSE OF CONVERSION FUNCTIONS
* 良好設計的轉型介面好棒棒
    * 過度使用就會夭壽雞掰
* Conversion operators are misleading
    * when there is no obvious single mapping between the class type and the conversion type.

* 例如一個 class `Date`
    * 我們可能會覺得轉到 `int` 很 OK
    * 問題是他的 **value** 要是什麼? 怎麼表示?
        * 可能是將 年/月/日 轉成 int
        * 也可能是從 1970 開始算的 timestamp
    * The problem is that **there is no single one-to-one mapping between an object of type `Date` and a value of type `int`.**
    * 這時候提供一個轉成 `int` 的轉型就會是個很糟的介面

* In such cases, it is better not to define the conversion operator.
* Instead, the class ought to **define one or more ordinary members to extract the information in these various forms.**
:::

#### Conversion Operators Can Yield Suprising Results
* 這個真的怕爆
* Too often users are *more likely to be surprised* if a conversion happens automatically *than to be helped by* the existence of the conversion.
    * 常常 user code 是被隱式轉換雷到而不是幫助到
* **除了一個例外: 轉換成 `bool`**
* 通常實務上會定義 `bool` 的轉換

* 可是在早期的標準，如果要定義轉換成 `bool` 會遇到一個難題:
    * 所有這種 class 的物件都可以做 arithmetic operation!
    ```cpp
    int i = 42;
    cin << i; // this code would be legal if the conversion to boolwere not explicit!
    ```
    * 如果上面的 `cin` 轉成 bool 不是 `explicit`(記得這是 C++11 才有的)，那 `cin` 會先轉成 `bool`
        * `bool` 被 promote 成 `int`，然後再把 `<<` 當成 binary shift operator...

    * 為此，pre C++11 有另外一種解法，不過看起來一樣很髒
        * https://en.cppreference.com/w/cpp/language/implicit_conversion#The_safe_bool_problem
#### `explicit` Conversion Operators
* 所以定義了 `explicit` conversion operators:
    ```cpp
    class SmallInt {
    public: // the compiler won’t automatically apply this conversion 
    explicit operator int() const { return val; } 
    // other members as before
    };
    ```
* 如果照上面定義，則下面的 code 就會這樣:
    ```cpp
    SmallInt si = 3; // ok: the SmallInt constructor is not explicit
    si + 3; // error: implicit is conversion required, but operatorint is explicit
    static_cast<int>(si) + 3; // ok: explicitly request the conversion
    ```
    
* 不過唯一的例外就是使用在 condition 時:
    * 你不加 `static_cast`，compiler 也會當成你有加
    * 合理，因為這是常用 idiom
    * 所以之前那些什麼 `if(cin>>i)` **之類的其實不是隱式轉換，而是 explicit 轉換!**
    * An explicit conversion will be used implicitly.... 有沒有很靠北 XD
* 會在這裡"隱式的使用"顯式轉換:
    * ![](https://i.imgur.com/IlBF6pB.png)

#### Conversion to `bool`
* 早期標準的 IO type 其實沒有定義轉到 bool，因為會發生上述情況，他真正定義的是 `void*` 轉換... 這樣就可以避免上面說的問題
* 不過新標準記捨棄這種定義，改成定義 explicit conversion to bool

:::info
* Best Practice: Conversion to `bool` is usually intended for use in conditions.
* As a result, operator `bool` ordinarily should be defined as `explicit`.
* 基本上定義 `bool` 轉換是合理的，但是我們還是不希望 class 被隱性轉換成 `bool`，所以還是要宣告成 `explicit`，反正用在 condition 時還是會 "隱式的使用 `explicit` 轉換" XD
:::

### 14.9.2 Avoiding Ambiguous Conversions
* 到尾巴為止都先跳過
* 基本上可以說是一種 dark side 了
1. mutual conversion
    ```cpp
    class A {
        A(const B&); // 可用這個將 B 轉換成 A
    };
    
    class B {
        operator A(); // 可用這個將 B 轉換成 A
    }
    ```
    * 用看的頭就痛
2. defined multiple conversions of which the result types are related
    ```cpp
    class A {
        operator int();
        operator double();
        // int double 可以互轉，你的 user code 在使用 class A 時會不小心寫出 ambiguous (implicit conversion) call 
    };
    ```