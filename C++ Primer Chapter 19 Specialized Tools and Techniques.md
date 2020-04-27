---
tags: C++
---

# C++ Primer Chapter 19 Specialized Tools and Techniques

## 19.1 Controlling Memory Allocation
* 某些程式希望自己控制 memory allocation 的機制，標準提供的方法無法滿足他們的需求
* 例如
    * 自行管理 `new` 配置空間的邏輯，將 object 放到特殊的 memory 之類的
    * C++ 允許 *overload* `new`/`delete` operator 來達成這個功能
        * 這個 `new`/`delete` operator 跟 12 章初步介紹的 `new`/`delete` 有所不同，之後會介紹

### 19.1.1 Overloading `new` and `delete`
* 這個 *overload* 實際上跟 14 章的 opeator overloading 的概念不太一樣
* 需先了解 `new` 跟 `delete` 更細節的機制
* 看 code:
```cpp
// new expressions
string *sp = new string("a value"); // allocate and initialize a string
string *arr = new string[10]; // allocate ten default initialized strings
```
* 上面例子，當寫一個 `new` expression 時，**實際上會執行三個步驟**
1. 呼叫 `operator new` 或 `operator new[]` 這兩個 library *function*
    * 它們會配置一塊足夠可以塞下 `new` expression 指定的(多個)物件的空間
    * **它們也只做這件事**，並不會對這塊(raw)空間做額外處理
2. C++ 在這個空間呼叫 `new` expression 需要的合適的 ctor，初始化該空間
3. 回傳指向這塊空間的指標

* 而當寫一個 `delete` expression 時會執行兩個步驟:
```cpp
delete sp; // destroy *sp and free the memory to which sp points
delete [] arr; // destroy the elements in the array and free the memory
```
1. C++ 在 `delete` expression 指向的空間執行合適的 dtor
    * 對，合適的 dtor，還記得 `virtual` dtor 吧
3. `delete` expression 呼叫 library *function* `operator delete` 或 `operator delete[]` 來釋放空間

