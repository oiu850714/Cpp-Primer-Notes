---
tags: C++
---

# C++ Primer Chapter 12 Dynamic Memory
## 可參照影片: https://www.youtube.com/watch?v=xGDLkt-jBJ4
* C++ lets us allocate objects dynamically.
* Dynamically allocated objects have a lifetime that is independent of where they are created; they exist until they are explicitly freed.
    * "where they are created": 變數定義的位置會決定他是 global, local or static
        * 並且有對應的 lifetime
    * 而動態配置的物件的 lifetime 要由 programmer 自己 handle

## 12.1 Dynamic Memory and Smart Pointers
* In C++, dynamic memory is managed through a pair of operators:
    * `new`, which allocates, and optionally initializes, an object in dynamic memory and returns a pointer to that object;
    * and `delete`, which takes a pointer to a dynamic object, destroys that object, and frees the associated memory.

* **It is surprisingly hard to ensure that we free memory at the right time**
    * Either we **forget to free the memory**—in which case we have a **memory leak**—or we **free the memory when there are still pointers referring to that memory**—in which case we have a pointer that refers to memory that is no longer valid.

* C++11 定義了兩種 smart pointer types 來把 raw pointer 包起來
    * A smart pointer acts like a regular pointer with the important exception that **it automatically deletes the object to which it points.**
    * **It takes the responsibility to delete the object when appropriate**
* C++11 defines two kinds of smart pointers that *differ in how they manage their underlying pointers*:
    * `st::shared_ptr`, which allows multiple pointers to refer to the same object
    * `std::unique_ptr`, which "owns" the object to which it points.
    * The library also defines a *companion class* named `std::weak_ptr` that is a weak reference to an object managed by a `shared_ptr`.
* 上面三個都定義在 `<memory>` 裡面


### 12.1.1 The `std::shared_ptr<T>` Class
* class template
* template argument 給的就是 `std::shared_ptr` 指向的 type
    ```cpp
    shared_ptr<string> p1; // shared_ptr that can point at a string
    shared_ptr<list<int>> p2; // shared_ptr that can point at a list of ints
    ```
* smart ptr 的 default initialize 會指向 `nullptr`
* smart pointer 也定義好 overloading，讓你可以用操作一般 ptr 的方式操作，可參考 table 12.1 12.2
    ```cpp
    // if p1 is not null, check whether it’s the empty string
    if (p1 && p1->empty())
        *p1 = "hi"; // if so, dereference p1 to assign a new value to that string
    ```
    * 上面也是用 short circuit 來判斷 `p1` 是否指向物件，再判斷指向的字串是否為空
    * smart ptr 被當成 condition 就是判斷他是否為 `nullptr`

