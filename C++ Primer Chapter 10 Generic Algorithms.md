# C++ Primer Chapter 10 Generic Algorithms

## 10.1 Overview
* In general, the algorithms do not work directly on a container.
* Instead, they operate by **traversing a range of elements bounded *by two iterators***

* 一個使用 find 這個 algo 的簡單小例子
    ```C++
    int val = 42; // value we’ll look for 
    // result will denote the element we want if it’s in vec,or vec.cend() if not
    auto result = find(vec.cbegin(), vec.cend(), val);
    // report the result
    cout << "The value " << val << (result == vec.cend() ? 
        " is not present" : " is present") << endl;
    ```
    * 這個例子可以看出使用 lib algo 的一些 "pattern"
    * 前兩個參數都是 iterator，指向同一個 container，代表了要處理的資料的範圍
    * 後面跟的參數則 depends on algo you use

* Because find operates in terms of iterators, we can **use the same find function to look for values in any type of container.**
    ```C++
    string val = "a value";
    // value we’ll look for 
    // this call to find looks through string elements in a list
    auto result = find(lst.cbegin(), lst.cend(), val);
    ```
* 除了 iterator，用 begin end 取得 built-in array 的頭尾指標也可以拿來用在 std algo
    ```C++
    int ia[] = {27, 210, 12, 47, 109, 83}; int val = 83;
    int* result = find(begin(ia), end(ia), val);
    ```
* iterator 不一定要指向 container 頭尾，subrange 也可以
    ```C++
    // search the elements starting from ia[1]up to but not including ia[4]
    auto result = find(ia + 1, ia + 4, val);
    ```

#### How the Algorithms Work
* 一樣用 find 來舉例，看 find 怎麼運作
    1. It accesses the first element in the sequence.
    2. 2. It compares that element to the value we want.
    3. If this element matches the one we want, find returns a value that identifies this element.
    4. Otherwise, find advances to the next element and repeats steps 2 and 3.
    5. 5. find must stop when it has reached the end of the sequence.
    6. If find gets to the end of the sequence, it needs to return a value indicating that the element was not found. This value and the one returned from step 3 must have compatible types.
    * 自己看 lol
* 上面的重點只有一個
* find 這個 algo 的操作**不會依賴 container type!**
* 只要有 iterator 可以存取 element 就萬事OK

#### Iterators Make the Algorithms Container Independent, ...
#### ...But Algorithms Do Depend on Element-Type Operations
* iterator 讓你在使用 std algo 時不用管你現在到底是使用什麼 container 存 element
* 但是 algo 還是會 depend on element type
* 例如 find 的第二步使用到的 element 的比較運算子
    * 如果你的 element 無法比較，那就不能使用 find
* 有些 algo 可能會需要 element 有 < 可以用等等
    * 總之每個 algo 會要求你的 element 必須要擁有特定的 operations 才可以使用
* 不過之後會講到，就算你的 element type 沒有提供 algo 要求的 operation，algo 也允許你傳遞一個 callable objec(可以想像成是一般的 function) 來充當這個 operation

#### KEY CONCEPT: ALGORITHMS NEVER EXECUTE CONTAINER OPERATIONS
* Primer p.378
* 這裡主要是在討論用 std algo 為何要使用 iterator 來對 container 的 element 做操作，這背後有幾個隱含意義
    1. 這樣不用管你是對哪個 contianer 操作，這前面講過，container independent
    2. **algo 不會改變 container size**，這大概可以避免其他 iterator 失效吧
    3. 其實你使用 algo 還是有可能會讓 container size 有變化，**但很重要的一點是，那不是 algo 造成的，那是因為你使用的 iterator 是一種叫做 inserter 的 iterator**，他真的會改變 container size，但不管怎麼樣 **algo 裡面都沒有直接更改 container size 的 operation**，這裡把什麼角色該做什麼給分開了，也算是一種抽象化吧


## 10.2 A First Look at the Algorithms
* std algo 超過一百個，不過其實 API 都有一套規則，可以降低學習曲線
* std algo 大部分是作用在一個範圍，叫做 **input range**
    * 其實就是兩個 iterators 包住的範圍
    * 這兩個 iterator 會擺在 std algo 參數的前兩個
        * \[begin, end)
* 接下來就是用 algo 是否讀, 寫, 或重排 range 內的 elements 來分類 algo


### 10.2.1 Read-Only Algorithms
* 之前用到的 find 跟 count 都屬於這類
* 再介紹一個 acculumate
    ```C++
    // sum the elements in vec starting the summation with the value 0
    int sum = accumulate(vec.cbegin(), vec.cend(), 0);
    ```
    * 他不是定義在 algorithm，是 numeric
    * Note: The type of the **third argument to accumulate determines which addition operator is used** and is the type that accumulate returns.

#### Algorithms and Element Types
* 上面 accumulate 的例子有一的隱含意義: iterator 指定的 elements 的 type 一定要可以個第三個參數的 type 做相加
    * 要馬 type match，要馬 convertible
* 舉個 string 的例子
    ```C++
    vector<string> v = {...};
    string sum = accumulate(v.cbegin(), v.cend(), string(""));
    ```
    * elements' type  跟 third arg 可以相加
    ```C++
    // error: no + on const char*
    string sum = accumulate(v.cbegin(), v.cend(), "");
    ```
    * 上面錯誤，因為第三個參數，也就是 ""，string literal，沒有定義 operator+；而且 string 也無法轉換成 char*

* Best Practice: 通常用 read algo，range 最好給 cbegin cend，除非你希望用 algo return 的 iterator 來更改其他東西，這時候就用普通的 begin end
    * 反正上面就 overload，你傳 cbegin 會用 const function 來接
    
#### Algorithms That Operate on Two Sequences
* 前面例子的 algo pattern 長這樣
    * algo(begin, end, value)
* 接下來介紹的長這樣
    * algo(begin1, end1, begin2)
* 這樣的 algo 會操作在兩個 sequences 之上，第一個是 \[begin1, end1)，第二個是 \[begin2, begin2+(end1-begin1))
* ***這種 algo 假定 begin2 開始至少有 end1-begin1 這個多個 elements，如果 begin2 開始 elements 少於這個數量，則是 UB***
* 用 equal 來解釋
    ```C++
    // roster2 should have at least as many elements as roster1
    equal(roster1.cbegin(), roster1.cend(), roster2.cbegin());
    ```
    * 比較 roster1.cbegin() 到 roster1.end() 跟 roster2.cbegin() 到 roster2.cbegin() + roster1.size() 的 elements 是否一樣，一樣則 return true
    * **再強調一次 algo 是作用在 iterator 之上，這意味著兩段 sequence 可以不用同種 container，甚至 element type 也不用一樣，在 equal 的例子，element 只要可以用 == 比較就行了**

    * 例如, roster1 可以是 vector\<string> and roster2 是 list\<const char*>.

