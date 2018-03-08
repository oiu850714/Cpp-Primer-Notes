# C++ Primer Chapter 13 Copy Control
* 第七章講的是如何用 class 定義新 type，包括 operation，封裝，以及 constructor 這些基本議題
* 這章要講的是如何定義 class object 在 copy, assign, move, 還有 destroy 時的行為
* 這些 feature 分別對應到下面幾個，之後會講:
    * copy constructor
    * move constructor
    * copy-assignment operator
    * move-assignment operator
    * destructor


* 當我們定義 class 的時候，我們會(明確地)指定出，class object 在 copy, move, assign, destroy 時會發生什麼事
    * 沒有指定的話就是 compiler 幫你合成一個
    * 如果要明確指定的話就是要用上面那五個 member functions
    * copy/move constructor 是在操控，當我們用一個 class object 來初始化另一個 class object 時的行為
    * copy/move-assignment operator 是在操作當我們把一個 class object assign 給另一個 class object 時的行為
    * destructor 是在操控當 class object 要被消滅時的行為
* **上述的行為被當作 C++ 的 copy control**
* 有些 class **一定要自己定義 copy control**，不然會悲劇
    * 什麼時候要自己定義之後會講，通常是自己要 dynamic resource 的時候

## 13.1 Copy, Assign, and Destroy
* 這三個比較簡單所以先講(


### 13.1.1 The Copy Constructor
* cstr，第一個參數是 reference to class type，如果第一個參數後面還有參數的話，全部都要有 default value
    ```C++
    class Foo { 
    public:
        Foo(); // default constructor
        Foo(const Foo&); // copy constructor
        // ...
    };
    ```
* 之後會講為什麼一定要用 reference type
* 然後 reference 幾乎都是 const，雖然我們還是可以定義 nonconst 的版本
* 然後 copy cstr 通常都會 used implicitly
    * class_name obj2 = obj1; //這樣就會呼叫 到 copy cstr
    * 所以不應該對 copy cstr 用 explicit

#### The Synthesized Copy Constructor
* 如果我們沒定義，compiler 會幫我們合成一個
    * *不像 default cstr，就算我們有定義其他 cstr，只要沒有定義 copy cstr，compiler 就一定會幫我們定義*
* Primer P.508 會說到，對於某些 class，compiler 生成的 copy cstr 反而會讓我們不能對這個 class 的 object 做 copy
    * 為看先猜是因為 class 有 member 不能 copy，例如有 iostream object 之類的
* 但如果沒有發生上面的情況的話，compiler 預設的 copy cstr 就會 memberwise copies nonstatic members 到新的 class object 內
* 那 compiler 生的 copy cstr 怎麼複製 class member? **簡單! 呼叫 member 自己的 copy constructor!!**

* 現在來對到目前為止 compiler 生的兩個 cstr，default 跟 copy，做個總結
    * synthesized default cstr 會呼叫所有 member 的 default cstr
    * synthesized copy cstr 則會呼叫所有 member 的 copy cstr

* 可是如果有 member 是 array 呢? array 理論上不是不能 copy?
    * array 的話，synthesized copy cstr 會把 element 一個一個 copy 到新的 class object 的 array 裡面

* 用 Sales_data 來舉例，compiler 幫他生的 synthesized copy cstr 等價於這樣:
    ```C++
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
        bookNo(orig.bookNo), units_sold(orig.units_sold), // copies orig.units_sold
        revenue(orig.revenue)
        {   } // empty body
    ```
    * 注意，是用 constructor initializer list，只有用這個 list 才會用到 member 的 constructor，在 function body 開始執行之後 member 都已經初始化完成了
        * 不知道上面在講啥的話要回去看第七章

#### Copy Initialization
* **我們現在終於可以好好討論 direct initialization 跟 copy initialization 的差異惹!**
    ```C++
    string dots(10, '.'); // direct initialization
    string s(dots); // direct initialization
    string s2 = dots; // copy initialization
    string null_book = "9-999-99999-9"; // copy initialization
    string nines = string(100, '9'); // copy initialization
    ```
* 當我們用 direct initialization，我們是告訴 compiler 用基本的 function matching(第六章) 來找對應的 constructor
* 當我們用 = 來初始化 object，我們是告訴 compiler 把 rhs of = 當作 initializer，然後讓新 object copy 這個 initializer
    * 如果 initializer 要轉換的話就做轉換，例如上面的 ```string null_book = "9-999-99999-9";```，literal 會先轉成 string，null_book 在用這個 temp string 來 copy initialize

* 通常你在宣告 object 時用 = 時都會呼叫 copy cstr
    * 不過 13.6 講了 move cstr，你會發現如果你有額外定義 move cstr 的話，在某些情況下就會呼叫 move cstr 而不是 copy cstr
* 總之現在的重點是**要知道什麼時候會呼叫 copy/move cstr**
    1. 宣告變數時用 =
    2. 把物件當 argument 傳到 *nonreference* parameter 時
    3. return object as *nonreference* type 時
    4. 你對一個 array 用 {initializer...}，則對應的 element 會被 copy initialize
        * 還有如果你定義的是 aggregate class(7.5.5, p.298)，就是那個跟 C struct 87% 像的 class，且也是用 {initializer...}，則 aggregate class 的 memeber 也會 copy initialize。
* 除了上面講的，container 的某些操作也是 copy initialize
    * 例如初始化 container，elements 都是用 copy 的
    * call insert 或 push_back 這類 member 時
    * *而像 **emplace** 這種的就是對 element 做 **direct initialize** 了*

#### Parameters and Return Values
* 就跟上面的 2 3 點一樣，不過有個重點
* 這裡講為什麼 copy cstr 一定要接 reference，(幹我好像想起來了)，如果 copy cstr 是接 nonreference，那在 function call 時為了要初始化 parameter 會呼叫 copy_cstr(argument)，可是**別忘了 copy_cstr 也是 function，這樣為了初始化 copy_cstr 的 parameter 又要對 copy_cstr 的 argument 複製一次，於是又呼叫 copy_cstr... GG

#### Constraints on Copy Initialization
* 就只是再說一次，宣告 explicit 的 cstr 不能用 =
    ```C++
    vector<int> vec(10); //OK
    vector<int> vec = 10; //error, vector<int> 接收 int 的 cstr 是 explicit，= 右邊的 10 沒辦法隱性轉成 vector 再 copy 給 vec
    ```
* 其他傳參數或 return value 也是一樣


#### The Compiler Can Bypass the Copy Constructor
* 這裡寫的很簡短，可是感覺很像 RVO?
* compiler 可以把
    ```C++
    string null_book = "9-999-99999-9"; // copy initialization
    ```
* 重寫成
    ```C++
    string null_book("9-999-99999-9"); // compiler omits the copy constructor
    ```
* 不過要能把第一個轉成第二個的條件是 copy/move cstr 一定要存在而且是 public

### 13.1.2 The Copy-Assignment Operator
* class 也**控制 object 被 assign 時發生什麼事**
    ```C++
    Sales_data trans, accum;
    trans = accum; // uses the Sales_datacopy-assignment operator
    ```
    * 最後的 assignment 就會用到 copy-assignment operator
* class 沒定義 compiler 就會生一個

#### Introducing Overloaded Assignment
* 雖然 14章才會講 operator overloading
* 反正你就是要定義 operator= 啦
* 對於 operator*x*() 這類 operator overloading member function，parameter list 內的東西就是 operand
* 有些 operator 一定要定義成 class 的 member function
    * 這時候 operator 的 lhs 會被 bound 成 this，rhs 就是傳進來的參數
* 總之 copy-assignment 會這樣定義:
    ```C++
    class Foo {
    public:
        Foo& operator=(const Foo&); // assignment operator
        // ...
    };
    ```
* 因為習慣上 class 的 operator 使用使用起來要跟 built-in type 的 operator 行為一致，**所以自定義的 = 也要回傳 lhs 的 reference**
* 而且你會發現 standard function 也會要求，如果你的 class operator 有自定義的話，例如 =，他的某些行為真的必須跟 built-in 一樣，例如 = 要 return lhs&，不然會有很ㄎㄧㄤ的結果

* Best Practice: Assignment operators ordinarily should return a reference to their lefthand operand.

#### The Synthesized Copy-Assignment Operator
* 你沒定義的話 compiler 會合成一個
* 在某些情況下跟 合成的 copy cstr 類似，compiler 合成的 cpy-assgin 反正會讓你不能 assign
    * 大概又是你有 member 不能 assign 的時候吧

* 如果合出來的 可以 assign，cpy-assign op 就會把所有的 rhs 的 nonstatic member 用他 member 自己的 cpy-assign op，assign 給 lhs 的 nonstatic member
    * 總之就是呼叫 member 的 cpy-assign op 啦

* 又來了，如果 member 是 array 呢?? **那就把所有 element 一個一個 assign!!**
    * 有人用這種很ㄎㄧㄤ的方法把 array 用 class 包起來，這樣可以做 assign，不過感覺是糞 code

* 合成的 cpy-assign op 就會 return lhs&

* 一樣來看 Sales_data compiler 合出來的 cpy-assign op 長什麼樣子
    ```C++
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
* 理論上你要在 dstr free 你用到的資源
* 除此之外 dstr 還會呼叫 member 的 dstr
    * 不用自己呼叫，member 要被破壞時就會自己呼叫了
* dstr 格式是這樣:
    ```C++
    public:
        ~class_name();
    ```
* 一個 class 只能有一個 dstr，而且不能吃參數，這意味著 dstr 不能 overloading

#### What a Destructor Does
* Just as a constructor has an initialization part and a function body (§ 7.5.1, p. 288), a destructor has a function body and a destruction part.
    * cstr 有 cstr_init_list + body, dstr 也有類似的東西
* 在 cstr，member 會在 body 執行之前初始化，而且 member *會按照他們宣告的順序依序初始化*
* 而在 dstr，body 會在所有 member 呼叫自己的 dstr 之前執行，而之後 member 會按照他們宣告的順序**倒過來依序破壞**
    * 有點 LIFO 的感覺
* 上面有講 dstr 也有對應的 destruction part，其實沒有辦法直接寫出來，是 implicit 的，沒有辦法像 cstr_init_list 寫出來
    * member 怎麼被破壞 depends on 他們自己的 dstr
    * 其實不用寫出來也很合理，因為每個 class 的 dstr 就只能有一個，代表只有一種破壞方式，所以就不用寫說要怎麼破壞了；cstr_init_list 則是可以指定要怎麼初始化 member，因為可以有很多種方式初始化，所以才需要 cstr_init_list 來明確指定要用哪種

* built-in type 沒有 dstr 這種東西，反正就只是說 built-in type 的那塊空間要歸還而已
    * **但是指標也是 built-in type，他要被破壞時並不會 delete 他指向的物件!!**

#### When a Destructor Is Called
* 什麼時候 class dstr 會被呼叫呢?
    * class object out of scope 時
    * 當一個 object 被破壞時，他的 member 也會呼叫自己的 dstr
    * 當 container，不管是 STL 還是 built-in array，被破壞時，他們的 element 會呼叫  dstr
    * 動態配置的指標被 delete 時，其指向的物件會 call dstr
    * 當你創造一個 temp object 時，創造這個 temp object 的 full expression 完成之後

* 注意看例子，註解會寫什麼時候會呼叫 dstr:
    ```C++
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
    * 我們只需要 delete 我們自己 allocate 的指標就行了，Sales_data 的 dstr 會自動呼叫，他的 member 都會自動釋放資源
        * 其實 Sales_data 只有 string member，string 會自己管理資源，Sales_data 不用去管他

#### The Synthesized Destructor
* 沒定義的話 compiler 會合成一個
* 一樣，某些情況下 compiler 合成的反而會讓我們不能"破壞物件"
    * 超 WTF 的，Primer p.508 會講
* 如果沒發生上面說的情況，那合成出來的 dstr 就會是...
    * ~class_name{}
    * 對，empty body!
    * 很合理阿，什麼事都不做
    * 其實不是啦，只是 member 的 dstr 會在 dstr 的 destruction part 被 call 而已
    * *The members are automatically destroyed after the (empty) destructor body is run.*

### 13.1.4 The Rule of Three/Five
* 超級重要
* **通常**，當有定義這五種 member (包括我們還沒講的剩下兩個有關 move 的 operation) 的其中一個的需要時，其他四個也必須定義，不然你的 class 的行為就會是錯的
* Primer 說必須把這五種 member 想成是一個單位，必須同時都存在

#### Classes That Need Destructors Need Copy and Assignment
* Often, the need for a destructor is more obvious than the need for the copy constructor or assignment operator.
* 之前練習題定義的 HasPtr 是個好例子
    * 他有用 new 要一塊空間
    * 如果用預設 dstr，就不會在破壞 HasPtr 物件時把那塊空間 delete，所以要自訂 dstr，之後會以 rule of thumb 告訴你說為什麼這時也要同時定義 copy cstr 跟 copy assign(以及更後面會講到的 move)
    
* 如果只定義了 dstr 而沒定義 copy cstr 跟 copy assign:
    ```C++
    class HasPtr { 
    public:
        HasPtr(const std::string &s = std::string()): 
            ps(new std::string(s)), i(0) { }
        ~HasPtr() { delete ps; } // WRONG: HasPtrneeds a copy constructor and copy-assignment operator 
        // other members as before
    };
    ```
* 這時候如果只用 compiler 合的那兩個 member:
    ```C++
    HasPtr f(HasPtr hp) // HasPtr passed by value, so it is copied 
    {
        HasPtr ret = hp; // copies the given HasPtr 
        // process ret 
        return ret;
    }
    ```
    * 執行 argument passing 時，hp.ps 就會跟 argument.ps 指向同一塊記憶體
    * 在執行 ret = hp 時，ret.ps 跟 hp.ps 也會指向同一塊記憶體
    * 當 f 結束後，ret 跟 hp 都會 call dstr，所以會把 ret.ps 指向的記憶體 delete，也把 hp.ps delete，**這樣就 double free!!*
    * 這時就已經 UB 了，而且 argument.ps 指向一塊 free 的空間，變成 dangling ptr

#### Classes That Need Copy Need Assignment, and Vice Versa
* 這是一個邏輯敘述，如果一個 class 必須自訂 dstr，*幾乎可以肯定他還要定義 copy cstr/assignment*；但是一個 class 需要定義 copy cstr/assignment，*不一定需要定義 dstr*

* 舉一個最簡單的例子就是，class 內有 ID 的概念(或 **unique** serial number)的話就需要定義 copy/ cstr/assignment，但不一定要 dstr
    * 這樣子的話就不能直接用合成的 copy cstr/assignment 來複製這個 ID，因為概念上這個 ID 要每個 object 都 unique
    * 所以需要自訂 copy cstr/assignment 來給予新的 object 一個新的 ID
* 如果這個 ID 不需要動態配置資源，且其他 member 也不需要，那其實就不用定義 dstr
* 所以這是第二個 rule，class 要定義 copy cstr ** 幾乎 iff** class 要定義 copy assignment；不過定義這兩個 member 不一定要定義 dstr

### 13.1.5 Using = default
* 可以用 ```= default``` 的語法明確告訴 compiler 幫我合成一個 member
    ```C++
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
    * 把 = default 寫在 class 裡面(也就是在宣告時)，或者外面(也就是在定義時)的差別，就是一個會 inline 一個不會
* Note: 只能對那些會有何成版本的 member 使用 ```= default```

* 13.1.6 Preventing Copies
* Best Practice: Most classes should define—either implicitly or explicitly—the default and copy constructors and the copy-assignment operator.
* 如果對於你的 class 來說，copy 這種 operations 完全不合理，**你必須明確定義他們為 ```=delete```，而不是不定義他們。
    * 因為不定義他們 compiler 就會合成一個(除非有 member 也不能 copy)

#### Defining a Function as Deleted
* 在 C++11，我們可以把 copy cstr 跟 copy assign 定義成 delete functions 來防止 class object 被 copy
    * A deleted function is one that is declared but may not be used in any other way.
    * ```= delete```
    ```C++
    struct NoCopy { 
        NoCopy() = default; // use the synthesized default constructor
        NoCopy(const NoCopy&) = delete;  // no copy
        NoCopy &operator=(const NoCopy&) = delete; // no assignment
        ~NoCopy() = default; // use the synthesized     destructor
        // other members
    };
    ```
* 跟 ```= default``` 的使用時機有些不同
    * 一定要在第一次宣告 member 時就使用 ```= delete```，不能在定義時才使用
        * 有這種差異其實很容易理解，因為 ```= default``` 是告訴 compiler 怎麼幫我 gen code，寫在定義也可以；而 ```= delete``` 是在 compiler 還在檢查 source code 時就要知道的事情，因為他才能檢查 source code 有沒有不小心使用這個 member
    * ```= delete``` 可以用在任何 member functions，不像 ```= defalut``` 只能用在 compiler 會合成的 member

        * Primer 很輕描淡寫的說可以用在一些 function matching 的場合，來導引怎麼 match...? 有些不可能的 matching 直接宣告出來然後寫成 ```= delete```，防止 ambiguous 嗎?

#### The Destructor Should Not be a Deleted Member
* 已經說可以用在任何 member function 了，當然包括 dstr
* 但最好不要用在 dstr...
* 因為你一旦用了，compiler 不准你定義這個 class 的 object，而且也不准你定義有這種 object 的 class 的 object，因為一旦 object out of scope，就會呼叫 dstr，可以已經 delete 了
* 但是你可以動態配置這種 class 的 object LOL，只是不能 delete 指標，因為 delete 會呼叫 dstr
    ```C++
    struct NoDtor {
        NoDtor() = default; // use the synthesized default constructor
        ~NoDtor() = delete; // we can’t destroy objects of type NoDtor
    };
    NoDtor nd; // error: NoDtordestructor is deleted 
    NoDtor *p = new NoDtor(); // ok: but we can’t deletep
    delete p; // error: NoDtor destructor is deleted
    ```
#### The Copy-Control Members May Be Synthesized as Deleted
* For some classes, the compiler defines these synthesized members as deleted functions:
    * dstr 會合成成 delete，如果有 member 的 dstr 宣告成 delete，或者 member 的 dstr 不能 access(例如 access specifier 是 private)
    * cpoy cstr 會合成成 delete，如果有 member 的 copy cstr 宣告成 delete 或不能 access；*copy cstr 在有 member 的 dstr 宣告成 delete 或者不能 access 時，也會合成成 delete*
    * cpy assign op 會合成成 delete，如果有 member 的 cpy assign op 是 delete 或不能 access，*或者 class 有 const 或 reference member*
    * default cstr 會合成成 delete，如果有 member 的 default cstr 是 delete 或不能 access，
        * 或者有 member 是 reference，可是沒有給 in-class initializer
            * 其實 reference 的 case，用 assign 是合法的，可是你仔細想想就會發現這個行為會跟預期結果不符合，那乾脆不要合成
        * 或者有一個 member 是 const type，而且他的 type 沒有明確定義 default cstr，那個 member 也沒有 in-class initializer
        * 以上兩點都是要在 class object 初始化時用 in-class intializer 給值的(reference 跟 const object 只能這樣，或者用 in-class initializer 給值)

* **看起來很細，但大方向的概念是，如果一個 class 有 member 他不能被 default constructed, copied，assigned, 或 destroyed，那對應的合成 member 就會被合成成 delete**

* 之後的章節(13.6.2, 15.7.2, 19.6)還會介紹其他會合成函數被合成成 delete 的情況

#### private Copy Control

* 阿 C++11 之前又沒有 ```= default; = delete```，可以用，怎麼做到等價行為的?
    * 把這五種 member 宣告成 private!

* 先看 copy 的例子
    ```C++
    class PrivateCopy {
        // no access specifier; following members are privateby default; see § 7.2 (p. 268)
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
* 這樣一般的 user code 就不能 copy `PrivateCopy` 的 object 了
* **可是 friend 還是可以**
* 如果要避免 friend copy，以前的做法就是**宣告但是不要定義 member**
    * 上面這個一般來說都是合法的，不過 15.2.1 會說到，有一個情況一定要定義 member
    * 應該是繼承的時候ㄅ，超難超噁

* 如果按照上面的作法，那麼 user code 使用 copy 就會是 **compile time error**，而 friend 使用 copy 則會是 **link time error**
* 不過上面都是幹話，C++11 請用 `= default; = private;`

## 13.2 Copy Control and Resource Management

* Ordinarily, classes that manage resources that do not reside in the classmust define the copy-control members.
* In order to define these members, **we first have to decide what copying an object of our type will mean.**
* Primer 說可以把 class 分成兩大類: behave like a value or like a pointer
    * 如果 class 是 valuelike，那 object 複製之後，兩個 objects 各自獨立，改變其中一個的狀態不會影響到另一個
        * 可以想成 class object 自己掌控自己的 states
    * 如果是 pointer-like，那 object 複製之後，兩個 objects 會以某種形式共享部分的 data，或者說改變其中一個 object 也會改變另一個
        * 可以想姓成這種 class 的 object 複製之後會*指向*同一塊 underlying data

* 標準的 container type 就算是 valuelike
* shared_ptr\<T> 就算是 pointer--like；我們之前定義的 StrBlob 也是屬於這類

* 而如果一個 class 不允許 copy 那就沒有必要討論他們是 valuelike 還是 pointer-like 了，因為這兩個本來就是建立在 copy 時的行為定義出來的名詞
    * 例如 iostream 或 unique_ptr\<T> 就是

* 為了介紹這兩種形式的 class 的 copy-control 怎麼寫，Primer 會把練習時用的 HasPtr 用兩種方式重新實作
    * 還記得他有一個 int 跟 string\* member
    * 其實複製 built-in type(除了指標)，通常都是直接用 assign，那這本來就跟 valuelike 行為一致
    * **真正決定我們定義的 class 像個 value 還是 pointer 取決於我們怎麼操作我們的指標 member**

### 13.2.1 Classes That Act Like Values
* 反正就是不能跟別的 objects 共用 resources 啦(除了 static member)
* To implement valuelike behavior HasPtr needs
    * A copy constructor that copies the string, *not just the pointer*
    * A destructor to free the string
    * A copy-assignment operator to f*ree the object’s existing string* and copy the string from its right-hand operand

    ```C++
    class HasPtr {
    public:
        HasPtr(const std::string &s = std::string()): 
            ps(new std::string(s)), i(0) { }
        // each HasPtr has its own copy ofthe string to which pspoints
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
    * Like the destructor, assignment destroys the left-hand operand’s resources.
    * Like the copy constructor, assignment copies data from the right-hand operand.
    * 先把 object 自身該丟掉的丟掉，再複製 rhs 的 data 到 lhs

* However, it is crucially important that these actions be done in a sequence that is correct even if an object is assigned to itself.
    * 你要確定 user code 寫出 obj = obj; 時 operator= 也要運作正確

* Moreover, when possible, we should also write our assignment operators so that they will leave the left-hand operand in a sensible state should an exception occur.
    * 我們要確保就算 exception 在 assign 時發生，lhs 也要能維持在一個合理的 state。
    ```C++
    HasPtr& HasPtr::operator=(const HasPtr &rhs) {
        auto newp = new string(*rhs.ps); // copy the underlying string 
        delete ps; // free the old memory
        ps = newp; // copy data from rhs into this object
        i = rhs.i;
        return *this; // return this object
    }
    ```

* 上面的寫法遵循一個 pattern: **先做 copy constructor 做的事，做好之後再做 destructor 做的事**
    * 個人認為是因為 dstr 幾乎不會噴 exception，可是 copy cstr 會，所以如果先做 dstr 再做 cpy cstr，如果 cpy cstr 噴 exception，那 object 就會處於一個不合理的狀態?

* 再來 demo 一個錯的 operator=
    ```C++
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

* 總之必須寫出一個 self assignment 也會正確的 code
    * 最基本的做法是檢查 this != &rhs
    * 另一個則是像 Primer 寫的，先把 rhs 的 data 都複製到 temp，再刪除 lhs 的 data，再把 temp 丟給 lhs，這樣就不用去管是否 self assignment，更 uniform
        * 不過效能?? 也許是跟 exception safe 的 trade off?

### 13.2.2 Defining Classes That Act Like Pointers
* 讓 HasPtr 看起來像指標
* 還是得定義 destructor
* 可是你用 raw pointer 就會發現 dstr 很難寫，你如果直接 delete，會導致其他 object 有 dangling ptr
* 最簡單的解法就是用 12章的 shared_ptr


* 但總是會有不能用 shared_ptr 的時候，這時就必須使用一些 reference count 的機制

#### Reference Counts
* 除了 copy cstr 的 cstr 都要 create 一個 counter，這個 counter maintain 有多少 object 跟我們 share states
    * 初始化時設成 1

* cpy cstr 則是把他複製的物件的 counter++
* dstr 當然就 counter--
    * 如果 counter == 0 就 delete
* cpy assign op 則是 rhs 的 counter++，lhs 的 counter--
    * 如果 lhs 的 counter == 0 就 delete

* 可是還沒講要怎麼放 reference count 阿
    * 但總之不能是 HasPtr 的 member
    ```C++
    HasPtr p1("Hiya!"); 
    HasPtr p2(p1); // p1 and p2 point to the same string
    HasPtr p3(p1); // p1, p2,and p3 all point to the same string
    ```
    * 生成 p3 的時候有辦法改 p1，可是改不到 p2，但是他們三個頭指向同一個 string!!
* 可以連 counter 都是動態配置的，然後 copy object 時直接 copy 指標，這樣就會不同 objects 就會指向相同 counter

#### Defining a Reference-Counted Class
* 可以這樣寫:
    ```C++
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
    * 多了一個 use 指標，default cstr 會 new 一個 size_t 給他指，初始化為 1，代表有一個 object shared 指向的 string

#### Pointerlike Copy Members "Fiddle"(撥弄) the Reference Count

* copy 時要指向 old object 的 string，並且增加 old string 對應的 counter
    * 仔細看 cpy cstr，他會把 nonptr member 跟 ptr member 都複製，亦即指向相同物件，並且增加 counter

* dstr:
    ```C++
    HasPtr::~HasPtr() {
        if (--*use == 0) {
        // if the reference count goes to 0 
        delete ps; // delete the string
        delete use; // and the counter
        }
    }
    ```
    * --counter == 0 時才能刪除物件跟 counter

* cpy assign op:
    * 一樣是做 cpy cstr 跟 dstr 的事情
    * 亦即，增加要 copy 的物件的 counter，然後原本指向的物件的 counter - 1，該刪除就刪除
    * 然後一樣要處理 self-assignment 的情況
    * 這時候 ++ 跟 -- counter 都會是對同一個 counter
        * **可是必須先 ++ 再 --，如果倒過來可能會遇到 counter == 0 然後就把物件刪除了；總之就是要先做 copy cstr 再做 dstr
    ```C++
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
## 13.3 Swap
* 9.2.5 有初步介紹，當時只是說如果 swap container，原本指向 container 的 iterator 不會失效(string 除外)
* 這裡進一步說明，**通常需要自定義 copy-control 的 class 也會定義一個 swap function**
* 自定義 swap 的其中一個原因是，如果你的 class 可用在那種需要重排順序的 STL lib，STL 會 call swap 來交換 element
    * 如果你沒有自定義專屬這種 class 的 swap，STL 就會用 std::swap function template 生出一個這種 class 的版本
* 但就跟 copy-control 一樣的問題，通常自己管理資源的 class，compiler 定義出來的 swap 通常都跟預期行為不一致
* *題外話，C++ 用 ADL 決定要使用哪個 swap，非常非常後面會講到什麼是 ADL*
* 概念上的 swap 就是
    ```C++
    HasPtr temp = v1; // make a temporary copy of the value of v1
    v1 = v2; // assign the value of v2 to v1
    v2 = temp; // assign the saved value ofv1to v2
    ```
* 所以會包一個 copy cstr 跟兩個 copy assignment
* 上面的例子假設是 valulike 版本:
    * 那原本在 v1 內指向的 string 就會被複製兩次，一次是 tmpe 呼叫的 copy cstr(v1)，一次是 v2 呼叫 copy assign(temp)
    * 而 v2 內指向的 string 也會複製一次，就是 v1 呼叫 copy assign(v2)
* **但實際上這些 copy 完全不需要，指要 swap 指向 string 的指標就好了:**
* 觀念注意: 如果有自定義 copy-control，那其實 swap 完全沒必要額外定義，因為 std::swap 會使用你定義的 copy-control 交換 object。自定義其實是為了優化。
    ```C++
    string *temp = v1.ps; // make a temporary copy of the pointer in v1.ps
    v1.ps = v2.ps; // assign the pointer in v2.ps to v1.ps
    v2.ps = temp; // assign the saved pointer in v1.ps to v2.ps
    ```
    
#### Writing Our Own swap Function
* We can override the default behavior of swap by *defining a version of swap that operates on our class.*
    ```C++
    class HasPtr {
        friend void swap(HasPtr&, HasPtr&); // other members as in § 13.2.1 (p. 511)
    };
    inline void swap(HasPtr &lhs, HasPtr &rhs) {
        using std::swap;
        swap(lhs.ps, rhs.ps); // swap the pointers, not the string data
        swap(lhs.i, rhs.i); // swap the int members
    }
    ```
    * swap 是 global function，要成為 HasPtr 的 friend
    * 這是我們第一次在 function 內使用這種 using 語法，等等會講這在幹嘛

* swap 短短ㄉ，用 inline
* 再來是 swap 兩種指標

#### swap Functions Should Call swap, Not std::swap
* 注意，以下講的東西只有告訴你應該要這樣寫才會有預期行為，而並沒有告訴你為什麼，詳細原因要等到 16.3 講 template 時跟 18.2.3 講 ADL 時才會說明
* 如果要 call 對某個 class 自定義的 swap 要這樣寫:
    ```C++
    void swap(Foo &lhs, Foo &rhs) {
        using std::swap;
        swap(lhs.h, rhs.h); // uses the HasPtr version of swap
        // swap other members of type Foo
    }
    ```
* using std::swap 是必須的，這樣子當在 `swap(Foo &lhs, Foo &rhs);` 時，呼叫 `swap(lhs.h, rhs.h)` 才會呼叫到 `swap(const HasPtr&, const HasPtr&);` 而不是 std::swap

#### Using swap in Assignment Operators
* 另一個自定義 copy-control 的 class 也要定義 swap 的原因來啦!!
* 用 swap 來定義 copy assignment op
* **These operators use a technique known as copy and swap.**
    * This technique swaps the lefthand operand with a copy of the right-hand operand:
    ```C++
    // note rhs is passed by value, which means the HasPtr copy constructor 
    // copies the string in the right-hand operand into rhs
    HasPtr& HasPtr::operator=(HasPtr rhs) {
        // swap the contents of the left-hand operand with the local variable rhs
        swap(*this, rhs); // rhs now points to the memory this object had used
        return *this; // rhs is destroyed, which deletes the pointer in rhs
    }
    ```
    * **注意參數是 call by value**
    * 先把 right hand operand 複製一份，然後再把 left hand operand 跟這個 temp object 做 swap，最後讓 (local 的) temp object dstr 把 left had operand 舊的資料(現在存在 temp object 裡面)刪掉
    * 這樣還是維持著**先 copy 再 delete 的順序**，**Strong exception safety**


## 13.4 A Copy-Control Example
* 並不是只有要管理資源的 class 需要 copy-control，你要寫 log 還是三小的，千奇百怪的理由都要自訂 copy-control，只是訂出來不一定會管理資源而已

* Primer 定義一兩個需要做 bookkeeping 的 class 當作例子
    * `Message` 跟 `Folder`
    * 來表示 email 跟儲存 email 的資料夾
* Each Message can appear in multiple Folders. However, there will be only one copy of the contents of any given Message.
* 為了表示上面的結構，每個 `Message` 會有一個 `set` 存著 pointers to `Folder`，代表這個 Message 出現在那些 `Folder`s，而每個 `Folder` 也會有一個 `set` 存著 pointers to `Message`，代表這個 `Folder` 存的所有 `Message`s
    * ![](https://i.imgur.com/QIi3K0Y.png)

* `Message` API:
    * `save`: 把一個 `Message` 存到一個 `Folder`
    * `remove` : 把一個 `Message` 從一個 `Folder` 移除
    * cstr: 創造 `Message` 的內容，但不存在任何 `Folder`
    * copy cstr: 複製 `Message` 的內容，原物件跟新物件是"不同 object"，但是原物件出現的 `Folder` 新物件都要出現
        * 必須 copy set of `Folder*`
        * 每個對應的 `Folder` 內的 set of `Message*` 也要新增一個來只像這個新 `Message` 物件
    * dstr: 將有存這個 `Message` 物件的 `Folder` 的 set 更新，拔掉指向這個 `Message` 物件的指標
        * 這個不難，因為 Message 也有 set of `Folder*`，可以找出有哪些 `Folder` 有這個 `Message`
    * cpy assign op: conents of lhs are replaced by rhs；原本包含舊內容的 `Folder` 要刪除這個信件，然後包含 rhs 的 `Folder` 要新增 lhs

* 會發現
    * dstr 跟 cpy assign 都要把 `Message` 從 `Folder` 移除
    * cpy cstr 跟 cpy assign 都要從 `Folder` 新增 `Message`
* 所以定義兩個 private utilities 給他們用
* Best Practice: 通常一個 class 的 cpy assign 會重複做 cpy cstr 跟 dstr 的事情，這時候通常會 refactor 成一個 unitilty 給他們共用

* `Folder` 也有類似的事情要做，新增或移除 `Folder` 時也要把對應 `Message` 的 `set<Folder*>` 移除指標
* `Folder` 會當成練習，不過這裡假設他有 `addMsg` 跟 `remMsg` 可以用
    ```C++
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
        // add this Messageto the Folders that point to the parameter
        void add_to_Folders(const Message&);
        // remove this Message from every Folder in folders 
        void remove_from_Folders();
    };
    ```
# The save and remove Members
* `Message` 只有這兩個 public member(除了 copy-control 之外)
    ```C++
    void Message::save(Folder &f) {
        folders.insert(&f); // add the given Folder to our list of Folders
        f.addMsg(this); // add this Message to f’s set of Messages
    }
    void Message::remove(Folder &f) {
        folders.erase(&f); // take the given Folder out of our list of Folders
        f.remMsg(this); // remove this Message to f’s set ofMessages
    }
    ```
    * When we save a Message, we store a pointer to the given Folder; when we remove a Message, we remove that pointer.
    * Folder 自己決定要怎麼更新他的 `set<Message*>`，`Message` 不用管

#### Copy Control for the Message Class
* 之前說過我們把 copy 跟 dstr 以及 cpy assign 重複做的事情拆成 private utility，分別是 `add_to_Folders` 跟 `remove_from_Folders`:
    ```C++
    // add this Message to Folders that point to m 
    void Message::add_to_Folders(const Message &m) {
    for (auto f : m.folders) // for each Folder that holds m
        f->addMsg(this); // add a pointer to this Message to that Folder
    }
    ```
    * 之前說過要被 copy 的 object 的 `set<Folder*>` 都要新增一份新 object，上面的 range for 就是在幹這件事
* 有了上面的 utility，copy cstr 就可以這樣寫:
    ```C++
    Message::Message(const Message &m): contents(m.contents), folders(m.folders) {
        add_to_Folders(m); // add this Messageto the Foldersthat point to m
    }
    ```
    * contents 直接用 cstr_init_list 初始化，而更動對應 `Folder` 則用 add_to_Folders 解決

#### The Message Destructor
* 一樣定義一個 dstr 還有 cpy assign op 共用的 utility:
    ```C++
    void Message::remove_from_Folders() {
        for (auto f : folders)  // for each pointer in folders
            f->remMsg(this); // remove this Message from that Folder
        folders.clear();
        // no Folder points to this Message
    }
    ```
* dstr 就可以這樣寫:
    ```C++
    Message::~Message() {
        remove_from_Folders();
    }
    ```

#### Message Copy-Assignment Operator
* 接下來就是用那兩個 utility 組出 copy assign op
    * 一樣，要處理 self-assignment，還要 exception safety
## 注意，這裡 Primer 裡面寫的 operator= 應該有 bug
* https://stackoverflow.com/questions/29308115/protection-again-self-assignment
* Primer 定義的 operator= 是吃 const Message&，這樣先呼叫 remove_from_Folder 就會把資料都刪光了
* 這個 case 應該是無可避免一定要檢查 this != &rhs
    ```C++
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
* 或者用 copy and swap 也可以
* ***不過 swap 之後，原本的指標還是指向舊的物件，所以必須自定義 swap，來更新 `Folder` 內的指標...==
* 這樣我有點不確定下面這個 operator= 是否正確，先用上面有檢察 lhs != rhs 的版本
    ```C++
    Message& Message::operator=(Message rhs) {
        remove_from_Folders();
        add_to_Folders(rhs);
        swap(*this, rhs);
    }
    ```
    
#### A swap Function for Message
* 總之為了要讓指標指向另一個 `Message` 物件，必須更新 `Folder` 內的指標，看不懂的話建議畫出指標怎麼指...
    ```C++
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
## 13.5 Classes That Manage Dynamic Memory
* 講過 N 遍了，如果你一定要自己分配動態記憶體，你一定要自訂 copy-control
* Primer 要定義一個像是 `vector` 的東西，不過沒有 template，存著 `string`，叫做 `StrVec`

#### StrVec Class Design
* 跟 `vector` 一樣，當空間滿了時，會直接要一個新的兩倍大空間，然後把 elements copy 過去。
* 但是如果直接 `new []`，新的空間的後半部份都會預設初始化，很沒效率，**所以會用 allocator**
* 要新增一個 element 時，用 construct
* 刪除 element 時，用 destroy
    * allocator 不清楚請去看 12.2.2

* 主要會有三個指標來維持當前配置的動態陣列的狀態:
* ![](https://i.imgur.com/FHoRJRK.png)
    * elements, which points to the first element in the allocated memory
    * first_free, which points just after the last actual element
    * cap, which points just past the end of the allocated memory
* 除此之外還會有一個 static data member `alloc`，他是 `allocator<string>`
* 我們還會定義四個 private utilities:
    * `alloc_n_copy`: 配置空間，然後複製給定 range 的 elements 到空間
    * `free`: 把所有配置空間內建構(constructed)的 elements 都破壞掉，並解對空間呼叫 deallocate
    * `chk_n_alloc`: 這個 function 保證一定可以再塞一個 element 到配置的空間，如果空間不夠，`chk_n_alloc` 會呼叫 `reallocate` 配置新空間
    * `reallocate`: 沒空間時 `reallocate` 會重新配置空間
    
#### StrVec Class Definition
* 講了上面的就來定義ㄅ
    ```C++
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
    
    // allocmust be defined in the StrVec implmentation file 
    allocator<string> StrVec::alloc;
    ```
    * 上面ㄉ都差不多，***不過再提醒一次，nonconst static member 一定要在 class definition 外再定義一次***，最好定義在 implementation file，也就是 .cpp 檔
    * default cstr 就宛如像在預設初始化一個 vector 一樣，讓指標 members 都射程 `nullptr`，代表沒有 elements
    * `size` 回傳真正擁有的 elements 數量
    * `capacity` 回傳可用空間
    * `chk_n_alloc` 會重配置空間，當 capacity 滿了的時候，亦即，當 `cap == first_free` 時
    * `begin` 跟 `end` 回傳 `elements` 跟 `first_free`，意味著第一個 element 跟最後一個 element 之後的位置

#### Using construct
* `StrVec` 的 `push_back` 實際上會先叫 `chk_n_alloc();` 確保可以直接塞 element，呼叫完之後直接在 first_free 指向的位置 construct 新的 element:
    ```C++
    void StrVec::push_back(const string& s) {
        chk_n_alloc(); // ensure that there is room for another element
        // construct a copy of s in the element to which first_free points     
        alloc.construct(first_free++, s);
    }
    ```
    * `construct` 之後的參數直接給 `std::string s`，這樣會用 `string` 的 copy constructor 來建構物件
    * 注意那個 `first_free++`，在建構好物件後必須讓 `first_free` 指向最新的未建構的位置

#### The alloc_n_copy Member
* `vector` 支援 copy 跟 assign，這時候需要支援一次複製多個 elements，所以我們也需要這種 utility，舊式 `alloc_n_copy`
* 記住 vector(還有其他 STL containers) 的行為是 valuelike，意味著 copy 跟 assign 會配置一份獨立的空間，copy/assign 完之後 changing one doesn't affect the other
* 這個 function 回傳一個 `pair`，分別指向新配置的空間的頭尾
    ```C++
    pair<string*, string*> StrVec::alloc_n_copy(const string *b, const string *e) {
        // allocate space to hold as many elements as are in the range
        auto data = alloc.allocate(e - b);
        // initialize and return a pair constructed from data and
        // the value returned by uninitialized_copy 
        return {data, uninitialized_copy(b, e, data)};
    }
    ```
    * 幹... 真是他媽的精簡
    * 參數代表一個 range，是要複製的空間的頭尾
    * 先用 allocate 配置一塊未建構 elements 的空間，大小跟要複製的空間一樣大(用 `e - b` 來表示)，return value 丟給 data，代表這個空間的第一個 element 的位置
    * return 的地方很炫泡，用 list initailizer \{} 建構一個 pair，first 是 data，代表新配置的空間的頭，而 `uninitialized_copy(b, e, data)` 是把 \[b, e) 代表的 range 的 element 複製到 `data` 代表的 range，然後會回傳 data 代表的 range 最後一個 element 之後的位置! 總之這個 pair，代表的就是新配置空間的頭尾
        * 不知道 uninitialize_copy 在幹嘛的一樣去看 12.2.2

# The free Member
* 把 elements 用 destroy 全部破壞掉，然後呼叫 deallocate
    ```C++
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
    * 注意，什麼時候該用 `first_free` 什麼時候該用 `cap`，要想清楚
    * 注意，因為傳給 deallocate 的 value 一定是要用 allocate 回傳的，所以要先檢查是否有指向空間
    * 如果不檢查的話，在以下這個情況會 UB:
        * 你預設初始化一個 `StrVec` 物件，然後直接 assign 他另一個 `StrVec` 物件(`operator=` 會 call `StrVec::free`)，這時候就會 UB，自己想想
    * 可是感覺就算檢查，`vec1 = vec2;`，如果 `vec2` 是空的，感覺還是會怪怪的耶...

#### Copy-Control Members
* 有了 `alloc_n_copy` 跟 `free`，我們就可以來實作 copy-control members 惹
* copy constructor:
    ```C++
    StrVec::StrVec(const StrVec &s) {
        // call alloc_n_copy to allocate exactly as many elements as in s
        auto newdata = alloc_n_copy(s.begin(), s.end());
        elements = newdata.first;
        first_free = cap = newdata.second;
    }
    ```
    * 因為 `alloc_n_copy` 配置的空間大小剛好就跟 `s` 內配置的大小一樣，所以 `cap` 跟 `first_free` 會指向同一個位置
* destructor:
    ```C++
    StrVec::~StrVec() { free(); }
    ```
* copy-assignment operator:
    * 一樣，**先複製在移除**，防止 self-assignment 出錯
    * 可我不懂耶... 這裡檢查 lhs != rhs 應該會比較好吧....
    ```C++
    StrVec &StrVec::operator=(const StrVec &rhs) {
        // call alloc_n_copy to allocate exactly as many elements as in rhs
        auto data = alloc_n_copy(rhs.begin(), rhs.end());
        free();
        elements = data.first;
        first_free = cap = data.second; return *this;
    }
    ```
#### Moving, Not Copying, Elements during Reallocation
* 我們要在配置空間滿的時候重新配置大一點的空間，我們要想想怎麼做:
    * 配置更大的空間(廢話
    * 這塊新空間的前面部份要塞舊的 elements
    * 舊的空間內的 elements 要刪除

* 由上面得知我們會需要把 string element 一個一個複製到新的空間
* 因為 string 是 valuelike type，copy 之後會有兩份獨立的 data，會有兩個 user(這裡是指 code，或者說兩個變數)使用(地址)不同但內容相同的物件
* **可是你仔細想想，我們雖然一定要把舊 string elements copy 到新的空間，可是一旦複製完之後，舊的資料馬上就會被刪除了，不會有人使用**，換句話說 user 一直都只有一個
* **如果有什麼方法可以避免這個 string 的 copy，效能會更好**

#### Move Constructors and std::move
* **終於要來介紹 C++11 的一個超狂新 feature 啦!**
* 12.6 才會詳細講，不過這裡先知道一點概念
* STL container 幾乎都有定義 move constructor
* Move constructors typically operate by **"moving" resources from the given object to the object being constructed.**
* The library guarantees that **the "moved-from" string remains in a valid, *destructible state*.**
    * 哇靠，destructible state!

* 可以想像 std::string 有一個 `char*` 指向配置的 C string，然後 **move cstr 會複製指標而不是複製指標指向的 C string**
* 除此之外還需要用到 `std::move`，定義在 `<utility>`
* 總之現在要知道的是，要對要被複製的 `string` 用 `std::move`，這樣 compiler 才會知道需要呼叫 `string` 的 move constructor，其詳細原理 13.6 會解釋
* 還有，不要對 `move` 用 `using`，理由 18.2 會解釋...
* 總之來看 `reallocate` 怎麼寫吧!

#### The reallocate Member
* 首先會配置空間
    * 配置一個比舊空間大一倍的空間
    * 如果原本的 `StrVec` 是 empty，那就配置大小為 1 個 string 的空間
    ```C++
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
    * 注意那個 `std::move` !!
    * `std::move(*elem++)` 回傳的 result 會讓 **function matching 選擇 move constructor**
    * STL container 被 move 後保證呼叫 destructor 是安全的
    * 要完空間，複製完舊字串之後，最後更新 `StrVec` 的三個指標

## 13.6 Moving Objects
* 我們在 13.1.1 看到，有非常多的情況會 copy 物件
    * 但這其中又有很多時候，被 copy 的物件馬上就會被破壞
    * 這時候用**move 而不是 copy**，效能會提升

* 舉例:
    * container 在重新配置更大的空間時，可以 move elements 而不是 copy elements，因為舊的空間沒人會再用了
    * 某些**只會有一個 user 使用的 class**，例如 unique_ptr 或者 IO class，他們的底層其實都有一個**不能被共享的指標**，這些 classes 因此被規定不能被 copy 但是可以被 move
        * 度ㄉ，IO 跟 unique_ptr 可以 move...

* 在舊標準裡沒有 move 這種東西，一定要用 copy，這代表會有不必要的複製
* 還有另一個缺點，STL container element type 在舊標準裡一定要 copyable，**在C++11，element type 只要可以 move 就行了**

### 13.6.1 Rvalue References
* 超重要的新功能
* 為了要支援 move operation，C++11 定義一種新的 reference 方式，**rvalue reference**
* **An rvalue reference(以下簡稱 rv ref) is a reference that must be bound to an rvalue.**
    * obtained by using && rather than &

* 之後會看到，rv ref 就是拿來 bind 那些準備要被破壞的物件
    * 因為他準備要被破壞，所以**我們可以把這個物件的內容 move 到其他物件**

* 所以要懂 rv ref 在幹嘛之前要先知道什麼是 rvalue，在 4.1.1
    * lval 跟 rval 是 expression 的 properties
    * 某些 exp 產生或需要 lval，其他則產生或需要 rval 
    * 一般來說，lval 代表的是一個物件的"身分"，rval 代表的是一個物件的"值"
    * 之前都說通常可以把 lval exp 拿來放在需要 rval exp 的地方，**需要呼叫 move constructor 的地方就是例外**

* Like any reference, an rvalue reference is just another name for an object.
* 之前也知道，我們沒辦法 bind regular reference，也就是 lvalue reference，到:
    * expressions that require a conversion
    * literals
    * 或者 expressions that return a rvalue
    * 注意 const reference 還是可以 bind 上面的東西，不過概念上不一樣，請去看第四章

* rv ref 則是遵循一個互補的規則，它可以 bind 上面這些東西，但卻**不能直接** bind lv ref 可以 bind 的東西:
    ```C++
    int i = 42;
    int &r = i; // ok: r refers to i
    int &&rr = i; // error: cannot bind an rvalue reference to an lvalue
    int &r2 = i * 42;  // error: i*42 is an rvalue
    const int &r3 = i * 42; // ok: we can bind a reference to const to an rvalue
    int &&rr2 = i * 42; // ok: bind rr2 to the result of the multiplication
    ```
    * 注意 const lv ref 一樣可以 bind rvalue，不過跟用 rv ref bind rvalue 不一樣
        * 至少 rv ref 還是可以改變 bind 的物件，const lv ref 就不型

* lval 更具體的例子如下:
    * Functions that return lvalue references
    * the assignment
    * subscript
    * dereference
    * and prefix increment/decrement operators
* 我們可以用 lval reference bind 上面的東西

* rval 更具體的例子如下:
    * Functions that return a nonreference type
    * along with the arithmetic,
    * relational,
    * bitwise,
    * and postfix increment/decrement operators, 
* 我們不能用 lv ref 綁上面的東西，可是可以用 lv ref to const 或 rv ref 來綁

#### Lvalues Persist; Rvalues Are Ephemeral
* **Lvalues have persistent state, whereas rvalues are either literals or temporary objects created in the course of evaluating expressions.**
* 因為 rv ref 只能綁 temporary objects，我們知道:
    * 被綁定的物件很快就會被破壞
    * 不會有其他 user 使用到這個物件
* 上面代表，code that uses rv ref，**可以把這個物件的 resources *搶過來***
* Note: Rvalue references refer to objects that are about to be destroyed. Hence, we can ***"steal"*** state from an object bound to an rvalue reference.

#### Variables Are Lvalues
* 雖然這樣想很ㄎㄧㄤ，不過一個 variable name 其實就是一個 exp，只有一個 operand，然後沒有 operator
    * 還記得 decltype 內把一個 variable 直接取 type 跟 (variable) 取 type，一個是 reference 一個不是ㄇ

* variable expression 也有 r/lvalue property
* variable expression 是 lvalue
* **因為如此，所以我們不能用一個 rv ref 來 bind 一個 variable**
    ```C++
    int &&rr1 = 42; // ok: literals are rvalues 
    int &&rr2 = rr1; // error: the expression rr1is an lvalue!
    ```
    * 注意上面的 code 是想把 rr2 bind 到 rr1，即使 rr1 是 rv ref，它還是個 variable，所以身為 rv ref 的 rr2 不能拿來綁 rr1!!

#### The Library move Function
* 雖然 rv ref 不能直接綁 variable，但我們可以 explicitly cast variable to 對應的 rvalue reference type
    * 不知道這在工三小... 用 static_cast??
* 我們也可以用 std::move 來得到對硬的 rvalue reference type
    ```C++
    int &&rr3 = std::move(rr1); // ok
    ```
    * 這 function 是用一個 16章會介紹的功能來達成回傳 rvalue reference 的
    * 呼叫 `std::move` 告訴 compiler 我們想把一個 lvalue 當作 rvalue 來用

* It is essential to realize that the call to move promises that we do not intend to use rr1 again except to assign to it or to destroy it.
    * 這個概念非常重要，你要假設某個 lvalue 被當作 `std::move` 的參數之後，你不能假設它擁有什麼 value，它的 value 是 *unspecified*，除非你重新對他做 assign
    * 總之呼叫了 move 之後的 object 只能做兩種操作，assign 或者直接破壞

* 這裡再提醒一次，請用 `std::move` 不要用 `using` 配上 `move`，原因 18章會解釋

### 13.6.2 Move Constructor and Move Assignment
* move cstr 第一個參數要是 class_name&&，也就是一個 rvalue reference；如果有額外參數，全部都要有 default arguments
* 注意，你的 move cstr 不光是把 rv ref 綁定的物件的 resources 搶過來而已，你還要讓 rv ref 綁定的的物件維持在一個 state，可以直接將這個物件破壞而不會出事
    * 概念上來說，一旦 resources 被搶過去，這個 rv ref 綁定的物件就不該再"指向"這些 resources
    * —responsibility for those resources has been assumed by the newly created object.
        * 維護這些 resources 的工作已經交給呼叫 move cstr 的物件了

* 用 `StrVec` 來舉例:
    ```C++
    StrVec::StrVec(StrVec &&s) noexcept // move won’t throw any exceptions 
    // member initializers take over the resources in s 
    : elements(s.elements), first_free(s.first_free), cap(s.cap)
    {
        // leave s in a state in which it is safe to run the destructor
        s.elements = s.first_free = s.cap = nullptr;
    }
    ```
    * 之後會講 noexcept 在幹嘛
    * 首先，move cstr 沒有配置新空間
        * 相反的，它把 s 配置的空間"搶過來"
    * 然後把 `s` 的指標設成 `nullptr`
    * 這樣 `s` 準備被破壞時，呼叫 dstr 才會是合法的
    * 如果沒有設定 `s` 的指標，那呼叫 dstr, which calls `free()`，就會把剛搶過來的空間給 deallocte

#### Move Operations, Library Containers, and Exceptions
* 因為 move cstr 是搶資源，沒有額外配置新空間，一般來說 move cstr 不會 throw exception，
* 如果我們確定我們寫的 move cstr 不會 throw，我們*必須告知 library(或 compiler)*
* 之後會看到，除非我們有"告知" lib 我們的 move 不會 throw，不然 lib 會假設我們的 move 可能會 throw 並且做一些額外處理
* 告知不會 throw 的方法之一是將 move cstr 宣告成 `noexcept`
    * 18.4 會講 `noexcept` 的細節，現在只要知道 noexcept 是要告知被宣告的 function 不會 throw
    ```C++
    class StrVec {
    public:
        StrVec(StrVec&&) noexcept; // move constructor
        // other members as before
    };
    StrVec::StrVec(StrVec &&s) noexcept : /* member initializers */
    {/* constructor body */}
    ```
    * 放在 parameter list 之後，cstr_init_list 之前
    * 宣告跟定義 cstr 都要寫 `noexcept`

* 請看 Primer p.536，說明為什麼要宣告成 `noexcept`
* 總之，你必須宣告成 `noexcept` 才能在某些特定情況下用 move cstr，否則 standard lib 都會用 copy cstr

#### Move-Assignment Operator
* 套路跟 copy-assignment 很像，move-assignment 做的就是結合 move 跟 dstr 做的
* 跟 move cstr 一樣，如果它不會 throw，必須宣告成 noexcept
* 也要確保 self-assignment...
    * 等等 temporary 怎麼會有 self assignment?!
    * 有地! `lol = std::move(lol);` 雖然很ㄎㄧㄤ，不過這樣就會出事
    ```C++
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
* 說了 N 遍，被 move 過的物件呼叫 dstr 必須是安全的
* 除此之外，物件需要*維持一個 valid 狀態*
    * valid 的意思是，對他做:
        * assignment
        * 其他*不會參考到它的 value 的* operations
    * 必須也要是安全的
* move 後的物件不需要維持原本的 value，user 也不可以假設它有任何的 value
* 例如你可以 move STL container，然後對它呼叫 `size()` 或者 `empty`，**不會爆炸，可是 value 是未定義**

* 而如果拿我們寫的 `StrVec` 舉例，我把 move `StrVec` 物件之後，它會維持跟 default initialize 一樣的狀態(三個指標都 == `nullptr`)，所以對它做任何操作所得到的結果就會跟一個 default initialize 的 `StrVec` 物件一樣
    * 但是更複雜的 class 例如 STL container 就不保證 move 後的結果會怎樣

#### The Synthesized Move Operations
* compiler 也會合成 move cstr 跟 move assign
* 當是何時會合成他們的條件跟 copy 很不一樣
* 還記得 copy-cstr 跟 copy-assign，只要你沒有定義，compiler 一定會合成一個，合成出來的要馬 memberwise copy/assign，要馬是 delete function

* 但對於某些 class，compiler move 則是連合都不合ㄌ
* "不合成的條件": 如果 class 只要 copy cstr，copy assign，dstr，三個裡面**只要有一個有定義**，那 compiler 就不會合成 move cstr 跟 move assign
* 所以有些 class 的確會沒有 move cstr 或 move assign 的(但每個 class 一定都會有上面那三個)

* 那如果沒有 move，當需要用到 move 的 context 會用什麼操作呢?
    * 就直接用對應的 copy 操作

* 合成 move 的條件:
    * 上面說的那三個操作都沒有自定義
    * 所有的 nonstatic data member 都可以 move
        * built-in 可以 move
        * class object，有定義 move 的話也可以 move
    ```C++
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
    
* compiler 如果合成 move，compiler 絕對不會主動把它定義成 delete function
* 但是如果我們明確的用 `= default'` 叫 compiler 合成 move，但是 class 卻不能 move 時，那 compiler 就會把 move 定義成 delete function
* 所以你用 default 叫 compiler 合成 move，而 move 何時會被定義成 `delete` 跟 copy 何時會被定義成 delete 有點類似，*除了一個重要例外*:
    * move constructor: 如果 class 有 member 有定義 copy cstr 但是沒有定義 move cstr，或者，有 member 沒有定義 copy cstr，但是它還是不能用 move(會發生這種情況就是這個 member 有 member 不能 move，或者主動宣告成 delete)，那 class 的 move 就會宣告成 delete function
    * move assignment: 跟 move constructor 很像
    * 如果 class 有 member 的 move cstr 或者 move assign 如果定義成 delete function 或者不可訪問(inaccessible)，這樣 class 的 move cstr/assign 就會定義 delete function
    * 如果 dstr 定義成 delete 或者 dstr inaccesible，那 move cstr 就會定義成 delete function
    * 如果 class 有 member 是 const 或者 reference，那 move assign 就會定義成 delete function

* 舉例...
    * 假設 `Y` 是一個有自訂 copy cstr 但是沒定義 move cstr 的 class:
## 這裡 Primer 寫錯了
* 假設 `Y` 是一個有自訂 copy cstr 但是 move cstr 宣告為 delete 的 class，這樣才對:
    ```C++
    // assume Y is a class that defines its own copy constructor "but not a move constructor" ERROR
    struct hasY {
        hasY() = default;
        hasY(hasY&&) = default;
        Y mem; // hasY will have a deleted move constructor
    };
    hasY hy, hy2 = std::move(hy); // error: move constructor is deleted
    ```
    * 這時候 `hasY` 的 move cstr 宣告成 `default`，這樣 compiler 就會把它定義成 delete function
    * `hy2 = std::move(hy)` 就會使用到 move cstr，但是是 `delete`，所以噴 error

* 最後一個 move member 跟 copy member 之間的關係:
* 如果 class 有定義 move cstr and/or move assign，那 compiler 合成的 copy cstr/assign 就會是 delete function
* 換句話說，**定義了 move cstr or move assign 的 class 一定要自訂 copy cstr/assignment，否則 compiler 就會合成 delete function**

#### Rvalues Are Moved, Lvalues Are Copied ...

* 如果 class 同時擁有 copy cstr 跟 move cstr，那 compiler 就用一般的 function matching 來決定要用哪個
* assignment 相同
* 例如 `StrVec` 的 copy cstr 是吃 `const StrVec&`，它可以 match 所有可以轉成 `StrVec` 的 argument
* 而 `StrVec` 的 move cstr 是吃 `StrVec&&`，只有 argument 是 nonconst rvalue 時才會使用
    ```C++
    StrVec v1, v2;
    v1 = v2; // v2 is an lvalue; copy assignment
    StrVec getVec(istream &); // getVec returns an rvalue
    v2 = getVec(cin); // getVec(cin) is an rvalue; move assignment
    ```
    * `v1 = v2;`，move assign，不是 viable function(不知道這是啥請去看第六章 function matching)，因為不能把 rv ref 綁到一個 lvalue，所以 move assign 根本不會考慮，於是使用 copy assign
    * 而 `v2 = getVec(cin);`，copy 跟 move assign 都是 viable，因為 `getVec(cin)` 是 rvalue
        * 如果要用 copy assign，需要做一次 const 轉換(記得它是 function matching 規則不知道第幾順位來著)，而用 move assign 則是 exact match，所以使用 move assign

#### ...But Rvalues Are Copied If There Is No Move Constructor
* 上面說的是 move/copy 都定義的情況，但如果 class 只定義 copy 沒有定義 move 呢?
* 那就直接用對應 copy member，即使我們強制用 `std::move` 傳一個 rvalue reference 也一樣
    ```C++
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
    * The call to `std::move(x)` in the initialization of z returns a Foo&& bound to x.
    * The copy constructor for Foo is *viable because we can convert a Foo&& to a const Foo&.*
    * 所以 z 會用 copy cstr

* 值得注意的是，用 copy cstr/assign 來帶去 move cstr/assign 幾乎是安全的

#### Copy-and-Swap Assignment Operators and Move
* 以下超屌超ㄎㄧㄤ
* 記得我們的 `HasPtr` 的 `operator=` 不是吃 reference，而是吃 value，我們還用這個來時做 copy-and-swap
* **如果這時我們再定義 move cstr，那這個 `operator=` 就同時會滿足 copy assign 跟 move assign! 超屌!**
    ```C++
    class HasPtr {
    public: // added move constructor 
        HasPtr(HasPtr &&p) noexcept : ps(p.ps), i(p.i) {p.ps = 0;}
        // assignment operator is both the move- and copy-assignment operator
        HasPtr& operator=(HasPtr rhs) 
            { swap(*this, rhs); return *this; }
        // other members as in § 13.2.1 (p. 511)
    };
    ```
    * 因為 `operator=` 的 argument 是 call by value，所以會呼叫 cstr 複製一份，**而要呼叫 copy 還是 move cstr 就取決於 argument 是 l/rvalue**
    * lvalue 就呼叫 copy cstr，rvalue 就呼叫 move cstr
    * 所以到底要 move 還是 copy，在 argument passing 階段就解決了，所以這個 `operator=` 可以同時當 copy assign 也可以當 move assign
    ```C++
    hp = hp2; // hp2 is an lvalue; copy constructor used to copy hp2
    hp = std::move(hp2); // move constructor moves hp2
    ```
    * `hp = hp2;`，`operator=` 在初始化 parameter 時，move cstr not viable，所以用 copy cstr
    * `hp = std::move(hp2);`，move cstr 跟 copy cstr 都 viable，move 是 exact match，用 move
    * copy assignment 時
        ![](https://i.imgur.com/Yhkhvef.png)
    * move assignment 時
        ![](https://i.imgur.com/irCTFyT.png)
        
* ADVICE: UPDATING THE RULE OF THREE(to FIVE)
    * 已經說了要把那三個當成一個 unit 來看待，定義其中一個代表其他的也要定義
    * 而通常**一定**要定義這三個的，也會自己額外配置資源，這樣的 class，通常定義另外兩個 move 效能會更好，節省不必要的動態空間配置

#### Move Operations for the Message Class
* 幹又是你..........
* 我們的 `Message` 跟 `Folder` 一樣可以因為定義 move 讓效能變好
* 可以對 data member，`string` 跟 `set` 用 move
* 可是還記得一個 `Message` 有一個 `folders` member ㄇ...，那是拿來記住 他所處的所有資料夾的，如果 move，必須把這個 member 內存的指標都設對...，`Folder` 的情況也是類似
* 因為 `Message` 的 move cstr 跟 move assign，都需要改 `folders` member，所以定義成一個 common private utility:
    ```C++
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
    * 注意這個 function 只是 move cstr/assign 會呼叫的 utility，並不是 move cstr/assign 本身
    * 還有，因為這個 utility 還是會額外配置空間(`f->addMsg(this);`)，所以可能會 throw，**這意味著呼叫這個 utility 的 move cstr/assign 不能宣告成** `noexcept`，不像之前的 `HasPtr` 一樣
    * 主要做的就是把原本所有 `Folder`s 內指向 rhs 的指標都拔掉，然後新增指向 lhs 的指標
    * 最後把 `folders` 給 clear，保證 `Message` 的 dstr 在把 `folders` 掃過一遍時它是空的
    * ![](https://i.imgur.com/jBXxlf7.png)
    * 注意你的 `folders` 裡面只是存指標，把 `folders` clear 不會影響到它們指向的東西

* 於是 move cstr 就可以這樣寫:
    ```C++
    Message::Message(Message &&m): contents(std::move(m.contents)) 
    {
        move_Folders(&m); // moves foldersand updates the Folderpointers
    }
    ```
    * `contents` 是 `string`，直接用 `string` 的 move cstr；然後 `this->folders` 用 default cstr，之後才會用 `move_Folders` 把 `this->folders` 在 `move_Folders` 用 move assign 給值(`folders = std::move(m->folders);`)
* move assign 就這樣寫:
    ```C++
    Message& Message::operator=(Message &&rhs) {
        if (this != &rhs) { // direct check for self-assignment
            remove_from_Folders();
            contents = std::move(rhs.contents); // move assignment
            move_Folders(&rhs); // reset the Folders to point to this Message 
        }
        return *this;
    }
    ```
    * 一樣直接確認 self-assign
    * 先把所有 `Folder`s 內指向 lhs 的指標移除，然後把 `this->folders` 清空(也就是 `remove_from_Folders` 做的事)
    * 之後把 rhs 的 `contents` move 過來
    * 最後也呼叫 `move_Folders`，把 set move 過來，把所有 `Folders` 內原本指向 rhs 的指標改成指向 lhs

* 反正上面的 `Message` 例子就是一堆指標要刪除或更改，看 code & 寫練習的時候要想清楚

#### Move Iterators
* 換看 `StrVec`
* `reallocate` 是用 iterator 的方式掃過(用 allocate)新配置的空間，然後一個一個 construct
    * ![](https://i.imgur.com/VnyXfzR.png)
* 其實轉換一下可以用 `uninitiailized_copy` 來做
* 但如字面意思，`uninitiailized_copy` 真的是用 copy 的，C++11 沒有什麼 uniniailized_move 這種東西
    * 注意! C++17(ry) 有 uninitialized_move !!
    * http://en.cppreference.com/w/cpp/memory/uninitialized_move
* 不過 C++11 定義了 move iterator
    * C++14/17 也有對這東西額外做擴充
    * http://en.cppreference.com/w/cpp/iterator/make_move_iterator
* 這 iterator 本質上就是把 \*(dereference) 這個操作改變，原本的 iterator 是回傳 lvalue reference，而 move iterator 則是回傳 rvalue reference
* 用 `make_move_iterator` 把一個普通 iter 轉成 move_iter:
* 然後來改寫 `StrVec::reallocate`:
    ```C++
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
    * 這樣 `reallocate` 就會把舊 elements 直接 move 到新空間而不是 copy 到新空間惹
    * 這裡雖然 `uninitialized_copy` 是想要"複製" element 到 `first`，但其實 `uninitialized_copy` 做的只是把 range 內的 elements 丟給 cstr 罷了，而因為我們是丟一個 rv ref to range 內的 elements，所以 function matching 就會選擇 move cstr 而不是 copy cstr。
    * 然後 `uninitialized_move` 怎麼用請去看 cppreference

* 然後那些原本吃一般 iterator 的 STL lib，你要確定他們不會 access 被 move 過的物件的 value 才能用 move_iter，否則會出事
    * you should pass move iterators to algorithms only when you are *confident* that the algorithm does not access an element after it has assigned to that element or passed that element to a user-defined function.

* ADVICE:DON’T BE TOO QUICK TO MOVE
    * move 沒有你想像中的好用
    * STL lib 裡面有大師級的 code 在 move，效能狂飆
    * 可是你只是個廢物 coder，常常沒有獲得效能，反而弄出一堆機掰 bug
    * 寫 move cstr 時，只有確定你一定需要 move，而且保證 move 之後的 object 是安全的
        * 所謂安全就是，move 之後 no user 會使用那個 object 的值

### 13.6.3 Rvalue References and Member Functions
* 你的 class member 除了 copy-control 的那五個以外，如果也有提供 move 版本的通常也會受惠
* 同名字的 member function 你就 overload 兩種，一種是 reference to const，一種是 rvalue reference to nonconst，這個 pattern 跟 copy/move cstr 給的參數一樣

* 例如 C++11 的 STL container，有提供 push_back 的就有兩種版本:
    ```C++
    void push_back(const X&); // copy: binds to any kind of X
    void push_back(X&&); // move: binds only to modifiable rvalues of type X
    ```
* `const X&` 可以接受任何可以轉換成 X 的參數，包括 modifiable rvalue
* `X&&` 只能接受 modifiable rvalue
* 不過當參數真的是 modifiable rvalue 時，`X&&` 是 exact match，`const X&` 還要做一次 const 轉換，所以 function matching 會選擇 `X&&` 的版本
* 反正你傳進來的只要是 temp object 就會用 `X&&` 的版本
* 下面給個例子:
    ```C++
    class foo {
    public:
     foo() { cout <<   "default ctor\\n"; }
     foo(int) { cout <<   "convert from type convertible to int\\n"; }
     foo(const foo &f) { cout <<   "copy ctor\\n"; }
     foo(foo &&f) noexcept { cout <<   "move ctor\\n"; }
    };
    int main() {
        vector<foo> vec;
        int i =   10;
        double d =   87.0;
        vec.reserve(10);
        foo f;
        cout <<   "----------\\n";
        vec.push_back(f); // copy
        cout <<   "----------\\n";
        vec.push_back(10); // 10 convert to temp foo, using push_back(T&&);
        cout <<   "----------\\n";
        vec.push_back(i); // i convert to temp foo, using push_back(T&&);
        cout <<   "----------\\n";
        vec.push_back(
        d); // d narrow down to int and convert to temp foo, using push_back(T&&)
        cout <<   "----------\\n";
        vec.push_back(foo()); // using push_back(T&&) directly
    }
    ```
    
* 另一個例子是對 `StrVec` 增加另一個版本的 push_back:
    ```C++
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
    * 注意看 && 的版本，呼叫 `construct` 時要給 `std::move(s)` 當參數。再提醒一次，雖然 s 是 string&&，可是直接用它時還是會把它當成 lvalue，一定要用 move 取得 rv ref
* 使用 push_back 時就會這樣 resolve:
    ```C++
    StrVec vec; // empty StrVec 
    string s = "some string or another";     
    vec.push_back(s); // calls push_back(const string&)
    vec.push_back("done"); // calls push_back(string&&)
    ```
    * These calls differ as to whether the argument is an lvalue (s)or an rvalue*(the temporary string created from "done")*

#### Rvalue and Lvalue Reference Member Functions
* 我們可以寫這樣的 code:
    ```C++
    string s1 = "a value", s2 = "another";
    auto n = (s1 + s2).find(’a’);
    ```
    對 `s1 + s2` 這個 temp object 呼叫 find
* 可是我們也可以寫這樣很奇葩的 code:
    ```C++
    s1 + s2 = "wow!";
    ```
    * 看起來很機掰，不過你只要想成 `operator=` 也是 member function 就會比較好理解
    ```C++
    int i, j;
    i + j = 87;
    ```
    * 而上面的 assign 不是 member function，所以會噴 error

* 在新標準提供了一種方法防止我們對 rvalue 做 assign
* 不過在舊標準已經訂出來的 class，因為 backward compatible，在 C++11 還是允許對他們的 rvalue 做 assign
* 但我們自己寫的 class 可以用這種新功能，防止這種ㄎㄧㄤ行為
    ```C++
    class Foo {
    public:
        Foo &operator=(const Foo&) &; // may assign only to modifiable lvalues
        // other members ofFoo
    };
    Foo &Foo::operator=(const Foo &rhs) & {
        // do whatever is needed to assign rhs to this object 
        return *this;
    }
    ```
* 有沒有看到那個 & ?
* 那叫做 **reference qualifier**，有兩種，& 和 &&
* **indicating that this may point to an rvalue or lvalue, respectively.**
* 它跟 const qualifier 一樣，只能出現在 nonstatic member function，並且宣告跟定義都要寫出來
* 上了 & 的話，只有 lvalue object 可以使用這個 member function；&& 則是 rvalue
    ```C++
    Foo &retFoo(); // returns a reference; a call to retFoo is an lvalue
    Foo retVal(); // returns by value; a call to retVal is an rvalue
    Foo i, j; // i and j are lvalues
    i = j; // ok: i is an lvalue
    retFoo() = j; // ok: retFoo() returns an lvalue
    retVal() = j; // error: retVal() returns an rvalue
    i = retVal(); // ok: we can pass an rvalue as the right-hand operand to assignment
    ```
    * 如果 `Foo` 在 `operator=(const Foo&)` 加上 &，`retVal() = j;` 就會噴 error，因為不能對 retVal() 回傳的 rvalue 用 `operator=`

* 一個 member function 可以同時用 const 跟 ref qualified，如果要同時用，要先寫 const 再寫 ref qualifier:
    ```C++
    class Foo {
    public:
        Foo someMem() & const; // error: const qualifier must come first
        Foo anotherMem() const &; // ok: constqualifier comes first
    };
    ```
    
#### Overloading and Reference Functions
* 我們可以直接用 const 來 overloading member function，我們也可以用 reference qualifier 來 overload
* 更甚者，這兩個東西還可以組合...
* 我們給 `Foo` 加一個 vector member，再配上一個 `sorted` function，回傳一個 vector 已經 sort 過的 `Foo` object
* 根據 reference 跟 const 的 overloading，我們可以這樣寫:
    ![](https://i.imgur.com/Dsh01AK.png)
    * 如果我們是用 rvalue 呼叫 `sorted`，那我們直接對這個 rvalue 內的 vector sort 也無妨(ㄈㄤˊ)(?
    * 如果是用 lvalue 或者 const rvalue 呼叫 sorted，因為"概念上"我們就不想 in-place sort(lvalue 是"不想"，const rvalue 是"不能")，所以我們要創一個 temp `Foo` 來 copy `*this`，然後對 temp object sort，再回傳 temp object
        * 因為不做 in place sort，所以索性把 `sorted` 宣告成 `const`

* 來看什麼 object 會呼叫哪種 `sorted`:
    ```C++
    retVal().sorted(); // retVal() is an rvalue, calls Foo::sorted()&&
    retFoo().sorted(); // retFoo() is an lvalue, calls Foo::sorted()const&
    ```

* 再來是如果你要用 ref qualifier 來 overload member function(而且 function name 跟 para list 都一樣)，你一定要所有 function 都明確寫出 ref qualifier，不然會 error:
    ```C++
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
    * 看上面的 `sorted`，如果沒給 ref qualifier，代表不管 lvalue 還是 rvalue 都可以呼叫它，那這樣的話 lvalue object 可以同時用這個 `sorted` 跟 `sorted &`，rvalue object 可以用 `sorted` 跟 `sorted &&`，都 ambiguous


