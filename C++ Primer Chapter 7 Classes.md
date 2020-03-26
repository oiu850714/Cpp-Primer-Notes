---
tags: C++
---

# C++ Primer Chapter 7 Classes
* class 背後真正想做到的是什麼?
    * **data abstraction** and **encapsulation**.
    * abstraction 個人認為是把要處理的事情(dealing with data)給抽象化成一些操作(operations)，這些操作構成了使用這個 class 的介面(interface)，而 class 的使用者只需要知到介面怎麼用，介面背後的實作使用者不用知道
    * encapsulation 其實就是把上面說的介面跟實作強制分開的意思，class 使用者除了不用知道實作以外，**他們也不該知道實作**。
        * 其實這裡的"不該知道"是指說，在寫 code 的時候，你要想的是怎麼使用 class 介面，而不是去想這個介面的背後會發生什麼事
        * 如果你一直去想背後實作，要馬你要換個方式去想，或者是 class 提供的介面太糟糕

* A class that uses data abstraction and encapsulation **defines an abstract data type.(ADT)**
    * In an abstract data type, the class designer worries about how the class is implemented.
    * Programmers who use the class *need not know how the type works.*
    * They can **instead think *abstractly*** about what the type does.

## 哲學課!

## 7.1 Defining Abstract Data Types
* The Sales_item class that we used in Chapter 1 is an abstract data type.
* We use a Sales_item object **by using its interface** (i.e., the operations described in § 1.5.1 (p. 20)).
    * isbn 阿，operator+ 阿，>> 阿，這些都是它的 interface
* We have no access to the data members stored in a Sales_item object.
    * 你沒有辦法直接存取到實際上存著 isbn 內容的 data member，你只能呼叫 isbn 這個 function 來拿到它
        * 其實你也不用知道他到底怎麼存 isbn 的，什麼都不用知道，你只要知道呼叫 isbn 這個 member function 可以拿到 ISBN 就好了
* Our *Sales_data* class (§ 2.6.1, p. 72) is *not* an abstract data type.
    * 如果忘記的話，就是那個 struct，裡面所有 data member 都可以存取，還要額外定義 function 來處理的那個鬼東西
    * To make Sales_data an abstract type, we **need to define operations for users of Sales_data to use.** Once Sales_data defines its own operations, we can encapsulate (that is, hide) its data members.

### 7.1.1 Designing the Sales_data Class
* 感覺 Primer 希望從一個 non ADT 的 Sales_data 一步一步幹成 ADT 的 Sales_item?
* The *Sales_item* class had one member function (§ 1.5.2, p. 23), named isbn, and supported the +, =, +=, <<,and >> operators.
    * 很後面的章節才會講到怎麼定義 operator，現在先用一般的 member function，也就是有名子的 function 代替
    * 有些 operator 其實不能用 member function 來實作，要用 global function，這之後的章節也會講
    * 之後也會講，因為一些原因，我們(預設)不用定義 = 怎麼實作
    * 綜合上述，我們 Sales_data 需要定義的 interface(operations) 有:
    * 
        * An isbn ***member*** function to return the object’s ISBN 
        * A combine ***member*** function to add(+=) one Sales_data object into another 
        * A (ordinary) function named add(+) to add two Sales_data objects
        * A (ordinary) read(>>) function to read data from an istream into a Sales_data object
        * A (ordinary) print(<<) function to print the value of a Sales_data object on an ostream

#### Using the Revised Sales_data Class
* 在我們想怎麼實作 Sales_data 的 interface，我們先想要怎麼使用它
* 如果我們實作了上述說的 Sales_data 的 interface，我們就可以把章節 1.6 使用 Sales_item 的一個簡單小程式改成這樣:
    ```cpp
    Sales_data total;    // variable to hold the running sum
    if (read(cin, total)) { // read the first transaction
        Sales_data trans;    // variable to hold data for the next transaction
        while(read(cin, trans)) {          // read the remaining transactions
            if (total.isbn() == trans.isbn())  // check the isbns
                total.combine(trans);     // update the running total
            else  {
                print(cout, total) << endl;     //print the results
                total = trans;        // process the next book
            }
        }
        print(cout, total) << endl;    // print the last transaction
    } else {                           // there was no input
        cerr << "No data?!" << endl;    // notify the user
    }
    ```

### 7.1.2 Defining the Revised Sales_data Class
* member 就跟 2.6.1(p.72) 定義的一樣
* 會有兩個 member function，isbn 跟 combine
* 還有一個額外的 member function 是拿來算平均的，avg_price，**他是 private member  function**
    * Not for general usage
    * It will be part of the implementation, not part of the interface.

* 宣告 member function 的方式跟宣告 ordinary function 的方式差不多
    * 一定要宣告(declare)在 class 裡面
    * 可以不用定義(define)在 class 裡面
    * 而那些構成 interface 但是是 ordinary function 的就宣告和定義在 class 外面就好

* 有了上述的知識，你可以寫一個基本的 class 宣告惹，包含了 data member，member function，跟 ordinary function
    ```cpp
    struct Sales_data { 
        // new members: operations on Sales_data objects
        std::string isbn() const { return bookNo; }
        Sales_data& combine(const Sales_data&);
        double avg_price() const;
        // data members are unchanged from § 2.6.1 (p. 72)
        std::string bookNo;
        unsigned units_sold = 0;
        double revenue = 0.0;
    };
    // nonmember Sales_data interface functions
    Sales_data add(const Sales_data&, const Sales_data&);
    std::ostream &print(std::ostream&, const Sales_data&);
    std::istream &read(std::istream&, Sales_data&);
    ```
    * 注意上面是用 struct 宣告 XD
    * Note: Functions **defined in the class are implicitly inline** (§ 6.5.2, p. 238).
        * 真的假的XDD

#### Defining Member Functions
* member function 可以定義在 class 宣告的外面，上面把 isbn 定義在 class 內，combine 和 avg_price 則定義在 class 外
* 先來解釋 isbn 這個已經定義好的 member function
    ```cpp
    std::string isbn() const { return bookNo; }
    ```
    * 其實長得跟 ordinary function 有 87%像
    * **但是 bookNo 打哪來的?**
#### **Introducing this**
* Primer p.300 會講一個例外(應該是 static member function)，不過我們在 call 一個 member function 時，實際上是透過某個 object 來呼叫 member function
    ```cpp
    total.isbn()
    ```
* 當 isbn 用到了某些 data member 時(例如 bookNo)，**他實際上是用到了呼叫他的那個 object 的 data member**
    * 上面這個例子，當 isbn 用到 bookNo，他實際上是用到 total.bookNo
* 為什麼會這樣呢?
    * Member functions access the object on which they were **called through an extra, implicit parameter named `this`.**
    * When we call a member function, **`this` is initialized with the address of the object** on which the function was invoked.
    * 當我們 call `total.isbn()`，compiler 就如同在 call **`Sales_data::isbn(&total)`** 一樣，會有一個隱性的 this 參數來接 object 的 address
        * Sales_data::isbn(&total)，雖然這是 pesudo-code，不過注意我用的是 scope operator，:: 左邊是 class name 不是 object name
* 總之，any direct use of a member of the class is assumed to be an implicit reference through this.
    * 你如果直接使用的 class 的 member(不管是 data 還是 function)，compiler 都假設微你是使用 this->member
    * 所以上面的 return bookNo; 就等於 return this->bookNo;

* **Because this is intended to always refer to “this” object, this is a *const pointer* (§ 2.4.2, p. 62). We cannot change the address that this holds.**

#### Introducing const Member Functions
* 再看一次 isbn 的定義
    ```cpp
    std::string isbn() const { return bookNo; }
    ```
    * 有沒有看到 parameter list 右邊有一個 const?
    * **The purpose of that const is to modify the type of the implicit this pointer.**
    * 上面說 this 是 const pointer，其實就是說 this 原本是 const pointer to object(也就是說這個 pointer 的 top-level 有 const，low-level 沒有)
    * member function 宣告成 const 就是讓 this 指標的 low-level 也賦予 const
        * 這樣有三小用處?
        * 有!請記得你所有使用的 member 都是透過 this(不管是沒有寫 this->member 還是直接使用 member)來存取的，既然 this 的 low-level 是 const，你就不能透過 this 來改 member 的值
        * **換句話說，宣告成 const 的 member function 不能更改任何 data member 的值!**
* 綜合上述還有一個重點，**如果你的 class object 宣告成 const，那就代表所有沒有宣告成 const 的 member function 你都不能 call**，因為 member function 都會隱性的傳入一個 this 進 function，而沒有 const 的 member function 傳進去的 this，他的型態是 class_type * const，換句話說他不能指向一個 const object！
* 上面的結論還引入一個觀念，就跟之前的類似
    * **如果你的 function 沒有要更改 member，請宣告成 const！**這樣 const object 也可以 call

* Note: **Objects that are const, and references or pointers to const objects, may call only const member functions.**

#### Class Scope and Member Functions
* Recall that a class is itself a scope
* The definitions of the member functions of a class are **nested inside the scope of the class itself.** Hence, isbn’s use of the name bookNo is **resolved as the data member defined inside Sales_data.**
* It is worth noting that isbn can use bookNo **even though bookNo is defined *after* isbn.**
    * 可以使用比 member function 更晚宣告的 member 是怎麼做到的呢?
    * p.283 會講到，C++ 是先把 class 的 member declaration 先編好，如果 class 內有定義的 code 再編，這樣的話再編定義的 code 時 member declaration 都已經有了

