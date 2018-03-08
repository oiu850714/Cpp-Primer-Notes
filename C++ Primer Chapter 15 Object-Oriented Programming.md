# C++ Primer Chapter 15 Object-Oriented Programming
* OOP:
    * Data Abstraction(ch7),
    * inheritence and dynamic binding(ch15)

* Inheritance: They make it easier to define new classes that are similar, but not identical, to other classes
* dynamic binding: they make it easier for us to write programs that can ignore the details of how those similar types differ.

## 15.1 OOP: An Overview
* 以下一樣先告訴你，*如果做到OOP*可以怎麼寫code，15.2開始才會詳細講細節
#### Inheritance
* class hierarchy
* base class
    * define members or operations common to the types in the hierarchy
* derived class
    * derived class defines those members that are specific to the derived class itself

* 為了舉例，Primer 要改寫 bookstore program，會有多種決定書的價格的策略，有的原價賣，有的打折賣，and so on 
* 所以我們會定義 base class - `Quote`，他代表了按照原價賣的書籍；
* 再定義一個 `Bulk_quote`，derived from `Quite`，代表了打折賣的書籍
    * 話說這裡英文寫 We will inherit a second class from `Quote`，主動被動注意一下
* 他們都有以下 members:
    * `isbn()`，return 書的 ISBN，他會定義在 `Quote`，所有階層中的 classes 都用他，他是類型無關(not depend on specific type)，所有 class 都會擁有這個 member
    * `net_price(size_t)`，他會回傳你買了特定數量的書之後所需的價錢，他會根據不同的 type 而有不同的行為→ `Quote` 跟 `Bulk_quote` 都會各自定義，是一種類型相關(type specific) 的 member

* C++ 用一種方法來區分上述兩種 member，也就是類型無關跟相關的 member - `virtual`:
    ```C++
    class Quote {
    public:
        std::string isbn() const;
        virtual double net_price(std::size_t n) const;
    };
    ```
    * 被定義為 `virtual` 的 member，derived class 必須定義，沒有的要不要定義都可以

* derived class 一定要寫他要繼承的 class: **class derivation list**:
    * 寫在 class definition 中，class name 的後面，大括號的前面:
    * 用逗號分隔的一堆 class name，代表要被繼承的 classes，前面可以加 access specifier，specifier 的功能之後會細講
    ```C++
    class Bulk_quote : public Quote {
    // Bulk_quoteinherits from Quote
    public:
        double net_price(std::size_t) const override;
    };
    ```
    * 不過還是說一下，如果是 public 繼承，we can use derived classs **as if** they were base class

* derived class 也一定要在 class definition 內把全部 base class 內寫的 virtual function 都宣告；這些 function 也可以額外寫一個 virtual，代表這是 base class 指名要自定義的 member；
* 還有因為 15.3 會講的原因，我們可以在 derived class 內的 member 加上 `override`，明確指示說他要改寫 base class 的 virtual function

#### Dynamic Binding
* 可用同一份 code 同時處理 `Quote` 跟 `Bulk_quote` 物件:
    ```C++
    // calculate and print the price for the given number of copies, applying any discounts
    double print_total(ostream &os, const Quote &item, size_t n) {
        // depending on the type of the object bound to the item parameter
        // calls either Quote::net_price or Bulk_quote::net_price
        double ret = item.net_price(n);
        os << "ISBN: " << item.isbn() // calls Quote::isbn
        << " # sold: " << n << " total due: " << ret << endl;
        return ret;
    }
    ```
    * 注意參數的 reference 同時可以綁這兩種 type 的物件
        * 15.2.3 會講 reference 來綁 base 跟 derived class 物件的方法
    * 因為我們是用 reference，所以呼叫 net_price 時**會由我們綁定的物件的型別*動態*決定要呼叫 base class 的 net_price 還是 derived class 的 net_price**
        * 15.2.1 會講...
        ```C++
        // basic has type Quote; bulk has type Bulk_quote
        print_total(cout, basic, 20) // calls Quote version of net_price
        print_total(cout, bulk, 20); // calls Bulk_quote version of net_price
        ```
### In C++, dynamic binding happens when a virtual function is called through a reference (or a pointer) to a base class.

## 15.2 Defining Base and Derived Classes
* 看看該怎麼定義 base class，以及藉由這個 base class 定義 derived class
### 15.2.1 Defining a Base Class
* `Quote`:
    ```C++
    class Quote {
    public:
        Quote() = default; // = default see § 7.1.4 (p. 264)
        Quote(const std::string &book, double sales_price):
                bookNo(book), price(sales_price) { }
        std::string isbn() const { return bookNo; }
        // returns the total sales price for the specified number of items
        // derived classes will override and apply different discount algorithms
        virtual double net_price(std::size_t n) const
            { return n * price; }
        virtual ~Quote() = default; // dynamic binding for the destructor
    private:
        std::string bookNo; // ISBN number of this item
    protected:
        double price = 0.0; // normal, undiscounted price
    };
    ```
    * 新功能:
        * 定義在 `net_price` 跟 destructor 前面的 `virtual`
            * **virtual dstr** 在 15.7.1 才會講...，不過 base class 幾乎都會定義 virtual dstr
            * https://stackoverflow.com/questions/461203/when-to-use-virtual-destructors
        * `protected` access specifier

#### Member Functions and Inheritance
* derived class 會繼承 base class 的 member；但有時候我們規定 derived class 必須要重新定義 base class 的某些 member，定義成專屬於有 derived class 行為的 member，C++ 用 virtual 來實現，我們說這種重新定義的操作叫 **override**

* `virtual`:
    * 沒被定義為 `virtual` 的 member 就是 base class 希望 derived class 不要 override 的；有定義 `virtual` 的就是希望要 override 的
    * **When we call a virtual function *through a pointer or reference*, the call will be dynamically bound**
        * 用 base_class 的指標或參考指向物件，物件如果是 base_class 就會呼叫 base_class 的 member；如果是 derived class 就呼叫 derived class 的 member
* 除了 `static` member function 跟 cstr 的 member function 都可以宣告成 `virtual`
* `virtual` 也只能出現在 class definition 內，把 member function 定義在 class 外時不可以加 `virtual`
* 還有，**在 base class 定義為 virtual 的 member function，在 derived class 也會 implicitly 定義成 virtual function**
    * 換句話說某ㄍ class 的 virtual member，他所有的子孫的這個 member 都會是 virtual
* 15.3 會講更多 virtual 的細節

* Member functions that are not declared as virtual **are resolved at compile time, not run time.**
    ```C++
    class foo {
    public:
     void print() { cout << "foo!\\n"; }
    };
    class bar : public foo {
    public:
     void print() { cout << "bar!\\n"; }
    };

    int main() {
        foo f;
        bar b;
        f.print();
        b.print();
        foo &f_ref = b;
        f_ref.print();   // 這裡超重要，如果你 foo.print 沒有定義成
                         // virtual，這裡就不會發生動態綁定，一定是呼叫
                         // foo::print!!!(因為你reference 是 foo&)
                         // 你可以試試加上 virtual 跟沒加會怎樣
                         // 除此之外，這裡 compiler 可以在編譯時期就決定要 call 哪個
                         // print，不用動態才知道
        bar &b_ref = b;
        b_ref.print(); // 這裡也是靜態就知道要呼叫 bar::print
    }
    ```
    * 上面的 code 可以看這份: http://www.cs.technion.ac.il/users/yechiel/c++-faq/redefining-nonvirtuals.html
    * 的確可以 redefine nonvirtual members，**但是在 user code 看到的行為要一樣**!

#### Access Control and Inheritance
* base class 的 private member，derived class(不管用 public/private 繼承)都一樣不能 access
* 所以有 `protected`，base class 的 `protected` member，derived class 可以 access，但其他 user code 一樣不能 access
    * 看我們定義的 `Quote`，因為我們希望 derived class 自己定義 `net_price`(宣告成 virtual)，而 `net_price` 又需要 access `price`，所以`price` 宣告成 `protected`
    * 而 `bookNo` 還是需告成 `private`，原因是我們希望 derived class 一樣用 `isbn` 這個 `public` member function 來取得 ISBN

* 15.5 會講更多 `protected`

### 15.2.2 Defining a Derived Class
* 把要繼承的 base class 寫在 class derivation list 裡面
    * 記住 base class 不能是 incomplete type
* derived class 必須把所有需要 override 的 member 都宣告
    * 例如 `Bulk_quote` 就要宣告 `net_price`

    ```C++
    class Bulk_quote : public Quote { // Bulk_quote inherits from Quote 
    public:
        Bulk_quote() = default;
        Bulk_quote(const std::string&, double, std::size_t, double); 
        // overrides the base version in order to implement the bulk purchase discount policy
        double net_price(std::size_t) const override;
    private:
        std::size_t min_qty = 0; // minimum purchase for the discount to apply 
        double discount = 0.0; // fractional discount to apply
    };
    ```
    * `Bulk_quote` 繼承了 `Quote` 的 `isbn`, `bookNo` 跟 `price` member
    * 自定義了 `net_price`，還有另外兩個 member: `min_qty` 跟 `discount`，拿來算當購買一定數量的書之後要怎麼打折
* 15.5 會詳細講 class derivation list 內的 access specifier；
* For now, what's useful to know is that **the access specifier determines whether users of a derived class are allowed to know that the derived class inherits from its base class**
    * 你可以想成如果 `private` 繼承，你就不能用 base class 的指標/參考來綁 derived class 的物件

* When the derivation is public, **the public members of the base class become part of the interface of the derived class as well.**
    * In addition, **we can bind an object of a publicly derived type to a pointer or reference to the base type.**
    * 如果是 public 繼承，base class 的 API 自動變成 derived class 的一部份
    * 此外你可以拿 base class 的指標/參考去綁 derived class 的物件

* 還有，這章只會介紹 **single inheritance**，也就是只繼承一個 base class；18.3 會講繼承多個 classes 的情況...

#### Virtual Functions in the Derived Class
* 其實 derived class 沒有一定要自定義 base class 的 virtual function(除非是 pure virtual function, 之後會講)；
* 如果沒有自定義的話，那 derived class 就用原來 base class 定義的 virtual member
* 15.3 會講 C++11 的新功能 `override`，明確表示說被宣告成 `override` 的 member 要 override 繼承的 base class 內的 virtual member
    * 寫在 const 跟 reference qualifier 的後面

