---
tags: C++
---

# C++ Primer Chapter 9 Sequential Containers
* 把 sequential containers 講完
* The container classes share a common interface, which each of the containers extends in its own way.
* container 有著共同的 interface，然後再各自延伸一些自己專屬的 interace
    * 共同 interface 讓 library 更好學，code 看起來更一致，觀念更密合
* Each kind of container offers a different set of performance and functionality trade-offs.
    * 雖然 interface 長一樣，不過效能跟功能還是會有差異

* A container holds **a collection of objects** of a specified type.
*  The **sequential containers let the programmer control the order in which the elements are stored** and accessed.
    * By contrast, the ordered and unordered associative containers, which we cover in Chapter 11, store their elements based on the value of a key.

* 還有一個東西叫做 container adapters，章節末會講
    * 可參考 adapter pattern

## 9.1 Overview of the Sequential Containers
* 以下要講的 container 都提供了 fast sequential access performance
* 不過其他的 operation 就有不同
    * add/delete elements to the container
    * nonsequential access

* ![](https://i.imgur.com/cJfHtcL.png)
    * With the exception of array, which is a fixed-size container, **the containers provide efficient, flexible memory management.**
        * The strategies that the containers use for storing their elements have inherent, and sometimes significant, impact on the efficiency of these operations. In some cases, these strategies also affect whether a particular container supplies a particular operation.
            * 這裡就演算法，container 到底用什麼資料結構存 element 會影響到他們拿 element 的效能跟可做的操作

* 一些小提醒，array 跟 forward_list 是 C++11
    * 其中 forward_list 沒有 size... 標準希望他跟手寫的 C linked list 一樣快，加了 size O() 不變，可是就不是 zero overhead zz

* 13 章還會講到，因為有了 C++ move constructor，STL 變得更快ㄌ，你真的沒事不要不用 STL

#### Deciding Which Sequential Container to Use
* Tip: Ordinarily, it is best to use vector unless there is a good reason to prefer another container.

* rules of thumb
    * Unless you have a reason to use another container, use a vector.
    * If your program has lots of small elements and space overhead matters, don’t use list or forward_list.
    * If the program requires random access to elements, use a vector or a deque.
    * If the program needs to insert or delete elements in the middle of the container, use a list or forward_list.
    * If the program needs to insert or delete elements at the front and the back, but not in the middle, use a deque.
    * If the program needs to insert elements in the middle of the container only while reading input, and subsequently needs random access to the elements:
        * First, decide whether you actually need to add elements in the middle of a container. It is often easier to append to a vector and then call the library sort function (which we shall cover in § 10.2.3 (p. 384)) to reorder the container when you’re done with input.
        * If you must insert into the middle, consider using a list for the input phase. Once the input is complete, copy the list into a vector.
    * 如果還有其他因素，你可以使用 11 章ㄉ


## 9.2 Container Library Overview
* The operations on the container types form a kind of hierarchy:
    * 某些 operations 是所有 container 都有
    * 某些是專屬 sequetial, associative(ch11), unorder(ch11) container 各自的
    * 剩下的就是特定 container 才有的

* 這小節先講所有 container 都有的剩下講 sequential container 有的

* In general, each container is defined in a header file with the same name as the type.
    * std::string 定義在 string，std::vector 定義在 vector
* **The containers are class templates**
    * 必須要提供 element type

* 我們可以存放幾乎任何類型的 object 到 container
    * Although we can store almost any type in a container, **some container operations impose requirements of their own on the element type**
    * 有些 container 上面可以做的 operation 會需要 element type 滿足一些條件才能用
    * 如果你的 element type 不滿足那個 operation 的需求，你還是可以定義 container<element_type> 的物件，可是這個 container object 就不能執行那個 operation，**而且你如果寫出這樣的 code，compiler 不但會噴 error，還會噴很奇怪的 error**

* 例如之前講過的，一定要提供 initializer 的 type
```cpp
// assume noDefault is a type without a default constructor
vector<noDefault> v1(10, init); // ok: element initializer supplied
vector<noDefault> v2(10); // error: must supply an element initializer
```

* Primer p.330 有一個 table 跟你說 common container operations

### 9.2.1 Iterators
* As with the containers, iterators have a common interface:
* For example, all the iterators on the standard container types let us access an element from a container, and they all do so by providing the dereference operator.
* Similarly for increment operator
* Table 3.6(p.107) 有列 common interface

#### Iterator Ranges
* The concept of an iterator range is fundamental to the standard library.
* An iterator range is denoted by a pair of iterators each of which refers to an element, *or to one past the last element*, in the **same** container.
    * \[ begin, end )
    * left-inclusive interval.
* end may == begin, when container is empty
* end 可以跟 begin 指向同一個位置，可是 end 不能指向比 begin 還要前面的位置

#### Programming Implications of Using Left-Inclusive Ranges
* If begin equals end, the range is empty
* If begin is not equal to end, there is at least one element in the range, and begin refers to the first element in that range
* We can increment begin some number of times until begin == end
* 根據上面我們可以寫這樣安全又正確的 loop:
```cpp
while (begin != end) {
    *begin = val; // ok: range isn’t empty so begindenotes an element
    ++begin; // advance the iterator to get the next element
}
```

### 9.2.2 Container Type Members
* 就是第七章說的，class 內也可以宣告 type
* 目前用到的 size_type, iterator, const_iterator 都是 type name
* 還有 reverse_iterator, 會倒過來 access 的 iterator
* Briefly, a reverse iterator is an iterator that goes backward through a container and **inverts the meaning of the iterator operations.**
    * 仔細看上面講的...，例如對 reverse_iterator 做 ++，實際上會指向**前一個** element

* 剩下的是 value_type, reference, const_reference，其實就是 element type 跟 reference to element 的 type
    * template 大法好
    * These element-related type aliases are most useful in generic programs, which we’ll cover in Chapter 16.
    * 最後這三個 16章會告訴你怎麼發揮他們的功用，generic programming，很黃很暴力

* 為了使用這些 type name，我們在宣告這些 type 的物件時需要把 class scope 也打出來
    * class_name::type_name obj;
    ```cpp
    // iter is the iterator type defined by list<string>
    list<string>::iterator iter;
    // count is the difference_type type defined by vector<int>
    vector<int>::difference_type count;
    ```


### 9.2.3 begin and end Members
* yield iterators that refer to the first and one past the last element in the container.
* \[c]\[r]begin \[c]\[r]end
    * 有 r 代表 reverse，有 c 代表 const
* 其實 begin end 有做 overloading，C++11 之前會根據你 call 的 object 是某為 const 來決定要用哪種，一個會回傳 iterator，一個是回傳 const_iterator
* When we use auto with begin or end, the iterator type we get depends on the container type.
    * 現在因為用 auto，只有在你的 object 是 const 時 auto it = obj.begin(); 才會回傳 const_iterator；用 cbegin 就可以解決這個問題

    ```cpp
    // type is explicitly specified
    list<string>::iterator it5 = a.begin();
    list<string>::const_iterator it6 = a.begin();
    // iterator or const_iterator depending on a’s type of a
    auto it7 = a.begin(); // const_iteratoronly ifa is const
    auto it8 = a.cbegin(); // it8is const_iterator
    ```
* BEST PRACTICE: **When write access is not needed, use cbegin and cend.**

### 9.2.4 Defining and Initializing a Container
![](https://i.imgur.com/rnLw4T5.png)

* Every container type defines a default constructor (§ 7.1.4, p. 263).
    * 不過要你的 element 有 defalut ctor 才能用
* 也有吃 size，跟吃 size 還有 initializer 的兩種 ctor

#### Initializing a Container as a Copy of Another Container
* class_name\<T> container2(container1);
    * container type 跟 element type 都要一樣
* class_name\<T> container2(iter1, iter2);
    * 用 iterator range 初始化的版本有幾個好處
    * **container type 不用一樣**
    * **element type 一樣或者可以轉型即可**

    ```cpp
    // each container has three elements, initialized from the given initializers
    list<string> authors = {"Milton", "Shakespeare", "Austen"};
    vector<const char*> articles = {"a", "an", "the"};
    list<string> list2(authors); // ok: types match deque<string>
    authList(authors); // error: container types don’t match
    vector<string> words(articles); // error: element types must match
    // ok: converts const char*elements to string
    forward_list<string> words(articles.begin(), articles.end());
    ```

* The constructor that takes two iterators uses them to denote a range of elements that we want to copy.
* Because the iterators denote a range, we *can use this constructor to copy a subsequence of a container.*

#### List Initialization
* C++11 可以 "list initailize" 一個 container
* 除了 array 之外，initailizer list 的大小也決定了 container (動態)的 size

#### Sequential Container Size-Related Constructors
* sequential 容器可以給兩個參數，第一個是 size，第二個(optional)是每個 element 的 initializer
    * 第二個沒給 element 就 value-initailize
    * 只有 element 有 default ctor 才能不給第二個參數，講過很多次了
* 注意，associative container 並不支援單給一個 size 的 ctor

#### Library `array`s Have Fixed Size
* Just as the size of a built-in array is part of its type, the size of a library array is part of its type.
* array<int, 42>; array<string, 10>;
    * 角括號裡面除了 element type 以外還要給 size
* 你要用 array 定義的 type，你還是必須把角括號兩個參數都打出來...
    * array<int, 42>::size_type
* Because the size is part of the array’s type, array does not support the normal container constructors.
    * 如果 support 就 doesn't make sense
    * 首先那個 agrument 不用是 constant expression.. QED
* The fixed-size nature of arrays also affects the behavior of the constructors that array does define. **Unlike the other containers, a default-constructed array is not empty: It has as many elements as its size.**
* These elements are **default initialized (§ 2.2.1, p. 43) just as are elements in a built-in array**
    * initializer 數量要 <= `array` size，跟 built-in array 一樣
    * 另外 element 一定要可以 default initialize 數量才可以 <= `array` size

* 還有一點，array 可以 copy! 只要 type 完全一樣的話(i.e. 角括號內的兩個參數都一樣)
    ```cpp
    int digs[10] = {0,1,2,3,4,5,6,7,8,9};
    int cpy[10] = digs; // error: no copy or assignment for built-in arrays
    array<int, 10> digits = {0,1,2,3,4,5,6,7,8,9};
    array<int, 10> copy = digits; // ok: so long as array types match
    ```

### 9.2.5 Assignment and swap
![](https://i.imgur.com/9EzjbkA.png)

* The assignment-related operators, listed in Table 9.4 (overleaf) **act on the entire container.**
* The assignment operator *replaces the entire range of elements in the left-hand container with copies of the elements from the right-hand operand*:
```cpp
c1 = c2; // replace the contents ofc1with a copy ofthe elements in c2
c1 = {a,b,c}; // after the assignment c1has size3
```

* array 也可以做 assignment 講過ㄌ，給點例子，因為其實寫起來有點ㄎㄧㄤ
```cpp
array<int, 10> a1 = {0,1,2,3,4,5,6,7,8,9};
array<int, 10> a2 = {0}; // elements all have value 0
a1 = a2; // replaces elements in a1
a2 = {0}; // error: cannot assign to an array from a braced list
```
* array 要用 =，rhs 要一模一樣的 type，而且不能是 list_initializer
* 然後 array `assign` 系列的 API 都不支援

#### Using `assign` (Sequential Containers Only)
* The **assignment** operator requires that the left-hand and right-hand operands **have the same type.**
    * 這裡是在解釋使用 `operator=` 時的情境
* The sequential containers (except `array`) also define a member named `assign` that lets us **assign from a different but compatible type, or assign from a subsequence of a container.**
    * assign 這個 member function 就可以吃不同 container，element type 只要可轉換就行了
* The `assign` operation replaces all the elements in the left-hand container with (copies of) the *elements specified by its arguments.*

```cpp
list<string> names;
vector<const char*> oldstyle;
names = oldstyle; // error: container types don’t match
// ok: can convert from const char*to string
names.assign(oldstyle.cbegin(), oldstyle.cend());
```

#### Using `swap`
* exchanges the contents of two containers of the same type.
* After the call to swap, **the elements in the two containers are interchanged:**
    ```cpp
    vector<string> svec1(10); // vector with ten elements
    vector<string> svec2(24); // vectorwith 24 elements
    swap(svec1, svec2);
    ```
    * After the swap, svec1 contains 24 string elements and svec2 contains ten.
* **Excepting `array`**, swapping two containers is guaranteed to be fast— the **elements themselves are not swapped; internal data structures are swapped.**
* Note: Excepting `array`, swap does not copy, delete, or insert any elements and is **guaranteed to run in constant time.**

* The fact that elements are not moved means that, *with the exception of string*, iterators, references, and pointers into the containers are **not invalidated.**
    * They refer to the same elements as they did before the swap.
    * However, **after the swap, those elements are in a different container.**
    * 看上面的 code，假設有一個 iter 指向 `svec1[3]`, swap 之後他會指向 `svec2[3]`
    * 可是標準定義 string 的 swap 還是會讓這些東西失效 QQ
* swap array 真的會動到 element 惹，是 O(n)

* C++11 多了 swap function(template)
    * The nonmember swap is of most importance in generic programs. As a matter of habit, it is best to use the nonmember version of swap.
    * 用 global swap 在 generic programming 好像很重要

### 9.2.6 Container Size Operations
* size, max_size, empty
* forward_list 沒有 size，其他 container 三個 member 都有

### 9.2.7 Relational Operators
* Every container type supports the equality operators (== and !=);
* all the containers *except the unordered associative containers* also support the relational operators (>, >=, <, <=).
* lfs 跟 rhs container 跟 element type 要一樣
* Comparing two containers **performs a pairwise comparison of the elements.**

#### **Relational Operators Use Their Element’s Relational Operator**
* We can use a relational operator to compare two containers **only if the appropriate comparison operator is defined for the element type.**
* The container equality operators use the element’s == operator, and the relational operators use the element’s < operator.
    * 阿剩下的哩? 其實用 < 跟 == 配上 ! 就可以組出剩下的 relational operator 了，所以不用自己定義
    ```cpp
    vector<Sales_data> storeA, storeB;
    if (storeA < storeB) // error: Sales_datahas no less-than operator
    ```
    * 上面是錯的，因為 Sales_data 沒有定義 relational operator

## 9.3 Sequential Container Operations
* 這節介紹專屬 sequential container 的 operator

### 9.3.1 Adding Elements to a Sequential Container
* 除了 array，sequential container 都可以 add element，並且他們都自己實作了記憶體管理，下表列出有關增加元素的 operation
* ![](https://i.imgur.com/kcyG6zf.png)
* When we use these operations, we must **remember that the containers use different strategies for allocating elements** and that these strategies **affect performance**.

#### Using `push_back`
* 除了 `array` 跟 `forward_list` 都支援(包含 `string`)
* The call to `push_back` **creates a new element at the end of container,increasing the size of container by 1.**

#### Using `push_front`
* 只有 `deque` 跟 `forward_list` 可以用

#### Adding Elements at a Specified Point in the Container
* 上面兩個 operations 只能插在 container 頭尾，這個可以插在任何地方，還可以一次插多個 elements
* `insert` - supported by `vector`, `deque`, `list`, `string`
* `forward_list` 要用別的 interface(9.3.4 p.350 會講)
* `insert` 有 overload，不過第一個參數都是吃 iterator，代表要插入的位置
* element(s) are **inserted *before* the position denoted by the iterator.**
* 弔詭的是，有些 container 不支援 push_front，卻可以用 insert 插在 container 的頭
    * 其實就是 API 介面設計哲學，你如果讓 vector 跟 string 可以有 push_front，那這樣 vector 版的是 O(n)，deque 版的是 O(1)，running time 不一致
    :::warning
    It is legal to insert anywhere in a `vector`, `deque`, or `string`. However, doing so can be an expensive operation.
    :::


#### Inserting a Range of Elements
* insert 從第二個參數到最後一個參數長的跟 container constructor 參數有 87% 像
    * 吃一個數字 n，插 n 個 default initializer
    * 吃一個數字 n 跟 initializer，插 n 個 initializer
    * 吃兩個 iterator begin 跟 end，把這個 range 間的東西都插到 container 指定的位置
    ```cpp
    svec.insert(svec.end(), 10, "Anna");
    // 在 svec 最後面插10個 "Anna" string
    vector<string> v = {"quasi", "simba", "frollo", "scar"};
    // insert the last two elements of v at the beginning of slist
    slist.insert(slist.begin(), v.end() - 2, v.end());
    slist.insert(slist.end(), {"these", "words", "will", "go", "at", "the", "end"});
    // run-time error: iterators denoting the range to copy from
    // must not refer to the same container as the one we are changing
    slist.insert(slist.begin(), slist.begin(), slist.end());
    ```
    * 注意最後一個例子，你最後兩個 iter 不能指向同一個 container，這是 runtime error
    * **為什麼不行？一樣的道理，因為你插 element 會改變 container size，所以 iterator 就 fail 了**

#### Using the Return from `insert`
* Under the new standard, the versions of insert that take a count or a range **return an iterator to the first element that was inserted.**
    * 舊標準是 return `void`
    * If the range is empty, no elements are inserted, and the operation returns its first parameter.
* use the value returned by insert to repeatedly insert elements *at a specified position* in the container:
    ```cpp
    list<string> lst; auto iter = lst.begin();
    while (cin >> word)
        iter = lst.insert(iter, word);
        // same as calling push_front
    ```
    * 先不管這 code 是插在第一個位置，你可能會覺得很奇怪，就算插在第 n 個位置，我們也沒必要更新 iter 阿?
    * 不對，這個 container 是 list，你仔細想 linked-list 怎麼做的
    * 之前說過，`insert` 是**把 element 插到參數 iter 的前一個位置**，所以 `insert` 之後如果沒有更新 `iter`，`iter` 就會指向新的 element 的**後面**，這時候再用 `iter` call `insert` 就不會插到 `list` 的最前面了
    * 更 general 的 case 看下圖
    * ![](https://i.imgur.com/nLC4o3t.png)

    ```cpp
    list<string> lst; auto iter = lst.begin();
    while (cin >> word)
        lst.insert(iter, word);
    ```
    * 第一份 code input 如果給 1 2 3 4，那 lst 內容會是 4 3 2 1
    * 第二份 code 則會是 1 2 3 4，可以自己試試
* 上面的 code 如果沒有更新 `iter` 那只是順序錯誤而已，如果是 `lst` 是 `vector<int>` 則會更慘，因為 `vector<T>::iterator` 在 vector size 增長後是會失效的。
    * `list` 的 iterator 則不會失效，你自己想想 linked-list 的特性就知道了
    * https://stackoverflow.com/questions/6438086/iterator-invalidation-rules 過者參考這篇，可以看哪些 container 經過哪些操作之後 iterator 會失效

* 結論是，如果你要固定插到 container 的第 n 個位置，你每插一次新的 element(s) 你就要更新你的 iterator，可以用 `insert` 回傳的 iter 來更新

#### Using the `emplace` Operations
* C++11:
    * `emplace_front`, `emplace`, and `emplace_back`
        * that **construct rather than copy** elements.
* When we call an emplace member, **we pass arguments to a constructor** for the element type
    * 你是 call constructor! 超ㄎㄧㄤ，所以 emplace 的參數是 element type 的 ctor  的參數
        * `emplace` 是用 variadic template 做的，很後面才會講
* construct an element **directly in space managed by the container.**

    ```cpp
    // construct a Sales_data object at the end of c
    // uses the three-argument Sales_data constructor
    c.emplace_back("978-0590353403", 25, 15.99);
    // error: there is no version of push_back that takes three arguments
    c.push_back("978-0590353403", 25, 15.99);
    // ok: we create a temporary Sales_dataobject to pass to push_back
    c.push_back(Sales_data("978-0590353403", 25, 15.99));
    ```
    * 注意第一個跟最後一個的差別，第一個直接在 container 內部建構物件，最後一個則是先建構物件，再複製到 container 裡面
        * 當然最後一個 `push_back` 有可能被 RVO 優化啦，但是有太多情況會導致無法 RVO 了
        * 另一個情況就是實作 move

### 9.3.2 Accessing Elements
![](https://i.imgur.com/655pC1Y.png)
* 自己看...

#### The Access Members Return References
* 上面那個 table 都 member 全部都是 return reference
    ```cpp
    if(!c.empty()){
        c.front() = 42;
        auto &v = c.back();
        v = 1024;
        auto v2 = c.back();
        v2 = 0;
    }
    ```
#### Subscripting and **Safe Random Access**
* If we want to ensure that our index is valid, we can use the at member instead. The at member acts like the subscript operator, but if the index is invalid, **at throws an out_of_range exception (§ 5.6, p. 193):**
    ```cpp
    vector<string> svec; // empty vector
    cout << svec[0]; // run-time error: there are no elements in svec!
    cout << svec.at(0); // throws an out_of_range exception
    ```

### 9.3.3 Erasing Elements
* there are also several ways to remove elements.
* ![](https://i.imgur.com/64TtXH8.png)
* 參數給一個 iter，移除那個位置的 element
* 或者兩個 iter，把這個 range 之間的 elements 都移除
* 兩個 functions 都 return (最後一個)被移除的 element 的下一個位置
    * That is, if `j` is the element following `i`, then `erase(i)` will return an iterator referring to `j`.
        * 這邊解說有點爛，`i` 是 iterator，`j` 是 element
    ```cpp
    list<int> lst = {0,1,2,3,4,5,6,7,8,9};
    auto it = lst.begin();
    while (it != lst.end())
        if (*it % 2)            // if the element is odd
            it = lst.erase(it); // erase this element
        else
            ++it;
    ```
    * 注意，又來了，必須用 `erase` 傳回傳的值來更新 iterator
* 反正不管怎樣有更動到 size 就要把 operation 回傳的 iterator 拿來更新啦

#### Removing Multiple Elements
```cpp
    elem1 = slist.erase(elem1, elem2);
```
* remove elements: \[elem1, elem2)

* remove all elements:
    ```cpp
    slist.clear(); // delete all the elements within the container
    slist.erase(slist.begin(), slist.end()); // equivalent
    ```
#### 9.3.4 Specialized forward_list Operations
![](https://i.imgur.com/tdz3cSG.png)
* When we add or remove an element, the **element before the one** we added or removed **has a different successor.**
* 可是單戀 linked-list 就沒有好簡單方法可以去改 predecessor
    * **所以 forward_list 的 add 跟 remove 都是*發生在指定的 iterator 的下一個位置***
    * 這樣當前指向的 element 就會是 add 或 remove 的 element 的 predecessor
* 因為上述行為跟普通加 element 或刪 element 的行為不一致，所以 forward_list 乾脆不定義相同名字的 API，直接定義別的
    * `insert_after`, `emplace_after`, and `erase_after`.

* 用上面的圖來說，如果要 remove elem3，你必須 call forward_lst.erase_after(elem2)
* 阿你會想如果我要 erase 第一個 element 怎麼辦? 又沒有 iterator 指向第一個 element 前面?
    * To support these operations, forward_list also defines `before_begin`, which returns an off-the-beginning iterator.
    * 所以標準定義了一個專屬 forward_list 的 iterator，指在第一個 element 前面(returns an off-the-beginning iterator.)
    * 當 forward_list 是 empty 時，你要插入 element 也是要用 before_begin()，這樣才會 element 變成第一個
* 專屬 API 列表
* ![](https://i.imgur.com/sGY4Zoz.png)
* When we add or remove elements in a forward_list, we haveto **keep track of *two* iterators**—**one to the element we’re checking and one to that element’s predecessor.**
    * 可以用 `before_begin` 跟 `begin` 達成
    ```cpp
    forward_list<int> flst = {0,1,2,3,4,5,6,7,8,9};
    auto prev = flst.before_begin(); // denotes element "off=the start" offlst
    auto curr = flst.begin(); // denotes the first element in flst
    while (curr != flst.end()) { // while there are still elements to process
        if (*curr % 2) // if the element is odd
            curr = flst.erase_after(prev); // erase it and move curr

        else {
            prev = curr; /*prev++? */  // move the iterators to denote the next
            ++curr;        // element and one before the next element

        }
    }
    ```

### 9.3.5 Resizing a Container
![](https://i.imgur.com/ZMMvvz8.png)
* 可變大變小，變小的話後面的 elements 丟掉，變大的話塞 value-initialized elements，或者用提供的 initializer 填 elements
    * 如果 element type 沒有 default ctor 你就一定要提供 initailizer
* 記得 iterator 那些的會失效!

### 9.3.6 Container Operations May Invalidate Iterators
* **重要**
* **Operations that add or remove elements from a container can invalidate pointers, references, or iterators to container elements.**
* 其實真的會發生 invalid 要看 container type，以及是否有重新配置空間
* 但是你的 code 就是要寫得夠 generic (?)

#### **After an operation that adds elements** to a container
1. **string 跟 vector**
    * 如果 container 重新配置，則全部 iterator(和其他) 都 invalid
    * 如果沒有重新配置，則你塞入新 element 的位置之前的 elements 對應的 iterator 都 remain valid，那個位置之後對應的 iterator 全部 invalid
2. **deque**
    * 如果在 `front` 或 `back` 之外的地方增加 element，iterator 那些的都 invalid
    * 這裡注意，如果是加在 front 或 back，**iterator invalid，但是 reference 跟 pointer remain valid**
        * 問號...
        * https://stackoverflow.com/questions/10373985/c-deque-when-iterators-are-invalidated 可以參考這篇
3. **list 跟 forward_list**
    *  Iterators, pointers, and references (including the off-the-end and the before-the-beginning iterators) to a list or forward_list remain valid

#### **After we remove an element**
* 首先指向被移除的 elements 的 iterator(和其他)會 invalid，這沒什麼好講的
1. **list 跟 forward_list**
    * All other iterators, references, or pointers (including the off-the-end and the before-the-beginning iterators) to a list or forward_list remain valid.
2. **deque**
    * 跟 add 時有點像，如果你移除的是 front 或 back 之外的 element，所有 iterator(和其他)都 invalid
    * 如果移除的是 back，指向 end() 的會 invalid，其他 iterator(和其他)不影響
    * 如果移除的是 front，其他 iterator(和其他)一樣不影響
3. **vector 跟 string**
    * 指向移除的位置之後的 elements 的 iterator(和其他) 都 invalid，指向之前的位置的 valid
        * 注意不管你移除哪一個，指向 end() 的都 invalid

:::info
* ADVICE:MANAGING ITERATORS
    * When you use an iterator (or a reference or pointer to a container element), **it is a good idea to minimize the part of the program during which an iterator must stay valid.**
    * Because code that adds or removes elements to a container can invalidate iterators, **you need to ensure that the iterator is repositioned, as appropriate, after each operation that changes the container.**
    * This advice is especially important for vector, string,and deque.
:::

#### Writing Loops That Change a Container
* Primer 教你怎麼寫一個基本正確的會更改 container 的 loop
* Loops that add or remove elements of a vector, string,or deque **must cater to the fact** that iterators, references, or pointers might be invalidated.
* they must be refreshed on each trip through the loop.
* Refreshing an iterator is easy if the loop calls insert or erase. **Those operations return iterators, which we can use to reset the iterator:**
    ```cpp
    // silly loop to remove even-valued elements and insert a duplicate of odd-valued elements
    vector<int> vi = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    auto iter = vi.begin(); // call begin, not cbegin because we’re changing vi
    while (iter != vi.end()) {
    if (*iter % 2) {
        iter = vi.insert(iter, *iter); // duplicate the current element
        iter += 2; // advance past this element and the one inserted before it
    } else
        iter = vi.erase(iter);// remove even elements
        //don't advance the iterator; iter denotes the element after the one we erase
    }
    ```
    * We refresh the iterator after both the `insert` and the `erase` because either operation can invalidate the iterator.
    * After the call to erase, there is no need to increment the iterator, because the **iterator returned from `erase` denotes the next element in the sequence.**
    * After the call to **`insert`, we increment the iterator twice.**
        * Remember, `insert` inserts ***before*** the position it is given and *returns an iterator to the inserted element.*
    * We add two to **skip over the element we added** and **the one we just processed.**

#### Avoid Storing the Iterator Returned from `end`
* 仔細再回顧上面說的，幾乎所有操作都會讓 container 的 `end()` invalid
* 這意味著如果將 `end()` 儲存起來的話就要要時常更新，否則會悲劇
    * 不要存一份 `end()` 的 copy！那沒意義！
    ```cpp
    auto end = c.end();
    for(auto it = c.begin(); it != end; it++)
        /*add or remove element*/
    ```
* 上面的 code 完全錯誤!
* 總之 `end()` 要用直接 call 的方式來取得，不要存一份 copy 來判斷
    * 也因為要成 call `end()`，通常 STL 實作會讓 call `end()` 超快
:::info
* Tips: Don’t cache the iterator returned from `end()` in loops that insert or delete elements in a `deque`, `string`,or `vector`.
:::


## 9.4 How a vector Grows
* library implementors use allocation strategies that reduce the number of times the container is reallocated
* When they have to get new memory, vector and string implementations typically **allocate capacity beyond what is immediately needed**

#### Members to Manage Capacity
* ![](https://i.imgur.com/z4FnbZM.png)
* The `vector` and `string` types provide members, described in Table 9.10, that let us **interact with the memory-allocation part of the *implementation.***
* `reserve(n)`:
    * 如果要求的 `n` 比 `capacity()` 多，就 *至少* allocate `n` 個 elements 那麼多的空間
    * 如果比 `capacity()` 小就不做事
* `resize(n)`
    * resize 會改變 element 個數
    * 一樣，如果 `n` 比 `capacity()` 小也不會導致 deallocate

* `shrink_to_fit`
    * C++11
    * `deque`, `vector`, `string` 可以用
    * 可以要求把不需要的 capacity 還回去
        * However, **the implementation is free to ignore this request.** There is no guarantee that a call to `shrink_to_fit` will return memory.

* `capacity()` and `size()`
    * The size of a container is the number of elements it already holds;
    * Its capacity is how many elements it can hold before more space must be allocated.
    ```cpp
    vector<int> ivec;
    // size should be zero;
    // capacity is implementation defined
    cout << "ivec: size: " << ivec.size() << " capacity: " << ivec.capacity() << endl;
    // give ivec 24 elements
    for (vector<int>::size_type ix = 0; ix != 24; ++ix)
        ivec.push_back(ix);
    // size should be 24;
    // capacity will be >= 24 and is implementation defined
    cout << "ivec: size: " << ivec.size() << " capacity: " << ivec.capacity() << endl;
    ivec.reserve(50); // sets capacity to at least 50; might be more
    // size should be 24; capacity will be >= 50 and is implementation defined
    cout << "ivec: size: " << ivec.size() << " capacity: " << ivec.capacity() << endl;
    ```
    * ![](https://i.imgur.com/dmlHH95.png)
* 你最後可以加這段 code 讓 size 跟 capacity 變成一樣
    ```cpp
    // add elements to use up the excess capacity
    while (ivec.size() != ivec.capacity())
        ivec.push_back(0);
    // capacity should be unchanged and size and capacity are now equal
    cout << "ivec: size: " << ivec.size()
    << " capacity: " << ivec.capacity() << endl;
    ```

* In fact, **as long as no operation exceeds the vector’s capacity, the vector must not reallocate its elements.**
* 接續上面的 code
    ```cpp
    ivec.push_back(42);
    // add one more element
    // size should be 51; capacity will be >= 51 and is implementation defined
    cout << "ivec: size: " << ivec.size()
    << " capacity: " << ivec.capacity() << endl;
    ```
    * 這時候 `vector` 就 reallocate 了

* 最後你可以用用看 `shrink_to_fit`，看看你用的實作會不會理你
    ```cpp
    ivec.shrink_to_fit(); // ask for the memory to be returned
    // size should be unchanged; capacity is implementation defined
    cout << "ivec: size: " << ivec.size() << " capacity: " << ivec.capacity() << endl;
    ```
    * 我用 gcc7.1 跟 clang6.0 都有理我耶，讚ㄛ

* Note: Each vector implementation can choose its own allocation strategy. **However, it must not allocate new memory until it is forced to do so.**


## 9.5 Additional string Operations
* 很多都標選讀，這裡主要講專屬 string 的 operation(其他 container 不能用)
* 有的是跟 C string 接的
* 總之快速讀過，要用再看

### 9.5.1 Other Ways to Construct strings
![](https://i.imgur.com/v0CJWeE.png)

#### The substr Operation
![](https://i.imgur.com/EHyvA9A.png)

### 9.5.2 Other Ways to Change a string

### 9.5.3 string Search Operations
* The string class provides six different search functions, each of which has four overloaded versions.
    * ㄎㄧㄤ
* return `size_type`, which is `unsigned`
    * **不要用 `signed` 去接**
![](https://i.imgur.com/fITQ1pf.png)
* 這邊要用到的時候再熟讀各種 overloading 的參數即可
* 但注意，**反過來搜尋(`rfind`, `last` 系列)** 的 functions 的 `pos` default 是 `npos`，並且是從 `pos` **開始往前搜**

### 9.5.4 The compare Functions
* mimic C `strcmp`
* https://stackoverflow.com/questions/9158894/differences-between-c-string-and-compare
* s.compare returns zero or a positive or negative value depending on whether s is equal to, greater than, or less than the string formedfrom the given arguments.

### 9.5.5 Numeric Conversions
* 把字串轉成 arithmetic type，或者倒過來
* ![](https://i.imgur.com/2fMTTpE.png)


## 9.6 Container Adaptors
* In addition to the sequential containers, the library *defines three sequential container adaptors: `stack`, `queue`,and `priority_queue`.*
* An **adaptor** is a general concept in the library.
* There are container, iterator, and function adaptors.
* Essentially, **an adaptor is a mechanism for making one thing act like another.**
* A container adaptor **takes an existing container type and makes it act like a different type.**
    * e.g. the `stack` adaptor **takes a sequential container** (other than `array` or `forward_list`) and makes it **operate as if it were a *stack(data structure)*.**

![](https://i.imgur.com/Sb05xYM.png)

#### Defining an Adaptor
* Each adaptor defines two constructors: the default constructor that creates an empty object, and **a constructor that takes a container** and initializes the adaptor by copying the given container.

* For example, assuming that `deq` is a `deque<int>`, we can use `deq` to initialize a new `stack` as follows:
    ```cpp
    stack<int> stk(deq); // copies elements from deq into stk
    ```

* By default both `stack` and `queue` are implemented in terms of `deque`, and a `priority_queue` is implemented on a `vector`.
    * 為何這些 adapter 底層預設是這樣選可以看這篇
    * https://stackoverflow.com/questions/102459/why-does-stdstack-use-stddeque-by-default
* We can override the default container type by naming a sequential container as a second (template) type argument when we create the adaptor:
    ```cpp
    // empty stack implemented on top of vector
    stack<string, vector<string>> str_stk;
    // str_stk2 is implemented on top of vector and initially holds a copy of svec
    stack<string, vector<string>> str_stk2(svec);
    ```
* 不是每種 container 都可以拿來給 adaptor 用，有限制
* 例如 adapter 都需要 add 跟 remove，所以 `array` 不行
* `forward_list` 也不行，要加到 container 尾巴會超慢(嚴格來說是沒有對 `back` 操作的 interface)
* `stack` 需要底層 container 有 `push_back`, `pop_back` 跟 `back` 這些 interface
    * 所以除了上面的跟不支援這些 operations 的 container 都可以當作 `stack` 底層的 container
* `queue` 還額外需要 `push_front`, `pop_front` 跟 `front`，所以只能用 `list` 跟 `deque`，不能用 `vector`
* `priority_queue` 則是需要 container 可以 random access(operator\[]), `front`, `push_back` 跟 `pop_back`，所以他可以用 `vector` 或 `deque`，但不能用 `list`


#### Stack Adaptor
* #include \<stack>
* ![](https://i.imgur.com/cGMub7E.png)
    ```cpp
    stack<int> intStack; // empty stack
    // fill up the stack
    for (size_t ix = 0; ix != 10; ++ix)
        intStack.push(ix); // intStackholds 0 . . . 9 inclusive
    while (!intStack.empty()) {
        // while there are still values in intStack
        int value = intStack.top(); // code that uses value
        intStack.pop(); // pop the top element, and repeat
    }
    ```


* 注意，Each container adaptor defines its own operations in terms of operations provided by the underlying container type.
    * 所以你才會看到為什麼 stack 不是用 push/pop_back，而是用 push 跟 pop
    * 這又是 API 設計哲學了，對於 stack 來說，叫 push 比 push_back 好，反正 stack 只能從其中一邊 塞/拿 資料，沒有 back 的問題
    * 而且你也不能呼叫 adapter 底層 container 的 `push_back`

#### The Queue Adaptors
![](https://i.imgur.com/Jo0BjbC.png)
* 解說跟 `stack` 87%像，自己看...
    * 痾，其實蠻重要的
    * 這裡可以重新回想起一個 heap 用 array/vector 就可以幹出來了
        * `std::push_heap`
* 注意 `queue` 跟 `priority_queue` 都是定義在 `<queue>` 裡面
* The library `queue` uses a first-in, first-out (FIFO) storage and retrieval policy.
* A `priority_queue` lets us **establish a priority among the elements** held in the queue.
    * **heap**
* Newly added elements are **placed ahead of all the elements with a lower priority.**
* By default, **the library uses the `operator<()` on the element type to determine** relative priorities.