#### Defining a Member Function outside the Class
* 當然，definition 要 match class 內的 declaration
    * return type，name，parameter list，還有 member function 的 const 如果有的話也要寫!
* 要定義在外面的話格式是這樣
    ```cpp
    return_type class_name::member_function (parameter_list) (這裡按照宣告看是否要給 const)
        body
    ```
    ```cpp
    double Sales_data::avg_price() const { 
        if (units_sold) return revenue/units_sold;
        else return 0;
    }
    ```
    * The function name, Sales_data::avg_price, **uses the scope operator** (§ 1.2, p. 8) to say that we are defining the function named avg_price that is **declared in the scope of the Sales_data class.** 
    * Once the compiler sees the function name, *the rest of the code is interpreted as being inside the scope of the class.*


#### Defining a Function to Return "This" Object
* 這個 hen 重要
* 想想看我們的 Sales_data 的 combine 是對應到 Sales_item 的 +=，換句話說 combine 的行為要很像 +=，例如 result 是 lvalue 之類的，這要怎麼做呢?
    * total.combine(trans)，total 宛如 += 的 left operand，trans 宛如 += 的 right operand
    ```cpp
    Sales_data& Sales_data::combine(const Sales_data &rhs)
    {
        units_sold += rhs.units_sold; // add the members ofrhs into 
        revenue += rhs.revenue;       // the members of "this" object 
        return *this; // return the object on which the function was called
    }
    ```
    * 模擬 += -> return lvalue -> return reference type
    * 注意最後一行的 return *this;
    * **return the object on which the function was called**

* 當我們寫這樣的 code
    ```cpp
    total.combine(trans); // update the running total
    ```
    * &total 會 assign 給 this，而 trans 會被 bound 到 rhs
* 所以當在執行
    ```cpp
    units_sold += rhs.units_sold; // add the members ofrhs into
    ```
    * 其實就是把 total.units_sold 跟 trans.units_sold 相加然後存回 total.units_sold 裡

* 最後一行的 return
    * 再講一次，我們的 combine 其實是在 mimic +=，所以行為要跟 += 類似，例如會回傳 left operand 當 lvalue
    * 而 function 要能回傳 lvalue，他的 return type 一定要是 reference，既然我們說 total.comibe(trans)，total 就是當作 left operand 的角色，那 return 的 reference 就會是 reference to total 的 type，也就是 Sales_data&
* 但我們到底要怎樣把 total 這個 object 回傳當作 +=(或者說 combine)的 left operand?
    * **We need to use this to access the object as a whole**
    * 既然 this 現在指向 total(或者說指向 call 了 member function 的 object)，那我們回傳 *this 就是回傳這個 object 啦!
    ```cpp
    return *this;
    ```
    * For the call above, we return a reference to total.

### 7.1.3 Defining Nonmember Class-Related Functions
* 有些函數是 87% 的 class 可能都會用到的，例如 add，sub 之類的
* Although such functions define operations that are conceptually part of the interface of the class, they are not part of the class itself.
    * 因為 C++ 定義 interface 的方式，這些 functions 並不會宣告(定義)成 class 的 member function
    * 而這些 nonmember function 一樣會宣告在 class 宣告的那個 header，而定義通常也會定義在 class 定義 member function 的 source file 裡面
        * 這樣把 class 的 interface 跟這些 nonmember function 放在一起，class user 只要 include 一份 header 就可以使用整個 class 的 interface 了

#### Defining the read and print Functions
* 長的就跟 2.6.2 的有 87%像
    * 不用真的回去看啦...
    ```cpp
    // input transactions contain ISBN, number of copies sold, and sales price 
    istream &read(istream &is, Sales_data &item) {
        double price = 0;
        is >> item.bookNo >> item.units_sold >> price;
        item.revenue = price * item.units_sold; 
        return is;
    }
    ostream &print(ostream &os, const Sales_data &item) {
        os << item.isbn() << " " << item.units_sold << " " << item.revenue << " " << item.avg_price();
        return os;
    }
    ```
    * 你會發現到，read 第三個吃的是 price(這筆交易對應的每本書的價格)，然後把 price 跟 units_sold 相乘在丟給 revenue，換句話說 read 在隱形之中把這些事情做掉了，class user 完全不用自己去相乘
    * print 也是，除了吧 isbn units_sold 跟 revenue 印出來之外，還會呼叫 avg_price 把這本書的平均售價印出來，class user 一樣不用自己算

* 這兩個 functions 還有一個特點，他們各自其實是 mimic >> 跟 <<，而 stream_obj >> obj 會回傳 stream_obj 當 lvalue，以達成連續使用 >> 或 << 的功能。
* 這兩個 functions 如果要做類似的事情的話，要做跟上面 return *this; 當 referece 類似的事情，要把參數的 stream reference 用 reference 的方式 return 回去
* 所以 return type 分別是 istream& 跟 ostream&
* 不過 parameter 用 stream& 是因為，**Primer 說 stream object 不支援 copy，所以只能用 reference 來 bind，而且對 stream 做 >> << 會改變 stream obj，所以 parameter 都是 plain reference。**
* 還有一個點是有關於 convention 的問題，print 並沒有 print newline，user 要自己 print
    * Ordinarily, **functions that do output should do minimal formatting.** That way **user code can decide** whether the newline is needed.

#### Defining the add Function
* 
    ```cpp
    Sales_data add(const Sales_data &lhs, const Sales_data &rhs) {
        Sales_data sum = lhs; // copy data members from lhs into sum         
        sum.combine(rhs);
        // add data members from rhs into sum return sum;
    }
    ```
    * add 跟 combine 本質上不同，combine 其實是 mimic +=，add 是 mimic +
    * 所以 add 的 return 才會是 plain object 不是 reference
    * 不過 add 可以用 combine 實作就是了
        * **而且這個實作方式背後是有很多哲學的**
        * 某種程度上，+= 也可以用 + 實作
            * 但是這個 overhead 比較高
            * 詳情看 13/14 章
* 這裡有個小問題，sum = lhs 是三小? 我們沒有定義兩個 class object 怎麼 assign 阿?
    * *By default, copying a class object copies that object’s members.*

### 7.1.4 Constructors
# **BOSE**
* Each class defines how objects of its type can be initialized.
* **Classes control object initialization by defining one or more special member functions known as constructors.**
* **The job of a constructor is to initialize the data members of a class object.** 
* **A constructor is run whenever an object of a class type is created.**

* 這裡只會講怎麼寫基本的 constructor
    * constructor 其實非常ㄎㄧㄤ，7.5，15.7，18.1.3 還有 13 章會有更ㄎㄧㄤ的介紹

* Constructors have the same name as the class.
    * 沒有 return type
* A class can have multiple constructors.
    * Like any other overloaded function (§ 6.4, p. 230), the constructors must differ from each other in the number or types of their parameters.

* Unlike other member functions, constructors **may not be declared as const** (§ 7.1.2, p. 258). **When we create a const object of a class type, the object does not assume its "constness"** until after the constructor completes the object’s initialization. Thus, constructors can write to const objects during their construction.
    * constructor 不能宣告成 const，因為他一定要初始化 member，把 constructor 宣告成 const 不 make sense。

#### The Synthesized Default Constructor
* 阿奇怪惹，我們之前都沒定義 constructor 阿?
* Classes control default initialization by defining a special constructor, known as the **default constructor.** The default constructor is one that takes no arguments.
* If our class does not explicitly define any constructors, **the compiler will implicitly define the default constructor for us**
    * known as the **synthesized default constructor.**
    * 這個 constructor 會怎麼初始化我們的 class object 呢?
        1. 如果你的 class 的 data member 有給 in-class initializer 的話，就用那個 initializer 初始化 data member(class foo{int i = 87;}; 那個 87 就是 in-class initializer)
        2. 如果沒給的話，就 default-initialize data member
    * 用我們的 Sales_data 當例子，因為 units_sold 跟 revenue 都有給 in-class initializer，他們就用那個 initializer 初始化，而 bookNo 沒給，那就用那個 type，也就是 string，的方式 default initialize，所以 bookNo 初始值是 empty string

#### Some Classes Cannot Rely on the Synthesized Default Constructor
* 第一，你只要定義了任何一個 constructor，compiler 就不會幫我們 generate default constructor
* 只有簡單到不行的 class 才會不定義自己的 constructor 用 compiler 給的
* The basis for this rule is that **if a class requires control to initialize an object in one case, then the class is likely to require control in all cases.**
    * compiler 在這種情況下不定義 default ctor 的哲學: 你如果定義了一種 ctor 並且要自己定義如何初始化，很有可能在所有其他的情境(其他 ctors) 時你也要自己控制如何初始化
        * 這個情境也包含 default ctor
* A second reason to define the default constructor is that for some classes, the **synthesized default constructor does the wrong thing.**
    * 你有 built-in type member，然後你又沒有在 class 宣告時給這些 member 一個 in-class initializer，且你又只有 compiler 給的 default constructor，那你這個物件被初始化的時候這些 member 就會有 undefined value
        * 不過一個 practice 就是你可以把所有的 built-in member 都給上 in-class initailizer，這樣就不會有 undefined value

