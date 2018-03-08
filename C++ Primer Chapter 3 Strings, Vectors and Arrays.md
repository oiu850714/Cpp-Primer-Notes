# C++ Primer Chapter 3 Strings, Vectors and Arrays
## 3.1 Namespace using Declarations
* 大概會用了
* **Headers Should Not Include using Declarations**
    * 所以你在寫 header 的時候就不能偷懶，不然不小心發生 collision 就會很想死
    * The reason is that the contents of a header are copied into the including program’s text.
    * If a header has a using declaration, then every program that includes that header gets that same using declaration. 
    * As a result, **a program that didn’t intend to use the specified library name might encounter unexpected name conflicts.**
    * using namespace_name::name 其實是把 name 搬到 global scope
* Primer 之後的例子會假設他們用到的 name 已經用 using 宣告了
    * 而且不會把 using 的地方寫出來，連 include 什麼 header 也不會寫了，節省空間

## 3.2 Library string Type
* 這章先講最基本使用，§9.5 再詳細講
### 3.2.1 Defining and Initializing strings
* 一般初始化 string 的幾個方式：
    ```C++
    #include <string>
    using std::string;

    int main() {
      string s1;          // default initialization; s1 is the empty string
      string s2 = s1;     // s2 is a copy of s1
      string s3 = "hiya"; // s3 is a copy of the string literal
      string s4(10, 'c'); // s4 is cccccccccc
    }
    ```
* 這裡突然遇見了大魔王，**Direct and Copy Forms of Initialization**
    ```C++
      string s5 = "hiya"; // copy initialization
      string s6("hiya");  // direct initialization
      string s7(10, 'c'); // direct initialization; s7is cccccccccc
      string s8 = string(10, 'c'); // copy initialization; s8 is cccccccccc
    ```
    * 不過沒有講這兩個的重要差異就是
    * 注意 s7 跟 s8 的差異，一個是 direct，一個是創造一個 temporary object 然後用 copy initialization 宣告 s8
    * 但是這種寫法一點好處也沒有，簡稱糞 code

### 3.2.2 Operations on strings
* reference 是你的好朋友
* Reading an Unknown Number of strings
    * 這裡有個經點寫法是拿來讀取未知數量的字串的：
    ```C++
    int main() {
      string word;
      while (cin >> word) {   // read until end-of-file
        cout << word << endl; // write each word followed by a new line
      }
      return 0;
    }
    ```
* Using getline to Read an Entire Line
    * The getline function takes an input stream and a string.
    *  This function reads the given stream up to and including the first newline and stores what it read—**not including the newline**—in its string argument.
    * Like the input operator, getline returns its istream argument.
        * 所以如果你要讀取未知行數的字串的話可以寫成這樣：
        ```C++
        int main() {
          string line;
          while (getline(istream, line)) {   // read until end-of-file
            cout << line << endl; // write each word followed by a new line
          }
          return 0;
        }
        ```
        記得，換行字元不會被塞到 string 裡，如果要換行要自己印
* empty, size 之類的
    * 介紹個 size_type：
    * The string::size_type Type
    * The string class—and most other library types—defines several **companion types.**
    * **These companion types make it possible to use the library types in a machine-independent manner.**
    * it is an unsigned type
        * big enough to hold the size of any string.
    * string.size() return 的就是 size_type
    * 所以你要接 return value 的 object 型態應該要是 string::size_type
    * 有夠冗，你可以用 auto len = string_obj.size();
    * 或者有時候 initializer 的 type 不是你要的 type，那你就可以用 decltype():
        * decltype(s.size()) count = 0;
    * 等等，string::size_type，所以可以在 class 內定義型別？
        * 可以!可以在 class 的 public 區域用 using 或者 typedef
    * 另一個重點，**既然他是 unsigned，你就要記得之前說的，mix signed and unsigned data can have surprising results**：
        ```C++
        int n = -10;
        string s = "9487";
        s.size() < n; // true!!!!
        ```
    * 不然你也可以很ㄎㄧㄤ用 int 去接 size() 的 return value 啦...

* Add two strings
    * trivial
* Adding Literals and strings
```C++
string s1 = "hello", s2 = "world"; // no punctuation in s1 or s2 
string s3 = s1 + ", " + s2 + ’\n’;
```
* As we saw in § 2.1.2 (p. 35), we can use one type where another type is expected if there is a conversion from the given type to the expected type.
    * 所以上面的各種 literal 都會先被轉成 string 再用 adding two strings 的方式來處理