:::warning
* [operator new](https://en.cppreference.com/w/cpp/memory/new/operator_new)
* [operator delete](https://en.cppreference.com/w/cpp/memory/new/operator_delete)
* 他們真的就是 function
* **return `void*`**
    * 可參考 `malloc`
:::

#### 知道這些步驟，然後呢?
* 只要 overload `operator new`/`operator delete` 等等 這些 library function，就可以自己控制要怎麼配置/釋放空間
* We can define our own versions of them and the *compiler won't complain about duplicate* definitions
    * Instead, the compiler will use our version in place of the one defined by the library.

:::danger
* Primer p.821
* 我們如果自定義(overload) global `operator new`/`operator delete[]` 等等，所有的 `new`/`delete` expression 都會改用自定義的版本
    * 這個定義必須要是正確的，比方說一定要回傳配置的空間，或者釋放空間
    * 沒做到的話 compiler 可能會噴 warning
* 後面會提到，其實可以不要定義在 global
:::

* 實際上 `operator new` 等 library function 可以定義在 global scope
    * 或者定義成 class (implicit `static`) member function
* 當 code 出現 `new`/`delete` expression 時，會根據 expression 來找尋對應的 `operator new`/`delete` 定義
    * 如果 expression 是 class type，則從 class scope 開始找
        * *包含所有 base classes*
        * 找到就用 class 定義的
        * 沒找到就往 global scope 找
    * 如果有自定義 `operator` `new`/`delete` 在 global scope 則就使用自定義版本
    * 若沒有就用 library 提供的

:::info
* 若想直接跳過 class scope 用 global scope，則一樣可以使用 scope operator
```cpp
::new
::delete
```
:::

#### The `operator new` and `operator delete` Interface
* library 總共定義八個 overloads
```cpp
// these versions might throw an exception
void *operator new(size_t); // allocate an object
void *operator new[](size_t); // allocate an array
void *operator delete(void*) noexcept; // free an object
void *operator delete[](void*) noexcept; // free an array

// versions that promise not to throw; see § 12.1.2 (p. 460)
void *operator new(size_t, nothrow_t&) noexcept;
void *operator new[](size_t, nothrow_t&) noexcept;
void *operator delete(void*, nothrow_t&) noexcept;
void *operator delete[](void*, nothrow_t&) noexcept;
```
* 前四種是會 throw `std::bad_alloc` 的版本，後四種不是
    * 雖然這樣講，但是 `operator delete` 的兩種版本跟 class dtor 一樣保證不 throw 啦，只是兩種版本 signature 有差

* 程式可以自定義上面的任何一種版本
    * 之前講過，要馬定義在 global 要馬定義成 member function
* **注意，定義成 member function 時，這個版本是會被當成 `static` member function 的**
    * 定義時有沒有寫 `static` 都可，總之一定會是 `static`
* 原因如下:
    * `operator new`/`delete` 只有在*物件被初始化之前*或*物件被破壞完之後*時才會被呼叫，在這個情境下根本不會有合法物件的存在，所以沒有必要定義成一般的 member function(然後透過 `this` 來操作 class member)

* 另外，`operator new`/`delete` 一定要 return `void*`，然後第一個參數一定要是 `size_t`
    * 可以定義 return 別的 type 試試，無法編譯
    * `size_t` 不能有 default argument

* 然後... `operator new`/`delete` 剩下的參數隨便你!
* 例如可以定義:
```cpp
void *operator new(size_t, int);
```
* 但是要讓 `new` expression 可使用這種版本的 `operator new`，則需要使用 placement `new`(12.1.2 p. 460 && 19.1.2 p. 823)
    * 其實沒什麼特別的，user code 需要使用不會 throw 的 `operator new` 也是需要使用 placement `new`
* placement `new` 做的其實就是把傳給他的 argument 丟給 `operator new` 額外的參數
:::info
[new expression #syntax](https://en.cppreference.com/w/cpp/language/new#Syntax)
* 可看官方說明，有一個 `placement_params`
:::

:::warning
* 雖然定義 `operator new` 時可以定義額外參數，**但是以下版本的 `operator new` 是標準留給 library 定義的，程式不能自己定義:**
```cpp
void *operator new(size_t, void*); // this version may not be redefined
```
* 19.1.2 會講這個版本的 `operator new` 在幹嘛
* 另外我自己試著重定義這個版本，g++ 噴的 error 比較不...正規，跟一般的 function 出現 redefinition 差不多
```cpp
#include <new>

void *operator new(size_t, void*) {
    return nullptr;
}

int main() {
    return 0;
}
```
```shell
main.cpp:3:7: error: redefinition of ‘void* operator new(size_t, void*)’
 void *operator new(size_t, void*) {
       ^~~~~~~~
In file included from main.cpp:1:
/usr/include/c++/8/new:168:14: note: ‘void* operator new(std::size_t, void*)’ previously defined here
 inline void* operator new(std::size_t, void* __p) _GLIBCXX_USE_NOEXCEPT
              ^~~~~~~~
main.cpp: In function ‘void* operator new(size_t, void*)’:
main.cpp:4:12: warning: ‘operator new’ must not return NULL unless it is declared ‘throw()’ (or -fcheck-new is in effect)
     return nullptr;

```
:::

* `operator delete` 則是
    * 一定要 return `void`
    * 第一個參數一定要吃 `void*`，也就是要被釋放的空間

* 如果 `operator delete`/`delete[]` 定義成 class member。則可以給第二個參數 `size_t`
    :::danger
    * 這裡感覺很恐怖，可能需要去找更詳細的解釋
    * The `size_t` parameter is **used when we delete objects that are part of an inheritance hierarchy.**
    * 不太確定實際上會被怎麼用..
    * If the base class has a **virtual destructor (§ 15.7.1, p. 622),** then *the size passed* to `operator delete` will **vary depending on the dynamic type of the object** to which the deleted pointer points.
    * **version** of the `operator delete` function that is run **will be the one from the dynamic type** of the object.
    :::

#### The `malloc` and `free` Functions
* 這個不用介紹吧...
* 從 C 來的
* `operator new`/`delete` 可用 `malloc`/`free` 實作
:::info
`std::allocator` 有可能是用 `malloc`/`free` 實作的
:::

### 19.1.2 Placement `new` Expressions
:::info
* 參考: https://blog.csdn.net/tennysonsky/article/details/74169847
    * 這邊 Primer 有講跟沒講一樣...
:::
* C++11 前還沒有 `std::allocator`
* 那時 user code 想要自訂空間如何配置都是使用 `operator new`/`delete`
* 但是有個問題，`operator new`/`delete` 沒有像是 `std::allocator::construct`(注意 C++17 之後這東西 deprecated，雖然我還是不知道為什麼)，可以藉由傳遞參數呼叫合適的 class ctor
* 這時要能透過 `operator new`/`delete` 呼叫適合的 ctor 只能用 placement `new` 了(p. 460, 12.1.2)

* 形式如下:
    * ![](https://i.imgur.com/6errgvv.png)
    * `place_address` 要是一個指標
    * `initializers` 則是初始化被動態配置的 class 物件的參數
    * 然後 placement `new` 用 `initailizers` 在該指標指向的空間初始化物件
* 先給例子:
    ```cpp
    // 範例改自: https://blog.csdn.net/tennysonsky/article/details/74169847
    class Foo
    {
        int Member_;
    };

    int main() {
        char* buff = new char[sizeof(Foo) * N];
        memset(buff, 0, sizeof(Foo) * N);
        Foo* pfoo = new (buff) Foo; // ***placement new***
    }
    ```
    * `main` 最後一行，直接將 `buff` 當作已配置好的空間丟給 placement `new`
    * 這個 `new` expression 會**直接在這塊空間初始化物件，而不是自己動態配置一塊空間**

:::warning
* 我覺得這裡 Primer 又用那種很不負責任的解說方式...
    * 例如直接先說，用什麼方式呼叫 placement `new` 會發生什麼事，卻連個 code 都還沒看到
    * 這個 section 甚至沒有練習 XD
:::

* 總之，如果呼叫 placement `new` 時，如果只有給定一個(指向某個空間的，例如上面例子的 `buff`)指標，沒有額外給任何參數，則 placement `new` 就會呼叫
```cpp
operator new(size_t, void*)
```
* 來 *「配置」空間*
    * **仔細看 signature**
    * 這是 19.1.1, p.822 時說的不能 redefine 的 `operator new`
    * **實際上這個 function 不會真的配置空間**
        * 它單純只是回傳它的 pointer argument
    * 整個 new expressions 最後會直接在這個 pointer 初始化物件
* 上面例子最後一行就是這個例子
```cpp
Foo* pfoo = new (buff)Foo; // placement new
```
* 說真的這樣我更不明白這個不能重定義的 function 是啥來幹嘛的了
:::info
* 雖然看起來 placement `new` 跟 `std::allocator` 有點像，但有幾個差異:
    1. 丟給 `allocator::construct` 的指標指向的空間一定要是用同個 allocator 配置的，可是丟給 placement `new` 的就不需要
    2. 甚至看看上面的例子，丟給它的位址甚至不是動態配置的!
        * 但是 Primer 說 19.6 會才會看到這個例子
        * 難道不覺得這編排有點詭異?
:::

#### Explicit Destructor Invocation
* Just as placement `new` is analogous to using [`std::allocator<T>::allocate`](https://en.cppreference.com/w/cpp/memory/allocator/allocate), an explicit call to a destructor is analogous to calling [`std::allocator<T>::destroy`](https://en.cppreference.com/w/cpp/memory/allocator/destroy)

* 總之可以自己呼叫 dtor
    ```cpp
    string *sp = new string("a value"); // allocate and initialize a string
    sp->~string();
    ```
    * 這時候 C++ **只會破壞物件，不會對物件存在的空間做任何事**，包含釋放空間
    * 換句話說，這塊空間可以被重新使用
    * 例如，也許變成下一次 placement `new` 的空間
:::info
* 根據上面貼的參考文章，自己這樣控制物件的空間，*可能*可以幫助減緩記憶體碎片化的問題(external fragmentation)，詳情 OS 課
* 但切記不要過早最佳化
:::