* table 12.1 給定 `shared_ptr` 跟 `unique_ptr` 共同的 interface
* ![](https://i.imgur.com/4QTCcBq.png)
    * `p.get()` 可能是串接 C API 需要用到的
    * 那個 `swap` 則是感覺怕爆
* table 12.2 是 `shared_ptr` 獨有的
* ![](https://i.imgur.com/cgXbr8f.png)
    * convertible to `T*`
        * array decay
        * `void*`
        * 還有 15 章
            * 繼承

#### The `std::make_shared<T>` Function
* The safest way to allocate and use dynamic memory is to call a library function named `std::make_shared`. 
* This function **allocates and initializes an object in dynamic memory and *returns a `shared_ptr<T>`*** that points to that object.
    * 他其實是 function template
    * 一樣在 `<memory>`
    * call 的時候要指定 object type
        * 有時候 function template 不用指定，但是去看 `make_shared` 的文件就會明白為何呼叫他一定要指定了
        * https://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared
            * 實際上有兩個 template argument
            * 第一個，也就是 `make_shared<T>` 的 `T`，是拿來用在 return type 的
            * 第二個則是 variadic template，用在傳給 `maked_share` 的 function argument
                * 是丟給 `T` 的 ctor 的 arguments
    ```cpp
    // shared_ptr that points to an int with value 42
    shared_ptr<int> p3 = make_shared<int>(42);
    // p4 points to a string with value 9999999999 
    shared_ptr<string> p4 = make_shared<string>(10, '9');
    // p5 points to an int that is value initialized (§ 3.3.1 (p. 98)) to 0
    shared_ptr<int> p5 = make_shared<int>();
    ```
* 注意 `make_shared` 的 API，長得跟 sequential container 的 `emplace` 很像，**裡面放的參數其實是物件的 ctor 的參數**
* 上面三個宣告的 `shared_ptr` 當然可以用 auto
    ```cpp
    // p6 points to a dynamically allocated, empty vector<string>
    auto p6 = make_shared<vector<string>>()
    ```

#### Copying and Assigning `shared_ptr`s
* When we copy or assign a `shared_ptr`, **each `shared_ptr` keeps track of how many other shared_ptrs point to the same object:**
    ```cpp
    auto p = make_shared<int>(42);
    // object to which p points has one user
    auto q(p); // p and q point to the same object
    // object to which p and q point has two users
    ```
* We can think of a `shared_ptr` **as if it has an associated counter,** usually referred to ***as a reference count.***
    * 當我們 copy `std::shared_ptr`，counter + 1
    * 這個 copy 亦即各種 context 之下物件被 copy 的那個 copy，例如
        * `auto q(p);`
        * `fun(p); // 如果 p passed by value`
        * `q = p;`
    * 當我們 assign 一個新 value 給 `shared_ptr` 物件，那個 ptr 原本指向的物件對應的 reference count 就會少 1，而新的 ptr value 指像的物件對應的 count 會加 1
    * 當 `shared_ptr` 被 destroy 也會少 1
        * 例如 out-of-scope
* 其實嚴格說起來，ref count 是 associated with pointed object，而不是 `shared_ptr`

* 當 `shared_ptr` 指向的物件的 ref count 變成 0，那個物件會自動被 delete 掉
    ```cpp
    auto r = make_shared<int>(42);
    // int to which r points has one user
    r = q;
    // assign to r, making it point to a different address
    // increase the use count for the object to which q points
    // reduce the use count of the object to which r had pointed
    // the object r had pointed to has no users; that object is automatically freed
    ```
* Note: It is *up to the implementation* whether to use a counter or another data structure to keep track of how many pointers share state.
    * library 到底用什麼資料結構來記住什麼物件被幾個 `shared_ptr` 指著其實你不用管，你只要知道 library 會處這件事情就好了，而且當那個物件沒人指時物件就會被 free 掉。
    * 但你看了 back to basics 了，你知道就是怎樣實作

#### `shared_ptr`s Automatically Destroy Their Objects ...

* The **destructor** for `shared_ptr` **decrements the reference count** of the object to which that `shared_ptr` points.
* If the **count goes to zero**, the `shared_ptr` destructor **destroys the object** to which the `shared_ptr` points and frees the memory used by that object.

#### ...and Automatically Free the Associated Memory
* The fact that the `shared_ptr` class automatically frees dynamic objects when they are no longer needed makes it fairly easy to use dynamic memory.
* 看看下面的範例
    ```cpp
    // factory returns a shared_ptr pointing to a dynamically allocated object 
    shared_ptr<Foo> factory(T arg) {
        // process argas appropriate 
        // shared_ptr will take care of deleting this memory 
        return make_shared<Foo>(arg);
    }
    ```
* The following function stores the `shared_ptr` returned by factory in a local variable:
    ```cpp
    void use_factory(T arg) {
        shared_ptr<Foo> p = factory(arg); // use p
    }
    // p goes out of scope; the memory to which p points is automatically freed
    ```
    * When `p` is destroyed, its reference count is decremented and checked. In this case, `p` is the only object referring to the memory returned by `factory`.
        * Because `p` is about to go away, the object to which `p` points will be destroyed and the memory in which that object resides will be freed.
* 可是你如果不想要物件在 `use_factory` 就被刪掉，你可以把 `p` return
    ```cpp
    shared_ptr<Foo> use_factory(T arg) {
        shared_ptr<Foo> p = factory(arg);
        // use p
        return p; // reference count is incremented when we return p
    }// p goes out of scope; the memory to which p points is not freed
    ```
    * ref count 先加再減
* 感覺要多看一點 case 或 garbage collection 的東西才知道要怎麼用 `shared_ptr` 比較好
* Because memory is not freed until the last `shared_ptr` goes away, **it can be important to be sure that `shared_ptr`s don’t stay around after they are no longer needed.**
    * 如果你明明某個 `shared_ptr` 已經不需要了卻沒有把他殺掉，那物件就不會被釋放
        * 例子是你把 `shared_ptr` 放到 container
        * 當你對 container 做一些操作，然後發現某些 shared_ptr 不需要了，**你必須明確的用 erase** 把他們消除，否則他們指向的物件就不會被釋放
        * 例如對 container 做 `std::unique`
            * 當然這時候不是比較 `shared_ptr` 啦，你應該會傳一個 binary predicate 進去 `std::unique`
        * 則 sequence 後面的垃圾應該用 `erase` 清掉


#### Classes with Resources That Have Dynamic Lifetime

* Programs tend to use dynamic memory for one of three purposes:
    1. They don’t know how many objects they’ll need
        * compile time 時無法知道
    3. They don’t know the precise type of the objects they need
        * 15 章繼承會講到
    5. They want to share data between several objects
        * 本質上這就是 `shared_ptr` 在幹的事
* 第一點的例子是 STL container
* 第二點的例子是繼承(15 章)
* 第三個例子接下來要給
    * In this section, we’ll define a class that *uses dynamic memory in order to let several objects share the same underlying data.*

* 到目前為止我們宣告的物件，其使用的資源的生命週期都跟該物件一致
    * 例如 `vector<int>` 裡面的 int 用的資源都跟 vector 共生死
    * 到目前的 code 都只用這種

* 不過有種寫法是，**宣告一個物件之後他要了一塊資源，而這個資源的生命週期跟這個物件的生命週期是不相關的**
* 假設我們想要訂一個 class `Blob`，他的行為是這樣
    * Blob object 會存一堆元素的集合(a collection of elements)
    * 當我們 copy `Blob` object 時，我們希望新複製的 object 內存的元素跟原本的是同一組(same underlying elements)
        * **share the same elements**
        * **refer to the same underlying elements.**

* 根據上面的行為有一個條件必須滿足，**當兩個物件共享底層的元素時，如果其中一個物件被 destroy，該物件不可以把底層元素刪除**
    ```cpp
    Blob<string> b1; // empty Blob
    {
        // new scope
        Blob<string> b2 = {"a", "an", "the"};
        b1 = b2; // b1 and b2 share the same elements
    }// b2 is destroyed, but the elements in b2 must not be destroyed
    // b1 points to the elements originally created in b2
    ```
    * When `b2` goes out of scope, **those elements must stay around, because `b1` is still using them.**

#### Defining the `StrBlob` Class
* Primer 寫 `Blob` 這最終會實作成 template，但還沒講，所以先用個只存 `std::string` 的版本
* The easiest way to implement a new collection type is to use one of the library containers to manage the elements.
    * 比方說 `Blob` 內放一個 `vector<string>`
* However, **we can’t store the vector directly in a Blob object.**
    * 這樣做，`Blob` object 死掉了，這個 vector 也會消失
    * 這樣沒辦法做到所謂的兩個 `Blob` object 共享 underlying object
    * **我們必須把 `vector<string>` 存在 dynamic memory 裡面**
* To implement the sharing we want, we’ll **give each `StrBlob` a `shared_ptr` to a dynamically allocated vector.**
* **That `shared_ptr` member will keep track of how many `StrBlob`s share the same vector** and will delete the vector when the last `StrBlob` using that vector is destroyed.
    * 基本上直接用 `shared_ptr` 處理"追蹤還有多少 `StrBlob` 指向這個 vector" 的責任
    ```cpp
    class StrBlob {
    public:
        typedef std::vector<std::string>::size_type size_type;
        StrBlob();
        StrBlob(std::initializer_list<std::string> il);
        size_type size() const { return data->size(); }
        bool empty() const { return data->empty(); }
        // add and remove elements
        void push_back(const std::string &t) {data->push_back(t);}
        void pop_back(); // element access
        std::string& front();     
        std::string& back();
    private:
        std::shared_ptr<std::vector<std::string>> data;
        // throws msg if data[i] isn’t valid
        void check(size_type i, const std::string &msg) const;
    };
    ```
    * 其實看一下 interfece，就像是簡化版的存著 string 的 vector
    * **但是這個 class 的 object，一旦用 assignment 之後，i.e. a = b;，a b 兩個 object 的 underlying data 會是相同的(不是單純複製而已)**
    * 注意 private data 是一個 `shared_ptr` to `vector<string>`
        
#### StrBlob Constructors
* 下面是 ctor，有兩種，一種 default，另一種吃 initializer_list
    ```cpp
    StrBlob::StrBlob(): data(make_shared<vector<string>>()) { } 
    StrBlob::StrBlob(initializer_list<string> il):
            data(make_shared<vector<string>>(il)) { }
    ```
    * 別忘記 `make_shared` 吃的參數是指向物件的 type 的 ctor 的參數，而這裡的 type 是 `vector<string>`
    * 第一個會 allocate 一個 empty vector，第二個會 allocate 一個 vector 裡面塞 il 給的字串

#### Element Access Members
* `pop_back`, `front`, and `back`
* 這些 member 在 acces 底層 vector 時都要先確認 element 存在，而這個邏輯直接抽出成一個 private utility `check`
```cpp
void StrBlob::check(size_type i, const string &msg) const {
    if (i >= data->size()) throw out_of_range(msg);
}
```
* 首先 `check` 第一個參數就是檢查 access 的 index 有沒有超過 vector 邊界
* 但第二個其實是要丟給 exception 當參數用的，是 error message
* 阿三個 public member functions 就給不同的參數給 check 以做確認
```cpp
string& StrBlob::front() {
// if the vector is empty, check will throw
    check(0, "front on empty StrBlob");
    return data->front();
}
string& StrBlob::back() {
    check(0, "back on empty StrBlob");
    return data->back();
}
void StrBlob::pop_back() {
    check(0, "pop_back on empty StrBlob");
    data->pop_back();
}
```
* 上面兩個 access member 跟 `pop_back` 都要在 vector 不為空的情況才能操作，所以傳給 `check` 的 index 都為 `0`(至少有一個 element)
* 然後 `front` 跟 `back` 應該還要有一個 `const` 的 overload 版本，Primer 說這丟到練習 XD

#### Copying, Assigning, and Destroying `StrBlob`s
* 真正會用到 `shared_ptr` 用處的地方來惹
* Our `StrBlob` has only one data member, which is a `shared_ptr`.
* When we copy, assign, or destroy a `StrBlob`,its `shared_ptr` member will be copied, assigned, or destroyed.
    * As we’ve seen
        * copying a `shared_ptr` increments its reference count;
        * assigning one `shared_ptr` to another increments the count of the right-hand operand and decrements the count in the left-hand operand;
        * and destroying a `shared_ptr` decrements the count.
    * 再複習一次 `shared_ptr` 的行為
* If the count in a `shared_ptr` goes to zero, the object to which that `shared_ptr` points is automatically destroyed.
    * 所以這時候這個 `shared_ptr` 指向的 `vector` 也會被 free 掉!

### 12.1.2 Managing Memory Directly
* 要你直接使用 `new` 跟 `delete` 惹!
* classes that do manage their own memory— unlike those that use smart pointers—**cannot rely on the default (synthesized) definitions for the members that copy, assign, and destroy class objects**
* 13 章才會講怎麼自己實作上面的行為，所以現在先假設不會在 class 內用 `new` 跟 `delete`(但可以用 smart pointer)

#### Using `new` to Dynamically Allocate and Initialize Objects
* `new` returns a pointer to the object it allocates:
    ```cpp
    int *pi = new int; // pi points to a dynamically allocated,
                       // unnamed, uninitialized int
    ```
* This `new` expression **constructs an object of type `int` on the *free store*** and returns a pointer to that object.
* heap 上的物件預設是 default initailized，所以 built-in value 是 undefined，如果是 class 就用 default ctor
    ```cpp
    string *ps = new string; // initialized to empty string
    int *pi = new int;
    // pi points to an uninitialized int
    ```
* We can initialize a dynamically allocated object using direct initialization (§ 3.2.1, p. 84).
    * 就是在 new 的 type 後面加上給 ctor 的參數
    ```cpp
    int *pi = new int(1024); // object to which pi points has value 1024 
    string *ps = new string(10, '9');
    // *ps is "9999999999"
    // vector with ten elements with values from 0 to 9
    vector<int> *pv = new vector<int>{0,1,2,3,4,5,6,7,8,9};
    ```
    * 最後一個例子就 list initialization，new 的時候一樣可以用
* We **can also value initialize** (§ 3.3.1, p. 98) a dynamically allocated object **by following the type name with a pair of empty parentheses:**
    * 在 type name `T` 後面加上括號就可以 value-initialize
    * 這對 class type 沒差，都 call default cotr
    * 但對 built-in type 差很多，會 initialize 成 0
    ```cpp
    string *ps1 = new string; // default initialized to the empty string
    string *ps = new string(); // value initialized to the empty string
    int *pi1 = new int; // default initialized; *pi1 is undefined
    int *pi2 = new int(); // value initialized to 0; *pi2 is 0
    ```
    * 注意 `pi1` 跟 `pi2` 差很多

:::info
* For the same reasons as we usually initialize variables, **it is also a good idea to initialize dynamically allocated objects**
* 初始化就是好事，**我們用 new 的時候最好也想辦法初始化我們的物件**
:::

* C++11 還可以讓你用 auto，但是是 `new auto`
    * 而且不能用 initializer list
    ```cpp
    auto p1 = new auto(obj); // p points to an object ofthe type of obj
                             // that object is initialized from obj
    auto p2 = new auto{a,b,c}; // error: must use parentheses for the initializer
    ```
    * The newly allocated object is *initialized from the value of obj*

#### Dynamically Allocated `const` Objects
* 當然可以 new 一個 const object
    ```cpp
    // allocate and initialize a const int
    const int *pci = new const int(1024);
    // allocate a default-initialized constempty string
    const string *pcs = new const string;
    ```
    * 一樣一定要初始化
    * **pointer returned by new is pointer to `const`**

#### Memory Exhaustion
* By default, if `new` is unable to allocate the requested storage, it throws an **exception of type `bad_alloc`** (§ 5.6, p. 193).
* 但可以像下面那樣寫防止 `new` `throw`
    ```cpp
    // if allocation fails, new returns a null pointer
    int *p1 = new int; // if allocation fails, new throws std::bad_alloc
    int *p2 = new (nothrow) int; // if allocation fails, new returns a null pointer
    ```
    * this form of new is referred to as **placement `new`.**
    * A placement `new` expression lets us **pass additional arguments** to `new`. In this case, we pass an ***object*** named `nothrow` that is defined by the library. 
* `bad_alloc` 跟 `nothtow` 都定義在 `<new>`

#### Freeing Dynamic Memory
```cpp
delete p; // p must point to a dynamically allocated object or be null
```
* Like `new`, a `delete` expression performs two actions:
    * It destroys the object to which its given pointer points,
    * and it frees the corresponding memory.
#### Pointer Values and `delete`
* **The pointer we pass to `delete` must either point to dynamically allocated memory or be a `nullptr`**
    * `delete` 其他 pointer 或者對 new 出來的 object `delete` N 次都是 UB
    ```cpp
    int i, *pi1 = &i, *pi2 = nullptr;
    double *pd = new double(33), *pd2 = pd;
    delete i; // error: i is not a pointer
    delete pi1; // undefined: pi1 refers to a local
    delete pd; // ok
    delete pd2; // undefined: the memory pointed to by pd2 was already freed
    delete pi2; // ok: it is always ok to delete a null pointer
    ```
* `const` object 當然也可以被 delete
    ```cpp
    const int *pci = new const int(1024);
    delete pci; // ok: deletes a const object
    ```
    
#### **Dynamically Allocated Objects Exist until They Are Freed**
* A dynamic object managed through a built-in pointer exists until it is explicitly deleted.
* Functions that return pointers (rather than smart pointers) to dynamic memory **put a burden on their callers**—
    * ***the caller must remember to delete the memory:***
    ```cpp
    // factory returns a pointer to a dynamically allocated object
    Foo* factory(T arg) {
        // process arg as appropriate
        return new Foo(arg); 
        // caller is responsible for deleting this memory
    }
    ```
    
* **Unfortunately, all too often the caller forgets to do so:**
    ```cpp
    void use_factory(T arg) {
        Foo *p = factory(arg);
        // use p but do not delete it
    }// p goes out of scope, but the memory to which p points is not freed!
    ```
    * In particular, when a pointer goes out of scope, **nothing happens to the object to which the pointer points.**
    * Once `use_factory` returns, **the program has no way to free that memory.**
    * 至少要改成這樣這 program 才不會 memory leak
    ```cpp
    void use_factory(T arg) {
        Foo *p = factory(arg);
        // use p
        delete p; // remember to free the memory now that we no longer need it
    }
    ```
    * 或者把指標的 value 傳回去
    ```cpp
    Foo* use_factory(T arg) {
        Foo *p = factory(arg);
        // use p
        return p; // caller must delete the memory
    }
    ```
    
#### Resetting the Value of a Pointer after a delete ...
#### ...Provides Only Limited Protection
* dangling pointers
    * After the delete, the pointer becomes what is referred to as a **dangling pointer.**
        * A dangling pointer is one that **refers to memory that once held an object but no longer does so.**
        * dangling ptr 就是指向一個已經被 delete 的物件的指標
    * dangling ptr 跟未初始化的 ptr 一樣，由他來 access 物件全部都是 UB
    * 一個簡單的小解法是，在 ptr 要 out of scope(也就是要被 destroy) 之前 delete object，這樣就不會有使用到 dangling ptr 的問題，因為一旦 delete 之後 ptr 就消失了
    * 如果不希望 ptr 被刪掉，那可以在 delete 之後讓 ptr = nullptr，然後要使用 ptr 之前都確定他是否為 NULL
* 但是
    * **there can be several pointers that point to the same memory.**
    * N 個指標指向同一塊 heap oject，只要有一個 delete object，其他 ptr 全部都會變成 dangling ptr
    ```cpp
    int *p(new int(42)); // p points to dynamic memory
    auto q = p; // p and q point to the same memory
    delete p; // invalidates both pand q
    p = nullptr; // indicates that p is no longer bound to an object
    ```
* In real systems, finding all the pointers that point to the same memory is surprisingly difficult.

### 12.1.3 Using `shared_ptr`s with new
![](https://i.imgur.com/UnAKOjR.png)
* 用 `unique_ptr` 初始化 `shared_ptr` 是 move semantics
* deleter 可以動態改變，酷
* We can also initialize a smart pointer from a (raw) pointer returned by `new`:
    ```cpp
    shared_ptr<double> p1; // shared_ptr that can point at a double 
    shared_ptr<int> p2(new int(42)); // p2 points to an int with value 42
    ```
    * The smart pointer constructors that take pointers are **explicit**
        * makes the ***intention*** of **"delegating the responsibility of memory management to smart pointer"** explicitly
        * must use the direct form of initialization
    ```cpp
    shared_ptr<int> p1 = new int(1024); // error: must use direct initialization
    shared_ptr<int> p2(new int(1024)); // ok: uses direct initialization
    ```
    * For the same reason, a function that returns a `shared_ptr` cannot implicitly convert a plain pointer in its return statement:
    ```cpp
    shared_ptr<int> clone(int p) {
        return new int(p); // error: implicit conversion to shared_ptr<int>
    }
    shared_ptr<int> clone(int p) {
        // ok: explicitly create a shared_ptr<int>from int*
        return shared_ptr<int>(new int(p));
    }
    ```
    * Practice: 能用 `std::make_shared` 就用，除非既有 API 是 raw pointer，或者要串接 C API

#### Don’t Mix Ordinary Pointers and Smart Pointers ...
* 當一個物件已經被 `shared_ptr` 指向，就不要再用任何的 raw ptr 來指向那個物件了
    * 你不知道那塊物件什麼時候被 shared_ptr 釋放
    * 請都用 `shared_ptr` 做管理
#### ...and Don’t Use `get` to Initialize or Assign Another Smart Pointer
* The code that uses the return from `get` must not delete that pointer.
    * 可能是接 C API 用的
    * 其實這句話也包含了不要把 `get` 回傳的 raw ptr 丟給 `shared_ptr`，因為 `shared_ptr` 就是屬於會對 ptr 呼叫 `delete` 的情境
* 絕不要對 `shared_ptr.get()` 回傳的 raw pointer 做 `delete`，這樣會讓 `shared_ptr` 維護的狀態(ref count阿, 他指向的物件阿之類的)直接爛掉
* 而且不要用 `shared_ptr.get()` 回傳的 ptr 來初始化一個新的 `shared_ptr`，這樣舊的 `shared_ptr`(被 call `get` 的那個) 跟新的 `shared_ptr`(用 `get` 的 raw ptr 初始化的那個)彼此並不相關(independent)
    * 各自的 reference count 都是 1
    * 可是卻指向相同的物件!

#### Other `shared_ptr` Operations
```cpp
p = new int(1024); // error: cannot assign a pointer to a shared_ptr
p.reset(new int(1024)); // ok: p points to a new object
```
* Like assignment, `reset` updates the reference counts and, if appropriate, `delete`s the object to which `p` points.
* The `reset` member is often used together with `unique` to control changes to the object shared among several `shared_ptr`s.
    * 不確定有什麼 pattern 會有這種情境

### 12.1.4 Smart Pointers and Exceptions
* In § 5.6.2 (p. 196) we noted that **programs that use exception handling to continue processing after an exception occurs need to ensure that resources are properly freed if an exception occurs.**
* 用 smart ptr 可以讓這件事情變簡單
    * 也可以用 RAII，但是還是可以搭配 smart ptr
    ```cpp
    void f() {
        shared_ptr<int> sp(new int(42)); // allocate a new object
        // code that throws an exception that is not caught inside f
    }// shared_ptr freed automatically when the function ends
    ```
    * When a function is exited, *whether through normal processing or due to an exception*, all the local objects are destroyed.
        * 管你是正常結束還是噴 exception，local object 都會先消滅掉
        * 所以如果 local object 是 `shared_ptr`，他就會檢查 reference count，如果是 0，就 `delete` object
    * 上面的例子，`sp` 是唯一指向 `int` 物件的 `shared_ptr`，所以把 `sp` 破壞掉之後物件就會被 `delete`

* In contrast, memory that we manage directly(using raw poiner) is not automatically freed when an exception occurs.
    * 如果你只是用 raw pointer 的話，exception 發生在你 `delete` 的邏輯執行之前就 leak 了
    * C++11 之前大概是把 raw pointer 包起來在 destructor 裡面 `delete` 吧

    ```cpp
    void f() {
        int *ip = new int(42); // dynamically allocate a new object
        // code that throws an exception that is not caught inside f
        delete ip;
        // free the memory before exiting
    }
    ```
    * If an exception **happens between the `new` and the `delete`**, and is not caught inside `f`, then this memory can never be freed.
    
#### Smart Pointers and Dumb Classes
* 簡單說你寫了一個像 C strcut 的 class，然後裡面會有 ptr 去 new 東西的時候，他一樣會發生 exception 導致 memory leak 的情況
    * 因為(要同時能給 C 用的) C struct 沒有 dtor
* Primer 舉一個同時要給 C 跟 C++ 用的 network library 當例子
    ```cpp
    struct destination; // represents what we are connecting to
    struct connection; // information needed to use the connection
    connection connect(destination*); // open the connection
    void disconnect(connection);
    void f(destination &d /* other parameters */) {
        // get a connection; must remember to close it when done
        connection c = connect(&d);
        // use the connection
        // if we forget to call disconnect before exiting f, there will be no way to close c
    }
    ```
    * If `connection` had a destructor, that destructor would automatically close the connection when f completes.
    * However, connection does not have a destructor...
    * This problem is nearly identical to our previous program that used a `shared_ptr` to avoid memory leaks.
    * It turns out that we can also use a `shared_ptr` to ensure that the `connection` is properly closed.


#### Using Our Own Deletion Code
* 你可以讓 smart pointer 指向一個不是 `new` 出來的物件
    * 但是必須告訴 smart pointer 當在應該 `delete` 物件時應該改做什麼事情，因為現在不能呼叫 `delete`
    ```cpp
    class_name obj;
    shared_ptr<class_name> ptr(&obj, deleter)
    ```
    * 所以這時必須提供一個 deleter，他是一個 callable object，吃一個 `class_name*` 參數
    * 等到 smart ptr 決定要清除資源的時候，他就不會對底層的 pointer call `delete`，而是把底層 pointer `class_name*` 丟到 deleter，讓 deleter 來清資源。
    * 當然這種 smart ptr 就不能吃 `make_shared` 回傳的物件惹
        * 這該不會是可以動態指定 deleter 的原因?

:::danger
* 最後是用 smart pointer 時的注意事項(convention)
![](https://i.imgur.com/BPJqeEN.png)
:::
    
### 12.1.5 `unique_ptr`
![](https://i.imgur.com/pfQQzEC.png)
* 注意 `u2` 要使用 deleter 時，template argument 要給 deleter type，這跟 `shared_ptr` 不同
    * https://stackoverflow.com/questions/21355037/why-does-unique-ptr-take-two-template-parameters-when-shared-ptr-only-takes-one
* 另外有 `release` 可以呼叫，這時 `unique_ptr` 不會 `delete` 物件，並且概念上把刪除物件的責任還給 caller
    * **一定要 assign `release` 的 return value**

* A `unique_ptr` “owns” the object to which it points. **Unlike `shared_ptr`,only one `unique_ptr` at a time can point to a given object.**
* The object to which a `unique_ptr` points is destroyed when the `unique_ptr` is destroyed.
* ~~首先，沒有 maked_unique 這種東西~~
    * C++14 就有惹，Primer 老ㄌ
    * *所以下面的 code example 如果有用 raw ptr 初始化 `unique_ptr` 的記得要想成是用 make_unique*
    ```cpp
    unique_ptr<double> p1; // unique_ptr that can point at a double 
    unique_ptr<int> p2(new int(42)); // p2 points to int with value 42
    ```
* 阿就說只能有一個 `unique_ptr` 指向某個物件惹，所以 `unique_ptr` 當然不支援 copy 或 assign 阿
    ```cpp
    unique_ptr<string> p1(new string("Stegosaurus"));
    unique_ptr<string> p2(p1); // error: no copy for unique_ptr
    unique_ptr<string> p3;
    p3 = p2; // error: no assign for unique_ptr
    ```
* Although we can’t copy or assign a `unique_ptr`, **we can transfer ownership** from one (nonconst) `unique_ptr` to another by calling `release` or `reset`:
    * 經過 transfer ownership 之後還是只會有一個 `unique_ptr` 指向("owns")物件
    ```cpp
    // transfers ownership from p1(which points to the string "Stegosaurus") to p2
    unique_ptr<string> p2(p1.release()); // release makes p1 null 
    unique_ptr<string> p3(new string("Trex"));
    // transfers ownership from p3 to p2
    p2.reset(p3.release()); // reset deletes the memory to which p2 had pointed
    ```
    * The `release` member returns the pointer currently stored in the `unique_ptr` and makes that `unique_ptr` null. Thus, `p2` is initialized from the pointer value that had been stored in `p1` and `p1` becomes null.
    * **Calling `release` breaks the connection between a `unique_ptr` and the object it had been managing.**
    * **Often the pointer returned by `release` is used to initialize or assign another smart pointer.**
    * 接住 `release` 回傳值的人就要負責 `delete` object，如果是用 smart_ptr 去接就交給 smart_ptr，如果是用 raw ptr，program 就要自己 free
        * 而且一定要 assign `release()` return value，否則 memory leak
        ```cpp
        p2.release(); // WRONG: p2 won’t free the memory and we’ve lost the pointer 
        auto p = p2.release(); // ok, but we must remember to delete(p)
        ```

#### Passing and Returning `unique_ptr`s
* 已經說了 `unique_ptr` 不能 copy，這樣要怎麼傳進參數呢?
    * 錯! 如果你寫成 return `unique_ptr` 是合法的!
    * 這是另一種 copy(move) 形式，13章會講
    ```cpp
    unique_ptr<int> clone(int p) { // ok: explicitly create a unique_ptr<int> from int* 
        return unique_ptr<int>(new int(p));
    }
    
    unique_ptr<int> clone(int p) { 
        unique_ptr<int> ret(new int (p));
        // ...
        return ret;
    }
    ```
    * 另外 function 定義吃 `shared_ptr` 通常定義成吃 value
        * 導致 caller 都要用 `std::move`
        * **make transfer exclusive ownership explicit**
        * 詳情 back to basics 影片
#### Passing a Deleter to `unique_ptr`
* 跟 `shared_ptr` 很類似
* However, for reasons we’ll describe in § 16.1.6 (p. 676), the way `unique_ptr` manages its deleter is differs from the way `shared_ptr` does.
* Overridding the deleter in a `unique_ptr` affects the `unique_ptr` type as well as how we construct (or reset) objects of that type.
    ```cpp
    std::unique_ptr<T, D> up
    ```



### 12.1.6 `weak_ptr`
![](https://i.imgur.com/oBkUlZU.png)

* 根據 back to basics 影片，盡量少用
    * 可以用在你的 ptr 有可能暫時為 dangling 的情境

* 重點：
    * doesn't control the lifetime of the object
    * 指向 `shared_ptr` 指向的物件
    * 不參與/影響 `shared_ptr` ref count 計算
        * 當對應 `shared_ptr` 刪除物件，`weak_ptr` 有可能會 "dangling"
* we initialize it from a `shared_ptr`:
    ```cpp
    auto p = make_shared<int>(42);
    weak_ptr<int> wp(p); // wp weakly shares with p; use count in p is unchanged
    ```
:::info
https://en.cppreference.com/w/cpp/memory/weak_ptr/weak_ptr
* C++14 可以 move `weak_ptr`
:::

* Because the object might no longer exist, *we cannot use a `weak_ptr` to access its object directly*. To access that object, **we must call `lock`.**
* The `lock` function checks whether the object to which the `weak_ptr` points still exists.
    * If so, lock returns a `shared_ptr` to the shared object.
    * As with any other `shared_ptr`, we are guaranteed that the underlying object to which that shared_ptr points continues to exist at least as long as that shared_ptr exists.
    ```cpp
    if (shared_ptr<int> np = wp.lock()) {
        // true if np is not null
        // inside the if, np shares its object with wp
    }
    ```
    * Inside the if, it is safe to use np to access that object

#### Checked Pointer Class
* 這裡會用到之前定義的 `StrBlob`
* 我們要為他加上一個類似 `StrBlob::iterator` 的功能!
* 所以定義了一個 class 叫做 `StrBlobPtr`，可以拿來"指向 `StrBlob` 的 elements"
    * 這個 `StrBlobPtr` 可能命名成 `StrBlobIter` 比較合適
    * 因為他的角色比較像是 `StrBlob::iterator`
* 概念上知道他是 iterator 了，所以需要可以用它存取到 `StrBlob` 的 elements
    * 代表需要存取 `StrBlob` 內部的 `shared_ptr` 指向的 `vector<string>`
* **但我們希望 `StrBlobPtr` 可以不要動到 `StrBlob` 內部物件的生命週期，所以 `StrBlobPtr` 在這個需求下就可用 `weak_ptr` 指向那個 `vector<string>`**

* `StrBlobPtr` 實際上會有兩個 member，一個是 `weak_ptr`, 一個是 `size_t`
    * 用 iterator 的概念去想就會很好理解為什麼有這兩個 member

* 一樣有一個 `check` member function 來確認 accesss 是否合法
    ```cpp
    // StrBlobPtr throws an exception on attempts to access a non existent element
    class StrBlobPtr {
    public:
        StrBlobPtr(): curr(0) { }
        StrBlobPtr(StrBlob &a, size_t sz = 0): wptr(a.data), curr(sz) { }
        // 注意那個 wptr 的初始化寫法，會使用到 StrBlob 的 private member，所以需要把 StrBlobPtr 宣告成 StrBlob 的 friend
        std::string& deref() const;
        StrBlobPtr& incr(); // prefix version
    private: // check returns a shared_ptr to the vector if the check succeeds     
        std::shared_ptr<std::vector<std::string>>
            check(std::size_t, const std::string&) const;
        // store a weak_ptr, which means the underlying vector might be destroyed
        std::weak_ptr<std::vector<std::string>> wptr;
        std::size_t curr; // current position within the array
    };
    ```
* 兩個 ctor
    * 一個 default ctor，把 `weak_ptr` member 設成 `nullptr`，`curr` 設成 `0`
    * 一個讓 `weak_ptr` "指向" 一個 `StrBlob`(內的 `data`)，然後把 `curr` 設成 `sz`
* It is worth noting that we cannot bind a `StrBlobPtr` to a `const StrBlob` object.
    * 因為 cotr 吃的是 non`const` 的 `StrBlob` 物件，所以 `const StrBlob` 不能傳進去
    * **感覺就好像 plain iterator 不能拿來指向 `const` container 一樣**

* `check` 是拿來檢查 `StrBlobPtr` 指向的 `StrBlob` 的底層 `data` 是否還存在，如果存在則檢查 `curr` 是否為合法 index，如果都是就 return 一個 `shared_ptr`，否則就噴 exception
    * 用 `shared_ptr.lock` 檢查 `vector<string>` 是否還在
    ```cpp
    std::shared_ptr<std::vector<std::string>>
        StrBlobPtr::check(std::size_t i, const std::string &msg) const {
        auto ret = wptr.lock();
        // is the vector still around?
        if (!ret)
            throw std::runtime_error("unbound StrBlobPtr");
        if (i >= ret->size())
            throw std::out_of_range(msg);
        return ret; // otherwise, return a shared_ptr to the vector
    }
    ```

##### Pointer Operations
* `operator*()` 跟 `operator++()`
    * 還沒學 overloading，改成定一般 member function `deref` 跟 `incr`
    ```cpp
    std::string& StrBlobPtr::deref() const {
        auto p = check(curr, "dereference past end");
        return (*p)[curr]; 
        // (*p)is the vector to which this object points
    }
    // prefix: return a reference to the incremented object 
    StrBlobPtr& StrBlobPtr::incr() {
        // if curr already points past the end of the container, can’t increment it 
        check(curr, "increment past end of StrBlobPtr");
        ++curr;
        // advance the current state
        return *this;
    }
    class StrBlob {
        friend class StrBlobPtr; 
        // other members as in § 12.1.1 (p. 456) 
        StrBlobPtr begin();  // return StrBlobPtrto the first element
        StrBlobPtr end();    // and one past the last element
    };
    // The begin and end members of class StrBlob should be defined outside the class body.
    // They can't be defined until class StrBlobPtr is complete.
    StrBlobPtr StrBlob::begin() { return StrBlobPtr(*this); } 
    StrBlobPtr StrBlob::end() { return StrBlobPtr(*this, data->size()); }
    ```
* 還會給 `begin` 跟 `end`，自己研究一下
    * 注意 `begin` 跟 `end` **不能定義**在 `StrBlob` 裡面，除非 `StrBlobPtr` 定義好了(**is complete type**)
    * 看一下 class 內用到的東西就會發現一定要先*宣告* `StrBlobPtr` 再*定義* `StrBlob` 才能再*定義* `StrBlobPtr`，可是 `StrBlob` 的 member function 又會用到 `StrBlobPtr` 的 object，所以 `StrBlob` 的這些用到 `StrBlobPtr` 的 member function 只能定義在 class 外面
        * 如果一頭霧水要再回去看第七章
        * or google

## 12.2 Dynamic Arrays

* 很多程式需要一塊連續的記憶體儲存多個一樣的 element
    * `string`, `vector` 等等
* C++ 提供兩種解法
    * 另一種 new expression
    * template `allocator`

* 其中 `allocator` 提供了更好且更彈性的效能

* 沒事不要自己配置記憶體，太ㄎㄧㄤ，請愛用 STL
    * 通常 STL 就可以拿來用，例如 `StrBlob` 內部是一個 `vector`
* 且在新標準之下，STL 會有更好的效能(`std::move`


:::info
* Best Practice
Most applications should use a library container rather than dynamically allocated arrays. Using a container is easier, less likely to contain memory-management bugs, *and* is likely to give better performance
:::
* 另外之前說過，如果自定義 class 沒有使用 `new`，則該 class 可使用 compile 生成的 ctor, copy, assign operation
:::warning
* Warning
Do not allocate dynamic arrays in code inside classes until you have read Chapter 13.
:::

### 12.2.1 new and Arrays
* 用 `int new obj_type[num];`
    ```cpp
    // call get_size to determine how many ints to allocate 
    int *pia = new int[get_size()]; // pia points to the first of these ints
    ```
    * 注意，這種形式的 `new` 的 return type 還是 pointer to first element，**不是 built-in array**
        * 還記得 dimension 是 array type 的一部分嗎
    * 因為這個原因，你不可以用 C++11 的 `std::begin` `std::end` 來取得他們的頭尾
        * 這兩個 template 會用到 array type 的 dimension 來決定邊界
        * 而 dynamic "array" 實際上不是 array type，所以沒有 dimension，所以不能用這兩個 function
    * 跟上面同樣的原因，dynamic array 同樣也不能ㄩㄥ在 range `for` 使用 `new obj_type[num]` return 回來的指標
    * 另外傳給 `new T[n]` 的 `n` 一定要是 integral type，但不用是 compile time `const`
* 使用 `new T[]` 可以結合 type alias
    ```cpp
    typedef int arrT[42]; // arrT names the type array of 42 ints
    int *p = new arrT; // allocates an array of 42 ints; p points to the first one
    ```
    * 這時候 code 就算沒有 `[]`，compiler 還是會用 `new []` 的版本
    * 好像在寫 `int *p = new int[42]` 一樣


#### Allocating an Array Yields a Pointer to the Element Type
* 雖然我們都說 `new T[]` 回傳的是**動態陣列**，但**嚴格來說他回傳的不是陣列**
    * 因為他回傳回來的就不是 array type object，而是 pointer to element type
* **Because the allocated memory does not have an array type, we cannot call `std::begin` or `std::end` (§ 3.5.3, p. 118) on a dynamic array.**
    * 之前講過了，不能用 `std::begin` `std::end`，還有 range `for`

#### Initializing an Array of Dynamically Allocated Objects
* 預設就是用 default initialize
* 如果要 value initialize，要加一個 empty parentheses`()`:
    ```cpp
    int *pia = new int[10]; // block of ten uninitialized ints 
    int *pia2 = new int[10](); // block of ten ints value initialized to 0
    string *psa = new string[10]; // block of ten empty strings
    string *psa2 = new string[10](); // block of ten empty strings
    ```
* C++11 允許使用 list initialization:
    ```cpp
    // block of ten ints each initialized from the corresponding initializer
    int *pia3 = new int[10]{0,1,2,3,4,5,6,7,8,9};     
    // block of ten strings; the first four are initialized from the given initializers
    // remaining elements are value initialized
    string *psa3 = new string[10]{"a", "an", "the", string(3,'x')};
    ```
* 這裡提一個動態靜態的概念，因為 allocated 的大小是動態才能知道的，所以 compiler 無法推論 initializer 的數量是否超過 allocated 大小；如果超過 allocated 的大小，那 `new` 就會 fail，不會配置任何空間，並且 `throw bad_array_new_length`
    * defined in `<new>`
* 還有，`new []` 的語法限制，`[]` 如果要給括號也只能給空括號，裡面不能給 initializer(要給 initializer 只能用上面的方法)。
    ```cpp
    auto PtrToSignel = new auto{87}; // legal
    auto PtrToMulti = new auto{9,4,8,7} // illegal
                                        // initialization of new-expression
                                        // for type 'auto' requires exactly
                                        // one element

    ```

#### It Is Legal to Dynamically Allocate an Empty Array
* `new T[num]`， `num` 可以給 0，而且不會出錯，但是 return 的指標不可以 dereference，是 UB
    ```cpp
    size_t n = get_size(); // get_size returns the number of elements needed
    int* p = new int[n]; // allocate an array to hold the elements
    for (int* q = p; q != p + n; ++q)
        /* process the array */;
    ```
    * 按上面這樣寫，n == 0，for 一次都不會執行
* 注意真正的 array type 不能宣告成 size 為 0:
    ```cpp
    char arr[0]; // error: cannot define a zero-length array
    char *cp = new char[0]; // ok: but cpcan’t be dereferenced
    ```
* When we use new to allocate an array of size zero, **new returns a valid, nonzero pointer**.
    * 還有，根據 stackoverflow 查的，你還是要對這種 size 為 0 的 new\[] 做 delete!
* **This pointer acts as the off-the-end pointer**
    * We can use this pointer in ways that we use an off-the-end iterator

#### Freeing Dynamic Arrays
* 對那些接受 `new T[]` 的指標 `p` ，需要用 `delete [] p;` 來歸還記憶體
* **對普通的 `new` 使用 `delete []` 或 vise versa 都是 UB**
* 還有一點，`delete` 一個 dynamic array，elements 是反向破壞
    * 其實 stack variable 也是
    * https://stackoverflow.com/questions/14688285/c-local-variable-destruction-order
    * 這個有講相關的，是在講 stack variable destruction order

* 還有，如果你是 `new` 一個 type alias，`new arrT`，which is array，你一樣要用 `delete []`，否則也是 UB
    ```cpp
    typedef int arrT[42]; // arrT names the type array of 42 ints
    int *p = new arrT; // allocates an array of 42 ints; p points to the first one
    delete [] p; // brackets are necessary because we allocated an array
    ```
:::warning
* 通常你 `delete` 的版本跟要被 delete 的 ptr 不 match 的話，compiler 可能檢查不出來
:::


#### Smart Pointers and Dynamic Arrays
* `unique_ptr` 有一種版本是拿來指向 dynamic array 的
    * 也就是宣告 `unique_ptr` 時角括號內的 type 要額外加上 `[]`:
    * 實際上就是 `unique_ptr` 有定義 partial template
    ```cpp
    // up points to an array of ten uninitialized ints 
    unique_ptr<int[]> up(new int[10]);
    up.release(); // automatically uses delete[] to destroy its pointer
    ```
    * 這樣在 `release` 時就會 call `delete[]` 而不是 `delete`。

* 使用這種版本的 `unique_ptr`，它的可用操作跟一般版本(table 12.1.5, p.470)有些不同:
    * ![](https://i.imgur.com/RNMBPcM.png)
    * 不能用 `operator.()` 跟 `operator->()`
        * 畢竟這種 `unique_ptr` 指向的是 array，這兩個 operator 不 make sense
    * 但可以用 `operator[]()`
        * 合理，用來存取 element
    ```cpp
    for (size_t i = 0; i != 10; ++i)
        up[i] = i; // assign a new value to each of the elements
    ```

#### `shared_ptr` to danymic array
* **結論，C++17 可以用，用法跟 `unique_ptr<T[]>` 一樣，table 12.6**
    * * https://stackoverflow.com/questions/13061979/shared-ptr-to-an-array-should-it-be-used
    * 總之是用新的 C++17 template hack 達成的...
* prior to C++17 就要用以下的方法
* `shared_ptr` 就沒有 array 版本了
    * C++17 有
* 你如果要用 `shared_ptr` 指向一個動態陣列，你必須提供 deleter 來做 `delete []`
    ```cpp
    // to use a shared_ptr we must supply a deleter 
    shared_ptr<int> sp(new int[10], [](int *p) { delete[] p; });
    sp.reset(); // uses the lambda we supplied that uses delete[] to free the array
    ```
    * 這裡直接傳一個會 call `delete[]` 的 lambda
* 如果你沒有提供 deleter，而且 `shared_ptr` 又指向一個動態陣列，就會是 UB
    * pointer 跟 delete 方式不 match

* 另外 `shared_ptr` 不支援 `operator[]` 跟 pointer arithmetic
* 如果你真的用 `shared_ptr` 來管裡動態陣列，你必須這樣存取這個陣列:
    ```cpp
    // shared_ptrs don’t have subscript operator and don’t support pointer arithmetic
    for (size_t i = 0; i != 10; ++i)
        *(sp.get() + i) = i; // use get() to get a built-in pointer
    ```
    * 必須拿 `shared_ptr` 內的 raw pointer 來存取每個 element

### 12.2.2 The `allocator` Class(template)
* 參考
    * https://en.cppreference.com/w/cpp/memory/allocator
    * 還有其他 container 的 ctor，你會看到有傳 element `T` 的 allocator
    * The `std::allocator` class template is the default Allocator used by all standard library containers if no user-specified allocator is provided.
* `new` 的缺點就是在**配置空間時同時初始化物件**
    * Combining initialization with allocation
* delete 也是在**破壞物件時同時歸還空間**
    * Combining destruction with deallocation
* 這在只 `new` 一個物件時是合適的，因為這種情境通常會希望把配置記憶體跟初始化一起做 
* 但是在一次配置多個物件時就不夠彈性，例如 `new T[n]` 就會一次 construct `n` 個物件，可是有時候我們說不定根本用不到 `n` 個物件，這樣沒用到的物件就浪費時間初始化了
* 而且 `new[]` 除了 initializer list 的寫法也只能 default 初始化，**這意味著沒有 default ctor 的物件不能使用 `new[]`**

* `allocator<T>` class 最大目的:
    * 讓我們 **decouple memory allocation from object construction**，將記憶體配置跟建構物件這兩件事分離

* 來看 `new []` 很沒效率的例子:
    ```cpp
    string *const p = new string[n]; // construct n empty strings
    string s;
    string *q = p; // q points to the first string
    while (cin >> s && q != p + n) 
        *q++ = s;    // assign a new value to *q
    const size_t size = q - p; //  remember how many stringswe read
    // use the array
    delete[] p; // p points to an array; 
    ```
    * 我們雖然配置 `n` 個 string，可是我們不一定會 `n` 個都使用
    * 還有，我們真的有使用的(小於 n 個)strings，在 `new` 使用 default 初始化之後我們又(馬上) assign 一次，這樣就給了兩次值

#### The `allocator` Class(template)
![](https://i.imgur.com/PPb619I.png)
* 想先跳過... 被劃掉的那兩個 deprecated
    * https://stackoverflow.com/questions/39414610/why-are-are-stdallocators-construct-and-destroy-functions-deprecated-in-c17

* 定義在 `<memory>`
* 介紹怎麼用，13.5 會有一個更完整的例子
* template
* 定義 `allocator<T>` 給的 type，代表他可以配置的物件的型態
* 當 `allocator` 配置記憶體時，他會根據給定型態的大小跟 OS 要記憶體以及*做適當的 memory alignment...*
    ```cpp
    allocator<string> alloc; // object that can allocate strings
    auto const p = alloc.allocate(n); // allocate nunconstructed strings
    ```
    
#### `allocator`s Allocate **Unconstructed** Memory
* The memory an `allocator` allocates is **unconstructed**.
* 要使用 `allocator` 分配的記憶體內的物件，一定要先 `construct`
* `allocator<T>.construct(p, args)`，接受一個指標，跟一堆參數，指標要指向 `allocator` 分配的一大塊記憶體的某個位置，一堆參數是要拿來當作物件的 ctor 參數的
    * `args` must match a constructor for that class
    ```cpp
    auto q = p; // q will point to one past the last constructed element
    alloc.construct(q++); // *q is the empty string
    alloc.construct(q++, 10, 'c'); // *q is cccccccccc
    alloc.construct(q++, "hi"); // *q is hi
    ```
    * 看那個 `q++` 有沒有很有感覺?就是要對 allocate 回傳的一大塊記憶體掃過去一次，把物件一個一個建構
    * C++11 之前，`allocator<T>.construct()` 只吃兩個參數，一個是指標，一個是 value，呼叫 `copy` ctor 來複製這個 value

:::warning
* It is an error to use raw memory in which an object has not been constructed:
    ```cpp
    cout << *p << endl; // ok: uses the string output operator
    cout << *q << endl; // disaster: q points to unconstructed memory!
    ```
:::

* 用完 `allocator<T>.construct()` 之後的物件，一定要對他 call `allocator<T>.destroy()`:
    ```cpp
    while (q != p)
        alloc.destroy(--q);
    // free the strings we actually allocated
    ```
    * 看看那個精美的 `--q`
    * `q` 是指向最後一個建構的物件之後的位置，所以要 prefix

* 當然也不可以對尚未呼叫 `construct` 的 element 直接 call `destroy`，UB
* 一旦 `destroy` 之後，可以呼叫 `deallocate` 歸還整大塊記憶體
    * 或者 reuse，construct elements again
    ```cpp
    alloc.deallocate(p, n);
    ```
    * `p` 一定要是 `allocate` 回傳的指標，n 也要是當出呼叫 allocate 時對應的大小
        * 宣告這兩個值為 `const` 是你的好朋友 

#### Algorithms to `Copy` and `Fill` Uninitialized Memory
* `construct` 一次一個太累了，可以用下面的類似 `std::copy`, `std::fill` 的 functions，一樣定義在 `<memory>`
    ![](https://i.imgur.com/AgVOhIS.png)
    * `uninitialized_fill_n` 不知為何要提 `unsigned` number n


* 下面小例子，有個 `vector<int> vi`，`allocator<int>` `alloc` 配置一塊他兩倍大小的記憶體，前半塞 `vi` 的 elements，後半塞給定的 `int` value:
    ```cpp
    // allocate twice as many elements as vi holds
    auto p = alloc.allocate(vi.size() * 2);
    // construct elements starting at p as copies of elements in vi
    auto q = uninitialized_copy(vi.begin(), vi.end(), p);
    // initialize the remaining elements to 42
    uninitialized_fill_n(q, vi.size(), 42);
    ```
    * `uninitialized_copy` 跟 `std::copy` interface 一致，會回傳建構完的最後一個 element 之後的位置
        * 可以拿這個指標來配置 `p` 後半段的部分
    * 注意，`uninitialized_copy` 第三個參數一定要指向一塊 unconstructed 的記憶體，否則是 UB

## 12.3 Using the Library: A Text-Query Program
* 一言以蔽之:簡單版 file `grep`
    * 完全字串比對
* Primer 要用一個例子來總結 Primer Part II 介紹的東西
* Text-Query Program
* 給定一個 text file，user 可以查詢某個字串出現在哪些行數，並且全部印出來:
    ```
    element occurs 112 times
    (line 36) A set element contains only a key; 
    (line 158) operator creates a new element
    (line 160) Regardless of whether the element 
    (line 168) When we fetch an element from a map, we
    (line 214) If the element is not found, find returns
    ...
    ```
    * 如果一個字串在同一行出現多次，我們只會算他出現一次
    * 輸出的 content 會按照行數排列
    * 上面的例子是搜尋 Primer 12章(文字)的內容

### 12.3.1 Design of the Query Program
* 要開始設計一個程式的好方法是先想他需要支援什麼操作(operations)
* 一旦知道需要什麼操作我們就可以開始考慮需要什麼資料結構
* 以下是 Text-query program 需要的操作:
    * 當在讀文字檔時，他必須記得每一行的行數，因為之後要印出來
        * 最簡單的方法是一次讀一行，並且帶隨一個 counter 來計算行數
    * 當輸出結果時，Program 必須有辦法找到 user 查詢的 word 出現的所有行數，以及對應的內容
    * 輸出的行數必須排序，並且沒有重複

* 思考一下，這些需求可以輕易地用 STL 達成:
    * 用 `vector<string>` 儲存整份文字檔，一個 `string` 一行；index 直接代表行數(不過是從 0 開始)
    * 用 `std::istringstream` 來處理每一行文字並切成一個一個 word，詳情第八章
    * 用 `std::map<string, std::set<size_t>>` 來存取每個 word 所出現的行數
        * word 當成 key，value 是出現的行數
        * 用 set 可以達到行數排序並且不重複
    * 最後因為一些原因，我們會用 `shared_ptr` 管裡某些資料結構，之後會講

#### Data Structures
* 我們當然可以把整個程式邏輯寫一個 procedured program 並且直接使用 STL 幹出來
* 可是如果把 STL 用 class 封裝起來，把 STL 給抽象掉，這樣在使用這個 class 時想的就會是 text-query 的操作，而不是 STL 的操作
* 更甚者，要擴充程式也比較容易(15 章會繼續擴充)

#### 實作
* 定義一個 class `TestQuery`，專門用來查找字串
    * 裡面有一個 `std::vector` 跟 `std::map`
    * `std::vector` 存著由檔案內容
    * `std::map` 則是拿來查找字串出現的行數的資料結構

    * API:
        * 總共有一個 ctor，吃 `std::ifstream` 來建構 `vector` 跟 `map`
        * 還有 `query` member function，找字串，會回傳一個 query object，之後會詳細說明
            * 其實就是把 query 所需要的詳細結果封裝起來的 class
                * 要找的字串啊
                * 行號啊
                * 每行的內容等等
        * 注意 query object **只是概念上有這些封裝的內容**
        * 實際上，之後會看到，是**用 `shared_ptr` 指向 `TextQuery` object 內部的 member**

* `QueryResult`
    * 上面說的 query object
    * hold results of a query
    * 還有一個 `print` global function 可以印出結果
        * 15 章應該會改寫成 `operator<<()`

:::info
這邊還是照著 Primer 看比較容易理解
:::

#### Sharing Data between Classes
* `QueryResult` 概念上要存的東西實際上是存在 `TextQuery` object 裡面的
    * 就是存著文字檔的 `vector` 以及存著轉換結果的 `map`
    * `QueryResult` 會需要 `map` 內某個 word 對應的行數
    * 並且利用這些行數去 access `vector` 並印出每一行的文字
* 既然如此就要決定，怎麼在 `QueryResult` 物件裡存取 `TextQuery` 的內容
* 實際上最單純的做法，可以直接 copy 整個檔案內容(也就是 `TextQuery` 內的 `vector`) 跟要查詢的文字對應的行數(也就是 `map` 內對應 word 的 `set`)
    * 可是 copy `set`，不便宜，copy `vector` 更貴
* 也可以用 pointer 指向這些物件，**可是這樣就要考慮到 pointer 什麼時候會 invalid 的問題，如果原本指向的 `TextQuery` 物件被破壞，那在 `QueryResult` 內的 pointer 就會變成 dangling pointer 了**
* 這種情境就是適合使用 `shared_ptr`，因為概念上我們就是讓 `TextQuery` 跟 `QueryResult` **共用**(shared)資料，而且又要**同步這些內部資料的 lifetime**，既然如此就用 `shared_ptr`

#### Using the `TextQuery` Class
* 一種 Design Practice 的常見情境是，當我們要定義一個 class，一個好方法是先用這個 class 寫一個 application:
    * user code
    ```cpp
    void runQueries(ifstream &infile) {
        // infile is an ifstream that is the file we want to query
        TextQuery tq(infile); // store the file and build the query map
        // iterate with the user: prompt for a word to find and print results
        while (true) {
            cout << "enter word to look for, or q to quit: ";
            string s; // stop if we hit end-of-file on the input or if a 'q' is entered
            if (!(cin >> s) || s == "q")
                break;
                // run the query and print the results 
                print(cout, tq.query(s)) << endl;
        }
    }
    ```
    
#### 12.3.2 Defining the Query Program Classes
* 來實作 `TetQuery` 跟 `QueryResult`，並且以共用底層資料為前題
* 他們會共用 `vector<string>`，保存文字檔，以及 `set<line_no>`，保存某個 word 出現的行數

* 所以這樣設計 data members:
    * `shared_ptr<vector<string>>`，指向存著文字檔的 `vector`
    * `map<string, shared_ptr<set<line_no>>>`，一個 `map`，mapping 到**動態配置的 `set`**
        * string to `shared_ptr<set>`!
    * `line_no`，只是一個自定義的型態，實際上是 `size_type`，用他來做 `vector<string>` 的 index
    ```cpp
    class QueryResult; // declaration needed for return type in the query function
    class TextQuery {
    public:
        using line_no = std::vector<std::string>::size_type; 
        TextQuery(std::ifstream&);
        QueryResult query(const std::string&) const;
    private:
        std::shared_ptr<std::vector<std::string>> file; // input file
        // map of each word to the set of the lines in which that word appears
        std::map<std::string, std::shared_ptr<std::set<line_no>>> wm;
    };
    ```
    * 因為會把 class definition 寫在 header file，所以這邊不用 `using` 防止 include header 的 code 發生 name collision

#### The `TextQuery` Constructor
* 吃一個 `std::ifstream&`，把檔案建構成 `vector<string>`，把每個 word 跟對應行數建成 `map`
    ```cpp
    // read the input file and build the map of lines to line numbers
    TextQuery::TextQuery(ifstream &is): file(new vector<string>) {
        string text;
        while (getline(is, text)) { // for each line in the file 
            file->push_back(text); // remember this line oftext
            int n = file->size() - 1; // the current line number 
            istringstream line(text); // separate the line into words
            string word;
            while (line >> word) { // for each word in that line
                // if word isn’t already in wm, subscripting adds a new entry
                auto &lines = wm[word]; // lines is a shared_ptr 
                if (!lines) // that pointer is null the first time we see word
                    lines.reset(new set<line_no>); // allocate a new set 
                lines->insert(n);
                // insert this line number } }
    }
    ```
    * ctor 會**配置一個 vector**(`file(new vector<string>>`)
    * 把檔案一行一行 push 到 vector (`file->push_back(text);`)
        * 注意要用 `operator->()`，因為 `file` 是 `shared_ptr`
    * 再來用 `std::istringstream` 把單字一個一個丟到 `map`
        * 注意這邊用 `reference` bind `word` 對應的 value，也是一個 `shared_ptr`
        * 記得如果對 `std::map` 用 `operator[]` 是 firstOrCreate
            * 沒有對應 element 時，會做 default construct
            * 所以如果當前 `word` 沒有出現在 map，就會創造一個新的 pair，pair.second 是一個**指向 null 的 `shared_ptr`**
        * 所以才要確認 `shared_ptr`，也就是 `lines` 是否為 null，如果是就要先 `reset`，指向一個 `set` 出來
        * 最後再把行號插到 `set` 內
            * 如果該行已經出現在 `set`，`insert` 就無效

#### The `QueryResult` Class
* 三個 data members:
    * `string`，要找尋的字串
    * `shared_ptr<vector<string>>`，指向存著文字檔的，跟 `TextQuery` 共用的 `vector<string>`
    * `shared_ptr<set<line_no>>`，指向共用的 `TextQuery` 內的 map 內，要查詢的 word 對應的 `set`

* 如果覺得我這樣寫很雞巴，就看看原文寫的吧，或者看 code

* 只有一個 ctor
    * 吃三個參數，分別把上面三個 members 初始化
    ```cpp
    class QueryResult {
        friend std::ostream& print(std::ostream&, const QueryResult&);
    public:
        QueryResult(std::string s,
            std::shared_ptr<std::set<line_no>> p, std::shared_ptr<std::vector<std::string>> f):
        sought(s), lines(p), file(f) { }
    private:
        std::string sought; // word this query represents
        std::shared_ptr<std::set<line_no>> lines; // lines it’s on     
        std::shared_ptr<std::vector<std::string>> file; // input file
    };
    ```
    
#### The `TextQuery::query` Function
* 吃一個 `string` `sought`，找出 `TextQuery` 內 `sought` 內對應的行數(對應的 `set`)
* 找到之後，用 `TextQuery` 對應的 members 來建構一個 `QueryResult` 並回傳
    * 如果 `map` 內沒有 `string` 呢?
        * 這時候對應的 `set` 不存在啊?
    * solve this problem by **defining a local `static` object that is a `shared_ptr` to an empty `set` of line numbers.**
    * 如果沒找到字串，回傳的 `QueryResult` 物件內指向 `set` 的 `shared_ptr` 就指向這個 `static` member
    ```cpp
    QueryResult TextQuery::query(const string &sought) const {
        // we’ll return a pointer to this set if we don’t find sought
        static shared_ptr<set<line_no>> nodata(new set<line_no>);
        // use find and not a subscript to avoid adding words to wm!
        auto loc = wm.find(sought);
        if (loc == wm.end())
            return QueryResult(sought, nodata, file); // not found
        else
            return QueryResult(sought, loc->second, file);
    }
    ```
    * 注意不能用 `operator[]` 要用 `map.find`，否則是 firstOrCreate

#### Printing the Results
```cpp
ostream &print(ostream & os, const QueryResult &qr) {
    // if the word was found, print the count and all occurrences
    os << qr.sought << " occurs " << qr.lines->size() << " " << 
        make_plural(qr.lines->size(), "time", "s") << endl;
    // print each line in which the word appeared 
    for (auto num : *qr.lines) // for every element in the set
    // don’t confound the user with text lines  starting at 0
    os << "\t(line " << num + 1 << ") " << 
        *(qr.file->begin() + num) << endl;
    return os;
}
```
* 注意，我們存的 `line_no` 都是 0-based，一般文字行號都是 1-based，所以印出行號時要額外 + 1
    * (`os << "\t(line " << num + 1 << ") ";`)
* 直接用 `set<T>.size` 得知 word 總共出現幾次
* 最後用 `*(qr.file->begin() + num)` 取得對應行數的字串
    * `qr.file->begin()[num]` 也可以
* 講完了
    * 另外就算找不到字串，我們的 class 也不會出問題
    * 會印出「word 出現 0 次」並且 `print` 內的 range for 一次也不會執行(因為 `set` 是 `empty`)
