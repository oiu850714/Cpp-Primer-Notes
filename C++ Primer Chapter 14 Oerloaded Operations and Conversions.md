# C++ Primer Chapter 14 Oerloaded Operations and Conversions

* Operator overloading lets us define the meaning of an operator when applied to operand(s) of a class type.
* **Judicious use** of operator overloading can make our programs easier to write and easier to read.
    * `cout << item1 + item2; // print the sum of two Sales_items`
    * `print(cout, add(data1, data2)); // print the sum of two Sales_datas`
        * 因為 `Sales_item` 有定義 operator +, =, << 的行為所以可以這樣寫
        * 反之 `Sales_data` 就沒有，code 看起來就不明確

## 14.1 Basic Concepts
* Overloaded operators are **functions with special names**: the keyword `operator` followed by the symbol for the operator being defined
    * 既然是 function，一樣有 return type, para list, 跟 function body

* 我們雖然可以定義 operator 的意義，不過 operand 的數量是不變的
    * unary, binary
    * binary 第一個 operand 就是 lhs，第二個是 rhs

* 除了 `operator()`，其他 operator 都不能有 default args
* 如果一個 operator 是定義成 class 的 member function，那他的第一個 operand 就會綁到 `this`，所以 para list 的參數數量會比這個 operator 所需要的 operand 數量少一個

* 一個 operator function 一定要馬是 memeber function，要馬是 global function，但是至少有一個 parameter 是 class type:
    ```C++
    // error: cannot redefine the built-in operator for ints 
    int operator+(int, int);
    ```
    * 這也意味著我們不能改變操作在 buil-in type 上的 operator 的意義(因為你沒辦法定義一個 operator function，其中所有參數都是 built-in type)

* 只有少部分的 operator 不能 overload，之後會講
* 19.1 會講怎麼 overload `new` 跟 `delete`
* 我們也不能創造新的 operator symbol
    * 比方說定義 `operator**` 之類的

* 而有些 symbol(+, -, \*, &) 同時是 unary/binary operator，這兩種都可以 overload，定義他們時的參數數量決定你是定義 unary/binary operator
* 還有一點，**operator 就算被 overloading，彼此之間的運算優先級以及結合律也不會改變**
    * `x== y+z;` 永遠等價於  `x== (y+ z);`

