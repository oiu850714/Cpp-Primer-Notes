
# C++ Primer Chapter 16 Templates and Generic programming
* Both OOP and generic programming deal with types *that are not known at the time the program is written.*
    * OOP deals with types that are not known until run time,
    * whereas in generic programming the **types become known during compilation.**
* When we write a generic program, **we write the code in a way that is independent of any particular type.**
    * 然後在使用這個 code 時提供型態
    * 你寫的 template，同時也代表著提供的型態必須滿足什麼需求，只要滿足就可以使用 template

* A template is a **blueprint or formula** for creating classes or functions.
    * supply the information needed to transform that blueprint into a specific class or function.
    * That transformation **happens during compilation.**
    

## 16.1 Defining a Template
* 想解決的問題: 幾乎同樣的 code 寫很多遍，只差在型別不同
* 例如 compare 兩個 value，原本我們會想要寫多種不同型態的版本:
    ```c++
    // returns 0 if the values are equal, -1 if v1 is smaller, 1 if v2 is smaller 
    int compare(const string &v1, const string &v2) {
        if (v1 < v2) return -1;
        if (v2 < v1) return 1;
        return 0;
    }
    int compare(const double &v1, const double &v2) {
        if (v1 < v2) return -1; 
        if (v2 < v1) return 1; 
        return 0;
    }
    ```
    * 長的 87% 像
    * 某種程度上就是 repeat yourself
    * 而且如果要這樣寫，我們一定得知道要比較的 type 是什麼；如果 user code 需要使用我們的 code，但是是想要比較他自己寫的 type，就不能用了

### 16.1.1 Function Templates
* A function template is a formula from which we can **generate type-specific versions of that function.**
* 上面的 compare 可以變成這樣:
    ```c++
    template <typename T>
    int compare(const T &v1, const T &v2) {
        if (v1 < v2)
            return -1;
        if (v2 < v1)
            return 1;
        return 0;
    }
    ```
    * keyword `template`，後面跟著 **template parameter list**，是一或多個 **template parameters**，用角括號包起來
        * 注意 list 不能為空
* temp para list 跟 func para list 有點像
    * func para list 定義 local variable 的型態，但沒有說如何初始化；在 run time 的時候才會由 function call 給定的 initializer 初始化這些變數
    * 而 temp para list 代表了使用在 function 或 class 內會用到的 type 或 value
        * 當我們使用 template 時，我們會提供 **templata *arguments*** -不管是直接還是間接提供-，這些 arguments 會被 bind 到 template parameters

* 我們的 `compare` function template 定義了一個 template parameter `T`，在 template 內，我們直接把 `T` 當成 type 來用
    * 至於 `T` 最後到底會代表什麼，端看 user code 怎麼使用 template，**而且會在 compile time 時決定**

#### **Instantiating** a Function Template
* 當我們*呼叫* function template 時，compiler 會用你提供的 function argument(s) **的型態** 來推斷 template arguments 是什麼
* 例如下面的 call:
    ```c++
    cout << compare(1, 0) << endl; // Tis int
    ```
    * arguments 型態是 `int`，compiler 會推斷 `compare` 定義的 `T` 是 `int`，然後把 `int` bind 到 `T` 上
* compiler 最後會在編譯時利用上面推斷出來的 `T` **實例化**出一個 function，例如如果寫下面的 code:
    ```c++
    // instantiates int compare(const int&, const int&)
    cout << compare(1, 0) << endl; // T is int
    // instantiates int compare(const vector<int>&, const vector<int>&) 
    vector<int> vec1{1, 2, 3}, vec2{4, 5, 6};
    cout << compare(vec1, vec2) << endl; // T is vector<int>
    ```
* compiler 會實例化出兩個不同版本的 `compare` functions
    * 第一個會把 `T` 替換成 `int`:
    ```c++
    int compare(const int &v1, const int &v2) {
        if (v1 < v2) return -1; 
        if (v2 < v1) return 1; 
        return 0;
    }
    ```
    * 第二個更ㄎㄧㄤ，會把 `T` 替換成 `vector<int>`，就不再打一份惹
    * 注意 `const` 是原本 template 定義時就有的，而不是包含在 compiler 推斷的 type 內；我猜之後大概會詳細講有關 `const` 要怎麼推斷之類的... 一定很噁，先留個紀錄 
* 術語上會說這些 compiler 生出來的 function 是 **an instantiation of the template**

#### Template Type Parameters
* template parameters 又有分兩種，先介紹第一種: template **type** parameters
* 上面的 `compare` 的 `T` 就屬於這種
* 我們可以拿 template type parameters 拿來放在 template 內**需要 type** 的地方，
    * 講得更精簡就是，我們可以在 template 內直接把它當作 type specifier 來用
    * 例如:
        * 宣告變數時
        * function template 宣告時的 function paramter 的型態
        * return type
    ```c++
    // ok: same type used for the return type and parameter
    template <typename T> T foo(T* p) {
        T tmp = *p; // tmp will have the type to which p points 
        // ... 
        return tmp;
    }
    ```
    * 上面的 code 三種都用到惹

* 然後你在 temp para list 要宣告 type parameter 時，你必須在前面加上 keyword `typename` 或 `class`，他們兩個本質上沒有差別，都是代表要宣告 type parameter
    ```c++
    // error: must precede U with either typename or class 
    template <typename T, U> T calc(const T&, const U&);
    ```
    * 用 `typename` 比較清楚就是，只是這是比較晚期的標準才有的，早期 code 都只有用 `class`

#### Nontype Template Parameters
* 另一種類型的 templare parameters 就是 **nontype** template parameters
* A nontype parameter **represents a value rather than a type.**
* 宣告的時候不是用 `typename/class` 宣告，而是真正的 type:
* 在 template 實例化時，nontype temp para 出現的地方，會被替換成 user 提供的 value，或者由 compiler 推斷(之後細講 compiler 怎麼推斷)
    * **既然這個 value 是在實例化時，也就是 compile time 時，就要決定的，換句話說它要是 `constexpr`**
* 我們可以再寫一個 `compare` 的 function template:
    ```c++
    template<unsigned N, unsigned M> 
    int compare(const char (&p1)[N], const char (&p2)[M]) {
        return strcmp(p1, p2);
    }
    ```
    * 注意 `N M` 是用 `unsigned` 宣告的而不是 `typename/class`
    * 這個 `compare` 是要拿來處理 string literals 時用的
    * string literal，也就是 `const char` array
    * 我們希望可以用這個 compare 來比較任何不同長度的 string literals，所以我們就用 nontype temp para `M N` 來代表長度，`N`/`M` 代表第一/二個 literal 的長度

* 當我們這樣 call:
    ```c++
    compare("hi", "mom")
    ```
* 會生出這種 function:
    ```c++
    int compare(const char (&p1)[3], const char (&p2)[4])
    ```
    * 這時 compiler 就會由 literal 的長度去推斷 `N M` 了
    * 注意有 null character

* 題外話，其實你可以用兩個不同的 type parameter 也可以做到一樣的事情，不一定要用 nontype:
    ```c++
    template <typename T, typename U> int comp(T &a1, U &a2) {
        return strcmp(a1, a2);
    }
    ```
    * 注意 `T` 跟 `U` 一樣要有 reference，推斷出來的 function 參數型別才會跟上面 nontype 版本的一樣；如果沒有 reference 則會推出 `const char*`

* 再來講什麼樣的型別可以拿來宣告 nontype temp para:
    * integral type
    * pointer
    * (lvalue) ref
* 而且要綁到 nontype temp para 的 arguments，一定要是 constant expression
    * integral type 比較好懂
    * ptr/ref 則是要綁到有 static lifetime 的物件，例如 global variable, static variable，或 function
        * ptr 可以用 `nullptr` 或 0 來實例化
    * 你不信你可以綁個 nonstatic 的試試看，compiler 推斷它不是 `constexpr` 之後就不知道怎麼實例化，就會噴你 error
    * 因為只有 static lifetime 的物件才有辦法在 compile time 時就決定要擺哪裡
	    * 包括 global 變數，function 內的 `static` 變數，`static` member

* 結論: Template arguments used for nontype template parameters **must be constant expressions.**

#### inline and constexpr Function Templates
* function template 一樣可以宣告成這兩種，格式如下:
    ```c++
    // ok: inline specifier follows the template parameter list 
    template <typename T> inline T min(const T&, const T&);
    // error: incorrect placement of the inline specifier
    inline template <typename T> T min(const T&, const T&);
    ```
    * `inline/constexpr` 宣告在角括號後面

#### **Writing Type-Independent Code**
* two important principles for writing generic code:
    * 這兩個 principle 應該是用在不會改變 function 的 parameter 的 value 用的啦
    * The function parameters in the template are references to const.
    * The tests in the body use only < comparisons.
* 用 const ref，我們的 code 可以用在不能被 copy 的物件
* 並且物件如果很大的話不會 copy
    * 上面兩點其實就跟寫一般的純讀取 function 的 principle 一樣

* 再來是 <:
    * 你可能會覺得我們的 `compare` 雖然看的懂，可是有點奇怪，為什麼不再用個 > 呢?
	    * 還是你根本沒認真看 code，沒發現他只用 `operator<`? 安捏姆湯喔
    ```c++
    // expected comparison
    if (v1 < v2) return -1;
    if (v1 > v2) return 1;
    return 0;
    ```
    * 原因很簡單，只要參數的型別有定義 < 就可以用原本的 template；如果寫成上面那樣子，參數型別還要定義 >
* 如果要讓這個 `compare` 更 general，更 portable，我們甚至不該用小於，要用 `less`(14.8.2):
    ```c++
    // version of compare that will be correct even if used on pointers; see § 14.8.2 (p. 575)
    template <typename T> int compare(const T &v1, const T &v2) {
        if (less<T>()(v1, v2)) return -1;
        if (less<T>()(v2, v1)) return 1;
        return 0;
    }
    ```
    * 這樣連 pointer 都可以用，回去看介紹 `less` 的部分，他可以拿來比較 pointer...

### Best Practice: **Template programs should try to minimize the number of requirements placed on the argument types.**

#### Template Compilation
* 當 compiler 看到 template definition 時，它不會 gen code；只有我們呼叫 template 時，compiler 才會 gen code，也就是實例化一個特定版本的 function
* The fact that code is generated only when we use a template (and not when we define it) **affects how we organize our source code and when errors are detected.**
    * 之後會看到 template 應該怎麼*宣告*和*定義*，並且該擺放在 source code 的哪個地方

* 以下是一般 function 和 class 的使用狀況:
    * 原本我們在呼叫一般的 function 時，compiler 只需要看到 function **declaration** 就可以了，definition 不需要看到
    * class 相似，我們要使用 class 時，class definition 必須要有，但是 member function(s) 的 definition 不需要看到
    * 因為這樣，我們通常會把 function declaration 跟 class definition 放在 header，而 function definition 跟 member function definition 放在 cpp 檔案(source file)內
        * inline 除外，inline 必須看到 definition，必須放在 header，詳情 15章後最被雷的地方(或者 ㄎㄧㄤ物 list 也有)

* 但是 template 不同!:
    * 你就想成，為了要實例化一個 function，compiler 當然要能看到 function template 的定義才有辦法生出一個完整的 function；
    * member function 也是一樣，例如 `vector<T>::push_back` 這種，它一定要能看到 push_back 的 template definition 的定義才能生出對應的 member function

* 所以如果你的 header 是定義 template，通常會把 template 的宣告**跟定義**都放在 header 內

#### KEY CONCEPT: TEMPLATES AND HEADERS
* 整份 template ***內*** 的 names 可以分為兩種，例如你可以想像成 class template 有一堆 member function(的名字)，還有拿來宣告物件的 type(的名字)
* 可以分成兩種:
    * 沒有依賴 template parameters
    * 有依賴 template parameters

* template provider，也就是實作 template 的人，必須要保證:
    1. 那些在 template 內被用到的，但是沒有依賴 template parameters 的 names，當 template 被 user code 使用時，必須是可見的 (visible)
        * visible 就是看得到名字的宣告就好，不需要看到這種名字的定義
    2. 還有 template 的 definition，包刮 function template 的 definition，class template 的 member function 的 definition，也要在 template 被使用時是可見的

* 而 user code，也就是使用這個 template 的 code，必須要保證:
    * 你提供給 template 的 template arguments，只要跟這些 arguments 有關的，且需要被 template 使用到的 function declarations，operators，types，在 template 實例化時，是要可見的
    * 比方說某個 template 需要用到 template parameter 的 `operator+`，那你提供的型別的 `operator+`( `operator+` 就是跟這個 parameter 有關的 function declarations) 在 template 實例化時是要可見的

* 上面的兩點只要你 well organized 你的 code，都很容易達成
    * template provide 要提供一個同時有宣告跟定義 template 的 header
    * user 則是在使用 template 時，要把丟給 template 的參數有關的型別都 include，使得 template defnition 看的到有關這個型別的 function/operators/types

#### Compilation Errors Are Mostly Reported during Instantiation
* 是一位很愛編譯時期決定的朋友呢!!
* **learn about compilation errors in the code inside the template**
* compiler 編譯 template 的 code 分很多階段:
    * The first stage is when we compile the template itself.
        * *generally can’t find many errors at this stage.*
        * something like syntax error
    * The second error-detection time is when the compiler sees a use of the template.
        * *still not much the compiler can check.*
        * typically will check that the number of the arguments is appropriate
        * also detect whether two arguments that are supposed to have the same type do so.
            * 例如你 template parameter 都是給 T，結果兩個參數型別卻不一樣
            ```c++
      template <typename T> int f(T t1, T t2); 
      f(1, 0.87); // error
      ```
        * 但也只能檢查這些了
    * **The third time when errors are detected is during instantiation.**
        * It is only then that type-related errors can be found.
        * Depending on how the compiler manages instantiation, *these errors may be reported at link time.*
            * 然後你就會看到一些 linker 噴的機掰 error...

* 我們在寫 template 時，我們要越 generic 越好，可是還是會對傳入的 type 有一些假設:
    * 例如我們的 `compare` 就假設 type 要支援 `operator<`
    * 還可以把 `compare` 的 code 一行一行來看它依賴什麼假設:
    ```c++
    if (v1 < v2) return -1; // requires < on objects of type T
    if (v2 < v1) return 1; // requires < on objects of type T
    return 0; // returns int; not dependent on T
    ```
* 還有你可以想為什麼 compiler 在編譯 template 本身時無法做太多 error checking，例如上面使用到的 <；它一定要知道 T 的型別才能知道 T 支不支援 <，所以它無法在編譯 template 時驗證 `v1 < v2` 是否合法

* 如果這樣寫:
    ```c++
    Sales_data data1, data2;
    cout << compare(data1, data2) << endl; // error: no < on Sales_data
    ```
    * compiler 就會在實例化的時候發現 `Sales_data` 沒有定義 <，噴 error