* 第三，some classes must define their own default constructor is that sometimes **the compiler is unable to synthesize one.**
    * 比方說你這個 class 有一個 member，他沒有 defalut constructor，那你這個 class compiler 一定沒辦法幫你 generate defalut constructor
    ```cpp
    class B {
      int number = 87;
      B(int i) { number = i; }
    };

    class A {
      B b;
    };

    int main() { A a; }
    ```
    * 上面這行 code 拿去編編看，A a; 會噴 error，而且 compiler 會跟你說他找不到 B::B()
* 所以結論是，一個 class compiler 只有在以下兩種情況都符合的時候會幫你生 default constructor
    1. 所有 member 都有 default constructor
    2. 你 class 沒有定義任何 constructor
    3. 13章好像還有一個例子會導致 compiler 不能產生 default constructor

#### Defining the Sales_data Constructors
* 我們會定義四種 Sales_data 的 constructors
    * An istream& from which to read a transaction.
    * A const string& representing an ISBN, an unsigned representing the count of how many books were sold, and a double representing the price at which the books sold.
    * A const string& representing an ISBN. This constructor will use default values for the other members.
    * An empty parameter list (i.e., the default constructor) which as we've just seen we must define because we have defined other constructors.
* 我們的 class 宣告就會變這樣
    ```cpp
    struct Sales_data { 
        // constructors added 
        Sales_data() = default;
        Sales_data(const std::string &s): bookNo(s) { }
        Sales_data(const std::string &s, unsigned n, double p): bookNo(s), units_sold(n), revenue(p*n) { }
        Sales_data(std::istream &);
        // other members as before
        std::string isbn() const { return bookNo; }
        Sales_data& combine(const Sales_data&);
        double avg_price() const;
        std::string bookNo;
        unsigned units_sold = 0;
        double revenue = 0.0;
    };
    ```
* ㄏㄏ上面有一堆東西要講，慢慢來
* 我們先看 defalut constructor
    ```cpp
    Sales_data() = default;
    ```
    * 那個 defalut 是三小?
    * We are defining this constructor *only becausewe* want to provide other constructors as well as the default constructor.
    * We want this constructor to **do exactly the same work as the synthesized version** we had been using.
    * C++11，Under the new standard, **if we want the default behavior, we can ask the compiler to generate the constructor for us by writing = default** after the parameter list.
        * 這種 = default 的寫法也只能用在沒有參數的 constructor 上，用在其他 constructor 會噴 error

#### Constructor Initializer List
* 再看另外兩個 constructor
    ```cpp
    Sales_data(const std::string &s): bookNo(s) { } 
    Sales_data(const std::string &s, unsigned n, double p):
        bookNo(s), units_sold(n), revenue(p*n) { }
    ```
    * parameter list 後面冒號到 function body 那坨是新語法，constructor initializer list
        * which specifies initial values for one or more data members of the object being created.
        * The constructor initializer **is a list of member names,** each of which is **followed by that member’s initial value in parentheses** (or inside curly braces). Multiple member initializations are separated by commas.
    * 第二個 constructor 有把所有 member 都給 initializer
    * 第一個則有些沒給，沒給的就會按照 synthesized default constructor 的方式來初始化
* It is usually best for a constructor to use an in-class initializer if one exists and gives the member the correct value.

* Best practice: Constructors **should not override in-class initializers except to use a different initial value.** If you can’t use in-class initializers, each constructor should **explicitly initialize every member of built-in type.**

* It is worth noting that both constructors have empty function bodies. *The only work these constructors need to do is give the data members their values. If there is no further work, then the function body is empty.*

#### Defining a Constructor outside the Class Body
* 剩下那個收 istream& 的 constructor 還沒講，這個 constructor 真的需要做事情，function 也不是空的，所以定義在 class 外面
    ```cpp
    Sales_data::Sales_data(std::istream &is) {
    read(is, *this);
    // read will read a transaction from is into this object
    }
    ```
    * 首先沒人規定 constructor 不能 call member function 或其他 function XD (read 是 global function)
    * 再來是如果定義在 class 外面，一樣不需要寫 return type，因為本來就沒有
    * In this constructor there is no constructor initializer list, although technically speaking, it would be more correct to say that the **constructor initializer list is empty.**
    * Even though the constructor initializer list is empty, ***the members of this object are still initialized before the constructor body is executed.***
    * 這很重要
        ```cpp
        CLASS() :i(1){}
        跟
        CLASS()
        {
           i = 1;
        }
        的差別
        ```
    * 總之就是 constructor 是在執行 constructor initializer list 時初始化 data member 的，即使是 empty list 也一樣；而進到 function body 之後，所有的 member，都已經按照特定的方式初始化完了(不管是 in-class 還是 default 還是三小)，***function 內的 assignment 都不是初始化!!!!!***


### 7.1.5 Copy, Assignment, and Destruction
* 這邊主要是說 class 也負責控制他們的物件在被 copy/assign/destroy 時的行為
    * 但只有講概念，細節在 13 章
* In addition to defining how objects of the class type are initialized, **classes also control what happens when we copy, assign, or destroy objects of the class type.**
* Objects are **copied in several contexts,** such as when we **initialize a variable** or when we **pass or return an object by value** (§ 6.2.1, p. 209, and § 6.3.2, p. 224).
* Objects are assigned when we use the **assignment operator**
*  Objects are **destroyed when they cease to exist,** such as when a local object is destroyed on **exit from the block in which it was created**
    * 或者被 delete，之後講
    * Objects stored in a vector (or an array) are destroyed when that vector (or array) is destroyed.
* If we **do not define** these operations, **the compiler will synthesize them for us.**
* Ordinarily, the versions that the compiler generates for us **execute by copying, assigning, or destroying *each member of the object.***
    * 拿 Sales_data 來說
        ```cpp
        total = trans; // process the next book
        ```
    * 宛如
        ```cpp
        // default assignment for Sales_data is equivalent to: 
        total.bookNo = trans.bookNo; 
        total.units_sold = trans.units_sold;
        total.revenue = trans.revenue;
        ```
* We’ll show how we can define our own versions of these operations in Chapter 13.
    * 好遠.....

* Some Classes Cannot Rely on the Synthesized Versions
    * 例如那些有指標的鬼東西
    * . In particular, the synthesized versions are unlikely to work correctly **for classes that *allocate resources that reside outside the class objects* themselves.**
    * 12 章會講 new delete，會再提到
    * 不過其實常常你可以使用 vector 或 string 來存你需要的資料，你不需要額外自行控管 dynamic memory
        primer 是說 generally should 啦...
    * 而且如果你用 vector 跟 string 來包裝你的 member，compiler 產生的 =，copy，delete 有 99.87% 都會是對的

* Warning: **Until you know** how to define the operations covered in Chapter 13, **the resources your classes allocate should be stored directly** as data members of the class.


## 7.2 Access Control and Encapsulation
* At this point, we have defined an interface for our class; **but nothing forces users to use that interface.**
    * not yet encapsulated

* public 跟 private
    * public 構成 class 的 interface
    * private sections encapsulate (i.e., hide) the implementation.

* 把我們的 Sales_data 改寫使得 interface 跟 implementation 分開
    ```cpp
    class Sales_data {
    public:
    // access specifier added
        Sales_data() = default;
        Sales_data(const std::string &s, unsigned n, double p): bookNo(s), 
        Sales_data(const std::string &s): bookNo(s) { }         
        Sales_data(std::istream&); std::string isbn() const { return bookNo; }     
        Sales_data &combine(const Sales_data&);units_sold(n), revenue(p*n) { }
    private:
    // access specifier added
        double avg_price() const { return units_sold ? revenue/units_sold : 0; }
        std::string bookNo;
        unsigned units_sold = 0;
        double revenue = 0.0;
    };
    ```
    * 現在外面的 code 都不能 access private member，你如果寫了會用到 private member 的 code 會直接噴 error
    * 一旦使用 access specifier，直到下一個 specifier 出現之前，所有的 member 的 access level 都跟這個 specifier 一致
* 上面的 code 還有一點，struct 改成 class 惹!
    * The only difference between struct and class is the default access level.
    * 不過會有一堆 convention 告訴你什麼時候該用 struct/class
        * As amatter of programming style, *whenwe define a class intending for all of its members to be public,we use struct. **If we intend to have private members, then we use class.***

### 7.2.1 Friends
* YYP: A 說 B 是他的朋友，不代表 B 覺得 A 是他的朋友
* 我們如果真的把 Sales_data 改成上面那樣，那那些構成 interface 的 **nonmember function** 就會 compile error，因為他們不是 member function，沒有 access private member 的權利。
    * 該怎麼辦呢?