### 10.2.2 Algorithms That Write Container Elements
* 好像沒什麼好講的...
* Write algo 是否安全取決於的給的 iterator 參數，讓 iterator 指向合法的位址是 Programmer 的責任；
* 只要 iterator 是合法的，write 就是合法的，至於怎麼合法，前面有講過惹

#### 用 fill 舉例
* 這裡舉了 write algo 的第一種 pattern: 前兩個參數給 range，第三個給 value，把 value 作用在這個 range，例如 fill
    ```C++
    fill(vec.begin(), vec.end(), 0); // reset each element to 0
    // set a subsequence of the container to 10
    fill(vec.begin(), vec.begin() + vec.size()/2, 10);
    ```
    * 只要你 fill 給的 range 是合法的，fill 就是 safe

#### Algorithms Do Not Check Write Operations
* write algo 的另一種 pattern: 用一個 iter 跟 size 來表示 range，通常這種 algo 的命名方式是 algo_n
    ```C++
    vector<int> vec; // empty vector
    // use vec giving it various values
    fill_n(vec.begin(), vec.size(), 0); // reset all the elements of vec to 0
    ```
    * 這裡也一樣，std algo 假定 iter 跟 n 這種 pattern 給定的 range 是合法的；下面給一個 UB 的 code
    ```C++
    vector<int> vec; // empty vector
    // disaster: attempts to write to ten (nonexistent) elements in vec
    fill_n(vec.begin(), 10, 0);
    ```
    
#### Introducing back_inserter
* 既然在講 write algorithm，就先提這種特別的 iterator
* 一種真的會塞 elements，也就是會改變 container 大小的 iterator
* Ordinarily, when we assign to a container element through an iterator, we assign to the element that iterator denotes.
* When we assign through an insert iterator, a new element equal to the right-hand value is added to the container.
* back_inserter takes a reference to a container and returns an insert iterator bound to that container:
    ```C++
    vector<int> vec; // empty vector
    auto it = back_inserter(vec); // assigning through itadds elements to vec *it = 42;
    // vec now has one element with value 42
    ```
    * When we assign through that iterator, *the assignment calls pushback to add an element* with the given value to the container:
    * 這個看一下大概就知道在幹嘛，還有從前面插入的版本(front_inserter)
    * 10.4 再說，先看看怎麼用他更實際的例子
    ```C++
    vector<int> vec; // empty vector // ok: back_inserter creates an insert iterator that adds elements to vec
    fill_n(back_inserter(vec), 10, 0); // appends ten elements to vec
    ```
    * 上面這個就只是把普通 iter 改成 inserter 就安全惹
* 10.4 再好好說明 inserter

#### Copy Algorithms
* 也算是 write algo
* 參數是三個 iter，前兩個是 input range，第三個是 dest，copy 會把 input range 的 element 複製到 dest 代表的 range
* 一樣假定 dest 的空間足夠
* copy return dest 代表的 range 的之後一個位置的 iterator
    ```C++
    int a1[] = {0,1,2,3,4,5,6,7,8,9};
    int a2[sizeof(a1)/sizeof(*a1)]; // a2 has the same size as a1
    // ret points just past the last element copied into a2
    auto ret = copy(begin(a1), end(a1), a2); // copy a1 into a2
    ```
    * 用 sizeof 的技巧讓兩個 array 一樣大，而且不用管 element type
* 其實很多 algo 都有提供一個 "copy version"，命名方式是 algo_copy，用 replace 來舉例
    ```C++
    // replace any element with the value 0 with 42 
    replace(ilst.begin(), ilst.end(), 0, 42);
    ```
    * copy version
    ```C++
    // use back_inserterto grow destination as needed
    replace_copy(ilst.cbegin(), ilst.cend(),
        back_inserter(ivec), 0, 42);
    ```
    * 就是把結果寫到另一個 range 而不是原本的 range

### 10.2.3 Algorithms That Reorder Container Elements
* Primer 舉一個例子，把 vector of strings(未排序) 裡面的 duplicate 拿掉
    * 先排序(sort)在移除重複(unique)
    ```C++
    void elimDups(vector<string> &words) {
        // sort words alphabetically so we can find the duplicates
        sort(words.begin(), words.end());
        // unique reorders the input range so that each word appears once in the
        // front portion of the range and returns an iterator one past the unique range
        auto end_unique = unique(words.begin(), words.end());
        // erase uses a vector operation to remove the nonunique elements
        words.erase(end_unique, words.end());
    }
    ```
    * 這裡自己看課本寫的
    * 比較重要的是關於 unique 之後的結果
    * 再強調一次，algo 沒辦法改 container size，unique 當然也不行
    * **unique 其實是把重複出現過的 element 搬到 container 尾巴的地方，然後回傳一個 iter，指向所有沒重複的 elements 的下一個位址**
    * **注意 unique 只能刪除相鄰重複的**，所以才藥用 sort 把相同的 value 都排在一起

#### Using Container Operations to Remove Elements
* 接續上面的例子，我們沒辦法用 algo 來改變 container size，只能用 container 提供的 operation 來改變，所以最後要用 unique 回傳的 iter 來 call container 的 erase
* Primer 說有一點值得提，就是就算 container 沒有 duplicate，把 unique 回傳的 iter 丟到 erase 還是安全的，因為這時 return value == words.end()，換句話說就是 erase 一個 empty range，安全!

## 10.3 Customizing Operations
* 很多 algo 要求 container 內的 element 需要可以使用某些 operation 才能使用，但這些 algo 也提供特殊的版本，可以讓你不要使用 element 預設的 operation，或者當 element 不知原娜種 operation 時，可以讓 programmer 自己傳入特定的 function 來當作這些 operation。這種 function 被稱為 predicate

### 10.3.1 Passing a Function to an Algorithm
* 繼續用前面消除重複字的例子
* 上面的 code 如果將刪除重複字的 vector 印出來，會得到沒有重複並按照字點順序排列的 string
* 如果我們希望印出來的 string 是先按照常度，再按照字點順序作排列，我們就要使用特別版本的 sort，可以讓我們傳入 predicate

#### Predicates
* 一個 function，return value 可以被當作 condition
    * 只有一個參數的叫做 unary predicate，兩個的叫做 binary predicate