* 結論:
    * It is up to the caller to guarantee that **the arguments passed to the template support any operations that template uses**, and that those operations **behave correctly** in the context in which the template uses them.
        * behave correctly，例如你傳進去給 `sort` 的 predicate 要滿足 strict weak ordering 一樣；如果你只是把 API 接起來(亦即，傳入一個 signature 一樣的 function)卻不滿足 `sort` 要的條件，行為就會是 UB

### 16.1.2 Class Templates
* blueprint for generating classes.
* differ from function templates in that the *compiler cannot deduce the template parameter type(s) for a class template.*
* to use a class template we **must supply additional information inside angle brackets**
* 反正 compiler 就是不會推斷參數啦zzz，我們要自己在腳括號填型別

#### Defining a Class Template
* 把 `StrBlob` 擴展成 `Blob`:
    * 為什麼好死不死要用一個 pointerlike 的型態來講解...，自幹個 vector 不好嗎...
    * our template will provide shared (and checked) access to the elements it holds.
    * As with the library containers, our users will have to specify the element type when they use a `Blob`.
    ```c++
    template <typename T> class Blob { 
    public:
        typedef T value_type;
        typedef typename std::vector<T>::size_type size_type; 
        // constructors
        Blob();
        Blob(std::initializer_list<T> il); 
        // number of elements in the Blob 
        size_type size() const { return data->size(); }
        bool empty() const { return data->empty(); }
        // add and remove elements
        void push_back(const T &t) {data->push_back(t);}
        // move version; see § 13.6.3 (p. 548)
        void push_back(T &&t) { data->push_back(std::move(t)); }
        void pop_back();
        // element access T& back();
        T& operator[](size_type i); // defined in § 14.5 (p. 566)
    private:
        std::shared_ptr<std::vector<T>> data; // throws msg if data[i] isn’t valid 
        void check(size_type i, const std::string &msg) const;
    };
    ```
    * 只會吃一個 template type parameter `T`
    * 說真的裡面包了一個指向 `vector` 的 `shared_ptr` 讓人覺得很ㄎㄧㄤ...
    * 注意 `typedef` 那一行的 `typename` 是必須的，之後會講語法
    * We use the type parameter anywhere we refer to the element type that the `Blob` holds.
        * 例如 `back` 跟 `operator[]` 都是回傳 `T&`
    * When a user instantiates a `Blob`, these uses of `T` will be replaced by the specified template argument type.

* 其實長得跟 `StrBlob` 很像，只是多了 `template` 宣告，然後 `string` 的地方都替換成 `T`

#### Instantiating a Class Template
* need to supply **explicit template arguments** that are bound to the template’s parameters.
    * The compiler uses these template arguments to **instantiate a specific class** from the template.
* 要用 `Blob` 時就跟用 `vector` 一樣要提供 element type:
    ```c++
    Blob<int> ia; // empty Blob<int>
    Blob<int> ia2 = {0,1,2,3,4}; // Blob<int> with five elements
    ```
    * 上面兩個都會用 `Blob<int>` 這個 template 實例化出來的 class:
    ```c++
    template <> class Blob<int> {
    public:
        typedef int value_type;
        typedef typename std::vector<int>::size_type size_type;
        Blob();
        Blob(std::initializer_list<int> il); 
        // ... 
        int& operator[](size_type i);
    private:
        std::shared_ptr<std::vector<int>> data;
        void check(size_type i, const std::string &msg) const;
    };
    ```
    * rewrites the `Blob` template, replacing each instance of the template parameter `T` by the given template argument, which in this case is `int`.

* The compiler *generates a different class* for each element type we specify:
    ```c++
    // these definitions instantiate two distinct Blob types 
    Blob<string> names; // Blob that holds strings
    Blob<double> prices;// different element type
    ```
    
* Note: Each instantiation of a class template *constitutes an **independent** class.*
    * The type Blob\<string> has no relationship to, or any special access to, the members of any other Blob type.

#### References to a Template Type in the Scope of the Template
* In order to read template class code, it can be helpful to remember that **the name of a class template is *not* the name of a type** (§ 3.3, p. 97).
* 這裡感覺沒很難...總之就是說 `Blob` 內的 `data` 也是用 `T` 來當作提供給 `vector` 跟 `shared_ptr` 的 argument

#### Member Functions of Class Templates
* 跟一般 class 一樣，我們可以把 class templates 的 member function 定義在 class template body 內或外
* 定義在 body 內的一樣是 implicitly `inline`
    * 這邊後面講的只是碎碎念，可以不用看: 其實好像一定要變成 `inline` 才對，不然如果把 class definition 定義在 header，這樣定義在 class body 內的 function 就會有多重定義的問題

* 定義在 body 外的 member function 因為還是有 template parameter，所以定義前也要用 `template` keyword，就如同定義 function template 那樣
* 不過就跟一般 member function 一樣，定義在外面時要加 class scope，class template member function 也要:
    ```c++
    ret-type StrBlob::member_name(parm_list)
    ```
    * 上面是一般 class member function 定義在 class 外的格式 
    ```c++
    template <typename T>
    ret-type Blob<T>::member_name(parm_list)
    ```
    * 這是 class template member function；**注意 class template name 不是 type name 這個觀念**，加上 template arguments(例如，`Blob<T>`)才是，所以我們應該是要在 scope operator 前面加 `Blob<T>` 而不只是 `Blob`

#### The check and Element Access Members
* 總之就是把 `StrBlob` 的改成 template...
    ```c++
    template <typename T>
    void Blob<T>::check(size_type i, const std::string &msg) const {
        if (i >= data->size()) throw std::out_of_range(msg);
    }
    ```
    * 一樣，`operator[]` 跟 `back` 會使用 `check` 這個 utility
* `operator[]` 跟 `back` 把 return type 改成 template paramter:
    ```c++
    template <typename T>
    T& Blob<T>::back() {
        check(0, "back on empty Blob");
        return data->back();
    }
    template <typename T>
    T& Blob<T>::operator[](size_type i) {
        // if i is too big, check will throw, preventing access to a non existent element 
        check(i, "subscript out of range");
        return (*data)[i];
    }
    ```
* `pop_back` 就更像惹，因為他連 return 都沒有:
    ```c++
    template <typename T> void Blob<T>::pop_back() {
        check(0, "pop_back on empty Blob"); 
        data->pop_back();
    }
    ```
* 除了這些，`operator[]` 跟 `back` 還有 `const` 版本，留在習題定義；還有 `front`

#### Blob Constructors
* 跟一般 member function 一樣，如果定義在 body 外也要加一些 template 的格式:
    ```c++
    template <typename T> Blob<T>::Blob(): 
        data(std::make_shared<std::vector<T>>()) { }
    ```
    * 注意是寫成 `Blob<T>::Blob()`
	    * 原因是因為，`Blob<T>` 是 class，`Blob<T>::Blob` 是 member function，他就只是個 member function，不需要加 template parameter
	    * member function 也同理，`T` 只有寫一次
* 吃 `initializer_list` 的 ctor:
    ```c++
    template <typename T> Blob<T>::Blob(std::initializer_list<T> il):
        data(std::make_shared<std::vector<T>>(il)) { }
    ```
    
 #### Instantiation of Class-Template Member Functions
 * By default, amember function of a class template is instantiated only if the program uses that member function.
     * compiler 很懶(X)，你用到了某個 class-template 的 member function 之後它才會實例化這個 member function
     * 其實就跟你一般的 member function 一樣，只要沒用到，就算沒有 definition 也沒關係

* 例如下面的 code:
    ```c++
    // instantiates Blob<int> class definition, and the initializer_list<int> constructor 
    Blob<int> squares = {0,1,2,3,4,5,6,7,8,9}; 
    // instantiates Blob<int>::size() const 
    for (size_t i = 0; i != squares.size(); ++i)
        squares[i] = i*i; // instantiates Blob<int>::operator[](size_t)
    ```
    * 看完這段 code 之後，compiler 只會實例化 `Blob<int>`, 還有它的三個 member functions: `size`, `operator[]`, `ctor(il)`

* 這種「沒用到就不實例化的機制」其實有好處，例如你提供給 template 的 type 就算沒有符合 class template 的某些 member functions 的要求，只要你沒有使用那些 member functions，你還是可以用這個 type 實例化 class

#### Simplifying Use of a Template Class Name inside Class Code
* 之前說要在用 clas template 時一定要提供 type argument(s) 其實有例外: 當你已經在 class template scope 內時，就可以不用提供
* 用 `BlobPtr` 舉例:
    ```c++
    // BlobPtr throws an exception on attempts to access a nonexistent element 
    template <typename T> class BlobPtr {
    public:
        BlobPtr(): curr(0) { }
        BlobPtr(Blob<T> &a, size_t sz = 0):
                wptr(a.data), curr(sz) { }
        T& operator*() const { 
            auto p = check(curr, "dereference past end");
            return (*p)[curr]; // (*p)is the vector to which this object points
        }
        // increment and decrement
        BlobPtr& operator++();
        BlobPtr& operator--();
        // prefix operators
        private: // check returns a shared_ptr to the vector if the check succeeds
        std::shared_ptr<std::vector<T>> 
            check(std::size_t, const std::string&) const;
        // store a weak_ptr, which means the underlying vector might be destroyed
        std::weak_ptr<std::vector<T>> wptr;
        std::size_t curr; // current position within the array
    };
    ```
    * 仔細看 `operator++/--` 的 return type 是 `BlobPtr`
    * 當你在 class template scope 內時，**compiler 會把 reference to class template itself 當成我們已經提供 template arguments 的樣子**:
    ```c++
    BlobPtr<T>& operator++();
    BlobPtr<T>& operator--();
    ```
    * 不知道實務上偏好哪個...，你要把 `<T>` 寫出來也合法就是

#### Using a Class Template Name outside the Class Template Body
* We must remember that we are not in the scope of the class *until the class name is seen*
    ```c++
    // postfix: increment/decrement the object but return the unchanged value 
    template <typename T>
    BlobPtr<T> BlobPtr<T>::operator++(int) {
    //^ not in scope       ^ in scope
        // no check needed here; the call to prefix increment will do the check 
        BlobPtr ret = *this; // save the current value 
        ++*this; // advance one element; prefix ++ checks the increment 
        return ret; // return the saved state
    }
    ```
    * 寫 return type 時還沒在 scope 內，所以 `<T>` 要寫出來
    * 在定義 `ret` 時已經在 scope 內，所以宣告時的 type specifier，也就是 `BlobPtr`，不用加 `<T>`，如同下面這樣:
        ```c++
        BlobPtr<T> ret = *this;
        ```
* 結論: 在 scope 內，我們直接使用 template 名字時可以不用加 template arguments

#### Class Templates and Friends
* 一個 class 有 friend，class 跟 friend 要不要是 template 都可以
* 一個 class template 如果有 nontemplate friends，這個 `friend` 都跟這個 class template 的所有 instantiations 都是 friend
* 一個 class template 如果有 template friends，**class 可以決定哪些 `friend` 的 instantiations 是 friend**

#### One-to-One Friendship
* 在 class 跟 friend 都是 templates 的情況下，最常見的 friendship 就是，讓 class template 用某 type `T` 實例化的 instantiation，跟 friend 也用 `T` 實例化的 instantiation，成為 friend
	* 例如我們希望 `Blob` 可以跟 `BlobPtr` 還有 `operator==` 成為好友，可是這樣講太攏統，因為他們三個都是 template；**我們可以具體指定說，他們三個各自用相同 type 實例化出來的 instantiations 互為 friend**

* 不過如果要在 class template 內說某 template 的某個 instantiation 是我的 friend，我們必須在 class template 的定義之前先**宣告**這個 friend template
    * 畢竟你要說某 instantiation 是 friend，你就要給定 template argument，要給定 template argument 就至少要知道被使用的 friend 是 template，所以就要看到 template 宣告

* A template declaration includes the template’s template parameter list:
    ```c++
    // forward declarations needed for friend declarations in Blob 
    template <typename> class BlobPtr;
    template <typename> class Blob; // needed for parameters in operator== 
    template <typename T>
        bool operator==(const Blob<T>&, const Blob<T>&);

    template <typename T> class Blob {
        // each instantiation of Blob grants access to the version of 
        // BlobPtr and the equality operator instantiated with the same type
        friend class BlobPtr<T>;
        friend bool operator==<T> (const Blob<T>&, const Blob<T>&);
        // other members as in § 12.1.1 (p. 456)
    };
    ```
* We start by declaring that `Blob`, `BlobPtr`, and `operator==` are templates. **These declarations are needed for the parameter declaration in the `operator==` function and the friend declarations in `Blob`.**
* 一個一個看每個 template 的**宣告**是為了用在哪裡:
    * 首先如果只是宣告 template，可以在 parameter list 那邊(也就是腳括號裡面)只打 keyword `template` 而不用給 parameter name
    * `BlobPtr` 的宣告是為了用在 `Blob` template 內的 friend declaration
    * `Blob` 的宣告是為了用在 `operator==` template 內的 function parameter 內
    * `operator==` 的宣告是為了用在 `Blob` template 內的 friend declaration
* 再來看 `Blob` 內的 friend declaration 到底在說明哪個 instantiation(s) 跟哪個 instantiation(s) 是 friend:
    * 在 `Blob` 內，不管是 `BlobPtr` 還是 `operator==` 都是用了 `Blob` 的 template parameter，也就是 `<T>`，來宣告，這意味著只有用相同的 type 來實例化的 `BlobPtr` 跟 `operator==` 才會是 `Blob` 的 friend
    * 看例子啦zzz:
    ```c++
    Blob<char> ca; // BlobPtr<char> and operator==<char> are friends 
    Blob<int> ia; // BlobPtr<int> and operator==<int> are friends
    ```
    * `BlobPtr<int>` 還有 `operator==<int>` 跟 `Blob<int>` 是朋友，可是跟 `Blob<char>` 就不是

#### General and Specific Template Friendship
* 上面 demo 讓同 type 的 instantiations 互為 friend，這裡來看更 general 的情況
* 亦即一個 class 可以讓某 template 的所有 instantiations 都成為 friend，或者只讓部分成為 friend
    ```c++
    // forward declaration necessary to be friend a specific instantiation of a template 
    template <typename T> class Pal;
    class C { // C is an ordinary, nontemplate class 
        friend class Pal<C>; // Pal instantiated with class C is a friend to C 
        // all instances of Pal2 are friends to C; 
        // no forward declaration required when we befriend all instantiations
        template <typename T> friend class Pal2;
    };
    template <typename T> class C2 { // C2 is itself a class template 
        // each instantiation of C2 has the same instance of Pal as a friend 
        friend class Pal<T>; 
        // a template declaration for Pal must be in scope 
        // all instances of Pal2 are friends of each instance of C2, prior declaration "not"(Primer 好像寫錯?? 自己測 code 是不需要先宣告 Pal2) needed 
        template <typename X> friend class Pal2;
        // Pal3 is a nontemplate class that is a friend of every instance of C2 
        friend class Pal3; // prior declaration for Pal3 not needed
    };
    ```
