# C++ Primer Chapter 12 Dynamic Memory

* The programs we’ve written so far have used objects that have welldefined lifetimes.
    * 這裡是指目前用過的 objects 都已經定義好 lifetime，什麼時候該死該破壞 compiler 都幫你搞定了
    * Global objects are allocated at program start-up and destroyed when the program ends.
    * Local, automatic objects are created and destroyed when the block in which they are defined is entered and exited.
    * Local static objects are allocated before their first use and are destroyed when the program ends

* C++ lets us allocate objects dynamically.
* Dynamically allocated objects have a lifetime that is independent of where they are created; they exist until they are explicitly freed.
    * 動態配置的物件他的 lifetime 要由 programmer 自己 handle 自己定義!

## 12.1 Dynamic Memory and Smart Pointers
* In C++, dynamic memory is managed through a pair of operators:
    * new, which allocates, and optionally initializes, an object in dynamic memory and returns a pointer to that object;
    * and delete, which takes a pointer to a dynamic object, destroys that object, and frees the associated memory.

* **It is surprisingly hard to ensure that we free memory at the right time!**
    * Either we **forget to free the memory**—in which case we have a memory leak—or we **free the memory when there are still pointers referring to that memory**—in which case we have a pointer that refers to memory that is no longer valid.

* C++11 定義了兩種 smart pointer types 來把 raw pointer 包起來
    * A smart pointer acts like a regular pointer with the important exception that **it automatically deletes the object to which it points.**
* C++11 defines two kinds of smart pointers that *differ in how they manage their underlying pointers*:
    * **shared_ptr**, which allows multiple pointers to refer to the same object
    * **unique_ptr**, which “owns” the object to which it points.
    * The library also defines a companion class named **weak_ptr** that is a weak reference to an object managed by a shared_ptr.
        * 第三個不知道在工三小ㄏㄏ
* 上面三個都定義在 \<memory> 裡面


### 12.1.1 The shared_ptr Class
* 首先，他是 class template
* 其實就好像你 raw pointer 可以宣告成指向任何 type 一樣啦，\<> 內給的就是 pointer 指向的 type
    ```C++
    shared_ptr<string> p1; // shared_ptrthat can point at a string
    shared_ptr<list<int>> p2; // shared_ptrthat can point at a listofints
    ```
* smart pointer 的 default cstr 會把底層的 raw ptr 設成 nullptr
* smart pointer 也定義好 overloading，讓你可以用操作一般 ptr 的方式操作
    ```C++
    // if p1 is not null, check whether it’s the empty string
    if (p1 && p1->empty())
        *p1 = "hi"; // if so, dereference p1 to assign a new value to that string
    ```
    * 上面也是用 short circuit 來判斷 p1 是否指向物件，再判斷指向的字串是否為空
    * smart ptr 被當成 condition 是等價於判斷底層 raw ptr 是否為 NULL

