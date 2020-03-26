---
tags: C++
---
* 參考
* rule of three/five
    * https://www.fluentcpp.com/2019/04/19/compiler-generated-functions-rule-of-three-and-rule-of-five/
* move semantics
    * https://www.youtube.com/watch?v=St0MNEU5b0o

# C++ Primer Chapter 13 Copy Control
* 第七章講的是如何用 `class` 定義新 type
    * 包括 operation
    * 封裝，以及 constructor 這些基本議題
* 這章要講的是如何定義 class object 在 copy, assign, move, 還有 destroy 時的行為
* 這些 feature 分別對應到下面幾個，之後會講:
    * copy constructor
    * move constructor
    * copy-assignment operator
    * move-assignment operator
    * destructor


* 當我們定義 class 的時候，我們會(明確地)指定出，class object 在 copy, move, assign, destroy 時會發生什麼事
    * 沒有指定的話 compiler (可能會)幫你合成(synthesize)一個
    * 如果要明確指定的話就是要定義上面那五個 member functions:
        * copy/move constructor
            * 定義當我們用一個 class object 來初始化另一個 class object 時的行為
        * copy/move-assignment operator
            * 定義當我們把一個 class object assign 給另一個 class object 時的行為
        * destructor
            * 定義當 class object 要被消滅時的行為
* **上述的行為被當作 C++ 的 copy control**
* 有些 class 的實作導致 **一定要自己定義 copy control**，不能用 compiler 合成的，否則會悲劇
    * 什麼時候要自己定義之後會講
        * 通常是自己要 allocation dynamic resources 的時候

## 13.1 Copy, Assign, and Destroy

* 這三個是 pre C++11 就有的，所以先講


### 13.1.1 The Copy Constructor
* 定義
    * 只要是 class ctor，並且:
        * 第一個參數是 reference to class type
        * 第一個參數後面還有參數的話，全部都要有 default value
    ```cpp
    class Foo { 
    public:
        Foo(); // default constructor
        Foo(const Foo&, int a = 9487); // copy constructor
        // ...
    };
    ```
