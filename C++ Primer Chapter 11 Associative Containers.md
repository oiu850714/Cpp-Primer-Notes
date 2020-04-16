---
tags: C++
---

# C++ Primer Chapter 11 Associative Containers
* query/positioned by key
* 主要有兩種: `std::map` 跟 `std::set`
* 用 key 查 `std::map` 可以拿到 key 對應的 value
* 用 key 查 `std::set` 可以看 key 是否在 set
* 下表是 STL 提供的 associative containers
* ![](https://i.imgur.com/KXlykRp.png)
    * ```map``` 跟 ```multimap``` 在 \<map>
    * ```set``` 跟 ```multiset``` 在 \<set>
    * unordered 版本定義在 ```<unordered_map>``` 跟 ```<unordered_set>```
* 有三種特性
* 2^3 = 八種 containers
    * 是 set 或 map
    * 同個 key 是只有一份還是多份，前綴會有 `multi`
    * element 是否會按照 key 排列，前綴會有 `unordered`
## 11.1 Using an Associative Container

* `std::map`

* collection of key-value pairs
    * 例如 key 是 name(`std::string`)，value 是 phone number(`std::string`)
    * *mapping* names to phone numbers
* The `std::map` type is often referred to as an associative array.
    * like a "normal" array except that **its subscripts don’t have to be integers.**
    * Values in a `std::map` are found by a key rather than  by their position

* `std::set`
* set 就只是一堆 key，沒有對應的 value
* set 用在當我們想知道一個 value 是否存在於 container 內
    * 用 `std::vector` 也可，但那是 O(n)，`std::map` 是 O(lg n)

### Using a map
* 一個 map 最經典又簡單的例子: 數一段字串內每個單字出現幾次
    ```cpp
    // count the number of times each word occurs in the input
    map<string, size_t> word_count; // empty map from string to size_t
    string word;
    while (cin >> word)
        ++word_count[word]; // fetch and increment the counter for word
    for (const auto &w : word_count) // for each element in the map
    // print the results
        cout << w.first << " occurs " << w.second
             << ((w.second > 1) ? " times" : " time") << endl;
    ```

* 其實蠻多東西要講的，例如那個 `++word_count[word]` 是三小，做了什麼事情，每一步的細節都要知道
    * 一樣，associative container 是 template
    * `std::map` 角括號內要給兩個參數，一個是 key 的 type 一個是 value type
        * 例如 `std::map<string, size_t>`
    * 對這個 map 用 `operator[]`，`[]` 內要提供 `std::string`，而 result type 就是 `size_t`
    * loop 用 `std::cin` 讀到的 `std::string` 來 index map
    * 如果 container 內有對應的 key，就取 key 對應的 element，沒有則將 key 存至 map，並 create element(所謂的 **firstOrCreate**)
    * 最後再對 value 做 `operator++`
    * 最後一個 range for 一樣是輪循 container 內的 element，這個 element 的 type 是 `std::pair<string, size_t>`，一樣是 template
        * 當你宣告 `std::map` 時，這個 pair 的兩個參數型態就會是 map 內角括號給的兩個型態，在這個例子就是 `std::pair<string, size_t>`
    * 你要拿 key 就要對 pair 用 `first` member，要拿 value 就要用 `second` member

* 總結
    * map 裡面的 element 是 `std::pair`，`std::pair` 裡面有 `first` 跟 `second`，對應 key 跟 value
    * `first` `second` 的 type 在宣告 `std::map` 時給定
    * 對 `std::map` 使用 `operator[]` 時要用 `first` 的 type

#### Using a `std::set`
* 一樣先舉例使用情境，從上面的例子推來；不過會忽略計算 `a` `the` `an` `or` `and` 這種詞彙
```cpp
// count the number of times each word occurs in the input
map<string, size_t> word_count; // empty map from string to size_t
set<string> exclude
    = {"The", "But", "And", "Or", "An", "A",
       "the", "but", "and", "or", "an", "a"};
string word;
while (cin >> word) // count only words that are not in exclude
if (exclude.find(word) == exclude.end())
    ++word_count[word]; // fetch and increment the counter for word
```
* `std::set` 也是 template
* 宣告時給定 element type
* **可以被 list initialize**(`std::map` 也可，不過寫法比較便秘)

* 上面的重點是這一行
    ```cpp
    if (exclude.find(word) == exclude.end())
    ```
    * `set<T>.find(obj)` 會在 container 內找 element，會**回傳指向該 element 的 iterator**
        * 如果有找到，iterator 就會指向 element，如果沒有，就回傳 `container.end()`
    * 這行 code 是在確定輸入的字串是否是我們無視掉的字，如果不是才存到 `std::map` 內

## 11.2 Overview of the Associative Containers
* Table 9.2 的 operation 都支援
* 再貼一次
* ![](https://i.imgur.com/oBGNoS3.png)
* associative container 不支援那些 sequential container 獨有的對**特定位置**的操作
    * 例如 `push_front/back` 系列，還有 `front/back` 等等
    * 這些操作不 make sense，因為 associative container 沒有位置的概念
* 也不支援吃 `(n, val)` 的這種 ctor，塞 n 個 value
* 也不能一次 `insert` n 個 element(`insert(n, val)`)
* associative container 當然也有獨有的 operations(table 11.7)，甚至 unordered 版本的還有提供一種改變 hash 方式的操作(hash 詳情演算法)
* 最後，**associative container 對應的 iterator 是 bidirectional 的**

### 11.2.1 Defining an Associative Container

* default ctor
* list initailize
* copy from same type of container
* copy from a range of elements that can be converted to container's element type
    ```cpp
    map<string, size_t> word_count; // empty
    // list initialization
    set<string> exclude = {"the", "but", "and", "or", "an", "a",
                           "The", "But", "And", "Or", "An", "A"};
    // three elements; authors maps last name to first
    map<string, string> authors = {
            {"Joyce", "James"},
            {"Austen", "Jane"},
            {"Dickens", "Charles"} };
    ```
    * 注意 `std::map` 型態用 list initialize 時，key value 都要寫，用 `{key, value}` 包住

#### Initializing a `std::multimap` or `std::multiset`
* several elements with the same key
    * 例如字典可用 multimap，key 是單字，value 是單字的其中一種定義(definition)

* The following example **illustrates the differences between the containers with unique keys and those that have multiple keys**.
    ```cpp
    // define a vector with 20 elements, holding two copies of each number from 0 to 9
    vector<int> ivec;
    for (vector<int>::size_type i = 0; i != 10; ++i) {
        ivec.push_back(i);
        ivec.push_back(i); // duplicate copies ofeach number
    }
    // iset holds unique elements from ivec; miset holds all 20 elements
    set<int> iset(ivec.cbegin(), ivec.cend());
    multiset<int> miset(ivec.cbegin(),
    ivec.cend()); cout << ivec.size() << endl; // prints 20
    cout << iset.size() << endl; // prints 10
    cout << miset.size() << endl; // prints 20
    ```

### 11.2.2 Requirements on Key Type
* 這裡先講 order 版本的 container 對 key type 的要求；11.4 講 unordered 版本的 container 對 key 的要求
* `std::map`, `std::multimap`, `std::set`, `std::multiset`
    * 這些 container 的 key type 必須定義比較 key 順序的方式
    * 預設是用 `operator<()`
    * 其實這個要被定義的操作就跟 `std::sort` 需要的 operation 一樣
    * 當然，想要主動提供代替 `operator<()` 的 predicate 也可以
        * 但因為 template 的關係，寫法也是比較便秘，之後會看到

#### Key Types for Ordered Containers
* 對 key type 要使用 `operator<()`，的或使用自定義的比較 function，都要滿足 **strict weak ordering**
    1. 給一對 key A, B，A op B 為 true，則 B op A 一定為 false
    2. 給三個 key A, B, C，如果 A op B 且 B op C，則 A op C
    3. 給一對 key A, B，如果 !(A op B) && !(B op A)，則我們說 A 跟 B 等價(equivalent)；而且如果 A 等價 B 且 B 等價 C，則 A 等價 C
* 舉例: `int` 的 `operator<()` 就滿足
    * **但是 `operator<=()` 就不滿足!**，如果你把 <= 傳給 `std::sort` 之類的很可能會發生災難(core dump)
* 結論，如果你的 class 定義的 `operator<()` ，或者你提供的作用在兩個這個 class 上的 object 的 callable object，符合上述特性，那這個 class 就可以當 key


* 而 C++ 在"看待"你定義的 class object 的順序時的確就是用 `operator<()` 經過上面的變化兜出其他 operator 的，**如果你的 `operator<()` 的定義有問題，亦即不滿足 strict ordering** 那就會出事
    * 例如現在是對 `std::string` 做運算，但你用 `operator<=()` 取代原本預設的 `operator<()`，那當兩個 string 的內容真的相等時，`operator==` 反而會是 `false`!
        * 因為 `operator==` 其實是 `!(a <= b) && !(b <= a)`，會是 `false`!


#### Using a Comparison Function for the Key Type
* 對於關聯容器來說，如何排列 elements 這件事情也是他們本身的一部份，換句話說就是關聯容器型態的一部份
    * 換句話說可以指定到 template 那個神奇的角括號內
* 所以你在定義關連容器時，其實是要連如何排序 elements 都指定的
    * 只是也可以不給，因為如果你不給的話，C++ 預設用 element 的 `operator<()` 當作排序方法

* 如果想要把那種沒有定義 `operator<()` 的 class 當成 key type，就不能不在宣告時給定排序的方法，不給的話就會噴 error
    * 例如 `Sales_data`
* 你可以對這種 class 額外給關連容器一個 function pointer type
    ```cpp
    bool compareIsbn(const Sales_data &lhs, const Sales_data &rhs) {
        return lhs.isbn() < rhs.isbn();
    }
    // bookstore can have several transactions with the same ISBN
    // elements in bookstore will be in ISBN order
    multiset<Sales_data, decltype(compareIsbn)*>
        bookstore(compareIsbn);
    ```
    * 角括號(`<>`)內除了給 element type(如果是 map 就給 key 跟 value 的 type，如果是 set 就給 key type 就好)，**還要給一個 function pointer type**
    * 而宣告時就要把特定的 function 當作參數傳進去，這裡就是把 `compareIsbn` 當作 bookstore 的參數
    * 用 `decltype` 免去寫複雜 type 的麻煩，不過記得 `decltype` 不會 decay function type to pointer，所以要加 `*`
    * 你參數要寫 compareIsbn 或者 &compareIsbn 都可以，因為前者會自動 decay 成後者
* 結論: 角括號跟括號都要額外傳參數

### 11.2.3 The pair Type
* 所以 `std::pair` 是啥?
* 定義在 `<utility>`
* 也是 template, 給兩個參數
* public data member, `first`, `second`, 對應到 template 的第一跟第二個參數

    ```cpp
    pair<string, string> anon; // holds two strings
    pair<string, size_t> word_count; // holds a stringand an size_t
    pair<string, vector<int>> line; // holds stringand vector<int>
    ```
* `std::pair` 的 default ctor 會 **value initialize** `first` 跟 `second`
    * 這也是為什麼我們的 `word_count` 的 `size_t` 會初始化成 0 的原因(詳情前面 `std::map` 的筆記)，因為 `std::map` 的 element 就是 `std::pair`

* 當然可用 list initializer
    ```cpp
    pair<string, string> author{"James", "Joyce"};
    ```
    * 但看了一下 cppreference，感覺是 list initialze(`{}`) fallback 回呼叫兩個參數的 ctor(`pair<T1, T2> p(v1, v2)`)
* 可用操作
    * ![](https://i.imgur.com/pPevyk4.png)

#### A Function to Create `std::pair` Objects
* 這裡複習，在 C++11 我們一樣可以對 return value 用 list initialize
    ```cpp
    pair<string, int> process(vector<string> &v) {
        // process v
        if (!v.empty())
            return {v.back(), v.back().size()}; // list initialize
        else
            return pair<string, int>(); // explicitly constructed return value
    }
    ```
* 舊版 C++ 不能用上面 list initialize 的寫法，要馬就用 else 呼叫 default ctor，要馬用 `std::make_pair`
    ```cpp
    if (!v.empty())
        return pair<string, int>(v.back(), v.back().size());
    if (!v.empty())
        return make_pair(v.back(), v.back().size());
    ```

## 11.3 Operations on Associative Containers
* 關連容器額外定義的 type
* ![](https://i.imgur.com/BNGiEyj.png)

* 對 `std::set` 來說
    * `key_type` 跟 `value_type` 一樣，跟你宣告 `std::set` 時給的 type 一樣
* 對 `std::map` 來說
    * `key_type` 是宣告 `std::map` 時給的第一個參數
    * `mapped_type` 是宣告 `std::map` 時給的第二個參數
        * **`set` 系列沒有 mapped_type**
    * `value_type` 則是 `pair<const key_type, mapped_type>`
* 看範例猜每個變數的型別
```cpp
set<string>::value_type v1; // v1  is a string
set<string>::key_type v2; // v2 is a string
map<string, int>::value_type v3; // v3 is a pair<const string, int>
map<string, int>::key_type v4; // v4 is a string
map<string, int>::mapped_type v5; // v5 is an int
```

### 11.3.1 Associative Container Iterators
* 複習，**對容器的 iterator 做 dereference 得到的是 `container::value_type`**
* 而 associative container 的 `value_type` 是 pair
    * 所以 `for(auto &e : map_obj)` 才會拿到 pair
    * **而且不能更改 `e.first`**，因為是 `const`

    ```cpp
    // get an iterator to an element in word_count
    auto map_it = word_count.begin();
    // *map_it is a reference to a pair<const string, size_t>object
    cout << map_it->first; // prints the key for this element
    cout << " " << map_it->second; // prints the value ofthe element
    map_it->first = "new key"; // error: key is const
    ++map_it->second; // ok: we can change the value through an iterator
    ```
#### Iterators for sets Are const
* 然後雖然 `set` 的 `value_type` 跟 `key_type` 一樣，而且也沒有 const，**可是 set 的 iterator 跟 const_iterator 一樣，沒有辦法拿來更改值**

* 其實為什麼 `set` iterator 跟 `map` 的 pair 會有 `const` 行為要從資料結構去想，因為如果可以直接藉由 `set` 的 iterator 或者 `map` 的 value_type 的 key 改值，這樣關連容器內的 element 順序就會亂掉
    ```cpp
    set<int> iset = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    set<int>::iterator set_it = iset.begin();
    if (set_it != iset.end()) {
        *set_it = 42; // error: keys in a set are read-only
    }
    cout << *set_it << endl; // ok: can read the key
    ```

#### Iterating across an Associative Container
* 一樣可用 `begin` `end` 把關連容器的 elements 掃一遍
    ```cpp
    // get an iterator positioned on the first element
    auto map_it = word_count.cbegin();
    // compare the current iterator to the off-the-end iterator
    while (map_it != word_count.cend()) {
    // dereference the iterator to print the element key--value pairs
        cout << map_it->first << " occurs "
             << map_it->second << " times" << endl;
        ++map_it; // increment the iterator to denote the next element
    }
    ```
    * 跟掃 `std::string` 或者 `std::vector` 這種 sequential container 有 87% 像

* 還有一點很重要，ordered 版本的 `set` `map`，**從 begin 掃到 end，掃出來的 element 是(按照 key)排序過的!**

#### Associative Containers and Algorithms
* std::algo 很多都廢掉
* 要馬是因為
    * map 掃出來的 key 是 const，write 類型的 algo 不能用
* 要馬是因為
    * 很多 algo 會用到 `std::find`，`std::find` 是 linear，對關連容器來說很慢

* 所以關連容器有定義一些自己專屬的 member algo
    * 例如 `find`，是 O(log n) 或更快(unordered 是 O(1))

* 如果真的要用 std::algo，多半是傳入關連容器的 iter 進去**當作 input range**，或者**用 inserter 當作 destination**
* 題外話，這樣感覺要寫出一個可以同時接受 sequential 跟 associative container 的 template 有困難


### 11.3.2 Adding Elements
* 用 insert member
    ```cpp
    vector<int> ivec = {2,4,6,8,2,4,6,8}; // ivec has eight elements
    set<int> set2; // empty set
    set2.insert(ivec.cbegin(), ivec.cend()); // set2 has four elements
    set2.insert({1,3,5,7,1,3,5,7});
    // set2
    now has eight elements
    ```
    * `map` `set` 這種 unique 系列的會自動無視重複的 key
    * 如果給定的 range 有重複 key，只有第一個 element 會被插進來

#### Adding Elements to a map
![](https://i.imgur.com/BVfiFGd.png)

* 記得 `map` 的 element type 是 `std::pair`，所以 `insert` 給的 argument type 也要是 `std::pair`
* 但是常常我們不會生一個 `std::pair` 再插進 map，通常會像下面這樣寫
    ```cpp
    // four ways to add word to word_count
    word_count.insert({word, 1});
    word_count.insert(make_pair(word, 1));
    word_count.insert(pair<string, size_t>(word, 1));
    word_count.insert(map<string, size_t>::value_type(word, 1));
    ```
    * 這時候 C++11 提供的第一種 brace initialize 方法就顯得好用很多

#### Testing the Return from insert
* unique 系列的關連容器，**`insert` 也是 return 一個 `pair<iter, bool>`**，`iter` 指向給定 key 的 element，`bool` 提示是否有插入成功
* word counting 可以重寫成這樣
    ```cpp
    // more verbose way to count number of times each word occurs in the input
    map<string, size_t> word_count; // empty map from string to size_t
    string word;
    while (cin >> word) {
    // inserts an element with key equal to word and value 1;
    // if wordis already in word_count, insert does nothing
    auto ret = word_count.insert({word, 1});
    if (!ret.second) // word was already in word_count
        ++ret.first->second; // increment the counter
    }
    ```
#### Unwinding the Syntax
* 上面的 ```++ret.first->second;``` 可以看成```++((ret.first)->second);``` 就比較好懂
* 還有，上面的 `ret` 是用 `auto` 宣告的，如果你要看 Pre C++ code，會變成這樣
    ```cpp
    pair<map<string, size_t>::iterator, bool> ret =
        word_count.insert(make_pair(word, 1));
    ```
    * 總是會有機會看 Legacy code ㄉ(抖

#### Adding Elements to `multiset` or `multimap`
* 再舉個用 multi 系列的小例子: 一本書，多個共同作者
* `multiset/multimap` 的 *`insert` 一定會插入 element*
    ```cpp
    multimap<string, string> authors;
    // adds the first element with the key Barth, John
    authors.insert({"Barth, John", "Sot-Weed Factor"});
    // ok: adds the second element with the key Barth, John
    authors.insert({"Barth, John", "Lost in the Funhouse"});
    ```
* multi 版本的 `insert` 就只回傳 `iter` 而已，指向新插入的 element

### 11.3.3 Erasing Elements
![](https://i.imgur.com/ryr298U.png)
* 後兩個跟 sequential 的一樣
* 第一個是把有這個 key 的 element 刪除，return 會說有幾個 element 被刪除
    * unique 版本的關連容器 always return 0 或 1
    ```cpp=
    // erase on a key returns the number of elements removed
    if (word_count.erase(removal_word))
        cout << "ok: " << removal_word << " removed\n";
    else
        cout << "oops: " << removal_word << " not found!\n";
    ```

### 11.3.4 Subscripting a map
![](https://i.imgur.com/UPOgWKW.png)

* `std::map` 跟 `std::unordered_map` 可以用 `operator[]`，還有 member function `at`
    * *但是剩下六種 associative container 都不能用，如果要寫 template 的話要注意*
* 跟 sequential 容器一樣，可以放 key 到 `[]` 內(順序容器是放 position)獲得對應 value
* However, unlike other subscript operators, **if the key is not already present, *a new element is created and inserted*** into the map for that key.
    * 對應這個 key 的 value 會被 value initialized(因為 pair 預設是 value initialize)
    ```cpp
    map <string, size_t> word_count; // empty map
    // insert a value-initialized element with key Anna; then assign 1 to its value
    word_count["Anna"] = 1;
    ```
    * 看清楚，因為 `word_count` 原本沒有 key 為 `"Anna"` 的 element，所以 `word_count` 會先創造一個 element，key 為`"Anna"`，value 用值初始化(`size_t` 為 `0`)
    * *value initialize 之後*，才會把 `1` assign 給 element
* **因為 `operator[]` 可能會插入 element，所以只有 nonconst map 可以用**

#### Using the Value Returned from a Subscript Operation
* map 的 `operator[]` 行為其實跟 sequential container 差很多
    * 對 sequential container 用 `operator[]`，跟對他們的 iter 做 dereference，得到的 type 都是 element type
    * 可是**對 `std::map` 做 `operator[]` 得到的是 `mapped_type`，而對 `std::map` 的 iter dereference 得到的是 `value_type`(element type)**

* 不過 `map` 的 `operator[]` 一樣還是 return lvalue，所以可以拿來讀寫:
    ```cpp
    cout << word_count["Anna"];
    // fetch the element indexed by Anna; prints 1
    ++word_count["Anna"];
    // fetch the element and add 1to it
    cout << word_count["Anna"];
    // fetch the element and print it; prints 2
    ```

* 因為 map 的 `operator[]` 操作會暗中插入 element，很多時候你可以用這點直接封裝那種 "firstOrCreate" 的邏輯
    * 但如果你不是要 firstOrCreate，只是單純想查某個 key 是否在 map 內的話則絕不能用 `operator[]`

### 11.3.5 Accessing Elements
![](https://i.imgur.com/ktsk4qB.png)
* 要用上表的哪個 function 端看你想要找什麼東西
* `operator[]`, `at` 只有 `map` 跟 `unordered_map` 可用
    * multi 版本的不 make sense
    * set 也沒有 element 也不 make sense
* 關於 `lower_bound`, `upper_bound`, `equal_range` 回傳的 iterator，可以看這個
    * https://youtu.be/2olsGf6JIkU?t=1852
    * 但是注意! 影片是用 generic algo 版本的做說明
    * 這三個也只有 order 版本的 associative container 可用
* 對 unique container 系列來說，`count` 永遠回傳 1 或 0
    ```cpp
    set<int> iset = {0,1,2,3,4,5,6,7,8,9};
    iset.find(1); // returns an iterator that refers to the element with key == 1
    iset.find(11); // returns the iterator ==iset.end()
    iset.count(1); // returns 1
    iset.count(11); // returns 0
    ```
#### Using `find` Instead of Subscript for maps
* 想找某個 key 卻又不想插入 element 時用 find
    ```cpp
    if (word_count.find("foobar") == word_count.end())
        cout << "foobar is not in the map" << endl;
    ```

#### Finding Elements in a `multimap` or `multiset`
* 可能會存有很多同樣的 keys
* (multi) associative container 會把 key 同樣的 elements 都擺在一起

* 如果要印 order-multi 系列的 container 內同樣 key 的 elements，可以有三種寫法
##### 第一種，用 `find` + `count`
```cpp
string search_item("Alain de Botton"); // author we’ll look for
auto entries = authors.count(search_item); // number of elements
auto iter = authors.find(search_item); // first entry for this author
// loop through the number of entries there are for this author
while(entries) {
    cout << iter->second << endl; // print each title
    ++iter; // advance to the next title
    --entries; // keep track ofhow many we’ve printed
}
```
* 先用 `count` 找到擁有同樣 key 的 elements 有多少個
* 再用 `find` 找到擁有同樣 key 的 elements 內*指向第一個的 iter*
* 不斷的對 iter 做 `operator++`，做 `count` 次，就可以把這些 elements 掃過一次

##### 第二個方法，用 iterator 找出 range，不用 `count`
```cpp
// definitions of authors and search_item as above
// beg and end denote the range of elements for this author
for (auto beg = authors.lower_bound(search_item),
            end = authors.upper_bound(search_item);
            beg != end; ++beg)
    cout << beg->second << endl; // print each title
```
* 需要用 `lower_bound` 跟 `upper_bound`
* 如果 container 裡有這個 key 的 elements，`lower_bound` 會指向第一個，`upper_bound` 會指向最後一個的後面一個
    * 精簡的說法就是 `[lower_bound, upper_bound)`
* 如果沒有這個 key 的 element，那 `lower_bound` 跟 `upper_bound` 會指向同一個地方，**這個地方的意義是，在這個位置插入這個 key 的 element，container 的 order 不會亂掉**
    * 可以推論出，如果你找的 key 比 container 內的所有 key 都大，那
* 總之，**如果 lower_bound() == upper_bound()，那 key 就不在 container 裡面**

##### 第三種，用 `equal_range` Function
* `return pair<iter1, iter2>`，`iter1` `iter2` 形成這個 key 的 range
```cpp
// definitions of authors and search_itemas above
// pos holds iterators that denote the range of elements for this key
for (auto pos = authors.equal_range(search_item);
        pos.first != pos.second; ++pos.first)
    cout << pos.first->second << endl; // print each title
```
* 實際上就是回傳 `{lower_bound(), upper_bound()}`
### 11.3.6 A Word Transformation Map
* 做一個程式，處理一段文字，可以把文字內指定的字串變換成另一個字串
* input 是兩個檔案，第一個存指定換字串變換的規則，另一個則是要變換的文字檔
* 規則的範例:
```
brb be right back
k okay?
y why
r are
u you
pic picture
thk thanks!
l8r later
```
* 變換的文字檔範例:
```
where r u
y dont u send me a pic
k thk l8r
```
* 輸出結果:
```
where are you
why dont you send me a picture
okay? thanks! later
```

* 三個 function:
    * `word_transform`: overall processing, 收兩個檔案的 `ifstream&`
    * `buildMap`: 吃第一個 `word_transform` 的第一個 `ifstream&`，建立出一個 `map` that maps string to phrase
    * `transform`: 吃一個字串，如果存在規則把字串做轉換，則回傳轉換後的字串，否則回傳原字串
```cpp
void word_transform(ifstream &map_file, ifstream &input) {
    auto trans_map = buildMap(map_file); // store the transformations
    string text;
    // hold each line from the input
    while (getline(input, text)) { // read a line of input
        istringstream stream(text); // read each word
        string word;
        bool firstword = true; // controls whether a space is printed
        while (stream >> word) {
            if (firstword)
                firstword = false;
            else
                cout << " "; // print a space between words
            // transform returns its first argument or its transformation
            cout << transform(word, trans_map); // print the output
        }
        cout << endl; // done with this line of input
    }
}
```
* 重點就是 call `buildMap` 跟 `transform`
* 還有文字檔一次 `getline` 一行，這樣處理完這行後直接印 endl，輸出的換行就會原文一模一樣
* 吃一行進來之後就用第八章學到的，用 istringstream 來處理這一行

#### Building the Transformation Map
```cpp
map<string, string> buildMap(ifstream &map_file) {
    map<string, string> trans_map; // holds the  transformations
    string key; // a word to transform
    string value; // phrase to use instead
    // read the first word into keyand the rest of the line into value
    while (map_file >> key && getline(map_file, value))
        if (value.size() > 1) // check that there is a transformation
            trans_map[key] = value.substr(1); // skip leading space
        else
            throw runtime_error("no rule for " + key);
    return trans_map;
}
```
* `while` condition 其實很有戲
    * 先吃要被處理的文字
    * 剩下的直接 `getline` 當成要轉換的規則
* 那個 substr(1) 是為了把 string 跟 phrase 中間的空格刪掉
    * 如果 key 跟 phrase 之間有多個空格，則 phrase 開頭還是會有空格
* 另外，這個轉換如果遇到一個 `key` 出現多個對應 `phrase` 的情形，則會使用最後一個 phrase，原因是我們使用了 `operator[]`

#### Generating a Transformation
```cpp
const string & transform(const string &s, const map<string, string> &m) {
    // the actual map work; this part is the heart of the program
    auto map_it = m.find(s);
    // if this word is in the transformation map
    if (map_it != m.cend())
        return map_it->second; // use the replacement word
    else return s; // otherwise return the original unchanged
}
```
* 應該簡單易懂不用解釋...

## 11.4 The Unordered Containers
* C++11 才有
* 用 hash 的方式存 element
    * 不懂在工三小的詳情演算法大師

* 換句話說 key 要支援 hash；
* 也要支援 `operator==`
* 如何讓自定義的 type 能被 hash，這個之後會講
* built-in 跟 string 都已經有內建 hash 惹
    * 還有 12 章要講的 smart pointer(抖)

* **An unordered container is most useful whenwe have a key type for which *there is no obvious ordering relationship among the elements***.
* These containers are also useful for applications in which the cost of maintaining the elements in order is prohibitive.

* 無序容器的效能取決於 hash(key_type) function 的品質，可能需要很多測試

* Tip: Use an unordered container if the key type is inherently unordered or if performance testing reveals problems that hashing might solve.

* 基本上 order 版本的操作 unordered 版本都支援(但效能不一樣)
* 所以理論上你把之前寫的 code 用到的有序版本掉換成對應的無序版本容器應該都不會錯，可以編
* **但是輸出的東西就不會按照 key order 排列惹!**
* 例如重寫之前的計數問題
    ```cpp
    // count occurrences, but the words won’t be in alphabetical order
    unordered_map<string, size_t> word_count;
    string word;
    while (cin >> word)
        ++word_count[word]; // fetch and increment the counter for word
    for (const auto &w : word_count) // for each element in the map
    // print the results
        cout << w.first << " occurs " << w.second
             << ((w.second > 1) ? " times" : " time") << endl;
    ```

#### Managing the Buckets
* The unordered containers are organized as a collection of buckets, each of which holds zero or more elements.
    * 大概要知道 hash 的概念才能知道這段在工三小
* **These containers use a hash function to map elements to buckets.**
* 要存取一個 element，首先要用 hash 算出他會在哪個 bucket，然後去那個 bucket 找 element
* 有同樣 hash value 的 element 都會在同一個 bucket
* 想當然耳，multi 版本的容器，同樣的 key 全部都會在同一個 bucket
* 總之，無序容器的效能 depends on hash 品質，element 數量，以及 bucket 的數量還有大小
* When a bucket holds several elements, **those elements are searched *sequentially* to find the one we want.**

* 下表示無序容器額外提供的操作，有些還可以拿來更改 bucket size
* ![](https://i.imgur.com/DWqQ7Td.png)

#### Requirements on Key Type for Unordered Containers
* `operator==()`
* `hash<key_type>`
    * 是個 template
    * **built-in 跟 pointer 還有 string，smart pointer，標準都已經定義**
* 其他 type 或者自定義的 class 都不能用這個 hash template，他不像 container 一樣泛用；相反的，我們要自己定義我們的 hash template，16 章會講

* 不過那些不能 hash 的 type 其實也可以在宣告無序容器的時候給定 hash function；就跟宣告 map 時額外給定 sort function 一樣的道理
* 用 `Sales_data` 舉例，這個 class 不能 hash
    * 也沒有 `operator==`
* 我們就額外定義 hash function 跟 `operator==`
* 注意，以下宣告的無序容器**第一個參數都是 bucket size**
    ```cpp
    size_t hasher(const Sales_data &sd) {
        return hash<string>()(sd.isbn());
    }
    bool eqOp(const Sales_data &lhs, const Sales_data &rhs) {
        return lhs.isbn() == rhs.isbn();
    }
    ```
* 然後這樣宣告無序容器:
    ```cpp
    using SD_multiset = unordered_multiset<Sales_data, decltype(hasher)*, decltype(eqOp)*>;
    // arguments are the bucket size and pointers to the hash function and equality operator
    SD_multiset bookstore(42, hasher, eqOp);
    ```
* 如果自定義的 class 有 `operator==`，可以在宣告無序容器時只給 hash function:
    ```cpp
    // use FooHash to generate the hash code; Foo must have an == operator
    unordered_set<Foo, decltype(FooHash)*> fooSet(10, FooHash);
    ```

## 延伸閱讀
* `ste::tie`
    * https://www.google.com/search?q=std%3A%3Aignore&oq=std%3A%3Aignore&aqs=chrome..69i57j69i58.1968j0j7&sourceid=chrome&ie=UTF-8