* 如果使用傳入 predicate 版本的 algo，這種 algo 會把 predicate 當作 operation 作用在 container 的 element 之上
* 所以你的 element type 一定要可以轉成 predicate 的參數的 type
##### 用吃 predicate 的 sort 來舉例
* Primer 說 11章會詳細的介紹 sort 的 predicate 需要滿足某些特性，現在只需要知道 predicate 只要可以定義所有可能的 elements 彼此之間的順序就行了(這他媽抽象，反正你懂ㄉ，11章再看)，我們之前寫的 isShorter 就是一個例子
    ```C++
    // comparison function to be used to sort by word length
    bool isShorter(const string &s1, const string &s2) {
        return s1.size() < s2.size(); 
    }
    // sort on word length, shortest to longest
    sort(words.begin(), words.end(), isShorter);
    ```
    * 如果把這個 predicate 丟給 sort，那排出來的字串就會按照長度排序，而不是字點順序
    * 亦即，長度為 3 的會在長度為 4 的前面，以此類推

#### Sorting Algorithms
* 上面的例子做完只會得到按照長度排序的字串，同長度的字串並沒有按照字點順序排列
* 可以用 stable_sort，不過實作方式是這樣
    ```C++
    elimDups(words); // put words in alphabetical order and remove duplicates
    // resort by length, maintaining alphabetical order among words of the same length         
    stable_sort(words.begin(), words.end(), isShorter);
    for (const auto &s : words) // no need to copy the strings
        cout << s << " "; // print each element separated by a space
    cout << endl;
    ```
    * 先用原本的方把讓 string 照字點順序排列
    * 再用 stable_sort，predicate 給 isShorter
    * 這樣的話，string 就會按照長度排列，而一樣長的就會維持原本的順序，而原本的順序就是字點順序
    * 注意，先對長度 sort 再對字串用字點順序就 sable sort 沒有用! 因為對字點順序來說，長一樣的字串是 content 一樣，而不是長度一樣，你一旦對字串用字點順序排序，長度不同的字串一定會被分開
        * 而且你先對長度排序的話，那些真正長一樣的字串雖然會分成一堆(因為同長度)可是不一定會相鄰，這樣 unique 沒辦法正確消除重複字串
        * 不相信的話自己寫code測試LOL

### 10.3.2 Lambda Expressions
* 傳入 algo 的 predicate，參數數量一定要 match algo 的規定
* 但有時候我們希望可以再傳入一些參數，這個用普通 function 做不到
    * 例如 algo 吃 binary predicate，可是我們很希望這個 predicate 吃三個參數，這用普通 function 做不到

* ex 10.13 使用 partition 就是一個例子，它只吃一個 unary predicate，於是我們只能把 5 這個數字寫死在 function 裡面
    * 我們希望有一個更彈性的方法可以讓這個 5 被任意指定

* 再用之前寫的 elimDups 舉例，我們希望再增加一些功能，讓它用長度以及自點順序排好字串之後最後只印出長度大於某 size 的字段
    ```C++
    void biggies(vector<string> &words, 
    vector<string>::size_type sz)
    {
        elimDups(words); // put words in alphabetical order and remove duplicates
        // resort by length, maintaining alphabetical order among words ofthe same length 
        stable_sort(words.begin(), words.end(), isShorter);
        // get an iterator to the first element whose size()is >= sz
        // compute the number of elements with size >=sz 
        // print words of the given size or longer, each one followed by a space
    }
    ```
* 按照上面的框架，我們現在變成要找到第一個長度 >= sz 的字串
* 可以用 algo find_if
    * 吃一個 range 跟 unary predicate
    * find_if return 指向第一個帶到 predicate 為 true 的 element 的 iterator

* 這個 function 很簡單，*如果可以吃兩個參數的話*
    ```C++
    bool greaterThan(const string& s, size_t iz);
    ```
    * 但我們 TMD 就是只能用一個參數RRR

#### Introducing Lambdas
* callable object
    * 到目前為止只學過兩種，function 跟 function pointer
    
    * 還有另外兩種
        * 定義了 operator() 的 class
        * 跟 lambda expressions

* A lambda expression represents a callable unit of code.
* It can be thought of as an unnamed, inline function.
    * has a return type, parameter list, and function body