#### Derived-Class Objects and the Derived-to-Base Conversion
* 這裡會講怎麼*圖解*一個 derived class 的物件長怎樣:
* 想像成是兩個子物件組成的，一個子物件包含了 drived class 自定義的 non`static` member，另一個則是包含了 base class 定義的 non`static` member
* 所以 `Bulk_quote` 物件有四個 data members: `bookNo` 跟 `price` 放在 base class 的子物件，`min_qty` 跟 `discount` 放在 `Bulk_quote` 的子物件
* 不過其實**標準沒有定義物件怎麼擺放在 memory**，但你還是可以像下面這樣想像:
    * ![](https://i.imgur.com/6Mfv8Tr.png)
    * 一定包含 base 跟 derived 兩部分的子物件，只不過沒有保證會放在一起

* 如果是 public 繼承 base class，我們就可以用使用 base class 的方式來使用 derived class(as if 關係)
    * 或者說 derived class 的物件**是一種** base class
    * 可以用 base class ptr/ref 來綁 derived class 物件，講過 N 遍惹
    ```C++
    Quote item; // object of base type
    Bulk_quote bulk; // object of derived type
    Quote *p = &item; // p points to a Quote object
    p = &bulk; // p points to the Quote part of bulk
    Quote &r = bulk; // r bound to the Quote part of bulk
    ```
* This conversion is often referred to as the **derived-to-base conversion**.
* As with any other conversion, the compiler will **apply the derived-to-base conversion implicitly (§ 4.11, p. 159).**
    * 這種轉換有成本ㄇ..?

### 總之結論就是，你可以在需要 base type 的參考或指標時提供 derived type 的參考或指標，因為 derived type 物件**是一種** base type 物件

#### Derived-Class Constructors
* 那 constructor 怎麼運作? 他有兩個子物件耶?
* 直接用 base class 的 cstr 來初始化 base-class part
* Note: Each class controls how **its** members are initialized.
* The base-class part of an object is initialized, along with the data members of the derived class, **during the initialization phase of the constructor** (§ 7.5.1, p. 288).
    * 也就是在執行 cstr init list 階段時一起把 base-class part 給初始化!
    * 可以在 derived class 的 cstr init list 呼叫 base class 的 cstr!
    ```C++
    Bulk_quote(const std::string& book, double p,
            std::size_t qty, double disc) : 
            Quote(book, p), min_qty(qty), discount(disc) { 
        // as before
    };
    ```
    * cstr init list 內的 `Quote(book, p)` 把 base-class part 初始化
* 總之，先跑 base class 的 cstr(不管你有沒有在 derive 的 cstr 內明確指定 base class 的 cstr 都一樣)，然後再跑 derive cstr 的 cstr init list，最後再跑 derive cstr 的 body
    * The base class is initialized first, and then the members of the derived class are initialized in the order in which they are declared in the class.
* 如果你沒有在 derive cstr init list 寫 base class 的 cstr，則 base-class part 就會 default init! 如果你有寫的話，就由你提供給 base cstr 的參數決定要選哪個 cstr

* 注意如果 base 的 default cstr 被 delete 的情況，就跟很久以前講的規則類似，不多提

#### Using Members of the Base Class from the Derived Class
* can access public/protected member of base class:
    ```C++
    // if the specified number of items are purchased, use the discounted price 
    double Bulk_quote::net_price(size_t cnt) const {
        if (cnt >= min_qty)
            return cnt * (1 - discount) * price;
        else
            return cnt * price;
    }
    ```

* 題外話，雖然 15.6 才會講 derived class scope，不過這裡先知道 derived class scope 是 nested 在 bass class 內
    * 所以會有那些 hidden rule 之類的...

* KEY CONCEPT: RESPECTING THE BASE-CLASS INTERFACE
* ![](https://i.imgur.com/POhzRDk.png)
    * 寫三小? 尊重 base class 的 interface!
    * 當你想要在 derived class 內存取從 base class 繼承過來的 public 或 protected member 時最好使用 base class 定義的介面，而不是直接存取這些 member!
    * 這點包含了初始化 derived class 的情況，**你應該呼叫 bass class 的 cstr 來初始化繼承過來的 member(例如我們的 `Bulk_quote` 呼叫了 `Quote` 的 cstr)，而不是直接在 derived class 內直接初始化這些繼承的 member**(儘管可以這麼做)。

### Interacting with interface!

#### Inheritance and static Members
* static member 整個 hierarchy 就只會有一份
* access control 跟一般 member 相同，如果是 public，則 base 或 derived class 都可以使用:
    ```C++
    class Base {
    public:
        static void statmem();
    };
    class Derived : public Base {
        void f(const Derived&);
    };
    ```
    * 下面全部都合法:
    ```C++
    void Derived::f(const Derived &derived_obj) {
        Base::statmem(); // ok: Base defines statmem 
        Derived::statmem(); // ok: Derived inherits statmem
        // ok: derived objects can be used to access static from base     
        derived_obj.statmem(); // accessed through a Derived object
        statmem(); // accessed through this object
    }
    
#### Declarations of Derived Classes
* 如果只是要單純宣告的話，不能加上 class derivation list:
    ```C++
    class Bulk_quote : public Quote; // error: derivation list can’t appear here 
    class Bulk_quote; // ok: right way to declare a derived class
    ```
    * derivation list 只能跟 class body 一同出現

#### Classes Used as a Base Class
* 要把某個 class 當成 base class 來用的話，它必須已經定義，不能只有宣告:
    ```C++
    class Quote; // declared but not defined
    // error: Quote must be defined
    class Bulk_quote : public Quote { ... };
    ```
    * 會噴 incomplete type error

* 這也隱含了一件事: class 不能繼承自己ㄏㄏ

* base class 自己也可以是某個 base class 的 derived class:
    ```C++
    class Base { /* .. . */};
    class D1: public Base { /* .. . */};
    class D2: public D1 { /* ... */};
    ```
    * 我們說 `Base` 是 `D1` 的 **direct base**，`Base` 是 `D2` 的 **indirect base**

* 繼承可以形成一條鍊:
    * 最靠近底部的 derived class(或者按 primer 說的 most derived class)，會繼承 direct base 的 member；而這個 base 又會繼承它自己的 direct base member 一直到最頂的 indirect base 為止

#### Preventing Inheritance
* 新標準可以指定某個 class 不能被繼承: `final`
    ```C++
    class NoDerived final { /**/}; // NoDerived can’t be a base class 
    class Base { /**/};
    // Last is final; we cannot inherit from Last class
    Last final : Base { /**/}; // Last can’t be a base class
    class Bad : NoDerived { /**/}; // error: NoDerived is final 
    class Bad2 : Last { /**/}; // error: Last is final
    ```
    
### 15.2.3 Conversions and Inheritance
#### WARNING: Understanding conversions between base and derived classes is essential to understanding how object-oriented programming works in C++.
* 再 講 一 次
* 一般來說我們只能把指標或參考綁到同 type，或者可以做 const 轉換的 type
* 不過一個重要例外就是 class inheritance
* **我們可以把一個 ptr/ref to base class 綁到 derived class 物件**
* 具體來說，我們可以把 `Quote&` 綁到 `Buik_quote` 物件，也可以把 `Bulk_quote` 物件的地址 assign 給 `Quote*`
* 這個例外引出一個重點:
    * 當我們使用 ptr/ref to base class 時，我們並不知道真正指向或綁定的物件的明確型別
    * 那個物件實際型別可以是 base class 也可以是 derived class

* 附帶一提，smart pointer 也支援這種操作 LOL 噁

#### Static Type and Dynamic Type
* **重要重要重要重要重要重要重要重要重要重要重要重要**
* static type:
    * variable 宣告時的型別，或者某個 expression 的型別
    * compile time 就知道
* dynamic type:
    * 某塊記憶體所代表的物件的型別，而這個物件又被一個 exp 或 variable 所表示(The dynamic type is the type of the object in memory that the variable or expression represents.)
    * runtime 才知道

* 舉例.. 當 `print_total` 呼叫 `net_price` 時:
    ```C++
    double ret = item.net_price(n);
    ```
    * `item` 的 static type 是 `Quote&`
    * `item` 的 dynamic type 則是取決於它綁定什麼物件
    * **That type cannot be known until a call is executed at run time.**

* 然後注意，只有 ptr/ref 才會發生 static/dynamic type 不同的情況，一個不是 ptr/ref 的 expression，它的 static/dynamic type 永遠一樣
    * 直接看例子，下面的 code 是錯ㄉ
    ```C++
    Quote q;
    Bulk_quote bq = q; // error: no viable conversion from Quote to Bulk_quote
    ```
    * 從 base 轉到 derive 是錯的，因為本來就沒有這種轉法；
    * 你自細想一下，真正存在的轉法是從 derive **的指標/參考** 轉到 base **的指標/參考**，當你定義 derived class 時，從 derive 轉到 base **的指標/參考**轉換也同時被定義了:
    ```C++
    Bulk_quote bq;
    Quote q = bq;
    ```
    * 之後(...and No Conversion between Objects)會講，不過你想一下，上面其實是呼叫 copy cstr，而 cstr 是吃 reference，所以其實是使用 reference 轉換
#### There Is No Implicit Conversion from Base to Derived ...
* 你覺得「沒辦法從 base 轉換到 derive」很機掰的話就這樣想，derive obj 包含了 base subobj，所以要轉換的時候就拿這個 subobj 來轉就好了；可是 base obj 就只有 base 的 member，它沒有 derive 的 member，如果要把 base obj 轉換成 derive obj 等於是要無中生有 derive obj 的 member
* 而且這件事情不管對 ptr/ref 或者一般 expression 都成立:
    ```C++
    Quote base;
    Bulk_quote* bulkP = &base; // error: can’t convert base to derived
    Bulk_quote& bulkRef = base; // error: can’t convert base to derived
    ```
    * 不能用 derive 的 ptr/ref 來綁 base object
    * 假設上面的合法好了，那我們就可以用 `bulkP` 或者 `bulkRef` 來存取 `base` 內部存在的 member，亦即只存在於 `Bulk_quote` 但不存在於 `Quote` 的 member

* 還有。不管 base ptr/ref 綁定的物件是什麼形別，你一樣不能拿他們來 assign 給 derve ptr/ref:
    ```C++
    Bulk_quote bulk;
    Quote *itemP = &bulk; // ok: dynamic type is Bulk_quote
    Bulk_quote *bulkP = itemP; // error: can’t convert base to derived
    ```
    * 因為 compiler 無法在 compile time 知道你的 base ptr/ref 到底是不是綁定 derive obj
* compiler 只會根據 static type 來決定你寫的 code 的型別轉換是否合法，不會去管 dynamic type
    * 不過還是有很ㄎㄧㄤ的最噁心的例外: `dynamic_cast`... 噁
    * 簡單說就是跟 compiler 說我把 base 轉成 derive 是安全的，你不要雞歪

#### ...and No Conversion between Objects
* 其實沒有 derive type 到 base type 的轉換，只有 ptr/ref to derive type 到 ptr/ref to base type 的轉換
* 你要記得，我們在初始化或 assign 物件時，我們其實是在 call function，初始化時呼叫 cstr，assign 時呼叫 `operator=`
* 這些 member function 一般來說都會有一個參數**是 reference**
    * 就算是 copy and swap 那種 call by value 的 `operator=`，它的參數也會呼叫 copy cstr，而 copy cstr 一定要用 reference

* 而且 cstr 等等的不是 `virtual`，你用 base ptr/ref 綁定 derived obj，呼叫的 cstr 等等其實都會是 base::cstr(而且在 compile time 就知道了)，
* 而想當然，base::cstr 等等的不會看到 derive class 的 member
* 舉個例子:
    ```C++
    Bulk_quote bulk; // object of derived type
    Quote item(bulk); // uses the Quote::Quote(const Quote&) constructor 
    item = bulk; // calls Quote::operator=(const Quote&)
    ```
    * 15.7.2 會細講 copy-control 跟繼承的關係，現在只要知道它一樣會 memberwise 初始化/assign member 就好了
    * 當你把 derived class obj 傳給 base 的 copy cstr 時，base 的 copy cstr 只會複製 base 原本就有的 member(也就是說複製 base-class part)，**並且無視 derived class 的 member（無視 derived-class part）**；`operator=` 相似
    * 因為 `Bulk_quote` part 被無視，我們說上面 `bulk` 的 `Bulk_quote` part 被 **sliced down**

* Warning: When we initialize or assign an object of a base type from an object of a derived type, only the base-class part of the derived object is copied, **moved**, or assigned. The derived part of the object is ignored.
    * **這樣感覺 move 就超不妙的= =**

![](https://i.imgur.com/JYMWyJ3.png)

## 15.3 Virtual Functions
* 先說明，以下的 code 可以編過，其實 ch6.1.2 就有講(雖然它只是跟你說 15章會講LOL):
* ![](https://i.imgur.com/FVLQxxH.png)
* `func_not_used` 沒有被呼叫過，所以其實沒有提供 definition 也可以
* member function 或 ordinary function 都是這樣子
* **但是 virtual function 就是例外**
    * 因為我們無法在 compile time 的時候知道綁到哪個物件的 virtual function，**所以一旦宣告了 virtual function，一定也要提供定義!**(普通的 function 是用到了再提供就好)

* 不過更細的說其實是如果你有宣告 class 物件，然後那 class 有 virtual，那個 virtual 一定要提供定義
    * ![](https://i.imgur.com/v2kaBRK.png)
    * ![](https://i.imgur.com/yp3s8IW.png)

#### Calls to Virtual Functions May Be Resolved at Run Time
* 當你使用 base*& 來存取 derive obj 並且呼叫 virtual function 時，其實 compiler 會 gen code(或者說實作一種方法)**在 runtime 的時候**去確認你綁定的物件的型別，然後決定要呼叫哪個 function
    * 呼叫的 function 就是綁定的物件的 dynamic type 定義的 function ㄏㄏ

* 用 `print_total` 當例子:
* 因為 item 是 `Quote&`, 然後 `net_price` 是 `virtual`，所以呼叫的 `net_price` 版本**取決於 item 綁定的物件的 dynamic type**
    ```C++
    Quote base("0-201-82470-1", 50);
    print_total(cout, base, 10); // calls Quote::net_price
    Bulk_quote derived("0-201-82470-1", 50, 5, .19);
    print_total(cout, derived, 10); // calls Bulk_quote::net_price
    ```
* It is crucial to understand that dynamic binding happens only when a virtual function is called through a pointer or a reference.
    * Primer 大概講了 87遍...
    ```C++
    base = derived; // copies the Quote part of derived into base
    base.net_price(20); // calls Quote::net_price
    ```
    * 注意 copy 只有 copy base-class part ㄛ
    * `base.net_price(20);` 是 compile time 決定要呼叫哪個ㄛ，因為是 plain object

#### Virtual Functions in a Derived Class
* 當 derived class override `virtual` function，要不要加 `virtual` 都可以，一旦某個 function 宣告為 `virtual` 所有的 derived class，不管幾層，都會是 `virtual`
* override 的時候，parameter 跟 return type 都要一模一樣
    * 除非 return 的是 ptr/ref to types that are themselves related by inheritance
    * 假設 D 繼承 B，則如果 B 的 virtual func return B\* 或 B&，則 D 的版本可以 return  D\* 或 D&；不過前提是 conversion from D to B 是 accessible(15.5會講要怎麼決定這件事情，不過應該是 public 繼承?)

#### The final and override Specifiers
* 15.6 會說我們其實可以在 derived class 宣告一個跟 base class 的 virtual function 一樣名字的 function，這是合法的，compiler 會把這個 function 跟 virtual function 分開獨立來看，而且這個 function 也不會 override base class 的 virtual function
* 不過實務上這通常是寫錯 code，例如你 base class virtual 是宣告成 const function，結果 derived class override 時忘記加 const，這樣就沒有 override
* 以前這種 bug 簡直就是悲劇還很難找，不過 C++11 的 `override` keyword 可以解決這個問題
    * 你在 derived class 對一個 function 宣告成 `override` 代表它一定要 override base class 的 member，如果沒 override 到 compiler 就會噴 error
    ```C++
    struct B {
        virtual void f1(int) const;
        virtual void f2();
        void f3();
    };
    struct D1 : B {
        void f1(int) const override; // ok: f1 matches f1 in the base
        void f2(int) override; // error: B has no f2(int) function
        void f3() override; // error: f3 not virtual
        void f4() override; // error: B doesn’t have a function named f4
    };
    ```
    * 注意只有重新定義 virtual function 才叫做 override(**only a virtual function can be overridden**)，所以 f3 的宣告噴 error；重定義 nonvirtual function 只是把 base class 定義的 nonvirtual function 名字給 hide 起來而已(因為 derive class scope nest 在 base class scope 內)

* 另一個新標準定義的是 `final` keyword，簡單說就是不准 derived class override 這個 member:
    ```C++
    struct D2 : B { 
        // inherits f2() and f3() from B and overrides f1(int)
        void f1(int) const final; // subsequent classes can’t override     
        f1(int)
    };
    struct D3 : D2 {
        void f2(); // ok: overrides f2 inherited from the indirect base, B
        void f1(int) const; // error: D2 declared f2 as final
    };
    ```
* 個人覺得一旦一個 member 都宣告成 virtual，之後的 member 要 override 就也填上 virtual 吧... 不然還要往回追，機掰
* `final` 跟 `override` 要寫在 `const` 跟 ref qualifier 以及 trailing return 後面

#### Virtual Functions and Default Arguments
* 超爛的預期行為
* virtual function 也可以定義預設參數，不過要注意:
    * 如果你是用 base*/& 來呼叫 function，不管實際上指到的物件是 base 還是 derived type，**都只會使用 base 的 function 定義的預設參數，永遠不會用到 derived 定義的**
* 如果你以為會用 derived 定義的預設參數就GG惹
* 所以... BEST PRACTICE: Virtual functions that **have default arguments should use the *same argument values* in the base and derived classes.**

#### Circumventing the Virtual Mechanism
* 有時候我們不希望用 dynamic binding，希望用 base class 的 virtual member，可以這樣寫:
    ```C++
    // calls the version from the base class regardless of the dynamic type of baseP 
    double undiscounted = baseP->Quote::net_price(42);
    ```
    * 用 scope operator 強制使用 base class 的 member，而且這樣寫也是靜態就能知道要綁誰

* 注意: 一般來說只會在 derived class 的 member 或者 class friend 內避免 dynamic binding
* 什麼時候會想要避免 dynamic binding? 最常見的例子就是 derived class 定義的 virtual 想要呼叫 base class 的 virtual
    * 這個狀況就是 base class 的 virtual 會做所有 hierarchy 內的 class 都需要做的 common work
    * 做完之後，derived class 的 virtual 再做專屬於它的工作

* 警告: 如果你在 derived class 內的 virtual 想呼叫 base 版本的，結果忘記用 scope operator，這樣會無限遞迴

## 15.4 Abstract Base Classes
* OOP 的一種概念:
    * 定義一種 base class，並且規定這個 class 的某些 interface，derived class 一定要實作自己的版本
    * 除此之外這個 base class 不准拿來創造物件
* C++ 用 **pure virtual function** 來實作這個機制
* base class 可以在宣告 virtual function 時給 `= 0` 來說明這個 virtual function 不須定義，這樣就會變成 pure virtual function；**一旦 class 內有包含 pure virtual function，他就不能拿來宣告物件**
* 例如我們想要定義一個 abstract base class 叫做 `Disc_quote`，代表需要使用打折策略的書籍，他有幾個 member 是定義來計算總價用的，例如打折的額度(discount)跟書本數量等等，然後利用 `net_price` 這個 member function 來計算打折後的價格；
    * 此時 `Disc_quote` 就可以把 `net_price` 宣告成 pure virtual function，這樣子就不能拿 `Disc_quote` 宣告物件，然後如果 derived class 要繼承他，就**一定要**定義 `net_price`；
* 其實定義 abstract base class 就是**為了定義一個通用的且必須實作的 interface**
* *題外話，Primer p.608-609 有講比較完整的為何有定義 abstract base classes 的需求，可以看看*

* 總之先來看 `Disc_quote` 的定義，他繼承 `Quote`，並且代表一個需要用一種折扣方法來計算價格的**概念**:
    ```C++
    // class to hold the discount rate and quantity         
    // derived classes will implement pricing strategies using these data
    class Disc_quote : public Quote {
    public:
        Disc_quote() = default;
        Disc_quote(const std::string& book, double price, 
                    std::size_t qty, double disc): 
                    Quote(book, price), quantity(qty), discount(disc) { }
        double net_price(std::size_t) const = 0;
    protected:
        std::size_t quantity = 0; // purchase size for the discount to apply 
        double discount = 0.0; // fractional discount to apply
    };
    ```
    * 雖然不能用 `Disc_quote` 來初始化物件，可是他該定義的還是要定義...，例如 cstr! 之後 `Disc_quote` 的 derived class 都需要 `Disc_quote` 的 cstr(which 呼叫 `Quote` 的 csr) 來初始化 base-clas part! 這都跟之前一樣。

* 不過其實也可以為 pure virtual function 提供定義就是了，不過一定要定義在 class definition 外面(primer p. 610)

#### Classes with Pure Virtuals Are Abstract Base Classes
* A class containing (or inheriting without overridding) a pure virtual function is an abstract base class.
    * 有定義 pure virtual function 的 class，或者繼承上述 class 卻沒有 override pure virtual 的 class 都是 abstract base class

* **An abstract base class defines an interface for subsequent classes to override.**
    * **We cannot (directly) create objects of a type that is an abstract base class.**
* 總之定義了 pure virtual 的 class 不能拿來宣告物件!
    ```C++
    // Disc_quote declares pure virtual functions, which Bulk_quote will override
    Disc_quote discounted; // error: can’t define a Disc_quote object
    Bulk_quote bulk; // ok: Bulk_quote has no pure virtual functions
    ```
    
#### A Derived Class Constructor Initializes Its Direct Base Class Only
* 這時你要定義 `Bulk_quote` 可以從 `Disc_quote` 繼承而不是 `Quote`
    ```C++
    // the discount kicks in when a specified number of copies of the same book are sold
    // the discount is expressed as a fraction to use to reduce the normal price
    class Bulk_quote : public Disc_quote {
    public:
        Bulk_quote() = default;
        Bulk_quote(const std::string& book, double price, std::size_t qty, double disc):     
            Disc_quote(book, price, qty, disc) { }
        // overrides the base version to implement the bulk purchase discount policy
        double net_price(std::size_t) const override;
    };
    ```
    * 注意，`Bulk_quote` 不需要**也不能**呼叫 `Quote` 的 cstr；你只要管你的 direct base 就行了，更上層的 base 交給 direct base 處理就好
        * Our new constructor passesits arguments to the `Disc_quote` constructor. That constructor in turn runs the `Quote` constructor.
    * 而且會先從最根的 class -在這裡是 `Quote`- 的 cstr 開始 run，再往下 run-這裡是 `Disc_quote` 的 cstr，最後 run `Bulk_quote` 的 cstr

#### KEY CONCEPT: REFACTORING
* Adding Disc_quote to the Quote hierarchy is an example of refactoring.
* 在 hierarchy 內新增 class，然後把一些 operations/data 移動到新 class
* 注意 refactoring 理論上不會影響到使用這個 hierarchy 的 usercode，不過使用到的 usercode 全部都要重編

## 15.5 Access Control and Inheritance
* class 除了控制如何初始化，user code 如何 access members，還有**derived class 如何 access members**

#### `protected` Members
* A blend of private and public:
    * Like private, protected members are inaccessible to users of the class.
    * Like public, protected members are accessible to members and friends of classes derived from this class.
* 還有一點，derived class 的 member 或者 `friend`，只能透過 derived class instance 來 access base class 的 member，詳情看下面的例子:
    ```C++
    class Base {
    protected:
        int prot_mem; // protected member
    };
    class Sneaky : public Base {
        friend void clobber(Sneaky&); // can access Sneaky::prot_mem
        friend void clobber(Base&); // can't access Base::prot_mem
        int j;
        // j is private by default
    };
    // ok: clobber can access the private and protected members in Sneaky objects
    void clobber(Sneaky &s) {
        s.j = s.prot_mem = 0;
    }
    // error: clobber can’t access the protected members in Base
    void clobber(Base &b) { b.prot_mem = 0; }
    ```
* 為什麼不能讓 derived class 的 friend 或者 derived object 本身可以直接 access base class 的 member 呢?
    * 假設可以，就會發生以下情況：你只要隨便定義一個 class，然後來繼承某個 base class，然後再這個 derived class 內宣告一狗屁的 `friend`s，這樣這接 friend 都可以直接 access base class 的 `protected` member 了，這樣其實就失去了 `protected` 的意義
    * 所以 C++ 只讓 derived class 跟它的 friend 透過 derived class object 來 access object 內部那些 base class 的 `protected` members，而不能直接 access 一個 base class object 內部的 `protected` member

* 但注意，如果你在 derived class 內的某個 member function 直接 access base class 的 member 其實還是可以的，因為 member function 都是靠 `this` 來 access 的，而這裡 `this` 的 type 是 `derived class*`

#### `public`, `private`,and `protected` Inheritance
* 對一個 derived class 從 base class 繼承來的 member 的存取權限取決於:
    * 那個 member 的 access specifier
    * 繼承 base class 時在 class derivation list 給定的 access specifier

* consider the following hierarchy:
    ```C++
    class Base {
    public:
        void pub_mem(); // public member
    protected:
        int prot_mem; // protected member
    private:
        char priv_mem; // private member
    };
    struct Pub_Derv : public Base {
        // ok: derived classes can access protected members
        int f() { return prot_mem; }
        // error: private members are inaccessible to derived classes 
        char g() { return priv_mem; }
    };
    struct Priv_Derv : private Base {
        // private derivation doesn’t affect access in the derived class
        int f1() const { return prot_mem; }
    };
    ```
    * derivation access specifier 不會影響到 derived class 如何去 access base class 的 member，不管你用 `public`/`private`/`proteced` 都一樣；真正會影響到的是 member 本身的 access specifier；
    * **derivation access specifier 影響到的是 user code 如何看待 derived class**
        * 注意這個 user code 還包含 derived class 的 derived class!!
* 按照上面說的，user code 就會長這樣:
    ```C++
    Pub_Derv d1; // members inherited from Base are public
    Priv_Derv d2; // members inherited from Base are private
    d1.pub_mem(); // ok: pub_mem is public in the derived class
    d2.pub_mem(); // error: pub_mem is private in the derived class
    ```
* 你可以想成，如果從 user code 的角度來看，如果一個 derived class 是 public 繼承 base class，那麼 derived class **是一種** base class；但是如果用 private 繼承，derived class **就不是** base class
* 或者是想成，private 繼承後，對 user code 來說，不管原本 base class 的 members 是 public/etc，現在全部都變成 private

* 上面有說過 user code 包含 base class 的孫子(?)，以下來看例子:
    ```C++
    struct Derived_from_Public : public Pub_Derv { 
        // ok: Base::prot_mem remains protected in Pub_Derv
        int use_base() { return prot_mem; }
    };
    struct Derived_from_Private : public Priv_Derv { 
        // error: Base::prot_mem is private in Priv_Derv
        int use_base() { return prot_mem; }
    };
    ```
    * 這裡用上面說的，members 全部變成 private 的思路來想就很好理解：任何 derived class 都不能 access base class 的 private members，而最一開始的 `Base` 被 `Priv_Derv`給繼承後就把 `prot_mem` 的 specifier 改成 `private` 了，所以 `Priv_Derv` 的兒子就不能 access `prot_mem`
* 那如果 derivation access specifier 是 `protected` 呢?
    * 那 `public` members 都會變成 protected(另外兩種 members 維持原本的 specifier)，這樣一般的 user code 就不能使用 derived class 內的 base class 的 `public` members(因為變成 `protected` 了)，而 derived class 的兒子還是可以用(因為任何 derived class 都可以 access base class 的 `protected` members)

#### Accessibility of Derived-to-Base Conversion
* 那 derivation access specifier 對物件轉型的影響呢?
    * 看是什麼 user code 使用 conversion
    * 還有看 derivation access specifier 是哪種

* 假設 `D` 繼承 `B`:
    * user code 只有在 `D` public 繼承 `B` 的情況下才能使用 `D` 轉 `B` 的轉型
    * `D` 的 members 跟 `friend`s 不管 `D` 如何繼承 `B` 都可以使用 `D` 轉 `B` 的轉型
    * 如果有 class `X` 繼承 `D`，則 `X` 的 members 或 `friend`s，在 `D` `public`/`protected` 繼承 `B` 的情況下可以使用 `D` 到 `B` 的轉換，`D` `private` 繼承 `B` 的話則不行

* Tip: 你可以這樣看，如果任何 user code，base class 的 `public` members 可以 access，則 user code 可以使用轉型，否則就不能使用

#### KEY CONCEPT: CLASS DESIGN AND PROTECTED MEMBERS
* 在沒有繼承的概念下我們可以把 user code 分成兩類: 一般 user 跟 implementors；
    * 一般 user 只能用 class 的 interface，也就是 `public` members
    * implementors 則是要寫 class member 跟 friend 的 code，這些 code 可以 access `private` members
* 而有了繼承的概念就出現第三種 user: derived class
    * base class 應該要把那些可以給 derived class 使用的 member 宣告成 `protected`

* 這樣以後 class 寫的時候就要分兩大部分，一個是給一般 user 用的 interface，一個是實作部分
    * 實作部分再分為兩類，一個是準備給 derived class 使用的部分，一個是專屬這個 class 的實作，derived class 也不能直接碰；分別對應到 `public` 跟 `private`

#### Friendship and Inheritance
* friendship 沒有 transitive(p. 279)，friendship 也不會繼承
    * 複習，B 是 A 朋友，C 是 B 的朋友，C 不一定是 A 的朋友
    * B 繼承 A，C 是 A 的朋友，C 不一定是 B 的朋友
* base 的 friend 沒有對 derived 有額外的存取權限；derived 的 friend 也沒有對 base 的額外存取權限:
    ```C++
    class Base { 
        // added friendd eclaration; other members as before
        friend class Pal; // Pal has no access to classes derived from Base
    };
    class Pal {
    public:
        int f(Base b) { return b.prot_mem; } // ok: Pal is a friend of Base
        int f2(Sneaky s) { return s.j; } // error: Pal not friend of Sneaky 
        // access to a base class is controlled by the base class, even inside a derived object
        int f3(Sneaky s) { return s.prot_mem; } // ok: Pal is a friend
    };
    ```
    * 注意 `int j` 是 `Sneaky` 的 `private` member，所以 `Pal` 當然不能 access，因為 `Pal` 不是 `Sneaky` 的 `friend`
    * 注意 `f3`，`Pal` 是 `Base` 的 friend，這意味著 **`Pal` 可以 access `Base` 的 derived class 內屬於 `Base` 的 members**
* 還有 friend 如果是 class，它的 base 或者 derived class 也不會有額外的存取權限；以下讓 `D2` 繼承 `Pal`:
    ```C++
    // D2 has no access to protected or private members in Base
    class D2 : public Pal {
    public:
        int mem(Base b) { return b.prot_mem; } // error: friendship doesn't inherit
    };
    ```
* 總結上面兩點的差異:
    * 一個是說某 A 就算是 B 的 friend，也不會跟 B 的 base 或 derived 是 friend
    * 一個是說 A 就算是 B friend，A 的 base 或 derived 也不會跟 B 是 friend

#### Exempting(免除?) Individual Members
* 拿來更改個別 member 權限用的
* 看例子:
    ```C++
    class Base {
    public:
        std::size_t size() const { return n; }
    protected:
        std::size_t n;
    };
    class Derived : private Base { // note: private inheritance
    public:
        // maintain access levels for members related to the size of the object
        using Base::size;
    protected:
        using Base::n;
    };
    ```
    * 注意 `Derived` 是 private 繼承 `Base`，所以原本在 `Base` 內的 `public`/`private` member 都會變成 `private`；
    * 於是用 using 來改變權限，這樣 `Derived` 內有關 `Base` 的 `size` 的 interface 就可以繼續讓一般 user code 使用

* 注意 using 只可以用在 direct/indirect base 的 `public`/`protected` members 上，換句話說就是 derived class 原本就可以 access 的 members 上
    * using 宣告在 `private`/`public`/`protected` 區間就會讓 member 有對應的 access 權限

#### Default Inheritance Protection Levels
* 沒啥，就只是說 class derivation list 沒給 specifier 的話預設是那種 specifier 而已
* `struct` 的話就 `public`，`class` 就 `private`
    ```C++
    class Base { /* .. . */};
    struct D1 : Base { /* .. . */}; // public inheritance by default
    class D2 : Base { /* .. . */}; // private inheritance by default
    ```
    
* NEVER USING DEFAULT SPECIFIERS，DECLARE IT EXPLICITLY

## 15.6 Class Scope under Inheritance
* Under inheritance, the scope of a derived class is nested (§ 2.2.4, p. 48) inside the scope of its base classes.
* 
* 舉例:
    ```C++
    Bulk_quote bulk; 
    cout << bulk.isbn();
    ```
    * 先從 `Bulk_quote` 找 `isbn`，沒找到；
    * 換從 `Bulk_quote` 的 base `Disc_quote` 找，沒找到；
    * 最後找 `Disc_quote` 的 base `Quote` 找，找到惹
    
#### Name Lookup Happens at Compile Time
* The static type (§ 15.2.3, p. 601) of an object, reference, or pointer determines which members of that object are visible.
    * 查找名字的*起點*是由被查找的物件(或ref/ptr)的 static type 決定的
    * 從那個 static type 的 scope 開始找，一路往 (indirect) base 找；直到找到名字為止
* 就算你是用 base ptr/ref 指向 derived 物件，亦即，static/dynamic type 不同，找名字一樣是由 static type，也就是這邊的 base，所定義的 scope 開始找

* 用例子解釋，假設我們為 `Disc_quote` 再加一個 member:
    ```C++
    class Disc_quote : public Quote {
    public:
        std::pair<size_t, double> discount_policy() const 
            { return {quantity, discount}; }
        // other members as before
    };
    ```
    * 你如果要使用 ptr/ref 來呼叫 `Disc_quote::discount_policy`，你這個 ptr/ref 只能指向 `Disc_quote` 或它的 derived 才有可能將 `Disc_quote::discount_policy` 考慮進去(也就是才有可能 visible)；
    * 如果用 `Disc_quote` 的 (indirect) base 的 ptr/ref 是不會把這個 `Disc_quote::discount_policy` 考慮進去的:
    ```C++
    Bulk_quote bulk; 
    Bulk_quote *bulkP = &bulk; // static and dynamic types are the same 
    Quote *itemP = &bulk; // static and dynamic types differ
    bulkP->discount_policy(); // ok: bulkP has type Bulk_quote*
    itemP->discount_policy(); // error: itemP has type Quote*
    ```
    * 無法使用 itemP(Quote*) 來找到 `Disc_quote::discount_policy`，就算 itemP 的 dynamic type(也就是指向的物件的 type) 是 `Disc_quote` 也一樣

#### Name Collisions and Inheritance
* 你 derived class 可以重定義一個 name，而因為 derived class 是 base class 的 nested scope，所以這個重定義的 name 會 **hides** base class 定義的 name
    ```C++
    struct Base { Base(): mem(0) { }
    protected:
        int mem;
    };
    struct Derived : Base {
        Derived(int i): mem(i) { } // initializes Derived::mem to i 
        // Base::mem is default initialized
        int get_mem() { return mem; } // returns Derived::mem
    protected: int mem; // hides mem in the base
    };
    ```
    * 注意上面 `Derived` 有重定義 `mem`；會隱藏 `Base::mem`；如果 `Derived` 沒有重定義 `mem`，那在 cstr init list 初始化 `mem` 就是錯的，不可能在 cstr 直接初始化 base 的 member
    * 下面的 code 行為:
    ```C++
    Derived d(42);
    cout << d.get_mem() << endl; // prints 42
    ```
    
#### Using the Scope Operator to Use Hidden Members
* 還是可以用 scope operator 直接使用被 hidden 的 name
    ```C++
    struct Derived : Base { 
        int get_base_mem() { return Base::mem; } 
        // ...
    };
    ```

* Best Practice: Aside from overriding inherited virtual functions, a derived class usually should not reuse names defined in its base class.
    * **除了 virtual function 以外拜託盡量不要重定義 name**

#### KEY CONCEPT: NAME LOOKUP AND INHERITANCE
* 這裡清楚講找名字跟繼承的關係，要好好看(Primer P.619)
* 寫 `p->mem() (or obj.mem())` 時要用以下步驟決定呼叫的 member 是誰:
    1. 看 p 或 obj 的 static type
    2. 從這個 static type 開始找名字，沒找到就往 enclosing scope 找(class 是往 (indirect)base class scope 找)
    3. 找到名字之後再對名字做 type checking(之後會講到，先找名字再做 type checking)，檢查呼叫名字的 code 是否有正確的使用名字
    4. 如果 type checking 也過了，compiler 根據名字本身是否為 `virtual` 做以下事情:
        1. 如果名字是 virtual && 你是透過 ptr/ref 來呼叫的，則 compiler 會 gen code，並且在 runtime 的時候使用指向的物件的 (dynamic) type 決定要呼叫哪個 function
        2. 否則的情況，也就是名字不是 virtual，或者不是透過 ptr/ref 來呼叫，compiler 就會使用一般的 function call

#### As Usual, Name Lookup Happens before Type Checking
* 6.4.1 有講過，不同 scope 的同名 function 不會 overload，只有在同一個 scope 的多個同名 function 才會
* 這代表著，定義在 derived class 的 function 就算跟 base class 的 function 同名也不會 overload，因為 base 跟 derived 不同 scope
* 而且 compiler 一旦找到名字之後就不會繼續往更大的 scope 找了，這有時候毀很 tricky
* 看例子:
    ```C++
    struct Base {
        int memfcn();
    };
    struct Derived : Base { 
        int memfcn(int); // hides memfcn in the base
    };
    Derived d; Base b; 
    b.memfcn(); // calls Base::memfcn
    d.memfcn(10); // calls Derived::memfcn 
    d.memfcn();  // error: memfcn with no arguments is hidden
    d.Base::memfcn();  // ok: calls Base::memfcn
    ```
    * `Base::memfcn` 跟 `Derived::memfcn` 並不會 overload，而且 `Derived::memfcn` 會 hides `Base::memfcn`
    * `d.memfcn(10)` 就是 tricky 的地方，因為查找的 scope 從 `Derived` 開始，compiler 會在裡面就找到 `memfcn` 它就不會繼續往下找，所以你可能會以為 compiler 會去找 `Base` 內的 `memfcn(int)` 來呼叫，實際上 compiler 根本不會去考慮它，所以 compiler 會噴你 error 說 no matching function in `Derived`

#### Virtual Functions and Scope
* 上面也是為何 base 跟 derived 的同個 virtual function 參數一定要一致的原因
* 如果 base/derived 定義的 virtual 參數不一樣，**就無法使用 ptr/ref to base 去呼叫 derived 定義的 virtual function**
* 看例子LOL:
    ```C++
    class Base {
    public:
        virtual int fcn();
    };
    class D1  : public Base {
    public:
        // hides fcn in the Base; this fcn is not virtual
        // D1 inherits the definition of Base::fcn()
        int fcn(int); // parameter list differs from fcnin Base
        virtual void f2(); // new virtual function that does not exist in Base
    };
    class D2 : public D1 {
    public:
        int fcn(int); // nonvirtual function hides D1::fcn(int)
        int fcn(); // overrides virtual fcn from Base 
        void f2(); // overrides virtual f2 from D1
    };
    ```
    * 上面的 interface 定義的爛透了，只是為了 demo 用而已
    * `D2` 的 `fcn(int)` 是 **hides** `Base::fcn()` 而不是 **override**，注意 hide 跟 override 的差別
    * `D1` 其實有兩個 `fcn`，一個是 `Base::fcn` 一個是 `D1::fcn`
        * 很重要: **就算被 hide，base 定義的東西還是會存在，只是說 compiler 在查找名字的時候不會考慮而已**
        * 所以你會發現，繼承 `D1` 的 `D2` 還是可以 **override** `Base::fcn`，因為 `D2` 還是看的到 Base 內定義的 virtual member

#### Calling a Hidden Virtual through the Base Class
* 有上面的 demo code 我們就可以來看怎麼寫會有什麼行為:
    ```C++
    Base bobj; D1 d1obj; D2 d2obj;
    Base *bp1 = &bobj, *bp2 = &d1obj, *bp3 = &d2obj; 
    bp1->fcn(); // virtual call, will call Base::fcn at run time 
    bp2->fcn(); // virtual call, will call Base::fcn at run time 
    bp3->fcn(); // virtual call, will call D2::fcn at run time
    D1 *d1p = &d1obj; D2 *d2p = &d2obj;
    bp2->f2(); // error: Base has no member named f2 
    d1p->f2(); // virtual call, will call D1::f2() at run time
    d2p->f2(); // virtual call, will call D2::f2() at run time
    ```
    * 前三個 call 都是用 base(也就是 Base) ptr 呼叫 virtual function，所以是 runtime 決定要 call 哪個
        * `bp1` 指向的物件是 `Base`，所以呼叫 `Base::fcn`
        * `bp2` 指向的物件是 `D1`，而 `D1` 沒有 `override` `Base::fcn`，所以一樣呼叫 `Base::fcn`
        * `bp3` 指向的物件是 `D2`，而 `D2` 有 `override` (indirect) base 定義的 `Base::fcn`，所以呼叫 `D2::fcn`
    * 後三個 call 則是用不同 type 的指標呼叫 f2
        * `bp2` 是 `Base*`，它查找的時候從 `Base` 開始找，根本看不到 `f2`，所以噴 error，就算它實際上指向的物件 type 是`D1` 也一樣
        * `d1p` 是 `D1*`，而它指向的物件是 `D1`，且 `D1::f2` 是 `virtual`，所以在 runtime 決定呼叫 `D1::f2`
        * `d2p` 是 `D2*`，而它指向的物件是 `D2`，且 `D2::f2` 是 `virtual`，所以在 runtime 決定呼叫 `D2::f2`
            * 注意這個例子的指標是 `D2*`，compiler 查找 `fc` 時也不會考慮 `D1::fcn`，即使 `D1::fcn` 是 `virtual`

* 在來看呼叫 non`virtual` function 會發生的情況:
    ```C++
    Base *p1 = &d2obj; 
    D1 *p2 = &d2obj; 
    D2 *p3 = &d2obj; 
    p1->fcn(42); // error: Base has no version of fcn that takes an int 
    p2->fcn(42); // statically bound, calls D1::fcn(int)
    p3->fcn(42); // statically bound, calls D2::fcn(int)
    ```
    * Base 有 fcn 這個名字，可是沒有 overload fcn(int)，所以 `p1->fcn(42)` 噴 error
    * D1 有 fcn，而且有 fcn(int)，所以 `p2->fcn(42)` 正確，並且使用 D1::fcn(int)
    * D2 有 fcn，而且有 fcn(int)，所以 `p3->fcn(42)` 正確，並且使用 D2:fcn(int)

#### Overriding Overloaded Functions
* 首先管你是不是 `virtual`，member function 可以被 overload
* 那 derived class 怎麼處理這些 base class 的 overloaded functions 呢?
    * 如果他們是 `virtual`，可以 `override`，
    * 如果希望這些 overloaded functions 都要可以從 derived class \*/& access，則 derived class 要馬全部都不要 override，要馬全部 override

* 可是有時候我就是只想 override 部分 functions 啊，全部做很麻煩ㄟ
* 可以用 `using` 解:
    * using base class 內 overloaded functions 的名字；這樣子 base class 內這些 functions 全部都會*加到* derived class 的 scope 內
    * 這時候你只要定義你想要的部分 functions 就行了，其他沒定義的就會使用 base class 定義的

## 15.7 Constructors and Copy Control
* 館你是不是 derived 一樣會有那五個 members，一樣會有可能被合成成 `delete`

#### 15.7.1 Virtual Destructors
* base class 的 dstr 通常要宣告成 `virtual`，這跟 dynamic binding 很有關係
* 假設你用一個 base* 指向 derived obj，然後直接對 base* 用 `delete`
    * 如果你的 base 的 dstr 不是 virtual，那你 compile time 時就會決定使用 base 的 dstr 來破壞 base* 指向的物件
    * **可是你這時明明就指向 derived 物件!!!**
* 所以因為 dynamic binding 的關係，你必須 runtime 決定要呼叫哪個正確的 dstr

* 來看怎麼定義 hierarchy 中的 dstr:
    ```C++
    class Quote {
    public: // virtual destructor needed if a base pointer pointing to a derived object is deleted 
        virtual ~Quote() = default; 
        // dynamic binding for the destructor
    };
    ```
    * 因為 `virtual` 會一直往 derived 傳，所以不管 derived class 的 dstr 是自定義還是用合成的，都會是 `virtual`
    * 所以只要把 hierarchy root 的 dstr 定義成 `virtual` 就可以(動態)正確決定要呼叫哪個 dstr 了
    ```C++
    Quote *itemP = new Quote; // same static and dynamic type 
    delete itemP; // destructor for Quote called
    itemP = new Bulk_quote; // static and dynamic types differ 
    delete itemP; // destructor for Bulk_quotecalled
    ```
    * 注意 new 的時候是一樣可以把 derived object 的指標丟給 base\*，亦即 `itemP = new Bulk_quote;` 是合法的

* Warning: Executing delete on a pointer to base that points to a derived object has undefined behavior if the base’s destructor is not virtual.
    * 沒把 base 的 dstr 宣告成 virtual 是 UB!

* 還記得之前的 rule of three/five 嗎? base class 的 dstr 就是一個很重要的例外，它一定要定義 dstr(成 `virtual`)，但不一定因為這樣就要定義 copy/assign

#### Virtual Destructors Turn Off Synthesized Move
* 你的 dstr 定義成 `virtual` 之後 compiler 就不會合成 move 了，你打 `= default` 也一樣

### 15.7.2 Synthesized Copy Control and Inheritance
* 管你是 base 還是 derived，synthsized copy-control 行為都一樣，memberwise init/copy/assign members **of the class itself**
* **除此之外還會對 *direct* base 用 base 定義的對應 member 來改變 base 定義的 members**
* 例如來看 `Bulk_quote` 的初始化流程
    * `Bulk_quote` 的 default cstr 會呼叫 `Disc_quote` 的 default cstr 來初始化 `Disc_quote` 定義的 member，而 `Disc_quote` 則呼叫 `Quote` 的
    * 我們之前定義的 `Quote` 預設 cstr 會把 `bookNo` 初始化成空字串，然後用 in-class initialize `price` 成 0
    * `Quote` 的 cstr 跑完後繼續跑 `Disc_quote` 的，他也 in-class initialize `qty` 跟 `discount`
    * `Disc_quote` 的也跑完後繼續跑 `Bulk_quote` 的，結束

* `Bulk_quote` 的合成 copy 也是類似

* 總之因為合成的 copy-control 會呼叫 base 對應的 member，對應的 member 應該要 accessible 並且不能是 `delete`d function

* derived class 的 dstr 也會呼叫 *direct* base 的 dstr，這個過程是遞迴的，直盜 hierarchy root 為止

* 我們也看到 `Quote` 沒有合成的 move members，因為他的 dstr 是 virtual，之後會看到，**這樣的 class 的 derived，compiler 也不會合成 move members**

#### Base Classes and Deleted Copy Control in the Derived
* 不管你是 base 還是 derived，你的合成 copy-control 也會因之前(13章)說的相同原因而定義成 `delete`
* 除此之外，**base class 的一些定義方式也會導致 derived 的 copy-control 被合成 `delete`**:
    * 如果 base 的 defalut cstr, copy cstr, copy assign, dstr 被 delete 或 inaccessible，那 derived 的對應合成就會是 delete，理由是 derived 的合成 copy-control 需要他們
    * 如果 base 的 dstr 是 delete 或 inaccessible，那 derived 的合成 dstr 也會是 delete，因為沒有辦法破壞 base-class part object
    * compiler 發現定義 move 會讓它變成 delete 時一樣不會定義；如果 base 對應的 move members 是 delete 或 inaccessible，則你如果宣告 derived 的 move `= default` 的話也會是 delete，因為 base-class part object 不能被 move
* 舉例:
    ```C++
    class B {
    public:
        B();
        B(const B&) = delete;
        // other members, not including a move constructor
    };
    class D : public B {
    // no constructors
    }; 
    D d; // ok: D's synthesized default constructor uses B’s default constructor
    D d2(d); // error: D's synthesized copy constructor is deleted
    D d3(std::move(d)); // error: implicitly uses D’s deleted copy constructor
    ```
    * 因為 B 定義了 copy cstr 並且是 delete，compiler 不會合成 B 的 move
    * 所以繼承 B 的 D，D 的 copy cstr 是 delete，因為 B 的 copy cstr 是 delete，所以 `D d2(d)` 噴 error；而因為 B 沒有 move，所以 D 不會合成 move；所以對 D 使用 std::move 還是會呼叫 copy cstr，所以還是噴 error

#### Move Operations and Inheritance
* 因為 base 的 dstr 是主動定義的(必須定義成 virtual)，所以 compiler 不會為 base 合成 move，這時 compiler 也不會為 derived class 合成 move
* 因為上面的行為，如果 derived 有預設 move 的需求，base 最好手動定義 move
* 可以直接在 base 說 move `= default;`:
    ```C++
    class Quote {
    public:
        Quote() = default; // memberwise default initialize
        Quote(const Quote&) = default; // memberwise copy
        Quote(Quote&&) = default; // memberwise move
        Quote& operator=(const Quote&) = default; // copy assign Quote& 
        operator=(Quote&&) = default; // move assign
        virtual ~Quote() = default; 
        // other members as before
    };
    ```
    * 這樣 compiler derived class 就不會不定義 move(以及其他 copy-control)了，除非有其他 13章說的情況

### 15.7.3 Derived-Class Copy-Control Members
* 之前已經看過，derived-class cstr 會在初始化階段(執行 cstr init list 的階段)把 base 初始化完成
* 其他的 copy-control member 也要做類似的事情，copy/move cstr/assign 也要 copy/move cstr/assign base 的 members
* 不過 derived-class 的 dstr 只需要處理自己的 members 就好，剩下的交給 base 的 dstr

#### WARNING: When a derived class defines a copy or move operation, that operation is responsible for copying or moving the entire object, including base-class members.
* 記住一定要在對應的 copy-control 做這件事

#### Defining a Derived Copy or Move Constructor
* 直接看 derived copy/move cstr 怎麼處理 base 的部分
* 其實就是呼叫 base 的 copy/move cstr XD，不過注意傳進去的參數有點ㄎㄧㄤ:
    ```C++
    class Base { /* .. . */};
    class D: public Base {
    public:
        // by default, the base class default constructor initializes the base part of an object 
        // to use the copy or move constructor, we must explicitly call that 
        // constructor in the constructor initializer list
        D(const D& d): Base(d) // copy the base members 
                /* initializers for members ofD */{ /* ... */ }
        D(D&& d): Base(std::move(d)) // move the base members 
                /* initializers for members of D */{ /* ... */ }
    };
    ```
    * 沒錯，直接把要 copy/move 的 derived class object 丟到 base 的 copy/move cstr 內，它會 copy/move 好 base-class part 的部分
    * 如果你沒有在 derived 的 cstr init list 呼叫 base 的 copy/move cstr，那就會 default cstr base-class part，這樣的複製會「不完全」

#### Derived-Class Assignment Operator
* 還用想ㄇ，呼叫 base::operator= R!!
    ```C++
    // Base::operator=(const Base&) is not invoked automatically D &D::operator=(const D &rhs) {
        Base::operator=(rhs); // assigns the base part 
        // assign the members in the derived class, as usual, 
        // handling self-assignment and freeing existing resources as appropriate
        return *this;
    }
    ```
    * 一樣把 type 為 derived 的 object 直接丟到 base 的 `operator=` 去，這樣會 assign base-class part
    * ㄚ不管你的 base 的 `operator=` 是自訂的還是 compiler 合成的都可以直接拿來用

#### Derived-Class Destructor
* 講過ㄌ，derived-class dstr 只要處理自己 allocate 的東西就好了，當它執行完之後，會呼叫自己的 members 的 dstr，以及 base class 的 dstr:
    ```C++
    class D: public Base {
    public: // Base::~Base invoked automatically 
        ~D() { /* do what it takes to clean up derived members */}
    };
    ```
* Objects are destroyed in the opposite order from which they are constructed: The derived destructor is run first, and then the base-class destructors are invoked, back up through the inheritance hierarchy.
    * 總之這個 order 很 stackful，懂吧?

#### Calls to Virtuals in Constructors and Destructors
* **注意，此 section 超ㄎㄧㄤ，要認真看，不過挺好理解的**
* 回顧一下一個物件的初始化以及被破壞的過程:
    * 初始化時，先初始化 (indirect) base 的部分，再初始化自己的部分
    * 破壞是，先破壞自己，再破壞 (indirect) base 的部分
* 仔細想，當在初始化一個物件時，還沒初始化完 base 的部分之前，**derived 的部分是未初始化的**；
* 當一個物件被破壞時，準備要破壞 base 的部分時，**derived 的部分已經被破壞**；
* **所以在初始化以及破壞物件時在處理 base class 的部分的時候，整個物件其實是不完整的，而且不完整的地方在於 derived-part object**
* 為了防止世界被破壞(?)，當在初始化或破壞時，如果處於在處理 base-class part 的階段，**這時 derived object 的 dynamic type 會被當成 base class!!!!!**
* 這個階段怎麼區別呢? 就是當現在執行什麼 class 的 cstr，這個 object 就是這個 class 的 type
* ***所以在 cstr 內呼叫的 virtual function，會被當做是由一個這個 class 的 type 的物件來呼叫的***，dstr 也是一樣
* 而且這個行為還包刮被間接呼叫的 virtual function，意即 cstr 呼叫某 A，結果 A 去呼叫某 virtual，則這個 virtual 也會發生一樣的行為

* 為什麼要這樣設計呢? 因為在呼叫 base cstr 時，derived-class part 還沒完成，如果這時綁定 derived-class 的 virtual，它很可能會使用 derived-class 的 member，這樣會是災難；dstr 的情況類似，不過 dstr 是 derived-class 的 member 已經被破壞

#### 15.7.4 Inherited Constructors
* 覺得這卵蛋很難用而且很難記 LOL，打筆記到時候再看
* C++11，derived 可以重用(reuse) base 定義的 cstrs，
* 不過雖然 cstrs 跟一般的 member 不一樣不是用 "繼承" 的，我們還是會說 cstrs 被 derived "繼承"(inherited)

* 出於跟 derived 只能直接初始化 direct base 一樣的原因，derived 也只能"繼承" direct base 的 cstrs
* 不過 class 不能繼承 default/copy/move cstrs
* 繼承 base cstrs 的方式是用 `using`:
    ```C++
    class Bulk_quote : public Disc_quote {
    public:
        using Disc_quote::Disc_quote; // inherit Disc_quote’s constructors 
        double net_price(std::size_t) const;
    };
    ```
* 注意這裡的 `using` 語意跟一般不太一樣，一般的 using 只不過是讓某個名字在當前 scope 可以 visible，**但這裡的 `using`，也就是如果對 cstrs 用 `using` 的話，compiler 是會 gen code 的**
    * gen 什麼 code? compiler 會在 derived 內一一定義跟繼承的 base cstrs 對應的 derived cstrs
    * 在講詳細一點，對於每個 cstr in base，compiler 會生一個 para list 一模一樣的 derived cstr 在 derived 內，形式長這樣:
    ```C++
    derived(parms): base(args){ }
    ```
    `derived` 是 derived class 的名字，`base` 是 base 的，`parm` 是 para list，`args` 把 `param` 一個一個一一送到 `args` 裡面
    * 舉例，如果直接讓 `Bulk_quote` 繼承 `Disc_quote` 的 cstr 則會生出跟下面等價的 cstr:
    ```C++
    Bulk_quote(const std::string& book, double price,
            std::size_t qty, double disc):
        Disc_quote(book, price, qty, disc) { }
    ```
    * 而如果 derived class 還有自己的 memebers，你直接 `using` base cstr 的話，這些 members 就會預設初始化

#### Characteristics of an Inherited Constructor
* 首先對 base cstrs 使用 `using` 不會造成 cstr 的 access level 被改變，原本是 `public`/`private`/`protected`，因為 `using` 而生的 derived cstrs 就會維持 `public`/`private`/`protected`
* 還有，`using` on cstrs 不能額外加 `explicit` 或 `constexpr`，完全由 base 的 cstr 決定，base 的 cstr 是 `explicit`/`constexpr`，生出來的就也會是
* 如果 base cstr 有 default args，**那些 args 不會被繼承...**
    * compiler 反而會 gen 多個 derived cstr 來對應這個有 default args 的 base cstr
    * 每一個 cstr 會 omit 其中一個 base 的有 default args 的 parameter
    * 例如某 base cstr 有兩個 parameters，第二個 para 有 default arg，則 compiler 會生出兩個 derived cstrs，第一個兩個 para 都有(並且沒有 default args)，第二個只會有一個 para，而且對應到 base 的第一個 para，也就是沒有 default arg 的那個 para
* 如果 base class 有多個 cstrs，則除了兩個例外以外，derived class 會"繼承"這些 cstrs
    * 第一個例外是，你如果說要用 using 繼承 base cstr，結果你在 derived 內手動定義了一個 cstr，這個 cstr 的 para list 跟某個 base cstr 的 para list 一模一樣，則這個 base cstr 不會被繼承
    * 第二個剛剛才講過，default/copy/move cstr 不會被繼承，這些 derived 對應的 cstr 會用一般的方法來合成(如果處於 compiler 會幫你合成的情況的話)
        * 還有，使用 `using` 繼承的 cstrs **不算是 user defined cstr**，所以如果一個 derived class 只有包含了用 `using`，繼承的 cstrs 的話，那 compiler 就會幫你合成 default cstr!!

## 15.8 Containers and Inheritance
* 你想要在一個 container 裡放整個 hierarchy 內不同的 class object，唯一的做法就是在 container 裡放 base 指標而不是直接放 object
* 如果你應宣告像是 `vector<base> vec` 的東西，然後你用 `vec.push_back(derived_obj)` 等等的，不會噴 error，不過那都是假的! 那樣只會呼叫 base 的 copy/move cstr，然後把 `derived_obj` 的 base-class part copy/move 過去:
    ```C++
    vector<Quote> basket;     
    basket.push_back(Quote("0-201-82470-1", 50));     
    // ok, but copies only the Quote part of the object into basket 
    basket.push_back(Bulk_quote("0-201-54848-8", 50, 10, .25)); 
    // calls version defined by Quote, prints 750, i.e., 15 * $50
    cout << basket.back().net_price(15) << endl;
    ```
    * 這份 code 不但複製 derived class 沒複製完全，call virtual function 時也沒有使用 dynamic type，因為是由物件直接呼叫的
#### Put (Smart) Pointers, Not Objects, in Containers
* 這是唯一解法...，你也不能用 reference，因為 container 不能用 ref 當 elements
* 這樣 copy 的就是 smart ptr 而不是物件:
    ```C++
    vector<shared_ptr<Quote>> basket; 
    basket.push_back(make_shared<Quote>("0-201-82470-1", 50)); 
    basket.push_back(make_shared<Bulk_quote>("0-201-54848-8", 50, 10, .25));
    // calls the version defined by Quote; prints 562.5, i.e., 15 * $50 less the discount
    cout << basket.back()->net_price(15) << endl;
    ```
    * 你直接在 `make_shared` 時指定 smart ptr 要指向的物件型別(如果是 raw ptr 就是指定 new 的型別)
    * 這份 code 複製的是 ptr 而不是物件，真正複製物件的 part 是在 `make_shared` 內，而 `make_shared` 可由我們ˇ定型別；除此之外因為是由 (smart) ptr 呼叫 virtual function，所以會 dynamic binding
* 總之做法就是 container 存 (smart)ptr to base，然後可以把 (smart)ptr to derived 存到 container 內(其實就是 derived to base conversion, p.597)，這樣就會有 dynamic binding 的效果
* 要記住 smart ptr 支援 dynamic binding
    * 這樣 C++11 之前要用 container 實現 dynamic binding 一定很嚇人 LOL，雖然 smart ptr 還是很恐怖

### 15.8.1 Writing a Basket Class
* 只能用 ptr/ref 來達到 dynamic binding 真的很嘲諷
* 所以通常會定義一個輔助性的 class 來把 ptr 包起來:
    ```C++
    class Basket {
    public:
        // Basket uses synthesized default constructor and copy-control members 
        void add_item(const std::shared_ptr<Quote> &sale) 
            { items.insert(sale); }
        // prints the total price for each book and the overall total for all items in the basket 
        double total_receipt(std::ostream&) const;
    private:
        // function to compare shared_ptrs needed by the multiset member 
        static bool compare(const std::shared_ptr<Quote> &lhs, 
                            const std::shared_ptr<Quote> &rhs)
        { return lhs->isbn() < rhs->isbn(); } 
        // multiset to hold multiple quotes, ordered by the compare member 
        std::multiset<std::shared_ptr<Quote>, decltype(compare)*> items{compare};
    };
    ```
    * 有沒有覺得光是 wrapper 就很噁心?
    * 用 `multiset` 儲存 hierarchy 中的 class objects，這樣會排序 objects，並且一樣(書本)的會擺在一起
    * 注意 set 需要 elements 定義 <(雖然會用 less(14章)，不過 less 還是使用 <)，但 `shared_ptr` 沒有 <，所以需要傳入 predicate `compare`
        * 注意這個 `compare` 算是 utility function，所以定義 `private`；而且它沒用到 non`static` member，所以定義成 `static` function

#### Defining the Members of Basket
* `Basket` 只有定義 `add_item` 跟 `total_receipt` 這兩個 API；`add_item` 已經定義，就只是在 `multiset` 內塞一個 `shared_ptr`；來看 `total_receipt`:
    ```C++
    double Basket::total_receipt(ostream &os) const {
        double sum = 0.0; // holds the running total
        // iter refers to the first element in a batch of elements with the same ISBN 
        // upper_bound returns an iterator to the element just past the end of that batch 
        for (auto iter = items.cbegin(); 
                  iter != items.cend(); 
                  iter = items.upper_bound(*iter)) {
            // we know there’s at least one element with this key in the Basket
            // print the line item for this book 
            sum += print_total(os, **iter, items.count(*iter));
        }
        os << "Total Sale: " << sum << endl; // print the final overall total 
        return sum;
    }
    ```
    * 請往回找已經定義的 `print_total`
    * 注意，一個 `Quote` object 代表的是*一本*書，以及這本書該用什麼方法算錢；一個 `Quote` 不是代表一堆書!
    * 所以一堆相同(ISBN)的 `Quote` objects 擺在一起，代表買了這麼多本相同的書
    * 而同一本書不可能用不同方式折價，所以理應當不會發生兩個 object 型態不同(型態其實代表著折價方式)，卻有相同 ISBN，所以所有同樣 ISBN 的書擺在一起，他們對應的 dynamic type 也會一樣，dynamic type 一樣代表著 `print_total` 呼叫的 `net_price` 也是同一個
    * 啊 loop 的第三個 expression 只是為了讓 iter 跳到下一本不同 ISBN 的書而已
    * 還有 iter 是 iterator type，deref 之後得到 element，而這裡是 shared_ptr，所以還要再 deref 一次才會得到 `Quote` object，所以才會再 `print_total` 內傳入 `**iter`

#### Hiding the Pointers
* `Basket::add_item` 還是要吃 ptr 啊! 這樣沒有把 ptr 藏起來，所以 user code 還是得寫這種東西:
    ```C++
    Basket bsk; 
    bsk.add_item(make_shared<Quote>("123", 45));
    bsk.add_item(make_shared<Bulk_quote>("345", 45, 3, .15));
    ```
* 所以要重定義 `add_item`...，它會 handle memory allocation，user code 不用自己做
    ```C++
    void add_item(const Quote& sale); // copy the given object 
    void add_item(Quote&& sale); // move the given object
    ```
    * 在這兩個 function 內，大概會出現這種東西:
    ```C++
    new Quote(sale)
    ```
    * 但這樣問題又出來了... `new` 只會 allocate 我們指定的 type，你傳一個 derived object 進去也沒用，只會複製 base-class part

#### Simulating Virtual Copy
* 解法是在 `Quote` 內新增一個 `virtual` `clone`，會 allocate 一份 copy...
    ```C++
    class Quote {
    public: 
        // virtual function to return a dynamically allocated copy of itself 
        // these members use reference qualifiers; see § 13.6.3 (p. 546) 
        virtual Quote* clone() const & 
            {return new Quote(*this);}
        virtual Quote* clone() && 
            {return new Quote(std::move(*this));}
        // other members as before
    };
    class Bulk_quote : public Quote { 
        Bulk_quote* clone() const & 
            {return new Bulk_quote(*this);} 
        Bulk_quote* clone() && 
            {return new Bulk_quote(std::move(*this));}
        // other members as before
    };
    ```
    * 注意 clone 有使用 ref qualifier(p. 546)，讓 rvalue 在 new object 時用 move，lvalue 用 copy
    * 媽的這樣還不是要寫一個回傳指標的 utility，雖然比較簡單LOL

* 用 `clone` 就可以寫 `add_item` 惹:
    ```C++
    class Basket { 
    public: 
        void add_item(const Quote& sale) { // copy the given object    
            items.insert(std::shared_ptr<Quote>(sale.clone())); 
        } 
        void add_item(Quote&& sale) { // move the given object
            items.insert( std::shared_ptr<Quote>(std::move(sale).clone())); 
        } // other members as before
    };
    ```
    * 一樣提供 overload，分別處理 l/rvalue 的情況
    * 注意 `sale` 是 reference，所以用它呼叫 virtual function會有 dynamic binding!
    * 所以 `sale.clone()` 會根據 sale 綁定的物件的 dynamic type 決定要呼叫 `Quote::clone` 還是 `Bulk_quote::clone`
    * 還有再提醒一次雖然有一個版本的參數是 rv ref，可是直接使用它時它還是被當作 lvalue，所以要加 std::move

## 15.9 Text Queries Revisited
* 最後一個章節惹...
* 複習一下 `TextQuery` 在幹嘛: 給定一個檔案，可以查詢某個 word 出現在哪些行數
* We’d like to extend the system to support more complicated queries.
* 範例文章:
    `Alice Emma has long flowing red hair. Her Daddy says when the wind blows through her hair, it looks almost alive, like a fiery bird in flight. A beautiful fiery bird, he tells her, magical but untamed. "Daddy, shush, there is no such thing," she tells him, at the same time wanting him to tell her more. Shyly, she asks, "I mean, Daddy, is there?"`
    
* Our system should support the following queries:
    * 12章的基本查字功能
    * ~(*word*)，查沒有出現 *word* 的行數
    * *word1*|*word2*，查出現 *word1* 或 *word2* 的行數
    * *word1*&*word2*，查出現 *word1* 且 *word2* 的行數
    * 而且可以 combine 他們! 挖草幹 compiler?(?:
        `fiery & bird | wind`
        * 會用 C++ 定義的 operator precedence 來 parse 這個 query

* 符合條件的行數會跟 12章的印法一樣

### 15.9.1 An Object-Oriented Solution
* 這裡在講什麼想 hierarchy
* 我們一開始會覺得直接繼承 `TextQuery` 就好
* 但這樣不OK，想想看用 ~ 的時候... 說真的我看不太懂這邊 Primer 在解釋三小，不管中文版還英文
* 直接定義:
    * WordQuery // Daddy 
    * NotQuery // ~Alice 
    * OrQuery // hair | Alice
    * AndQuery // hair & Alice
* 讓他們共享 base class
* 它們每個都只會有兩個 operations:
    * `eval`: 吃一個 `TextQuery` object 然後回傳 `QueryResult`
    * `rep` : 回傳一個 `string` 表示某個 query 的 string(例如 word1|word2 這種東西的字串)；它會被 `eval` 還有 output operator 使用

#### Abstract Base Class
* 這四個是共同繼承某 base class，they are conceptually siblings.
* 既然他們 share same interface，代表 base class 應該要定義成 abstract base class 來代表這個 interface
* We'll name our abstract base class `Query_base`, indicating that its role is to serve as the root of our query hierarchy.
    * `eval` 跟 `rep` 都是 pure virtual function
* `Query_base` 直接被 `Word_query` 跟 `Not_query` 繼承
* `AndQuery` 跟 `OrQuery` 比較特別: 他們有兩個 operands，我們乾脆直接定義另一個 abstract base class，叫做 `Binary_Query`，它會繼承 `Query_base` 然後 `And/OrQuery` 再繼承 `BinaryQuery`
* 示意圖
* ![](https://i.imgur.com/MdFylLf.png)

#### KEY CONCEPT: INHERITANCE VERSUS COMPOSITION
* 其實要怎麼組織 hierarchy 是超難的問題，詳情軟工；不過 inheritance ㄍ composition 要能分辨
* 其實就是 "is a" 跟 "has a" 的差別啦，大一 OOP 有講
* 不過一個設計夠好的 hierarchy，如果是 public 繼承的話，通常需要 base object 的地方都可以直接拿 derived object 來代替才對

#### Hiding a Hierarchy in an Interface Class
* 我們不希望 user code 直接碰這個 hierarchy
    * 你想想，需要他們用 算 word1&word2 就要用 AndQuery，用 | 要改成 OrQuery，感覺很累人
* 相反的我們希望 user code 可以寫類似這樣的東西(這樣應該是最容易的):
    ```C++
    Query q = Query("fiery") & Query("bird") | Query("wind");
    ```
* 為此我們需要再定義一個 `Query` class，把整個 hierarchy 包起來，它算是一種 **interface class**
    * 它會儲存 ptr to `Query_base`
        * dynamic binding 來惹
    * 它也會提供 `Query_base` 提供的 operations，`eval` 跟 `rep`
    * 再來就是 overload output operator
* user code 全部都是由 `Query` class 間接的使用整個 hierarchy
    * 再額外定義三個 overload operator，分別是 & | ~
        * & 會產生一個 `AndQuery` 然後被 `Query` 內的 ptr 綁住
        * | 產生 `OrQuery`
        * ~ 產生 `NotQuery`
    * 還有 cstr，吃一個 string
        * 產生 `WordQuery`

![](https://i.imgur.com/OhtP34s.png)

#### Understanding How These Classes Work
* 基本上要計算一個 query 就是按照上面的 tree 在爬的，從某個 node 一直往下算算到所有 leaf
* 當一個 `Query` object 形成時對應的 tree 也會出現，直接對它呼叫 `eval` 就會爬完整棵樹
* 詳情 Primer p.638~639
* When new to object-oriented programming, it is often the case that the hardest part in understanding a program is understanding the design.
    * Once you are thoroughly comfortable with the design, the implementation flows naturally.
* 用下面 table 來表示如何 design 這個 program 要解的問題:
* ![](https://i.imgur.com/kwBVUfF.png)

### 15.9.2 The Query_base and Query Classes
* We'll start our implementation by defining the Query_base class:
    ```C++
    // abstract class acts as a base class for concrete query types; all members are private 
    class Query_base { 
        friend class Query;
    protected:
        using line_no = TextQuery::line_no; // used in the eval functions 
        virtual ~Query_base() = default;
    private: 
        // eval returns the QueryResult that matches this Query 
        virtual QueryResult eval(const TextQuery&) const = 0; 
        // rep is a string representation of the query 
        virtual std::string rep() const = 0;
    };
    ```
    * Because we don't intend users, or the derived classes, to use `Query_base` directly, `Query_base` has no public members.
    * All use of `Query_base` will be through Query objects.
    * `Query` 會呼叫 (private)virtual，所以把它宣告成 `friend`
    * `line_no` 跟 dstr 都宣告成 `protected`，因為 derived class 會用到
        * `line_no` 拿來宣告 set
        * dstr 是因為 derived class 一定要可以 access，不然不能刪物件 LOL，跟是否 virtual 沒關係，雖然一定要 virtual 就是了

#### The Query Class
* The Query class provides the interface to (*and hides*) the Query_base inheritance hierarchy.
* 包含一個 `shared_ptr<Query_base>`
* define its own versions of eval and rep
* constructor that takes a string, create a new `WordQuery`, bind shared_ptr to it
* The &, |,and ~ operators will create `AndQuery`, `OrQuery`, and `NotQuery` objects, respectively
    * return a Query object bound to its newly generated object.
* To support these operators, Query **needs a constructor that takes a shared_ptr** to a Query_base and stores its given pointer.
    * make this constructor `private` because we don’t intend general user code to define Query_base objects
    * Because this constructor is private, we’ll need to make the operators `friends`

* 總之長這樣啦:
    ```C++
    // interface class to manage the Query_base inheritance hierarchy
    class Query { 
        // these operators need access to the shared_ptr constructor
        friend Query operator~(const Query &);
        friend Query operator|(const Query&, const Query&);
        friend Query operator&(const Query&, const Query&);
    public: 
        Query(const std::string&); // builds a new WordQuery 
        // interface functions: call the corresponding Query_base operations 
        QueryResult eval(const TextQuery &t) const     
            { return q->eval(t); }
        std::string rep() const { return q->rep(); } 
    private:
        Query(std::shared_ptr<Query_base> query): q(query) { } 
            std::shared_ptr<Query_base> q;
    };
    ```
    * | & ~ 會用到 private cstr，所以宣告 `friend`
    * cstr(string)，我們只能宣告不能定義，因為它會創造 `WordQuery` object，而我們還沒定義 `WordQuery`(總之就是那個宣告跟定義順序的問題)
    * `eval` 跟 `rep` 都是直接呼叫對應的 `Query_base` 的 (virtual) function；注意，`Query` 本身的 `eval` 跟 `rep` 不是 virtual，因為它沒有繼承 `Query_base`

#### The Query Output Operator
```C++
std::ostream & operator<<(std::ostream &os, const Query &query) {
    // Query::rep makes a virtual call through its Query_base pointer to rep() 
    return os << query.rep();
}
```
* 注意 `Query::rep` 最後都會呼叫 (virtual)`rep`，which 動態決定要 call 哪種版本的 `rep`
* 例如:
    ```C++
    Query andq = Query(sought1) & Query(sought2); 
    cout << andq << endl;
    ```
    * `cout << andq` 會呼叫 `Query::rep`，which 透過 shared_ptr 呼叫 `Query_base::rep`，which 在動態時綁定 `AndQuery`，所以最後會呼叫 `AndQuery::rep`

### 15.9.3 The Derived Classes
* 每個 `Query_base` 的 derived class 的 operand(s) 一樣可以是任何 `Query_base` 的 derived class
    * 舉例來說，`NotQuery` 的 operand 可以是 `WordQuery`, `AndQuery`, `OrQuery`, `NotQuery`
* To allow this flexibility, **the operands must be stored as pointers to `Query_base`.** That way we can bind the pointer to whichever concrete class we need.
* 但是上面只是幹話，我們其實會用已經定義好了 interface class `Query` 來達到上面說的事情，這樣就不用自己管理 `shared_ptr` 了
    * 天啊又是互相使用對方型別的例子...

#### The WordQuery Class
* 就尋找一個 word 啊...
    ```C++
    class WordQuery: public Query_base {
        friend class Query; // Query uses the WordQuery constructor 
        WordQuery(const std::string &s): query_word(s) { }
        // concrete class: WordQuery defines all inherited pure virtual functions 
        QueryResult eval(const TextQuery &t) const 
                { return t.query(query_word); }
        std::string rep() const 
                { return query_word; }
        std::string query_word; // word for which to search
    };
    ```
    * 然後 eval 會吃一個 `TextQuery`(12章定義的)，直接用它的 `query` 回傳 `QueryResult`
    * 它也沒有 public members，所以 `Query` 一樣要當 friends
* 因為現在有定義 `WordQuery` 了，我們可以定義 `Query` 的 cstr 了:
    ```C++
    inline Query::Query(const std::string &s): q(new WordQuery(s)) { }
    ```
    * This constructor allocates a `WordQuery` and initializes its pointer member to point to that newly allocated object.

#### The NotQuery Class and the ~ Operator
* The ~ operator generates a NotQuery, *which holds a Query*, which it negates:
    ```C++
    class NotQuery: public Query_base {
        friend Query operator~(const Query &);
        NotQuery(const Query &q): query(q) { } 
        // concrete class: NotQuery defines all inherited pure virtual functions 
        std::string rep() const {return "~(" + query.rep() + ")";} 
        QueryResult eval(const TextQuery&) const;
        Query query;
    };
    inline Query operator~(const Query &operand) {
        return std::shared_ptr<Query_base>(new NotQuery(operand));
    }
    ```
    * 一樣，`NotQuery` 的 operand 可以是那四種的其中一種，所以在 `NotQuery::rep` 內呼叫的 `query.rep` 最後也會 dynamic binding
    * The ~ operator *dynamically* allocates a new `NotQuery` object.
        * 注意它的 return  其實很複雜
        * return type 是 `Query`，不過回傳的是 shared_ptr，所以其實使用到了 `Query` 的吃 shared_ptr 的(private)cstr
        * eval 太複雜，晚意點再定義

#### The BinaryQuery Class
* abstract base class
    ```C++
    class BinaryQuery: public Query_base {
    protected:
        BinaryQuery(const Query &l, const Query &r, std::string s): 
                lhs(l), rhs(r), opSym(s) { }
        // abstract class: BinaryQuery doesn’t define eval
        std::string rep() const
            { return "(" + lhs.rep() + " " + opSym + " " + rhs.rep() + ")"; }
        Query lhs, rhs; // right- and left-hand operands
        std::string opSym; // name of the operator
    };
    ```
    * The data in a `BinaryQuery` are the two `Query` operands and the corresponding operator symbol.
    * `BinaryQuery::rep` 呼叫的 rep 一樣會 dynamic binding
    * 注意，`BinaryQuery` 沒有定義 eval(這個 pure virtual)，所以他一樣是 abstract base class

#### The AndQuery and OrQuery Classes and Associated Operators
* 長的很像一起定義啦:
    ```C++
    class AndQuery: public BinaryQuery {
        friend Query operator&(const Query&, const Query&);
        
        AndQuery(const Query &left, const Query &right): 
            BinaryQuery(left, right, "&") { }
        // concrete class: AndQuery inherits rep and defines the remaining pure virtual
        QueryResult eval(const TextQuery&) const;
    };
    inline Query operator&(const Query &lhs, const Query &rhs) {
        return std::shared_ptr<Query_base>(new AndQuery(lhs, rhs)); 
    }
    class OrQuery: public BinaryQuery {
        friend Query operator|(const Query&, const Query&);

        OrQuery(const Query &left, const Query &right):
            BinaryQuery(left, right, "|") { }
        QueryResult eval(const TextQuery&) const;
    };
    inline Query operator|(const Query &lhs, const Query &rhs) {
        return std::shared_ptr<Query_base>(new OrQuery(lhs, rhs));
    }
    ```
    * 說真的，定義 friend.. 然後會 call 自己的 constructor... 真的很ㄎㄧㄤ
    * 他們各自的 cstr 都呼叫了 base-cstr (呼叫 `BinaryQuery`)
    * Like the ~ operator, the & and | operators return a `shared_ptr` bound to a newly allocated object of the corresponding type.
        * 然後在 return 的時後又轉成 Query...

### 15.9.4 The eval Functions
* The eval functions are the heart of our query system:
    * The OrQuery eval operation returns the union of the results of its two operands;
    * AndQuery returns the intersection.
    * The NotQuery is more complicated: It must return the line numbers that are not in its operand’s set.
* 會使用到 p.490 exercise 定義的 QueryResult，假設它有 `begin` `end` 可以用
* 還有 `get_file` 可以拿到 file

#### `OrQuery::eval`
* represents the union of the results for its two operands, which **we obtain by calling eval on each of its operands.**
* 因為 operands 是 `Query`，它的 `eval` 又會啟動那一連串該死的 virtual call，總之最後會得到 operands 運算後的結果...
    ```C++
    // returns the union of its operands' result sets
    QueryResult
    OrQuery::eval(const TextQuery& text) const {
        // virtual calls through the Query members, lhs and rhs
        // the calls to eval return the QueryResult for each operand
        auto right = rhs.eval(text), left = lhs.eval(text);
        // copy the line numbers from the left-hand operand into the result set 
        auto ret_lines = make_shared<set<line_no>>(left.begin(), left.end());
        // insert lines from the right-hand operand 
        ret_lines->insert(right.begin(), right.end()); 
        // return the new QueryResult representing the union of lhs and rhs 
        return QueryResult(rep(), ret_lines, left.get_file());
    }
    ```
    * 其實最主要是 `set` 可以直接 insert 一個 range，很好用

#### `AndQuery::eval`
* 跟 `OrQuery::eval` 很像，不過會用 STL 的 `set_intersection` 來找共同 line_no:
    ```C++
    // returns the intersection of its operands' result sets 
    QueryResult
    AndQuery::eval(const TextQuery& text) const {
        // virtual calls through the Query operands to get result sets for the operands 
        auto left = lhs.eval(text), right = rhs.eval(text);
        // set to hold the intersection of left and right
        auto ret_lines = make_shared<set<line_no>>();
        // writes the intersection of two ranges to a destination iterator     
        // destination iterator in this call adds elements to ret     
        set_intersection(left.begin(), left.end(), 
            right.begin(), right.end(), inserter(*ret_lines, ret_lines->begin()));
        return QueryResult(rep(), ret_lines, left.get_file());
    }
    ```
    * `set_intersection` ㄎㄧㄤ，你想知道它怎麼找的詳情演算法大師
    * 注意 `set_intersection` 最後一個 arg 是 `inserter`

#### NotQuery::eval
* 看 code...:
    ```C++
    // returns the lines not in its operand's result set
    QueryResult
    NotQuery::eval(const TextQuery& text) const {
        // virtual call to eval through the Query operand
        auto result = query.eval(text);
        // start out with an empty result set
        auto ret_lines = make_shared<set<line_no>>();
        // we have to iterate through the lines on which our operand appears 
        auto beg = result.begin(), end = result.end();
        // for each line in the input file, if that line is not in result,     
        // add that line number to ret_lines
        auto sz = result.get_file()->size();
        for (size_t n = 0; n != sz; ++n) {
            // if we haven't processed all the lines in result
            // check whether this line is present
            if (beg == end || *beg != n)
                ret_lines->insert(n); // if not in result, add this line
            else if (beg != end) ++beg; // otherwise get the next line number in result if there is one
        }
        return QueryResult(rep(), ret_lines, result.get_file());
    }
    ```
    * `get_file` 其實是回傳 `shared_ptr<vector<string>>`，size 就是 file 行數
    * We generate that set by iterating through sequenital integers up to the size of the input file. We’ll put each number that is not in result into ret_lines.
    * for loop 內的ㄎㄧㄤ邏輯要想一下，總之就是原本沒有的 line_no 就插進去到 `ret_lines` 內