* 用 friend!
    * A class can allow another class or function to access its nonpublic members **by making that class or function a friend.**
    ```cpp
    class Sales_data {
    friend Sales_data add(const Sales_data&, const Sales_data&);
    friend std::istream &read(std::istream&, Sales_data&);
    friend std::ostream &print(std::ostream&, const Sales_data&);
    public:
    // access specifier added
        Sales_data() = default;
        Sales_data(const std::string &s, unsigned n, double p): bookNo(s), 
        Sales_data(const std::string &s): bookNo(s) { }         
        Sales_data(std::istream&); std::string isbn() const { return bookNo; }     
        Sales_data &combine(const Sales_data&);units_sold(n), revenue(p*n) { }
    private:
    // access specifier added
        double avg_price() const { return units_sold ? revenue/units_sold : 0; }
        std::string bookNo;
        unsigned units_sold = 0;
        double revenue = 0.0;
    };
    ```
* 在 class 內把 nonmember function 宣告成 friend，nonmember function 就可以 accees class 的 private member 了

* KEY CONCEPT:BENEFITS OF ENCAPSULATION
    * 哲學課時間
    * Encapsulation provides two important advantages:
        * User code cannot inadvertently corrupt the state of an encapsulated object.
            * 就是強制讓你的 class object 只會在實作者定義的狀態間變換
            * 你自己想想你在用 OS 給的 C library 的時候有沒有這種功能?
            * 沒有!都他媽寫在文件裡! 你沒有任何 compiler 或 language syntax 的輔助跟你說你寫了糞 code!
                * 如果對我在講的垃圾話沒感覺，你可以去多用用 POSIX Library，你就會知道文件有多難啃，function 有多難理解，code 有多容易寫糞惹
        * The implementation of an encapsulated class can change over time without requiring changes in user-level code
            * 只要實作者定義了一個好的介面，並且都不改他，那跟這個介面接起來的 user code 就不用改，就算實作者更改了背後隱藏起來的實作，user code 也不會有感覺!

#### Declarations for Friends
* A friend declaration **only specifies access.** **It is not a general declaration of the function.**
    * 他真的不是宣告，宣告成 friend 的時候，compiler 會在 class scope 再往外一層的 scope 找這個 declaration
* 看似像宣告的 friend 語法其實不是宣告，按照標準的話，你還是要在 class 外面宣告這個 function，這樣 user code 在 include class header 時才看的到這個構成 class interface 的 nonmember function

## 7.3 Additional Class Features
* type members,
* in-class initializers for members of class type,
* mutable data members,
* inline member functions,
* returning *this from a member function,
* more about how we define and use class types,
* and class friendship.

### 7.3.1 Class Members Revisited
* Primer 再額外實作兩個 classes 來說明上述概念，Screen and Window_mgr

#### Defining a Type Member

* A **Screen** represents a window on a display. Each Screen has **a string member that holds the Screen’s contents**, and **three string::size_type members** that represent the **position** of the cursor, and the **height** and **width** of the screen.

* In addition to defining data and function members, a **class can define its own local names for types.**
    * 這些 class 內定義的 type 也有 access control，可以是 public 或 private
    ```cpp
    class Screen { 
    public:
        typedef std::string::size_type pos;
    private: pos cursor = 0; 
        pos height = 0, width = 0; 
        std::string contents;
    };
    ```
    * 你讓定義的 type name 也變成 user code 可用的 interface
    * 你會覺得這有什麼洨用，這可以讓 user code 不要去想說之後定義的 public function 要回傳高度寬度之類的，他的 type 其實是 size_type，而是一個 class 自定義的由名字就可解釋的 type... 反正就是一種抽象化
* 上面要用 using 取代 typedef 也可以
* 還有一點，p.284 會講到，type name 跟一般的 member 不一樣，我們在 member function 使用 type name 之前就一定要看到定義
    * 這跟之前說的 member 也可以定義在 member function 之後不一樣
* 所以需要把 type name 宣告在 class 定義的最前方

#### Member Functions of class Screen
* a *constructor* that will let users define the size and contents of the screen, along with *members to move the cursor and to get the character at a given location*

    ```cpp
    class Screen {
    public:
        typedef std::string::size_type pos;
        Screen() = default; // needed because Screen has another constructor     
        // cursori nitialized to 0 by its in-class initializer 
        Screen(pos ht, pos wd, char c): height(ht), width(wd), contents(ht * wd, c) { } 
        char get() const   // get the character at the cursor 
            { return contents[cursor]; } // implicitly inline
        inline char get(pos ht, pos wd) const; // explicitly inline 
        Screen &move(pos r, pos c); // can be made inline later
    private: 
        pos cursor = 0;
        pos height = 0,
        width = 0;
        std::string contents;
    };
    ```
    * 上面一些注意事項
        * 三個參數的 ctor 雖然有用 ctor initializer list，不過他沒初始化到 cursor，他用 in-class initialzer 初始化 cursor
        * 還有注意看他怎麼初始化 contents，**他給了兩個參數!!**，因為 string 可以用兩個參數來初始化
        * 其實 contructor initializer list 就是在 call member 的 constructor，所以才可以很ㄎㄧㄤ的給兩個參數給 str，因為 string 本來就可以給兩個參數來初始化
        * 上面的 get 有 overloading

#### Making Members inline
* Classes often have small functions that can benefit from being inlined.
* 可以直接在 class 定義裡面宣告 function 時加上 inline，然後 definition 寫在 class 定義的外面，這樣還是會 inline
    * 你也可以宣告跟定義都寫，甚至可以只寫在定義
        * https://isocpp.org/wiki/faq/inline-functions#where-to-put-inline-keyword
        * 上面有到底該寫在哪的 best practice
        * If "it" can’t be observed from the caller’s code, "it" shouldn’t be in the public: part of the class body.
    ```cpp
    inline // we can specify inline on the definition
    Screen &Screen::move(pos r, pos c) {
        pos row = r * width; // compute the row location 
        cursor = row + c; // move cursor to the column within that row 
        return *this;
        // return this object as an lvalue
    }
    char Screen::get(pos r, pos c) const // declared as inlinein the class
    {
        pos row = r * width; // compute row location
        return contents[row + c]; //return character at a given column
    }
    ```
    * 上面的 move 在 class 內宣告時沒給 inline，get 則是在宣告時有給 inline，兩者最後都會發 inline request 給 compiler
        * 但是只寫在 member function definition 的話，class definition 會比較好讀
        * 而且"到底要不要宣告這個不會 default inline 的 member function 為 inline" 其實也是一種實作細節，只看 class definition 的 public parts 時其實不需要在意有無 inline
* inline member function 還有一個重點!
    * For the same reasons that we define inline functions in headers (§ 6.5.2, p. 240), **inline member functions should be defined in the same header as the corresponding class definition.**
    * 再提醒一次，compiler 在編譯的時候決定到底要不要把 function inline 之前他一定要知道定義阿，所以 inline function 要定義(define)在 class 定義的地方(或者說定義要跟 class 定義在同一個 translaton unit)，不然 compiler 會看不到

#### Overloading Member Functions
* ctor 都可以 overload 了，member function 當然也可以
    * 怎麼定義不會衝突，以及 call function 時怎麼 match 都跟第六章講的一樣
* 這個例子定義了兩個 get，兩個的參數數量不同所以不會搞混

#### mutable Data Members
* 凱特大嬸! 這東西 1993 年就有惹ㄏㄏ
    * 這他媽還是個 keyword
* It sometimes (but not very often) happens that **a class has a data member that we want to be able to modify, *even inside a const member function.***
    * We indicate such members by ***including the mutable keyword* in their declaration.**

* **A mutable data member is never const, even when it is a member of a const object.**
    * 就算 function 宣告為 const，你還是可以在這個 function 內改變 mutable member 的 value
* As an example, we’ll give Screen a mutable member named **access_ctr, which we’ll use to track how often each Screen member function is called:**
    ```cpp
    class Screen { public: void some_member() const;
    private:
        mutable size_t access_ctr; // may change even in a const object 
        // other members as before
    };
    void Screen::some_member() const {
        ++access_ctr; // keep a count ofthe calls to any member function 
        // whatever other work this member needs to do
    }
    ```
    * some_member 是 const function，可是還是可以改宣告為 mutable 的 access_ctr
    
#### Initializers for Data Members of Class Type
* 上面說的還會定義另一個 class，這裡出現惹
* In addition to defining the Screen class, we’ll define a **window manager** class that **represents a collection of Screens** on a given display.
    * have a vector of Screens
    * By default, we’d like our Window_mgr class to *start up with a single, default-initialized Screen.*
        * 在 C++11，這件事情最簡單的達成方法就是用 in-class initializer
    ```cpp
    class Window_mgr {
    private: // Screens this Window_mgr is tracking 
    // by default, a Window_mgr has one standard sized blank Screen             
    std::vector<Screen> screens{Screen(24, 80, ' ')};
    };
    ```
    * 有沒有突然覺得這樣寫可以幹掉一堆事情很爽www
    * Primer p.274 說 in-class initializer 只能用 = 或者 {} 這兩種
    * 你可以想想看為啥不能用 () 那種
        * vector\<int> ivec(94, 87);
    * 跟 member function 宣告 syntax collision 啦!
    * https://stackoverflow.com/questions/24836526/why-c11-in-class-initializer-cannot-use-parentheses
    * 不過你還是可以用 = 搭配 ctor 啦
        * vector\<int> ivec = vector\<int>(94, 87); 
    * 不然乾脆都放在 constructor initializer list 阿 LOL
        * 別，在 C++11 後能用 in-class 就要優先用