* 之後會講為什麼一定要用 (`const`)reference type
* 然後 reference 幾乎都是 `const`，雖然我們還是可以定義 non`const` 的版本(可以兩個都定義，low-level `const` 不會被 ignore)
    * 指定義 non`const` 版本的 class 不是 [`CopyConstructible`](https://en.cppreference.com/w/cpp/named_req/CopyConstructible)
        ```cpp
        class NonConstCopyCtor {
            NonConstCopyCtor(NoncostCopyCtor &obj) {}
        };
        
        int main() {
            const NonConstCopyCtor n1;
            NonConstCopyCtor n2 = n1; // error
        }
        ```
* 然後 copy ctor 會在一些情境下被 compiler implicit 使用
    * e.g.
    ```cpp
    `class_name obj2 = obj1;`
    ```
    * 這叫做 copy initialize，馬上會講到，這裡會 implicit 呼叫 copy ctor
    * **如果對 copy ctor 宣告成 `explicit` 則會噴 error**
        * 所以不應該對 copy cstr 用 `explicit`

#### The Synthesized Copy Constructor
* 如果我們沒定義，compiler 會幫我們合成一個
    * *不像 default ctor，就算我們有定義其他 ctor，只要沒有定義 copy ctor，compiler 就一定會幫我們定義*
    * *另外 Primer P.508 會說到，對於某些 class，compiler 生成的 copy ctor 反而會讓我們不能對這個 class 的 object 做 copy*
        * 未看先猜是因為 class 有 member 不能 copy，例如有 iostream object 之類的
    * 但如果沒有發生上面的情況的話，**compiler 合成的 copy ctor 就會 memberwise copies non`static` members 到新的 class object 內**
    * 如何 copy members?
    * 呼叫 member 的 copy ctor!

:::info
* synthesized default ctor 會呼叫所有 member 的 default ctor
* synthesized copy ctor 則會呼叫所有 member 的 copy ctor
:::

* 可是如果有 member 是 C array 呢? C array 理論上不是不能 copy?
    * array 的話，synthesized copy ctor 會把 element 一個一個 copy 到新的 class object 的 array 裡面
    * element 一樣呼叫 element type 的 copy ctor

* 用 `Sales_data` 來舉例，compiler 幫他生的 synthesized copy ctor 等價於這樣:
    ```cpp
    class Sales_data {
    public:
        // other members and constructors as before     
        // declaration equivalent to the synthesized copy constructor
        Sales_data(const Sales_data&);
    private:
        std::string bookNo;
        int units_sold = 0;
        double revenue = 0.0;
    };
    // equivalent to the copy constructor that would be synthesized for Sales_data 
    Sales_data::Sales_data(const Sales_data &orig):     
        bookNo(orig.bookNo), units_sold(orig.units_sold),
        revenue(orig.revenue)
        {   } // empty body
    ```
    * 注意，是用 constructor initializer list copy initialize 的
        * 只有在 ctor init list 階段才能初始化 member
        * 不知道上面在講啥的話要回去看第七章

#### Copy Initialization
* **我們現在終於可以好好討論 direct initialization 跟 copy initialization 的差異惹!**
    ```cpp
    string dots(10, '.'); // direct initialization
    string s(dots); // direct initialization
    string s2 = dots; // copy initialization
    string null_book = "9-999-99999-9"; // copy initialization
    string nines = string(100, '9'); // copy initialization
    ```
* direct initialization，其實就是告訴 compiler 用基本的 function matching(第六章) 來找對應的 constructor
    ```cpp
    string dot(10, '.');
    ```
    * 利用 function matching 找到 `std::string(size_t, char)`
    ```cpp
    string s(dots);
    ```
    * 利用 function matching 找到 `std::string(const string&)`
        * 剛好是 copy ctor
* 當我們用 `=` 來*初始化* object，我們是告訴(ask for) compiler 叫他用 `=` 的 rhs copy initialize lhs
    * 如果 initializer 要轉換的話就做轉換(implicit conversion)
    * 例如
    ```cpp
    string null_book = "9-999-99999-9";
    * string literal 會先轉成 `std::string`，`null_book` 在用這個 temp `std::string` 來 copy initialize
    ```
* 通常你在宣告 object 時用 `=` 都會呼叫 copy ctor
    * 不過 13.6 講了 move ctor，你會發現如果你有額外定義 move ctor 的話，在某些情況下就會呼叫 move ctor 而不是 copy ctor
* 總之**現在的重點是要知道什麼時候會呼叫 copy/move ctor**
    1. 宣告變數時用 `=`
    2. 把物件當 argument 傳到 *nonreference* parameter 時
    3. return object as *nonreference* type 時
    4. 你對一個 array 宣告時用 initializer list `{initializer...}`，則 array 每對應為致的 elements 會被 copy initialize
    5. 還有如果你定義的是 aggregate class(7.5.5, p.298)
        * 就是那個跟 C struct 87% 像的 class
        * 且也是用 `{initializer...}` 初始化，則 aggregate class 的 memeber 也會用list 內的 value 做 copy initialize
* 除了上面講的，container 的某些操作也是 copy initialize
    1. 例如初始化 container，elements 都是 copy initialize
    2. call `insert` 或 `push_back` 這類 member 時
    3. *而像 `emplace` 這種的就是對 element 做 **direct initialize** 了*

##### Parameters and Return Values
* 就跟上面的 2 3 點一樣，不過有個重點
* 這裡講為什麼 copy cstr 一定要接 reference
    * 要呼叫任何 function 時需要 pass class object 參數
    * 而 nonreference 參數是 passed by value
    * 所以需要呼叫 class object 的 copy ctor
    * **而，copy ctor 也是 function**
    * 這時如果 copy ctor 吃 nonreference type，則 class object 又是 passed by value，所以又要呼叫 copy ctor
    * GG

##### Constraints on Copy Initialization
* 就只是再說一次，宣告 `explicit` 的 copy ctor 就不能用 copy initialization(`=`)
    ```cpp
    vector<int> vec(10); //OK
    vector<int> vec = 10; // error
    ```
    * `vector<int>` 接收 `int` 的 ctor 是 `explicit`
    * `=` 右邊的 `10` 沒辦法隱性轉成 `vector` 再 copy 給 vec
    ```cpp
    void f(vector<int>); // f’s parameter is copy initialized
    f(10); // error: can’t use an explicit constructor to copy an argument
    f(vector<int>(10)); // ok: directly construct a temporary vector from an int
    ```
    * `f(10)` 會 implicit 呼叫 `std::vector(size_t)`, which is error，再將 temp vector 拿來 copy initailize f 的 parameter
* 其他傳參數或 return value 也是一樣


#### **The Compiler Can Bypass the Copy Constructor**
* https://www.youtube.com/watch?v=hA1WNtNyNbo
* compiler 可以把
    ```cpp
    string null_book = "9-999-99999-9"; // copy initialization
    ```
* 重寫成
    ```cpp
    string null_book("9-999-99999-9"); // compiler omits the copy constructor
    ```
* 請試著想想 `std::string` 的實作，再來看看這兩行 code 在效能上可能會有什麼差異
* compiler 可以做這層轉換，但要有以下前提才能做:
    * copy/move ctor 一定要存在而且是 `public`
    * 不然 semantic 可能會不一樣
        ```cpp
        ClassNoCopy c1;
        ClassNoCopy c2 = "blabla";
        ```
        * 假設 `ClassNoCopy` 可以用 C string 初始化，且沒有 copy ctor
        * 那如果 compiler 強制 bypass copy ctor，則原本 c2 語意上是 copy, which is illegal，現在變成呼叫 `c2("blablabla")`

### 13.1.2 The Copy-Assignment Operator
* class 也**控制 object 被 assign 時發生什麼事**
    ```cpp
    Sales_data trans, accum;
    trans = accum; // uses the Sales_dat acopy-assignment operator
    ```
    * 最後的 assignment 就會用到 copy-assignment operator
* 一樣，class 沒定義 compiler 就會合成一個

#### Introducing Overloaded Assignment
* 雖然 14 章才會講 operator overloading
* 反正你就是要定義 `operator=` 這個 member function 啦
* 對於 `operator*x*()` 這類 operator overloading member functions，parameter list 內的東西就是 operand
* 有些 `operator*x*` 一定要定義成 class 的 member function
    * 這時候 `operator` 的 lhs 會被 bound 成呼叫該 operator member function 的物件的 `this`，rhs 就是傳進 operator member function 的參數
* 總之 copy-assignment 會這樣定義:
    ```cpp
    class Foo {
    public:
        Foo& operator=(const Foo&); // assignment operator
        // ...
    };
    ```
* 因為按照 convention，class 的 operator 使用使用起來要跟 built-in type 的 operator 行為一致，**所以自定義的 `operator=` 也要回傳 lhs 的 reference**
* 而且 STL 也會要求，如果 class operator 有自定義的話，例如 `=`，他的某些行為真的必須跟 built-in 一樣，例如 `operator=` 要 return `lhs&`，不然會有很ㄎㄧㄤ的結果

:::info
* Best Practice:
    * Assignment operators ordinarily should return a reference to their left hand operand.
:::

#### The Synthesized Copy-Assignment Operator
* 你沒定義的話 compiler 會合成一個
* 在某些情況下，跟合成的 copy ctor 類似，compiler 合成的 copy-assginment operator 反而會讓你不能 assign
    * 大概又是你有 member 不能 assign 的時候吧

* 如果 compiler 合出來的可以 assign，copy-assignment operator 就會把所有的 `rhs` 的 non`static` members 用 member 自己的 copy-assignment operator assign 給 `lhs` 的 non`static` members

* 又來了，如果 member 是 array 呢?**那就把所有 elementw 一個一個 assign**
    * 有人用這種很ㄎㄧㄤ的方法把 array 封裝一層 class，這樣可以做 assign，不過感覺是糞 code

* 而合成的 copy-assignment operator 就會 return `lhs&`
* 一樣來看 `Sales_data` compiler 合成出來的 copy-assignment operator 長什麼樣子
    ```cpp
    // equivalent to the synthesized copy-assignment operator
    Sales_data&
    Sales_data::operator=(const Sales_data &rhs)
    {
        bookNo = rhs.bookNo; // calls the string::operator=
        units_sold = rhs.units_sold; // uses the built-in int assignment 
        revenue = rhs.revenue; // uses the built-in double assignment
        return *this; // return a reference to this object
    }
    ```
    * 總之就是呼叫每個 member 定義的 copy assignment operator

### 13.1.3 The Destructor
* 理論上你要在 dtor 釋放你用到的資源
* 除此之外 dtor 還會呼叫 member 的 dtor
    * 不用自己呼叫，member 要被破壞時就會自己呼叫了
* dtor 格式是這樣:
    ```cpp
    public:
        ~class_name();
    ```
* class dstr 而且不能吃參數，這意味著 dtor 不能 overloading 並且只有一個

#### What a Destructor Does
* Just as a constructor has an initialization part and a function body (§ 7.5.1, p. 288), **a destructor has a function body and a destruction part.**
    * ctor:
        * constructor initialize list + body
    * dtor 也有類似的東西
* 在 ctor，class member 會在 body 執行之前初始化完成，而且 member *會按照他們宣告在 class 內的順序依序初始化*
    * 一樣，忘記的話回去看第 7 章
* **而 dtor 則是先執行 body，執行之後 class members 會按照他們宣告的相反順序呼叫各自的 dtor**
    * 有點 LIFO 的感覺
* 上面有講 dtor 也有對應的 destruction part
    * 但其實沒有辦法直接寫出來，是 implicit 的，沒有辦法像 ctor init list 那樣寫出來
    * 而 member 怎麼被破壞 depends on 他們自己的 dtor
    * 其實不用寫出來也很合理，因為每個 class 的 dtor 就只能有一個，代表只有一種破壞方式，所以就不用寫說要怎麼破壞了；
    * ctor init list 則是可以指定要怎麼初始化 member，因為可以有很多種方式初始化，所以才需要 ctor init list 來明確指定要用哪種

* built-in type 沒有 dtor 這種東西，反正就只是說 built-in type 的那塊空間要歸還而已
    * **但是指標也是 built-in type，他要被破壞時並不會 delete 他指向的物件**

#### When a Destructor Is Called
* 什麼時候 class dtor 會被呼叫呢?
    * class object out of scope 時
    * 當一個 object 被破壞時，class members 也會呼叫自己的 dtor
    * 當 container，不管是 STL 還是 built-in array，被破壞時，他們的 elements 會呼叫 dtor
    * 動態配置的指標被 delete 時，其指向的物件會 call dtor
    * 當你寫了一個 expression，並創造多個 temp objectw 時，這個 full expression 算完之後會 call 這些 temp objects 的 dtor

* 注意看例子，註解會寫什麼時候會呼叫 dstr:
    ```cpp
    { // new scope
        // p and p2 point to dynamically allocated objects Sales_data
        *p = new Sales_data;
        // p is a built-in pointer
        auto p2 = make_shared<Sales_data>(); // p2 is a shared_ptr
        Sales_data item(*p); // copy constructor copies *p into item 
        vector<Sales_data> vec; // local object
        vec.push_back(*p2); // copies the object to which p2 points 
        delete p; // destructor called on the object pointed to by p
    }// exit local scope; destructor called on item, p2, and vec
     // destroying p2 decrements its use count; if the count goes to 0, the object is freed
     // destroying vec destroys the elements in vec
    ```
    * 我們只需要 `delete` 我們自己 allocate 的指標 `p` 就行了，`Sales_data` 的 dtor 會自動呼叫，class members 都會自動釋放資源
        * 例如底層會動態配置字串的 `std::string`
            * string 會自己管理資源，`Sales_data` 不用去管他

#### The Synthesized Destructor
* 沒定義的話 compiler 會合成一個
* 一樣，某些情況下 compiler 合成的反而會讓我們不能"破壞物件"
    * 超 WTF 的，Primer p.508 會講
* 如果沒發生上面說的情況，那合成出來的 dtor 就會是...
    * `~class_name() {}`
    * empty body!
    * 很合理阿，什麼事都不做
    * 其實不是，只是 member 的 destroy 會在 dtor 的 destruction part 被執行而已
    * *The members are automatically destroyed after the (empty) destructor body is run.*

### 13.1.4 The Rule of Three/Five
* 超級重要
* **通常**，當有定義這五種 member functions(包括我們還沒講的剩下兩個有關 move 的 operation)的其中一個的需要時，其他四個理論上也必須定義
* 不然你的 class 在 copy control 的行為就會是錯的
* Primer 說必須把這五種 member 想成是一個單位，必須同時都存在/不存在
* 以下分成幾種情境來講

#### 1. Classes That Need Destructors Need Copy and Assignment
* Often, the need for a destructor is more obvious than the need for the copy constructor or assignment operator.
* 之前練習題定義的 `HasPtr` 是個好例子
    * 他有用 `new` 一塊空間
    * 如果用合成的 dtor，就不會在破壞 `HasPtr` 物件時把那塊空間 `delete`
    * 所以要自訂 dtor，`delete` 那塊空間

* 通常如果是在上面這種情境下定義了 dtor，也意味著必須定義 copy ctor 跟 copy assign
* 如果只定義了 dtor 而沒定義 copy ctor 跟 copy assign:
    ```cpp
    class HasPtr { 
    public:
        HasPtr(const std::string &s = std::string()): 
            ps(new std::string(s)), i(0) { }
        ~HasPtr() { delete ps; } // WRONG: HasPtr needs a copy constructor and copy-assignment operator 
        // other members as before
    };
    ```
* 呼叫以下 function 就會 UB
    ```cpp
    HasPtr f(HasPtr hp) // HasPtr passed by value, so it is copied 
    {
        HasPtr ret = hp; // copies the given HasPtr
        // process ret 
        return ret;
    }
    ```
    * 執行 argument passing(copy) 時，`hp.ps` 就會跟 `argument.ps` 指向同一塊記憶體
    * 在執行 `ret = hp` 時，`ret.ps` 跟 `hp.ps` 也會指向同一塊記憶體
        * 因為合成的 copy ctor 就是做 memberwise copy，所以兩個 `HasPtr` 物件內的指標 value 會一樣
    * 當 `f` 結束後，`ret` 跟 `hp` 都會 call dtor，所以會把 `ret.ps` 指向的記憶體 `delete`，也把 `hp.ps` `delete`
        * **這樣就 double free!!*
    * 這時就已經 UB 了，而且 argument.ps 指向一塊 free 的空間，變成 dangling ptr

#### 2. Classes That Need Copy Need Assignment, and Vice Versa
* 1. 還意味著另一件事情:
    * 如果一個 class 必須自訂 dtor，*幾乎可以肯定他還要定義 copy cstr/assignment*；
    * 但是一個 class 需要定義 copy ctor 或 copy assignment，*不一定需要定義 dtor*

* 舉一個最簡單的例子就是，class 內有 unique ID 的概念
    * 每個物件裡面都會有一個 unique ID，每個物件的 ID 的 value 要不一樣
    * 這樣就需要定義 copy/ ctor/ copy assignment，否則 ID 就直接 copy
    * 但不一定要 dtor
    * 如果這個 ID 不需要動態配置資源，且其他 member 也不需要，那其實就不用定義 dstr
* 所以總結第二個 rule:
    * class 要定義 copy cstr **幾乎 iff** class 要定義 copy assignment；不過定義這兩個 members 不一定要定義 dtor

### 13.1.5 Using `= default;`
* 可以用 `= default` 的語法明確告訴 compiler 幫我合成一個 copy control member
    ```cpp
    class Sales_data {
    public:
    // copy control; use defaults 
    Sales_data() = default;
    Sales_data(const Sales_data&) = default; 
    Sales_data& operator=(const Sales_data &); 
    ~Sales_data() = default;
    // other members as before
    };
    Sales_data& Sales_data::operator=(const Sales_data&) = default;
    ```
* 把 `= default` 寫在 class definition 內，或者 class definition 外面的差別，就是一個會 implicit `inline` 一個不會
    * 為何會 `inline` 是為了滿足 One Definition Rule
    * https://stackoverflow.com/questions/9734175/why-are-class-member-functions-inlined?answertab=votes#tab-top
    * 忘記去看第七章
* Note: 只能對 copy control 那些 members 使用 `= default`

### 13.1.6 Preventing Copies
:::info
* Best Practice:
    * Most classes should define—either implicitly or explicitly—the default and copy constructors and the copy-assignment operator.
:::

* 如果對於你的 class 來說，複製物件這個操作完全不合理，所以你不希望 user copy copy/assign object，**你必須明確定義他們為 `= delete`
    * 注意，不定義 copy/assignment 無法達成目的
    * 因為不定義他們，compiler 就會合成一個
        * 有時候 compiler 合成的 copy/assignment 的確會讓物件無法 copy/assign，之後會講

#### Defining a Function as Deleted
* 在 C++11，我們可以把 copy ctor 跟 copy assignment 定義成 `delete` functions 來防止 class object 被 copy/assign
    * A `delete`d function is one that is **declared but may not be used in any other way.**
    ```cpp
    struct NoCopy { 
        NoCopy() = default; // use the synthesized default constructor
        NoCopy(const NoCopy&) = delete;  // no copy
        NoCopy &operator=(const NoCopy&) = delete; // no assignment
        ~NoCopy() = default; // use the synthesized     destructor
        // other members
    };
    ```
* `= default` 跟 `= default` 的使用時機有些不同
    * 一定要在 class definition 內第一次宣告 member function 時就使用 `= delete`，不能在 class definition 外才使用
        * 這種差可以這樣理解
        * 因為 `= default` 是告訴 compiler 怎麼幫我 gen code，寫在 class 外定義也可以；
        * 而 `= delete` 是在 compiler 還在檢查 source code 時就必須知道的事情，因為他才能檢查 source code 有沒有不小心使用這個 member function
    * 另外 `= delete` 可以用在任何 member functions，不像 `= defalut` 只能用在 copy control member

        * 但是使用時機，Primer 很輕描淡寫的說可以用在一些 function matching 的場合，來導引(guide to)怎麼 match...?
        * 怕，感覺這種 code 就 code smell

#### The Destructor Should Not be a Deleted Member
* `= delete` 可用在任何 member function，當然包括 dtor
* 但最好不要用在 `dtor`...
* 一旦用了，compiler 就不准 user code 定義這個 class 的 object，而且*也不准定義有這種 member 的 class 的 object*，因為一旦 object out of scope，就會呼叫 dtor，但 dtor 無法使用
* 但是你可以動態配置(new)這種 class 的 object，只是不能 `delete` 指標，因為 `delete` 會呼叫 dtor
    * 也不能用 smart pointer 指
    ```cpp
    struct NoDtor {
        NoDtor() = default; // use the synthesized default constructor
        ~NoDtor() = delete; // we can’t destroy objects of type NoDtor
    };
    NoDtor nd; // error: NoDtordestructor is deleted 
    NoDtor *p = new NoDtor(); // ok: but we can’t delete p
    delete p; // error: NoDtor destructor is deleted
    ```
#### The *Copy-Control Members* May Be Synthesized as `delete`d
* For some classes, the compiler defines these synthesized members as deleted functions:
    * dtor 會合成成 delete:
        * 如果有 class member 的 dtor 宣告成 `delete`，或者 member 的 dtor 不能 access(例如 access specifier 是 `private`)
    * cpoy cstr 會合成成 delete:
        * 如果有 clas member 的 copy ctor 宣告成 `delete` 或不能 access；
        * copy ctor 在有 class member 的 dtor 宣告成 `delete` 或者不能 access 時*
    * copy assignment 會合成成 delete:
        * 如果有 class member 的 copy assignment 是 `delete` 或不能 access
        * 或者 class 有 `const` 或 `reference` member
    * default ctor 會合成成 delete:
        * 如果有 class member 的 default ctor 是 `delete` 或不能 access
        * 或者有 class member 是 `reference`，可是沒有給 in-class initializer
        * 或者有一個 member 是 const type，而且他的 type 沒有明確定義 default ctor，那個 member 也沒有 in-class initializer
* 其實 copy assignment 在 reference 的 case，memberwise assign 是合法的
    * 可是你仔細想想就會發現這個行為會跟預期結果不符合，那乾脆不要合成

* 看起來很細規則很多
* 但背後的核心概念是，**如果一個 class 有 class member 不能被 default constructed, copied，assigned, 或 destroyed**，那 該 class 對應的 copy-control 就會被合成為 `delete`

* 另外還有幾個例子會造成 copy-control 被合成為 `delete`
    * 13.6.2(p. 539)
    * 15.7.2(p. 624)
    * 19.6(p. 849)

#### `private` Copy Control

* pree C++11 之前沒有 `= default;`，`= delete`，可以用，怎麼做到等價行為的?
    * 把 copy-control members 宣告成 `private`!

* 先看 copy 的例子
    ```cpp
    class PrivateCopy {
        // no access specifier; following members are private by default; see § 7.2 (p. 268)
        // copy control is private and so is inaccessible to ordinary user code
        PrivateCopy(const PrivateCopy&);
        PrivateCopy &operator=(const PrivateCopy&);
        // other members
        public:
        PrivateCopy() = default; // use the synthesized     
        default constructor ~PrivateCopy();
        // users can 
        define objects of this type but not copy them
    }
    ```
* 這樣 user code 就不能 copy `PrivateCopy` 了
* **可是 class `friend` 還是可以**
* 如果要避免 `friend` copy，以前的做法就是**宣告 copy-control，但是不要定義**
* 但是這種做法要注意以下情境:
    * non`friend` user code 使用 copy-control 是 **compile time error**
    * 而 friend 使用 copy-control 則會是 **link time error**
* 總之 C++11 請用 `= delete;`

## 13.2 Copy Control and Resource Management
* Ordinarily, classes that manage resources that do not reside in the class must define the copy-control members.
* In order to define these members, **we first have to decide what copying an object of our type will mean.**
    * 這種自己管理 resource 的 class 需要由實作者自己決定到底 copy-control 代表什麼意思
* Primer 說可以把 class 分成兩大類:
    * behave like a value or like a pointer
* 如果 class 是 valuelike，那 object 複製之後，兩個 objects 各自獨立，改變其中一個的狀態不會影響到另一個
    * 可以想成 class object 自己掌控自己的 states
    * 例子: `std::string`，STL container
* 如果是 pointer-like，那 object 複製之後，兩個 objects 會以某種形式共享部分的 data
    * 或者說改變其中一個 object 也會改變另一個
    * 可想像成這種 class 的 object 複製之後會*指向*同一塊 underlying data
    * 例子: `std::shared_ptr`，`StrBlob`

* 另外，如果一個 class 不允許 copy 那就沒有必要討論他們是 valuelike 還是 pointer-like 了，因為這兩個本來就是建立在 copy 時的行為定義出來的名詞
    * 例如 `std::iostream` 或 `std::unique_ptr<T>` 就是

* 為了介紹這兩種形式的 class 的 copy-control 怎麼寫，Primer 會把練習時用的 `HasPtr` 用兩種方式重新實作
    * 還記得他有一個 `int` 跟 `std::string` member
        * 題外話，那個 `int` 在解說時沒什麼用
    * member 如果是 built-in type(除了指標)，則 copy 通常都是直接用 assign，因為這些 built-in type 本來就有 "value"，就跟 valuelike 行為一致
    * **真正決定我們定義的 class 為 valuelike 還是 pointerlike 取決於我們怎麼操作 class pointer-type member**

### 13.2.1 Classes That Act Like Values
* 反正就是不能跟別的 objects 共用 resources 啦(除了 static member)
* To implement valuelike behavior `HasPtr` needs
    * A copy constructor that *copies* the `string`, *not just the pointer*
    * A destructor to free the `string`
    * A copy-assignment operator to *free the object’s existing string* and copy the string from its right-hand operand

    ```cpp
    class HasPtr {
    public:
        HasPtr(const std::string &s = std::string()): 
            ps(new std::string(s)), i(0) { }
        // each HasPtr has its own copy of the string to which ps points
        HasPtr(const HasPtr &p): ps(new std::string(*p.ps)), i(p.i) { }
        HasPtr& operator=(const HasPtr &);
        ~HasPtr() { delete ps; }
    private:
        std::string *ps; int i;
    };
    ```
    * 自己看啦很好懂

#### Valuelike Copy-Assignment Operator
* Assignment operators typically **combine the actions of the destructor and the copy constructor.**
    * 這基本上是一種 pattern 了，valuelike class 的 `operator=` = dtor + copy ctor
    * Like the destructor, assignment destroys the left-hand operand’s resources.
    * Like the copy constructor, assignment copies data from the right-hand operand.
    * 把 object 自身該丟掉的丟掉，並且複製 rhs 的 data 到 lhs

* However, **it is crucially important that these actions be done in a sequence that is correct even if an object is assigned to itself.**
    * 你要確定 user code 寫出 `obj = obj;` 時 `operator=(rhs)` 也要運作正確

* Moreover, **when possible, we should also write our assignment operators so that they will leave the left-hand operand in a sensible state should an exception occur.**
    * 我們要確保就算 exception 在 assign 時發生，lhs 也要能維持在一個合理的 state。
    ```cpp
    HasPtr& HasPtr::operator=(const HasPtr &rhs) {
        auto newp = new string(*rhs.ps); // copy the underlying string 
        delete ps; // free the old memory
        ps = newp; // copy data from rhs into this object
        i = rhs.i;
        return *this; // return this object
    }
    ```

* 上面的寫法遵循一個 pattern:
    * **先做 copy constructor 做的事，做好之後再做 destructor 做的事**
    * 因為 dtor 幾乎不會噴 exception，可是 copy ctor 會
    * 如果先做 dtor 再做 copy ctor，而 copy ctor 噴 exception，那 object 可能就會變成不合理的狀態

* 來 demo 一個錯的 `operator=()`
    ```cpp
    // WRONG way to write an assignment operator!     
    HasPtr&
    HasPtr::operator=(const HasPtr &rhs) {
        delete ps; // frees the stringto which this object points
        // if rhs and *this are the same object, we’re copying from deleted memory!
        ps = new string(*(rhs.ps));
        i = rhs.i;
        return *this;
    }
    ```
    * 如果 self assignment 就會爆炸
    * 內部的 pointer 變成 dangling pointer 了
    * 如果 new 的時候 `throw std::bad_alloc`，則 `lhs` 的 pointer 也會是 dangling pointer

* 總之必須寫出一個 self assignment 也會正確的 code
    * 最基本的做法是檢查 `this != &rhs`
    * 另一個則是像 Primer 寫的，先把 rhs 的 data 都複製到 temp，再刪除 lhs 的 data，再把 temp 丟給 lhs，這樣就不用去管是否 self assignment，更 uniform

### 13.2.2 Defining Classes That Act Like Pointers
* 讓 `HasPtr` 看起來像指標
* 還是得定義 destructor
* 這時會發現，pointerlike object 內部如果用 raw pointer 的話 copy control 就會很難寫，
    * 例如 dtor，你如果直接 `delete` 該 pointer，會導致其他 object 有 dangling pointer
* 最簡單的解法就是用 12章的 `shared_ptr`

* 但總是會有不能用 `shared_ptr` 的時候
    * 這時就必須自行實作 reference count 的機制

#### Reference Counts
* create pointerlike 物件時必須要初始化內部的 ref count
    * ctor 內要實作
* 除了 copy ctor，其他 ctors 都要直接 create 一個 counter，這個 counter maintain 有多少 object 跟我們 share states
    * 初始化時設成 1，因為這些物件都是獨立被初始化的，沒有其他物件跟他們 shrare states
* **而 copy ctor 則是把他複製的物件的 counter++**
* dtor 當然就 counter--
    * 如果 counter == 0 就 delete
* copy assignment 則是 rhs 的 counter++，lhs 的 counter--
    * 如果 lhs 的 counter == 0 就 delete
    * 然後再將 lhs 的 counter 同步成 rhs 的 counter


* 講那麼多可是還沒講要怎麼存 object 內的 reference count
* 首先不能是 `HasPtr` 的 member
    ```cpp
    HasPtr p1("Hiya!"); 
    HasPtr p2(p1); // p1 and p2 point to the same string
    HasPtr p3(p1); // p1, p2, and p3 all point to the same string
    ```
    * 生成 `p3` 的時候有辦法改 `p1`，可是改不到 `p2`，但是 `p1` 跟 `p2` 都指向同一個 string!
        * 這樣這些物件的 counter 就不一致
* 解法: 可以連 counter 都是動態配置的，然後 copy object 時直接 copy 指像 counter 指標，這樣就會不同 objects 就會指向相同 counter
    * 詳情 back to basics smart pointer
    * `shared_ptr` 就是這樣做的

#### Defining a Reference-Counted Class
* `HasPtr` 可以這樣寫:
    ```cpp
    class HasPtr {
    public: 
        // constructor allocates a new string and a new counter, which it sets to 1
        HasPtr(const std::string &s = std::string()): 
            ps(new std::string(s)), i(0), use(new std::size_t(1)) {}
        // copy constructor copies all three data members and increments the counter
        HasPtr(const HasPtr &p): ps(p.ps), i(p.i), use(p.use)
            { ++*use; }
        HasPtr& operator=(const HasPtr&); ~HasPtr();
    private:
        std::string *ps;
        int i;
        std::size_t *use;
        // member to keep track of how many objects share *ps
    };
    ```
    * 多了一個 `use` 指標，default ctor 會 `new` 一個 `size_t` 給他指，初始化為 `1`，**代表有一個 object shared 指向的 `std::string`**

#### Pointerlike Copy Members "Fiddle"(撥弄) the Reference Count

* `HasPtr` copy 時內部要指向 rhs object 的 string，並且增加 rhs object 對應的 counter，同時也指向該 counter

* dtor:
    ```cpp
    HasPtr::~HasPtr() {
        if (--*use == 0) {
        // if the reference count goes to 0 
        delete ps; // delete the string
        delete use; // and the counter
        }
    }
    ```
    * `--use == 0` 時才能刪除 `std::string` 跟 `use` counter

* copy assignment:
    * 一樣是做 copy ctor 跟 dtor 的事情
    * 亦即，增加要 rhs 的 counter，然後 lhs counter - 1，counter 為 0 就刪除 lhs allocate 的 resource
    * 一樣要處理 self-assignment 的情況
        * 先做 copy ctor 再做 dtor
        * **這時候 `++` 跟 `--` counter 都會是對同一個 counter**
        * **可是必須先 `++` 再 `--`，如果倒過來可能會遇到 `counter == 0` 然後就把物件刪除了；總之就是要先做 copy ctor 再做 dtor
    ```cpp
    HasPtr& HasPtr::operator=(const HasPtr &rhs)
    {
        ++*rhs.use; // increment the use count of the right-hand operand
        if (--*use == 0) { // then decrement this object’s counter
            delete ps;    // if no other users
            delete use; // free this object's allocated members
        }
        ps = rhs.ps; // copy data from rhs into this object
        i = rhs.i;
        use = rhs.use;
        return *this;
    }
    ```
* 題外話，13.2 的 exercise 的 `TreeNode` 硬要寫成 pointer like 整個很奇怪...


## 13.3 Swap
* https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom/3279550#3279550
    * 參考
* 9.2.5 有初步介紹，當時只是說如果 swap container，原本指向 container 的 iterator 不會失效(string 除外)
* 這裡進一步說明，**通常需要自定義 copy-control 的 class 也會定義一個 `swap` function**
* 自定義的 swap function 在一些情境下會被使用:
    * 舉例，STL algorithms 在對一個 range 做 reorder 時，會呼叫 element type 的 `swap`
* 但如果你沒有自定義專屬這種 element type 的 swap，STL 就會用 std::swap function template 生出一個這種 class 的版本
    * 這個版本大致上如下
    ```cpp
    void swap(HasPtr &v1, HasPtr &v2) {
        HasPtr temp = v1; // make a temporary copy of the value of v1
        v1 = v2; // assign the value of v2 to v1
        v2 = temp; // assign the saved value ofv1to v2
    }
    ```
* 這個寫法其實就是單純使用 element type 的 copy ctor 以及 copy assignment 來完成的
    * 只要這兩個 copy control 有實作正確，為該 element type 而生的 `std::swap` 就不會有問題
* **但實際上這些 copy 完全不需要**
    * 嚴格上來說只要 swap 兩個 `HasPtr` 物件內部的 `string*` 指標即可
    ```cpp
    string *temp = v1.ps; // make a temporary copy of the pointer in v1.ps
    v1.ps = v2.ps; // assign the pointer in v2.ps to v1.ps
    v2.ps = temp; // assign the saved pointer in v1.ps to v2.ps
    ```
:::info
* 觀念注意: 如果有自定義 copy-control，那其實 swap 完全沒必要額外定義，因為 `std::swap` 會使用你定義的 copy-control(copy/move ctor 跟 copy/move assignment，其中 move 之後會講到)來交換 object。
    * 只要 copy control 實作正確則生出來的 `swap` 行為也會是正確的
* 自定義 class 專屬的 `swap` 其實是為了優化。
:::
    
#### Writing Our Own `swap` Function
* We can override the default behavior of `swap` by defining a version of `swap` that operates on our class.
    ```cpp
    class HasPtr {
        friend void swap(HasPtr&, HasPtr&);
        // other members as in § 13.2.1 (p. 511)
    };
    inline void swap(HasPtr &lhs, HasPtr &rhs) {
        using std::swap;
        swap(lhs.ps, rhs.ps); // swap the pointers, not the string data
        swap(lhs.i, rhs.i); // swap the int members
    }
    ```
    * `swap` 是 global function，要宣告成 `HasPtr` 的 `friend`
    * 另外 `swap` 內有一行 `using std::swap;`，這是我們第一次在 function 內使用這種 `using` 語法，等等會講這在幹嘛
        * 還有該死的超後面的 18 章才會真正講他的原李

* `swap` 本質上是要優化 class object 的 swap 行為，一不做二不休用 `inline`
* 具體時做很單純，就是將內部 member 做交換
    * 而 `std::string*` 就是做指標交換，而不是交換各自指向的 `std::string` 物件

#### ~~`swap` Functions Should Call `swap`, Not `std::swap`~~
#### **`friend` `swap` functions should write `swap(a, b);` inside function body, not `std::swap(a, b);`"**
* 個人覺得這句標題實在是太爛了，有誤導之嫌，所以重寫
* 注意，以下講的東西只有告訴你應該要這樣寫才會有預期行為，而並沒有告訴你為什麼，詳細原因要等到 16.3 講 template 時跟 18.2.3 講 ADL 時才會說明
* 如果要 call 對某個 class 自定義的 swap 要這樣寫:

* 這裡建議看懂
    * https://stackoverflow.com/questions/28130671/how-does-using-stdswap-enable-adl
    * https://stackoverflow.com/questions/4782692/what-does-using-stdswap-inside-the-body-of-a-class-method-implementation-mea/35277325
    * 還有 copy swap idiom 那篇再來看


#### Using swap in Assignment Operators
* 自定義 copy-control 的 class 也要定義 swap 的原因如下:
    * **可用 `swap` 來定義 copy assignment operator**
    * These operators use a technique known as **copy and swap**.
    * This technique **swaps the lefthand operand with a copy of the right-hand operand:**
    ```cpp
    // note rhs is ****passed by value****, which means the HasPtr copy constructor 
    // copies the string in the right-hand operand into rhs
    HasPtr& HasPtr::operator=(HasPtr rhs) {
        // swap the contents of the left-hand operand with the local variable rhs
        swap(*this, rhs); // rhs now points to the memory this object had used
        return *this; // rhs is destroyed, which deletes the pointer in rhs
    }
    ```
    * **注意參數是 passed by value**
        * 先把 right hand operand (用 class 的 copy ctor)複製一份(也就是 "copy and swap" 的 "copy")
        * 然後再把 left hand operand 跟這個 temp `rhs` object 做 `swap`
        * 最後讓 (local 的) `rhs` temp object dtor 把 `rhs` 破壞掉
            * 而這時 `rhs` 內部資料是原來屬於 `lhs` 的資料

* 用這個 copy and swap 的好處是
    1. **他已經處理了 self-assignment 的情況**
    2. 這樣還是維持著**先 copy 再 delete 的順序**，**Strong exception safety**
        * 唯一可能會噴 exception 的地方就是建立 `rhs` 時呼叫的 copy ctor
            * 如果 throw，則 lhs 也是維持原本的狀態

## 13.4 A Copy-Control Example
* 並不是只有要管理資源的 class 需要 copy-control
    * 你要寫 log，bookkeeping 某些事件，etc，各種理由可能都要自訂 copy-control
    * 只是定義出來不一定會管理資源而已

* Primer 定義一兩個需要做 bookkeeping 的 class 當作例子(我覺得這個例子的實用性很低，跟現實好像也有點差距)
    * `Message` 跟 `Folder`
    * 來表示 email 跟儲存 email 的資料夾
* Each `Message` can appear in multiple `Folder`s. However, **there will be only one (underlying) copy of the contents of any given Message**.
* 為了表示上面的結構，每個 `Message` 會有一個 `std::set<Folder *>` 存著 pointers to `Folder`
    * 代表這個 Message 出現在那些 `Folder`s
* 而每個 `Folder` 也會有一個 `std::set<Message *>` 存著 pointers to `Message`
    * 代表這個 `Folder` 存的所有 `Message`s
* ![](https://i.imgur.com/QIi3K0Y.png)
    * 題外話，我覺得這兩個 class 宣告成 `const` 會很奇怪
* `Message` API:
    * `save(Folder&)`: 把一個 `Message` 存到一個 `Folder`
    * `remove(Folder&)` : 把一個 `Message` 從一個 `Folder` 移除
    * default ctor: 創造 `Message` 的內容，但不存到任何 `Folder`
    * copy ctor: 複製 `Message` 的內容，原物件跟新物件是"不同 object"，但是原物件存在的 `Folder`s 新物件都要出現
        * 必須 copy `std::set<Folder*>`
        * 每個對應的 `Folder` 內的 `std::set<Message*>` 也要新增一個 `Message*`，指向這個新 `Message` 物件，代表該 `Folder` 內有這個新 `Message`
    * dtor: 將 `Message` 從對應 `Folder` 中移除
    * copy assignment: contents of `lhs` are replaced by `rhs`；原本包含舊內容的 `Folder`s 要刪除 `lhs`，然後包含 `rhs` 的 `Folder`s 要新增 `lhs`

* 仔細觀察會發現
    * dtor 跟 copy assignment 都要把 `Message` 從 `Folder` 移除
    * copy ctor 跟 copy assignment 都要從 `Folder` 新增 `Message`
* 所以定義兩個 private utilities 給他們用
:::info
Best Practice: 通常一個 class 的 copy assignment 做的事情會跟 copy cstr && dtor 重複到
這時候通常會 refactor 成一個 private unitilty 給他們共用
:::

* `Folder` 也有類似的事情要做，新增或移除 `Folder` 時也要把對應 `Message` 的 `set<Folder*>` 移除指標
* `Folder` 會當成練習，不過這裡假設 `Folder` 有定義 `addMsg(Message *)` 跟 `remMsg(Message *)` 可以用
    ```cpp
    class Message {
        friend class Folder;
    public:
        // folders is implicitly initialized to the empty set
        explicit Message(const std::string &str = ""): contents(str) { }
        // copy control to manage pointers to this Message
        Message(const Message&); // copy constructor
        ~Message(); // destructor
        Message& operator=(const Message&); // copy assignment 
        // add/remove this Message from the specified Folder’s set of messages
        void save(Folder&);
        void remove(Folder&);
    private:
        std::string contents; // actual message text
        std::set<Folder*> folders; // Folders that have this Message
        // utility functions used by copy constructor, assignment, and destructor
        // add this Message to the Folders that point to the parameter
        void add_to_Folders(const Message&);
        // remove this Message from every Folder in folders 
        void remove_from_Folders();
    };
    ```
# The `save` and `remove` Members
* `Message` 除了 copy-control 之外還有這兩個 `public` member function
    ```cpp
    void Message::save(Folder &f) {
        folders.insert(&f); // add the given Folder to our list of Folders
        f.addMsg(this); // add this Message to f’s set of Messages
    }
    void Message::remove(Folder &f) {
        folders.erase(&f); // take the given Folder out of our list of Folders
        f.remMsg(this); // remove this Message to f’s set ofMessages
    }
    ```
    * When we `save` a `Message`(to a `Folder`), we store a pointer to the given `Folder`; when we `remove` a `Message`, we `remove` that pointer.
    * `Folder` (利用 `addMsg`/`remMsg`) 自己決定要怎麼更新他的 `std::set<Message*>`，`Message` 不用管

#### Copy Control for the `Message` Class
* 之前說過我們把 copy 跟 dtor 以及 copy assignment 重複做的事情拆成 private utility
    * `add_to_Folders(const Message &m)`
    * `remove_from_Folders()`:
    ```cpp
    // add this Message to Folders that point to m 
    void Message::add_to_Folders(const Message &m) {
        for (auto f : m.folders) // for each Folder that holds m
            f->addMsg(this); // add a pointer to this Message to that Folder
    }
    ```
    * 之前說過要被 copy 的 `Message` 的 `set<Folder*>` 都要新增該 `Message` object，上面的 range for 就是在幹這件事
* 有了上面的 utility，copy ctor 就可以這樣寫:
    ```cpp
    Message::Message(const Message &m): contents(m.contents), folders(m.folders) {
        add_to_Folders(m); // add this Messageto the Foldersthat point to m
    }
    ```
    * `contents` 直接用 ctor_init_list 初始化
    * 而新增該 `Message` 到舊 `Message` 物件的對應 `Folder`s 則用 `add_to_Folders` 解決

#### The Message Destructor
* 一樣定義一個 dotr 還有 copy assignment 共用的 utility:
    ```cpp
    void Message::remove_from_Folders() {
        for (auto f : folders)  // for each pointer in folders
            f->remMsg(this); // remove this Message from that Folder
        folders.clear();
        // no Folder points to this Message
    }
    ```
* dtor 就可以這樣寫:
    ```cpp
    Message::~Message() {
        remove_from_Folders();
    }
    ```

#### Message Copy-Assignment Operator
* 接下來就是用那兩個 utility 組出 copy assignment
    * 一樣，要處理 self-assignment，還要 exception safety
## 注意，這裡 Primer 裡面寫的 `operator=()` 有 bug
* https://stackoverflow.com/questions/29308115/protection-again-self-assignment
* Primer 定義的 `operator=()` 是吃 const Message&，這樣在 self-assignment 時，先呼叫 `remove_from_Folders` 就會把資料都刪光了
* 這個 case 應該是無可避免一定要檢查 `this != &rhs`
    * 或者 `remove_from_Folders()` 不要呼叫 `set::clear()`，改由 caller 端自己處理
    ```cpp
    Message& Message::operator=(const Message &rhs) {
        if (this != &rhs) { // self-assigment check is necessary
                            // while remove_from_Folders() do folders.clear()
            remove_from_Folders();    // update existing Folders
            contents = rhs.contents;  // copy message contents from rhs
            folders = rhs.folders;    // copy Folder pointers from rhs
            add_to_Folders(rhs);      // add this Message to those Folders
        } 
        return *this;
    }
    ```

#### A `swap` Function for Message
* `Message` class 內的 member 是 `std::string` 跟 `std::set`
    * 都有專屬的 `swap` template specialization
    * 這樣的話我們自定義 `Message` 的 `swap`，然後在內部呼叫 member 的 `std::string` 跟 `std::set` 的 `swap`，效能會比較好
* 只是 `Message` 的 swap 還要處理一些內部指標的狀態，直接看 code 比較好理解
    ```cpp
    void swap(Message &lhs, Message &rhs) {
        using std::swap; // not strictly needed in this case, but good habit
        // remove pointers to each Message from their (original) respective Folders
        for (auto f: lhs.folders)
            f->remMsg(&lhs);
        for (auto f: rhs.folders)
            f->remMsg(&rhs);
        // swap the contents and Folder pointer sets 
        swap(lhs.folders, rhs.folders); // uses swap(set&, set&)
        swap(lhs.contents, rhs.contents); // swap(string&, string&)
        // add pointers to each Message to their (new) respective Folders
        for (auto f: lhs.folders) 
            f->addMsg(&lhs);
        for (auto f: rhs.folders)
            f->addMsg(&rhs);
    }
    ```
    1. 先將 `lhs` 跟 `rhs` 各自存在的 `Folder`s 內指向他們的指標一除，代表暫時將 `Message`s 從 `Folder`s 移除
    2. 這時再 swap `Message` 內容(`contents`) 跟原本存在的資料夾列表(`folders`)
    3. 最後再把各自更新的資料夾列表新增 `lhs`/`rhs` 各自的 `Message` 指標，代表新增 `lhs`/`rhs` 到 `Folder`s 中

## 13.5 Classes That Manage Dynamic Memory
* 再講一次，如果一定要實作 class 自己分配 resource，一定要自訂 copy-control
    * 也再講一次，這種情境大部分都可用 STL 解決，不要造輪子
* Primer 舉例，定義一個像是 `vector` 的東西
    * non template
    * element 為 `std::string`，叫做 `StrVec`

#### `StrVec` Class Design
* 跟 `std::vector` 一樣，當空間滿了時，會直接要一個更大的空間，然後把 原空間的 elements copy 過去。
* 但是如果直接 `new []`，新的空間的後半部份都會 default initialize，很沒效率
    * **所以會用 `std::allocator`**
* 新增 element 時，用 `std::allocator::construct`
* 刪除 element 時，用 `std::allocator::destroy`
    * `std::allocator` 不清楚請去看 12.2.2
* 另位會有三個 `std::string*` 指標來維持當前配置的動態陣列的狀態:
* ![](https://i.imgur.com/FHoRJRK.png)
    * `elements`, which **points to the first element** in the allocated memory
    * `first_free`, which **points just after the last actual element**
    * `cap`, which **points just past the end of the allocated memory**
* 除此之外還會有一個 `static` data member `alloc`，他是 `std::allocator<string>`
* 我們還會定義四個 private utilities:
    * `alloc_n_copy(const string *b, const string*e)`: 配置一個跟 range(b, e) 一樣大空間，並複製該 range 的 elements 到新的空間
    * `free()`: 把 `StrVec` 配置的空間歸還
    * `chk_n_alloc()`: 這個 function 先確認當前配置空間是否可以再塞一個 element，如果空間不夠，`chk_n_alloc` 會呼叫 `reallocate` 配置新空間
        * 題外話這裡的邏輯跟 function name 不符，應該叫做 `checkOrRealloc` 之類的
    * `reallocate`: 重新配置一個比當前更大的空間
    
#### `StrVec` Class Definition
* 來定義 `StrVec` 的 interface 吧
    ```cpp
    // simplified implementation of the memory allocation strategy for a vector-like class
    class StrVec {
    public:
        StrVec(): // the allocator member is default initialized 
        elements(nullptr), first_free(nullptr), cap(nullptr) { }
        StrVec(const StrVec&); // copy constructor     
        StrVec &operator=(const StrVec&); // copy assignment
        ~StrVec(); // destructor
        void push_back(const std::string&); // copy the element
        size_t size() const { return first_free - elements; }
        size_t capacity() const { return cap - elements; }
        std::string *begin() const { return elements; }
        std::string *end() const { return first_free; }
        // ...
    private:
        static std::allocator<std::string> alloc; // allocates the elements
        void chk_n_alloc() // used by functions that add elements to a StrVec 
            { if (size() == capacity()) reallocate(); }
        // utilities used by the copy constructor, assignment operator, and destructor 
        std::pair<std::string*, std::string*> alloc_n_copy (const std::string*, const std::string*);
        void free(); // destroy the elements and free the space 
        void reallocate(); // get more space and copy the existing elements
        std::string *elements; // pointer to the first element in the array
        std::string *first_free; // pointer to the first free element in the array
        std::string *cap; // pointer to one past the end of the array
    };
    
    // allocmust be ***defined in the StrVec implmentation file***
    allocator<string> StrVec::alloc;
    ```
    * `std::vector` 有的東西 `StrVec` 也有
        * default ctor
        * copy control
        * begin/end
        * push_back
        * size/capacity
    * *再提醒一次，non`const` `static` member 一定要在 class definition 外再定義一次*
        * 並且定義在 implementation file，也就是 .cpp 檔
        * 如果定義在 header 內，那多個 include 這個 header 的 TU 就會 link error
    * default ctor 讓內部維持狀態的 `std::string*` 指標都設為 `nullptr`，代表沒有 elements
    * `size` 回傳真正擁有的 elements 數量
    * `capacity` 回傳目前空間可存放的 elements 數量
    * `chk_n_alloc` 會確認當前空間可再塞入一個 element，否則重配置空間
    * `begin` 跟 `end` 回傳 `elements` 跟 `first_free`，意味著第一個 element 跟最後一個 element 之後的位置

#### Using `std::allocator::construct`
* `StrVec` 的 `push_back` 實際上會先叫 `chk_n_alloc();` 確保可以直接塞 element，呼叫完之後直接在 `first_free` 指向的位置 `construct` 新的 element:
    ```cpp
    void StrVec::push_back(const string& s) {
        chk_n_alloc(); // ensure that there is room for another element
        // construct a copy of s in the element to which first_free points     
        alloc.construct(first_free++, s);
    }
    ```
    * `construct` 之後的參數直接給 `std::string s`，這樣會用 `std::string` 的 copy constructor 來建構物件
        * 覺得看不懂就回去看 12.2.2(p. 482)
    * 注意那個 `first_free++`
        * 在建構好物件後必須讓 `first_free` 指向最新的未建構的位置

#### The `alloc_n_copy` Member
* `StrVec` 的 copy ctor 跟 copy assignment 都會一次複製多個 elements，所以講這個邏輯寫成 private utility，就是 `alloc_n_copy`
* `StrVec` 的行為跟 `std::vector` 一樣是 valuelike，意味著 copy 跟 assign 會配置一份獨立的空間並將 copy/assign 的 `rhs` 內的 elements 複製一份到該空間
    * copy/assign 完之後 changing one doesn't affect the other
* `alloc_n_copy` 回傳一個 `pair`，分別指向新配置的空間的頭尾
    ```cpp
    pair<string*, string*> StrVec::alloc_n_copy(const string *b, const string *e) {
        // allocate space to hold as many elements as are in the range
        auto data = alloc.allocate(e - b);
        // initialize and return a pair constructed from data and
        // the value returned by uninitialized_copy 
        return {data, uninitialized_copy(b, e, data)};
    }
    ```
    * 參數代表一個 input range，是要複製的 elements
    * 先用 `static` member `alloc` 配置一塊未建構的空間
        * 大小跟 input range 一致
    * 最後再對這塊空間用 `std::uninitialized_copy`(`<memory>` 把 input range 初始化
        * 只是這裡一併寫在 `return` statemenet
        * 用 list initailizer `{}` 初始化 return 的 `std::pair`
        * 第一個 element 是新配置的空間的頭，也就是 `data`
        * 第二個 element 則是 `uninitialized_copy(b, e, data)` 的 return value
            * 代表 output range 的尾，也就是 `data` 指向的 output range 的尾

#### The `free` Member
* 把 elements 用 `std::allocator::destroy` 全部破壞掉，然後呼叫 `std::allocator::deallocate`
    ```cpp
    void StrVec::free() {
        // may not pass deallocate a 0 pointer; if elements is 0, there’s no work to do
        if (elements) { 
            // destroy the old elements in reverse order 
            for (auto p = first_free; p != elements; /* empty */)
                alloc.destroy(--p);     
            alloc.deallocate(elements, cap - elements);
        }
    }
    ```
    * 題外話，什麼時候該用 `first_free` 什麼時候該用 `cap`，要想清楚
        * 只需要對 `[elements, first_free)` range 做 `destroy`
        * 但需要對 `[elements, cap)` range 做 `deallocate`
    * 注意，需要先檢查 `elements` 是否為 `nullptr`
        * 因為傳給 `deallocate` 的 pointer 一定是要用 `allocate` 回傳的，所以要先檢查是否有指向空間
        * 不檢查的話會 UB
#### Copy-Control Members
* 實作 `alloc_n_copy` 跟 `free`，我們就可以來實作 copy-control members
* copy constructor:
    ```cpp
    StrVec::StrVec(const StrVec &s) {
        // call alloc_n_copy to allocate exactly as many elements as in s
        auto newdata = alloc_n_copy(s.begin(), s.end());
        elements = newdata.first;
        first_free = cap = newdata.second;
    }
    ```
    * 因為 `alloc_n_copy` 配置的空間大小剛好就跟 `s` 內配置的大小一樣，所以 `cap` 跟 `first_free` 會指向同一個位置
* destructor:
    ```cpp
    StrVec::~StrVec() { free(); }
    ```
    * 直接呼叫 `free()` 即可
* copy-assignment:
    * 一樣，**先複製在移除**，防止 self-assignment 出錯 && strong exception guarantee
    ```cpp
    StrVec &StrVec::operator=(const StrVec &rhs) {
        // call alloc_n_copy to allocate exactly as many elements as in rhs
        auto data = alloc_n_copy(rhs.begin(), rhs.end());
        free();
        elements = data.first;
        first_free = cap = data.second; return *this;
    }
    ```
#### `move`ing, Not Copying, Elements during Reallocation
* 來實作 `reallocate`
* 要配置新空間時要做的事情:
    * 配置更大的空間(廢話
    * 這塊新空間的前面部份要塞舊的 elements
    * 要刪除舊的空間內的 elements

* 由上面得知我們會需要把舊空間 `std::string` element 一個一個複製到新的空間
    * 因為 `std::string` 是 valuelike type
    * copy 之後會有兩份獨立的 data，會有兩個 user(這裡是指 code，或者說兩個變數)使用(地址)不同但內容相同的物件
        ```cpp
        std::string a = "colleague so cute";
        auto b = a;
        ```
        * `a` 跟 `b` 這兩個 user 指向的物件配置的底層資源互相獨立

* 但仔細想想，在實作 `reallocate` 的情境:
    * 我們雖然一定要把舊 `std::string` elements copy 到新的空間，可是一旦複製完之後，舊的資料馬上就會被刪除了，不會有人使用
    * 換句話說 user 一直都只有一個
* **如果有什麼方法可以避免這個 string 的 copy，效能會更好**

#### Move Constructors and `std::move`
* **先介紹 C++11 的一個新 feature**
* 12.6 才會詳細講，不過這裡先知道一點概念
    * 其實我個人覺得可以把這整段般到 12.6 再講，否則在不知道 move semantics 的情況下看這段解說有點對牛彈琴

* STL container 幾乎都有定義 move constructor
    * 包含 `std::string`

* Move constructors typically operate by **"moving" (underlying) resources from the given object to the object being constructed.**

* **The library guarantees that the "moved-from" `std::string` *remains in a valid, destructible state*.**
    * 我覺得這個 valid, destructible state 在這個時候先講的話就是標準的幹話
* 可以想像 `std::string` 有一個 `char*` 指向配置的 C string，然後 **move ctor 會複製指標而不是複製指標指向的 C string**
    * 並且將被 move 的原 `std::string` 物件內的 `char*` 指標只向 `nullptr`
    * 這樣原物件就會變成 "destructible state"
        * 亦即破壞該 `std::string` 也不會出問題
* 另外，要告訴 compiler 要使用 move ctor 還需要用到 `std::move`
    * 定義在 `<utility>`
* 總之現在要知道的是，對要被複製的 `std::string` 用 `std::move`
    ```cpp
    std::string moved_str = "42";
    std::string move_ctor_str(std::move(moved_str));
    ```
* 還有，不要對 `move` 用 `using`
    ```cpp
    // *** don't do this ***
    using std::move;
    std::string moved_str = "42";
    std::string move_ctor_str(move(moved_str));
    ```
    * 理由 18.2 會解釋...

#### The reallocate Member
* 上面霧煞煞沒關係，先看完 13.6 再來看一次也可以
* 總之來看 `reallocate` 怎麼寫吧
* 首先會配置空間
    * 最單純的配置方式是，配置一個比舊空間大一倍的空間
    * 如果原本的 `StrVec` 是 empty，那就配置大小為 1 個 string 的空間
    ```cpp
    void StrVec::reallocate() {
        // we’ll allocate space for twice as many elements as the current size
        auto newcapacity = size() ? 2 * size() : 1;
        // allocate new memory
        auto newdata = alloc.allocate(newcapacity);
        // move the data from the old memory to the new 
        auto dest = newdata; // points to the next free position in the new array
        auto elem = elements; // points to the next element in the old array
        for (size_t i = 0; i != size(); ++i) 
            alloc.construct(dest++, std::move(*elem++));
        free(); // free the old space once we’ve moved the elements
        // update our data structure to point to the new elements
        elements = newdata;
        first_free = dest;
        cap = elements + newcapacity;
    }
    ```
    * 注意那個 `std::move`
    * `std::move(*elem++)` 回傳的 result 會讓 **function matching 選擇 move constructor**
    * 要完空間，並且 move 完舊字串之後，一樣呼叫 `free()` 歸還舊空間
    * 最後更新 `StrVec` 的三個指標

## 13.6 Moving Objects
![](https://i.imgur.com/l8AmcvL.png)
* 先給這個
    * 看完 13.6 甚至 16 章時記得複習:
        * every C++ expression has a type and value category.

* 我們在實作 copy control 時看到，有非常多的情況會 copy 物件
    * 但這其中又有很多時候，被 copy 的物件馬上就會被破壞(about to be destroyed)
        * 或者 no user of that object
    * 這時候如果 **move 而不是 copy** 物件，效能會提升
    * 把要被破壞的物件的內部資源的 "ownership" 直接轉交給新的物件

* 舉例:
    1. 13.5 的 `StrVec`，或者 STL container
        * 在重新配置更大的空間(reallocate)時，可以 move elements 而不是 copy elements
        * 因為舊的 elements 沒人會再用了(no user of that objects)
    2. 某些**只會有一個 user 使用的 class**，例如 `std::unique_ptr` 或者 IO class，他們的底層其實都有一個**不能被共享的指標**，**這些 classes 因此被規定不能被 copy，但是可以被 move**
        * move 時將物件底層資源的 ownership 轉交給新物件

* 在 pre C++11 沒有直接將物件 move 的機制，一定要用 copy
    * 這代表會有不必要的複製，例如 STL container reallocate 時
* 還有另一個缺點，STL container element type 在舊標準裡一定要 copyable(定義 copy ctor)，**在C++11，element type 只要可以 move (定義 move ctor就行了**
    * 還不確定這是什麼情境

### 13.6.1 Rvalue References
* C++11 定義一種新的 reference ，**rvalue reference**
* **An rvalue reference(以下簡稱 rref) is a reference that must be bound to an rvalue.**
    * obtained by using ``&&`` rather than `&`

* 之後會看到，rref 就是拿來 bind 那些準備要被破壞(about to be destroyed)的(temp)物件
    * 因為他準備要被破壞，所以**我們可以把這個物件的內榮 move 到其他物件**
    * 內容包含:
        * 底層 allocate 的 elements
        * 或者一些資源的 ownership
            * `shared_ptr`, IO class 維護的指標等等

* 所以要懂 rref 在幹嘛之前要先知道什麼是 rvalue(4.1.1)
    * lvalue 跟 rvalue 是 c++ expressionS 的 property
        * **Every C++ expression has a type, and belongs to a value category.**
    * 某些 expression 產生(yield)或需要(require) lvalue
    * 某些則產生或需要 rvalue
    * 一般來說，lvalue 代表的是一個物件的"身分"(identity)，rvalue 代表的是一個物件的"值"(value)
    * 之前都說通常可以把 lvalue expression 拿來放在需要 rvalue expression 的地方
        * **初始化 rvalue reference 的就是一個例外**

* Like any reference, an rvalue reference is just another name for an object.
* 之前也知道，我們沒辦法 bind regular(lvalue) reference 到以下的 expressions:
    * expressions that require a conversion
    * literals
    * expressions that return a rvalue
    * **注意 const reference 還是可以 bind 上面的東西**

* rref 則是有點倒過來
    * 它可以 bind 上面這些東西，但*不能直接* bind lvalue
    ```cpp
    int i = 42;
    int &r = i; // ok: r refers to i
    int &&rr = i; // error: cannot bind an rvalue reference to an lvalue
    int &r2 = i * 42;  // error: i*42 is an rvalue
    const int &r3 = i * 42; // ok: we can bind a reference to const to an rvalue
    int &&rr2 = i * 42; // ok: bind rr2 to the result of the multiplication
    ```
    * 注意 const lref 一樣可以 bind rvalue，不過跟用 rref bind rvalue 不一樣
    * 至少 rref 還是可以改變 bind 的物件，const lref 就不行

* lvalue 更具體的例子如下:
    * return type lvalue reference 的 function
    * 以下 operators:
        * assignment
        * subscript
        * dereference
        * and prefix increment/decrement operators
* 我們可以用 lref bind 上面的東西

* rvalue 更具體的例子如下:
    * return type 為 nonreference 的 functions
    * 以下 operators:
        * arithmetic
        * relational
        * bitwise
        * and postfix increment/decrement operators, 
* 我們不能用 lref bind 上面的東西，可是可以用 lref to const 或 rref 來 bind

#### Lvalues Persist; Rvalues Are Ephemeral
* **Lvalues have persistent state, whereas rvalues are either literals or temporary objects created during evaluating expressions.**
* 因為 rref 只能綁 temporary objects，我們可以得到以下推論:
    * 被綁定的物件很快就會被破壞(about to be destroyed)
    * 不會有其他 user 使用到這個物件(no user to that temp object)
* 可以再得到一個結論:
    * **我們可以將 rref bind 的物件的內部 resource (或者說內部狀態) 搶過來(move 到新物件)**

:::info
* Note: Rvalue references refer to objects that are about to be destroyed. Hence, **we can "steal" state from an object bound to an rvalue reference**.
:::

#### Variables Are Lvalues
* expression 也可以只有一個 variable
    * 只有一個 operand
    * 然後沒有 operator
* variable expression 也有 r/lvalue property
    * **variable expression 是 lvalue**
    * 不管該 varialbe 是什麼 type，即使是 type 為 rvalue reference，他就是個 lvalue
* 再講一次，Every C++ expression **has a type**, and **belongs to a value category**.
* **因為如此，所以我們不能用一個 rref 來 bind 一個 variable(expression)**
    ```cpp
    int &&rr1 = 42; // ok: literals are rvalues 
    int &&rr2 = rr1; // error: the expression rr1is an lvalue!
    ```
    * 注意上面的 code 是想把 `rr2` bind 到 `rr1`，即使 `rr1` 的 type 是 rvalue reference，這個 variable expression 的 value category 還是 lvalue，所以 rr2 不能直接拿來綁 `rr1`!
* 一個 variable 是 lvalue 有點難理解的話可以這樣想
    * A variable persists until it goes out of scope.
    * 所以他沒有即將被破壞(about to be destroyed)，而且也有一個 user

:::warning
* **A variable(expression) is an lvalue**;
* we cannot directly bind an rvalue reference to a variable expression even if that variable was defined as an rvalue reference *type*
:::

#### The Library `std::move` Function
* 雖然 rref 不能直接綁 variable，但我們可以 explicitly cast variable to 對應的 rvalue reference type，using `std::move`
    ```cpp
    int &&rr3 = std::move(rr1); // ok
    ```
    * calling `std::move` tell compiler we have an lvalue that we **want to treat as if it were an rvalue**.
        * 呼叫 `std::move` 告訴 compiler 我們想把一個 lvalue 當作 rvalue 來用
    :::info
    * 講一些跳脫解說的幹話
    * 又是一種 explicit intention
    這個在定義類似吃 `std::unique_ptr` 的 function 的 interface 時很重要
    * 定義成 pass by value
    * 導致你只能在 caller 端傳 argument 時使用 std::move(unique_ptr_obj)
    * 也間接導致你明確表達你要 explicit transfer resource ownership to the callee
    :::
    * `std::move` 是用一個 16 章會介紹的 utility 來達成回傳 rvalue reference 的
        * 怕，怕爆 `std::remove_reference` 之類的
* **It is essential to realize that the call to `std::move` promises that we do not intend to use `rr1` again**
    * except to assign to it
    * or to destroy it.
* 這個概念非常重要，某個 lvalue 被當作 `std::move` 的參數之後，你不能假設它擁有什麼 value
    * 它的 value 是 *unspecified*，除非你重新對他做 assign
* 總之呼叫了 `std::move()` 之後的 object 只能做兩種操作，assign 或者直接破壞
* 參考:
    * https://stackoverflow.com/questions/7027523/what-can-i-do-with-a-moved-from-object
    * 
https://stackoverflow.com/questions/7027523/what-can-i-do-with-a-moved-from-object

* 這裡再提醒一次，請用 `std::move` 不要用 `using std::move`，原因 18 章會解釋

### 13.6.2 Move Constructor and Move Assignment
* move ctor 第一個參數要是 class_name&&，也就是一個 rvalue reference
    * 有額外參數則全部都要有 default arguments
* 注意，你的 move ctor 不光是把 rref 綁定的物件的 resources 搶過來而已，你還要讓 rref 綁定的的物件維持在一個 valid state，可以直接將這個物件安全破壞(harmless)
    * 概念上來說，一旦 resources 被 newly created object 接手，這個 rref 綁定的物件就不該再"指向"這些 resources
    * —**responsibility** for those resources has been transferred by the newly created object.
        * 維護這些 resources 的責任已經轉移到 newly created object 了

* 用 `StrVec` 來舉例:
    ```cpp
    StrVec::StrVec(StrVec &&s) noexcept // move won’t throw any exceptions 
    // member initializers take over the resources in s 
    : elements(s.elements), first_free(s.first_free), cap(s.cap)
    {
        // leave s in a state in which it is safe to run the destructor
        s.elements = s.first_free = s.cap = nullptr;
    }
    ```
    * 之後會介紹 `noexcept` 在幹嘛
    * 首先，move cotr 沒有配置新空間
        * 相反的，它把 moved from object 配置的空間"搶過來"
    * 然後把 moved from object 的內部指標設成 `nullptr`
    * 這樣 moved from object 準備被破壞時，呼叫 dtor 才會是合法的
        * 如果沒有設定 moved from object 的內部指標，那對他呼叫 dtor，就會把剛搶過來的空間給 deallocte

#### Move Operations, Library Containers, and Exceptions
* 因為 move ctor 是搶資源，沒有額外配置新空間，一般來說 move ctor 不會 throw exception
* **如果我們確定我們寫的 move ctor 不會 throw exception，我們*必須告知 library(或 compiler)**
* 之後會看到，除非我們有"告知" library 我們的 move ctor 不會 throw exception，不然 library 會假設我們的 move ctor 可能會 throw 並且做一些額外處理
* 告知不會 throw exception 的方法之一是將 move ctor 宣告成 `noexcept`
    * 18.1.4 會講 `noexcept` 的細節，現在只要知道 `noexcept` 是要告知被宣告的 function 不會 throw exception
        * 18 章有講 namespace 跟 exception，這裡是在 exception 的章捷
    ```cpp
    class StrVec {
    public:
        StrVec(StrVec&&) noexcept; // move constructor
        // other members as before
    };
    StrVec::StrVec(StrVec &&s) noexcept : /* member initializers */
    {/* constructor body */}
    ```
    * 放在 parameter list 之後，ctor_init_list 之前
    * 宣告跟定義 ctor 都要寫 `noexcept`

##### deepen understanding on `noexcept`
* 前面有說，如果我們的 move ctor 可能會 throw exception，則 library 需要做一些額外處理
* 用 STL container 來舉例
* STL container 有保證，當對他做操作時噴了 exception，則 container 會維持在做操作之前的狀態
    * strong exception gaurantee
* 之前也看到 `StrVec` 會有 reallocate 的動作，STL container 也有
    * 將舊物件般至新配置的空間
* 如果 element type 有宣告 move ctor **且為 `noexcept`**，則 container 才會使用 move ctor 來搬移物件
* 如果 element type move ctor 會 throw exception 的會發生以下情境:
    * container 先配置更大的空間
    * **將物件從舊空間 move construct 到新空間**
    * move 到某個 element 時 throw exception
    * 這時，已經被 move 的，在原空間的 elements 就已經不是原本的狀態了
    * 而新的空間也還沒完全複製完成
    * **但這時 container 已經無法變回還沒 reallocate 之前的狀態了**
    * 導致無法維持 container 保證的 strong exception gaurantee
    * **所以 element type 的 move ctor 沒有 `noexcept` 的話 container 會改成使用 copy ctor**
* fallback 回 copy ctor 的話就很簡單了，因為就是先 allocate 新空間再將就 elements copy 到新空間，最後再刪除舊 elements
    * 如果 copy 到一半噴 exception 則將新配置的空間直接刪除即可，container 內部還是維持在操作之前的狀態

* **總之，move ctor 必須宣告成 `noexcept`，否則 standard lib 都會用 copy ctor**

#### Move-Assignment Operator
* 套路跟 copy-assignment 很像，move-assignment 做的就是 dtor + move ctor
* 跟 move ctor 一樣，如果它不會 throw，必須宣告成 `noexcept`
* 也要確保 self-assignment...
    * 等等 temporary 怎麼會有 self assignment?!
        * `lol = std::move(lol);`
    * 只能直接 check `this != &rhs` 了
```cpp
StrVec &StrVec::operator=(StrVec &&rhs) noexcept {
    // direct test for self-assignment
    if (this != &rhs) {
        free(); // free existing elements
        elements = rhs.elements; // take over resources from rhs
        first_free = rhs.first_free;
        cap = rhs.cap;
        // leave rhs in a destructible state 
        rhs.elements = rhs.first_free = rhs.cap = nullptr;
    }
    return *this;
}
```
    
#### A Moved-from Object Must Be Destructible
* 說了 N 遍，被 move 過的物件呼叫 ctor 必須是安全的
* 除此之外，物件需要*維持一個 valid 狀態*
    * valid 的意思是，對他做:
        * assignment
        * 其他*不會參考到它的 value 的* operations
    * 必須也要是安全的
* move 後的物件不需要維持原本的 value 甚至任何 value
    * user 也不可以假設它有任何的 value
* 例如你可以 move STL container，然後對它呼叫 `size()` 或者 `empty`，**不會爆炸，可是 value 是未定義**

* 如果拿我們寫的 `StrVec` 舉例，我把 move `StrVec` 物件之後，它會維持跟 default initialize 一樣的狀態(三個指標都 == `nullptr`)，所以對它做任何操作所得到的結果就會跟一個 default initialize 的 `StrVec` 物件一樣
    * 但是更複雜的 class 例如 STL container 就不保證 move 後的結果會怎樣
    * 而且我們也不該 rely on `StrVec` 當前的行為


#### The Synthesized Move Operations
* compiler 也會合成 move ctor 跟 move assignment
* **但是何時會合成他們的條件跟 copy 很不一樣**
    * TL;DR
    * 不要記這些規則，請遵循 rule of five
        * 全定義或全不定義
* 還記得 copy-ctor 跟 copy-assign
    * 只要你沒有定義，compiler always 會合成一個
    * 合成出來的要馬 memberwise copy/assign，要馬是 `delete` function

* 但對於某些 class，compiler move 則是連合都不合ㄌ
##### 不合成的條件:
* 如果 class 只要 copy cstr **或** copy assign **或** dstr，那 compiler 就不會合成 move cstr 跟 move assignment
* 所以有些 class 的確會沒有 move ctor 或 move assignment 的(但每個 class 一定都會有上面那三個，可能是 `delete`)

* 那如果沒有定義 move，當需要用到 move 的 context 會用什麼操作呢?
    * 就會改回直接用對應的 copy 操作

##### 合成 move 的條件:
* 上面說的那三個操作都沒有自定義
* 所有的 non`static` data member 都可以 move
    * built-in 可以 move
    * class object，有定義 move 的話也可以 move
    ```cpp
    // the compiler will synthesize the move operations for X and hasX
    struct X {
        int i; // built-in types can be moved 
        std::string s; // string defines its own move operations
    };
    struct hasX {
        X mem; // X has synthesized move operations
    };
    X x, x2 = std::move(x); // uses the synthesized move constructor
    hasX hx, hx2 = std::move(hx); // uses the synthesized move constructor
    ```
* 跟 copy 不同，compiler 如果合成 move，合成出來的一定是 never `delete`
* 但是如果我們明確的用 `= default'` 叫 compiler 合成 move，但是 class 卻不能 move 時，那 compiler 就會把 move 定義成 `delete` function
    * 從這邊看就知道鬼則有點鬧了，別記
* 所以你用 `= default` 叫 compiler 合成 move，而 move 何時會被定義成 `delete` 跟 copy 何時會被定義成 `delete` 有點類似
    * *除了一個重要例外*:
    * move constructor:
        * 如果 class 有 member 有定義 copy ctor 但是沒有定義 move ctor
        * 或者有 member 沒有定義 copy ctor，但是它還是不能用 move
            * (會發生這種情況就是這個 class 有 member 不能 move，或者主動宣告成 delete)
        * 那 class 的 move 就會宣告成 `delete` function
    * move assignment: 跟 move constructor 很像
        * 如果 class 有 member 的 move ctor 或者 move assignment 如果定義成 delete function
        * 或者不可訪問(inaccessible)
        * 這樣 class 的 move cstr/assign 就會定義 delete function
    * 如果 dtor 定義成 `delete` 或者 dtor inaccesible，那 move ctor 就會定義成 `delete` function
    * 如果 class 有 member 是 `const` 或者 reference，那 move assign 就會定義成 `delete` function

* 舉例...
* ~~假設 `Y` 是一個有自訂 copy ctor 但是沒定義 move ctor 的 class:~~
:::danger
這裡 Primer 寫錯了
:::
* 假設 `Y` 是一個有自訂 copy ctor 但是 move ctor 宣告為 delete 的 class，這樣才對:
    ```cpp
    // assume Y is a class that defines its own copy constructor "but not a move constructor" ERROR
    struct hasY {
        hasY() = default;
        hasY(hasY&&) = default;
        Y mem; // hasY will have a deleted move constructor
    };
    hasY hy, hy2 = std::move(hy); // error: move constructor is deleted
    ```
    * 這時候 `hasY` 的 move ctor 宣告成 `default`，這樣 compiler 就會把它定義成 `delete` function
    * `hy2 = std::move(hy)` 就會使用到 move ctor，但是是 `delete`，所以噴 error


* 最後一個 move member 跟 copy member 之間的關係:
    * 如果 class 有定義 move ctor and/or move assignment，那 compiler 合成的 copy ctor/assignment 就會是 `delete` function

* 換句話說，**定義了 move ctor or move assignment 的 class 一定要自訂 copy ctor/assignment，否則 compiler 就會合成 `delete` function**

#### Rvalues Are Moved, Lvalues Are Copied ...

* 如果 class 同時擁有 copy ctor 跟 move ctor，那 compiler 就用一般的 function matching 來決定要用哪個
    * assignment 相同
* 例如 `StrVec` 的 copy ctor 是吃 `const StrVec&`，它可以 match 所有可以轉成 `StrVec` 的 argument
* 而 `StrVec` 的 move ctor 是吃 `StrVec&&`，只有 argument 是 non`const` rvalue 時才會使用
    * 說真的... `const` rvalue 是怎樣...
    ```cpp
    StrVec v1, v2;
    v1 = v2; // v2 is an lvalue; copy assignment
    StrVec getVec(istream &); // getVec returns an rvalue
    v2 = getVec(cin); // getVec(cin) is an rvalue; move assignment
    ```
    * `v1 = v2;`，move assignment 不是 viable(第六章 function matching) function
        * 因為不能把 rref 綁到一個 lvalue，所以 move assignment 根本不會考慮，於是使用 copy assignment
    * 而 `v2 = getVec(cin);`，copy 跟 move assignment 都是 viable functions，因為 `getVec(cin)` 是 rvalue
        * 如果要用 copy assignment，需要做一次 `const` 轉換
        * 而用 move assignment 則是 exact match
        * 所以使用 move assignment

#### ...But Rvalues Are Copied If There Is No Move Constructor
* 上面說的是 move/copy 都定義的情況，但如果 class 只定義 copy 沒有定義 move 呢?
* 那就直接用對應的 copy member，~~即使我們強制用 `std::move` 傳一個 rvalue reference 也一樣~~
    * 這邊注意，`std::move` 實際上是回傳 xvalue，which is rvalue，which can bind to rref or lref to `const`
    ```cpp
    class Foo {
    public:
        Foo() = default;
        Foo(const Foo&); // copy constructor
        // other members, but Foo does not define a move constructor
    };
    Foo x;
    Foo y(x); // copy constructor; x is an lvalue
    Foo z(std::move(x)); // copy constructor, because there is no move constructor
    ```
    * The call to `std::move(x)` in the initialization of z returns a `Foo&&` bound to x.
    * The copy constructor for `Foo` is *viable because we can convert a `Foo&&` to a const `Foo&`.*
    * 所以 z 會用 copy ctor
* 這邊講解真的很怪
    * 應該說 `std::move(x)` return rvalue，而我們可以用 `const ref` 綁到 rvalue，所以可以呼叫 copy ctor

* 值得注意的是，用 copy ctor/assignment 來使用在原本需要 move ctor/assignment 幾乎是安全的
    * 因為從定義上，被 moved 後的 object
        * 需要 remain valid
        * 可以被安全破壞
    * 而被使用到 copy ctor 的 object，也符合上面這兩個定義
        * 該物件沒被更動，所以 remain valid
        * 沒被更動當然可以被安全破壞


#### Copy-and-Swap Assignment Operators and Move
* 記得我們的 `HasPtr` 的 `operator=` 不是吃 reference，而是吃 value，我們還用這個來實作 copy-and-swap
* **如果這時我們再定義 move ctor，那這個 `operator=` 就同時會是 copy assignment 跟 move assignment!**
    ```cpp
    class HasPtr {
    public: // added move constructor 
        HasPtr(HasPtr &&p) noexcept : ps(p.ps), i(p.i) {p.ps = 0;}
        // ***assignment operator is both the move- and copy-assignment operator***
        HasPtr& operator=(HasPtr rhs) 
            { swap(*this, rhs); return *this; }
        // other members as in § 13.2.1 (p. 511)
    };
    ```
    * 因為 `operator=` 的 argument 是 call by value，所以會呼叫 ctor 複製一份
        * **而要呼叫 copy 還是 move ctor 就取決於 argument 是 l/rvalue**
    * lvalue 就呼叫 copy ctor，rvalue 就呼叫 move ctor
    * 所以到底要 move 還是 copy，在 argument passing 階段就解決了，所以這個 `operator=` 可以同時當 copy assignment 也可以當 move assignment
    ```cpp
    hp = hp2; // hp2 is an lvalue; copy constructor used to copy hp2
    hp = std::move(hp2); // move constructor moves hp2
    ```
    * `hp = hp2;`，`operator=` 在初始化 parameter 時，move ctor not viable，所以用 copy ctor
    * `hp = std::move(hp2);`，move ctor 跟 copy ctor 都 viable，move ctor 是 exact match，用 move
    * copy assignment 時
        ![](https://i.imgur.com/Yhkhvef.png)
    * move assignment 時
        ![](https://i.imgur.com/irCTFyT.png)
        
* ADVICE: UPDATING THE RULE OF THREE(to FIVE)
    * 已經說了要把 copy ctor/copy assignment/dtor 當成一個 unit 來看待
        * 定義其中一個代表其他的也要定義
    * 而通常**一定**要定義這三個的，也會自己額外配置資源
        * 這樣的 class，通常定義另外兩個 move 效能會更好，節省不必要的動態空間配置
    * 而且 move 在 compiler 是否定義&&怎麼定義的規則真的太鬧了，不要記
        * 直接定義比較快

#### Move Operations for the `Message` Class
* 又用這個例子...
* 我們的 `Message` 跟 `Folder` 一樣可以因為定義 move semantics 讓效能變好
* 可以對 data member `string` 跟 `set` 用 move
* 還記得一個 `Message` 有一個 `folders` member 嗎
    * 那是拿來記住該 `Message` 所處的所有資料夾
    * 如果呼叫 `Message` 的 move ctor，必須把要被 move 的 Message 這個 member 內存的指標都設對
    * 將 `folders` 內有指像舊 `Message` 的指標移除，並且指向新的 newly created object
        * `Folder` 的情況也是類似
* 因為 `Message` 的 move ctor 跟 move assignment，都需要改 `folders` member，所以定義成一個 common private utility:
    ```cpp
    // move the Folder pointers from m to this Message
    void Message::move_Folders(Message *m) {
        folders = std::move(m->folders); // uses set move assignment
        for (auto f : folders) { // for each Folder
            f->remMsg(m); // remove the old Message from the Folder
            f->addMsg(this); // add this Message to that Folder
        }
        m->folders.clear(); // ensure that destroying m is harmless
    }
    ```
    * 這個 function 是 move ctor/assignment 會呼叫的 utility
    * 還有，因為這個 utility 還是會額外配置空間(`f->addMsg(this);`)，所以可能會 throw
    * 這意味著呼叫這個 utility 的 move ctor/assignment 不能宣告成 `noexcept`，不像之前的 `HasPtr` 一樣
    * 主要做的就是把原本所有 `Folder`s 內指向 `rhs` 的指標都拔掉，然後新增指向 `lhs` 的指標
    * 最後把 `folders` 給 `clear`，保證 `Message` 的 dtor 在把 `folders` 掃過一遍時它是空的
        * `folders` 被 move 之後 remain valid
            * 但是不知道他到底值是什麼
            * 所以呼叫 `clear` 強制清空
                * 並且 `clear` 不會相依 `folders` 的 current value，所以 safe
    * ![](https://i.imgur.com/jBXxlf7.png)

* 於是 move ctor 就可以這樣寫:
    ```cpp
    Message::Message(Message &&m): contents(std::move(m.contents)) 
    {
        move_Folders(&m); // moves foldersand updates the Folderpointers
    }
    ```
    * `contents` 是 `string`，直接用 `string` 的 move ctor
    * 然後 `this->folders` 用 default ctor，之後才會用 `move_Folders` 把 `this->folders` 在 `move_Folders` 用 move assign 給值(`folders = std::move(m->folders);`)
* move assignment 就這樣寫:
    ```cpp
    Message& Message::operator=(Message &&rhs) {
        if (this != &rhs) { // direct check for self-assignment
            remove_from_Folders();
            contents = std::move(rhs.contents); // move assignment
            move_Folders(&rhs); // reset the Folders to point to this Message 
        }
        return *this;
    }
    ```
    * 一樣直接確認 self-assignment
    * 先把所有 `Folder`s 內指向 `lhs` 的指標移除，然後把 `this->folders` 清空(也就是 `remove_from_Folders` 做的事)
    * 之後把 `rhs` 的 `contents` move 過來
    * 最後也呼叫 `move_Folders`，把 `set` move 過來，把所有 `Folders` 內原本指向 rhs 的指標改成指向 lhs

* 反正上面的 `Message` 例子就是一堆指標要刪除或更改，看 code & 寫練習的時候要想清楚

#### Move Iterators
* 換看 `StrVec`
* `StrVec::reallocate` 是用 iterator 的方式掃過(用 allocate)新配置的空間，然後一個一個 construct
    * ![](https://i.imgur.com/VnyXfzR.png)
* 其實可以用 `uninitiailized_copy` 來做
* 但如字面意思，`uninitiailized_copy` 真的是用 copy ctor，C++11 沒有什麼 `uniniailized_move` 這種東西
    * C++17 有 `uninitialized_move`
    *  http://en.cppreference.com/w/cpp/memory/uninitialized_move
* 不過 C++11 定義了 move iterator
    * C++14/17 也有對這東西額外做擴充
    * http://en.cppreference.com/w/cpp/iterator/make_move_iterator
* 這 iterator 本質上就是把 `*(dereference)` 這個操作改變
    * 原本的 iterator 是回傳 lvalue reference
    * 而 move iterator 則是回傳 rvalue reference
* 用 `make_move_iterator` 把一個普通 iter 轉成 move_iter:
* 然後來改寫 `StrVec::reallocate`:
    ```cpp
    void StrVec::reallocate() {
        // allocate space for twice as many elements as the current size
        auto newcapacity = size() ? 2 * size() : 1;
        auto first = alloc.allocate(newcapacity);
        // move the elements
        auto last = uninitialized_copy(make_move_iterator(begin()), 
                                       make_move_iterator(end()),
                                       first);
        free(); // free the old space
        elements = first; // update the pointers 
        first_free = last;
        cap = elements + newcapacity;
    }
    ```
    * 這樣 `reallocate` 就會把舊 elements 直接 move 到新空間而不是 copy 到新空間了
    * 這裡雖然 `uninitialized_copy` 是想要"複製" element 到 `first`，但其實 `uninitialized_copy` 做的只是把 range 內的 elements 丟給 ctor 罷了
    * 而因為我們是丟一個 rref to range 內的 elements，所以 function matching 就會選擇 move ctor 而不是 copy ctor。
    * 另外 `uninitialized_move` 怎麼用請去看 cppreference

* 另外值得一提
    * 那些原本吃 iterator 的 STL algo，你要確定他們不會 access 被 move 過的物件的 value 才能用 move_iter，否則會出事
    * you should pass move iterators to algorithms only when **you are *confident* that the algorithm does not access an element after it has assigned to that element or passed that element to a user-defined function.**

:::warning
* ADVICE:DON’T BE TOO QUICK TO MOVE
    * move 通常比較常用在 class implementation
    * 在 user code 比較少用
    * 使用 move 時，只有確定你一定需要 move，而且保證 move 之後的 object 不會有 user
:::

### 13.6.3 Rvalue References and Member Functions
* class 的其他 member functions 如果也有提供 move overloaded 版本通常也會受惠
    * 同名字的 member function 你就 overload 兩種，一種是 reference to `const`，一種是 rvalue reference to non`const`
    * 這個 pattern 跟 copy/move ctor 給的參數一樣

* 例如 C++11 的 STL container，有提供 push_back 的就有兩種版本:
    ```cpp
    void push_back(const X&); // copy: binds to any kind of X
    void push_back(X&&); // move: binds only to modifiable rvalues of type X
    ```
* `const X&` 可以接受任何可以轉換成 `X` 的參數，包括 modifiable rvalue
* `X&&` 只能接受 modifiable rvalue
* 不過當參數真的是 modifiable rvalue 時，`X&&` 是 exact match，`const X&` 還要做一次 `const` 轉換，所以 function matching 會選擇 `X&&` 的版本
* 下面給個例子:
    ```cpp
    class foo {
    public:
        foo() { cout << "default ctor\n"; }
        foo(int) { cout << "convert from type convertible to int\n"; }
        foo(const foo &f) { cout << "copy ctor\n"; }
        foo(foo &&f) noexcept { cout << "move ctor\n"; }
    };
    int main() {
        vector<foo> vec;
        int i = 10;
        double d = 87.0;
        vec.reserve(10);
        foo f;
        cout << "----------\n";
        vec.push_back(f); // copy
        cout << "----------\n";
        vec.push_back(10); // 10 convert to temp foo, using push_back(T&&);
        cout << "----------\n";
        vec.push_back(i); // i convert to temp foo, using push_back(T&&);
        cout << "----------\n";
        vec.push_back(d); // d narrow down to int and convert to temp foo, using push_back(T&&)
        cout << "----------\n";
        vec.push_back(foo()); // using push_back(T&&) directly
    }
    ```
    
* 另一個例子是對 `StrVec` 增加另一個版本的 `push_back`:
    ```cpp
    class StrVec {
    public:
        void push_back(const std::string&); // copy the element 
        void push_back(std::string&&); // move the element
        // other members as before
    }; 
    // unchanged from the original version in § 13.5 (p. 527)
    void StrVec::push_back(const string& s) {
        chk_n_alloc(); // ensure that there is room for another element 
        // construct a copy of sin the element to which first_free points     
        alloc.construct(first_free++, s);
    }
    void StrVec::push_back(string &&s) {
        chk_n_alloc(); // reallocates the StrVec if necessary
        alloc.construct(first_free++, std::move(s));
    }
    ```
    * 注意看吃 rref 的版本
        * 呼叫 `construct` 時要給 `std::move(s)` 當參數。
        * 再提醒一次，雖然 `s` 的 type 是 `string&&`，可是 variable 就是 lvalue，一定要用 `std::move` 取得 rref
* 使用 `push_back` 時就會這樣 resolve:
    ```cpp
    StrVec vec; // empty StrVec 
    string s = "some string or another";     
    vec.push_back(s); // calls push_back(const string&)
    vec.push_back("done"); // calls push_back(string&&)
    ```
    * These calls differ as to whether the argument is an lvalue (`s`) or an rvalue (the temporary string created from `"done"`)

#### Rvalue and Lvalue Reference Member Functions
* 我們可以寫這樣的 code:
    ```cpp
    string s1 = "a value", s2 = "another";
    auto n = (s1 + s2).find(’a’);
    ```
    對 `s1 + s2` 這個 temp object 呼叫 find
* 可是我們也可以寫這樣很奇葩的 code:
    ```cpp
    s1 + s2 = "wow!";
    ```
    * 看起來很機掰，不過你只要想成 `operator=` 也是 member function 就會比較好理解
    ```cpp
    (s1 + s2).operator=("wow");
    ```
    * 題外話，這招在 build-in type 當然行不通
    ```cpp
    int i, j;
    i + j = 87; // error
    ```
    * 而上面的 assign 不是 member function，所以會噴 error

* 在新標準提供了一種方法防止我們對 rvalue 做 assign
* 不過在舊標準已經訂出來的 class，因為 backward compatibility，在 C++11 還是允許對他們的 rvalue 做 assign
    * 所以上面的奇葩 `std::string` assign 還是合法的
* 但我們自己寫的 class 可以用這種新功能，防止這種ㄎㄧㄤ行為
    ```cpp
    class Foo {
    public:
        Foo &operator=(const Foo&) &; // may assign only to modifiable lvalues
        // other members of Foo
    };
    Foo &Foo::operator=(const Foo &rhs) & {
        // do whatever is needed to assign rhs to this object 
        return *this;
    }
    ```
    * 有沒有看到那個 `&` ?
    * 那叫做 **reference qualifier**，有兩種，`&` 和 `&&`
* `&&`/`&` **indicating that this may point to an rvalue or lvalue, respectively.**
* 它跟 `const` qualifier 一樣，只能出現在 non`static` member function
    * 並且宣告跟定義都要寫
* 加上 `&` 的話，只有 lvalue object 可以使用這個 member function；`&&` 則是 rvalue
    ```cpp
    Foo &retFoo(); // returns a reference; a call to retFoo is an lvalue
    Foo retVal(); // returns by value; a call to retVal is an rvalue
    Foo i, j; // i and j are lvalues
    i = j; // ok: i is an lvalue
    retFoo() = j; // ok: retFoo() returns an lvalue
    retVal() = j; // error: retVal() returns an rvalue
    i = retVal(); // ok: we can pass an rvalue as the right-hand operand to assignment
    ```
    * 如果 `Foo` 在 `operator=(const Foo&)` 加上 &，`retVal() = j;` 就會噴 error，因為不能對 `retVal()` 回傳的 rvalue 用 `operator=`

* 一個 member function 可以同時用 `const` 跟 ref qualifier
    * 如果要同時用，要先寫 `const` 再寫 ref qualifier:
    ```cpp
    class Foo {
    public:
        Foo someMem() & const; // error: const qualifier must come first
        Foo anotherMem() const &; // ok: constqualifier comes first
    };
    ```
    
#### Overloading and Reference Functions
* 我們可以直接用 `const` 來 overloading member function
* 我們也可以用 reference qualifier 來 overload
    * 但我除了 Primer 舉的例子，不知道還有什麼適合的使用時機
* 更甚，這兩個東西還可以組合...
* 我們給 `class Foo` 加一個 `std::vector` member，再配上一個 `sorted` function，回傳一個內部 `std::vector` 已經 sort 過的 `Foo` object
* 根據 reference qualifier 跟 `const` 的 overloading，我們可以這樣寫:
    * ![](https://i.imgur.com/ojtsTl3.png)

    * 如果我們是用 rvalue 呼叫 `sorted`，那我們直接對這個 rvalue 內的 vector sort 也無妨(?
    * 如果是用 lvalue 或者 const rvalue 呼叫 sorted，因為"概念上"我們就不想/不能 in-place sort(lvalue 是"不想"，const rvalue 是"不能")，所以我們要創一個 temp `Foo` 來 copy `*this`
        * 然後對 temp object sort，再回傳 temp object
        * 因為不做 in place sort，所以索性把 `sorted` 宣告成 `const`

* 來看什麼 object 會呼叫哪種 `sorted`:
    ```cpp
    retVal().sorted(); // retVal() is an rvalue, calls Foo::sorted()&&
    retFoo().sorted(); // retFoo() is an lvalue, calls Foo::sorted()const&
    ```

* 再來是如果你要用 ref qualifier 來 overload member function，而且 function name 跟 para list 都一樣，**你一定要所有 function 都明確寫出 ref qualifier，不然會 error:**
    ```cpp
    class Foo {
    public:
        Foo sorted() &&;
        Foo sorted() const; // error: must have reference qualifier
        // Comp is type alias for the function type (see § 6.7 (p. 249))
        // that can be used to compare intvalues using Comp = bool(const int&, const int&);
        Foo sorted(Comp*); // ok: different parameter list
        Foo sorted(Comp*) const; // ok: neither version is reference qualified
    }
    ```