* When we mix strings and string or character literals, at least one operand to each + operator must be of string type:
    * operator+ 至少要有一邊是 string，這樣另一邊才會被轉成 string
        ```C++
        int main() {
          string s1 = "www";
          string s2 = "wtf " + s1;
          string s4 = s1 + ", ";           // ok: adding a stringand a literal
          string s5 = "hello" + ", ";      // error: no string operand
          string s6 = s1 + ", " + "world"; // ok: each + has a string operand
          string s7 = "hello" + ", " + s2; // error: can’t add string literals
        }
        ```
    * 注意上面的 s6，第二個 operator+ 看起來貌似沒有符合，但是加法是 left associative，會先算完第一個 +，然後 value 是 string，這個 value 再當作第二個 + 的 operand，所以第二個 + 還是有一個 operand 是 string

### 3.2.3 Dealing with the Characters in a string
* Advice: 用 c*name* 來 include C header，這種 header 定義的 name 會在 std 裡面，可是 C 那種 name.h 的 header 定義的 name 就沒有在 std 內。
* 這裡還突然殺出來介紹 range for
    * 有沒有 reference 差很多

## 3.3 Library vector Type
* 基礎自己看
* vector 是 class template，不是 class
* template 可以想像成是給 compiler 的一串指令，一種食譜(?)，compiler 會看著這串指令以及你提供的額外資訊(additional information，也就是角括號<>內的東西)來 **generate** 程式碼
* C++11 List initialization
* 一個以前沒注意到的：
    * If our vector holds objects of a type that we cannot default initialize, then we must supply an initial element value; it is not possible to create vectorsof such types by supplying only a size.
    * 如果你提供的 type 不能 default initial ，一定要提供一個 initializer，那塞這種 type 的 vector 就一定要給第二個參數當作 initializer；這種 error 在 syntax level 還檢查不出來w
* List initialization 還有個ㄎㄧㄤ點：
    ```C++
    vector<string> v1{"yoo"}; //list initialized
    vector<string> v2("yoo"); //error
    vector<string> v3{10};  // construct vector of ten default empty string!!!
    vector<string> v4{10, "hi"}; // construct vector of ten "hi" string!!!
    ```
    * In order to list initialize the vector, the values inside braces must match the element type. 
    * 如果 list 裡面提供的 initializer 不能拿來做 list initialization，那 compiler 會嘗試使用括號的方式來解讀
    * 別寫這種糞 code...

* Practice: 不要先創一個某個 size 的 vector 再去改 element 的值，除非 element 全部一樣，不然效率很糟，C++ 標準有要求 vector 的實作效能，run time 的時候再 push_back 就行了。
    * 第九章還會講到我們可以用 vector 提供的一些功能來優化增加 element 時的效率(!)
* 如果你的 for loop body 會增加 element，你就不能用 range for!
    * **The body of a range for must not change the size of the sequence over which it is iterating.**

### 3.3.3 Other vector Operations
很多都跟 string 有 87% 像，但是有幾點要注意：
1. vector 是 container，有很多 operation 其實是 depends on element type 的，例如 relational operator，如果 element 不支援 relational operator，那 vector<T> 就不能比較

2. 而且更機掰的是，這是編譯時期才能決定的朋友呢
    ```C++
    class no_default_constructor_class {
      int integer;

    public:
      no_default_constructor_class() : integer(0) {}
      no_default_constructor_class(int para) : integer(para) {}
    };

    int main() {
      vector<no_default_constructor_class> vector_1;
      vector<no_default_constructor_class> vector_2(
          10, no_default_constructor_class(100));
      vector<no_default_constructor_class> vector_3(10);
      cout << (vector_3 == vector_2);
      //上面這行是個編譯時期才會噴一堆噁心東西的朋友呢
      // vector<T> 要能比較 iff T 可以比較
    }
    ```
* vector 的 assignment operator 還支援 list_initializer(string 也可以，應該 container 都支援):
    ```C++
    int main() {
      vector<int> intv{1, 2, 3, 4, 5};
      intv = {6, 7, 8, 9, 10};
      string str = {'1', '2', '3', '4', '5'};
      str = {'6', '7', '8', '9'};
    }
    ```
    * intv = {6, 7, 8, 9, 10};
    * str = {'6', '7', '8', '9'};

## 3.4 Introducing Iterators
* More general way to access a container's element
* All of the library container support iterator
    * But only some of them support subscript operator
### 3.4.1 Using Iterators
* What it does:
    * Like pointer, it gives as indirect access to an object
        * element in a container
* 題外話，string 不是 container，但是它支援大部分的 container 操作，包括 iterator
* 那些有 begin 跟 end 的 type 就可以使用 iterator
    ```C++
    auto it = v.begin(); // v is vector<T>
    ```
    * 在 C++11 之後請用 auto 宣告 iterator
        * 還記得沒用 auto 有多痛苦ㄇ? stable_vector
    