### 7.3.2 Functions That Return \*this 
* 接下來要寫一些 set 背景 character 的 function，也有 overload，分別是 set 當前 cursor 位置的字元，或者給定位址設定該位址的字元

    ```cpp
    class Screen {
    public:
        Screen &set(char);
        Screen &set(pos, pos, char);
        // other members as before
    };
    inline Screen &Screen::set(char c) {
        contents[cursor] = c; // set the new value at the current cursor location 
        return *this;
        // return this object as an lvalue 
    }
    inline Screen &Screen::set(pos r, pos col, char ch) {
        contents[r*width + col] = ch; // set specified location to given value 
        return *this;
        // return this object as an lvalue
    }
    ```
    * our set members return a reference to the object on which they are called
    * 聯合之前的 move, which also returns lvalue:
        ```cpp
        // move the cursor to a given position, and set that character
        myScreen.move(4,0).set(’#’);
        ```
    * 就可以這樣 call member function
    * concatenate a sequence of these actions into a single expression
    * **execute on the same object.**
    * 不過其實如果你這兩個 function return 的不是 reference，一樣可以這樣子連 call，可是第二個 call 的 member function 就只會作用在 temporary opject 上，不但行為完全不一樣，還會有一個複製 object 的 overhead
    
#### Returning \*this from a const Member Function 
* 再加一個 member function 叫做 display，把 contents 印粗乃
* 我們希望這個 display 也可以跟 move 還有 set 一樣在一個 object 上面連 call，所以他應該要 return reference
* 但是邏輯上來說 display 應該要宣告成 const，因為他不會改變 object
    * 可是這樣 display 回傳的 this 就會是 low-level const，對 \*this call set 就會是 error，因為 set 不是 const function
    ```cpp
    Screen myScreen; // if display returns a const reference, the call to setis an error
    myScreen.display(cout).set(’*’);
    ```
    * 那這樣連 call 的技巧就廢掉啦! 怎麼辦ㄋ
#### Overloading Based on const
* We can overload a member function based on whether it is const for the same reasons that we can overload a function based on whether a pointer parameter points to const (§ 6.4, p. 232).
    * 超饒舌，我們可以 overload const member function，其原因跟下面一樣：我們可以 overload function，如果他的參數的 low-level const 不一樣的話
        * 第六章的再複習一次，如果你真的按照上面說的 overload ordinary funtion，那 const object 就沒辦法用 low-level 是 nonconst 的版本，因為 low-level 是 nonconst 的參數，也就是 pointer 或 reference to nonconst，沒辦法指向或綁定 const object
        * 如果你是 nonconst object，你兩種 function 都可以 call，不過根據第六章的 match rule，nonconst 版本的會是 better match
* C++ Primer 會這樣做 overload：display 其實是個 wrapper，把 do_display 包起來，然後 display 再回傳 \*this。
    * do_display 只有一個 const 版本，他是 private member function，user code 也用不到
        * **而且 const 版的 display 也只能 call const function，如果 do_display 不是 const 他就不能 call 了**
    ```C
    class Screen { 
    public: // display overloaded on whether the object is const or not 
    Screen &display(std::ostream &os)
        {  do_display(os); return *this; }
    const Screen &display(std::ostream &os) const
        {  do_display(os); return *this; }
    private: 
        // function to do the work of displaying a Screen 
        void do_display(std::ostream &os) const {os << contents;} 
        // other members as before
    };
    ```
    * 再說一件事 const member function 只有說不能改 non mutable 的 **member**，nonmember 隨便你，比方說上面 do_display 傳進來的 os 就不是 const，隨便你亂改都可以
    * do_display 既然是 member function，那他其實也會隱性的收到 this，當 const object call 他時，傳進來的 this 的 low-level 是 const；當 nonconst object call 他時，因為 do_display 是 const，傳進來的 this 的 low-level 一樣會被轉成 const。
* 結論是，when we call display on an object, **whether that object is const determines which version of display is called:**
    ```cpp
    Screen myScreen(5,3);
    const Screen blank(5, 3); myScreen.set(’#’).display(cout); // calls nonconst version
    blank.display(cout); // calls constversion
    ```
* ADVICE: USE PRIVATE UTILITY FUNCTIONS FOR COMMON CODE
    * 這是根據上面 do_display 的 coding practice 所做的 advice
    * 你會覺得很ㄎㄧㄤ，一行 call do_display 跟一行 os << contents 比起來好像沒有比較簡單，這到底在銃三小?
    * We do so for several reasons:
        * A general desire to **avoid writing the same code in more than one place.**
        * We **expect that the display operation will become more complicated as our class evolves.** As the actions involved become more complicated, it **makes more obvious sense to write those actions in one place, not two.**
        * 除了上面一點說的，我們可能還會對 do_display 內的 code 額外加一些 debugging code，如果 do_display 的 code 只有一份會比較簡潔
        * 最後，這裡把 do_display 的定義寫在 class 內，換句話說預設是 inline，而且又那麼短，基本上一定會 inline，所以 runtime 不會有 function call overhead
    * In practice, **well-designed C++ programs tend to have lots of small functions such as do_display that are called to do the "real" work** of some other set of functions.


### 7.3.3 Class Types
* Every class defines a unique type.
* Two different classes define two different types even if they define the same members.
* 宣告 class object 的時候可以用 C 的方式宣告
    * class class_name obj;
    * struct struct_name obj;
    * 有沒有加 `class/struct` keyword 都可以

* We can also declare a class without defining it:
    ```cpp
    class Screen; // declaration of the Screen class
    ```
    * 之前偷用過ㄌ
    * sometimes **referred to as a forward declaration, introduces the name Screen into the program** and indicates that Screen refers to a class type.
    * **After a declaration and before a definition** is seen, the type Screen is an **incomplete type**—it’s known that Screen is a class type but not known what members that type contains.
    * We can **use an incomplete type in only limited ways:** We **can define pointers or references to such types,** and we **can declare *(but not define)* functions that use an incomplete type** as a parameter or return type.

* A class **must be defined—*not just declared*—before we can write code that creates objects** of that type.
    * Otherwise, the compiler does not know how much storage such objects need.
* Similarly, the class must be defined before a reference or pointer is used to access a member of the type. 
* With one exception that we’ll describe in § 7.6 (p. 300), **data members can be specified to be of a class type only if the class has been defined.** 
    * The type must be complete because the compiler needs to know how much storage the data member requires.
    * 簡單說，你要定義一個 class，他所有會用到的 member 都要已經定義了才行
        * 可以想成是，都已經要定義 class 了，所以必須知道 member 要怎麼創建，要知道怎麼創建，就要知道 member 的定義
* 因上一個的原因，class 沒辦法在定義的時候說有一個 member object 跟自己同 type，因為這個 class 還沒定義完阿!
    * 雞生蛋蛋生雞
* 但是 class 內可以有 pointer to 這個 class，因為 pointer 只要看到 declare 就可以定義了。

* 其實我覺得這就只是用C++的方法解釋 C 的這種鬼東西?
    ```cpp
    class Link_screen {
        Screen window;
        Link_screen *next;
        Link_screen *prev;
    };
    ```
    
### 7.3.4 Friendship Revisited
* A class can also make another class its friend or it can declare specific member functions of another (previously defined) class as friends.
* In addition, a friend function can be defined inside the class body. Such functions are implicitly inline.
* 別的 class 也可以宣告為自己的 friend
* 別的 class 的 member function 也可以...
* 用 Window_mgr 跟 Screen 來解釋
    * Window_mgr 需要碰到 Screen 的 private member
        * 有個 clear member function，會把 Screen 的 contents 全部設成 blank
    * 把 Window_mgr 設成 Screen 的 friend
    ```cpp
    class Screen {
        // Window_mgr members can access the private parts of class Screen
        friend class Window_mgr;
        // ... rest of the Screenclass
    };
    ```
* The member functions of a friend class **can access all the members, including the nonpublic members,** of the class granting friendship.
* 把 Window_mgr 宣告成 Screen 的 friend 之後 clear 就可以這樣寫
    ```cpp
    class Window_mgr {
    public: // location ID for each screen on the window 
        using ScreenIndex = std::vector<Screen>::size_type;
        // reset the Screen at the given position to all blanks
        void clear(ScreenIndex);
    private:
        std::vector<Screen> screens{Screen(24, 80, ' ')};
    };
    void Window_mgr::clear(ScreenIndex i) {
        // s is a reference to the Screen we want to clear
        Screen &s = screens[i];
        // reset the contents of that Screen to all blanks
        s.contents = string(s.height * s.width, ' ');
    }
    ```
    * clear contents, height, width 都用到了
* friend is not transtive
    * 現在是 Screen 把 Window_mgr 設成 friend，但是 Window_mgr 的 friend 不會因此可以 access Screen 的 private member。
    * 幹，你一定沒有看懂原文
    * 原文的意思是，A 宣告 B 為 friend，B 宣告 C 為 friend，但這時 A 的 friend 一樣沒有 C

* Note: Each class controls which classes or functions are its friends.

#### Making AMember Function a Friend
* 這麼粗暴的把整個 class 都列為 friend 太ㄎㄧㄤ，其實你可以指定說某個 class 的某些 member function 要當 friend 就好了
    ```cpp
    class Screen {
        // Window_mgr::clear must have been declared before class Screen 
        friend void Window_mgr::clear(ScreenIndex);
        // ... rest of the Screenclass
    };
    ```
