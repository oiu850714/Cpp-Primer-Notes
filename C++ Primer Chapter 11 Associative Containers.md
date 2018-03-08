# C++ Primer Chapter 11 Associative Containers
* query by key
* 主要有兩種: map 跟 set
* 都是用 key 來 "query" container
* 用 key 查 map 可以拿到 key 對應的 value
* 用 key 查 set 可以看 key 是否在 set
* 下表是 STL 提供的 associative containers
* ![](https://i.imgur.com/KXlykRp.png)
    * ```map``` 跟 ```multimap``` 在 \<map>
    * ```set``` 跟 ```multiset``` 在 \<set>
    * unordered 版本定義在 ```<unordered_map>``` 跟 ```<unordered_set>```
* 有三種特性
    * 是 set 或 map
    * 同個 key 是只有一份還是多份，前綴會有 ```multi```
    * element 是否會按照 key 排列，前綴會有 ```unordered```
## 11.1 Using an Associative Container
* 哎呀是擅長演算法的大師呢!
    * 嗚嗚嗚嗚
* 先介紹我們想要處理的問題，再介紹 map 跟 set 到底是什麼東西以及怎麼解決這些問題

#### map
* collection of key-value pairs
    * 例如 key 是 name，value 是 phone number
    * **mapping** names to phone numbers
* The map type is often referred to as an associative array.
    * like a "normal" array except that **its subscripts don’t have to be integers.**
    * Values in a map are found by a key rather than  by their position
* 如果你真的讓 map 存 name 跟 phone number，你就可以餵他 name 然後得到對應的 phone number

#### set
* set 就只是一堆 key，沒有對應的 value
* set 用在當我們想知道一個 value 是否存在於 container 內
    * 用 vector 不就可以ㄌ，掃一遍阿
    * 因為這樣很慢阿 zzz，演算法大師

#### Using a map
* 一個 map 最經典又簡單的例子就是數單字有多少個!
    ```C++
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

* 其實蠻多東西要講的，例如那個 ++word_count\[word] 是三小，做了什麼事情，每一步的細節都要知道
    * 一樣關聯容器是 template
    * map 角括號內要給兩個參數，一個是 key 的 type 一個是 value type
    * 對這個 map 用 \[]，\[] 內要提供 string，而 result type 就是 size_t
    * loop 用讀到的 string 來 index map
    * 如果 container 內有對應的 element，就取那個 element
    * 如果沒有，subscript operator(\[]) 就會創造一個新的 element，他的 key 就是這個 string，value 是 0(value initialize?)
    * 最後再對 value 做 ++
    * 最後一個 range for 一樣是輪循 container 內的 element，這個 element 的 type 是 ```pair```，一樣是 template，也要吃兩個參數
        * 當你宣告 map 時，給定的兩個參數就會是 map 內 pair 的兩個參數，在這個例子就是 pair\<string, size_t>
    * 你要拿 key 就要對 pair 用 first member，要拿 value 就要用 second member

* 總結
    * map 裡面的 element 是 pair，pair 裡面有 first 跟 second，對應 key 跟 value
    * first second 的 type 在宣告 map 時給定
    * 用 \[] 時用 first 的 type

#### Using a set
* 一個小例子，從上面的例子推來，不過會無視 a the an or and 這種詞彙
* 可以用 set 達成
    ```C++
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
    * set 也是 template
    * 宣告時給定 element type
    * 可以被 list initialize

* 上面的重點是這一行
    ```if (exclude.find(word) == exclude.end())```
    * set\<T>.find(obj) 會在 container 內找 obj，會回傳 iterator
        * 如果有找到，iterator 就會指向 obj，如果沒有，就回傳 container.end()
    * 這行 code 是在確定輸入字串是否是我們無視掉的字，如果不是才存到 map 內

## 11.2 Overview of the Associative Containers
* Table 9.2 的 operation 都支援，這些只要是 container 都支援
* 再貼一次
* ![](https://i.imgur.com/oBGNoS3.png)
* associative container 不支援那些 sequential container 獨有的對**位置**的操作
    * 例如 push_front/back 系列，還有 front/back 等等
    * 這些操作不 make sense，因為 associative container 沒有位置的概念
* 也不支援 constructor(n, val) 這種，塞 n 個 value，也不能一次塞 n 個 element(insert(n, val))
* associative container 當然也有獨有的 operations(table 11.7)，甚至 unordered 版本的還有提供一種改變 hash 方式的操作(詳情演算法大師)
* 最後一個，associative container 產生的 iterator 是 bidirectional 的

### 11.2.1 Defining an Associative Container

* default cstr
* list initailize
* copy from same type of container
* copy from a range of elements that can be converted to container's element type
    ```C++
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
    * 注意 map 型態用 list initialize，key value 都要寫，用 {key, value} 包住

#### Initializing a multimap or multiset
* several elements with the same key
    * 例如字典可用 multimap，key 是單字，value 是對應的其中一種意思
    
* The following example illustrates the differences between the containers with unique keys and those that have multiple keys.
    ```C++
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
* map, multimap, set, multiset
    * 這些 container 使用時必須定義比較 element 順序的方式
    * 預設是用 element 的 operator<
    * 其實這個要被定義的操作就跟 std::sort 需要的 operation 一樣

#### Key Types for Ordered Containers
* 這個 operation *op*，不管是從 element 的 operator< 來的或者你自己定義的，都要滿足 **strict weak ordering**
    1. 給一對 key A, B，不可能 A op B && B op A
    2. 給三個 key A, B, C，如果 A op B 且 B op C，則 A op C
    3. 給一對 key A, B，如果 !(A op B) && !(B op A)，則我們說 A 跟 B 等價；而且如果 A 等價 B 且 B 等價 C，則 A 等價 C
* 因為 map, 的 key 都是 unique，亦即任兩個 key 都不等價，如果現在給一對等價 key A, B，那用 A 或 B 去 index map 一定會拿到一樣的 value

* 結論，如果你的 class 定義的 operator< ，或者你提供的作用在兩個這個 class 上的 object 的 callable object，符合上述特性，那這個 class 就可以當 key


* 上面講的東西就是數學啦，如果一個 operator 滿足 strict ordering，就可以用這個 operator 來都出其他所有的比較運算子
    * a < b，就用原本的
    * a == b，就是 !(a < b) && !(b < a)
    * a > b，!(a < b || a == b)，注意這裡的 == 是上面定義的，下面用到的 == 也是
    * a != b，!(a == b)
    * and so on

* 而 C++ 在"看待"你定義的 class object 的順序時的確就是用 < 經過上面的變化兜出其他 operator 的，**如果你的 operator< 的定義有問題，亦即不滿足 strict ordering** 那就會出事
    * 例如現在是對 string 做運算，蛋你用 <= 取代原本預設的 <，那當兩個 string 的內容真的相等時，operator== 反而會是 false!
        * 因為 !(a <= b) && !(b <= a) 會是 false


#### Using a Comparison Function for the Key Type
* 對於關聯容器來說，如何排列 elements 這件事情也是他們本身的一部份，換句話說就是關聯容器型態的一部份
    * 你可以把"排列"這個操作想成關連容器的 private member function
* 所以你在定義關連容器時，其實是要連如何排序 elements 都給出來的
    * 只是因為如果你不給的話，C++ 預設用 element 的 operator< 當作排序方法罷了

* 那如果你想要把那種沒有定義 operator< 的 class 當成 key type，就不能不在宣告時給排序的方法，不給的話就會噴 error
    * 例如 Sales_data
* 你可以對這種 class 額外給關連容器一個 function pointer type
    ```C++
    bool compareIsbn(const Sales_data &lhs, const Sales_data &rhs) {
        return lhs.isbn() < rhs.isbn();
    }
    // bookstore can have several transactions with the same ISBN
    // elements in bookstore will be in ISBN order     
    multiset<Sales_data, decltype(compareIsbn)*> 
        bookstore(compareIsbn);
    ```
    * 角括號內除了給 element type(如果是 map 就給 key 跟 value 的 type，如果是 set 就給 key type 就好)，還要給一個 function pointer type
    * 而宣告時就要把特定的 function 當作參數傳進去，這裡就是把 compareIsbn 當作 bookstore 的參數
    * 用 decltype 免去寫複雜 type 的麻煩，不過記得她不會 decay function type to pointer，所以要加 *
    * 你參數要寫 compareIsbn 或者 &compareIsbn 都可以，因為前者會自動 decay 成後者

### 11.2.3 The pair Type
* \<utility>
* template, 給兩個參數
* public data member, first, second, 對應到 template 的第一跟第二個參數

    ```C++
    pair<string, string> anon; // holds two strings
    pair<string, size_t> word_count; // holds a stringand an size_t
    pair<string, vector<int>> line; // holds stringand vector<int>
    ```
* pair 的 default cstr 會**值初始化** first 跟 second
    * 這也是為什麼我們的 word_count 的 size_t 會初始化成 0 的原因(詳情最前面 map 的筆記)，因為 map 的 element 就是 pair

* 當然可用 list initializer
    ```C++
    pair<string, string> author{"James", "Joyce"};
    ```
* 下表示可用操作
* ![](https://i.imgur.com/pPevyk4.png)

#### A Function to Create pair Objects
* 這裡複習，在 C++11 我們一樣可以對 return value 用 list initialize(return value 跟初始化變數有 87%像)
    ```C++
    pair<string, int> process(vector<string> &v) {
        // process v
        if (!v.empty())
            return {v.back(), v.back().size()}; // list initialize
        else
            return pair<string, int>(); // explicitly constructed return value
    }
    ```
* 舊版 C++ 不能用上面 if 的寫法，要馬就用 else 的，要馬用 make_pair
    ```C++
    if (!v.empty()) 
        return pair<string, int>(v.back(), v.back().size());
    if (!v.empty())
        return make_pair(v.back(), v.back().size());
    ```
    
## 11.3 Operations on Associative Containers
* 關連容器額外定義的 type
* ![](https://i.imgur.com/BNGiEyj.png)

* 對 set 來說
    * key_type 跟 value_type 一樣，跟你宣告 set 時給的 type 一樣
* 對 map 來說
    * key_type 是宣告 map 時給的第一個參數
    * mapped_type 是宣告 map 時給的第二個參數，set 系列沒有 mapped_type
    * value_type 則是 pair<const key_type, mapped_type>

    ```C++
    set<string>::value_type v1; // v1is a string 
    set<string>::key_type v2; // v2 is a string
    map<string, int>::value_type v3; // v3 is a pair<const string, int>
    map<string, int>::key_type v4; // v4 is a string
    map<string, int>::mapped_type v5; // v5 is an int
    ```
    
### 11.3.1 Associative Container Iterators
* 注意，**對關連容器的 iterator 做 dereference 得到的是 value_type**
    * 所以 for(auto &e : map_obj) 才會拿到 pair
    * **而且不能更改 e.first**，因為是 const
    
    ```C++
    // get an iterator to an element in word_count 
    auto map_it = word_count.begin();
    // *map_it is a reference to a pair<const string, size_t>object
    cout << map_it->first; // prints the key for this element
    cout << " " << map_it->second; // prints the value ofthe element
    map_it->first = "new key"; // error: key is const
    ++map_it->second; // ok: we can change the value through an iterator
    ```
#### Iterators for sets Are const
* 然後雖然 set 的 value_type 跟 key_type 一樣，而且也沒有 const，**可是 set 的 iterator 跟 const_iterator 一樣，沒有辦法拿來更改值**

* 其實為什麼 set iterator 跟 map 的 pair 會有 const 行為要從資料結構去想，因為如果可以直接藉由 set 的 iterator 或者 map 的 value_type 的 key 來改值，這樣關連容器內的 element 順序就會亂掉

    ```C++
    set<int> iset = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}; 
    set<int>::iterator set_it = iset.begin();
    if (set_it != iset.end()) {
        *set_it = 42; // error: keys in a setare read-only
    }
    cout << *set_it << endl; // ok: can read the key
    ```
    
#### Iterating across an Associative Container
* 一樣可用 begin end 把關連容器的 elements 掃一遍
    ```C++
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
    * 跟掃 string 或者 vector 這種 sequential container 有 87%像

* 還有一點很重要，order 版本的 set map 系列，**從 begin 掃到 end，掃出來的 element 是排序過的!**

#### Associative Containers and Algorithms
* std::algo 很多都廢掉，要馬是因為
    * map 掃出來的 key 是 const，write algo 會爛掉
* 要馬是因為
    * 很多 algo 會用到 find，std::find 是 linear，對關連容器來說很慢

* 所以關連容器有定義一些自己專屬的 member algo
    * 例如 find，是 O(log n) 或更快

* 如果真的要用 std::algo，多半是傳入關連容器的 iter 進去**當作 input range**，或者**用 inserter 當作 destination**


### 11.3.2 Adding Elements
* 用 insert member
    ```C++
    vector<int> ivec = {2,4,6,8,2,4,6,8}; // ivec has eight elements
    set<int> set2; // empty set
    set2.insert(ivec.cbegin(), ivec.cend()); // set2 has four elements     
    set2.insert({1,3,5,7,1,3,5,7});
    // set2
    now has eight elements
    ```
    * map set 這種 unique 系列的會自動無視重複的 key
    * 如果給定的 range 有重複 key，只有第一個會被插進來

#### Adding Elements to a map
![](https://i.imgur.com/BVfiFGd.png)

* 記得 map 的 element 是 pair，所以 insert 給的(或指向的)參數也要是 pair
* 但是常常我們不會生一個 pair 再插進 map，通常會像下面這樣寫
    ```C++
    // four ways to add word to word_count 
    word_count.insert({word, 1}); 
    word_count.insert(make_pair(word, 1)); 
    word_count.insert(pair<string, size_t>(word, 1));
    word_count.insert(map<string, size_t>::value_type(word, 1));
    ```
    * 這時候 C++11 提供的第一種方法就顯得好用很多

#### Testing the Return from insert
* unique 系列的關連容器，insert return 一個 pair<iter, bool>，iter 指向有給定 key 的 element，bool 提示是否有插入成功
* word counting 可以重寫成這樣
    ```C++
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
* 還有，上面的 ret 是用 auto 宣告的，如果你要看 Legacy code，會變成這樣
    ```C++
    pair<map<string, size_t>::iterator, bool> ret = 
        word_count.insert(make_pair(word, 1));
    ```
    * 總是會有機會看 Legacy code ㄉ(抖

#### Adding Elements to multiset or multimap
* 再舉個用 multi 系列的小例子: 一個作者，多本書
* *insert 一定會插入 element*
    ```C++
    multimap<string, string> authors; 
    // adds the first element with the key Barth, John
    authors.insert({"Barth, John", "Sot-Weed Factor"});
    // ok: adds the second element with the key Barth, John
    authors.insert({"Barth, John", "Lost in the Funhouse"});
    ```
* multi 版本的 insert 就只回傳 iter 而已，指向新插入的 element

### 11.3.3 Erasing Elements
![](https://i.imgur.com/ryr298U.png)
* 後兩個跟 sequential 的一樣
* 第三個是把有這個 key 的 element 刪除，return 會說有幾個 element 被刪除
    * unique 版本的關連容器 always return 0 或 1
    ```C++
    // erase on a key returns the number of elements removed
    if (word_count.erase(removal_word))
        cout << "ok: " << removal_word << " removed\n";
    else
        cout << "oops: " << removal_word << " not found!\n";
    ```
    
### 11.3.4 Subscripting a map
![](https://i.imgur.com/UPOgWKW.png)

* map 跟 unordered_map 可以用 \[]，還有 member function ```at```
* 跟 sequential 容器一樣，可以放 key 到 \[] 內(順序容器是放 position)獲得對應 value
* However, unlike other subscript operators, if the key is not already present, ***a new element is created and inserted*** into the map for that key.
    * 對應這個 key 的 value 會被 value initialized(因為 pair 預設是 value initialize)
    ```C++
    map <string, size_t> word_count; // empty map 
    // insert a value-initialized element with key Anna; then assign 1 to its value
    word_count["Anna"] = 1;
    ```
    * 看清楚，因為 word_count 原本沒有 key 為 "Anna" 的 element，所以 map 會先創造一個 element，key 為 "Anna"，value 用值初始化
    * ***值初始化之後***，才會把 1 assign 給 value
* 因為 \[] 可能會插入 element，所以只有 nonconst map 可以用

#### Using the Value Returned from a Subscript Operation
* 對 map 做 \[] 行為其實跟順序容器差很多
    * 對順序容器做 \[]，跟對順序容器的 iter 做 \*，得到的 type 都是 element type
    * 可是對 map 做 \[] **得到的是 mapped_type，對 map 的 iter 做 \* 得到的是 value_type(element type)

* 不過跟一般 \[] 一樣都是 return lvalue，所以可以拿來讀寫:
    ```C++
    cout << word_count["Anna"]; 
    // fetch the element indexed by Anna; prints 1     
    ++word_count["Anna"];
    // fetch the element and add 1to it
    cout << word_count["Anna"];
    // fetch the element and print it; prints 2
    ```

* 因為 map 的 \[] 操作會按中插入 element，很多時候可以寫很簡潔的 code 但是做很多事情
* 不過還是有問題，例如你只是想要拿到 value_type 可是當給定的 key 不在 map 時不要插入之類的

### 11.3.5 Accessing Elements
![](https://i.imgur.com/4tVvzlJ.png)
* 要用上表的哪個 function 端看你想要找什麼東西
* 對 unique 系列來說，count 永遠回傳 1 或 0
    ```C++
    set<int> iset = {0,1,2,3,4,5,6,7,8,9}; 
    iset.find(1); // returns an iterator that refers to the element with key == 1
    iset.find(11); // returns the iterator ==iset.end()
    iset.count(1); // returns 1
    iset.count(11); // returns 0
    ```
#### Using find Instead of Subscript for maps
* 想找某個 key 卻又不想插入 element 時用 find
    ```C++
    if (word_count.find("foobar") == word_count.end())
        cout << "foobar is not in the map" << endl;
    ```
    
#### Finding Elements in a multimap or multiset
* 可能會存有很多同樣的 keys
* 這些容器會把 key 同樣的 elements 都擺在一起

* 如果要印 order-multi 系列的 container 內同樣 key 的 elements，可以有三種寫法
    ```C++
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
    * 先用 count 找到擁有同樣 key 的 elements 有多少個
    * 再用 find 找到擁有同樣 key 的 elements 內指向第一個的 iter
    * 不斷的對 iter 做++，做 count 次，就可以把這些 elements 掃過一次

#### A Different, Iterator-Oriented Solution
* 第二個方法，都用 iterator，不用 count
* 需要用 lower_bound 跟 upper_bound
    ```C++
    // definitions of authors and search_item as above
    // beg and end denote the range of elements for this author
    for (auto beg = authors.lower_bound(search_item),
                end = authors.upper_bound(search_item);
                beg != end; ++beg)
        cout << beg->second << endl; // print each title
    ```
* 如果 container 裡有這個 key 的 elements，lower_bound 會指向第一個，upper_bound 會指向最後一個的後面一個
    * 精簡的說法就是 \[lower_bound, upper_bound)
* 如果沒有這個 key 的 element，那 lower_bound 跟 upper_bound 會指向同一個地方，**這個地方的意義是，在這個位置插入這個 key 的 element，不會有任何 element 的 order 會爛掉**
    * 可以推論出，如果你找的 key 比 container 內的所有 key 都大，那 lower/upper_bound return 的 iter == container.end()
* 注意 Primer 這裡沒有 demo 任何有關 map 的 iterator 和時會 invalid 的 code，iterator 在 map 插入新東西時都不會 invalid，可是不知道像這種掃 range 的方式，如果插了新的東西，會不會掃到新插入的 element
* 總之，**如果 lower_bound() == upper_bound()，那 key 就不在 container 裡面**

#### The equal_range Function
* return pair<iter1, iter2>，iter1 iter2 形成這個 key 的 range...
    ```C++
    // definitions of authors and search_itemas above
    // pos holds iterators that denote the range of elements for this key
    for (auto pos = authors.equal_range(search_item);
            pos.first != pos.second; ++pos.first)
        cout << pos.first->second << endl; // print each title
    ```
    
### 11.3.6 A Word Transformation Map
* 做一個程式，可以把指定的字串變換成另一個(短語)字串
* input 是兩個檔案，第一個存著變換字串的規則，另一個則是要變換的文字檔
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
    * word_transform: overall processing, 收兩個檔案的 ifstream&
    * buildMap: 吃第一個 wrod_transform 的第一個 ifstream，建立出一個 map that maps string to phrase 
    * transform: 吃一個字串，如果存在規則把字串做轉換則回傳轉換後的字串
```C++
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
* 重點就是 call buildMap 跟 transform
* 還有文字檔一次 getline 一行，這樣處理完這行後直接印 endl，輸出的換行就會跟文字檔一模一樣
* 吃一行進來之後就用第八章學到的，用 istringstream 來處理這一行

#### Building the Transformation Map
```C++
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
* 那個 substr(1) 是為了把 string 跟 phrase 中間的空格刪掉
* 這個轉換如果遇到一個 string 出現多個對應 phrase 的情形，則會使用最後一個 phrase，原因是我們使用了 \[] operator

#### Generating a Transformation
```C++
const string & transform(const string &s, const map<string, string> &m) {
    // the actual mapwork; this part is the heart of the program
    auto map_it = m.find(s);
    // if this word is in the transformation map
    if (map_it != m.cend())
        return map_it->second; // use the replacement word
    else return s; // otherwise return the original unchanged
}
```
* 應該簡單易懂不用解釋...

## 11.4 The Unordered Containers
* 用 hash 的方式存 element
    * 不懂在工三小的詳情演算法大師

* 換句話說 key 要支援 hash；還要支援 ==
* 要讓自定義的 type 能被 hash，這個之後會講
* built-in 跟 string 都已經有內建 hash 惹

* An unordered container is most useful whenwe have a key type for which there is no obvious ordering relationship among the elements.
* These containers are also useful for applications in which the cost of maintaining the elements in order is prohibitive.

* 無序容器的效能取決於 hash function 的品質，這需要很多很ㄎㄧㄤ的測試

* Tip: Use an unordered container if the key type is inherently unordered or if performance testing reveals problems that hashing might solve.

* 基本上 order 版本的操作 unordered 版本都支援(但效能不一樣)
* 所以理論上你把之前寫的 code 用到的有序版本掉換成對應的吳旭版本容器應該都不會錯，可以編
* **但是輸出的東西就不會按照 key order 排列惹!**
* 例如重寫之前的計數問題
    ```C++
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
* **These containers use a hash function tomap elements to buckets.**
* 要存取一個 element，首先要用 hash 算出他會在哪個 bucket，然後去那個 bucket 找 element
* 有同樣 hash value 的 element 都會在同一個 bucket
* 想當然耳，multi 版本的容器，同樣的 key 全部都會在同一個 bucket
* 總之，無序容器的效能 depends on hash 品質，element 數量，以及 bucket 的數量還有大小(滿了會丟到其他 bucket 之類的)
* When a bucket holds several elements, **those elements are searched *sequentially* to find the one we want.**

* 下表示無序容器額外提供的操作，有些還可以拿來更改 bucket size
* ![](https://i.imgur.com/DWqQ7Td.png)

#### Requirements on Key Type for Unordered Containers
* operator==
* hash<key_type>
    * 是個 template
    * built-in 跟 pointer 還有 string，smart pointer 都有
    * 其他 type 或者自定義的 class 都不能用這個 hash template，他不像 container 一樣泛用；相反的，我們要自己定義我們的 hash template，16章會講

* 不過那些不能 hash 的 type 其實也可以在宣告無序容器的時候給定 hash function；就跟宣告 map 時額外給定 sort function 一樣的道理
* 用 Sales_data 舉例，這個 class 不能 hash
* 我們就額外定義 hash function 跟 ==
* 注意，以下宣告的無序容器**第一個參數都是 bucket size**
    ```C++
    size_t hasher(const Sales_data &sd) {
        return hash<string>()(sd.isbn());
    }
    bool eqOp(const Sales_data &lhs, const Sales_data &rhs) {
        return lhs.isbn() == rhs.isbn();
    }
    ```
* 然後這樣宣告無序容器:
    ```C++
    using SD_multiset = unordered_multiset<Sales_data, decltype(hasher)*,
                            decltype(eqOp)*>;
    // arguments are the bucket size and pointers to the hash function and equality operator
    SD_multiset bookstore(42, hasher, eqOp);
    ```
* 如果自定義的 class 有 operator==，可以在宣告無序容器時只給 hash function:
    ```C++
    // use FooHashto generate the hash code; Foomust have an == operator 
    unordered_set<Foo, decltype(FooHash)*> fooSet(10, FooHash);
    ```