* 如果 container 為 empty 則 v.begin() == v.end() // both point to one pass the end of the associated container
* 跟 pointer 一樣，用 dereference operator 取 object
    * 所以要注意 iterator 是否為 valid
    * 給個確認是否為 valid 的例子：
    ```C++
    string s("some string");
    if (s.begin() != s.end()) { // make sure s is not empty
        auto it = s.begin(); // itdenotes the first character in s 
        *it = toupper(*it); // make that character uppercase
    }
    ```
    * condition 就是在處理 s 為 empty string 的情況
#### Moving Iterators from One Element to Another
* ++
* 題外話，如果 it == v.end()，it 是不能被 dereference 的，會 UB
* 如果是要用 iterator 遍歷一個 container，請用 != 不要用 <：
    * for (auto it = v.begin(); it != v.end(); ++it);
    * **GENERIC PROGRAMMING**
        * 因為所有 container 都支援 != 跟 ==，可是只有部分支援 <=，用 != 做到一樣效果，而且不用考慮 container 是否支援，換句話說對哪種 container 都一樣，generic!
#### The begin and end Operations
* return type depends on whether container is const
    * const_iterator
* 但常常這不是我們要的行為，我們在遍歷 container 時如果只要讀沒要寫，我們希望用 const_iterator 去指它
    * **C++11，cbegin and cend**
        * Regardless of whether the vector (or string)is const,they return a const_iterator.
#### **Some vector Operations Invalidate Iterators**
* 還記得 stable_vector 嗎! 就是在處理這件事!
*  Any operation, such as push_back, **that changes the size of a vector** potentially invalidates all iterators into that vector.
*  到底怎麼變成 invalid 在第九章會講
### 3.4.2 Iterator Arithmetic
* 所有 container 都支援 ++ 跟 --，
* 少數如 vector 跟 string 支援一次 + 或 - n
* 少數如 vector 跟 string 支援 relational operators
* iter 相減，
    * difference_type


## 3.5 Arrays
* Like pointer and reference，it's compound type
### 3.5.1 Defining and Initializing Built-in Arrays
* array is an object!
    * So there are pointer or reference to arrays
    ```C++
    int main() {
      int arr[10];
      int *ptrs[10];           // ptrs is an array of ten pointers to int
      int &refs[10] = /* ? */; // error: no arrays ofreferences
      int(*Parray)[10] = &arr; // Parraypoints to an array often ints
      int(&arrRef)[10] = arr;  // arrRefrefers to an array often ints
    }
    ```
    * 看最後兩個 pointer/reference to array 的例子，宣告的時候括號不可少，就跟 function pointer 一樣靠北
    * By default, type modifiers bind right to left
### 3.5.2 Accessing the Elements of an Array
* array 一樣支援 range for
    * As in the case of string or vector,it is best to use a range for when we want to traverse the entire array.
    * Because the **dimension is part of each array type**, the system knows how many elements are in scores.
* index 的 type 是 size_t
    *  size_t is a machine-specific unsigned type that is guaranteed to be large enough to hold the size of any object in memory
### 3.5.3 Pointers and Arrays
* 陣列跟 pointer 的淵源ㄏㄏ
* **When we use an array, the compiler ordinarily converts the array to a pointer.**
    ```C++
    int main() {
      int a[100] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
      int i = 10;
      cout << a[i] << endl;
      cout << i[a] << endl;
      //互相傷害的噁心 array indexing
    }
    ```
    ```C++
    * string *p = &nums[0]; // ppoints to the first element in nums
    * string *p2 = nums; // equivalent to p2=&nums[0]
    ```
    * in *most* places whenwe use an array, the compiler automatically substitutes a pointer to the first element:
* There are various implications of the fact that operations on arrays are often really operations on pointers.
    * 例如 auto，如果你的 initializer 是 array，型別會推斷成 pointer to element
        ```C++
        int ia[10] = {};
        auto ia2(ia); // ia2 is int* that points to first element in ia
        auto ia2(&ia[0]); // equivalent to above; now it’s clear that ia2has type int*
        ```
    * 不過如果是用 decltype，就還是會維持 array type
        ```C++
        int main() {
          int ia[10] = {};
          int *p = ia;
          int i = 0;
          decltype(ia) ia3 = {0, 1, 2, 3, 4,
                              5, 6, 7, 8, 9}; // ia3 is an array often ints
          ia3 = p;    // error: can’t assign an int*to an array
          ia3[4] = i; // ok: assigns the value ofito an element in ia3
        }
        ```
#### Pointers Are Iterators
* 因為 pointer 都支援當初 iterator 定義的那幾個 operations，所以他是 iterator
* C++11 的 std::begin() 跟 std::end()
    * 你自己去算 array 的頭尾真的太ㄎㄧㄤ，很容易錯(請看 Primer P.118)，所以新標準就給了這兩個 function，好用
    ```C++
    int main() {
      int ia[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}; // ia is an array of ten ints
      int *beg = begin(ia); // pointer to the first element in ia
      int *last = end(ia);  // pointer one past the last element in ia
    }
    ```