* 可是你想嘛，你要說某 class 的某 member function 是我的 frined，那某 class 就不能只是單單看到 forward declaration(class class_name; 這種)而已了，
* Making a member function a friend **requires careful structuring of our programs to accommodate interdependencies among the declarations and definitions.**
* 假設 class A 的 member function f_a 要當 class B 的 friend，那你的 code structure 要長這樣
    1. 先**宣告** class B
    2. 再**定義** class A，可是裡面的 f_a 用**宣告**的就好，不能用定義的，因為目前 class B 只有**宣告**而已
    3. 再定義 class B
    4. 最後再定義 f_a
* 你可以試著調換上述順序，你會發現需要定義的時候還沒定義，或者需要宣告的時候還沒宣告，諸如此類的
    * 操你媽的如果分寫在不同檔案怎麼辦.......zzzz

#### Overloaded Functions and Friendship
* 你如果要把某些 overloaded functions 全部變成某 class 的 friend，那你全部都要寫在 class 內宣告 friend


#### Friend Declarations and Scope
* 如果只是要把某 class A 或某 nonmember function F 宣告成 class B 的 friend，在宣告成 friend 之前，A 跟 F 甚至可以不用宣告
* 為什麼可以這樣呢?
* **When a name first appears in a friend declaration, that name is implicitly "assumed" to be part of the surrounding scope.**
* However, the **friend itself is not actually declared** in that scope (§ 7.2.1, p. 270)
* **所以 friend 其實會去 class scope 再往外一層的 scope(也就是 surrounding scope) 找 name**
* 不過 friend nonmember funcion 還是可以定義在 class 裡面
    * 感覺就糞 code...
    * **不過你就算定義在 class 裡面，你還是要在 class 外面提供一分宣告。**
    * 即使我們只是從宣告了 friend function 的 class 的 member call 了這個 friend function，我們還是要在 class 外面宣告這個 friend function，不然 member function 會找不到這個 friend function。
    ```cpp
    struct X {
        friend void f() { /* friend function can be defined in the class body */ }
        X() { f(); } // error: no declaration for f 
        void g();
        void h();
    };

    void X::g() { return f(); } // error: f hasn’t been declared
    void f(); // declares the function defined inside X
    void X::h() { return f(); } // ok: declaration for fis now in scope
    ```
    
* It is important to understand that a friend declaration affects access **but is not a declaration in an ordinary sense.**
    * 不過有些 compiler 可以接受就是zzz
    * g++ 不行，水喔


## 7.4 Class Scope
* Every class defines its own new scope.
* 要取得 class scope 內的 name，要馬隊 class_name 用 ::，要馬對 class object 用 .，指標用 ->
* class 自成一個 scope 的事實解釋了為什麼我們把 member function 定義在 class 外面時需要加上 class_name::
    * **因為在 class 外，member name 被隱藏起來了**

* 而且一旦你用了 class_name:: 之後，class_name 的所有 member 你都看的到了，這也是為什麼下面的 code 合法的原因
    ```cpp
    void Window_mgr::clear(ScreenIndex i) {
        Screen &s = screens[i];
        s.contents = string(s.height * s.width, ’ ’);
    }
    ```
    * 照理來講 ScreenIndex 是定義在 Window_mgr 內的一個 type member，可是因為這裡用了 Window_mgr:: 宣告 member function，跟在他之後定義的所有東西在查找 name 的時候都看的到 Window_mgr 這個 scope 了，所以 compiler 也看的到在 Window_mgr 內的 ScreenIndex
       * Once the **class name is seen,** the remainder of the definition—including the parameter list and the function body—**is in the scope of the class.**
    * function body 內的 screens 也是同理
* 但是定義再 class 外面的 member function 的 return type 就不是這樣了
    * 你想想看，return type 是出現在 class_name:: 之前，所以在看 return type 時是在 class_name 這個 scope 之外的
* 如果 return type 是定義在 class_name 裡面，那你的 return type 就要給 class_name:: 這個 prefix 來告訴 compiler 這個 type 是定義在 class_name 裡面
* 舉個例子，假設現在 Window_mgr 要額外加一個 function 可以讓 user code 新增一個 screen，然後這個 function 會回傳新的 screen 的 ScreenIndex，因為 ScreenIndex 這個 type 是定義在 Window_mgr 裡面，所以要這樣寫
    ```cpp
    class Window_mgr { 
    public: 
        // add a Screen to the window and returns its index 
        ScreenIndex addScreen(const Screen&); /
        / other members as before
    }; 
    // return type is seen before we’re in the scope of Window_mgr
    Window_mgr::ScreenIndex 
    Window_mgr::addScreen(const Screen &s) {
        screens.push_back(s); return screens.size() - 1;
    }
    ```
    * 注意上面定義在 class 外面的 member function，return type 要加上 Window_mgr::
* 另外補充，這邊 primer 居然沒有解釋 member function 的 trailing return type 已經在 class scope 所以不用加上 `class_name::`，覺得母湯


### 7.4.1 Name Lookup and Class Scope
* In the programs we’ve written so far, **name lookup (the process of finding which declarations match the use of a name)(神定義)** has been relatively straightforward:
    * First, look for a declaration of the name in the block in which the name was used. Only names declared before the use are considered.
    * If the name isn’t found, look in the enclosing scope(s).
    * If no declaration is found, then the program is in error.
* The way names are resolved inside member functions defined inside the class may seem to behave differently than these lookup rules.
    * 比方說你貌似可以在 member function 內用比 member function 還之後宣告的 member
    * 但那是假的! 原因如下

* Class definitions are processed(compiled) in two phases:
    * First, the member declarations are compiled. 
    * Function bodies are compiled only after the entire class has been seen.

* Note: Member function **definitions**(也就是 function body) are processed after the compiler processes all of the declarations in the class.

* Classes are processed in this two-phase way to **make it easier to organize class code.**

* Because member function bodies are not processed until the entire class is seen, they can use any name defined inside the class.
    * 所以 member function 才可以用比他之後還晚宣告的 member，因為在處理 member function body 的時候，compiler 已經把整個 class 的 declaration 的部分都看完了。
* 如果不按照上面的方法處理 class，亦即用原本處理 ordinary function 的方法，要用一個 name 之前 name 一定要宣告，*那我們就一定要把某 member function 會用到的 name(包括其他 member function)都宣告(注意，只要宣告就好不用定義)在這個 member function 之前*

#### Name Lookup for Class Member Declarations
* This two-step process applies only to names used in the body of a member function.
* 上面說的兩部處理只在 member function **的 body 適用**
* 宣告 member function 時，用的 return type, paramete list 用到的 type，還是必須先宣告過。
* 如果你宣告的 member function 用到的 name 目前還不存在在 class 定義內，那 compiler 會直接往宣告 class 的 scope 找 name。

    ```cpp
    typedef double Money;
    string bal;
    class Account {
    public:
        Money balance() { return bal; }
    private:
        Money bal; // ...
    };
    ```
    * 上面這例子 demo compiler 怎麼 look up name
    * 在 member function balance 的**宣告部分**，return type 是用一般的方式找的，而 Money 目前還沒出現，所以往外找，找到 typedef dounle Money
    * 在 balance 的 body 內，return bal 是用 class 兩段式找法找 name 的，所以他看的到 private member bal，return 的就是這個 private member bal

#### Type Names Are Special
* 如果 class 外面定義了某個 name 為 type 的話，class 內部不能再重新定義這個 name
* 只有 type name 才這樣

    ```cpp
    typedef double Money;
    class Account {
    public:
      Money balance() { return bal; } // uses Money from the outer scope
    private:
      typedef double Money; // error: cannot redefine Money
      Money bal;
    };
    ```
    
* 原因:
    * https://stackoverflow.com/questions/45384956/why-cant-redefine-type-names-in-class-in-c
* Tip: **Definitions of type names usually should appear at the beginning of a class.** That way any member that uses that type will be seen after the type name has already been defined.


#### Normal Block-Scope Name Lookup inside Member Definitions

* 當在 member function body 內做 name look up 時會按照下面方式
    1. 如果在 body 內這個 name 被使用的 code 之前就看到 name 的宣告，就用那個 name
    2. 如果上述不成立，就從整個 class 找，所有 class 的 member 都會被檢查
    3. 如果上述不成立，就從 member function 被定義之前的 code 找，找不到就噴 error

* 最好不要讓 member function 參數的名字跟其他 member 的名字 collision；下面是為了 demo 用而寫的糞 code
    ```cpp
    // note: this code is for illustration purposes only and reflects bad practice
    // it is generally a bad idea to use the same name for a parameter and a member 
    int height;
    // defines a name subsequently used inside Screen 
    class Screen {
    public:
        typedef std::string::size_type pos;
        void dummy_fcn(pos height) {
            cursor = width * height; // which height? the parameter
        }
    private:
        pos cursor = 0;
        pos height = 0,
        width = 0;
    };
    ```
    * In this case, the height parameter hides the member named height.
    * 你真的要在這種情況下使用 member height，要用 this-> 或者 Screen::
    ```cpp
    // bad practice: names local to member functions shouldn’t hide member names 
    void Screen::dummy_fcn(pos height) { 
        cursor = width * this->height; 
        // member height 
        // alternative way to indicate the member 
        cursor = width * Screen::height; 
        // member height
    }
    ```