* Unlike a function, *lambdas may be defined inside a function.*
* A lamba expression has the form
    * ![](https://i.imgur.com/P79D7QT.png)
    * capture list is an (often empty) **list of local variables** **defined** in the enclosing function; 
    * return type, parameter list, and function body are the same as in any ordinary function.
    * a lambda must use a trailing return (§ 6.3.3, p. 229) to specify its return type.
        * 就是用 -> 指定 return type 的語法
* We can omit either or both of the parameter listand return type but must always include the capture list and function body:
    ```C++
    auto f = [] { return 42; };
    ```
    * Here, we’ve defined f as a callable object that takes no arguments and returns 42.
    ```C++
    cout << f() << endl; // prints 42
    ```
    * We call a lambda the same way we call a function by using the call operator
    * 忽略 (parameter list) 等價於這個 lambda exp 不吃參數
    * If we omit the return type, the lambda has an **inferred return type that depends on the code in the function body.**
        * 如果你忽略 lambda exp 的 return type，則 lambda exp 的 return type 會用 lambda exp 的 function body 的 return statement 去推論
        * 如果 body 只是一個 return exp; 那 return type 就是 exp 的 type
        * 否則 return type 就是 void..
    * 總結一句話: Lambdas with function bodies that contain anything other than a single return statement that do not specify a return type return void.
        * lambda exp 如果沒指定 return type 然後 body 又以一句以上的 statements，就 return void

#### Passing Arguments to a Lambda
* 跟一般傳遞參數一樣
    * 不過 lambda exp 不能有 default argument

* 來寫一個跟之前寫的 isShorter 等價的 lambda exp
    ```C++
    [](const string &a, const string &b) 
        { return a.size() < b.size();}
    ```
    * The empty capture list (就是那個 \[])indicates that this lambda will not use any local variables from the surrounding function
* 我們可以把上面這個 lambda 丟到 stable_sort 來代替 isShorter，達到跟之前把字串按照常度排列的效果
    ```C++
    // sort wordsby size, but maintain alphabetical order for words ofthe same size 
    stable_sort(words.begin(), words.end(), 
        [](const string &a, const string &b)
            { return a.size() < b.size();});
    ```
    * 這樣 stable_sort 就會把這個 lambda 當成一般的 function 來 call

    * 這裡也 demo 了將 lambda exp 傳入參數的寫法
        * 那些需要吃 function 的參數，都可以直接丟一個 lambda exp
        * 記住 lambda exp 就是個 exp，exp 可以出現的位置他都可以出現

#### Using the Capture List
* 前面都沒用到這個功能，這個功能才能真正幫我們解決想要額外傳 argument 的問題 LOL
    * 希望可以丟一個 predicate 給 find_if，但是這個 predicate 可以傳入一個任意長度的 sz，然後檢視 string.size 有沒有大於 sz
* 先講 capture list 怎麼用
* 如果你的 lambda 想要使用 enclosing function 的 local variable，你必須在 \[] 裡面指定那些變數
* 可以這樣寫
    ```C++
    [sz](const string &a)
        { return a.size() >= sz; };
    ```
    * sz 是定義在前面說的 ```biggies``` 那個框架
    * 如果沒有在 \[] 內指定 sz 則會噴 error
    ```C++
    // error: sz not captured
    [](const string &a)
        { return a.size() >= sz; };
    ```
    
#### Calling find_if
* 直接把上面寫的 lambda 丟到 find_if 就可以解決我們的問題惹
    ```C++
    // get an iterator to the first element whose size()is >= sz
    auto wc = find_if(words.begin(), words.end(), 
        [sz](const string &a)
            { return a.size() >= sz; });
    ```
    * The call to find_if *returns an iterator to the first element that is at least as long as the given sz*, or a copy of words.end() if no such element exists.
* 有了 wc 這個 iterator，我們就可以計算在 words 裡長度 >= sz 的字串有多少
    ```C++
    // compute the number of elements with size >= sz 
    auto count = words.end() - wc;
    cout << count << " " << make_plural(count, "word", "s")
         << " of length " << sz << " or longer" << endl;
    ```
    * make_plural 只是之前定義的 function，決定要不要印複數而已

#### The for_each Algorithm
* 吃一個 range 跟 predicate，對每個 element call 那個 predicate
* 可以用它把長度 >= sz 的字串印出來
    * range 傳入 wc, words.end()
    ```C++
    // print words ofthe given size or longer, each one followed by a space
    for_each(wc, words.end(),
        [](const string &s)    
            {cout << s << " ";});
    cout << endl;
    ```
    * 這裡提一下，lambda look up name 的方式除了 capture list 以外就跟普通 function 一樣，所以他當然可以用 cout
    * **lambda 也可以直接使用 function 定義的 static variable，不用在 \[] 內指定**

#### Putting It All Together
* 把上面 ```biggies``` 的框架完成:
    ```C++
    void biggies(vector<string> &words, vector<string>::size_type sz)
    {
        elimDups(words); // put words in alphabetical order and remove duplicates
        // sort wordsby size, but maintain alphabetical order for words of the same size 
        stable_sort(words.begin(), words.end(),     
            [](const string &a, const string &b)
                { return a.size() < b.size();});
        // get an iterator to the first element whose size() is >= sz
        auto wc = find_if(words.begin(), words.end(),
            [sz](const string &a)
                { return a.size() >= sz; });
        // compute the number of elements with size >= sz
        auto count = words.end() - wc;
        cout << count << " " << make_plural(count, "word", "s")
             << " of length " << sz << " or longer" << endl;
        // print words of the given size or longer, each one followed by a space
        for_each(wc, words.end(),
            [](const string &s)
                {cout << s << " ";});
        cout << endl;
    }
    ```

### 10.3.3 Lambda Captures and Returns
* 其實我們 define lambda 的時候，compiler 會生成一個 class
    * 14 章會說這個 class 到底怎麼生成的
* 當我們在丟給某 function 一個 lambda exp 時，則會同時定義 class 跟產生這個 class 的 object，這個 object 就會是某 function 的 argument
* 同樣地，當我們用 auto f = lambda exp 的形式時，f 會是那個 lambda 的 class 的 object
* 那這個 class 到底有什麼 member 呢? 答案就是 capture list \[] 裡面寫的東西!
    * Like the data members of any class, **the data members of a lambda are initialized when a lambda object is created.**
    * 那些 member 就跟普通的 class object 的 member 一樣，**在 lambda object 被創造時被初始化**

#### Capture by Value
![](https://i.imgur.com/9wXk7zj.png)

* Similar to parameter passing, **we can capture variables by value or by reference.**
* capture list 裡面如果要用 capture by value 的方式，那那個 object 一定要可以 copy
    * 例如 istream 就不能用 capture by value
* **Unlike parameters, the value of a captured variable is copied when the lambda is created, not when it is called:**
    ```C++
    void fcn1() {
        size_t v1 = 42; // local variable
        // copies v1into the callable object named f 
        auto f = [v1] { return v1; };
        v1 = 0;
        auto j = f(); // j is 42; f stored a copy of v1 when we created it
    }
    ```
    * 注意 f 在被創造時就 copy v1 的 value 了，而不是把 f 當成 function 拿來 call 時才 copy v1 的 value

#### Capture by Reference
```C++
void fcn2() {
    size_t v1 = 42; // local variable
    // the object f2 contains a reference to v1 
    auto f2 = [&v1] { return v1; };
    v1 = 0;
    auto j = f2(); // j is 0; f2 refers to v1; it doesn’t store it
}
```
* A variable captured by reference acts like any other reference.
* When we use the variable inside the lambda body, we are using the object to which that reference is bound.

* 如果你要 capture by reference，你就要注意到你 bind 的物件的 lifetime，你自己要確保你在 call lambda 時你 bind 的物件還活著，否則就是 UB
* 用上面 for_each 來印長度大於給定 sz 的字串來舉例
* 之前的 lambda 是直接拿 cout 來用，如果想要寫得更 general 就必須 capture 一個 reference to ostream
    ```C++
    void biggies(vector<string> &words, vector<string>::size_type sz, ostream &os = cout, char c = ' ')
    {
        // code to reorder words as before
        // statement to print count revised to print to os
        for_each(words.begin(), words.end(),
            [&os, c](const string &s)
                { os << s << c; });
    }
    ```
    * ostream 不能 copy 所以一定要用 reference(或者 pointer to ostream，不過你懂ㄉ...)
* 還有另一個例子是不該使用 capture by reference 的
* 你的 function 也可以回傳 lambda object，這時可想而知這個 lambda object 不能 capture by reference，因為回傳的的 function 的 local variable 已經消失了，capture list 都是 copy 或 bind local variable 的，這樣是 UB

* ADVICE: KEEP YOUR LAMBDA CAPTURES SIMPLE
    * 這裡在說 capture by reference 或者 pointer 會有什麼需要注意的，可以的話盡量不要這樣用
    * Primer p.394，就不貼上來了

#### Implicit Captures
* 另一種 capture list 的語法
* \[=] 告訴 compiler lamda 內用到的 function local variable 都是用 capture by value，而 \[&] 則是 capture by reference
    ```C++
    // sz implicitly captured by value
    wc = find_if(words.begin(), words.end(),     
        [=](const string &s)
            { return s.size() >= sz; });
    ```
* 你也可以兩個混用
    ```C++
    void biggies(vector<string> &words, vector<string>::size_type sz, ostream &os = cout, char c = ' ')
    {
        // other processing as before 
        // os implicitly captured by reference; c explicitly captured by value         
        for_each(words.begin(), words.end(),
            [&, c](const string &s)
                { os << s << c; });
        // os explicitly captured by reference; c implicitly captured by value     
        for_each(words.begin(), words.end(),
            [=, &os](const string &s)
                { os << s << c; });
    }
    ```
    * \[] 內第一個 by implicit capture，之後擺 explicit
    * implicit 跟 explicit 的 capture 方式不能一樣
        * 如果 implicit 是 by value，explcit 就要是 reference，反之亦然

#### Mutable Lambdas
* lambda 預設不能更改 capture by value 的 variable 的值，如果要做更改的話，要加上 keyword ```mutable```，加在 parameter list 後面，而且 para list 不能省略
    ```C++
    void fcn3() {
        size_t v1 = 42; // local variable
        // f can change the value of the variables it captures
        auto f = [v1] () mutable { return ++v1; };     
        v1 = 0;
        auto j = f(); // j is 43
    }
    ```
* capture by reference 的 variable 能否更改只取決於 bind 的 object 是否為 const
    ```C++
    void fcn4() {
        size_t v1 = 42; // local variable
        // v1 is a reference to a nonconst variable 
        // we can change that variable through the reference inside f2
        auto f2 = [&v1] { return ++v1; };
        v1 = 0;
        auto j = f2(); // j is 1
    }
    ```
    
#### Specifying the Lambda Return Type
* 有時候你的 lambda 不只是一句 return，還有其他 statement；
* 這樣的話如果不給 return type，compiler 就會當成是 return void，而這時如果你的 lambda body 內有 return value，就會噴 error
* 用 std algo ```transform``` 來舉例
* ```transform``` 吃四個參數，前兩個是 input range，第三個是 dest，第四個是 predicate
* 附帶一提，```transform``` 如果 dest 跟 input range 一樣，那就會直接把 element 丟到 predicate 的塞回 element 的位置，有點 ```replace ``` 的感覺
    * ```transform``` 會把每個 element 都丟到 predicate，然後把 return value 塞到 dest 指定的 range
    ```C++
    transform(vi.begin(), vi.end(), vi.begin(), 
        [](int i) { return i < 0 ? -i : i; });
    ```
    * 上面這個 lambda 一樣不用指定 return type，因為 lambda body 內只有一個 return statement
    * 可是如果寫成等價的 if else 形式就會噴 error
    ```C++
    // error: cannot deduce the return type for the lambda
    transform(vi.begin(), vi.end(), vi.begin(),
        [](int i) 
            { if (i < 0) return -i; else return i; });
    ```
    * lambda body 內不是單一一個 return statement，所以 compiler 預設 return type 是 void，還是 body 內還是有 return value!
    * 要改成這樣
    ```C++
    transform(vi.begin(), vi.end(), vi.begin(), 
        [](int i) -> int
        { if (i < 0) return -i; else return i; });
    ```
    
### 10.3.4 Binding Arguments
* 如果 predicate 要用 N 遍，或者我們需要的 predicate 需要很多 statement 才能完成，我們還是要寫成 function，可是這樣就有 argument 數量不合的問題，所以 C++11 有 bind 可以用，很ㄎㄧㄤ
* 簡單來說，要用一個 function 來替換掉一個 capture list 不為空的 lambda 很麻煩

```std::bind```
* 定義在 \<functional>
* The bind function can be thought of as **a general-purpose function adaptor**
    ```C++
    auto newCallable = bind(callable, arg_list);
    ```
    * ```newCallable``` 是一個 callable object，```arg_list``` 是要傳給 ```callable``` 的參數
    * 換句話說，當我們 call ```newCallable```，```newCallable``` 會去 call ```callable```，然後把 ```arg_list``` 傳給 ```callable```

* ```arg_list``` 裡面會有一種叫做 **placeholder** 的東西，長得像 \_n，其中 n 是正整數，他們代表了要傳給 ```newCallable``` 的參數，n 就代表這是第幾個參數
* 你一定覺得我沒在說人話，我用例子來舉例
    ```C++
    // check6 is a callable object that takes one argument of type string
    // and calls check_size on its given string and the value 6
    auto check6 = bind(check_size, _1, 6);
    ```
    * 還記的 check_size 是一個接受 string 跟 size_t 的 function
    * 如果按照上面的方式宣告 check6，那 check6 就會是一個 callable object，除此之外它吃一個參數，型態跟 check_size 第一個參數一樣，也就是 string
    * 之後如果拿 check6 來 call 就會變這樣
        ```check6(string_obj);```
    * check6 的第一個參數，也就是 string_obj，會被丟到 check_size 對應的_1 的位置，所以實際上就等於 call
        ```check_size(string_obj, 6);```
        
* 更一般的想法是，**當你用 ```bind``` 回傳的 value 來宣告 callable object 時，你在 ```bind``` 的 ```arg_list``` 內放了幾個 placeholder，你宣告的 callable object 就會有幾個參數**
* 再寫一個例子
    ```C++
    string s = "hello";
    bool b1 = check6(s);
    // check6(s) calls check_size(s, 6)
    ```
    
* 於是我們就可以利用 bind 做到跟 lambda capture list 一樣的事情惹!
    ```C++
    auto wc = find_if(words.begin(), words.end(), 
        [sz](const string &a) {...})
    auto wc = find_if(words.begin(), words.end(), 
        bind(check_size, _1, sz));
    ```
    * This call to bind **generates a callable object that binds the second argument of check_size to the value of sz.**

#### Using placeholders Names

* 那些 \_n 定義在 std::placeholders 裡面
    ```C++
    #include <functional>
    using std::placeholders::_1;
    using namespace std::placeholders;
    ```
    
#### Arguments to bind
* bind 產生出來的 callable object 參數傳遞方式是這樣:
    * 第一個參數會被傳到 bind call 內填入 \_1 的位置
    * 以此類推
* 給個例子
    ```C++
    // g is a callable object that takes two arguments
    auto g = bind(f, a, b, _2, c, _1);
    g(var1, var2); // == f(a, b, var2, c, var1);
    ```

#### Using to bind to Reorder Parameters
* 我們甚至可以用 bind 來讓一個已經寫好的 function 意義相反
    ```C++
    // sort on word length, shortest to longest 
    sort(words.begin(), words.end(), isShorter);
    // sort on word length, longest to shortest
    sort(words.begin(), words.end(), bind(isShorter, _2, _1));
    ```

#### Binding Reference Parameters
* bind 給的 arg_list 裡面，如果不是 placeholder，預設是用 copy 的方式
* 可是就跟使用 lambda 或者某些 type 的限制一樣，我們希望可以用 reference
* 用之前的小例子舉例
    ```C++
    // os is a local variable referring to an output stream 
    // c is a local variable of type char     
    for_each(words.begin(), words.end(),
        [&os, c](const string &s)
            { os << s << c; });
    ```
* lambda 改寫成 function 很簡單
    ```C++
    ostream &print(ostream &os, const string &s, char c) {
        return os << s << c;
    }
    ```
* 可是直接用之前一樣的方式用 bind 會噴 error
    ```C++
    // error: cannot copy os     
    for_each(words.begin(), words.end(), bind(print, os, _1, ' '));
    ```
    * os 不能用 copy 的

* 為此我們需要一個 std 的函數，ref...
    ```C++
    for_each(words.begin(), words.end(), 
        bind(print, ref(os), _1, ' '));
    ```
    * The ref function returns an object that contains the given reference and that is itself copyable.
    * 也有 const 版本的 cref，一樣定義在 \<functional> 裡面

* 你問 ref 是三小? 我也不知道 LOL，reference_wrapper? 之後再查查

#### BACKWARD COMPATIBILITY: BINDING ARGUMENTS
* ㄜ...完全不能理解，感覺也不是很重要

## 10.4 Revisiting Iterators
* 除了 container_type\<T>::iterator 以外，\<iterator> 裡面還定義了很多種特殊ㄉ
    * Insert iterators
        * 可以用它來增加 container 的 elements
    * Stream iterators
        * 作用在 stream object 上的
    * Reverse iterators
        * 倒過來掃 container 的 iterator，除了 forward_list 之外的 container 都有
    * Move iterators
        * 最後一個 13 章才會講

### 10.4.1 Insert Iterators
![](https://i.imgur.com/IgINgPN.png)

* iterator adapter
* takes a container
* and yields an iterator that adds elements to the specified container.
* When we assign a value through an insert iterator, the **iterator calls a container operation to add an element at a specified position** in the given container.
* 三種
    * inserter
        * 吃 container 跟 iter
        * 插入 element 都是 call insert，插在 iter 前一個位智
    * front_inserter
        * call container::push_front，換句話說沒有 push_front 就不能用
    * back_inserter
        * call push_back
* 注意 inserter 怎麼操作的
    ```C++
     auto it = inserter(c, iter);
     *it = val;
     ```
     * 上面等價於
     ```C++
     it = c.insert(it, val); // it points to the newly added element
     ++it; // increment itso that it denotes the same element as before
     ```
* 你可能會覺得 front_inserter(c) 跟 inserter(c, c.begin()) 很像，但其實差很多
    ```C++
    list<int> lst = {1,2,3,4};
    list<int> lst2, lst3; // empty lists
    // after copy completes, lst2 contains 4 3 2 1 
    copy(lst.cbegin(), lst.cend(), front_inserter(lst2));
    // after copy completes, lst3 contains 1 2 3 4
    copy(lst.cbegin(), lst.cend(), inserter(lst3, lst3.begin()));
    ```
    * front_inserter 永遠插在 container 最前面
    * inserter 就算你一開始指定在 container 開頭，一旦插入 element 之後 inserter 就不是指在 container 開頭了，而是開頭的下一個位置

### 10.4.2 iostream Iterators
* istream_iterator
* ostream_iterator
* These iterators treat their corresponding stream **as a sequence of elements of a specified type.**

* 這個搭配 algo 用可以直接把答案丟到 output 而不是 container，其實感覺蠻讚的(?

#### Operations on istream_iterators
![](https://i.imgur.com/op2ML8D.png)

* When we create a stream iterator, we must specify the type of objects that the iterator will read or write.
    ```istream_iterator<int>```
    * istream_iter 會用到 >> operator，所以你 <> 內給的 type 需要實作 >>
* 你可以用 default initialize 它，概念上它會指向 EOF
* 也可以給一個 istream object 當參數，它就會綁定那個 istream object
    ```C++
    istream_iterator<int> int_it(cin); // reads ints from cin
    istream_iterator<int> int_eof; // end iterator value
    ifstream in("afile");
    istream_iterator<string> str_it(in); // reads stringsfrom "afile"
    ```
* 上面的 code 又 demo 了一次繼承的好處: istream_iterator 可以綁 ifstream!!
* 也可以這樣寫來拿 istream_iterator 來讀東西到 container
    ```C++
    istream_iterator<int> in_iter(cin); // read ints from cin 
    istream_iterator<int> eof; // istream "end" iterator
    while (in_iter != eof) // while there’s valid input to read
    // postfix increment reads the stream and returns the old value of the iterator
    // we dereference that iteratorto get the previous value read from the stream
        vec.push_back(*in_iter++);
    ```
    * The postfix increment advances the stream by reading the next value but returns the old value of the iterator.
    * That old value contains the previous value read from the stream. We dereference that iterator to obtain that value.
* 是不是覺得有點多此一舉? 真正ㄎ一的 code 在下面!
    ```C++
    istream_iterator<int> in_iter(cin), eof; // read ints from cin
    vector<int> vec(in_iter, eof); // construct vec from an iterator range
    ```
    * This constructor reads cin until it hits end-offile or encounters an input that is not an int.

#### Using Stream Iterators with the Algorithms
* 之後會講到 iterator 其實可以按照他們能做的操作來做分類，有些 algo 會要求傳進來的 iter 需要特定的類別
* stream iterator 是屬於最廢的那種，不過他還是可以用在一些 algo 上面
    ```C++
    istream_iterator<int> in(cin), eof;
    cout << accumulate(in, eof, 0) << endl;
    ```
    上面的 accumulate 會把 stdin 的 int 吃光然後做加總
    
#### istream_iterators Are Permitted to Use **Lazy Evaluation**
* When we bind an istream_iterator to a stream, **we are not guaranteed that it will read the stream immediately.** The implementation is permitted to delay reading the stream until we use the iterator.
    * 很自然阿，IO 要丟給 OS 優化
* We are guaranteed that before we dereference the iterator for the first time, the stream will have been read.
* **如果你綁了一個 istream_iterator 到 stream 上，然後什麼 read 都沒做，或者你有一些 stream 會互相同步(第八張講的 tie)，這時候就要很注意惹**
    * 遇到問題再說吧zzz

#### Operations on ostream_iterators
![](https://i.imgur.com/Ho1CT6i.png)
* 一樣，\<T> 要定義 <<
* 初始化一定要給 ostream
* 上面的 d 要是 C string
* ostream_iterator 沒有 EOF 的概念 
    ```C++
    ostream_iterator<int> out_iter(cout, " ");
    for (auto e : vec)
        *out_iter++ = e; // the assignment writes this element to cout
    cout << endl;
    ```
* 其實按照上面 table 的定義，我們可以這樣寫，這樣是等價的:
    ```C++
    for (auto e : vec)
        out_iter = e; // the assignment writes this element to cout
    cout << endl;
    ```
    * 可是所有 iterator 就只有 i/ostream_iterator 可以不用 dereference 跟 ++，這樣寫不具有通用性，而且不熟 iterator 的人會不知道 i/ostream_iterator 可以這樣用

* 真的要把 container elements 印到 output 也有很簡潔的方法，可以用 copy
    ```C++
    copy(vec.begin(), vec.end(), out_iter);
    cout << endl;
    ```
    
#### Using Stream Iterators with Class Types
* Primer 用 Sales_item 來舉例，因為它 >> << 都有定義
* 可以不用用 cin 那些，而是用 stream_iterator 來操作
    ```C++
    istream_iterator<Sales_item> item_iter(cin), eof;
    ostream_iterator<Sales_item> out_iter(cout, "\n");
    // store the first transaction in sum and read the next record
    Sales_item sum = *item_iter++;
    while (item_iter != eof) {
        // if the current transaction (which is stored in item_iter) has the same ISBN
        if (item_iter->isbn() == sum.isbn())
            sum += *item_iter++; // add it to sum and read the next transaction
        else {
            out_iter = sum; // write the current sum
            sum += *item_iter++;  // read the next transaction
        }
        
    }
    out_iter = sum; // remember to print the last set of records
    
    
### 10.4.3 Reverse Iterators
![](https://i.imgur.com/QS3vfLN.png)
* 注意上圖各種 offset by one!!
* **普通版本跟 reverse 版本會沒對齊，背後的原因是要讓操作他們的方法一致，你不用因為你現在操作的事 reverse 版本就要增加或減少一個 offset**
* **讓 reverse 的 ++ -- 意義倒過來也是一樣的原因**
* traverses a container backward
* A reverse iterator *inverts the meaning of increment (and decrement)*.
    * Incrementing (++it) a reverse iterator moves the iterator to the previous element;
    * derementing (--it) moves the iterator to the next element.

* We obtain a reverse iterator by calling the rbegin, rend, crbegin, and crend members.
* return reverse iterators **to the last element** in the container **and one "past" (i.e., one before) the beginning of the container.**

* 把 container elements 倒過來印的小例子
    ```C++
    vector<int> vec = {0,1,2,3,4,5,6,7,8,9}; 
    // reverse iterator ofvector from back to front 
    for (auto r_iter = vec.crbegin(); // binds r_iterto the last element
            r_iter != vec.crend(); // crend refers 1 before 1st element
            ++r_iter) // decrements the iterator one element
        cout << *r_iter << endl; // prints 9, 8, 7,. .. 0
    ```
    
* 上面說的讓操作 iterator 的 algo 不用管 iter type 的好處，可以用下面的例子 demo
    ```C++
    sort(vec.begin(), vec.end()); // sorts vecin "normal" order
    // sorts in reverse: puts the smallest element at the end ofvec
    sort(vec.rbegin(), vec.rend());
    ```
    * 第二個 sort 直接把 vector 倒過來排序!

#### Reverse Iterators Require Decrement Operators
* 只有定義了 ++ 跟 -- 的 iterator 才有 reverse 版本
    * 換句話說 forward_list 跟 stream 的 iterator 沒有 reverse 版本

#### Relationship between Reverse Iterators and Other Iterators
* 假設使用 find 找一個字串的第一個 word，用逗號來找
    ```C++
    // find the first element in a comma-separated list
    auto comma = find(line.cbegin(), line.cend(), ',');
    cout << string(line.cbegin(), comma) << endl;
    ```
* 簡單明瞭；可是如果要找最後一個 word，貌似可以用 reverse iterator
    ```C++
    // find the last element in a comma-separated list
    auto rcomma = find(line.crbegin(), line.crend(), ',');
    // WRONG: will generate the word in reverse order
    cout << string(line.crbegin(), rcomma) << endl;
    ```
* 可是這樣是錯的!，這樣印出來的最後一個 word 內容會相反!
    * 假設 input: FIRST,MIDDLE,LAST
    * 印出來的會是: TSAL
* 下圖是示意圖
* ![](https://i.imgur.com/VVvGulO.png)

* 可以用 reverse iterator 提供的 member function 來解決
    ```C++
    // ok: get a forward iterator and read to the end of line
    cout << string(rcomma.base(), line.cend()) << endl;
    ```
    * base 會 return 對應的普通 iterator，注意看上面的示意圖，base 跟 rcomma 差一格!
* These differences are needed to ***ensure that the range of elements, whether processed forward or backward, is the same.***
* 其實你也可以說，不管是 begin, end 還是 rbegin, end，都是左閉區間的形式
    * \[begin, end), \[rbegin, rend)
    * 左閉區間，而且要指向同一個 range 的 elements，意味著對應的 begin 跟 end 要 offset by 1


## 10.5 Structure of Generic Algorithms
* 第一種是，用他們需要什麼樣的 iterator，叫做 iterator categories，來分類 std algo
* ![](https://i.imgur.com/YTa1dXY.png)
* 第二種是看 std algo 是否讀，寫，重排 elements，在附錄 A 有
* std algo 也有遵循一些 parameter passing 跟 naming convention，講完 iterator categories 之後會說

### 10.5.1 The Five Iterator Categories
* 跟 container 一樣 iterator categories 就是用 iterator 的操作來分類 iterator 的
    * container 則是用操作分 sequential, associative 等等
* 有意些操作是所有 iterator 共有，有些是特定類型的才有
* iterator 還有分階級，高階級的支援所有低階級的操作(除了 output iterator)
* **標準在定義 st algo 時也定義了傳進來的每個 iterator 參數至少需要是什麼等級的 iterator**
    * 例如 find 只需要 input iterator，replace 前兩個參數需要 forward iterator，第三個需要 output iterator

* 你如果傳進比較廢的 iterator 是 error
    * Primer 說很多 compiler 甚至不會噴 error?!

#### The Iterator Categories
##### Input iterators
* 支援 operations:
    * ++
    * ==, !=
    * \*(dereference)，**可是只能出現在 assignment 的右手邊!**
    * ->(dereference 的變形)，一樣只能在右手邊
* 至於它什麼時候會 invalid depends on 他指向的 container (或 stream)，或者本身對 iterator 的操作
    * 例如改變 container size
    * 或者對某個 istream_iterator 做 ++，其他指向同 stream 的 iterator 就會 invalid
* 使用這類型的 algo 有 find，accumulate 等等

##### Output iterators
* 他支援的操作可以想像成是 input iterator 的補集
    * ++
    * \*，**但是可以出現在 assignment 的左手邊**
* 使用這類型的 algo 有 copy 的第三個參數；ostream_iterator 也算是 output iterator
##### 以上兩種 iterator 都只能用一次(single pass，對一個 element 做一次讀或寫)，之後就不保證 valid，所以你不可以把 iterator 存起來(save state of a iterator)

##### Forward iterators
* 可以對一個 sequence 做讀跟寫
* 雖然只能往一個方向移動，可是可以對同一個 element 做多次讀跟寫(multi-pass)
    * Therefore, we can use the saved state of a forward iterator.
    * 可以把 iterator 存起來之後做讀寫
* replace 就用 forward_iterator，forward_list 回傳的也是 foward iterators

##### Bidirectional iterators
* 除了支援 forward iterator 所有操作，還額外有 -- 可以用
* algo reverse 就需要 bidirectional iterators，
    * 還有所有 container 除了 forward_list 以外回傳的 iterator 都符合 bidirectional iterators 的要求(注意我不是說他們回傳的是 bidirectional iterators ㄛ)

##### Random-access iterators
* provide **constant-time access to any position in the sequence.**
* 除了支援 bidirectional 的操作還支援以下的
    * The relational operators (<, <=, >,and >=)，比較兩個 iterator 的位置(要指向同個 container，否則 UB)
    * addition and subtraction(+, +=, -, -=)
        * iter *op* int
    * -
        * iter - iter
    * iter\[n]
        * syntax sugar for *(iter + n)

* sort 就需要 random-access iterators
    * array, deque, vector, string 回傳的都是 random-access iterators
    * **指向 built-in array 的 pointer 也算ㄛ!!**
    
### 10.5.2 Algorithm Parameter Patterns
* Most of the algorithms have one of the following four forms:
    * alg(beg, end, *other* *args*);
    * alg(beg, end, dest, *other args*);
    * alg(beg, end, beg2, *other args*);
    * alg(beg, end, beg2, end2, *other args*);

* beg end 代表 input range，
* dest 代表第二個 range，假設想寫多少 element 都可以
* beg2 代表第二個 range，假設範圍大小跟 beg end 一樣
* beg2 end2 也代表第二個 range


#### Algorithms with a Single Destination Iterator
* 就是使用 dest 的，algo 假設寫入任意數量的 elements 是安全的
* 更多時候 dest 其實是 inserter 或者 ostream_iterator

#### Algorithms with a Second Input Sequence
* 使用 beg2 或 beg2 end2 的 algo
* 使用 beg2 單一參數的 algo 假設從 beg2 開始寫入 end-begin 數量個 elements 是安全的
* 其他讀寫或計算就 depends on algo

#### 10.5.3 Algorithm Naming Conventions
* These conventions deal with
    * how we supply an operation to use in place of the default < or == operator
    * whether the algorithm writes to its input sequence or to a separate destination.

#### Some Algorithms Use Overloading to Pass a Predicate
```C++
unique(beg, end); // uses the == operator to compare the elements
unique(beg, end, comp); // uses comp to compare the elements
```
* 直接用 overloading
* 因為參數數量不一樣，不可能 ambiguous，所以直接 overloading

#### Algorithms with \_if Versions
* 有些 algo 第三個參數是一個 value
* 他們會提供一個 algo_if 的版本，第三個參數改為 predicate
```C++
find(beg, end, val); // find the first instance ofvalin the input range
find_if(beg, end, pred); // find the first instance for which predis true
```
* 因為參數數量一樣，又有可能會 type collision(雖然很瞎)，標準索性不用 overloading

#### Distinguishing Versions That Copy from Those That Do Not
* 有些更改 input range elements 排列的 algo 會提供一個 algo_copy 的版本，把重排的 elements 寫到另一個 range
```C++
reverse(beg, end); // reverse the elements in the input range
reverse_copy(beg, end, dest);// copy elements in reverse order into dest
```
* 有些 algo 甚至提供了 algo_copy_if 的版本，額外吃兩個參數，一個是代表第二個 range 的 iterator，一個是 predicate
```C++
// removes the odd elements from v1 
remove_if(v1.begin(), v1.end(), [](int i) { return i % 2; });
// copies only the even elements from v1into v2; v1 is unchanged
remove_copy_if(v1.begin(), v1.end(), back_inserter(v2),
        [](int i) { return i % 2; });
```

## 10.6 Container-Specific Algorithms
* Unlike the other containers, **list and forward_list define several algorithms as members.**
    * sort, merge, remove, reverse,and unique.

* 首先，std::sort 不能用在這兩種 container，因為這兩種提供的 iterator 是 bidirectional，std::sort 需要 random-access

* 其他種 std::algo 可以用，可是 performance 很糟
* 用 list 個別定義的版本會用 list 的方法來 swap element(把 link 交換 blablabla，詳情資料結構)，效能會好很多
* 其他 list 沒定義的 algo 都可以直接套在 list，而且效能不會有差異
* 下面是 list 獨自定義的 algo
* ![](https://i.imgur.com/3aqUS01.png)

#### The splice Members
* list.splice()
* 這是專屬於 list 這種 data structure 才有的演算法，所以一般 \<algorithm> 沒有 splice
* ![](https://i.imgur.com/BsHsGws.png)

#### The List-Specific Operations Do Change the Containers
* 之前說 std::algo 不會改變 container size，可是 list 個別定義的 algo 會!
* 