#### Pointer Arithmetic
* iterator 支援的基本上都支援
* ptr 可以指向一個 object，或者 object 的下一個位置，其他都是 UB
* ptr 相減的 type 為 ptrdiff_t

### 3.5.4 C-Style Character Strings
#### **Caller Is Responsible for Size of a Destination String**

### 3.5.5 Interfacing to Older Code
* 跟 legacy code 的愛恨鳩葛
* string 可用 C string 做初始化
* string 可 + C string;
    * string1 = string1 + Cstring;
    * 但不能兩個 operator 都是 C string
    * string1 += Cstring;
    * 右邊的 operand 可以是 C string
    * 但是反過來，string 要轉回 C string 還是很麻煩
        * string::c_str 可以用
        * return const char*
        * 但是這個 const char* 在你更改 string 的內容之後就要假設為失效，跟 vector 的 iterator 很像
        * 比較好的做法是不要對 string::c_str 回傳的東西直接做 access
        * 先把他用 strncpy copy 到一個 temp C string，然後對這個 temp 做運算
        * If a program needs continuing access to the contents of the array returned by str(), the programmust copy the array returned by c_st
#### Using an Array to Initialize a vector
* 用 begin() 跟 end() 把頭尾傳進去
    * vector<int> v1(begin(int_arr), end(int_arr));
* 也可以只傳部分的 array
    * vector<int> v1(int_arr+87, int_arr+94);

#### Note
Modern C++ programs should use vectors and iterators instead of built-in arrays and pointers, and use strings rather than C-style array-based character strings.

### 3.6 Multidimensional Arrays
* 嚴格來說 C++ 沒有這種東西
* array of arrays
#### Using a Range for with Multidimensional Arrays
* Let the system manage the indices for us.
* 如果要改多維陣列的值那當然要用 reference 宣告
* 但如果是只要讀呢?
* 除了最內層的 loop 其他也必須要用 reference 宣告
    * why?
    ```C++
    for (auto row : ia)
        for (auto col : row)
            ;
    ```
    * Because row is not a reference, **when the compiler initializes row it will convert each array element (like any other object of array type) to a pointer to that array’s first element.**
    * 因為如果你外層迴圈不是用 reference，那內部的迴圈會把 row 轉成 pointer to first element(就是普通的把陣列 decay 成指標的轉法)，那 col 就會 loops on 一個 type 為指標的 object，他不是 sequence，所以會噴 error。
#### Pointers and Multidimensional Arrays
* 管你幾維陣列，就跟一維的一樣，大部分的時候 array 都會被轉成 pointer to first element
* 只是這裡 first element 的 type 一樣是 array，所以這個 pointer 的 type 就是 pointer to first inner array
    ```C++
    int main() {
      int ia[3][4]; // array of size 3; each element is an array of ints of size 4
      int(*p)[4] = ia; // ppoints to an array off our ints
      p = &ia[2];      // pnow points to the last element in ia
    }
    ```
    * 一樣，p 宣告時的括號是必須的
* C++11 請用 auto 或 decltype
```C++
int main() {
  int ia[3][4] = {};
  for (auto p = ia; p != ia + 3; ++p) { // q points to the first element of an
                                        // array of four ints; that is, q points
                                        // to an int
    for (auto q = *p; q != *p + 4; ++q)
      cout << *q << ' ';
    cout << endl;
  }
}
```
* 看起來還是很麻煩，可以用 C++11 的 begin() 跟 end()
```C++

int main() {
  int ia[3][4] = {};
  // p points to the first array in ia
  for (auto p = begin(ia); p != end(ia);
       ++p) { // q points to the first element in an inner array
    for (auto q = begin(*p); q != end(*p); ++q)
      cout << *q << ' '; // prints the intvalue to which q points
    cout << endl;
  }
}
```
* 這樣寫還真的比較清楚，因為你就不用去想到底什麼時候 array name 會被轉成 pointer to first element 了，你知道 begin() 跟 end() 一定傳 pointer to element 給你，讚讚
#### Type Aliases Simplify Pointers to Multidimensional Arrays
```C++
using int_array = int[4]; // new style type alias declaration; see § 2.5.1 (p. 68) 
typedef int int_array[4]; // equivalent typedefdeclaration; § 2.5.1 (p. 67)
// print the value of each element in ia, with each inner array on its own line 
for (int_array *p = ia; p != ia + 3; ++p) { 
    for (int *q= *p; q != *p + 4; ++q) 
        cout << *q << '';
    cout << endl;
}
```
* typedef 看起來就有夠噁