* 一樣慢慢來看:
    * `Pal` 要先宣告的原因，是因為 `class C` 要用它來宣告特定的 instantiation(也就是用 `C` 實例化的 `Pal`)成為 friend
        * 注意 C 是一般的 class
    * 如果你希望某個 template 的所有 instantiations 都跟我是 friend，你就不用先宣告它是 template 了
        * 例如在 `class C` 內宣告成 `friend` 的 `Pal2`
        * 還有在 class template `C2` 內宣告成 `friend` 的 `Pal2`
    * 阿在 `C2` 內的 `Pal3` 就只是一般的 class 而已，要成為 friend 也不用先宣告

* 還有注意一點很重要，如果希望在 class template 內，某 template 的所有 instantiations 都跟我是 friend，**在 class template body 內 friend declaration 時給定的 template parameter(s) 要跟 class template 的 template parameter(s) 的名字不一樣!**
    * 例如上面在 class template `C2` 內宣告的 `Pal2`，它用的 template parameters 是 `X`，跟 `C2` 用的 `T` 不一樣
    * 還有在 `C` 裡面宣告的 `Pal2` 用的是 `T`，可是 `C` 只是一般 class 所以也沒衝突
    * 再來 `C` 內宣告的 `Pal<C>` 只是某個特定 instantiation，不是 template 宣告，所以沒有「讓所有 instantiations 都變成 friend」的需求

* 總結，你的 template friendship 可以一對一(同 type 對同 type)，一對全部(同 type 對所有 types)，還有一對特定(自己指定，例如上面 `C` 裡面的 `Pal<C>`)
    * 最常見的還是一對一跟一對全部

#### Befriending the Template’s Own Type Parameter
* C++11 可以直接把 type parameter 當成 friend，猛...
    ```c++
    template <typename Type> class Bar {
        friend Type; // grants access to the type used to instantiate Bar
        // ...
    };
    ```
    * Here we say that whatever type is used to instantiate Bar is a friend. 
    * Thus, for some type named Foo, Foo would be a friend of Bar\<Foo>, Sales_data afriend of Bar<Sales_data>, and so on.

#### Template Type Aliases
* 把某個 template instantiation alias 成一個 type:
    ```c++
    typedef Blob<string> StrBlob;
    ```
* 你不能 `typedef` 一個 template name
* 可是在 C++11 你可以這樣寫:
    ```c++
    template<typename T> using twin = pair<T, T>; 
    twin<string> authors; // authors is a pair<string, string>
    ```
    * 概念上就是，你 alias 出來的 name，在上面的例子就是 `twin`，他還是個 template，不是型別
* 你 `using` = 右邊的東西如果有用到 template 的話，你還可以固定他的某些 type parameter(s):
    ```c++
    template <typename T> using partNo = pair<T, unsigned>;
    partNo<string> books; // books is a pair<string, unsigned> 
    partNo<Vehicle> cars; // cars is a  pair<Vehicle, unsigned>
    partNo<Student> kids; // kids is a pair<Student, unsigned>
    ```
* 個人覺得沒有很好用...

#### `static` Members of Class Templates
* class template 也可以定義 `static` members:
    ```c++
    template <typename T> class Foo {
    public:
        static std::size_t count() { return ctr; } 
        // other interface members
    private:
        static std::size_t ctr; 
        // other implementation members
    };
    ```
* Each instantiation of `Foo` **has its own instance of the static members.**
* That is, for any given type X, there is one Foo\<X>::ctr and one Foo\<X>::count member.
    ```c++
    // instantiates static members Foo<string>::ctr and Foo<string>::count 
    Foo<string> fs; 
    // all three objects share the same Foo<int>::ctr and Foo<int>::count members
    Foo<int> fi, fi2, fi3;
    ```
* class template 的 `static` members 的 definition 一樣只能有一個
    * 但是 template 的每個 instantiation 都有一個 `static` member，所以定義這個 member 時前面還是要給定 class template (的 instantiation)的 scope:
    ```c++
    template <typename T> 
    size_t Foo<T>::ctr = 0; // define and initialize ctr
    ```
    * 這樣特定 type T 的 `Foo` 就有特定的 `Foo<T>::ctr`

* 跟一般 class 一樣，你可以用 instantiation(i.e. 就是個 class type) 或者 instantiation 的 instances 來 access static member(s)；記得不是用 template name 而是 instantiation 來 access(也就是要給定 parameter type):
    ```c++
    Foo<int> fi; // instantiates Foo<int> class 
                 // the staticdata member ctr
    auto ct = Foo<int>::count(); // instantiates Foo<int>::count 
    ct = fi.count(); // uses Foo<int>::count
    ct = Foo::count(); // error: which template instantiation?
    ```
    * 上面寫 `Foo::count` 是錯的

* class template `static` member 跟其他 member function(s) 一樣，要被 user code 用到才會被 instantiated

### 16.1.3 Template Parameters
* 習題都叫你用它寫了一個簡單 `Vec` 了才要詳細講...
* temp para 的名字跟 func para 一樣，它的名字沒有特殊意義，你想叫什麼都行:
    ```c++
    template <typename Foo> Foo calc(const Foo& a, const Foo& b) {
        Foo tmp = a; // tmp has the same type as the parameters and return type 
        // ... 
        return tmp; // return type and parameters have the same type
    }
    ```
#### Template Parameters and Scope
* follow normal scoping rules
* 當 temp para 宣告之後，到 template 宣告或定義結束為止都可以使用這個 temp para
* 跟其他 name 一樣，會把 outer scope 定義的名字給 hide!
* Unlike most other contexts, however, a name used as a template parameter may not be reused within the template:
    * 在其他的 syntax，如果你 outer scope 定義一個東西叫做 `A`，然後在 function/class 內重定義 `A` 是 OK 的，總之就會把外面定義的 `A` 給 hide；但在 template 內這樣重定義會直接噴 error:
    ```c++
    typedef double A;
    template <typename A, typename B> void f(A a, B b) {
        A tmp = a;// tmp has same type as the template parameter A, not double
        double B; // error: redeclares template parameter B
    }
    ```
    * 你的 temp para `A` 隱藏了外面的 `typedef` 使用的 `A`，所以 `temp` 的型態跟你實例化時給定的型態一樣，而不是 `double`
    * B 直接被重定義成 `double`，所以噴 error

* 既然不能重定義 temp para，當然也不能這樣寫:
    ```c++
    // error: illegal reuse of template parameter name V 
    template <typename V, typename V> // ...
    ```
    
#### Template Declarations
* must include the template parameters:
    ```c++
    // declares but does not define compare and Blob 
    template <typename T> int compare(const T&, const T&);
    template <typename T> class Blob;
    ```
* 跟 function dec/def 一樣，temp dec 內寫的 temp para 的名字不一定要跟 def 寫的一樣:
    ```c++
    // all three uses of calc refer to the same function template 
    template <typename T> T calc(const T&, const T&); // declaration 
    template <typename U> U calc(const U&, const U&); // declaration 
    // definition of the template 
    template <typename Type>
    Type calc(const Type& a, const Type& b) { /* .. . */}
    ```
    * 這三個都代表同一個 template
* Of course, every declaration and the definition of a given template must have the same number and kind (i.e., type or nontype) of parameters.
    * 宣告跟定義的重點是，要維持同數量跟同型別的 temp para
    * 這裡的同型別是指說 template parameter 是用 `typename` 宣告的，或者是 `nontype` parameter

#### Best Practice: 16.3 會講原因，不過最好把一個 file 會用到的 template 全部都先宣告在一開始

#### Using Class Members That Are Types
* 還記得我們會用 `::` 來存取 class 內的 type 或 static member
* 這時候 compiler 因為知道 class 的定義，它有辦法知道 `class_name::obj` 的 `obj` 是型別還是 static 物件
* 可是這件事情在 template 上就有困難:
    * 如果你是對 type parameter T 使用 `T::mem` 這樣 compiler 只能在 instantiate 時才知道 T 的 `mem` 是 type 還是 static member
    * 或者更甚，用 `T` 來構成型別的型別也會碰到一樣的狀況，例如 `vector<T>`
* 但 compiler 為了要處理 template，它一定要知道這時 `::` 右邊的東西是什麼，例如看到下面這種 code:
    ```c++
    T::size_type * p;
    ```
    * compiler 一定要知道你到底是在宣告一個變數 `p`，還是寫一個 expression
	    * 也就是說，compiler 一定要知道你到底是在用 `size_type` 宣告 `p`，還是在用 `size_type` 跟 `p` 相乘
* 而在這情況下，**compiler 直接假設它不是型別，是 static member**
* 因為這樣，如果我們想要使用的 temp para 的 member 是一個型別而不是物件，我們必須明確指定它是型別，如下:
    ```c++
    template <typename T> typename T::value_type top(const T& c) {
        if (!c.empty())
            return c.back();
        else
            return typename T::value_type();
    }
    ```
    * 這 code 吃一個 container type `c`，型別為 `T`，並且回傳 container 的最後一個 element；如果 `c` 是空的則回傳由 element 型別預設初始化的物件
    * 仔細看，在我們想要使用 `T::value_type` 的地方，也就是 function `top` 的 return type，跟 function 內的第二個 `return` statement，**我們都額外加了 `typename`，告訴 compiler `value_type` 是一個型別**
* 如果你要把 `T::value_type` 當成 type 使用卻沒有在前面加 `typename`，compiler 就會噴:
    * `missing 'typename' prior to dependent type name 'T::value_type'T::value_type`
    * 這也是為什麼在很前面的地方會看到 template 內用 T 的 member 宣告 template 自定義的型別時需要額外加 `typename` 的原因:
    ```c++
    typedef typename std::vector<T>::size_type size_type;
    ```
    * 不只直接使用 T 時需要加，包含了使用 T 的 instantiation(在上面的例子是 `vector<T>`)，總之如果要用那些代表型別的  member 的話也要加 `typename`

#### Default Template Arguments
* Under the new standard, we can supply default arguments for both function and class templates.
* 早期的版本只有 class template 可以放 default template arguments，function template 不行
* 重寫 `compare` 來舉例:
    ```c++
    // compare has a default template argument, less<T> 
    // and a default function argument, F() 
    template <typename T, typename F = less<T>>
    int compare(const T &v1, const T &v2, F f = F()) {
        if (f(v1, v2)) return -1;
        if (f(v2, v1)) return 1;
        return 0;
    }
    ```
    * 這裡 `F` 代表的是 callable object 的型別，預設是 `less<T>`，注意 `less<T>` 也是個型別
    * `f` 則代表一個函數參數，準備綁定一個 callable object
        * 注意 `F f = F()` 是呼叫 `F` 的copy ctor；如果 user code 沒有提供 F 的話就會是 `less<T>` 的 copy ctor

* 所以這個 template 要實例化時，要馬可以不提供 `F` 的 template argument，要馬不提供 `f` 的 function argument，會有四種情況...隨便寫個幾種:
    ```c++
    bool i = compare(0, 42); // uses less; i is -1 
    // result depends on the isbns in item1 and item2
    Sales_data item1(cin), item2(cin);
    bool j = compare(item1, item2, compareIsbn);
    ```
    * 第一個 call 不管 template 還是 function parameter 都用 default argument 初始化
    * 第二個則是丟了一個 function name 進去:
        * 這個 function name 有些要求，以前差不多也提過；回傳值要能轉成 `bool`，吃的兩個參數要跟 `Sales_data` 相容
        * 這時 F 就會**被推斷**成 `compareIsbn` 的型別
            * 可是 Primer 居然沒說推斷的型別會不會 decay 成指標... 看起來應該是會，因為 `F` 在這裡除了是 template parameter，也是 function parameter 的型別，而 function parameter 的型別不能是函數型別

* template default arguments 跟 function 還有一個共通點，只能從最右邊的 parameter 開始提供 default arguments；總之就是某 template parameter 要提供 default argument 的話，它右邊的 parameters 都要有 default arguments

#### Template Default Arguments and Class Templates
* 之前有說過 user code 要使用 class template，一定要在 template name 後面加上角括號；如果搭配上面的 default arguments 會怎樣呢? 又如果 template 所有 parameters 都有 default arguments 的話呢?
    * 答: **不管怎樣都要有角括號**，就算可以不用提供 arguments 也一樣，**這時會寫一個空的角括號: `<>`**
    ```c++
    template <class T = int> class Numbers { 
        // by default T is int
    public:
        Numbers(T v = 0): val(v) { }
        // various operations on numbers
    private: T val;
    };
    Numbers<long double> lots_of_precision;
    Numbers<> average_precision; // empty <> says we want the default type
    ```
    * 總之就是要有角括號...

#### 16.1.4 Member Templates
* class(template) 可以定義一個 member，它本身就是 template，叫做 **member templates**
    * 噁...
    * **這時你可以回去看看 C++ 標準內的 container 的 API，尤其是那些吃一對 iterator range 的 API，他們其實是 template!! 目的是在 instantiates member functions，他們會吃不同的 iterator 型別
* **不能是 `virtual`**
	* 至於為什麼，去 google... 噁

#### Member Templates of Ordinary (Nontemplate) Classes
* 來看一般 class 怎麼定義這種東西
* we'll define a class that is *similar to the default deleter* type used by unique_ptr
    * `operator()(ptr)`
    * print message when the deleter is executed
* Because we want to use our deleter with any type, we’ll make the call operator a template:
    ```c++
    // function-object class that calls delete on a given pointer
    class DebugDelete { 
    public:
        DebugDelete(std::ostream &s = std::cerr): os(s) { }
        // as with any function template, the type of T is deduced by the compiler 
        template <typename T> void operator()(T *p) const
            { os << "deleting unique_ptr" << std::endl; delete p; }
    private:
        std::ostream &os;
    };
    ```
    * 定義 member template 一樣要用 `template` 宣告，給定 type parameter, etc

* We can use this class as a replacement for delete:
    ```c++
    double* p = new double;
    DebugDelete d; // an object that can act like a delete eexpression
    d(p); // calls DebugDelete::operator()(double*), which deletes p
    int* ip = new int;
    // calls operator()(int*) on a temporary DebugDelete object
    DebugDelete()(ip);
    ```
* 基本上它跟 `delete` 有 87% 像，你可以把它拿來替換 `unique_ptr` 的預設 deleter，要這樣宣告:
    ```c++
    // destroying the the object to which p points
    // instantiates DebugDelete::operator()<int>(int *)
    unique_ptr<int, DebugDelete> p(new int, DebugDelete());
    // destroying the the object to which sp points
    // instantiates DebugDelete::operator()<string>(string*)
    unique_ptr<string, DebugDelete> sp(new string, DebugDelete());
    ```
    * 在 `unique_ptr` 的角括號提供型態，在 ctor 提供(暫時)物件
    * `unique_ptr` 準備呼叫 deleter 時就會把指向的物件的型態塞給 `DebugDelete` 的 `operator()`，這樣對應的 instantiation 就會被實例化
        * 或者應該說，`unique_ptr<T>` 被實例化時，因為 dtor 就會呼叫 `DebugDelete::operator()<T>`，所以實例化 dtor 時，對應的 `operator()<T>` 就會被實例化
    ```c++
    // sample instantiations for member templates of DebugDelete 
    void DebugDelete::operator()(int *p) const { delete p; }
    void DebugDelete::operator()(string *p) const { delete p; }
    ```
    * 這邊 Primer 好像少打 `os << "deleting unique_ptr" << std::endl;`