* 總之 member function 內宣告的變數，包括參數跟 local variable，都不要跟 member name collision

#### After Class Scope, Look in the Surrounding Scope
* ㄜ就這樣..
* 如果你想要 access 一個被 inner scope hide 的 outer scope variable，你可以用 scope operator，例如上面的 height，最外面的 scope 也有一個
    * 可以用 ::height 存取

#### Names Are Resolved Where They Appear within a File
* When a member is defined outside its class, the **third step** of name lookup **includes names declared in the scope of the member definition as well as those that appear in the scope of the class definition.** 
    * 在工三小呢?
    * 第三部就是說，member 的宣告跟 memeber 的定義的 scope 都會考慮
* ![](https://i.imgur.com/k4Mum6Q.png)
    * 上面的例子，setHeight 的定義用到了 verify，雖然在 setHeight 宣告的時候 verify 不在 scope 內，可是在 setHeight 定義的時候 verify 已經在 scope 內了，所以 compiler 還是找的到定義在 seiHeight 內的 verify 是誰。


## 7.5 Constructors Revisited
* 多講一點

### 7.5.1 Constructor Initializer List
* 當我們宣告一般變數時，如果想要給特定值，通常會提供 initializer；而不是先宣告，然後再在下一行給值
    * string s1 = "hsilu"; // good
    * string s2; s2 = "hsilu"; // bad
* constructor initializer list 的概念就跟上面一樣
* If we do not explicitly initialize a member in the constructor initializer list, **that member is default initialized *before the constructor body starts executing.*** For example:
    ```cpp
    Sales_data::Sales_data(const string &s, unsigned cnt, double price)
    {
        bookNo = s;
        units_sold = cnt;
        revenue = cnt * price;
    }
    ```
    * 概念上來說，data member 會在 ctor 的 body 開始執行之前就完成初始化
    * 而這裡只有提供一個 empty ctor initializer list，所以所有 member 都是 default initialize
    * 初始化完之後再執行 ctor 的 body，**此時 body 內的 = 全部都是 assignment，不是初始化
* 上面的 code 最終結果跟之前的有用 ctor init list 的版本一樣
* 可是你用 ctor init list 初始化跟用 assignment 給值，這兩種的差異要看被初始化的 member 是什麼 type，有可能效果一樣，有可能不同
    * How significant this distinction is depends on the type of the data member.

* Constructor Initializers Are Sometimes Required
    * 既然都說了 ctor body 內的 = 都是 assignment，那就**意味著那些不能用 = 來給值的 member 要在 ctor init list 就初始化!** 例如 const object 跟 reference，**還有那些不能 default initialize 的 class object!**
    ```cpp
    class ConstRef {
    public:
        ConstRef(int ii);
    private:
        int i;
        const int ci;
        int &ri;
    };
    // error: ci and ri must be initialized     
    ConstRef::ConstRef(int ii) {
        // assignments: 
        i = ii; // ok 
        ci = ii; // error: cannot assign to a const 
        ri = i; // error: ri was never initialized
    }
    ```
* 再說一次，結論就是**當 ctor 的 body 開始執行時，class member 的初始化就已經全部完成了**
* 上面錯誤的 ctor 應該寫成這樣
    ```
    ConstRef::ConstRef(int ii): i(ii), ci(ii), ri(i) { }
    ```
* We must use the constructor initializer list to provide values for members that are const, reference, or of a class type that does not have a default constructor.

* ADVICE: USE CONSTRUCTOR INITIALIZERS
    * practice 就是用 ctor init list，因為如果你的 ctor 想要主動對 member 做初始化，你本來就應該寫在 ctor init list 裡面，而不是在 ctor body 用 assigment 的方式給值。
    * 而且就像上面說的，有些 member 是不能用 assignment 給值的，直接用 ctor init list 可以不用去顧慮到哪些 member 不能用 =

#### Order of Member Initialization
* ctor init list 給定 member initializer 的順序一點意義都沒有
    ```cpp
    class_name::class_name(int i, int j, int k): member_i(i), member_j(j), member_k(k);
    class_name::class_name(int i, int j, int k): member_k(k), member_i(i), member_j(j);
    ```
    * 上面兩個是等價的
* 真正影響到 class member 初始化順序的是 member 在 class 定義內宣告的順序
* 如果你有 member 是依賴定一個 member 來初始化的，這時候 class 定義 members 的順序跟 ctor init list 給的順序如果有差異可能就會有問題了(UB)。
    ```cpp
    class X { 
        int i;
        int j;
    public: 
    // undefined: i is initialized before j     
        X(int val): j(val), i(j) { }
    };
    ```
    * class 內明明就是先初始化 i 再 j，你卻在 ctor init list 說 i 要靠 j 來初始化，這是 UB
    * 上面的 ctor 寫成
        * X(int val): i(j), j(val) { }
    * 也可以，反正順序不重要，可是這時候你就會感受到怪異了，上面 code section 內的寫法看起來就很正常，可是 list 的順序不重要，是假的!

* Some compilers are kind enough to generate a warning if the data members are listed in the constructor initializer *in a different order from the order in which the members are declared.*

* BEST PRACTICE: It is a good idea to **write constructor initializers in the same order as the members are declared.** Moreover, when possible, **avoid using members to initialize other members.**
* It is a good idea **write member initializers to use the constructor’s parameters** rather than another data member from the same object.
    * 如果 member 彼此初始化方式是獨立的，那我們根本不用管 member 宣告的順序惹
    ```cpp
    X(int val): i(val), j(val) { }
    ```
    上面 ctor 根本不用管 i, j 宣告在 class 內的順序
    
#### Default Arguments and Constructors
* 如果某 ctor 他所有 parameter 都有 default value，亦即可以不用給參數，他就會等價於 default ctor。
    * 你這時候反而不能定義一個真的沒參數的 default ctor，因為當你真的宣告一個不吃參數的 object，會 ambiguous call

### 7.5.2 Delegating Constructors
* A delegating constructor uses another constructor from its own class to perform its initialization.
    * It is said to "delegate" some (or all) of its work to this other constructor.
        * delegate: 把工作丟給別人做(?
    
* Like any other constructor, a delegating constructor has a member initializer list and a function body.
    * the member initializer list **has a single entry that is the name of the class itself.**
    * followed by a parenthesized list of arguments.
    * The argument list must match another constructor in the class.
    * 其實就是 call 另一個 ctor 的意思

```cpp
class Sales_data {
public:
    // nondelegating constructor initializes members from corresponding arguments
    Sales_data(std::string s, unsigned cnt, double price): 
        bookNo(s), units_sold(cnt), revenue(cnt*price) { }
    // remaining constructors all delegate to another constructor
    Sales_data(): Sales_data("", 0, 0) {} 
    Sales_data(std::string s): Sales_data(s, 0,0) {}
    Sales_data(std::istream &is): Sales_data()
        { read(is, *this); }
    // other members as before 
};
```
* 第一個之外的 ctor 都把部分工作丟給第一個 ctor 做
* The constructor that takes an istream& also delegates. It delegates to the default constructor, *which in turn delegates to the three-argument constructor.*
    * 看看第四個吃 istream& 的 ctor，實際上 delegate 了兩次


* When a constructor delegates to another constructor, **the constructor initializer list and function body of the delegated-to constructor are both executed.**
* delegated-to ctor 的 body 會在 delegating ctor 的 body 之前執行。
    * 後進先出
    * In Sales_data, the function bodies of the delegated-to constructors happen to be empty. **Had the function bodies contained code, that code would be run *before control returned to the function body of the delegating constructor*.**


### 7.5.3 The Role of the Default Constructor

* 複習一下 default initialization 跟 value initialization
* Default initialization happens 
    * When we define nonstatic variables (§ 2.2.1, p. 43) or arrays (§ 3.5.1, p. 114) at block scope without initializers
    * When a class that itself has members of class type uses the synthesized default constructor (§ 7.1.4, p. 262)
    * When members of class type are not explicitly initialized in a constructor initializer list (§ 7.1.4, p. 265)

* Value initialization happens 
    * During array initialization when we provide fewer initializers than the size of the array (§ 3.5.1, p. 114)
    * When we *define a local static object without an initializer* (§ 6.1.1, p. 205)
    * When we explicitly request value initialization by writing an expressions of the form T() where T is the name of a type 
        * The vector constructor that takes a single argument to specify the vector’s size (§ 3.3.1, p. 98) uses an argument of this kind to value initialize its element initializer.

* Classes must have a default constructor in order to be used in these contexts.
    * class object 要能達到上面兩種初始化，class 一定要提供 defalut ctor

    ```cpp
    class NoDefault {
    public:
        NoDefault(const std::string&);
        // additional members follow, but no other constructors
    };
    struct A { 
        // my_mem is public by default; see § 7.2 (p. 268) 
        NoDefault my_mem;
    }; 
    A a; // error: cannot synthesize a constructor for A
    struct B { 
        B() {} // error: no initializer for b_member 
        NoDefault b_member;
    }
    ```
    * 如果你要為一個 class 寫 default ctor，但是存在一個 member 不能 default initialize，那你一定要在這個 default ctor 的 ctor init list 內初始化這個 member，不然就會像上面 B 的預設 ctor 一樣噴 error

* Best Practice: In practice, **it is almost always right to provide a default constructor if other constructors are being defined.**
    * 你就強制讓你的每個 class 都提供預設 ctor 就不用擔心會發生上面的問題了

#### Using the Default Constructor
* 注意宣告使用 default ctor 初始化物件的語法
    * `class_name obj;`
* 而不是
    * `class_name obj();`
    * 這是在宣告 function!
        * obj is function without parameter and return `class_name`


### 7.5.4 Implicit Class-Type Conversions
* 那些可以只用一個 argument 呼叫的 ctor 就恰好定義了從該 argument type 到 class type 的 conversion
    * 又叫 converting constructor
* 14 章的 operator overloading 會講怎麼把 class type 轉成其他 type

* 因為上述的關係，結合我們之前定義的 `Sales_data`，你可以這樣寫
    ```cpp
    string null_book = "9-999-99999-9"; 
    // constructs a temporary Sales_data object // with units_sold and revenue equal to 0 and bookNo equal to null_book
    item.combine(null_book);
    ```
    * combine 原本是吃一個 Sales_data object 當參數，而這裡卻是餵給他 string，這是合法的，compiler 會用 Sales_data 那個吃 string 的 ctor 把這個 string 轉成一個 ** temporary Sales_data 物件**，然後把這個物件給 combine 當作參數
        * **注意，只有 const reference 才可以 bind 暫時物件，所以這裡 combine 宣告的 const Sales_data& 必不可少**

#### Only One Class-Type Conversion Is Allowed
* 上面說明的 implicit **class type** conversion 不能連續發生兩次

    ```cpp
    // error: requires two user-defined conversions: 
    // (1) convert "9-999-99999-9"to string
    // (2) convert that (temporary) string to Sales_data
    item.combine("9-999-99999-9");
    ```
    * "9-999-99999-9" 必須先轉成 `std::string`，因為 `Sales_data` 只有吃 `std::string` 的 ctor，沒有吃 C string 的 ctor
    * **可是這樣就會把 "9-999-99999-9" 從 C str 轉成 string 再轉成 Sales_data，這種轉換不允許，會噴 error**

* 真的要做 N 次轉換，就不能有連續兩次是 implicit **class type** conversion
    ```cpp
    // ok: explicit conversion to string, implicit conversion to Sales_data 
    item.combine(string("9-999-99999-9"));     
    // ok: implicit conversion to string, explicit conversion to Sales_data
    item.combine(Sales_data("9-999-99999-9"));
    ```
    * 第一個例子，先明確轉成 `std::string` 再 implicit 轉成 `Sales_data`
    * 第二個例子，先 implicit 轉成 `std::string` 再明確轉成 `Sales_data`

* 注意如果是長這樣
    ```cpp
    class Sales_data {
        Sales_data(int i);
    }
    
    Sales_data s = 'x'; // 'x' promote to int and implicit convert to Sales_data
    ```
    * 這樣有一次不是 class-type conversion，所以合法
#### Class-Type Conversions Are Not Always Useful
* 這種隱性轉換有時候其實爛爆ㄌ，難用ㄉ要死，有時候不希望 compiler 做隱性轉換
    * 首先這種轉換得到的 object 都是暫時物件，你做隱性轉換的地方執行完之後物件就不能 access 了
    * 有的時候可以允許這樣的轉換的話，反而會降低可讀性
        * 比方說可以把 `std::cin` implicit 轉成 `Sales_data` 怎麼想都很ㄎㄧㄤ

#### Suppressing Implicit Conversions Defined by Constructors
* 用 `explicit` 宣告某個只有一個參數的 ctor，避免 compiler 使用那個 ctor 在某個 context 把那個參數的物件轉換成 class object
    ```cpp
    explicit Sales_data(std::istream&);
    ```
* 如果吃 istream 的 csor 這樣宣告，`item.combine(cin);` 這種詭異的 code 就會噴 error
* `explicit` 只適用在可用一個參數就能呼叫的 ctor，因為需要多參數來呼叫的 ctor 本來就不會拿來做隱性轉換

* 然後 `explicit` 只能宣告在 class 定義內，你 member funtion 的定義寫在 class 外面的時候不能加 `explicit`，會 syntax error
    ```cpp
    // error: explicit allowed only on a constructor declaration in a class header('explicit' outside class definition)
    explicit Sales_data::Sales_data(istream& is) {
        read(is, *this);
    }
    ```
    
#### explicit Constructors Can Be Used Only for Direct Initialization
* 宣告成 `explicit` 的 ctor 只能用 (參數) 的方式給物件，不能用 = 參數的方式
* 例如，如果沒有對吃 `std::cin` 的 ctor 用 `explicit` 的話這樣寫合法:
    ```cpp
    Sales_data data1 = cin;
    Sales_data data2(cin); // 等價上面的
    ```
    * 光用看的就覺得ㄎㄧㄤ爆
* 用了 `explicit` 後只能用第二種

#### Explicitly Using Constructors for Conversions
* `explicit` ctor 還是可以 explicitly 使用(XD
    * 例如自己寫 `class_name(parameter)`
    * 或者 `static_cast\<class_name> parameter` 都是合法的
    ```cpp
    // ok: the argument is an explicitly constructed Sales_data object 
    item.combine(Sales_data(null_book));
    // ok: static_castcan use an explicit constructor
    item.combine(static_cast<Sales_data>(cin));
    ```
    * 注意上面吃 null_book(string) 的 code Primer 也是把它改成 explicit，所以才要明確寫出 Sales_data(null_book) 這種強制轉換

#### Library Classes with explicit Constructors
* The `std::string` constructor that takes a single parameter of type const char* (§ 3.2.1, p. 84) is not explicit.
    * 所以才可以寫 `string str = "hsilu";` 這種 code
* The `std::vector` constructor that takes a size (§ 3.3.1, p. 98) is explicit.
    * 所以才不能寫 vector\<int> vec = 10; 這種鬼東西
    * 如果可以的話他會跟 vector\<int> vec(10); 等價，可是 = 的寫法整個很 confusing

### 7.5.5 Aggregate Classes
* 這根本就對應到 C struct...
* 反正就是一包 data pack 的概念
* 這邊就直接看 Primer 了，不打筆記


### 7.5.6 Literal Classes
* 又是有關 constexpr 的東西...
* 再強調一次 C\++14 在 constexpr 的部分不相容 C\++11，建議之後再看

## 7.6 static Class Members
* Classes sometimes need **members that are associated with the class,** rather than with individual objects of the class type.

#### Declaring static Members

```cpp
class Account {
public:
    void calculate() { amount += amount * interestRate; }
    static double rate() { return interestRate; }
    static void rate(double);
private:
    std::string owner;
    double amount;
    static double interestRate;
    static double initRate();
};
```
* The static members of a class **exist outside any object.**
* **Objects do not contain** data associated with **static data members**.
* 所以 Account object 只會有 amount 跟 owner 這兩個 object 而已
* There is only one interestRate object that will **be shared by all the Account objects.**
* Similarly, **static member functions are not bound to any object;**
    * 注意，之前說的，會把每個 member function 隱性的傳入 this 這件事情不會發生!
* 因為上述，static member function 不能宣告成 const，也不能在裡面使用 this
    * 不能宣告成 const 是因為，原本 member function 宣告成 const 就是在改變傳入的 this 的 low-level const，阿現在 this 根本不會傳到 static member function，所以 C++ 索性限制 static member function 不能宣告成 const
* 不能用 this 的真正效果是，所有 nonstatic member 都不能訪問! 原本 nonstatic member 本來就是(隱性的)透過 this 去存取的，現在沒有 this，根本存取不到


#### Using a Class static Member
* Access a static member directly through the scope operator:
    ```cpp
    double r;
    r = Account::rate();
    // access a static member using the scope operator

* 雖然 static member 不屬於任何一個 class object，我們還是可以透過 object(或 ptr/ref to object)去存取 static member
    ```cpp
    Account ac1;
    Account *ac2 = &ac1;
    // equivalent ways to call the static member rate function 
    r = ac1.rate();
    // through an Account object or reference
    r = ac2->rate(); // through a pointer to an Account object
    ```

* member function 也可以直接存取 static member
    ```cpp
    class Account {
    public:
        void calculate() { amount += amount * interestRate; }
    private:
        static double interestRate;
        // remaining members as before
    };
    ```
    
#### Defining static Members
* static member function 你要定義在 class 內還外都可以，不過 class 外不可以再寫一次 static，會 error
* **不過 static data member 一定要在 class 外面再定義一次!**，理由如下:
    * **Because static data members are not part of individual objects of the class type, they are not defined when we create objects of the class.**
    * 亦即 static member 不是在創造物件的時候初始化的，而且這意味著你在 class definition 裡面寫 static data member 的宣告時，**真的只有宣告**，你必須在 class 外面再定義一次 member
    * http://en.cppreference.com/w/cpp/language/static
* 所以 static member 也不是靠 ctor 初始化的


#### In-Class Initialization of static Data Members


# 跳過，覺得 7.5 後半跟 7.6 都要之後再看一次