* 下表給定 shared_ptr 跟 unique_ptr 共同的 interface
* ![](https://i.imgur.com/duawgE6.png)
* 而這張是 shared_ptr 獨有的
* ![](https://i.imgur.com/tyLGNUR.png)



#### The make_shared Function
* The safest way to allocate and use dynamic memory is to call a library function named make_shared. 
* This function **allocates and initializes an object in dynamic memory and *returns a sharedptr*** that points to that object.
    * 他其實是 function template
    * 一樣在 \<memory>
    * call 的時候要指定 object type
    ```C++
    // shared_ptr that points to an int with value 42
    shared_ptr<int> p3 = make_shared<int>(42);
    // p4 points to a string with value 9999999999 
    shared_ptr<string> p4 = make_shared<string>(10, '9');
    // p5 points to an int that is value initialized (§ 3.3.1 (p. 98)) to 0
    shared_ptr<int> p5 = make_shared<int>();
    ```
* 注意 make_shared 的 API，長得跟 sequential container 的 emplace 很像，**裡面放的參數其實是物件的 cstr 的參數!**
* 還有上面三個宣告的 type 其實都可以用 auto 代替
    ```C++
    // p6 points to a dynamically allocated, empty vector<string>
    auto p6 = make_shared<vector<string>>()
    ```

#### Copying and Assigning shared_ptrs
* When we copy or assign a shared_ptr, **each shared_ptr keeps track of how many other shared_ptrs point to the same object:**
    ```C++
    auto p = make_shared<int>(42);
    // object to which p points has one user
    auto q(p); // p and q point to the same object
    // object to which p and q point has two users
    ```
* We can think of a shared_ptr **as if it has an associated counter,** usually referred to ***as a reference count.***
    * 當我們 **copy** shared_ptr，counter + 1
    * 這個 copy 亦即各種 context 之下物件被 copy 的那個 copy，例如
        * auto q(p);
        * fun(p);
        * q = p;
    * 當我們 assign 一個新 value 給 shared_ptr 物件，那個 ptr 指像的物件對應的 reference count 就會少 1(當然，新 value 指像的物件對應的 count 會加 1)
    * 當 shared_ptr 被破壞也會少 1
        * 例如 out-of-scope

* 當 shared_ptr 指像的物件的 count 變回 0，那個物件會自動被 free 掉
    ```C++
    auto r = make_shared<int>(42);
    // int to which r points has one user
    r = q;
    // assign to r, making it point to a different address
    // increase the use count for the object to which q points
    // reduce the use count of the object to which r had pointed
    // the object r had pointed to has no users; that object is automatically freed
    ```
* Note: It is *up to the implementation* whether to use a counter or another data structure to keep track of how many pointers share state.
    * library 到底用什麼資料結構來記住什麼物件被幾個 shared_ptr 指著其實你不用管，你只要知道 library 會處這件事情就好了，而且當那個物件沒人指時物件就會被 free 掉。
    * 所以你不用管到底是每個指標都有一個 counter，還是每個物件都有一個 counter 然後指標去使用 counter，這種事情，這種事情是實作決定的

#### shared_ptrs Automatically Destroy Their Objects ...
* Primer 在點點點三小啦
* 反正殺掉 object 的 code 就是寫在 destructor 裡面
* The **destructor** for shared_ptr **decrements the reference count** of the object to which that shared_ptr points.
* If the **count goes to zero**, the shared_ptr destructor **destroys the object** to which the shared_ptr points and frees the memory used by that object.

#### ...and Automatically Free the Associated Memory
* The fact that the shared_ptr class automatically frees dynamic objects when they are no longer needed makes it fairly easy to use dynamic memory.
* 看看下面的範例
    ```C++
    // factory returns a shared_ptr pointing to a dynamically allocated object 
    shared_ptr<Foo> factory(T arg) {
        // process argas appropriate 
        // shared_ptrwill take care ofdeleting this memory 
        return make_shared<Foo>(arg);
    }
    ```
* The following function stores the shared_ptr returned by factory in a local variable:
    ```C++
    void use_factory(T arg) {
        shared_ptr<Foo> p = factory(arg); // use p
    }// p goes out of scope; the memory to which p points is automatically freed
    ```
    * When p is destroyed, its reference count is decremented and checked. In this case, p is the only object referring to the memory returned by factory.
        * Because p is about to go away, the object to which p points will be destroyed and the memory in which that object resides will be freed.
* 可是你如果不想要物件在 use_factory 就被刪掉，你可以把 p return
    ```C++
    shared_ptr<Foo> use_factory(T arg) {
        shared_ptr<Foo> p = factory(arg);
        // use p
        return p; // reference count is incremented when we return p
    }// p goes out of scope; the memory to which p points is not freed

* 感覺要多看一點 case 或 garbage collection 的東西才知道要怎麼用 reference count 比較好
* Because memory is not freed until the last shared_ptr goes away, **it can be important to be sure that shared_ptrs don’t stay around after they are no longer needed.**
    * 如果你明明某個 shared_ptr 已經不需要了卻沒有把他殺掉，那物件就不會被釋放
        * 例子是你把 shared_ptr 放到 container
        * 當你對 container 做一些操作，然後發現某些 shared_ptr 不需要了，**你必須明確的用 erase** 把他們消除，否則他們指向的物件就不會被釋放


#### Classes with Resources That Have Dynamic Lifetime

* Programs tend to use dynamic memory for one of three purposes:
    1. They don’t know how many objects they’ll need
    2. They don’t know the precise type of the objects they need
    3. They want to share data between several objects
* 第一點的例子是 container
* 第二點的例子在 15章才會講，繼承之類ㄉzz
* 第三個例子接下來要給
    * In this section, we’ll define a class that *uses dynamic memory in order to let several objects share the same underlying data.*

* 到目前為止我們宣告的物件，其使用的資源的生命週期都跟物件一致
    * 例如 vector\<int> 裡面的 int 用的資源都跟 vector\<int> 共生死
    * 到目前的 code 都只用這種

* 不過有種寫法是，**宣告一個物件之後他要了一塊資源，而這個資源的生命週期跟這個物件的生命週期是不相關的**
* 假設我們想要訂一個 class Blob，他的行為是這樣
    * Blob object 會存一堆元素的集合(a collection of elements)
    * 當我們複製 Blob object 時，我們希望新複製的 object 內存的元素跟原本的是同一組
        * 用 Python 觀點來看 a.elem is b.elem == true
        * **share the same elements**
        * **refer to the same underlying elements.**

* 根據上面的行為有一個條件必須滿足，**當兩個物件共享底層的元素時，如果其中一個物件被 destroy，他不可以把底層元素刪除**
    ```C++
    Blob<string> b1; // empty Blob
    {
        // new scope
        Blob<string> b2 = {"a", "an", "the"};
        b1 = b2; // b1 and b2 share the same elements
    }// b2 is destroyed, but the elements in b2 must not be destroyed
    // b1 points to the elements originally created in b2
    ```
    * When b2 goes out of scope, **those elements must stay around, because b1 is still using them.**

#### Defining the StrBlob Class
* Primer 說 Blob 這鬼東西最終會實作成 template... 但還沒講，所以先用個只存 string 的版本
* The easiest way to implement a new collection type is to use one of the library containers to manage the elements.
    * 比方說 Blob 內放一個 vector\<string>
* However, we can’t store the vector directly in a Blob object.
    * 可是這樣做，Blob object 死掉了，這個 vector 也會消失
    * 這樣沒辦法做到所謂的兩個 Blob object 共享 underlying object
    * **我們必須把 vector 存在 dynamic memory 裡面**
* To implement the sharing we want, we’ll **give each StrBlob a shared_ptr to a dynamically allocated vector.**
* **That shared_ptr member will keep track of how many StrBlobs share the same vector** and will delete the vector when the last StrBlob using that vector is destroyed.
* 所以這個 StrBlob 到底要銃三小?
    * Primer 在這裡先定義一點點的 interface 作為 demo 之用
    ```C++
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
    * 注意 private data 是一個 shared_ptr
        * 還有，這裡還沒給 cstr，沒有給你看怎麼把新的 StrBlob object 指向共用的 vector

#### StrBlob Constructors
* 下面是 cstr，有兩種，一種 default，另一種吃 initializer_list，可以達成任意個字串
    ```
    StrBlob::StrBlob(): data(make_shared<vector<string>>()) { } 
    StrBlob::StrBlob(initializer_list<string> il):
            data(make_shared<vector<string>>(il)) { }
    ```
    * 別忘記 make_shared 吃的參數是 element 的 constructor 的參數，而這裡的 element 是 vector\<string>
    * 第一個會 allocate 一個 empty vector，第二個會 allocate 一個 vector 裡面塞 il 給的字串

#### Element Access Members
* pop_back, front, and back
* 這些 member 在 acces 底層 vector 時都要先確認 element 存在，所以把這個 check 又寫成一個 private function
```C++
void StrBlob::check(size_type i, const string &msg) const {
    if (i >= data->size()) throw out_of_range(msg);
}
```
* 首先這個 check 第一個參數就是檢查 access 的 index 有沒有超過 vector 邊界
* 但第二個其實是要丟給 exception 當參數用的，是 error message
    * 可能之後這個 class 還要拿來 demo 夠完整個 exception 寫法，所以在這裡就先寫好吧
* 阿三個 member 就給不同的參數給 check 以做確認
```C++
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
* 上面兩個 access member 跟 pop_back 都只要確認 vector 不為空就好了，所以傳給 check 的 size 都為 0
* 然後 front 跟 back 應該還要有一個 cont 的 overload 版本，Primer 說這丟到練習 XD

#### Copying, Assigning, and Destroying StrBlobs
* 真正會用到 shared_ptr 用處的地方來惹
* 好消息是這三種用 default 的版本就可以用到 shared_ptr 的功能惹!
* Our StrBlob has only one data member, which is a shared_ptr.
* Therefore, when we copy, assign, or destroy a StrBlob,its shared_ptr member will be copied, assigned, or destroyed.
* *As we’ve seen, copying a sharedptr increments its reference count; assigning one sharedptr to another increments the count of the right-hand operand and decrements the count in the left-hand operand; and destroying a sharedptr decrements the count.*
    * 再複習一次 shared_ptr 的行為
* If the count in a shared_ptr goes to zero, the object to which that shared_ptr points is automatically destroyed.
    * 所以這時候這個 shared_ptr 指向的 vector 也會被 free 掉!

### 12.1.2 Managing Memory Directly
* 要你直接使用 new 跟 delete 惹!
* classes that do manage their own memory— unlike those that use smart pointers—**cannot rely on the default definitions for the members that copy, assign, and destroy class objects**
* 13 章才會講怎麼寫上面的 cstr，所以現在先假設不會在 class 內用 new 跟 delete(但可以用 smart pointer)

#### Using new to Dynamically Allocate and Initialize Objects
* new returns a pointer to the object it allocates:
    ```C++
    int *pi = new int; // pipoints to a dynamically allocated,
    // unnamed, uninitialized int
    ```
* This new expression **constructs an object of type int on the free store** and returns a pointer to that object.
* heap 上的物件預設是 default initailized，所以 built-in value 是 undefined，如果是 class 就用 default cstr
    ```C++
    string *ps = new string; // initialized to empty string
    int *pi = new int;
    // pi points to an uninitialized int
    ```
* We can initialize a dynamically allocated object using direct initialization (§ 3.2.1, p. 84).
    * 就是在 new 的 type 後面加上給 cstr 的參數
    ```C++
    int *pi = new int(1024); // object to which pi points has value 1024 
    string *ps = new string(10, '9');
    // *ps is "9999999999"
    // vector with ten elements with values from 0 to 9
    vector<int> *pv = new vector<int>{0,1,2,3,4,5,6,7,8,9};
    ```
    * 最後一個例子就 list initialization，new 的時候一樣可以用
* We can also value initialize (§ 3.3.1, p. 98) a dynamically allocated object by following the type name *with a pair of empty parentheses:*
    * 在 type name 後面加上括號就可以 value-initialize
    * 這對 class type 沒差，都 call default cstr
    * 但對 built-in type 差很多，會 initialie 成 0
* 跟很久以前說的一樣，**我們用 new 的時候最好也想辦法初始化我們的物件**

* C++11 當然還可以讓你用 auto 阿!
    ```C++
    auto p1 = new auto(obj); // p points to an object ofthe type of obj
                             // that object is initialized from obj
    auto p2 = new auto{a,b,c}; // error: must use parentheses for the initializer
    ```
    * 不過只能用括號的形式，並且只能吃一個參數，compiler 直接用那個參數來推斷型別
    * The newly allocated object is *initialized from the value of obj*

#### Dynamically Allocated ***const*** Objects
* 當然可以 new 一個 const object
    ```C++
    // allocate and initialize a const int
    const int *pci = new const int(1024);
    // allocate a default-initialized constempty string
    const string *pcs = new const string;
    ```
    * 一樣一定要初始化
    * ptr returned by new is pointer to const

#### Memory Exhaustion
* By default, if new is unable to allocate the requested storage, it throws an** exception of type bad_alloc** (§ 5.6, p. 193).
* 可以像下面那樣寫防止 new throw
    ```C++
    // if allocation fails, new returns a null pointer
    int *p1 = new int; // if allocation fails, new throws std::bad_alloc
    int *p2 = new (nothrow) int; // if allocation fails, new returns a null pointer
    ```
    * this form of new is referred to as **placement new.**
    * A placement new expression lets us **pass additional arguments** to new. In this case, we pass an object named nothrow that is defined by the library. 
* bad_alloc 跟 onthtow 都定義在 \<new>

#### Freeing Dynamic Memory
```C++
delete p; // pmust point to a dynamically allocated object or be null
```
* Like new, a delete expression performs two actions:
    * It destroys the object to which its given pointer points,
    * and it frees the corresponding memory.
#### Pointer Values and delete
* The pointer we pass to delete **must either point to dynamically allocated memory or be a null pointer**
    * delete 其他 pointer 或者 delete new 出來的 object 兩次都是 UB
    * 注意上面說的"兩次"，如果 ptr1 跟 ptr2 都指向同一個 object，這時先 delete ptr1 再 delete ptr2，就會是 UB
    ```C++
    int i, *pi1 = &i, *pi2 = nullptr; double *pd = new double(33), *pd2 = pd; delete i;
    // error: i is not a pointer
    delete pi1; // undefined: pi 1refers to a local
    delete pd; // ok delete pd2;
    // undefined: the memory pointed to by pd2 was already freed
    delete pi2; // ok: it is always ok to delete a null pointer
    ```
* const object 當然也可以被 delete
    ```C++
    const int *pci = new const int(1024);
    delete pci; // ok: deletes a const object
    ```
    
#### Dynamically Allocated Objects Exist until They Are Freed
* A dynamic object managed through a built-in pointer exists until it is explicitly deleted.
* Functions that return pointers (rather than smart pointers) to dynamic memory **put a burden on their callers—*the caller must remember to delete the memory:***
    ```C++
    // factory returns a pointer to a dynamically allocated object
    Foo* factory(T arg) {
        // process arg as appropriate
        return new Foo(arg); 
        // caller is responsible for deleting this memory
    }
    ```
    
* Unfortunately, all too often the caller forgets to do so:
    ```C++
    void use_factory(T arg) {
        Foo *p = factory(arg);
        // use p but do not delete it
    }// p goes out of scope, but the memory to which p points is not freed!
    ```
    * In particular, when a pointer goes out of scope, **nothing happens to the object to which the pointer points.**
    * Once use_factory returns, **the program has no way to free that memory.**
    * 至少要改成這樣這 program 才不會 memory leak
    ```C++
    void use_factory(T arg) {
        Foo *p = factory(arg);
        // use p
        delete p; // remember to free the memory now that we no longer need it
    }
    ```
    * 或者把指標的 value 傳回去
    ```C++
    Foo* use_factory(T arg) {
        Foo *p = factory(arg);
        // use p
        return p; // caller must delete the memory
    }
    
#### Resetting the Value of a Pointer after a delete ...
#### ...Provides Only Limited Protection
* dangling pointers
    * After the delete, the pointer becomes what is referred to as a dangling pointer.
        * A dangling pointer is one that refers to memory that once held an object but no longer does so.
        * dangling ptr 就是指向一個已經被 delete 的物件的指標
    * dangling ptr 跟未初始化的 ptr 一樣，由他來 access 物件全部都是 UB
    * 一個簡單的小解法是，在 ptr 要 out of scope 的前一個 delete object，這樣就不會有使用到 dangling ptr 的問題，因為一旦 delete 之後 ptr 就消失了
    * 如果不希望 ptr 被刪掉，那可以在 delete 之後讓 ptr = nullptr，然後要使用 ptr 之前都確定他是否為 NULL
* 但是! there can be several pointers that point to the same memory.
    * N 個指標指向同一塊 heap oject，只要有一個 delete object，其他 ptr 全部都會變成 dangling ptr
    ```C++
    int *p(new int(42)); // p points to dynamic memory
    auto q = p; // p and q point to the same memory
    delete p; // invalidates both pand q
    p = nullptr; // indicates that p is no longer bound to an object
    ```
* In real systems, finding all the pointers that point to the same memory is surprisingly difficult.

### 12.1.3 Using shared_ptrs with new
![](https://i.imgur.com/5XqUqmg.png)

* We can also initialize a smart pointer from a pointer returned by new:
    * 簡而言之你可以用 new 回傳的 pointer 來初始化 smart pointer
    ```C++
    shared_ptr<double> p1; // shared_ptr that can point at a double 
    shared_ptr<int> p2(new int(42)); // p2 points to an int with value 42
    ```
    * The smart pointer constructors that take pointers are **explicit**
        * must use the direct form of initialization
    ```C++
    shared_ptr<int> p1 = new int(1024); // error: must use direct initialization
    shared_ptr<int> p2(new int(1024)); // ok: uses direct initialization
    ```
    * For the same reason, a function that returns a shared_ptr cannot implicitly convert a plain pointer in its return statement:
    ```C++
    shared_ptr<int> clone(int p) {
        return new int(p); // error: implicit conversion to shared_ptr<int>
    }
    shared_ptr<int> clone(int p) {
        // ok: explicitly create a shared_ptr<int>from int*
        return shared_ptr<int>(new int(p));
    }
    ```
    * 直接用 make_shared 不是比較好ㄇ...
    * **還有如果用 raw ptr 來初始化 shared ptr，raw ptr 一定要已經指向一怪 new 出來的 object**

* Primet p.464 中間還有講一個很特別的東西，就是你 shared_ptr 指向的其實是一個 raw pointer 的時候，你不能單純用 delete 在 shared_ptr 底層的 ptr 上阿! 因為這時候他指向的是一個 pointer，而不是一個 new 出來的 object(或者可能有更複雜的情況之類的)，這時候你可以用一些 member function 跟  shared_ptr 說當要刪除資源的時候要用什麼方法刪除，而不是單純 call delete，P.468 會講

#### Don’t Mix Ordinary Pointers and Smart Pointers ...
* 這裡自己看...很純
* 當一個物件已經被 shared_ptr 指向，就不要再用任何的 raw ptr 來指向那個物件了
    * 你不知道那塊物件什麼時候被 shared_ptr 釋放
#### ...and Don’t Use get to Initialize or Assign Another Smart Pointer
* The code that uses the return from get must not delete that pointer.
    * 接 C API 用的啦操
    * 其實這句話也包含了不要把 get 回傳的 raw ptr 丟給 shared_ptr，因為 shared_ptr 就是屬於會 delete ptr 的 code
* 絕對不要對 shared_ptr.get() 回傳的 raw pointer 做 delete，這樣會讓 shared pointer 維護的狀態(count阿, 他指像的物件阿之類的)直接爛掉
* 而且不要用 get 回傳的 ptr 來初始化一個新的 shared_ptr，這樣舊的 shared_ptr(被 call get 的那個) 跟新的 shared_ptr(用 get 的 raw ptr 初始化的那個)彼此並不相關，各自的 reference count 都是 1，可是卻指向相同的物件!!!!

#### Other shared_ptr Operations
```C++
p = new int(1024); // error: cannot assign a pointer to a shared_ptr
p.reset(new int(1024)); // ok: ppoints to a new object
```
* Like assignment, reset updates the reference counts and, if appropriate, deletes the object to which p points.
* The reset member is often used together with unique to control changes to the object shared among several shared_ptrs.
    * ㄜ....
    * 這段 section 不知道在寫三小

### 12.1.4 Smart Pointers and Exceptions
* In § 5.6.2 (p. 196) we noted that programs that use exception handling to continue processing after an exception occurs **need to ensure that resources are properly freed if an exception occurs.**
    * RAII concept
    ```C++
    void f() {
        shared_ptr<int> sp(new int(42)); // allocate a new object
        // code that throws an exception that is not caught inside f
    }// shared_ptr freed automatically when the function ends
    ```
    * When a function is exited, *whether through normal processing or due to an exception*, all the local objects are destroyed.
        * 管你是正常結束還是噴 exception，local object 都會先消滅掉
        * 所以如果 local object 是 shared_ptr，他就會檢查 reference count，如果是 0，就 delete object
    * 上面的例子，sp 是唯一指向 int 物件的 shared_ptr，所以把 sp 破壞掉之後物件就會 free

* In contrast, memory that we manage directly is not automatically freed when an exception occurs.
    * 如果你只是用 raw pointer 的話，exception 發生就完ㄌ!
    * C++11 之前大概是把 raw_pointer 包起來在 destructor 裡面 free 吧

    ```C++
    void f() {
        int *ip = new int(42); // dynamically allocate a new object
        // code that throws an exception that is not caught inside f
        delete ip;
        // free the memory before exiting
    }
    ```
    * If an exception happens between the new and the delete, *and is not caught inside f*, then this memory can never be freed.
    
#### Smart Pointers and Dumb Classes
* 簡單說你寫了一個像 C strcut 的 class，然後裡面會有 ptr 去 new 東西的時候，他一樣會發生 exception 導致 memory leak 的情況
    * 因為他沒有一個OK的 destructor!
* Primer 舉一個同時要給 C 跟 C++ 用的 network library 當例子
    ```C++
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
    * If connection had a destructor, that destructor would automatically close the connection when f completes.
    * However, connection does not have a destructor...
    * This problem is nearly identical to our previous program that used a shared_ptr to avoid memory leaks.
    * It turns out that we can also use a shared_ptr to ensure that the connection is properly closed.


#### Using Our Own Deletion Code
* 你其實可以讓 smart pointer 指向一個不是 new 出來的物件
    ```C++
    class_name obj;
    shared_ptr<class_name> ptr(&obj, deleter)
    ```
    * 可是你這時候就不能對那個物件直接用 delete 了，很合理吧?
    * 所以你這時必須提供一個 deleter，他是一個 function，吃一個 class_name* 參數
    * 等到 smart ptr 決定要清除資源的時候，他就不會對底層的 pointer call delete，而是把底層 pointer class_name* 丟到 deleter，讓 deleter 來清資源。
    * 當然這種 smart ptr 就不能吃 make_shared 回傳的物件惹..

* 最後是用 smart pointer 時的注意事項(convention)
![](https://i.imgur.com/BPJqeEN.png)

    
### 12.1.5 unique_ptr
![](https://i.imgur.com/wTopEd9.png)

* A unique_ptr “owns” the object to which it points. **Unlike shared_ptr,only one unique_ptr at a time can point to a given object.**
* The object to which a unique_ptr points is destroyed when the unique_ptr is destroyed.
* 首先，沒有 maked_unique 這種東西
    * 錯! C++14 就有惹，Primer 老ㄌQQ
    * ***所以下面的 code example 如果有用 raw ptr 初始化 unique_ptr 的記得要想成是用 make_unique***
* unique_ptr 只能用 new expression 來初始化，*explicit*
    ```C++
    unique_ptr<double> p1; // unique_ptr that can point at a double 
    unique_ptr<int> p2(new int(42)); // p2 points to int with value 42
    ```
* 阿就說只能有一個 unique_ptr 指向某個物件惹，所以 unique_ptr 當然不支援 copy 或 assign 阿
    ```C++
    unique_ptr<string> p1(new string("Stegosaurus"));
    unique_ptr<string> p2(p1); // error: no copy for unique_ptr
    unique_ptr<string> p3;
    p3 = p2; // error: no assign for unique_ptr
    ```
* Although we can’t copy or assign a unique_ptr, **we can transfer ownership** from one (nonconst) unique_ptr to another by calling release or reset:
    * 經過 transfer ownership 之後還是只會有一個 unique_ptr 指向物件
    ```C++
    // transfers ownership from p1(which points to the string Stegosaurus) to p2
    unique_ptr<string> p2(p1.release()); // release makes p1 null 
    unique_ptr<string> p3(new string("Trex"));
    // transfers ownership from p3 to p2
    p2.reset(p3.release()); // reset deletes the memory to which p2 had pointed
    ```
    * release 是 return raw pointer 喔
        * 這樣才合理阿，如果他 return unique_ptr 不就又變成 copy
    * The release member returns the pointer currently stored in the unique_ptr and makes that unique_ptr null. Thus, p2 is initialized from the pointer value that had been stored in p1 and p1 becomes null.
    * Calling release breaks the connection between a unique_ptr and the object it had been managing.
    * **Often the pointer returned by release is used to initialize or assign another smart pointer.**
    * 接住 release 回傳值的人就要負責 free，如果是用 smart_ptr 去接就交給 smart_ptr，如果是用 raw ptr，program 就要自己 free
        ```C++
        p2.release(); // WRONG: p2 won’t free the memory and we’ve lost the pointer 
        auto p = p2.release(); // ok, but we must remember to delete(p)
        ```

#### Passing and Returning unique_ptrs
* 已經說了不能 copy unique_ptr，這樣要怎麼船進參數呢?
    * 錯! 如果你寫成 return unique_ptr 是合法的!
    * 這是另一種 copy 形式，13章會講
    ```C++
    unique_ptr<int> clone(int p) { // ok: explicitly create a unique_ptr<int> from int* 
        return unique_ptr<int>(new int(p));
    }
    
    unique_ptr<int> clone(int p) { 
        unique_ptr<int> ret(new int (p));
        // ...
        return ret;
    }
    
#### Passing a Deleter to unique_ptr
* 跟 shared_ptr 很類似
* However, for reasons we’ll describe in § 16.1.6 (p. 676), the way unique_ptr manages its deleter is differs from the way shared_ptr does.
* Overridding the deleter in a unique_ptr affects the unique_ptr type as well as how we construct (or reset) objects of that type.


### 12.1.6 weak_ptr
![](https://i.imgur.com/oBkUlZU.png)

* 不控制物件的 lifetime
* a weak_ptr points to an object that is managed by a shared_ptr.
    * **does not change the reference count** of that shared_ptr.
* **That object will be deleted even if there are weak_ptrs pointing to it**—hence the name weak_ptr, which captures the idea that a weak_ptr shares its object "weakly."
* we initialize it from a shared_ptr:
    ```C++
    auto p = make_shared<int>(42);
    weak_ptr<int> wp(p); // wp weakly shares with p; use count in p is unchanged
    ```
* Because the object might no longer exist, *we cannot use a weakptr to access its object directly*. To access that object, **we must call lock.**
* The lock function checks whether the object to which the weak_ptr points still exists.
    * ㄇㄉ 這什麼鳥
    * If so, lock returns a shared_ptr to the shared object.
    * As with any other shared_ptr, we are guaranteed that the underlying object to which that shared_ptr points continues to exist at least as long as that shared_ptr exists.
    ```C++
    if (shared_ptr<int> np = wp.lock()) {
        // true if np is not null
        // inside the if, np shares its object with wp
    }
    ```
    * Inside the if, it is safe to use np to access that object

#### Checked Pointer Class
* 這裡會用到之前定義的 StrBlob
* 我們要為他加上一個類似 iterator 的功能!
* 所以定義了一個 class 叫做 StrBlobPtr，可以拿來"指向 StrBlob 的 elements"
* 我們希望可以 StrBlobPtr 可以不要動到 StrBlob 內部物件的生命週期，所以 StrBlobPtr 其實內部是一個 weak_ptr
* 除此之外我們希望這個 class 可以在 user access 不存在的物件時噴 exception(用 weak_ptr.lock() 來實作))
* StrBlobPtr 實際上會有兩個 member，一個是 weak_ptr, 一個是 size_t
    * 用 iterator 的概念去想就會很好理解為什麼有這兩個 member

* 一樣有一個 check function 來確認 accesss 是否合法
    ```C++
    // StrBlobPtr throws an exception on attempts to access a non existent element
    class StrBlobPtr {
    public:
        StrBlobPtr(): curr(0) { }
        StrBlobPtr(StrBlob &a, size_t sz = 0): wptr(a.data), curr(sz) { }
        std::string& deref() const;
        StrBlobPtr& incr();
        // prefix version
    private: // check returns a shared_ptr to the vector if the check succeeds     
        std::shared_ptr<std::vector<std::string>>
            check(std::size_t, const std::string&) const;
        // store a weak_ptr, which means the underlying vector might be destroyed
        std::weak_ptr<std::vector<std::string>> wptr;
        std::size_t curr;
        // current position within the array
    };
    ```
* 兩個 cstr，一個不吃參數，把 weak_ptr 設成 null，curr 設成 0
* 一個讓 weak_ptr "指向" 一個 StrBlob(內的 data)，然後把 size curr 設成指向的 string element
* It is worth noting that we cannot bind a StrBlobPtr to a const StrBlob object.
    * 因為 constructor 吃的是 nonconst 的 StrBlob 物件，所以 const 不能傳進去
    * **感覺就好像 plain iterator 不能拿來指向 const container 一樣**

* check 是拿來檢查指向的 StrBlob 的底層 data 是否還存在，如果存在則檢查 curr 是否為合法 index，如果都是就 return 一個 shared_ptr，否則就噴 exception
    * 用 lock 檢查 vector 是否還在
    ```C++
    std::shared_ptr<std::vector<std::string>>
        StrBlobPtr::check(std::size_t i, const std::string &msg) const {
        auto ret = wptr.lock();
        // is the vector still around?
        if (!ret)
            throw std::runtime_error("unbound StrBlobPtr");
        if (i >= ret->size())
            throw std::out_of_range(msg);
        return ret; // otherwise, return a shared_ptrto the vector
    }
    ```

#### Pointer Operations
* * 跟 ++
    * 還沒學 overloading，用 deref 跟 incre
    ```C++
    std::string& StrBlobPtr::deref() const {
        auto p = check(curr, "dereference past end");
        return (*p)[curr]; 
        // (*p)is the vectorto which this object points
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
    }; // The begin and end members of class StrBlob should be defined outside the class body. They can't be defined until class StrBlobPtr is complete.
    StrBlobPtr StrBlob::begin() { return StrBlobPtr(*this); } 
    StrBlobPtr StrBlob::end() { return StrBlobPtr(*this, data->size()); }
    ```
    * 還會給 begin 跟 end，自己研究一下
    * 注意 begin 跟 end 不能**定義**在 StrBlob 裡面，除非 StrBlobPtr **定義**好了
    * 你看一下 class 內用到的東西就會發現一定要先宣告 StrBlobPtr 再定義 StrBlob 才能再定義 StrBlobPtr，可是 StrBlob 的 member function 又會用到 StrBlobPtr 的 object，所以 StrBlob 的這些用到 StrBlobPtr 的 member function 只能定義在 class 外面
        * 如果對我講的一頭霧水要再回去看第七章


## 12.2 Dynamic Arrays

* 很多程式需要一次動態配置多個連續相同物件的能力
    * string, vector 等等
* C++ 提供兩種解法
    * 另一種 new expression
    * template `allocator`

* 其中 `allocator` 提供了更好且更彈性的效能

* 沒事不要自己配置記憶體，太ㄎㄧㄤ，請愛用 STL

### 12.2.1 new and Arrays
* 用 `int new obj_type[num];`
    ```C++
    // call get_sizeto determine how many ints to allocate 
    int *pia = new int[get_size()]; // pia points to the first of these ints
    ```
    * 注意，這種形式的 new 的 return type 還是 pointer to first element，**不是 built-in array**
    * 因為這個原因，你不可以用 C++11 的 begin end 來取得他們的頭尾
    * 同樣也不能在 range for 使用 `new obj_type[num]` return 回來的指標
    * **num 必須是 integral type，但不用是 constexpr**
* 覺得麻煩可以結合 type alias
    ```C++
    typedef int arrT[42]; // arrT names the type array of 42 ints
    int *p = new arrT; // allocates an array of 42 ints; p points to the first one
    ```
    * 這時候 code 就算沒有 \[]，compiler 還是會用 new \[] 的版本來配置

#### Allocating an Array Yields a Pointer to the Element Type
* 雖然我們都說 new T\[] 回傳的是動態**陣列**，但**嚴格來說他回傳的不是陣列**
    * 因為他回傳回來的就不是 array type object，而是 element type
* 你只能說他 allocate 一大塊 memory，不能說它是一個 array
* **Because the allocated memory does not have an array type, we cannot call begin or end (§ 3.5.3, p. 118) on a dynamic array.**
    * 之前講過了，不能用 begin end，還有 range for

#### Initializing an Array of Dynamically Allocated Objects
* 預設就是用 default initialize
* 如果要值初始化，要加一個空括號:
    ```C++
    int *pia = new int[10]; // block of ten uninitialized ints 
    int *pia2 = new int[10](); // block of ten ints value initialized to 0
    string *psa = new string[10]; // block of ten empty strings
    string *psa2 = new string[10](); // block of ten empty strings
    ```
* C++11 允許使用 list initialization:
    ```C++
    // block of ten ints each initialized from the corresponding initializer
    int *pia3 = new int[10]{0,1,2,3,4,5,6,7,8,9};     
    // block of ten strings; the first four are initialized from the given initializers
    // remaining elements are value initialized
    string *psa3 = new string[10]{"a", "an", "the", string(3,'x')};
    ```
* 這裡提一個動態靜態的概念，因為 allocated 的大小是動態才能知道的，所以 compiler 無法推論 initializer 的數量是否超過 allocated 大小；如果超過 allocated 的大小，那 new 就會 fail，不會配置任何空間，並且 `throw bad_array_new_length`
* 還有，new \[] 的語法限制，\[] 如果要給括號也只能給空括號，裡面不能給 initializer(要給 initializer 只能用上面的方法)。
* **因為無法給 initializer 也就代表不能用 auto 來 new**

#### It Is Legal to Dynamically Allocate an Empty Array
* new T\[num] 可以給 0，而且不會出錯，但是 return 的指標不可以 dereference，是 UB
    ```C++
    size_t n = get_size(); // get_size returns the number of elements needed
    int* p = new int[n]; // allocate an array to hold the elements
    for (int* q = p; q != p + n; ++q)
        /* process the array */;
    ```
    * 按上面這樣寫，n == 0，for 一次都不會執行
* 注意真正的 array type 不能宣告成 size 為 0:
    ```C++
    char arr[0]; // error: cannot define a zero-length array
    char *cp = new char[0]; // ok: but cpcan’t be dereferenced
    ```
* When we use new to allocate an array of size zero, **new returns a valid, nonzero pointer**.
    * 還有，根據 stackoverflow 查的，你還是要對這種 size 為 0 的 new\[] 做 delete!

#### Freeing Dynamic Arrays
* 對那些接受 `new T\[]` 的指標 p ，需要用 `delete [] p;` 來歸還記憶體
* **對普通的 `new` 使用 `delete []` 或 vise versa 都是 UB**
* 還有一點，delete 一個動態陣列，elements 是反向破壞，亦即先破壞最後一個 and so on
* 還有，如果你是 new 一個 **type alias，which 其實是 array，你一樣要用 delete []，否則也是 UB**
    ```C++
    typedef int arrT[42]; // arrT names the type array of 42 ints
    int *p = new arrT; // allocates an array of 42 ints;
    p points to the first one delete [] p; // brackets are necessary because we allocated an array
    ```
    
#### Smart Pointers and Dynamic Arrays
* unique_ptr 有一種版本是拿來指向 dynamic array 的，也就是角括號內的 type 要額外加上 \[]:
    ```C++
    // up points to an array of ten uninitialized ints 
    unique_ptr<int[]> up(new int[10]);
    up.release(); // automatically uses delete[]to destroy its pointer
    ```
    * 這樣在 release 時就會 call delete \[] 而不是 delete。

* 使用這種版本的 unique_ptr，它的可用操作跟一般版本有些不同:
    * ![](https://i.imgur.com/RNMBPcM.png)
    * 不能用 dot 跟 ->，但可以用 \[]
    ```C++
    for (size_t i = 0; i != 10; ++i)
        up[i] = i; // assign a new value to each of the elements
    ```

* shared_ptr 就沒有 array 版本ㄌ，你如果要用 shared_ptr 指向一個動態陣列，你必須提供 deleter 來做 `delete []`
    ```C++
    // to use a shared_ptr we must supply a deleter 
    shared_ptr<int> sp(new int[10], [](int *p) { delete[] p; });
    sp.reset(); // uses the lambda we supplied that uses delete[] to free the array
    ```
    * 這裡直接用一個會 call delete\[] 的 lambda
* 如果你沒有提供 delete，而且 shared_ptr 又指向一個動態陣列，就會是 UB

* 如果你真的用 shared_ptr 來管裡動態陣列，你必須這樣存取這個陣列:
    ```C++
    // shared_ptrs don’t have subscript operator and don’t support pointer arithmetic
    for (size_t i = 0; i != 10; ++i)
        *(sp.get() + i) = i; // use getto get a built-in pointer
    ```
    * 必須拿 shared_ptr 內的 raw pointer 來存取每個 element
        * 幹既然都用 raw pointer 了就直接用 \[] 就好了阿zzz

### 12.2.2 The allocator Class
* new 的缺點就是在配置空間時**同時**初始化物件
* delete 也是在**破壞物件**時同時歸還空間
* 這樣不夠彈性，例如 `new T[n]` 就會一次創造 n 個物件，可是有時候我們說不定根本用不到 n 個物件，這樣沒用到的物件就浪費時間初始化了
* 而且 new 也只能 default 初始化，**這意味著沒有 default cstr 的物件不能 new!!!**
* allocator 讓我們 decouple memory allocation from object construction，將記憶體配置跟建構物件這兩件事分離

* 來看 new \[] 很沒效率的例子:
    ```C++
    string *const p = new string[n]; // construct nempty strings
    string s;
    string *q = p; // q points to the first string
    while (cin >> s && q != p + n) 
        *q++ = s;    // assign a new value to *q
    const size_t size = q - p; //  remember how many stringswe read
    // use the array
    delete[] p; // p points to an array; 
    ```
    * 我們雖然配置 n 個 string，可是我們不一定會 n 個都使用
    * 還有，我們真的有使用的(小於 n 個)strings，在 new 初始化之後我們又賦值一次，這樣就給了兩次值

#### The allocator Class
![](https://i.imgur.com/XLwobHP.png)
* 定義在 \<memory>
* 介紹怎麼用，13.5 會有一個更完整的例子
* template
* 定義 allocator 一定要給 type，代表他可以配置的物件的型態
* 當 allocator 配置記憶體時，他會根據給訂型態的大小跟 OS 要記憶體以及做適當的 memory alignment(我是不懂這裡 Primer 提 alignment 要幹嘛啦，如果不懂的話，這是計組的東西)
    ```C++
    allocator<string> alloc; // object that can allocate strings
    auto const p = alloc.allocate(n); // allocate nunconstructed strings
    ```
    
#### allocators Allocate Unconstructed Memory
* The memory an allocator allocates is *unconstructed*.
* 要使用 allocator 分配的記憶體內的物件，一定要先 construct
* allocator\<T>.construct，接受一個指標，跟一堆參數，指標要指向 allocator 分配的一大塊記憶體的某個位置，一堆參數是要拿來當作物件的 cstr 參數的
    * In particular, if the , object is a class type, these arguments must match a constructor for that class
    ```C++
    auto q = p; // q will point to one past the last constructed element
    alloc.construct(q++); // *q is the empty string
    alloc.construct(q++, 10, ’c’); // *q is cccccccccc
    alloc.construct(q++, "hi"); // *q is hi!
    ```
    * 看那個 `q++` 有沒有很有感覺?就是要對 allocate 回傳的一大塊記憶體掃過去一次，把物件一個一個建構
    * C++11 之前，construct 只吃兩個參數，一個是指標，一個是 value，呼叫 copy cstr 來複製這個 value

* It is an error to use raw memory in which an object has not been constructed:
    ```C++
    cout << *p << endl; // ok: uses the string output operator
    cout << *q << endl; // disaster: q points to unconstructed memory!
    ```

* 用完 construct 之後的物件，一定要對他 call `destroy`:
    ```C++
    while (q != p)
        alloc.destroy(--q);
    // free the strings we actually allocated
    ```
    * 看看那個精美的 `--q`
    * q 是指向最後一個建構的物件之後的位置，所以要 prefix--

* 當然也不可以對位建構的 element 直接 call destroy，UB
* 一旦 destroy 之後，可以呼叫 deallocate 歸還整大塊記憶體，或者 reuse，construct elements
    ```C++
    alloc.deallocate(p, n);
    ```
    * p 一定要是 allocate 回傳的指標，n 也要是當出呼叫 allocate 時對應的大小

#### Algorithms to Copy and Fill Uninitialized Memory
* `construct` 一次一個太累了，可以用下面的 function，一樣定義在 \<memory>
    * ![](https://i.imgur.com/hxO6UIE.png)

* 下面小例子，有個 vector\<int> vi，allocator 配置一塊他兩被大小的記憶體，前半塞 vi 的 elements，後半塞給定的 int value:
    ```C++
    // allocate twice as many elements as vi holds
    auto p = alloc.allocate(vi.size() * 2);
    // construct elements starting at p as copies of elements in vi
    auto q = uninitialized_copy(vi.begin(), vi.end(), p);
    // initialize the remaining elements to 42
    uninitialized_fill_n(q, vi.size(), 42);
    ```
    * uninitialized_copy 會回傳建構完的最後一個 element 之後的位置，所以可以拿這個指標來填配置記憶體後半段的部分
    * 注意，uninitialized_copy 第三個參數一定要指向一塊**未建構物件**的記憶體，否則是 UB

## 12.3 Using the Library: A Text-Query Program

* Primer 要用一個例子來總結 Primer Part II 介紹的東西
* Text-Query Program
* 給定一個 text file，user 可以查詢某個字串出現在那些行數，會全部印出來:
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
* 要開始設計一個程式的好方法是先想他需要支援什麼操作
    * 或者把 design pattern 聖經啃完
* 一旦知道需要什麼操作我們就可以開始考慮需要什麼資料結構
* 以下是 Text-query program 需要的操作:
    * 當在讀文字檔時，他必須記得每一行的行數，因為之後要印出來
        * 最簡單的方法是一次讀一行，並且帶隨一個 counter 來計算行數
    * 當輸出結果時，Program 必須有辦法找到 user 查詢的 word 出現的所有行數，以及對應的內容
    * 輸出的行數必須排序，並且沒有重複

* 用你ㄉ腦袋 reduce 一下就會發現這些需求可以輕易地用 STL 達成:
    * 用 `vector<string>` 儲存整的文字檔，一個 `string` 一行；index 直接代表行數
    * 用 `istringstream` 來處理一行文字，詳情第八章
    * 用 `set` 來存取一個 word 所出現的行數，這樣就達成排序且不重複
    * 用 `map<string, set<line_no>>` 來建構一個查找的資料結構，key 就是要找尋的 word，value 就是對應的出現行數的 set
    * 最後因為一些原因，我們會用 shared_ptr 管裡某些資料結構，之後會講
        * `line_no` 是自定義的型態，實際上就是 size_type 

#### Data Structures
* 我們當然可以把整個程式邏輯直接使用 STL 幹出來，可是如果把 STL 用 class 包起來，把 STL 給抽象掉，這樣在使用這個 class 時想的就會是 text-query 的操作，而不是 STL 的操作。更甚者，要擴充程式也比較容易(15 章會繼續擴充)
    * 總之就是一種 design pattern 的概念

* 定義一個 class，專門用來查找 word
    * `TextQuery`，裡面有一個 `vector` 跟 `map`
    * `vector` 存著由檔案建構而成的 string，`map` 則是拿來查找字串出現的行數的資料結構

    * API:
        * 總共有一個 cstr，吃 ifstream 來建構 `vector` 跟 `map`
        * 還有 query function，找字串，會回傳一個 query object，之後會詳細說明
            * 其實就是把 query 所需要的詳細結果都包起來的 class，要找的字串啊，行號啊，文件內容啊
            * 注意 query object **只是概念上有這些東西**，實際上之後會看到，是用 `shared_ptr` 指向 `TextQuery` object 內部的 member 

    * 上面說的 query object 定義為 `QueryResult`，hold results of a query；還有一個 print function 可以印出結果

#### Sharing Data between Classes
* 我們的 `QueryResult` 要存的東西一開始是存在 `TextQuery` object 裡面的(就是存著文字檔的 `vector` 以及存著轉換結果的 `map`)，既然如此我們就要決定怎麼在 `QueryResult` 物件裡存取 `TextQuery` 的內容
* 我們可以直接 copy `vector` 跟 `map` 內對應 word 的 `set`，可是 copy `set`，不便宜，copy `vector` 更貴
* 我們也可以用 iterator 指向這些物件，**可是這樣就要考慮到 iterator 什麼時候會 invalid 的問題**，就算 query 操作沒有使用 iterator 改變 size，但**如果原本指向的 TextQuery 物件 out of scope，那在 QueryResult 內的 iterator 就會 invalid 了**
* 上面的好解法是用 `shared_ptr`，因為概念上我們就是讓 `TextQuery` 跟 `QueryResult` 共用資料，而且又要**同步這些內部資料的 lifetime**，既然如此就用 `shared_ptr`

#### Using the TextQuery Class
* 又一個 Design Practice，當我們要定義一個 class，一個好方法是先用這個 class 寫一個 application:
    ```C++
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
* 概述跟之前一樣，不過再強調一次要考慮到 `TetQuery` 跟 `QueryResult` 共用資料的事情
* 他們會共用 `vector<string>`，保存文字檔，以及 `set<line_no>`，保存某個 word 出現的行數

* 所以這樣設計 data members:
    * `shared_ptr<vector<string>>`，指向存著文字檔的 `vector`
    * `map<string, shared_ptr<set<line_no>>>`，一個 map，mapping 到動態配置的 set，set 一樣用 shared_ptr 指著
    * line_no，只是一個自定義的型態，實際上是 size_type，用他來做 `vector<string>` 的 index
    ```C++
    class QueryResult; // declaration needed for return type in the query function
    class TextQuery {
    public:
        using line_no = std::vector<std::string>::size_type; 
        TextQuery(std::ifstream&);
        QueryResult query(const std::string&) const;
    private:
        std::shared_ptr<std::vector<std::string>> file; // input file
        // map of each word to the set of the lines in which that word appears
        std::map<std::string,
        std::shared_ptr<std::set<line_no>>> wm;
    };
    ```
    * 因為會把 class definition 寫在 header file，所以這邊不用 `using` 防止 include header 的 code 發生 name collision

#### The TextQuery Constructor
* 吃一個 ifstream&，把檔案建構成 map
    ```C++
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
    * cstr 會配置一個 vector( `file(new vector<string>>`)
    * 把檔案一行一行 push 到 vector (`file->push_back(text);`)
        * 注意要用 ->，因為 file 是 shared_ptr
    * 再來用 `istringstream` 把單字一個一個丟到 map
        * 注意這邊用 reference bind `word` 對應的 value，也是一個 shared_ptr
        * 記得如果對 map 用 \[]，但是沒有對應 element 時，會 default 初始化，所以如果當前 `word` 沒有出現在 map，就會創造一個新的 pair，pair.second 是一個**指向 null 的 shared_ptr**
        * 所以才要確認 shared_ptr，也就是 `lines` 是否為 null，如果是就要先 reset，要一塊 set 出來
        * 最後再把行號插到 set 內，如果已經有對應的行號，insert 就不做事

#### The QueryResult Class
* 三個 data members:
    * `string`，要找尋的字串
    * `shared_ptr<vector<string>>`，指向存著文字檔的，跟 `TextQuery` 共用的 vector
    * `shared_ptr<set<line_no>>`，指向共用的 `TextQuery` 內的 map 內，對應的 shared_ptr 指向的 ser
    * 如果你覺得我這樣寫很雞巴，你就看看原文寫ㄉㄅ，或者看 code

* 只有一個 cstr，吃三個參數，分別把上面三個 members 初始化
    ```C++
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
    
#### The query Function
* 吃一個 `string`，找出 `TextQuery` 內 word map 內對應的 set
* 找到之後，用 `TextQuery` 對應的 members 來建構一個 `QueryResult` 並回傳
    * 如果沒找到 string 呢?這時候對應的 set 不存在啊?
    * solve this problem by **defining a local static object that is a shared_ptr to an empty set of line numbers.**
    * 如果沒找到字串，回傳的 `QueryResult` 物件內指向 set 的指標就複製這個 static member
    ```C++
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
    * 注意要用 find，不能用 \[]，不然會插入不存在的字串到 map!

#### Printing the Results
```C++
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
* 注意，我們存的 line_no 都是 0-based，一般文字行號都是 1-based，所以印出行號時要額外 + 1() (`os << "\t(line " << num + 1 << ") ";`)
* 直接用 set\<T>.size 得知 word 總共在多少行出現過
* 還有一個地方，用 `*(qr.file->begin() + num)` 取得對應行數的字串
* 都講完ㄌ，不過有個地方可以注意，就算找不到字串，我們的 class 也不會出問題，會印出「word 出現 0 次」並且 `print` 內的 range for 一次也不會執行(因為 set 是 empty set)