#### Member Templates of Class Templates
* 換介紹 class template 內怎麼定義 member templates...
* 記得 member template 的 type parameter 不要跟 class template 的有 name collision
* 其實如果你有 member function 需要吃各種不同 container 的 iterator 來當作一個 range 的話，就可以寫成 template
* 例如我們來為 `Blob` 寫一個:
    ```c++
    template <typename T> class Blob { 
        template <typename It> Blob(It b, It e);
        // ...
    };
    ```
    * template 中的 template...
* 那怎麼把 member templates 定義在 body 外?
    * We must provide the template parameter list for the class template and for the function template.
    * The parameter list for the class template comes first, followed by the member’s own template parameter list:
    ```c++
    template <typename T> // type parameter for the class
    template <typename It> // type parameter for the constructor 
    Blob<T>::Blob(It b, It e):
        data(std::make_shared<std::vector<T>>(b, e)) { }
    ```
    * ctor 直接是 template，ㄎㄧㄤ到翻天ㄛㄛㄛㄛ
    * 這時候也會發現 compiler 會自動推斷 function 的 template parameter 是好事，不然你大概要寫什麼連續兩對角括號之類的...

#### Instantiation and Member Templates
* To instantiate a member template of a class template, we must **supply arguments for the template parameters for *both* the class and the function templates.**
	* 注意 supply 不代表 code 一定要打出來
* 不過這些 type 都會在"還不錯"的時機就決定好了，我們不用一直打出來
    * template argument for class 已經在宣告物件的時候給定，之後要用這個物件時 compiler 就已經知道 temp argument 是哪種了
    * 而 member templates 則是在你給定 function arguments 時由 compiler 自行推斷
    * 綜合兩點，其實直接用起來沒有那麼恐怖LOL:
    ```c++
    int ia[] = {0,1,2,3,4,5,6,7,8,9};
    vector<long> vi = {0,1,2,3,4,5,6,7,8,9};
    list<const char*> w = {"now", "is", "the", "time"}; 
    // instantiates the Blob<int> class 
    // and the Blob<int> constructor that has two int* parameters
    Blob<int> a1(begin(ia), end(ia)); 
    // instantiates the Blob<int> constructor that has
    // two vector<long>::iterator parameters
    Blob<int> a2(vi.begin(), vi.end()); 
    // instantiates the Blob<string> class and the Blob<string> 
    // constructor that has two list<const char*>::iterator parameters
    Blob<string> a3(w.begin(), w.end());
    ```
    * ctor 都 template 化，簡直...
    * 注意你在使用 class instantiation(s)，以及傳遞不同的參數給 ctor 時，compiler 都會生出不同的 class 跟 member，超猛ㄛ
    * 上面的 code 生了 `Blob<int>` 以及他的兩個 ctors，還有 `Blob<string>` 以及他的一個 ctor

### 16.1.5 Controlling Instantiations
* 先講想解決什麼問題:
    * 當你直接在某個 TU 使用 template 時，compiler 就會在那個 TU 生出實例；如果你在不同的但之後要 link 起來的 TUs 都使用到這個 template，而且提供一樣的 template argument(s)，這樣這些不同 TU 的 object files 都會各自有一份實例(但 link 不會噴 error 就是，我還不知道 compiler 怎處理的... 既然實例都是他生的，他自己會處理吧)，這樣 code 會很肥
* 所以 c++11 提供一個方法讓這些 TU 都只共用一份實例，叫做 **explicit instantiation**:
    ```c++
    extern template declaration; // instantiation declaration 
    template declaration;// instantiation definition
    ```
    * 其中上面的 *declaration* 是 function/class declaration，並且把 template arguments 都填上去:
    * 看例子...:
    ```c++
    // instantion declaration and definition 
    extern template class Blob<string>; // declaration
    template int compare(const int&, const int&); // definition
    ```
    
* 當 compiler 看到你用 `extern` 宣告，也就是寫一個 instantiation declaration 時，**compiler 不會在這個 TU 生出一個 declaration 對應的實例**；這種宣告方式是跟 compiler **承諾**說其他 TU 會有這個實例，compiler 不需要生一個
* 而當你是直接寫一個 instantiation definition 時，compiler 就會在當前 TU 直接產生出對應的實例；**這個 definition 在整個 program 內只能有一個**，declaration 則可以有很多個
* 而因為 compiler 會在我們使用 template 時就自動生出對應的實例，所以我們應該要在使用 template 的某個實例之前就宣告上面的 `extern` declaration，不然 compiler 還是會在當前 TU 生一個實例:
    ```c++
    // Application.cc 
    // these template types must be instantiated elsewhere in the program 
    extern template class Blob<string>;
    extern template int compare(const int&, const int&);
    Blob<string> sa1, sa2; // instantiation will appear elsewhere
    // Blob<int> and its initializer_list constructor instantiated in this file 
    Blob<int> a1 = {0,1,2,3,4,5,6,7,8,9}; 
    Blob<int> a2(a1); // copy constructor instantiated in this file
    int i = compare(a1[0], a2[0]); // instantiation will appear elsewhere
    ```
    * `Application.o` 會包含有 `Blob<int>` 的 class definition，以及 `Blob<int>` 吃 `initializer_list` 的 ctor 還有 copy ctor 的 definition
	* 而 `compare<int>` 跟 `Blob<string>` 的實例則會在別的 TU 內產生:
    ```c++
    // templateBuild.cc
    // instantiation file must provide a (nonextern) definition for every 
    // type and function that other files declare as extern 
    template int compare(const int&, const int&);
    template class Blob<string>; // instantiates all members of the class template
    ```
    * 上面兩個 explicit definition 就會讓 compiler 在 `templateBuild.o` 產生出對應的實例
    * 如果要 build 上面的程式，這兩個 object files 必須 link


#### Instantiation Definitions Instantiate All Members
* 注意我們寫一個 class template 的 explicit definition 時，compiler 根本還沒辦法知道你會用到哪些 member functions，所以 compiler 索性把所有 member 都給實例化ㄏㄏ，不管你之後有沒有用到
* 這隱含了一件事，當你在寫 explicit definition 並且提供 type 時，**你提供的 type 必須確保可以用在所有的 class member function 上，因為他們全部都會被實例化**

### 16.1.6 Efficiency and Flexibility
* 先說，這裡的東西大概看得懂，可是沒感覺
* 如果你的 template 只能在「很有彈性的 API」跟「有效能」選擇，你要好好考慮
* STL 的 `shared_ptr` 跟 `unique_ptr` 他們處理 user defined deleter 的方式就是個好例子
	* `shared_ptr` 可以動態更改成不同的型別的 deleter，只要呼叫 `shared_ptr::reset` 就行了；*deleter 不屬於 `shared_ptr` 的型別的一部份*
	* 相反的，`unique_ptr` 的 deleter 則被定義成 `unique_ptr` 的 template type parameter 之一，*deleter 屬於 `unique_ptr` 的型別的一部份*
* 首先雖然我們不知道 lib 怎麼實作 `shared_ptr`，但我們可以推論說 `shared_ptr` **一定是間接的使用 deleter**：**deleter 不能是 `shared_ptr` 的 direct member**， `shared_ptr` 只能用一個指標，或者像是 `std::function` 那樣的東西去存取它，使用 deleter 時就要透過這個指標呼叫。
    * 為什麼不能把 deleter 宣告成 `shared_ptr` 的 member 呢?
        * 因為我們要到 run-time 才會知道 deleter 的型別，而且在 run-time 時還可透過 reset 傳入不同型別的物件當作新的 deleter，這件事情如果把 deleter 宣告成 direct member 是做不到的，因為 member 的型別根本不能改變
        * **所以 `shared_ptr` 要使用 deleter 會多一層 indirect call**
* 再來是 `unique_ptr`，因為 `deleter` 是 template argument 的一員，換句話說在 compile time 時就能知道 deleter 的型別，所以 `unique_ptr` 就能把 deleter 當作 direct member
	* 這樣就不會有 `shared_ptr` 那一層多的 call 了

* 總結:一個在 compile time 決定要執行什麼 deleter，省掉 run-time overhead；一個在動態時期才決定，可是可以讓你很彈性的換一個(不同型別的) deleter


##### 操你媽的 16.1 就打了快一千行
## 16.2 Template Argument Deduction
* 之前說過你使用 function template 時 compiler 會幫你推斷型別，他到底怎麼推斷然後推出什麼型別是這章要講的重點
* 然後這個過程叫做 **Template Argument Deduction**

### 16.2.1 Conversions and Template Type Parameters
* 先記住大觀念:
    * 在 function template 的情況下，**compiler 幾乎不太愛把傳入給 function template 的 function argument 做轉型**
    * 取而代之的是直接生一個 exact match 的 function instantiation
* 其他 compiler 會做轉型的情況如下:
    * 你傳進去的 function argument 的 top level `const` 還是會無視
    * 如果 function parameter 是 ref/ptr，則傳入 non`const` 版本的 argument 會被轉型成 `const` 版本
    * array/function 如果不是用 ref 來 bind，則還是會被轉成 pointer

* 其他的轉型，如 arithmetic conversions (§ 4.11.1, p. 159), derived to-base (§ 15.2.2, p. 597), and user-defined conversions (§ 7.5.4, p. 294, and § 14.9, p. 579)，都不會做，都是直接生對應的 instantiations

* 看例子:
* `fobj` 跟 `fref`，參數吃 reference 跟 value
    ```c++
    template <typename T> T fobj(T, T); // arguments are copied 
    template <typename T> T fref(const T&, const T&); // references 
    string s1("a value"); 
    const string s2("another value"); 
    fobj(s1, s2); // calls fobj(string, string); const is ignored 
    fref(s1, s2); // calls fref(const string&, const string&) 
                  // uses premissible conversion to const on s1
    int a[10], b[42];
    fobj(a, b); // calls f(int*, int*)
    fref(a, b); // error: array types don’t match
    ```
    * 仔細看傳進 function 的物件的型別
    * `fobj(s1, s2)`, s1 ,s2 一個是 `std::string` 一個是 `const std::string`，不過已經說過，當你 function 是 call by value 時，top level `const` 會無視，所以 `fobj` 都是呼叫 `fobj(string, string)`; 而 `fref` 則是使用到 plain reference 轉成 `const` reference 的轉換(對 `s1` 做轉換)，所以一樣 legal;
    * 而傳進 `a` 跟 `b` 時就不一樣了，首先他們型別不同(array size 不同)，不過傳給 `fobj` 時都會轉成 `int*` 所以沒差；可是傳給 `fref` 時就不會轉型，這樣 `fref` 的第一個跟第二個 parameter 推論出來的型別就會有衝突(deduced conflicting types)，所以噴 error

* 小結論: `const` conversions and array or function to pointer are the only automatic conversions for arguments to parameters with template types.

#### Function Parameters That Use the Same Template Parameter Type
* 這裡其實就是在講上面的 deduced conflicting types error
* 如果有多個 function parameter 用了同一個 template type parameter 當作型態宣告，那你使用 function template 時傳的對應的 arguments 的型態一定要一樣，不然就會發生這種 error
* 例如我們之前寫的 `compare`:
    ```c++
    long lng; 
    compare(lng, 1024); // error: cannot instantiate compare(long, int)
    ```
    * compiler 推斷第一個 type 是 `long`，第二個是 `int`，不 match，噴 error
* 解決上面的方法之一是使用兩個 type parameters:
    ```c++
    // argument types can differ but must be compatible(comparable?) 
    template <typename A, typename B> 
    int flexibleCompare(const A& v1, const B& v2) {
        if (v1 < v2) return -1; 
        if (v2 < v1) return 1; 
        return 0;
    }
    long lng; 
    flexibleCompare(lng, 1024); // ok: calls flexibleCompare(long,int)
    ```
    * 這樣上面的 call 就會合法；記得這樣的話 `operator<` 就要能作用在這兩個不同的 type 上

#### Normal Conversions Apply for Ordinary Arguments
* 其實就是說 function template 內，如果是用正常型態宣告的 function parameters，它還是保有一般的 conversion 機制:
    ```c++
    template <typename T> ostream &print(ostream &os, const T &obj) {
        return os << obj;
    }
    ```
    * 上面的 code，一般的轉換規則會用在第一個 parameter 上(包括 user defined conversion 三小的，and ref/ptr 的多型機制，whatever)，所以下面的 code work:
    ```c++
    print(cout, 42); // instantiates print(ostream&, int)
    ofstream f("output");
    print(f, 10); // uses print(ostream&, int); converts f to ostream&
    ```
    * 第二個 call 因為存在把 `ofstream&` 到 `ostream&` 的轉換，所以 compiler 一樣會呼叫 `print(ostream&, int)`

### 16.2.2 Function-Template Explicit Arguments
* 還是有情況是 compiler 連推斷 template arguments 的型別都沒辦法的，這時我們可以明確指示型別
* 這種情況最常發生在 function return type 跟 parameter list 的任何 type 都不一樣的時候(看到例子你就懂惹)

#### Specifying an Explicit Template Argument
* 定義一個 `sum`，吃兩個不同型別的 args，然後可以讓 user 指定回傳的型別，*以達到 user 要求的精確值*:
    ```c++
    // T1 cannot be deduced: it doesn’t appear in the function parameter list 
    template <typename T1, typename T2, typename T3>
    T1 sum(T2, T3);
    ```
    * 這時就很明顯，compiler 不可能單純用你傳的 function arguments 來推斷 `T1` 的型別
    * The caller must provide an **explicit template argument** for this parameter on *each* call to sum.
    ```c++
    // T1 is explicitly specified; T2 and T3 are inferred from the argument types 
    auto val3 = sum<long long>(i, lng); // long long sum(int, long)
    ```
    
* Explicit template argument(s) are matched to corresponding template parameter(s) **from left to right;**
    * the first template argument is matched to the first template parameter, the second argument to the second parameter, and so on.
    * 換句話說，template arguments 只能從最右邊開始省略的意思，**而且要 compiler 有辦法推斷型別的情況才能省略**
* 上面的省略 convention 意味著你 template type parameters 的宣告順序很重要! 看例子:
    ```c++
    // poor design: users must explicitly specify all three template parameters 
    template <typename T1, typename T2, typename T3>
        T3 alternative_sum(T2, T1);
    ```
    * 這個情況是 compiler 不能推斷最右邊的 type，所以你一定要明確指定型別，而要為它提供代表它左邊所有的也要被提供(不然你提供的 type 會被當成是 `T1` 的型別)，換句話說就是全部的 type 都要明確提供...
    ```c++
    // error: can’t infer initial template parameters 
    auto val3 = alternative_sum<long long>(i, lng);
    // ok: all three parameters are explicitly specified
    auto val2 = alternative_sum<long long, int, long>(i, lng);
    ```