![](https://i.imgur.com/hsFlfBj.png)

#### Calling an Overloaded Operator Function Directly
* 你要直接 call operator function 還是間接使用 operator(配上對應的 operand)都可以，這兩者是等價的
    ```C++
    // equivalent calls to a nonmember operator function
    data1 + data2; // normal expression
    operator+(data1, data2); // equivalent function call
    ```
    * 兩個都會使用 nonmember function `operator+`
* 而如果 operator function 是定義成 member function，這樣寫才等價:
    ```C++
    data1 += data2; // expression-based "call"
    data1.operator+=(data2); // equivalent call to a member operator function
    ```
    * 其中如果 data1 改成指標， member access operator 就要改成 `->`

#### Some Operators Shouldn't Be Overloaded
* 這裡超ㄎㄧㄤ= =
* 因為 operator function 本質上是 function call，先前學過，function parameter 參數運算順序是未定義，這意味著 operator function 的參數運算也是未定義...
* 換句話說，那些原本運作在 built-in 上面有保證運算順序的 operator，用在 class object 上就沒有!
* `&&, ||, ,(comma),`
* 更慘的是 `&&` 跟 `||`，他們 overloading 之後並不會有 short circuit!!，兩個 operands 都會運算!
* 總之這些 operator 最好不要 overload!

* 另一個不要 overloading comma (還有 address of)的原因，是因為 C++ 已經為 class type 定義這兩個 operator 的意義了，你如果再重定義，會有奇怪的行為
    * address of 我可以理解，不過 comma 我就不知道惹... 待查

* Best Practice: Ordinarily, the comma, address-of, logical AND, and logical OR operators should not be overloaded.

#### Use Definitions That Are Consistent with the Built-in Meaning
* 設計 class 時，你應該要先想他提供什麼 operations(畢竟這就是 class 提供的 interface)
* 在這些 operations 之中，存在跟某些 operator 有邏輯對應關係(比方說概念上都是加法)的 operations，才適合使用 overloading(比方說用 operator+ 來實作加法)
* 更具體的例子如下:
    * 如果 class 支援 IO，你應該要定義 shift operators(`>>`, `<<`)，並且要讓他們的操作結果跟作用在 built-in 時的情況一致
    * 如果 class 會測試兩個 class 物件是否相等，你應該要定義 `operator==`；如果定義 `operator==` 也要定義 `operator!=`
    * 如果 class 物件存在一個順序(order)的概念，你應該要定義 `operator<`，如果定義 `operator<` 也要把其他 relational operators 都定義
        * 不過注意，sort 只會吃 `operator<` 並且用他組出其他 relational operators 的意義，而不是用你定義的 operators! 你定義其他 operators，最好也跟 sort 組出來的 operators 一致，亦即，你可以全部用 `operator<` 幹出來
        * 還有某些 class 只有等價但沒有順序的概念，這時候只要定義 operator==/!= 就好，不要定義小於，也不要用小於來定義 ==，都沒順序了怎麼用 < 定義 ==...
    * operator function 的 return type 也要跟使用再 built-in 的情況一致
        * 算數運算子應該要回傳 class object 的 value(nonreference)
        * 邏輯運算子要回傳 bool
        * assignment 跟 compound assignment 要回傳 reference to lhs

#### Assignment and Compound Assignment Operators
* `operator=` 13章講過ㄌ，不過概念上必須上 lhs 跟 rhs 的 value 一致
* 如果 class 有定義算術運算子跟bitwise運算子，最好也定義對應的 compound assignment
    * 而且 ?= 應該就要先做 lsh ? rhs 再把這個結果 assign 給 lhs
* 下面看一下，簡短的說明該怎麼定義 operator overloading，以及什麼時候不該 overload
* ![](https://i.imgur.com/FdayPu3.png)

#### Choosing Member or Nonmember Implementation
* 必須想我們要把 operator function 定義成 member 還是 global
* 有時候是都可以，有時候只能選一種
* 下面的 guideline 可以幫忙選:
    * The assignment (=), subscript ([]), call (()), and member access arrow (->) operators **must** be defined as members.
    * The compound-assignment operators ordinarily *ought* to be members. However, unlike assignment, they are not required to be members.
    * Operators that **change the state of their object or that are closely tied to their given type**—such as increment, decrement, and dereference—usually should be members.
    * *Symmetric* operators—those that might convert either operand, such as the arithmetic, equality, relational, and bitwise operators—usually should be **defined as ordinary nonmember functions.**
        ```C++
        class_name operator+(clas_name, class_name);
        class_name operator+(clas_name, convertible_type);
        class_name operator+(convertible_type, class_name);
        ```
        
* 如果你把 operator function 定義成 member，之前說過，第一個 operand 會綁到 this；
    ```C++
    string s = "world";
    string t = s + "!"; // ok: we can add a constchar*to a string
    string u = "hi" + s; // would be an error if+were a member ofstring
    ```
    * 如果 `operator+` 是定義成 member，那 `s + "!"` 就等於 `s.operator+("!");`
    * 而 `"hi" + s;` 就等於 `"hi".operator+(s);` 噴 error
* 因為 `string` 是把 + 定義成 global，所以 `s + "!";` 等於 `operator+(s, "!");`，`"hi" + s;` 等於 `operator+("hi", s);`
    * const char* 都會被轉換成 `string` 再用 operator(string, string); 做運算
    * 總之這樣定義的話，只需要滿足其中一個 operand 是 `string`，然後兩個 operands 都可以(unambiguously)轉換成 `string` 就行

## 14.2 Input and Output Operators
### 14.2.1 Overloading the Output Operator <<
* first parameter of an output operator is a reference to a non`const` `ostream` object.
    * nonconst 因為 write to ostream 會改變 ostream 的 state
    * parameter 是 reference 是因為不能 copy。這兩個之前都講過了
* The second parameter ordinarily should be a reference to const of the class type wewant to print.
    * 通常 output object "邏輯上"不會改變 object，所以用 const reference
        * reference 避免 copy
    * 如果 object 內容有實作上必須會改變的 member，請用 `mutable`
* return type 要跟 C++ lib overload 的 << 一樣，回傳 `ostream`

#### The Sales_data Output Operator
* 拿 Sales_data 來舉例
    ```C++
    ostream &operator<<(ostream &os, const Sales_data &item) {
        os << item.isbn() << " " << item.units_sold 
           << " " << item.revenue << " " << item.avg_price();
        return os;
    }
    ```
    * 其實做的事情就跟之前寫的 `print` 一樣

#### Output Operators Usually DoMinimal Formatting
* 這是一個 practice
    * 例如上面的 << 沒有印出 newline
* An output operator that does minimal formatting **lets users control the details of their output.**

#### IO Operators Must Be Nonmember Functions
* lhs 就一定要是 IO type...
* 因為 >> << 要是 nonmember，然後 IO 通常又會讀 class 的 nonpublic member，所以要在 class 內宣告成 friend

### 14.2.2 Overloading the Input Operator >>
* 一樣 nonmember
* 第一個參數是 `istream&`
* 第二個參數是 **nonconst** reference
* 回傳第一個參數
    ```C++
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
    * 這裡還對 IO 做一些 error handling，自己看
* Note: Input operators **must deal with the possibility that the input might fail**; output operators generally don’t bother.

#### Errors during Input
* 總之 input 就有可能 error
* 這裡有個小技巧，與其每個 member 都檢查有沒有讀取成功，不如一次讀近來然後檢查 istream，有 error 就把 object 設成某個狀態

* Putting the object into a valid state is especially important if the object might have been partially changed before the error occurred.
* Best Practice: Input operators should decide what, if anything, to do about error recovery.
* 反正這裡 error handling 看情況啦，上面是 toy example

#### Indicating Errors
* 除了 error handling，你還要有一種方式來告知 user code 有 error
* 就算有成功讀取資料到 data member，更進階的狀況是還有知道資料個是有無正確等等
* 這時候可以利用 iostream 的那些 flag 來告知 user code!!!!!
    * 幹好猛ㄛ，這樣就可以充分使用那些 flag 惹
    * 但通常只會設定 failbit，另外的 eof 跟 badbit 最好讓 iostream object 自己設定比較好

## 14.3 Arithmetic and Relational Operators
* 再說一次，nonmember
* 一般來說給定的參數不會被改變，所以用 const reference
* 通常這些 operator 都是藉由 operands 計算出一個 **local** value，然後回傳 value
* 在來就是，通常會定義這些 operators，也會定義對應的 compound assignment
    * 會傾向在一般版本內使用對應的 compound assignment 來實作:
    ```C++
    // assumes that both objects refer to the same book 
    Sales_data
    operator+(const Sales_data &lhs, const Sales_data &rhs) {
        Sales_data sum = lhs; // copy data members from lhs into sum 
        sum += rhs;
    // add rhs into sum
        return sum;
    }
    ```
    * 某整程度上有 code reuse

### 14.3.1 Equality Operators
* 基本上就是呼叫每個 member 的 == 來做 Logical AND...，除非你自己知道有些 member 的 value 不在你的"相等"的概念內
    ```C++
    bool operator==(const Sales_data &lhs, const Sales_data &rhs) {
        return lhs.isbn() == rhs.isbn() 
        && lhs.units_sold == rhs.units_sold 
        && lhs.revenue == rhs.revenue;
    }
    bool operator!=(const Sales_data &lhs, const Sales_data &rhs) {
        return !(lhs == rhs);
    }
    ```
   * 注意又是一個用已定義的 operator 來實作另一個 operator 的例子
* 以下是 design principle:
    * 還有如果 class 有等價的概念，請定義 ==，不要使用 named function 來當 interface，這樣 user code 還要額外記要怎麼用
        * 而且很多 STL API 都會用 operator==，一定義好很多 function 都可以用了
    * == 真的就是要拿來代表說兩個 class 物件"等價"的，不要拿去做其他事
    * 還有你定義出來的操作要有 transitive:
        * a == b 且 b == c, 則 a == c
    * 定義了 ==，也請定義 !=
    * 還有上面的其中一個應該要由另一個來實作!!

### 14.3.2 Relational Operators
* 想要定義 `operator<` 的話必須滿足兩個要求
    * strict weak ordering(詳情看 11.2)
    * 如果已經為 class 定義了 `operator==`，那麼你由 strict weak ordering 組出來的 ==，亦即 !(a<b) && !(b<a)，要跟你已經定義的 `operator==` 一致

* 由上面的例子可以發現，`Sales_data` 不該定義 `operator<`
* 首先他已經定義了 `operator==`，回傳結果是比較 `Sales_data` 三個 member 的值
        * 這裡只能這樣定義 ==，因為如果有任何一個 member 不一樣，邏輯上(或說現實上)這兩筆 data 都不會當成相同
* 那如果假設我們的 `operator<` 是只有比較 isbn 的話，那會發生無法滿足第二個要求的情況
    * isbn 一樣，可是另外兩個 member 不一樣，這時候 `operator<` 組出來的 == 會為真，可是自定義的 operaotr== 卻為假
* 你可能會覺得不然把另外兩個 member 也拿來比較，先比 isbn 再來 unit_sold 再 revenue 之類的，可以滿足第二點(因為三個 member 都比到，如果三個 member 真的都一樣，`operator<` 組出來的 == 跟 operator== 的行為就會一致
    * 可是這樣子的比較本質上沒有任何意義，而且如果先比 isbn 再比 revenue 再 unit_sold 也是一種比較方式ㄚ? 你這樣要 user code 認為是哪一種? 之前說過一個 operator 如果會有 ambiguous，就不該 overloading
* 結論就是 `Sales_data` 這個例子不適合定義 operator<
* 不過還是有例外啦，有 partial order 之類的鬼東西，不過這比較像數學
    * https://en.wikipedia.org/wiki/Lattice_(order)
    * 你可以想成，如果你定義出來的 <，就算不能滿足第二點，可是排出來的結果還是有意義的話，那定義 < 就有意義

## 14.4 Assignment Operators
* 除了 13章講的 `operator=`，還可以定義不同 type 的 rhs，沒有一定要 class type
* 例如 `std::vector`:
    ```C++
    vector<string> v;
    v = {"a", "an", "the"};
    ```
* 或額外為 `StrVec` 定義:
    ```C++
    class StrVec {
    public:
        StrVec &operator=(std::initializer_list<std::string>);
        // other members as in § 13.5 (p. 526)
    };
    ```
* 記得都要 return rhs&
    ```C++
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
* 還有之前說過，只能定義成 member

#### Compound-Assignment Operators
* 他跟 `operator=` 不一樣，不用是 member，但不要自找麻煩，請定義成 member
* 一樣 return lhs&
    ```C++
    // member binary operator: left-hand operand is bound to the implicit this pointer 
    Sales_data& Sales_data::operator+=(const Sales_data &rhs) {
        units_sold += rhs.units_sold;
        revenue += rhs.revenue;
        return *this;
    }
    ```
    * assumes that both objects refer to the same book

## 14.5 Subscript Operator
* 如果你的 class 用起來像 container，通常會定義這個 operator
* 一定要是 member function
* return reference
    * 可以當 lvalue
* 最好做 const overload，這樣 const 物件就會呼叫 const 版本
    * 如果沒定義 const 版本，const 物件也不能呼叫 nonconst 版本

* 用 `StrVec` 舉例:
    ```C++
    class StrVec {
    public:
        std::string& operator[](std::size_t n)
            { return elements[n]; }
        const std::string& operator[](std::size_t n) const 
            { return elements[n]; }
        // other members as in § 13.5 (p. 526)
    private:
        std::string *elements; // pointer to the first element in the array
    };
    ```
    * 注意有 const overload

* 可以這樣用:
    ```C++
    // assume svec is a StrVec
    const StrVec cvec = svec; // copy elements from svec into cvec
    // if svec has any elements, run the string empty function on the first one 
    if (svec.size() && svec[0].empty()) {
        svec[0] = "zero"; // ok: subscript returns a reference to a string 
        cvec[0] = "Zip"; // error: subscripting cvec returns a reference to const
    }
    ```
    * 對 `const StrVec` assign 會噴 error

## 14.6 Increment and Decrement Operators
* 用起來 iterator 的 class 會實作
* 一樣不用是 member，但請定義成 member，因為他會改變 object state
* 有 prefix 跟 postfix 兩種
    * 那我們要怎麼分別定義他們呢?
* 會先講怎麼定義 prefix 再講 postfix

#### Defining Prefix Increment/Decrement Operators
* 一樣用 `StrBlobPtr`:
    ```C++
    class StrBlobPtr {
    public:
        // increment and decrement
        StrBlobPtr& operator++();
        StrBlobPtr& operator--(); 
        // other members as before
    };
    ```
    * 注意 return reference
* 實作:
    ```C++
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
    * 注意 -- 的版本，要先 --curr 再 check，如果原本 curr 是 0，-- 之後會變最大正數(因為他是 unsigned)，帶進 check 才會炸裂；

#### Differentiating Prefix and Postfix Operators
* Normal overloading cannot distinguish between these operators.
* 用很ㄎㄧㄤ的解法: 讓後置版本多吃一個參數...:
    ```C++
    class StrBlobPtr { 
    public:
        // increment and decrement 
        StrBlobPtr operator++(int); // postfix operators     
        StrBlobPtr operator--(int); 
        // other members as before
    };
    ```
    * When we use a postfix operator, the compiler supplies 0 as the argument for this parameter.
    * Its sole purpose is to distinguish a postfix function from the prefix version.
    * 最後最重要的，return 的是 value

* 實作:
    ```C++
    // postfix: increment/decrement the object but return the unchanged value
    StrBlobPtr StrBlobPtr::operator++(int) {
        // no check needed here; the call to prefix increment will do the check
        StrBlobPtr ret = *this; // save the current value 
        ++*this; // advance one element; prefix ++ checks the increment
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
#### Calling the Postfix Operators Explicitly
* 你用符號的方式呼叫後置版本，可以不用提供那個 int；可是你用 function call 來呼叫就一定要提供那個 int 惹
* 如果你沒提供的話也不會噴 error，可是會 call 到 prefix 版本的:
    ```C++
    StrBlobPtr p(a1); // ppoints to the vector inside a1
    p.operator++(0); // call postfix operator++
    p.operator++(); // call prefix operator++
    ```
    
## 14.7 Member Access Operators
* dereference (\*) and arrow (->)
* 常表示在 iterator 或 smart pointer class
    ```C++
    class StrBlobPtr { 
    public:
        std::string& operator*() const { 
            auto p = check(curr, "dereference past end");
            return (*p)[curr]; // (*p)is the vector to which this object points
        }
        std::string* operator->() const {
            // delegate the real work to the dereference operator 
            return & this->operator*();
        } 
        // other members as before
    };
    ```
    * 注意兩個都宣告成 const function，因為不會更改 `StrBlobPtr` 的 state
    * 這樣 const 跟非 const `StrBlobPtr` 都可以呼叫
* `operator*` 可以做任何事(儘管這樣不好)，你可以回傳 42 或三小，隨便
* 可是 `opetator->` 就不一樣了，詳細說明如下
* 當你寫一個 exp `A->B`，這時會等價於兩種情況
    * 如果 `A` 是指標，那 `A->B` 就等於 `(*A).B`
    * 可是如果 `A` 是一個有實作 `operator->` 的 class 物件，則 `A->B` 就等於 `A.operaotr->()->mem`，一個遞迴概念...
* 注意***這邊 Primer p.570 頁第二個等價的 expression，() 前面少了一個箭頭!***
* 總之你可以想成，如果某個物件他的 `operator->` 回傳的不是指標，也就是回傳的是物件，他就會拿這個回傳物件的 `operator->` 來呼叫；**只有當回傳的型態是指標時才會使用指標內建的 -> 那個運算，也就是 `(*A).B`
* 這也是為什麼上面實作的 `operator->` 看起來那麼奇怪是回傳指標的原因

* 總結: The overloaded arrow operator **must return either a pointer** to a class type **or an object of a class type that defines its own operator arrow.**

## 14.8 Function-Call Operator
* 可以讓 class object 用起來像 function
* Because such classes can also store state, they can be more flexible than ordinary functions.
    * 這感覺比 normal function 還要屌翻天(ry
* Toy example:
    ```C++
    struct absInt {
        int operator()(int val) const
            { return val < 0 ? -val : val; }
    };
    ```
    * 定義的時候看起來有夠ㄎㄧㄤ，兩個括號連在一起
    * 用起來就很像 function...
    ```C++
    int i = -42;
    absInt absObj;
    // object that has a function-call operator
    int ui = absObj(i); // passes ito absObj.operator()
    ```
* 一定要定義成 member
* 一樣可以定義多個 `operator()`，但是要跟第六章介紹的一樣，signature 要不同

* 然後我們說 class 有定義 `operator()` 的話，他的物件就叫做 **function objects**，"act like functions"

#### Function-Object Classes with State
* Function-object classes often contain data members that are used to customize the operations in the call operator.
    * 看例子比較快...
    ```C++
    class PrintString {
    public:
        PrintString(ostream &o = cout, char c = ’ ’):
            os(o), sep(c) { }
        void operator()(const string &s) const
            { os << s << sep; }
    private:
        ostream &os; // stream on which to write
        char sep;
    };
    ```
    * 你可以根據初始化時提供的參數，決定在使用 `operator()` 時會做的事情
    ```C++
    PrintString printer; // uses the defaults; prints to cout
    printer(s); // prints s followed by a space on cout
    PrintString errors(cerr, '\n');
    errors(s); // prints s followed by a newline on cerr
    ```
* function objects 最常用在當作 generic algorithm 的 predicate:
    ```C++
    for_each(vs.begin(), vs.end(), PrintString(cerr, '\n'));
    ```
    
### 14.8.1 Lambdas Are Function Objects
* When we write a lambda, the compiler translates that expression into an unnamed object of an unnamed class (§ 10.3.3, p. 392).
* 一個沒有名字的 class 的，沒有名字的 object
* 這個 class 包含了一個 `operator()` member!
    ```C++
    // sort words by size, but maintain alphabetical order for words of the same size
    stable_sort(words.begin(), words.end(),
        [](const string &a, const string &b)
            { return a.size() < b.size();});
    ```
    * 上面的 lambda 宛如:
    ```C++
    class ShorterString {
    public:
        bool operator()(const string &s1, const string &s2) const
            { return s1.size() < s2.size(); }
    };
    ```
    * 至於為什麼上面這個 class 的 `operator()` 有 `const` 呢? 那是因為 lambda 預設不能跟改 capture list 內的值，所以就宣告成 `const`
    * 如果要可以改 capture list 的值必須把 lambda 宣告成 `mutable`，這樣對應的 `operator()` 就不會是 `const`
    * 所以上面的 `stable_sort` 也可以改寫成:
    ```C++
    stable_sort(words.begin(), words.end(), ShorterString());
    ```

#### Classes Representing Lambdas with Captures
* 那我們要怎麼用 class 實作 lambda 的 capture list 呢?
* 概念上是這樣:
    * 還記得如果 capture list 是用 reference，則 programmer 有責任確保 lambda 被呼叫時，參考的變數還存在
        * 因為這樣，所以 compiler 可以直接假定在實作 `operator()` 時能用一個 reference 參數來綁物件，而不是使用一個 data member 來複製物件
    * 而如果是 capture by value，則必須要創造 lambda 時就把 value 複製；這其實也代表了 lambda object 的 cstr 必須接一個參數來傳入這個 value，而且 lambda object 還要有一個 member 來存這個 value

* 例如很久以前這樣寫的 code:
    ```C++
    // get an iterator to the first element whose size()is >=sz
    auto wc = find_if(words.begin(), words.end(),
        [sz](const string &a)
    ```
    * 上面的 lambda 生出來的 class 大概就會長這樣:
    ```C++
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
    * 具體的說就是，lambda capture list 內，cap by value 的，class cstr 就會有對應的參數，然後也會有對應的 private member；如果是 cap by reference，class 的 `operator()` 內就會有一個對應的 reference 參數。
    * 所以上面寫的 find_if 可以這樣改寫:
    ```C++
    // get an iterator to the first element whose size()is >=sz
    auto wc = find_if(words.begin(), words.end(), SizeComp(sz));
    ```
    
### 14.8.2 Library-Defined Function Objects
* 一堆有名子，可以 call 的 class template... 定義在 \<functional>
* ![](https://i.imgur.com/983MjA0.png)
* 拿來呼叫的時候，他會使用你給定的 type 對應的 operation 來運算
    ```C++
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
* 這裡舉例比較快，例如 sort `vector<string>` 會按照字點順序排列(也就是使用 string 的 `operator<` 來排列)；你想要倒著排可以這樣寫:
    ```C++
    // passes a temporary function object that applies the > operator to two strings
    sort(svec.begin(), svec.end(), greater<string>());
    ```
    * 其實使用這個 lib func object 也是有好處，如果要定義這種簡單的 function 直接拿 lib 給的 template 生一個就好了，不用自定義一個 function，然後傳進來

* 這個鬼東西還可以用在排序指標(?!)上!還記得直接**對指標用 < 是未定義行為**，可是我們可以用 less<T*> 來比較指標! 標準定義這樣的行為是可以的...
* 所以如果你真的有要排列指標的需求可以這樣寫:
    ```C++
    vector<string *> nameTable; // vector of pointers
    // error: the pointers in nameTableare unrelated, so < is undefined 
    sort(nameTable.begin(), nameTable.end(), [](string *a, string *b)
                    { return a < b; });
    // ok: library guarantees that less on pointer types iswell defined
    sort(nameTable.begin(), nameTable.end(), less<string*>());
    ```
* 還有一點，**關聯容器其實不是直接拿 key 的 `operator<` 來做排序的，而是拿 less`<key_type>` 來排序**，這其實也代表了我們可以直接在 key 裡面用指標，可以定義 set of pointers 或 map，其中 key 是指標

### 14.8.3 Callable Objects and function
* 之前介紹過的 callable objects:
    * functions
    * pointers to functions
    * lambdas (§ 10.3.2, p. 388)
    * objects created by bind (§ 10.3.4, p. 397)
    * and classes that overload the function-call operator

* 既然他是 object，他就有 type
    * 只不過有些 object 的 type 沒有名字

* 不過兩種不同 type 的 callable objects 可能有相同的 **call signature**
    * 就是你很久很久以前學過的那個 signature，第一次可能叫做function signature，但是 C++ 有很多東西可以當 function 一樣來呼叫，所以叫 call signature
    `int(int, int)`
* call sig 指定了 return type 跟 para list

#### Different Types Can Have the Same Call Signature
* Sometimes we want to **treat several callable objects that share a call signature as if they had the same type.**
    ```C++
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
    * 三個不同 type，不過 call sig 都一樣: `int(int, int)`

* 這裡先說實務上可能會有一種應用，就是一個 container，裡面存一堆 function pointer
* 可能在 C 很常見吧，不過 C 的 callable objects 只有 function/function pointer
* 但是前面已經看過了，C++ 有一狗屁的 callable objects，我們沒辦法存一個 function pointer，然後指向各種不同的 callable objects，畢竟他們 type 不一樣:
    * 假設這樣定義 map，且 add 跟 mod 都是上面定義的:
    ```C++
    // maps an operator to a pointer to a function taking two intsand returning an int 
    map<string, int(*)(int,int)> binops;
    ```
    * 則這樣合法:
    ```C++
    // ok: addis a pointer to function ofthe appropriate type 
    binops.insert({"+", add}); // {"+", add} is a pair§ 11.2.3 (p. 426)
    ```
    * 這樣不合法:
    ```C++
    binops.insert({"%", mod}); // error: mod is not a pointer to function
    ```
    * 不能拿普通的函數指標去指 lambda 物件

#### The Library function Type
* 所以 C++11 定義了 `function` 啦ㄏㄏ，基本上就是 callable object 的 wrapper
* template，\<> 內指定 call signature，有這種 call signature 的物件都可以被 assin 給 `function` 物件
* 例如這樣定義，`function<int(int, int)>`
* 則上面三個函數都可以 assign:
    ```C++
    function<int(int, int)> f1 = add; // function pointer
    function<int(int, int)> f2 = divide(); // object ofa function-object class
    function<int(int, int)> f3 = [](int i, int j) // lambda { return i * j; };
    cout << f1(4,2) << endl; // prints 6
    cout << f2(4,2) << endl; // prints 2
    cout << f3(4,2) << endl; // prints 8
    ```
    
* 然後我們的 container 就可以這樣定義:
    ```C++
    // table of callable objects corresponding to each binary operator 
    // all the callables must take two intsand return an int 
    // an element can be a function pointer, function object, or lambda
    map<string, function<int(int, int)>> binops;
    ```
    * 於是所有有 `int(int, int)` call signature 的物件都可以當 value 了:
    ```C++
    map<string, function<int(int, int)>> binops = {
        {"+", add}, // function pointer
        {"-", std::minus<int>()}, // library function object
        {"/", divide()}, // user-defined function object
        {"*", [](int i, int j) { return i * j; }}, // unnamed lambda
        {"%", mod} }; // named lambda object
    ```
    * 也可以這樣寫扣:
    ```C++
    binops["+"](10, 5); // calls add(10, 5)
    binops["-"](10, 5); // uses the call operator of the minus<int>object 
    binops["/"](10, 5); // uses the call operator of the divide object 
    binops["*"](10, 5); // calls the lambda function object
    binops["%"](10, 5); // calls the lambda function object
    ```
    * `[]` 回傳 callable object，然後再用他的 `operator()`

#### Overloaded Functions and function
* 如果我們有 overload function 情況會有點機掰:
    ```C++
    int add(int i, int j) { return i + j; }
    Sales_data add(const Sales_data&, const Sales_data&);
    map<string, function<int(int, int)>> binops;
    binops.insert( {"+", add} ); // error: which add?
    ```
    * 注意 `binops.insert( {"+", add} );` 這一行，如果我們只有一個 `add` 的話是不會噴 error 的，可是我們現在 overload `add` 就是不行，雖然你會覺得很容易知道要選有 `int(int, int)` call signature 的 `add`
    * 真正的原因是因為 template 的機制，詳情 https://stackoverflow.com/questions/30393285/stdfunction-fails-to-distinguish-overloaded-functions
    * 看不懂的話等 16章，這裡先記起來

* 解法是用 function pointer 或 lambda:
    ```C++
    int (*fp)(int,int) = add; // pointer to the version of add that takes two ints
    binops.insert( {"+", fp} ); // ok: fp points to the right version ofadd
    ```
    ```C++
    // ok: use a lambda to disambiguate which version of add we want to use 
    binops.insert( {"+", [](int a, int b) {return add(a, b);} } );
    ```

* 以下是 `function` 可用操作:
* ![](https://i.imgur.com/EeBxKFA.png)


## 14.9 Overloading, Conversions, and Operators
* 以前講過，non`explicit` 的只有一個參數的 cstr 等於定義了從 parameter type 到 class type 的轉換 -- converting constructors
* 我們其實也**可以定義從 class type 到某種 type 的轉換**
* **conversion operator**
* Converting constructors and conversion operators **define class-type conversions.** Such conversions are also **referred to as user-defined conversions.**

### 14.9.1 Conversion Operators
* A conversion operator is a special kind of member function that converts a value of a class type to a value of some other type.
    ```C++
    operator type() const;
    ```
* Conversion operators can be defined for any type (other than void) that can be a function return type (§ 6.1, p. 204).
    * 也不能轉成 array 或 function type lol
    * 不過可以轉成 pointer 跟 reference type(to data member/functions)...

* 沒有明確的參數跟回傳值
* 一定要定義成 member
* conversion 不應該改變物件本身，所以宣告成 const
* Toy example:
    ```C++
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
    * `SmallInt` 同時定義了 conversion **from and to** int，因為有 cstr 跟 conversion operator

* 然後可以寫這種 code:
    ```C++
    SmallInt si;
    si = 4; // implicitly converts 4 to SmallInt then calls SmallInt::operator=
    si + 3; // implicitly converts si to int followed by integer addition
    ```

* 注意很久很久以前講的，user defined conversion 只會"連續隱式轉換"一次(4.11.2 p.162)，但是這種轉換之前或之後還是可以搭配 built-in 轉換
* 換句話說其實 `SmallInt` 已經定義了 arithmetic type 的轉換，因為其他arithmetic type 可以先轉成 int 再轉成 `SmallInt`，也可以從 `SmallInt` 轉成 int 之後再轉成其他 arithmetic type:
    ```C++
    // the double argument is converted to int using the built-in conversion     
    SmallInt si = 3.14; // calls the SmallInt(int) constructor
    // the SmallInt conversion operator converts si to int;
    si + 3.14; // that int is converted to double using the built-in conversion
    ```
* 不能對 conversion operator 傳參數，而 conversion operator 回傳的型別就是你指定要轉換過去的型別
* 雖然沒有指定回傳型別，但是你 function body 內一定要回傳同型別的 expression
    ```C++
    class SmallInt;
    operator int(SmallInt&); // error: nonmember
    class SmallInt { 
    public:
        int operator int() const; // error: return type
        operator int(int = 0) const; // error: parameter list
        operator int*() const { return 42; } // error: 42 is not a pointer
    };
    ```
#### CAUTION: AVOID OVERUSE OF CONVERSION FUNCTIONS
* 良好設計的轉換介面好棒棒，過度使用就會夭壽雞掰掰
* Conversion operators are misleading when there is no obvious single mapping between the class type and the conversion type.

* 例如一個 class `Date`，我們可能會覺得轉到 int 很 OK，問題是他的 **value** 要是什麼? 怎麼表示?
    * The problem is that **there is no single one-to-one mapping between an object of type Date and a value of type int.**

* In such cases, it is better not to define the conversion operator. Instead, the class ought to **define one or more ordinary members to extract the information in these various forms.**

#### Conversion Operators Can Yield Suprising Results
* ...操你媽
* Too often users are more likely to be surprised if a conversion happens automatically than to be helped by the existence of the conversion.
    * 常常 user code 是被隱式轉換雷到而不是幫助到
* 除了一個例外: 轉換成 bool
* 通常實務上會定義 bool 的轉換

* 可是在早期的標準，如果要定義轉換成 bool 會遇到一個難題: 所有這種 class 的物件都可以做 arithmetic operation!
    ```C++
    int i = 42;
    cin << i; // this code would be legal if the conversion to boolwere not explicit!
    ```
    * 如果上面的 cin 轉成 bool 不是 `explicit`，那 cin 會先轉成 `bool`，`bool` 被 promote 成 `int`，然後再把 << 當成 shift operator...

#### explicit Conversion Operators
* 所以定義了 explicit conversion operators:
    ```C++
    class SmallInt {
    public: // the compiler won’t automatically apply this conversion 
    explicit operator int() const { return val; } 
    // other members as before
    };
    ```
* 如果照上面定義，則下面的 code 就會這樣:
    ```C++
    SmallInt si = 3; // ok: the SmallInt constructor is not explicit
    si + 3; // error: implicit is conversion required, but operatorint is explicit
    static_cast<int>(si) + 3; // ok: explicitly request the conversion
    ```
    
* 不過唯一的例外就是使用在 condition 時，你不加 `static_cast`，compiler 也會當成你有加，所以之前我們那些什麼 if(cin>>i) **之類的其實不是隱式轉換，而是 explicit 轉換**!!
* An explicit conversion will be used implicitly.... 有沒有很機掰
* 會在這裡"隱式的使用"顯式轉換:
    * ![](https://i.imgur.com/IlBF6pB.png)

#### Conversion to bool
* 早期標準的 IO type 其實沒有定義轉到 bool，因為會發生上述情況，他真正定義的是 `void*` 轉換... 這樣就可以避免上面說的問題 LOL
* 不過新標準記捨棄這種定義，改成定義 explicit conversion to bool

* Best Practice: Conversion to bool is usually intended for use in conditions. As a result, operator bool ordinarily should be defined as explicit.
    * 基本上會定義 bool 轉換，不過要用 explicit，反正用在 condition 時還是會 隱式的使用 explicit 轉換ㄏㄏ

### 14.9.2 Avoiding Ambiguous Conversions
* 到尾巴為止都先跳過... 很難用到也記不得