#### Normal Conversions Apply for Explicitly Specified Arguments
* 注意之前說 template 幾乎不太做 conversion，不過那只有在 compiler 自己推斷型別的狀況；**如果你用上面的方法明確指定 template argument 的型別，那對應的 function argument 就會使用一般的 conversion:**
    ```c++
    long lng; 
    compare(lng, 1024); // error: template parameters don’t match
    compare<long>(lng, 1024); // ok: instantiates compare(long, long)
    compare<int>(lng, 1024); // ok: instantiates compare(int, int)
    ```
    * 記得我們寫的 `compare` 只有一個 template type parameter，`T`
    * 第一個合法 call 會把 `1024` 轉成 `long`
    * 第二個合法 call 則是把 `lng` 轉成 `int`

### 16.2.3 Trailing Return Types and Type Transformation
* 明確指定 template argument 大概只有在要讓 user 自己確認 return type 時有用，其他時候都是添 user 的麻煩(?
* 例如下面的 code:
    ```c++
    template <typename It> 
    ??? &fcn(It beg, It end) {
        // process the range 
        return *beg; // return a reference to an element from the range
    }
    ```
    * 吃一個 range，然後 return 某個 element 的 reference
    * 這時候我們要寫 return type 就很奇怪，因為根本沒辦法知道，所以可能要讓 return type 也是 template arg，然後讓 user 明確指定，可是對 user 來說在這個情況下提供 element type 很奇怪，他會覺得你對 iterator deref 不就知道了zzz
    * 例如 user 這樣寫:
    ```c++
    vector<int> vi = {1,2,3,4,5}; 
    Blob<string> ca = { "hi", "bye" }; 
    auto &i = fcn(vi.begin(), vi.end()); // fcn should return int&
    auto &s = fcn(ca.begin(), ca.end()); // fcn should return string&
    ```
* 我們知道 return type 跟 `*beg` 是相同的，可是你不能直接在 return type 那裏直接寫 `*beg` 阿 lol，而且在 parsing return type 時 compiler 還沒進到 function scope，所以 compiler 也看不到 `beg`
* 所以這裡用 c++11 的 `decltype`，而且因為上面說的，還沒看到 `beg` 的關係，你要寫成 **trailing return type(§ 6.3.3, p. 229)**:
    ```c++
    // a trailing return lets us declare the return type after the parameter list is seen 
    template <typename It> auto fcn(It beg, It end) -> decltype(*beg) {
        // process the range 
        return *beg; 
        // return a reference to an element from the range
    }
    ```
    * 阿別忘記，`decltype` 傳給他一個 dereference expression 時它會推出 reference type
    * Thus, if fcn is called on a sequence of `string`s, the return type will be `string&`. If the sequence is `int`, the return will be `int&`.

#### The Type Transformation Library Template Classes
* 這個章節很ㄎㄧㄤ，小心(?
* 首先上一個 section 是達到 return element *的 reference* 的效果，可是我們沒辦法用上面的方式 return element 的 value，亦即，return copy，而 iterator 又沒有 operation 可以拿到 element 的 value(都是拿到 lvalue，會被 `decltype` 當成 reference)
* 所以我們要得到 elements 的 nonreference type 要用別的方法:
    * we can use a **library type transformation template**.
    * defined in the `<type_traits>` header
* 這 header 的鬼東西基本上是要寫 **meta programming** 用的，Primer 不會講，不過我們還是可以這它來處理現在面臨的問題
* 下表是 header 內有的東西:
    * ![](https://i.imgur.com/EIEWksH.png)
* 在這個 case 我們可以用裡面的 `remove_reference`:
    * 它有一個 template type parameter
    * 還有一個 (public) type member: `type`
* **如果實例化一個 remove_reference 時給定一個 reference type，那 `type` 就會是 refereed-to type:**
    * `remove_reference<int&>`, the type member will be `int`.
    * `remove_reference<string&>`, type will be `string`.

* 如果用在我們要處理的問題:
    ```c++
    remove_reference<decltype(*beg)>::type
    ```
    * 我們就可以得到 `*beg` 綁定的的 element type!
* 我們在把原本的 function template 宣告改一下:
    ```c++
    // must use typename to use a type member of a template parameter; see § 16.1.3 (p. 670) 
    template <typename It> auto fcn2(It beg, It end) -> 
            typename remove_reference<decltype(*beg)>::type
    {
        // process the range 
        return *beg; // return a copy of an element from the range
    }
    ```
    * 這樣回傳的 type 就會是 (nonreference)element type 了!
* 注意在 trailing return type 那邊有加一個 `typename`，因為這個 type 是 depends on `remove_reference` 的 type parameter(也就是那該死的 `decltype(*beg)`)的，所以要用 `typename` 告訴 compiler `type` 是 type member (§ 16.1.3, p. 670)
	* 不是 depends on `It` 喔，這很重要

* 其他表中的 template，用起來都跟 `remove_reference` 差不多，都有一個 `type` member，它實際代表的型別取決於你用哪個 template，如果你實例化時傳入的 type 不 make sense(in terms of template itself)，那 `type` 有可能跟你實例化傳入的型別相同
    * 例如 `remove_pointer`，如果你傳指標進去，它的 `type` 就會把指標移除
    * 如果你傳非指標進去，它的 `type` 就會跟你傳的型別一樣

### 16.2.4 Function Pointers and Argument Deduction
* 看到 function pointer 就知道要頭痛惹...
* 當我們初始化或 assign 一個 function template 到一個 function pointer 時，compiler 用 function pointer 的型別去推斷 template arguments
* 看例子:
    ```c++
    template <typename T> int compare(const T&, const T&); 
    // pf1 points to the instantiation int compare(const int&, const int&)
    int (*pf1)(const int&, const int&) = compare;
    ```
* 你還是可以寫出很ㄎㄧㄤ的 error，例如你 overload function，然後他們都吃 function pointer 的時候...
    ```c++
    // overloaded versions of func; each takes a different function pointer type 
    void func(int(*)(const string&, const string&)); 
    void func(int(*)(const int&, const int&));
    func(compare); // error: which instantiation of compare?
    ```
    * 因為有兩種實例化可以使用，compiler 直接噴 error

* 不過你還是可以用 explicit template argument 解決啦:
    ```c++
    // ok: explicitly specify which version ofcompare to instantiate 
    func(compare<int>); // passing compare(constint&, constint&)
    ```
    
### 16.2.5 Template Argument Deduction and References
* 這種 func temp:
    ```c++
    template <typename T> void f(T &p);
    ```
* function's parameter 是 reference to template type parameter
* 看這種 template 要記得兩件事:
    * Normal reference binding rules apply;
    * and consts are *low level, not top level.*
#### Type Deduction from Lvalue Reference Function Parameters
* 如果 function parameter 是 ordinary(lvalue) reference，也就是 T&，則 binding rule 告訴我們只能傳入 lvalue
    * 這個 lvalue 可能是也可能不是 `const`，如果是的話，T 就會被 deduced 成 `const` type:
    ```c++
    template <typename T> void f1(T&); // argument must be an lvalue 
    // calls to f1 use the referred-to type of the argument as the template parameter type 
    f1(i); // i is an int; template parameter T is int 
    f1(ci); // ci is a const int; template parameter T is const int
    f1(5); // error: argument to a & parameter must be an lvalue
    ```
* 如果 function parameter 是 `const T&`，則 binding rule 告訴我們我們可以綁定任何種類的 value--an object (`const` or otherwise), a temporary, or a literal value.
    * **如果 function parameter 已經宣告成 `const`，則 deduced type `T` 就不會是 `const` 了**；**`const` 已經是 *function* parameter type 的一部分，它就不會變成 *template* parameter type 的一部分了:**
    ```c++
    template <typename T> void f2(const T&); // can take an rvalue 
    // parameter in f2is const&; constin the argument is irrelevant 
    // in each of these three calls, f2’s function parameter is inferred as const int& 
    f2(i); // i is an int; template parameter T is int 
    f2(ci); // ci is a const int, but template parameter T is int
    f2(5); // a const& parameter can be bound to an rvalue; T is int
    ```
    * **知道 `const` 到底包含在 function parameter 還是 template parameter 內很重要，因為你很可能會拿 template parameter 來宣告變數，所以你當然要知道它是不是 const** 
    * 你可以把上面的 code 丟到 IDE 然後看 IDE 會呼叫的實例內，角括號的型態是 `int`，並沒有 `const`

#### Type Deduction from Rvalue Reference Function Parameters
* 總之你的 function parameter 是 `T&&` 時會用到，要 exactly 長得像 `T&&`，不能是什麼 `vector<T>&&` 之類的
    ```c++
    template <typename T> void f3(T&&);
    f3(42); // argument is an rvalue of type int; template parameter T is int
    ```
    * 規則基本上跟 lvalue ref 差不多

#### Reference Collapsing and Rvalue Reference Parameters
* 這裡很機掰...
* ***可以去看 Scott Meyers 大大說的 Universal Reference***
* 如果你某 template 的 function parameter 是定義 rv ref，你一定會覺得這樣的 code 會噴 error:
    ```c++
    template <typename T> void f(T&&);
    int i;
    f(i); // ?
    ```
    * 畢竟之前學過不能用 rv ref 來綁 lvalue
* **但這他媽又是一個例外...**
* 首先如果你對一個收 rv ref 的 parameter 傳 lvalue 進去，compiler **會把 template argument 推論成 lv ref**
* 換句話說上面 code 的 `T` 在呼叫 `f(i)` 時會被推論成 `int&'
* 那這樣看起來就更奇怪了，因為這時 f 的 *function* parameter 看起來就好像是 rv ref to int&，問號，不是說不能有 ref to ref?
* However, **it is possible to do so indirectly through a type alias (§ 2.5.1, p. 67) or through a template type parameter.**
    * 你可以用 type alias 來定義這種東西，或者是現在說的情況來達成 ref to ref

* 如果真的間接定義出 ref to ref，則會發生 reference "collapse":
    * 首先 ref to ref 有四種狀態: l to l, l to r, r to l, r to r
    * **除了 r to r，其他的都會 "collapse" 成 lvalue reference**

* 或者這樣說:
    * X& &, X& &&, and X&& & all collapse to type X&
    * The type X&& && collapses to X&&

* 再提醒一次，以上的情況只有在 type alias，跟 template type parameter 是 rv ref 時才會發生

* 所以上面的 `f(i)` 到底會推成三小?
    * `f` 原本吃 rv ref，你給他 lvalue，所以在這個特殊情況下，*template* type parameter `T` 會被推成 ref type `int&`
    * 而 rv ref to `int&`，就會 collapse 成 `int&`，所以 *function* parameter 會是 `int&`
* 總之 compiler 會推出像這樣的實例(展示用，not valid code):
    ```c++
    // invalid code, for illustration purposes only 
    void f3<int&>(int& &&); // when T is int&, function parameter is "int& &&"
    ```
* 經過 collapse 之後就是這樣:
    ```c++
    void f3<int&>(int&); // when T is int&, function parameter collapses to int&
    ```

* There are two important consequences from these rules:
    * A function parameter that is an rvalue reference to a template type parameter (e.g., T&&) can be bound to an lvalue
        * 這個情況下 rv ref 可以 bind lvalue
    * If the argument is an lvalue, then the deduced template argument type will be an lvalue reference type and the function parameter will be instantiated as an (ordinary) lvalue reference parameter (T&)
        * 你如果傳的 argument 是 lvalue，會導致對應的 template type parameter 是 reference type；然後 function parameter type 也會是 lvalue reference

* 既然他可以綁定 lv ref 又可以綁 rv ref，換句話說我們可以傳入任何種類的 value(argument) 給這種 rvalue reference 綁定
* 但之後會看到，**其實只會在兩個地方用到這個功能，在其他地方把 lvalue 傳給 temp 的 rv ref 只會是悲劇**

#### Writing Template Functions with Rvalue Reference Parameters
* 上面那該死的情境會導致 template type parameter 被推成 reference type，那下面的 code 就會很機掰:
    ```c++
    template <typename T> void f3(T&& val) {
        T t = val; // copy or binding a reference? 
        t = fcn(t); // does the assignment change only tor val and t? 
        if (val == t) { /* .. . */} // always true if T is a reference type
    }
    ```
    * 自己想想 T 是一般物件或者是 reference 時會發生什麼事，差很多

* 總結: in practice，template 會用到 rv ref 的情況只有下面兩個:
    * template forward its arguments, 16.2.7 會講這是三小
    * template 被 overloaded，16.3 會講
* 現在先知道一個重要的東西，用了 rv ref 的 template 通常會做 overload，長得很像 13.6.3 overload 的方法:
    ```c++
    template <typename T> void f(T&&); // binds to non const rvalues
    template <typename T> void f(const T&); // lvalues and const rvalues
    ```
* 這時候 compiler 就會跟據傳入給 `f` 的 type 決定要丟給哪個版本的 `f`；modifiable rvalues 就會丟給 `f(T&&)`，lvalue 或者 `const` rvalue 就會給 `f(T&)`

### 16.2.6 Understanding `std::move`
* The library move function (§ 13.6.1, p. 533) is a good illustration of a template that uses rvalue references.
* In § 13.6.2 (p. 534) we noted that although we cannot directly bind an rvalue reference to an lvalue, we can use move to obtain an rvalue reference bound to an lvalue.
* Because move can take arguments of essentially any type, *it should not be surprising that move is a function template.*

#### How `std::move` Is Defined
* 首先我討厭這裡 Primer 的安排，它先用 `static_cast` demo 給你看說可以把 lvalue 轉成 rvalue，再跟你說 `static_cast` 可以這樣用...
* 總之，這種轉換 compiler 不會隱性幫你做，不過你可以用 `static_cast` 明確指示說你要這樣轉換

* 來看 `move` 怎麼定義:
    ```c++
    // for the use of typename in the return type and the cast see § 16.1.3 (p. 670) 
    // remove_reference is covered in § 16.2.3 (p. 684) 
    template <typename T> 
    typename remove_reference<T>::type&& move(T&& t) {
        // static_cast covered in § 4.11.3 (p. 163) 
        return static_cast<typename remove_reference<T>::type&&>(t);
    }
    ```
    * 你可以去翻翻 source code，還真的長這樣
* return type 依賴於 `remove_reference` 這個，所以用 `typename` 跟 compiler 告知
* 最後 `return` 時用 `static_cast` 還有 `remove_reference`，下面講他們到底在幹嘛
* 首先記得我們是要 demo 的是**不管你傳的是 lvalue 還是 rvalue，`move` 都會回傳一個 rvalue**
* 再來，看清楚 move 的參數是 `T&&`，是 rvalue ref，**所以我們要看我們傳入 l/rvalue 給 move 時 `T` 分別會被推斷成什麼型態**
* 用這段 code 講解:
    ```c++
    string s1("hi!"), s2; 
    s2 = std::move(string("bye!")); // ok: moving from an rvalue
    s2 = std::move(s1); // ok: but after the assigment s1 has indeterminate value
    ```
#### How `std::move` Works
* 先看 `s2 = std::move(string("bye!"));` 的情況:
    * 因為只是很普通的把 rvalue 丟給 rvalue ref 綁，所以 `T` 就只是按照普通規則推斷成 `string`
    * 所以使用 `remove_reference` 時會實例化一個 `remove_reference<string>`
    * `remove_reference<string>::type` 一樣是 `string` (傳入的 type 如果不是 reference 的話就維持原本的 type)
    * `remove_reference<string>::type&&` 就會是 rvalue ref 惹!
    * 所以 `move` 的 return type 是 rvalue ref
* 你也可以說這個 call 實例化了 `move<string>`:
    ```c++
    string&& move(string &&t)
    ```
* 再來看 `s2 = std::move(s1);`
    * 因為你把 lvalue 傳給 rvalue ref，所以這裡的 `T` 被推斷成 reference type，也就是 `string&`
    * 所以你會實例化一個 `remove_reference<string&>`
    * 他的 `type` member 就會把 reference 移除，所以是 `string`
    * 那你的 move return type 是 `type&&`，所以還是 `string&&`
    * 題外話，這時 move 的 *function* parameter 會從 `string& &&` collapse 成 `string&`
* 你也可以說這個 call 實例化了 `move<string&>`:
    ```c++
    string&& move(string &t)
    ```
    * 這才是常用的情況，我們希望把 rvalue ref 綁到一個 lvalue 上!

#### `static_cast` from an Lvalue to an Rvalue Reference Is Permitted
* 你看，Primer 都用了才講...
* Binding an rvalue reference to an lvalue **gives** code that operates on the **rvalue reference permission to clobber the lvalue.**
* 例如還記得的話，13.6.1 的 `StrVec` 的 `reallocate` 就有用 `move` 把舊的空間的 elements 直接 move 到新空間，因為我們知道舊的空間的 elements 準備被破壞惹
* 最後，請不要手動 call `static_cast` 把 lvalue 轉成 rvalue_reference，要轉也是 call `move`，比較好用，而且我們可以直接在 debug 時看哪裡可能用了 `move` 導致物件意外被弄爛
	* 你可以想成 `std::move` 就是這種噁心的 `static_cast` 的一個 wrapper，讓你比較簡單使用

### 16.2.7 Forwarding
* use case: 有的時候 function template 會有一種寫法，就是把 function parameter 丟給另一個 function 來幫你做事情，template 本身有點像 wrapper 的概念
* 這個時候我們可能會需要**確保 function parameter 被丟進另一個 function 時，它們的所有特性都要被保留**，例如 `const`，跟是否為 reference type，rvalue 還是 rvalue，否則另一個 function 的執行行為可能不是我們想要的
* 例如我們設計一個 function template 叫做 `flip1`，他會收一個 callable object 跟兩個參數，然後他做的事情就是呼叫 callable object，並且把兩個參數傳進去，只是順序倒過來；
* 以下是*錯誤的實作版本*
    ```c++
    // template that takes a callable and two parameters 
    // and calls the given callable with the parameters "flipped" 
    // flip1 is an incomplete implementation: top-level const and references are lost 
    template <typename F, typename T1, typename T2> 
    void flip1(F f, T1 t1, T2 t2) {
        f(t2, t1);
    }
    ```
    * 這個 template，在傳入的 callable object 的 parameter type 是 nonreference 的時候都是正確的，因為如果是 nonref type，這代表 callable object 無法改變傳入的 arguments。而實際上 `flip1` 呼叫他時，傳入的 `t1 t2` 也不會被改變，所以行為正常
    * 但是如果 callable object 的 parameter type 是 reference type 就會有問題了，因為如果是 reference type，那概念上這代表 callable object 有意思要更改被綁定的 argument，**可是當你在把 `t1 t2` 傳遞給 `flip1` 時，首先你的 `flip1` 會把 template type parameter(也就是 `T1` `T2`) 推成 nonreference type，這樣的話 `flip1` 的 function parameter type 也會是 nonreference type**，所以傳入到 `flip1` 的 arguments 是**被複製** 到 `t1 t2`，你在 `flip1` 不管怎麼更改 `t1 t2` 都改不到原本的 arguments 了
        * 注意這裡 `f` 還是可以改到傳給 `f` 的 `t1 t2`，因為 `f` 的 parameter 是 reference type，可是 `f` 還是改不到傳給 `flip1` 的 arguments。
* 看 code...
* 例如你這樣定義 `f`:
    ```c++   
    void f(int v1, int &v2) // note v2 is a reference {
        cout << v1 << " " << ++v2 << endl;
    }
    ```
    * 如果你做下面的呼叫:
    ```c++
    f(42, i); // f changes its argument i
    flip1(f, j, 42); // fcalled through flip1leaves junchanged
    ```
    * 直接呼叫 `f` 可以改到 argument，可是透過 `flip1` 呼叫就沒有改到了，也就是說 `j` 不會被更改
    * 如果看不懂我上面在打三小，你可以看 compiler 生出什麼實例:
    ```c++
    void flip1(void(*fcn)(int, int&), int t1, int t2);
    ```
	    * 注意 `t1` 跟 `t2` 的型態都是 `int`，不是 `int&`!
    * The value of **j is copied into t1.** The **reference parameter in f is bound to t1, not to j.**

#### Defining Function Parameters That Retain Type Information
* 所以要來解決上面的問題惹
* we need to rewrite our function so that its parameters **preserve the "lvalueness"** of its given arguments.
* Thinking ahead a bit, we can imagine that we’d also like to preserve the `const`ness of the arguments as well.

* **做法是把 function parameter 的型態定義成 rvalue reference**
    * 首先定義成 ref，不管是 r 還 l(這裡我們是定義成 rvalue reference)，都會保持被綁定的型態的 `const`ness 因為 low level `const` never ignored
    * 而用 ref collapsing，這樣如果你傳進 function 的 argument 是 lvalue，則 parameter type 就會是 lv ref，argument 是 rvalue，parameter type 就會試 rv ref，忘記我在講啥去看 reference collapsing(§ 16.2.5, p. 687)
    ```c++
    template <typename F, typename T1, typename T2> 
    void flip2(F f, T1 &&t1, T2 &&t2) {
        f(t2, t1);
    }
    ```
* 以下是解釋，因為太ㄎㄧㄤ，完全不想打中文:
    * 一下說 `T1` 一下說 `t1`，要小心看，一個是 template type parameter 一個是 function argument
    * As in our earlier call, if we call `flip2(f, j, 42)`, the l**value** `j` is ***passed*** to the parameter `t1`. However, in `flip2`, **the type deduced for `T1` is `int&`, which means that the type of `t1` collapses to `int&`.** The reference `t1` is bound to `j`. When `flip2` calls `f`, **the reference parameter `v2` in `f` is bound to `t1`, which in turn is bound to `j`**. When `f` increments `v2`, it is changing the value of `j`.

* 小總結: A function parameter that is an rvalue reference to a template type parameter (i.e., T&&) **preserves the constness and lvalue/rvalue property of its corresponding argument.**
    * 簡單說，你傳入的 argument 的 `const`/lr 是什麼，你的 template 對應的 function parameter 就會是什麼，而且是 reference，所以會綁定 argument

* 但 `flip2` 只解決一半的問題...
    * 你傳 lvalue ref 給它行為會正常，可是傳 rvalue reference 就會出事

* 看例子:
    ```c++
    void g(int &&i, int& j) {
        cout << i << " " << j << endl;
    }
    ```
    * 如果我們想用 `flip2` 來 call `g`，
    ```c++
    flip2(g, i, 42); // error: can’t initialize int&& from an lvalue
    ```
    * 這時你的 `t1` 會綁到 42，`t2` 會綁到 i，而且他們本身是 rvalue reference
    * 可是重點來了，一個 variable name，不管它是 r/lvalue ref，直接用它的時候它就是 lvalue(§ 13.6.1, p. 533)，所以你把 `t2` 丟給 `g` 的 `i` 時，等於是把 lvalue 丟給 rvalue ref 來綁，所以會噴 eror...

#### Using `std::forward` to Preserve Type Information in a Call
* 講難聽一點啦，上面的問題是 c++11 才會碰到的，新標準當然要定義東西來解決R
* 跟 `move` 一樣定義在 `<utility>
* 呼叫的時候**一定要用 explicit template argument**
* forward **returns an rvalue reference to that explicit argument type.** 
    * That is, the return type of `forward<T>` is `T&&`.
    * 注意他跟 `move` 的差異，`move` 是不管你傳 l 還 r 進去，他都回傳 r；`forward` 則是回傳你傳進去的 type 的 rvalue reference，根據 collapsing rule，如果傳進去的 type 是 l，`forward` 就會 return l，如果是 r, `forward` 就會 return r；看 Primer p.694

* 所以你可以寫這樣的 function:
    ```c++
    template <typename Type> intermediary(Type &&arg) {
    finalFcn(std::forward<Type>(arg)); // ...
    }
    ```
    * 看上面的 `Type` 跟 `arg`
    * 因為 `arg` 是 rvalue ref，所以他可以把傳進來的 argument 的所有 type information 都保留(`const`/lr)
    * 如果 argument 是 rvalue，則 `Type` 是 plain type(就是沒有 reference)，那 `forward<Type>` 就是 `Type&&`
    * 如果 argument 是 lvalue，則 `Type` 是 lvalue reference，這時 `forward<Type>` 一樣是 lvalue reference type，因為 `forward` 實際上回傳的是 **rvalue ref to lvalue ref**，根據 reference collapsing，這樣是 lvalue ref

* 再總結: When used with a function parameter that is an rvalue reference to template type parameter (`T&&`), `forward` preserves all the details about an argument's type.

* 所以總結，我們的 `flip` 應該這樣寫:
    ```c++
    template <typename F, typename T1, typename T2> 
    void flip(F f, T1 &&t1, T2 &&t2) {
        f(std::forward<T2>(t2), std::forward<T1>(t1));
    }
    ```
    * ㄎㄧㄤ到爆...
    * If we call `flip(g, i, 42)`, i will be passed to g as an `int&` and 42 will be passed as an `int&&`.

* 最後一個注意: 不要對 `forward` 用 `using`，理由跟 `move` 一樣，18.2.3 會講

### 16.3 Overloading and Templates
* function template 可以被另一個同名字的 function template 或者一般 function 給 overload；其參數數量跟型態要符合 overload 需求
* function matching(p.233, 也就是選出 viable functions 的規則)套用在 function template 的情況如下:
	* 所有可以 match(或者說 deduce 的) 的 template instantiations 都是 candidate functions(忘記這是啥去翻第六章 index)
	* 所有可以 match 的 template instantiations 同時也是 viable functions，因為可以 deduce 出來的 instantiations 一定可以拿它們呼叫...
	* 如第六章的 rank 方式，function(templates) 也是用須要做的 conversion 來排名
		* 不過別忘記 function template 可以做的 conversion 很少
	* 一樣，如果 viable 裡面有一個 function 全部參數都是 better match，他就會被選擇來呼叫；如果沒有則看下列:
		* 如果有多個以上 ambiguous，**但是這些 viable 裡面 nontemplate function 只有一個**，那就是這個 nontemplate function 被呼叫
		* 如果 viable 內一個 nontemplate function 都沒有，而有多個 function templates ambiguous，**但是其中一個比其他都更加 specialized**，則他被呼叫
		* 否則就是 ambiguous call
* Warning: Correctly defining a set of overloaded function templates **requires a good understanding of the relationship among types and of the restricted conversions applied to arguments in template functions.**
	* 拜託還是不要打自己腳去 overload 這種鬼東西......
#### Writing Overloaded Templates
* overload matching 規則那麼複雜根本記不起來，還是先看 code 吧
* 我們來定義一些 debug 時很有用的 functions，`debug_rep`，每個版本都會回傳 `std::string`，代表某個物件的 string representation
	```c++
	// print any type we don’t otherwise handle
	template <typename T> string debug_rep(const T &t) {
		ostringstream ret; // see § 8.3 (p. 321) 
		ret << t; // uses T’s output operator to print a representation of t 
		return ret.str(); // return a copy of the string to which ret is bound
	}
	```
	* 這裡用了 `std::ostreamstring` 簡化字串處理，不過傳入物件要支援 `operator<<`
* 接下來定義另一個版本，吃 `T*`
	```c++
	// print pointers as their pointer value, followed by the object to which the pointer points 
	// NB: this function will not work properly with char*; see § 16.3 (p. 698) 
	template <typename T> string debug_rep(T *p) {
		ostringstream ret; 
		ret << "pointer: " << p;
		if (p) // print the pointer’s own value
			ret << " " << debug_rep(*p); // print the value to which p points
		else
			ret << " null pointer"; // or indicate that the pis null
		return ret.str(); // return a copy ofthe string to which ret is bound
	}
	```
	* 注意這個版本會呼叫前一個版本 LOL，超ㄎㄧㄤ
	* 然後這個版本對 `char*` 無效，因為 `operator<<` 已經對 `char*` 做 overload 了，因為 `ret << "pointer: " << p ;` 會把 `p` 當成 C style string 把指向的字串印出來直到 null character 為止，p.698 會講要怎麼處理這個情況

* 我們可以這樣用這兩個 templates:
	```c++
	string s("hi"); 
	cout << debug_rep(s) << endl;
	```
	* 上面這個直接把 s 的 string representation(這樣講很奇怪，但 string 也是物件，而且支援 <<，我們就說 string 有「string representation」吧)印出來
* 我們來分析 compiler 怎麼 match `debug_rep`
	* 首先只有第一個版本的 `debug_rep` 是 viable
	* 因為第二個版本是吃指標，而我們是傳物件進去，不可能把一個物件轉成指標，所以 deduction fails
			* 原來他說的會 fail 是指這種這麼瞎的情況... 原本想說什麼 type 都馬可以 instantiate，原來是指這種把物件轉成指標的情況會爛掉
			* 可以寫寫範例，看 compiler 會不會噴你 error
	```c++
	template <typename T> void eat_ptr(T *) {}

	int main() {
	  int i;
	  eat_ptr(i);
	}
	```
		* 所以因為只有一個 viable 所以就選他惹
* 接下來看下面這個:
	```c++
	cout << debug_rep(&s) << endl;
	```
	* 注意!**兩個 template 都是 viable!!**
	* 第一個 template 會產生 `debug_rep(const string*&)`，超ㄎㄧㄤ reference to pointer
	* 第二個會產生 `debug_rep(string*)`
	* 第二個 viable 是 exact match，第一個 viable 要把 normal pointer 轉成 pointer to `const`，所以第二個被呼叫
		* 覺得這邊怪怪的 LOL，就算第一個函數參數沒有 `const`，這邊還是會選第二個，不會 ambiguous，還是這是合法規則? 還是這是他媽的 specialized ?
### 題外話，cppreference 也是用 Candidate/viable function 來說明這些鬼東西的: http://en.cppreference.com/w/cpp/language/overload_resolution

#### Multiple Viable Templates
* 再看一個例子:
	```c++
	const string *sp = &s; 
	cout << debug_rep(sp) << endl;
	```
	* 這時兩個 template 都是 exact match! (其實就跟上面我在靠北的狀況一樣，只是我是說兩個都沒有 `const`，這邊是兩個都有 `const`)
	* 第一個生出 `debug_rep(const string*&)`
	* 第二個生出 `debug_rep(const string*)`
	* 這個 case 用第六章的 rule 來看是 ambiguous 的，所以你懷疑的沒錯
	* **但是!!** 這裡如果用 template 獨有的 matching rule，則會呼叫第二個 function，因為他 more specialized LOL

* Primer 是這樣解釋這個情況的:
	* 如果沒有上面那個 rule，則宣告成吃指標的那個模板，永遠不可能在我傳入 ptr to `const` 時，被呼叫(我故意這樣斷句的，請慢慢看)。真正的癥結點在於 **const T& 可以吃 ANY TYPE**，包括指標，我們說這樣的 template **更加的 general**，比吃指標的版本還要 general。如果沒有這個 rule，那傳入常數指標時永遠會 ambiguous
* 一個小總結: When there are several overloaded templates that provide an equally good match for a call, **the most specialized version is preferred.**

#### Nontemplate and Template Overloads
* 如果要讓一般函數跟模板函數 overload 呢?
* 為此我們再定義一個一般函數:
	```c++
	// print strings inside double quotes 
	string debug_rep(const string &s) {
	return '"' + s + '"';
	}
	```
* 這時寫這樣的 code 會如何?:
	```c++
	string s("hi"); 
	cout << debug_rep(s) << endl;
	```
	* 這時有兩個 viable functions, 而且 equally good LOL
	* `debug_rep<string>(const string&)`，從第一個模板生的
	* `debug_rep(const string&)`，一般函數
	* **但這裡的規則就是，一般函數會被呼叫!**
	* 你就想嘛，template 比較 general，你寫的一般函數比較專屬於某個型態，所以 compiler 就順你的意去呼叫那個一般函數了
* 再一個小總結: When a nontemplate function provides an equally good match for a call as a function template, the nontemplate version is preferred.
	* 如果有一個一般函數跟模板實例一樣好，compiler 會挑一般函數來呼叫

#### Overloaded Templates and Conversions
* 我們還沒說怎麼處理上面的傳入 C style string 的窘境阿
* 假設我們寫這樣的 code:
	```c++
	cout << debug_rep("hi world!") << endl; // calls debug_rep(T*)
	```
	* 我們會以為，我們都已經寫了一個吃 `std::string` 的一般函數了，compiler 應該會 match 它(然後做必要的轉型)吧? 錯......
	* 首先這個狀況有三個 viable functions:
	* `debug_rep(const T&)`, with `T` bound to `char[10]`
	* `debug_rep(T*)`, with `T` bound to `const char`
	* `debug_rep(const string&)`, which requires a conversion from `const char*` to `string`
	* 首先第一個第二個 template 都是 exact match(注意第二個從 array decay 成 ptr 算是 exact match，詳情 p.245)
	* 第三個 viable 反而還需要 user defined(正確來說是 standard defined) conversion(C style string to `std::string`)，所以不會考慮
	* 而上面已經講過原因了，吃指標的版本比吃 reference 的版本更 specialized，所以會被呼叫
* 解決方法就是定義兩個吃 `char*` 的一般 function...
	```c++
	// convert the character pointers to string and call the string version of debug_rep 
	string debug_rep(char *p) {
		return debug_rep(string(p)); 
	}
	string debug_rep(const char *p) {
		return debug_rep(string(p));
	}
	```
	* 這樣你傳入 `char[]` 時，有四個版本都是 exact match(注意現在總共有五個 function(templates)...，包含 `debug_rep(const string&)`)，阿 compiler 會挑一般函數的，不用轉成 `const` 的
	* 你傳入 `const char[]` 時，有三個 exact match，compiler 挑吃 `const char*` 的一般函數

#### Missing Declarations Can Cause the Program to Misbehave
* 這裡真的很靠北...
* 請注意看我們的，最上面那兩個吃 C style string 的版本，他們的 return 會呼叫吃 `const string&` 的**一般版本**...? 嗎?
	* **注意，如果你沒有在寫這兩個版本的函數之前先宣告吃 `const string&` 的版本，那 compiler 就會使用吃 `const T&` 的 template 生一個實例出來給你呼叫= =**，
	* 注意那個吃 `const &` 的模板做的事情跟我們自定義的 `const string&` 做的事情是不一樣的
	```c++
	template <typename T> string debug_rep(const T &t); 
	template <typename T> string debug_rep(T *p); 
	// the following declaration must be in scope 
	// for the definition of debug_rep(char*) to do the right thing 
	string debug_rep(const string &); 
	string debug_rep(char *p) {
		// if the declaration for the version that takes a const string& is not in scope 
		// the return will call debug_rep(constT&) with T instantiated to string 
		return debug_rep(string(p));
	}
	```
* TIP: **Declare every function in an overload set before you define any of the functions.** That way you don’t have to worry whether the compiler will instantiate a call before it sees the function you intended to call.
	* 既然在寫定義的時候，如果會因為沒看到某些宣告，compiler 就偷偷幫你生一個不該生的實例的話，那就先把 overload set 的 functions 全部宣告完後，再開始定義

## 16.4 Variadic Templates
* 可以吃任意數量的 template parameter 的 templates
* 這些數量可以任意變化的 parameters 叫做 **parameter pack**
* 有兩種 parameter packs:
	* template parameter packs(tpp): 代表有 0 到多個 template parameters
	* function parameter packs: 代表有 0 到多個 function parameters
* 用 ellipsis(`...`)來說明一個 function 或 template parameter(name) 是一個 parameter pack
	* 具體來說，如果在 template parameter list(也就是腳括號內)使用 `typename ...` 或者 `class ...` 來宣告 template parameter name，那個 name 就代表了 template parameter packs，**代表了 0 到多個不同的 types**
	* 如果是在 function parameter list 內想要用 function parameter pack，則這個對應的參數型別就要是 template parameter pack LOL，繞口令?
* 看例子= =
	```c++
	// Args is a template parameter pack; rest is a function parameter pack 
	// Args represents zero or more template type parameters 
	// rest represents zero or more function parameters
	template <typename T, typename... Args>
	void foo(const T &t, const Args& ... rest);
	```
	* `foo` 是個 variadic template function，有一個 template type parameter `T`，跟一個 template parameter pack，`Args`
		* `Args` 代表了額外的 0 到多個 type parameters
	* 而 `foo` 的 function parameter list 則有一個 function parameter `T`，確切型別要看 instantiation，還有一個 function parameter pack，`rest`
		* `rest` 則是代表了額外的 0 到多個 function parameters
* 在用這種有 pack 的 template 時，compiler 除了幫你 deduce type 外，還會 deduce 有多少數量的型別:
	```c++
	int i = 0;   double d = 3.14;   string s = "how now brown cow";
	foo(i, s, 42, d); // three parameters in the pack
	foo(s, 42, "hi"); // two parameters in the pack 
	foo(d, s); // one parameter in the pack 
	foo("hi"); // empty pack
	```
	* `foo` 的第一個 arguments 會用來推斷 `T` 的型別；剩下的則都拿來推斷 template parameter pack 內包含的型別(以及數量)
* compiler 會按照上面的 code 生出以下四個 instantiations:
	```c++
	void foo(const int&, const string&, const int&, const double&);
	void foo(const string&, const int&, const char(&)[3]);
	void foo(const double&, const string&);
	void foo(const char(&)[3]);
	```
#### The `sizeof...` Operator
* 有時需要靜態就知道 pack 內有幾個 parameters，這時就可以用 `sizeof...` 來知道:
	```c++
	template<typename ... Args>  void g(Args ... args) {
		cout << sizeof...(Args) << endl; // number of type parameters 
		cout << sizeof...(args) << endl; // number of function parameters
	}
	```
	* **return constant expression**
	* 傳入 `sizeof...` 的 argument 一定要是 parameter pack，不管是 function/template parameter pack 都可以

### 16.4.1 Writing a Variadic Function Template
* 還記得第六章講的 `initializer_list` 嗎，他可以吃任意數量的**同型別的** arguments
* Variadic functions 則是用在我們既不知道 user code 會傳的參數的型別，也不知道 user code 會傳多少參數
* 之後會定義一個 `errorMsg` 來示範怎麼用
* 不過在這之前先定義一個 variadic 版本的 `print`
	* 他會把所有 arguments，管你有幾個&&什麼型別，全部都印出來
* 然後 Primer 這裡講了一點很ㄎ一ㄤ的: **Variadic functions are often recursive** (§ 6.3.2, p. 227)
	* 第一次呼叫時先從 pack 內抓一個參數出來處理，然後 pack 內剩下的參數就拿來遞迴呼叫...
	* 你可能會覺得這樣 function 的 interface 跟參數要怎麼傳? 看例子就懂啦:
	```c++
	// function to end the recursion and print the last element 
	// ***this function must be declared before the variadic version of print is defined*** 
	template<typename T> 
	ostream &print(ostream &os, const T &t) {
		return os << t; // no separator after the last element in the pack
	}
	// this version of print will be called for all but the last element in the pack 
	template <typename T, typename... Args> 
	ostream &print(ostream &os, const T &t, const Args&... rest)
	{
		os << t << ", "; // print the first argument
		return print(os, rest...); // recursive call; print the other arguments
	}
	```
	* 重點就是第二個版本的 `print`，裡面的 `return` 部分；直接把 `rest...` 這樣傳遞的話，pack 內的第一個參數就會被綁到 `t`，剩下的綁到下一次 `print` 的 `rest...`
	* 而第一個版本的 `print` 等等會講是拿來幹嘛的
	* 舉例子:
	```c++
	print(cout, i, s, 42); // two parameters in the pack
	```
	![](https://i.imgur.com/WTmMwlh.png)
	* 前兩次 call 時，第一個版本都不是 viable function，因為他只吃兩個參數，而前兩次 call 一次傳四個參數，一次傳三個；
	* 但第三次 call 時，只傳兩個參數，這樣的話兩個版本的 `print` 都會是 viable；
		* 注意 parameter pack 可以為空，所以第二個版本的 `print` 雖然看起來有三個參數，但是只傳遞兩個參數的話這個 `print` 還是 viable 的
	* 而根據 specialized rule，nonvariadic template 比 variadic 更 specialized，所以會呼叫第一個版本的 `print`
* **注意如果沒有讓第一個版本的 `print` 先宣告，會編不過噴 error! 而不是 Primer 說的無限遞迴...**
* 另外我原本想要這樣寫:
	```c++
	template <typename T, typename... Args>
	ostream &print(ostream &os, const T &t, const Args &... rest)
	{
	  if (sizeof...(rest) > 0)
	  {
	    cout << "sizeof: " << sizeof...(rest) << "\n";
	    os << t << "\n";           // print the first argument
	    return print(os, rest...); // recursive call; print the other arguments
	  }
	  else
	  {
	    return os << t;
	  }
	}

	int main() { print(cout, 1, 2, 3, 4, 5) << endl; }
	```
	* 注意那個 `if (sizeof...(rest) > 0 )` 的部分；我原本是想要用這個來防止 compiler 去找 `print(ostream)` 這個 function，可是還是會噴 error，結果有人跟我說這要用 **SFINAE** 來解...
	* **或者!!** 用 C++17 更ㄎ一ㄤ的 `if constexpr`....:
	```c++
	template <typename T, typename... Args>
	ostream &print(ostream &os, const T &t, const Args &... rest)
	{
	  if constexpr (sizeof...(rest) > 0)
	  {
	    cout << "sizeof: " << sizeof...(rest) << "\n";
	    os << t << "\n";           // print the first argument
	    return print(os, rest...); // recursive call; print the other arguments
	  }
	  else
	  {
	    return os << t;
	  }
	}

	int main() { print(cout, 1, 2, 3, 4, 5) << endl; }
	```
	* 注意那個ㄎ一ㄤ到爆的 `if constexpr (sizeof...(rest) > 0)`
	* 其實看到這種寫法大概就知道為何 C++11 這樣編不過，因為他根本不能知道你在 (C++11內的) `if` 擺的 expression 是 constant expression 還是沙小阿，所以上面那個遞迴肯定就會考慮到 `print(ostream)` 這種情況
### 16.4.2 Pack Expansion
* 除了拿 pack 的 size，我們也可以把它 **expand**
* 在 expand 的時候，我們也要提供一個 **pattern** 來說明怎麼展開每個 pack 內的 element
* 基本上 expand 就是把一個 pack 變成一個一個組成原本 pack 的 elements
	* 要 trigger expand，就是在 pack 右邊擺 `...`，換句話說 Primer 又先斬後奏了，因為上面的 code 已經用到了
* 例如我們的 `print` 就有用到兩次 expand:
	```c++
	template <typename T, typename... Args> 
	ostream & 
	print(ostream &os, const T &t, const Args&... rest) // expand Args 
	{
		os << t << ", "; 
		return print(os, rest...); // expand rest
	}
	```
	* 第一個 expand 把 `Args` 展開然後**生成 `print` 的 function *parameter* list**
	* 第二個 expand 是在遞迴 call `print` 的時候；這時則是**生成 `print` 的 *argument* list**
* 我們再細看他們各自的 "pattern":
	* 展開 `Args` 的時候，pattern 其實就是 `const Args&`，這樣每個 element 就會在展開的時候多了 `const` 跟 reference
		* 寫的更精確一點: `const type&`
		* 例如:
	```c++
	print(cout, i, s, 42); // two parameters in the pack
	```
	* 簡單來說就會被展開成這樣啦...:
	```c++
	ostream& print(ostream&, const int&, const string&, const int&);
	```
	* 而第二個遞迴 call `print` 時，pattern 就很單純，只有 pack 本身的 name 而已，也就是 `rest`
		* 總之會展開成這樣:
	```c++
	print(os, s, 42);
	```
#### Understanding Pack Expansions
* 上面的 expand 看起來很 toy? 來看看更ㄎ一ㄤ的展法
* For example, we might write a second variadic function that calls `debug_rep` (§ 16.3, p. 695) on each of its arguments and then calls `print` to print the resulting `string`s:
	```c++
	// call debug_rep on each argument in the call to print 
	template <typename... Args> 
	ostream &errorMsg(ostream &os, const Args&... rest) {
		// print(os, debug_rep(a1), debug_rep(a2), ..., debug_rep(an) 
		return print(os, debug_rep(rest)...);
	}
	```
	* 在 `errorMsg` 內呼叫 `print` 的地方用的 pattern 是 `debug_rep(rest)`
	* That pattern says that **we want to call `debug_rep` on each element in the function parameter pack rest**.
	* The resulting expanded pack will be a comma-separated list of calls to `debug_rep`
	* 看 code:
	```c++
	errorMsg(cerr, fcnName, code.num(), otherData, "other", item);
	```
	* 裡面呼叫 `print` 的部分會變這樣:
	```c++
	print(cerr, debug_rep(fcnName), debug_rep(code.num()), 
				debug_rep(otherData), debug_rep("otherData"),
				debug_rep(item));
	```
* 如果你寫下面的 code，則無法編譯，**注意看 `...` 的位置!!!**
	```c++
	// passes the pack to debug_rep; print(os, debug_rep(a1, a2, ..., an)) 
	print(os, debug_rep(rest...)); // error: no matching function to call
	```
	* 最上面的 code 是寫 `debug_rep(rest)...`，這裡則是寫 `debug(rest...)`，	`...` 作用的 pattern 不同
	* 硬要展開的話就會變成這樣:
	```c++
	print(cerr, debug_rep(fcnName, code.num(),
						  otherData, "otherData", item));	
	```
	* 阿 `debug_rep` 只會吃一個參數，所以就錯惹
### 16.4.3 Forwarding Parameter Packs
* 太棒了... 又是 forward，這次還要你 forward pack...
* 其實 `vector::emplace_back` 就是這樣實作的 LOL
* 這張要你重回很久以前寫的 `StrVec`，要為他增加一個 `emplace_back`
	* `emplace_back` 是 STL container 內，為了直接在 container 管理的空間中，利用傳給 `emplace_back` 的參數，原地 construct elements 的一個 member。他是 variadic template
		* 爆幹繞口令
	* 注意，而我們這個 `StrVec` container 只有存 `std::string` 一種型別，但是因為 `string` 的 ctor 有很多種，參數個數也不同，所以還是要把 `StrVec` 的 `emplace_back` 寫成 variadic template
	* **然後我們希望可以用到 `string` 的 move ctor，這意味著我們需要把傳進來的 rvalue 的型別原封不動的丟給 ctor -> *會用到 `forward`***
	* 為了做到這件事情，`emplace_back` 的 function parameters 就要是 rvalue reference to template parameter；這之前在講 reference collapsing 時講過
	```c++
	class StrVec {
	public:
		template <class... Args> void emplace_back(Args&&...);
		// remaining members as in § 13.5 (p. 526)
	};
	```
	* 注意看上面用到的 template parameter pack，他用到了 `&&`，代表展開之後的每個 type 是 rvalue reference to its corresponding argument
	* 然後傳給我們 `private` member `alloc` 這個 allocator 的 `construct`，**傳入 arguments 時就要使用 `forward` 保持原本的型別**:
	```c++
	template <class... Args>
	inline
	void StrVec::emplace_back(Args&&... args) {
		chk_n_alloc(); // reallocates the StrVec ifnecessary 
		alloc.construct(first_free++, std::forward<Args>(args)...);
	}
	```
	* 看上面那個奇葩的 `std::forward<Args>(args)...`，同時把 template parameter pack 跟 function parameter pack 都用到了...
	* 然後如果你之前沒想到的話，你仔細想，`construct` 其實也是 variadic template...
	* 總之這樣寫 expansion 的話會展開成 `std::forward<Ti>(ti)`，`Ti` 就是傳給 `emplace_back` 的第 i 個 argument 的型別，`ti` 則是第 i 個 argument 本身
	* 看 code...:
	```c++
	svec.emplace_back(10, ’c’); // adds ccccccccccas a new last element
	```
	* 則丟給 `construct` 的 expansion 會被展開成:
	```c++
	std::forward<int>(10), std::forward<char>(c)
	```
* 總之用了 `forward` 之後我們可以保證，如果原本傳給 `emplace_back` 的 value 是 rvalue，則傳給 `construct` 的 value 一樣是 rvalue:
	```c++
	svec.emplace_back(s1 + s2); // uses the move constructor
	```
	* 丟給 `construct` 時:
	```c++
	std::forward<string>(string("the end"))
	```
		* 那個 `"the end"` 應該只是隨便亂填的...
		* `forward` 腳括號內為什麼是 `string` 如果忘了，請去看介紹 `forward` 的章節，總之這個情況傳給 `construct` 的 value 型別為 rvalue reference；所以最後會由 `constructor` 呼叫 `string` 的 move ctor
		* 其實要自己弄一個 `construct` 直接呼叫對應的 ctor 也不是不行，既然有了 variadic template 跟 forward，你就寫個什麼 `T(std::forward<Args>(args)...)` 之類的...
#### ADVICE: FORWARDING AND VARIADIC TEMPLATES
* 基本上 variadic templates 很常會跟 `forward` 一起使用，其 pattern 跟我們寫的 `emplace_back` 很像:
	```c++
	// fun has zero or more parameters each of which is 
	// an rvalue reference to a template parameter type 
	template<typename... Args>
	void fun(Args&&... args) // expands Args as a list of rvalue references 
	{
		// the argument to work expands both Args and args 
		work(std::forward<Args>(args)...);
	}
	```
	* 跟在 `construct` 內一樣，我們在 `work` 也會同時展開 `Args` 跟 `args`(function/template parameter packs)
	* 跟 `emplace_back` 也一樣，讓參數是 rvalue reference，可以傳入任何 type；用 forward，則可以維持原本 argument 的型別

## 16.5 Template Specializations
* template 定義的 code 有時候不可能滿足所有 type，甚至根本不能編譯或者行為是錯的
* 這時候我們就可以根據某個 type 的特定性為它寫一個特別的版本，這樣行為才會正確或更有效率
* 例如我們的 `compare`，完全不能比較 character pointers
	* 我們希望比較 char ptr 時是呼叫(lib已經寫好且行為正確的) `strcmp`，而不是用原本 template 定義的直接比較指標
* 實際上 16.1.1 已經寫了一個有點像的東西，還記得的話:
	```c++
	// first version; can compare any two types 
	template <typename T> int compare(const T&, const T&); // second version to handle string literals 
	template<size_t N, size_t M>
	int compare(const char (&)[N], const char (&)[M]);
	```
	* 但是上面有兩個 nontype parameter 的版本只能處理 string literal 或者 char array，
	* 如果你傳 `char*`，一樣是第一個版本會被呼叫:
	```C++
	const char *p1 = "hi", *p2 = "mom";
	compare(p1, p2); // calls the first template
	compare("hi", "mom"); // calls the template with two nontype parameters
	```
	* 第二個版本 non viable 因為不可能把 `char*` 轉成 reference to array
* 我們可以定義對第一個版本的 template 定義 **template specialization** 來解決這個問題，
	* A specialization is a separate definition of the template **in which one or more template parameters are specified to have particular types.**
* 詳見下面
#### Defining a Function Template Specialization
* 格式長這樣:
	```c++
	// special version of compare to handle pointers to character arrays 
	template <> 
	int compare(const char* const &p1, const char* const &p2) {
		return strcmp(p1, p2);
	}
	```
	* 用 `template<>` 來說這是 specialization，然後 template type parameter 全部都要給定
* 注意上面的 specialization 的 parameter type 很ㄎㄧㄤ，為什麼要這樣寫得先從原本的 template 講起:
	```c++
	template <typename T> int compare(const T&, const T&);
	```
	* 首先我們的 specialization，除了 T 以外，其他型別都要一致，包含 `const` 阿，reference 阿等等
	* 而這邊原本的 template 它是吃一個 **reference to 「`const` T」**
	* 我們這邊是希望寫一個 T 為 `const char*` 的 specialization
	* 所以我們的 specialization 要宣告為 **reference to 「`const` "`const` char\*"」**也就是指標本身的 top-level 也要是 `const` 啦，reference to `const` pointer to `const` char
#### Function Overloading versus Template Specializations
* 本質上定義一個 template specialization 是在搶 compiler 的工作(因為它做的不會照你的期望)
* **It is important to realize that *a specialization is an instantiation*; it is not an overloaded instance of the function name.**
* Specializations instantiate a template; they do not overload it. As a result, **specializations do not affect function matching.**
* 所以你把上面的 `compare` 寫成一個 template specialization 跟寫成一個 nontemplate ordinary function 的效果是不一樣的:
	* 一個會影響 function matching 一個不會
	* 假設原本有這兩種:
	```c++
	// first version; can compare any two types 
	template <typename T> int compare(const T&, const T&); // second version to handle string literals 
	template<size_t N, size_t M>
	int compare(const char (&)[N], const char (&)[M]);
	```
	* 然後你定義了第一個的 specialization，就如同上面定義的那個參數很噁心的版本
	* 當你寫
	```c++
	compare("hi", "mom")
	```
	* 兩個都是 exact match，但是因為 template nontype parameter 的版本更 specialized，所以 compiler 會選它，這時你定義的 specialization 完全不會被用到
	* 但如果你不要定義成 specialization，定義成 ordinary function
	* 則這時會有三個 viable function，一個就是你新定義的一般 function，另外兩個是 template 生的；而之前也學到，這種情況下 compiler 會選擇一般 function。

#### KEY CONCEPT: ORDINARY SCOPE RULES APPLY TO SPECIALIZATIONS 
* 要 specialize 某 template 時 template 的宣告必須 in scope
* **要使用這個 specialization 時這個 specialization 的宣告也要 in scope!!** 
* 如果你在使用某特定的 instantiation 時，對應的 specialization 還沒 in scope，compiler 就會幫你生一個它自己的版本，這樣就用不到你的 specialization 了，這樣(預期)行為就會出錯
* 而且這種 error 很難找
* Best Practice: Templates and their specializations should be **declared in the same header file.** **Declarations** for all the templates with a given name **should appear first, followed by any specializations of those templates.**
#### Class Template Specializations
* 當然可以 specialize class template...
* Primer 用 `hash` 舉例...
* 還記得 unordered container 嗎...
* 要用 `hash<key_type>`
* 如果我們要在 unorder 容器放 `Sales_data` 之類的沒有支援 `hash` 的 type，但又想要容器用預設的 `hash<key_type>` 我們就要 specialize 一個特別的版本...
* 要定義下列 member:
	* `operator()`，吃 container 的 `key_type`，return `size_t`
	* 兩個 type member: `result_type` 跟 `argument_type`，分別就是 call operator 的 return type 跟 argument type
	* default ctor 跟 copy assignment(可以讓 compiler 合成)
* 要對 class/function template 定義 specialization 有一個前提...，你必須在跟它同樣 scope 的地方定義 specialization!
* 但是 `hash` 是定義在 `std` 這個 namespace 內的阿? 我們要怎麼在這裏面定義 specialization 呢?
* 18.2 章會講到，我們現在只要知道如下操作就可:
	* 我們需要"開啟"(open) namespace:
	```c++
	// open the std namespace so we can specialize std::hash 
	namespace std {
	}	// close the std namespace; note: no semicolon after the close curly
	```
* 這樣你在 {} 內定義的 name 就會屬於 `std` 這個 namespace 了
* 下面的 code 會寫一個針對 `Sales_data` 的 specialization:
	```c++
	// open the std namespace so we can specialize std::hash 
	namespace std { 
	template <> 						// we’re defining a specialization with
	struct hash<Sales_data> // the template parameter ofSales_data 
	{
		// the type used to hash an unordered container must define these types 
		typedef size_t result_type; 
		typedef Sales_data argument_type; // by default, this type needs == 
		size_t operator()(const Sales_data& s) const; 
		// our class uses synthesized copy control and default constructor
	}; 
	size_t
	hash<Sales_data>::operator()(const Sales_data& s) const 
	{
		return hash<string>()(s.bookNo) ^ 
			   hash<unsigned>()(s.units_sold) ^ 
			   hash<double>()(s.revenue);
	}
	}// close the stdnamespace; note: no semicolon after the close curly
	``` 
* 我們對 `hash` 這個 template 定義一個 specialization，`hash<Sales_data>`
* 它在幹嘛不重要啦...反正你他媽又不會 hash，總之它避免掉了寫爛 hash 的可能，直接使用 library 定義好的 `hash<string/unsigned/double>` 來算 hash code
* 總之寫了這個以後，我們就可以寫下面的 code 了:	
	```c++
	// uses hash<Sales_data>and Sales_dataoperator== from § 14.3.1 (p. 561) 
	unordered_multiset<Sales_data> SDset;
	```
* 阿還要在 `Sales_data` 內宣告 `hash<Sales_data>` 為 friend，因為它會用到 private member
	```c++
	template <class T> class std::hash; // needed for the friend declaration 
	class Sales_data { 
		friend class std::hash<Sales_data>; 
		// other members as before
	};
	```
* 還有，整個 specialization 要定義在 `Sales_data` 的 header 內，這樣 compiler 才看的到...
#### Class-Template Partial Specializations
* class template specialization 不一定要全部 temp arg 都提供，或者"只提供某些 arg 的某些特性"(const , l/r reference 之類的)，這叫做 partial specializations
* A class template **partial specialization is itself a template.**
	* Users must supply arguments for those template parameters that are not fixed by the specialization.
* 再強調一次，只有 class template 可以 partial，function template 不行
* 例如 std 的 `remove_reference`，其實有很多種 partial specializations:
	```c++
	// original, most general template 
	template <class T> struct remove_reference {
		typedef T type;
	}; 
	// partial specializations that will be used for lvalue and rvalue references 
	template <class T> struct remove_reference<T&> // lvalue references 
		{ typedef T type; };
	template <class T> struct remove_reference<T&&> // rvalue references
		{ typedef T type; };
	```
	* 第一個 template 最 general，可以實例化任何 type；後面兩個是 partial specializations
* 這裡真的不知道在銃三小...
* 看 code...:
	```c++
	int i; 
	// decltype(42) is int, uses the original template 
	remove_reference<decltype(42)>::type a; 
	// decltype(i) is int&, uses first (T&) partial specialization 
	remove_reference<decltype(i)>::type b; 
	// decltype(std::move(i)) is int&&, uses second (i.e., T&&) partial specialization
	remove_reference<decltype(std::move(i))>::type c;
	```
#### Specializing Members but Not the Class
* 我們可以只 spectialize 某個 member:
	```c++
	template <typename T> struct Foo {
		Foo(const T &t = T()): mem(t) { } 
		void Bar() { /* .. . */} 
		T mem; 
		// other members of Foo
	};
	template<> // we’re specializing a template
	void Foo<int>::Bar() // we’re specializing the Bar member of Foo<int>
	{ 
		// do whatever specialized processing that applies to ints
	}
	```
	* 我們這樣寫的話，當我們真的定義了 `Foo<int>` 且使用 `Bar` 時，compiler 就不會實例化一個，而是用我們定義的:
	```c++
	Foo<string> fs; // instantiates Foo<string>::Foo() 
	fs.Bar(); 		// instantiates Foo<string>::Bar()
	Foo<int> fi; 	// instantiates Foo<int>::Foo() 
	fi.Bar(); 		// uses our specialization ofFoo<int>::Bar()
	```
	* 最後一行 code 會用我們定義的 `Foo<int>::Bar()`


