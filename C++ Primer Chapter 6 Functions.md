---
tags: C++
---

# C++ Primer Chapter 6 Functions
* 一樣，有 define 跟 declare 之分
* A function is a **block of code with a name.**
    * LOL
* take arguments and yield a result
* may be overloaded
    * same name may refer to different functions

## 6.1 Function Basics
* execute functions through **call operator**
    * ()
    * The call operator **takes an expression that is a function or points to a function.**
        * :O 所以是指，給它 function name 或者 pointer to function 都可以? 簡單來說就是 pointer to function 不用 dereference?
    * The type of call expression is the return type of the function

#### Parameters and Arguments
* C/C++ 千年老題目
* Arguments are the initializers for a function’s parameters.
    * 簡短明瞭解釋這兩個的差別，雖然還是很機掰
* 我們雖然知道哪個 argument 是哪個 parameter 的 initializer，但**我們一樣不知道哪個 initializer 會先被算**，預算順序未定義。
* 數量不多不少剛剛好
    * intializer 要跟 parameter 同 type 或者可以被轉成 parameter 的 type

* 題外話，下面的憤 code legal
    ```cpp
    int foo(int, int) // legal.......
    {
      return 87;
    }

    int main() { cout << foo(94, 87) << endl; }
    ```
    * 你沒辦法用 unnamed parameters，可是你 call function 時還是要照傳
    * 這有可能是為了符合某種(legacy) API 規範
#### Function Return Type
* 幾乎什麼 type 都可以傳，除了 array 跟 function
    * 可是可以傳 pointerce/referen to array/function



### 6.1.1 Local Objects

* In C++, names have scope (§ 2.2.4, p. 48), and objects have **lifetimes.**
* The scope of a name is the part of the program’s text in which that name is visible.
* The lifetime of an object is the time during the program’s execution that the object exists
    * 新手殺
* Parameters and variables defined inside a function body are referred to as **local variables.**
    * "local" to that function and **hide** declarations of the same name made in an outer scope
* Objects defined outside any function exist throughout the program’s execution.
    * Such objects are created when the program starts and are not destroyed until the program ends.

#### Automatic Objects
* Objects that exist only while a block is executing
* After execution exits a block, the values of the automatic objects created in that block are undefined.
    * Parameters 也是 automatic objects
#### Local static Objects
* local variable whose **lifetime continues *across* calls to the function**
* define local variables as static
* Each local static object is initialized before the first time execution passes through the object’s definition.
    * 反正就是在 program memory model 內，static object 會被跟 global 放在同一區塊
* destroyed when the program terminates
* static 如果沒給 initializer 就會 value initialize(built-in 給 0)

### 6.1.2 Function Declarations
* The name of a function must be declared before we can use it.
* 只可定義一次，但可宣告多次
* 因為沒有 body，所以 parameter 可以只給 type
* 你其實只要知道 function 的 interface(i.e. return type, function name, parameter list) 就知道怎麼 call function 了。
* function declaration 又叫做 function prototype
#### Function Declarations Go in Header Files
* Recall that variables are declared in header files (§ 2.6.3, p. 76) and defined in source files.
* For the same reasons, **functions should be declared in header files and defined in source files.**
* 雖然自行宣告 prototype 完全合法，但如果你每個會用到某個 function 的 source file 都自行宣告一次，這樣很容易出錯
* 所以請把宣告放在 header，只宣告一次在 header，然後所有會用到 function 的 source file 去 include header，這樣還會避免掉不同 source 各自宣告然後宣告錯的問題
* The source file that defines a function should include the header that contains that function’s declaration.
    * 真正定義(define)那個 function 的 source 也要 include 有那個 function 的 declaration 的 header
    * That way the compiler will **verify** that the definition and declaration are consistent.
    * 其實不宣告也可以 link(ry
    * 但就是多了這層檢查

### 6.1.3 Separate Compilation
* 怎麼突然就在講這麼重要的東西...
* best practice 就是用一個 header 宣告會用到的 function，其他 source 去 include 他
* To produce an executable file, **we must tell the compiler where to find all of the code we use.**
    * 注意，是產生「executable」


## 6.2 Argument Passing
* If the parameter is a reference (§ 2.3.1, p. 50), then the parameter is bound to its argument. Otherwise, the argument’s value is copied
* when parameter is a reference, we say that its **corresponding *argument* is “passed by reference”** or that **the *function* is “called by reference.”**
* 如果 parameter 不是 reference，那對應的 ***argument* 就是 passed by value**，或者說 ***function* 是 called by value**

### 6.2.1 Passing Arguments by Value
* **Changes made to the variable have no effect on the initializer:**
#### Pointer Parameters
* 一樣是 called by value!
*  After the copy, **the two pointers are distinct.** However, **a pointer also gives us indirect access to the object to which that pointer points.**

* Best Practice:
    * 在 C，你要改變其他 function 的 local variable 只能用傳指標的方式來達成，在 C++ 請用 reference。

### 6.2.2 Passing Arguments by Reference
* They are often used to allow a function to change the value of one or more of its arguments.
#### Using References to Avoid Copies
* copy 效率差，**有些 class 甚至不能 copy**
    * 有些 class 可以定義成不能被 copy，詳情 13~15 章

* 之後會提到，如果你的 function 沒有要更改 argument，然後你又要用 reference，你應該宣告成 reference to const

#### Using Reference Parameters to Return Additional Information

### 6.2.3 const Parameters and Arguments
* Just as in any other initialization, when we copy an argument to initialize a parameter, top-level consts are ignored
* As a result, **top-level const on parameters are ignored.**
    ```cpp
    void fcn(const int i) { /* fcn can read but not write to i */}
    ```
    * can pass either const int or int to fcn
* 這件事情還會牽扯到之後會講的 function overloading
    ```cpp
    void fcn(const int i) { /* fcn can read but not write to i */}
    void fcn(int i) { /* .. . */} // error: redefines fcn(int)
    ```
    * 上面的 overload 是錯的，compiler 沒辦法知道當你 call fcn 並給它特定參數時，到底該呼叫哪個 function。
    * In C++, we can define several different functions that have the same name. However, we can do so only if their **parameter lists are sufficiently different.**
    * 因為 pass argument 的時候 parameter 的 top-level const 會被無視，這樣根本就不知道該呼叫哪個版本的 function
#### Pointer or Reference Parameters and const
* 記得 reference 跟 pointer 都不能指向或綁定不同的 base type
    * 其實也是很煩(?
    * 至於他們怎麼處理 top-level 跟 low-level const 之前講過惹，top 會無視，low 不會
        * 詳細看 Primer p.213 的例子
* 假設你 interface 長這樣: void func(int&);
    * 你只能 pass int 進去
    * We **cannot pass a literal, an expression that evaluates to an int, an object that requires conversion, or a const int object.**
    * 有沒有很煩...
* 如果你改成這樣: void func(const int&);
    * 那你就可以傳 const int 或者 literal 進去，只是不能改值；你也可以傳其他原本可以轉成 int 的 type，它會綁定一個 converted temp object。
        * 反正 parameter 都是宣告成 const 了，這本來就代表 function 不會去更改傳進去的 argument，所以 parameter 去 bind 一個 temp obj 也沒差ㄅ
#### Use Reference to const When Possible
* It is a somewhat common mistake to define parameters that a function does not change as (plain) references.
    * 這樣會讓 caller 以為 function 可能會改變 argument
* Moreover, using a reference instead of a reference to const **unduly limits the type of arguments that can be used with the function.**
    * 就上面講的，你只能傳 base type 一樣的，並且不能是 const，其他都不行
    * As we’ve just seen, we cannot pass a const object, or a literal, or an object that requires conversion to a plain reference parameter.
    * 不過之後會講到 T&& 這種 C++11 新定義的東西，行為又不一樣惹
    * 這裡可以把 Primer P.214 再好好看一次，跟你說如果你錯把應該寫成 const T& 的 parameter 寫成 T&，會有什麼後果
        * 不能傳需要轉型的 object
        * const T& 也不能傳

* Primer 還 demo 了一個不夠了解問題而做的 workaround，結果導致更慘的 code
    ```cpp
    bool is_sentence(const string &s) {
    // if there’s a single period at the end of s,then s is a sentence
    string::size_type ctr = 0;
    return find_char(s, ’.’, ctr) == s.size() - 1 && ctr == 1;
    }
    ```
    * 如果你 find_char 第一個 para 宣告成 plain ref，那這段 code 會噴 compile error
    * 然後你如果不夠了解問題所在，你可能會想去改 is_sentence 的 parameter type，把它改成 plain ref
        * **But that fix only propagates the error—callers of is_sentence could pass only nonconst strings.**
* 總之結論就是 ref parameter 沒有要更改 argument 的時候要宣告成 reference to const
* 但很有可能這個 API 爛掉了，而且不能動...
    * 慘
    * 這時候你可以寫成
    * string temp_str = s;
    * 然後把 temp_str 傳給 find_char


### 6.2.4 Array Parameters
* Arrays have two special properties that affect how we define and use functions that operate on arrays:
* 之前講過 array 跟 function 不能被 copy，換句話說 function 沒辦法直接宣告 array 跟 function 當作 parameter
    * 但是可以宣告 pointer to function/array 當 parameter

* 還有之前講過的 array 常常被 decay 成 pointer to first element，把 array 傳進 function 時也是如此
* 雖然不能直接宣告 array 當參數，可是可以宣告很像 array 的參數(ry
    ```cpp
    void print(const int*);
    void print(const int[]); // shows the intent that the function takes an array
    void print(const int[10]); // dimension for documentation purposes (at best)
    ```
    * 以上三個參數的型態都一樣，不過第二個跟第三個有 comment 的功能，第二個說我要收一個 array，第三個還說我要收一個 size 為 10 的 array；不過他們的 type 都是 pointer to (const) int。
        * 注意第三個的 10 不是 type，僅僅只是提示作用
* When the compiler checks a call to print, it checks only that the argument has type const int*(或者可以被轉成 const int*):

    ```cpp
    int i = 0, j[2] = {0, 1}; print(&i); // ok: &i is int*
    print(j); // ok: j is converted to an int* that points to j[0]
    ```
* If we pass an array to print, that argument is automatically converted to a pointer to the first element in the array; **the size of the array is irrelevant.**
    * Because arrays are passed as pointers, functions ordinarily don’t know the size of the array they are given.
    * size 不重要的意涵就是，callee 光看這個 const int* 參數是不知道原始 array 的 size 的，你必須要用其他方是跟 callee 講說 array 的 size(或者說 array 的邊界)
* 有三個方法
    1. Using a Marker to Specify the Extent of an Array
        * 類似 C string 放 null character 當 marker 的概念
        * 不過這個方法就要保證說，你的 array 內放的 data 不會用到這個 "marker"
            * 例如你是傳 int array 的話就有點怪，因為 any int value 本質上應該都是合法的
    2. Using the Standard Library Conventions
        * A second technique used to manage array arguments is to pass pointers to the first and one past the last element in the array.
        * 也就是把 array 的起始跟結束位置都當作 argument 傳進 function
        * standard lib 常常這樣搞，有點像是 begin() end() 那樣
        * interface: void func(T*, T*);
        * caller: func(begin(arr), end(arr));
        * 雖然可以用 begin end，不過有時候可能只要存取 subarray，這時候 API 定成這樣，計算 array 邊界的責任就落到 caller 身上了
        * https://github.com/ericniebler/range-v3
            * 這個ㄎㄧㄤ物可以看
    3. Explicitly Passing a Size Parameter
        * Common in C and older C++... QQ
        * interface: void func(T*, size_t size);
        * caller: func(arr, arr_len);
        * alternative caller: func(arr, end(arr)-begin(arr));
#### Array Parameters and const
* 如果 function 沒有要更改 array 內容，那你的 pointer to element 也要宣告成 const，這理由之前講過了。
#### Array Reference Parameters
* 你宣告 reference 時的 type 可以是 ref to array，parameter 當然也可以宣告成 reference to array
    ```cpp
    // ok: parameter is a reference to an array; the dimension is part of the type
    void print(int (&arr)[10]) {
    for (auto elem : arr)
        cout << elem << endl;
    }
    ```
    * **the dimension is part ofthe type**
    * 這跟上面 pointer 的 syntax suger 裡面 [] 裡面的數字不一樣了，syntax sugar 的數字只是用來 document，這裡的數字就跟宣告 reference to array 一樣，是型別的一部份
    * 但是這也意味著這個 function 只能收 array of size 10 的陣列來 bind，其他 size 都不可以，很鳥
    * 16 章會有一種新方法可以 pass a reference parameter to an array of any size.
        * template 怕爆
#### Passing a Multidimensional Array
* 噁...
* 首先記得 C++ 沒有多維陣列
* array of arrays.
* As with any array, a multidimensional array is passed as a pointer to its first element (§ 3.6, p. 128).
* Becausewe are dealing with an array of arrays, that element is an array, **so the pointer is a pointer to an array.**
* **The size of the second (and any subsequent) dimension is part of the element type and must be specified:**
    ```cpp
    // matrixpoints to the first element in an array whose elements are arrays often ints
    void print(int (*matrix)[10], int rowSize) { /* ... */}
    ```
    * pointer to the array of 10 ints
    * 第二個參數就給(外圍的)array 的長度
* 等價的 syntax:
    ```cpp
    void print(int matrix[][10], int rowSize) { /* .. . */}
    ```

### 6.2.5 main: Handling Command-Line Options
* It turns out that `main` is a good example of how C++ programs pass arrays to functions.
* command-line options are passed to main in two (optional) parameters:
    ```cpp
    int main(int argc, char *argv[]) { ... }
    ```
    * argv is an **array** of **pointers**
* alternative
    ```cpp
    int main(int argc, char **argv) { ... }
    ```

### 6.2.6 Functions with Varying Parameters
* intializer_list
* 16章的鬼東西，variadic template
* parameter type, ellipsis, 跟 C 接的
* initializer_list 可以吃任意數量，單個 type 的參數
    * 他94ㄍ template
    * ![](https://i.imgur.com/TEMgEra.png)
    * the elements in an initializer_list are always **const values; there is no way to change the value of an element in an initializer_list.**
* 實際上用起來像這樣:
    ```cpp
    void error_msg(initializer_list<string> il) {
    for (auto beg = il.begin(); beg != il.end(); ++beg)
        cout << *beg << " " ;
    cout << endl;
    }
    ```
    * 有 begin end 其實也可以用 range for
* caller 要 call function 時長這樣:
    ```cpp
    // expected, actualare strings
    if (expected != actual)
        error_msg({"functionX", expected, actual});
    else
        error_msg({"functionX", "okay"});
    ```
    * When we pass a sequence of values to an initializer_list parameter, **we must enclose the sequence in curly braces**

#### Ellipsis Parameters
* Primer 歸類為選讀章節www
* 幹不對，Primer 根本沒有講細節wwwwww
* Ellipsis parameters should be **used only for types that are common to both C and C++.** In particular, **objects of most class types are not copied properly when passed to an ellipsis parameter.**


## 6.3 Return Types and the return Statement

### 6.3.1 Functions with No Return Value
* void
* 其實 return void 的 function 還是可以寫 expression，不過那個 expression 也要是 return void，例如另一個也是 return void 的 function

### 6.3.2 Functions That Return a Value
* return 的 exp 的 type 要跟 function prototype 給的 return type 一樣或者可以轉成 function prototype 的 return type
* 如果一個 function 會 return value，且如果在會 return 的 loop 後面到 function 尾端為止都沒有 return，compiler (可能)會報錯，但這樣是 UB
    * clang 會幫你塞 illegal instruction 喔 >.^

#### How Values Are Returned
* Values are returned in **exactly the same way as variables and parameters are initialized:**
* The return value is used to **initialize a temporary *at the call site*,** and that temporary is the result of the function call.
* 跳很遠來講，不要 **return reference to automatic variable(local object)**
#### **Never Return a Reference or Pointer to a Local Object**
* 上面講惹
* When a function completes, its storage is freed (§ 6.1.1, p. 204).
* After a function terminates, references to local objects refer to memory that **is no longer valid:**

#### Functions That Return Class Types and the Call Operator
* Like any operator the call operator has associativity and precedence
    * The call operator has the same precedence as the dot and arrow operators
    * left associative
    * As a result, if a function returns a pointer, reference or object of class type, **we can use the result of a call to call a member of the resulting object.**

    ```cpp
    // call the sizemember ofthe string returned by shorterString
    auto sz = shorterString(s1, s2).size();
    ```

    * 直接用 shorterString 回傳的 object 用 member selector 選擇 size
    * shorterString(s1, s2)
    * shorterString(s1, s2).size
    * shorterString(s1, s2).size();

#### **Reference Returns Are Lvalues**
* Whether a function call is an lvalue (§ 4.1.1, p. 135) depends on the return type of the function.
    * return reference 就是 lvalue，其他都是 rvalue
* 所以如果是 return reference，就可以把 function call 放在 assignment 左邊
    ```cpp
    char &get_val(string &str, string::size_type ix) {
        return str[ix]; // get_valassumes the given index is valid
    }
    int main() {
        string s("a value");
        cout << s << endl; // prints a value
        get_val(s, 0) = 'A'; // changes s[0]to
        A cout << s << endl; // prints A value return 0;
    }
    ```
    * 看上面那行 get_val(s, 0) = 'A';
        * 看起來很ㄎㄧㄤ，可是真的可以這樣寫
#### List Initializing the Return Value
* C++11 的...(我還)不知道可以拿來幹嘛的功能
    ```cpp
    vector<string> process() {
    // ... // expected and actual are strings
        if (expected.empty())
            return {}; // return an empty vector
        else if (expected == actual)
            return {"functionX", "okay"}; // return list-initialized vector
        else
            return {"functionX", expected, actual};
    }
    ```
    * 反正 return value 就跟宣告變數很像，變數可以被怎麼宣告，return value 就可以怎麼給值；宣告可以給 initializer_list，return value 也可以
        * 嚴格上來說是因為 vector 的 ctor 有一種是可以吃 initializer_list 的，所以才合法
    * 如果 return value 是 built-in type，你就是放一個 element 數量為 1 的 initializer_list，總之就跟變數宣告一樣啦
        * return {1};

#### Return from main
* 唯一一個 return non void 可是可以不用寫 return 的 function
* 如果直接寫 return *numeric value*，這樣的 code 是 machine dependent
    * 可以 include cstdlib 用定義好的 macro
    * e.g. return EXIT_FAILURE; return EXIT_SUCCRSS;

#### Recursion
* a function that calls itself directly or indirectly
```cpp
// calculate val!,which is 1*2*3.. . *val
int factorial(int val) {
    if (val > 1)
        return factorial(val-1) * val;
    return 1;
}
```
* There must always be a path through a recursive function that does not involve a recursive call; otherwise, the function will recurse “forever,” meaning that the function will continue to call itself until the program stack is exhausted.
    * - recursion loop.

### 6.3.3 Returning a Pointer to an Array
* 之前講過雖然 function 不能收 array 當參數，卻可以收 pointer/reference to array
* return 也是一樣的規則
    * 不過因為語法上的限制，function 要宣告 return pointer/reference to array 會變得很醜(C姊姊
    * 可以用之前學的 typedef 或 using 解決
    ```cpp
    typedef int arrT[10]; // arrTis a synonym for the type array often ints
    using arrT = int[10]; // equivalent declaration ofarrT; see § 2.5.1 (p. 68)
    arrT* func(int i); // func returns a pointer to an array often ints
    ```
* 如果不用 type alias 呢?
* 首先我們要先記得 array 的 type 是跟在 identifier 之後的
    ```cpp
    int arr[10]; // arr is an array often ints
    int *p1[10]; // p1 is an array of ten pointers
    int (*p2)[10] = &arr; // p2 points to an array often ints
    ```
* 照著這個原則...As with these declarations, if we want to define a function that returns a pointer to an array, **the dimension must follow the function’s name.**
* Type (*function(parameter_list))[dimension]
    * 超ㄎㄧㄤ
    * 注意上面的 function 旁邊有 *
    * 括號因為優先權的關係一樣是必須的

* 來點例子ㄅ..
    ```cpp
    int (*func(int i))[10];
    ```
    ![](https://i.imgur.com/PegwV0T.png)

#### Using a Trailing Return Type
* C++11 讓你不用那麼痛苦
* A trailing return type follows the parameter list and is preceded by ->
* 原本填 return type 的地方填 auto

#### Using decltype
* 你可以直接在 function return type 那邊用 decltype(obj)，然後 function name 前面加 * ，這樣 return type 就是 pointer to obj
    ```cpp
    int odd[] = {1,3,5,7,9};
    int even[] = {0,2,4,6,8};
    // returns a pointer to an array of five int elements
    decltype(odd) *arrPtr(int i) {
        return (i % 2) ? &odd : &even; // returns a pointer to the array
    }
    ```
    * The return type for arrPtr uses decltype to say that the function returns a pointer to whatever type odd has.
    * 記得 decltype 是少數不會把 array 當成 pointer 看的
        * The only tricky part is that we must remember that decltype does not automatically convert an array to its corresponding pointer type.


### 6.4 Overloaded Functions
* Functions that have the **same name** but **different parameter lists** and that appear in the **same scope** are **overloaded**.
* e.g.
    ```cpp
    void print(const char *cp);
    void print(const int *beg, const int *end);
    void print(const int ia[], size_t size);
    ```
* When we call these functions, the compiler can deduce which function we want based on the argument type we pass:
    ```cpp
    int j[2] = {0,1};
    print("Hello World"); // calls print(constchar*)
    print(j, end(j) - begin(j)); // calls print(constint*, size_t)
    print(begin(j), end(j)); // calls print(constint*, constint*)
    ```
#### Defining Overloaded Functions
* Consider a database application with several functions to find a record based on name, phone number, account number, and so on.
* Function overloading lets us *define a collection of functions, each named `lookup`*, that differ in terms of how they do the search.
    ```cpp
    Record lookup(const Account&); // find by Account
    Record lookup(const Phone&); // find by Phone
    Record lookup(const Name&); // find by Name Account acct;
    Phone phone;
    Record r1 = lookup(acct); // call version that takes an Account
    Record r2 = lookup(phone); // call version that takes a Phone
    ```
* The compiler uses the argument type(s) to figure out which function to call.
* **It is an error for two functions to *differ only in terms of their return types*.**
    * 反正 C++ 的 overload 就是不准只有 return type 不同
    * 某些語言(Perl, Haskell)就可以

#### Determining Whether Two Parameter Types Differ
* 所以怎樣才叫做不同 type?
* 傳遞參數給 function 會轉型，包括已經定義好的轉型以及 non-const 轉 const
* 先來看看 parameter list 其實相同的幾個例子:
    ```cpp
    // each pair declares the same function
    Record lookup(const Account &acct);
    Record lookup(const Account&); // parameter names are ignored

    typedef Phone Telno;
    Record lookup(const Phone&);
    Record lookup(const Telno&); // Telnoand Phone are the same type
    ```
#### Overloading and const Parameters
* As we saw in § 6.2.3 (p. 212), **top-level const (§ 2.4.3, p. 63) has no effect on the objects that can be passed to the function.**
    * 就跟初始化的時候 initailizer 的 top-level const 會被無視一樣，因為被初始化的變數以及 function parameter 都是複製一份 initializer(或 argument) 的 value，不會影響到 initializer(argument)
* A parameter that has a top-level const is indistinguishable from one without a top-level const:
    ```cpp
    Record lookup(Phone);
    Record lookup(const Phone); // redeclares Recordlookup(Phone)

    Record lookup(Phone*);
    Record lookup(Phone* const); // redeclares Recordlookup(Phone*)
    ```
    * 注意第二對例子，那是指標的 top-level const
* On the other hand, we can overload based on whether the parameter is a reference (or pointer) to the const or nonconst version of a given type; **such consts are low-level:**
    ```cpp
    // functions taking constand nonconstreferences or pointers have different parameters
    // declarations for four independent, overloaded functions
    Record lookup(Account&);  // function that takes a reference to Account
    Record lookup(const Account&); // new function, that takes a reference to const Account
    Record lookup(Account*); // new function, that takes a pointer to Account
    Record lookup(const Account*); // new function, that takes a pointer to const Account
    ```
    * Because there is no conversion (§ 4.11.2, p. 162) from const, *we can pass a const object (or a pointer to const) only to the version with a const parameter.*
        * 如果你今天要傳 reference 或 pointer to const，這個時候只有 low-level 是 const 版本的函數可以接這個參數，因為根本不存在 (implicit) const to nonconst 的轉換
    * Because there is a conversion to const, we can call either function on a nonconst object or a pointer to nonconst.
        * 如果你今天要傳 reference 或 pointer to nonconst，這時候其實 lowe-level 是或不是 const，函數都可以接，因為存在 nonconst to const 的轉換
    * However, as we’ll see in § 6.6.1 (p. 246), the compiler will prefer the nonconst versions when we pass a nonconst object or pointer to nonconst.
        * 但是之後會講到，**如果你現在傳的物件真的是 low-level nonconst，compiler 會 prefer 用 nonconst 版本的函數來接** (p. 246)

#### const_cast and Overloading
* In § 4.11.3 (p. 163) we noted that const_casts are most useful in the context of overloaded functions.
    ```cpp
    // return a reference to the shorter oftwo strings
    const string &shorterString(const string &s1, const string &s2) {
        return s1.size() <= s2.size() ? s1 : s2;
    }
    ```
    * 如果我們只有上面的 function，當我們丟 nonconst 的 string 進去的時候，function 會回傳 const 版本的，這樣其實有點麻煩，因為原本的 string 本來就不是 const
    ```cpp
    string &shorterString(string &s1, string &s2) {
        auto &r = shorterString(const_cast<const string&>(s1), const_cast<const string&>(s2));
        return const_cast<string&>(r);
    }
    ```
    * 如果我們給那個收 const string& 的 function 包一層像是上面的 wrapper，這樣這個 wrapper 回傳的就還是 nonconst string&
        * 注意，因為我們知道原本的 string 本來就不是 const，對它做 const_cast 丟到 const 版本的 function 再把回傳值的 const cast 掉是安全且合法的
        * 還有一點 Primer 沒講，這邊貌似看起來是回傳 reference to local object，其實不然，你仔細看就知道它最後會 bind 到的 object 其實就是 nonconst 當初傳進來的 argument，所以不是 bind to local object。
* ADVICE:WHEN NOT TO OVERLOAD A FUNCTION NAME
    * We should only overload operations that actually do similar things.
    * 可以看 Primer p.233 的例子，到底要不要 overload name 要看使用情境，多看 code 才比較感覺的粗乃

#### Calling an Overloaded Function
* 所以 compiler 到底怎麼決定你到底 call 哪個 overloaded function?
    * **Function matching (also known as overload resolution)** is the process by which a particular function call is associated with a specific function from a set of overloaded functions.
    * The compiler determines which function to call **by comparing the arguments** in the call **with the parameters** **offered by each function** in the overload set.
    * 其實當 overloaded functions 的參數數量不同，或者數量相同但是型態完全無關的時候要判斷是滿簡單的
    * 可是如果數量相同型態又相關的話呢?
    * Determining which function is called when the overloaded functions have the same number of parameters and those parameters arerelated by conversions (§ 4.11, p. 159) can be less obvious.
    * 6.6 才會講，這裡先講一點
* For now, what’s important to realize is that for any given call to an overloaded function, there are three possible outcomes:
    * The compiler finds exactly one function that is a **best match** for the actual arguments and generates (machine)code to call that function.
    * There is no function with parameters that match the arguments in the call, in which case the compiler issues an error message that **there was no match**.
    * There is **more than one function that matches** and **none of the matches is clearly best.** This case is also an error; it is **an ambiguous call.**

### 6.4.1 Overloading and Scope
* 首先，管它是什麼 name，只要一旦被宣告，它就會 **hide** outer scope 相同的 name
    * 管你這個 name 是宣告成 function 還是 object 還是 built-in type
    * 另外，**function 可以宣告在 function 裡面**，雖然很糞
        * 以下的 code 是 for demo purpose
    * 所以就會發生以下的情境
    ```cpp
    string read();
    void print(const string &);
    void print(double); // overloads the print function
    void fooBar(int ival) {
        bool read = false; // new scope: hides the outer declaration of read
        string s = read(); // error: read is a bool variable, not a function
        // bad practice: usually it’s a bad idea to declare functions at local scope
        void print(int); // new scope: hides previous instances of print
        print("Value: "); // error: print(const string &)is hidden
        print(ival); // ok: print(int)is visible
        print(3.14); // ok: calls print(int); print(double)is hidden
    }
    ```
    * 上面 fooBar 內 call read 會炸掉比較容易理解
    * 可是 call print(string) 會炸掉就比較ㄎㄧㄤ，因為雖然都是 function，可是 print(const string&) 跟 print(double) 都是宣告在 outer scope，而你又在 fooBar 內宣告(雖然這動作很糟糕)了 print，這個 print 就把 outer scope 定義的 print 給隱藏了。
    * 反正 compiler 看到你使用一個名字不會先管它是什麼 type，而是管它出現在哪裡，而且會先從最近的 scope 開始找，一旦找到之後就停下來了；
    * 所以這裡就是在 fooBar 這個 scope 內找到了 print 之後就停下來，不會理會 outer scope 定義的 fooBar
    * 這樣就彷彿好像只有一個 print(int) 一樣，所以你 call printf("Value: "); 就會噴 error 了
    * 其他後面幾個 call 都合法，不過 print(3.14) 一樣是用 print(int)，因為會轉型
* **Note: In C++, name lookup happens before type checking.**



## 6.5 Features for Specialized Uses
* 這個 section 會介紹 default arguments, inline and constexpr functions, and some facilities that are often used during debugging.

### 6.5.1 Default Arguments
* 有些 function 87% 的時間 argument value 其實都是給同一個，這時候就可以把這個 value 當作 parameter 的 default argument
* Functions with default arguments can be called with or without that argument.
* We may define defaults for one or more parameters. **However, if a parameter has a default argument, all the parameters that follow it must also have default arguments.**
* 來點例子
    ```cpp
    typedef string::size_type sz; // typedef see § 2.5.1 (p. 67)
    string screen(sz ht = 24, sz wid = 80, char backgrnd = ' ');
    ```
    ```cpp
    string window;
    window = screen(); // equivalent to screen(24,80,' ')
    window = screen(66); // equivalent to screen(66,80,' ')
    window = screen(66, 256); // screen(66,256,' ')
    window = screen(66, 256, '#'); // screen(66,256,'#')
    ```
    * Arguments in the call are resolved by position.
    * The default arguments are used for the trailing (right-most) arguments of a call.
    * 拿這個例子來說，如果你要給特定的值給 background，那你一定要提供前兩個 arguments
        * window = screen(, , ’?’); // error: can omit only trailing arguments
        * 沒有上面這種寫法

    * 跟 Python 解 argument 比起來就輸惹，不過也比較簡單
        * window = screen('?'); // calls screen('?',80,' ')
        * 就算你是寫這面這樣，實際上也是 screen('?', 80, ' ')
            * 字元其實是 char，所以上面的是合法的，因為可以把 char 轉成 size_type
            * 假設它是對應 ASCII 63，那就會傳 63 給 ht
* 因為 default argument 有上面的特性跟限制，你在排列 function parameter 就需要特別設計：
    * Part of the work of designing a function with default arguments is ordering the parameters so that **those least likely to use a default value appear first** and **those most likely to use a default appear last.**
        * 最常用到 default argument 的 parameter 要排 parameter list 最右邊，反之最左邊
#### Default Argument Declarations
* redeclare function 是合法的
* 但是 function 的每個 parameter 的 default value，在同一個 scope 內，只能指定一次
* 然後你要指定某個 parameter 有 default value，所有都右邊的 parameter 也都要有 default value 才行
    ```cpp
    // no default for the height or width parameters
    string screen(sz, sz, char = ' ');
    string screen(sz, sz, char = '*'); // error: redeclaration
    string screen(sz = 24, sz = 80, char); // ok: adds default arguments
    ```
    第三個例子可以就是因為，第三個參數，也就是第一跟第二個參數的所有右邊的參數，已經有 defalut value 了
* **Best Practice: Default arguments ordinarily should be specified with the function declaration in an appropriate header.**

#### Default Argument Initializers
* 你以為 default argument 只能填一個 literal 嗎? 錯! 除了 function local variable 以外它都可以填!
* 可是填 function call 或者 depends on global variable 的 function call 整個就會變成地獄...
    ```cpp
    // the declarations of wd, def, and ht must appear outside a function
    sz wd = 80;
    char def = ' ';
    sz ht();
    string screen(sz = ht(), sz = wd, char = def);
    string window = screen(); // calls screen(ht(), 80, ' ');
    ```
    * **Names used as default arguments are resolved in the scope of the function declaration.**
    * default argument initializer 裡面用到的 name 會先從 function declaration 的 scope 開始找
        * 阿一般來說 function 都是宣告在 global scope，換句話說你會用到的 name 都是 global 的... 噁
    *  The value that those names represent is **evaluated at the time of the call:**
        *  所以是 runtime 決定的!

    ```cpp
    void f2() {
    def = '*'; // changes the value of a default argument
    sz wd = 100; // hides the outer definition of wd but does not change the default
    window = screen(); // calls screen(ht(), 80, '*')
    }
    ```
    * 注意上面的 sz wd = 100;，它 hide outer scope 的 wd，可是 screen 查找的 wd 一樣不是這個 local 的 wd! 是跟 screen 宣告的時候同個 scope 或更外面的 scope 的 wd


### 6.5.2 Inline and constexpr Functions
* A function specified as inline (usually) is expanded "in line" at each call. If shorterString were defined as inline, then this call
    ```cpp
    cout << shorterString(s1, s2) << endl;
    ```
(*probably*) would be expanded during compilation into something like
    ```cpp
    cout << (s1.size() < s2.size() ? s1 : s2) << endl;
    ```
* The run-time overhead of making shorterString afunction is thus removed.
* putting the keyword ***inline*** before the function’s return type:
* Note: The inline specification is **only a request to the compiler.** The compiler may choose to *ignore* this request.

#### constexpr Functions
* C++11 ㄉ噁心東西
    * C++14 還有擴充一些功能，**並且有一些不相容C++11的功能，之後要注意**
* A constexpr function is a function that can be used in a constant expression (§ 2.4.4, p. 65).
* like any other function **but must meet certain restrictions:**
    * The return type and the type of each parameter must be a literal type (§ 2.4.4, p. 66)
    * and the function body must contain exactly one return statement:
    ```cpp
    constexpr int new_sz() { return 42; }
    constexpr int foo = new_sz(); // ok: foo is a constant expression
    ```
    * The compiler can verify—**at compile time**—that a call to new_sz returns a constant expression, so we can use new_sz to initialize our constexpr variable, foo
    * 如果可以的話，compiler 會在 compile time 就把 constexpr 都算完，把用到 constexpr 的地方全部替換成 value
    * 這種替換的動作其實跟 inline 有 87% 像 -> **constexpr functions are implicitly inline.**
    * 只要在 runtime 的時候都不會有動作或改變的 statement 都可以出現在 constexpr function 裡面
        * For example, a constexpr function may contain null statements, type aliases (§ 2.5.1, p. 67), and using declarations.
    * A constexpr function is permitted to return a value that is not a constant:
        * 幹..
        ```cpp
        // scale(arg)is a constant expression if arg is a constant expression
        constexpr size_t scale(size_t cnt) { return new_sz() * cnt; }
        ```
        * 如果 `cnt` 的 initializer 是 constexpr，那 `scale(cnt)` 就是 constexpr
        * 反正如果你傳了不是 constexpr 的 obj 給 constexpr function，那這 function 就變成普通的 function 惹
        ```cpp
        int arr[scale(2)]; // ok: scale(2)is a constant expression
        int i = 2;
        // i is not a constant expression
        int a2[scale(i)]; // error: scale(i)is not a constant expression
        ```

#### Put inline and constexpr Functions in Header Files
* 你想想嘛，compiler 看到這兩種 function 需要把 function 展開，所以他只有宣告是不夠的，它需要 definition；**所以 C++ 允許這兩種 function 被定義多次，可是都要長的一模一樣**
* 所以你還是把他們兩個都定義在 header file 吧...
* 看清楚是 ***定義*** 在header，不是*宣告*在header

### 6.5.3 Aids for Debugging
* The idea is that the program will contain debugging code that is executed only while the program is being developed.
* When the application is completed and ready to ship, the debugging code is turned off.
* uses two preprocessor facilities: **assert and NDEBUG.**

#### The assert Preprocessor Macro
* 度，assert 是該死的 macro
* The assert macro takes a single expression, which it uses as a condition:
    * assert(expr);
* evaluates expr and if the expression **is false (i.e., zero), then assert writes a message and terminates the program.**
* The assert macro is defined in the cassert header.
    * As we’ve seen, preprocessor names are **managed by the preprocessor not the compiler (§ 2.3.2, p. 54).**
    * 所以不用用 using，也不用加 namespace
* As with preprocessor variables, **macro names must be unique within the program.**
    * ㄇㄉ，整個 program 都不能重複ㄛ
* In practice, it is a good idea to avoid using the name assert for our own purposes even if we don’t include cassert.
    * 就算沒有要用到 library 的 assert，我們也最好不要用這個名字當 name，Why?
        * 因為很多 header 也會 include cassert，一旦 include 了，你還用這個名字就會 collision。
* The assert macro is often used to check for conditions that "cannot happen."
* 來個例子
    * For example, a program that does some manipulation of input text might know that all words it is given are *always longer than a threshold.* That program might contain a statement such as
    ```cpp
    assert(word.size() > threshold);
    ```

### The NDEBUG Preprocessor Variable
* 就這個 macro 可以把 assert 給關掉
* 如果 define 它，assert code 就不會執行
* 然後大部分的 compiler 會提供類似下面的 option 定義 preprocessor variable
    * CC -D NDEBUG main.c
* It can be useful as an aid in getting a program debugged but **should not be used to substitute for run-time logic checks or error checking that the program should do.**
    * 不要誤用 assert!
* P.242 的大便自己看zzz
    * 哭哭，我現在不覺得他是大便ㄌ

## 6.6 Function Matching
* Primer 又標記成選讀惹...
* It is not so simple when the overloaded functions have the same number of parameters and **when one or more of the parameters have types that are related by conversions.**
    ```cpp
    void f();
    void f(int);
    void f(int, int);
    void f(double, double = 3.14); // 這個跟上面的參數型態相關
    f(5.6); // calls void f(double, double)
    ```
#### Determining the Candidate and Viable Functions
* 第一步是先找哪先 function 要被考慮
    * 叫做 candidate functions
        * 反正 candidate functions 就是 compiler 經過 name lookup 後找到的第一個(批) function(s)
    * A candidate function is a function with the same name as the called function and for which a declaration is visible at the point of the call.
* 第二步就是從 candidate functions 挑出可以被用該次 caller 給的 argument(s) 呼叫的那些 functions，叫做 **viable functions**
    * The second step selects from the set of candidate functions those functions that can be called with the arguments in the given call.
    * 要成為 viable function
        * 參數數量要一樣(要考慮 default argument 的數量)
        * 參數型別也要一樣
            * 或者可以從 given call 轉換成 function call 的 parameters
    * 上面的例子，由 f(5.6) 就可以先刪掉參數數量不同的兩個 (f(); 跟 f(int, int);)
    * 剩下的兩個
        * 一個是收一個 int，argument 給的 5.6 可以轉成 int，所以是 viable
        * 另一個是收兩個 double，第一個的 double 直接 match argument 給的 5.6，第二個 parameter 有 default argument 所以可以不給，所以是 viable
* Note: If there are **no viable functions,** the **compiler will complain** that there is no matching function.
#### Finding the Best Match, If Any
* 第三步就是從 viable functions 裡面挑一個 best match 的 function(如果有的話)
    * We'll explain the details of "best" in the next section, but the idea is that the closer the types of the argument and parameter are to each other, the better the match.
    * 在這個例子，因為 call f(int) 要做轉換，call f(double, double = 3.14) 不用，所以 f(double, double = 3.14) 比 f(int) 好。
        * 之後會詳細講到底什麼是 best match

#### Function Matching with Multiple Parameters
* 如果 caller 長這樣呢?
    * f(42, 2.56);
* viable function 選法還是一樣，所以剩下 f(int, int) 跟 f(doube, double = 3.14) 這兩個 viable functions
* **The compiler then determines, argument by argument, which function is (or functions *are*) the best match.**

* There is an overall best match if there is one and only one function for which
    * The match for each argument is **no worse than the match** required by any other viable function
        * 否定說明..
    * **There is at least one argument** for which the match is **better than the match provided by any other viable function**
* If after looking at each argument **there is no single function that is preferable, then the call is in error.**
    * The compiler will complain that **the call is ambiguous.**

* 按照上面的規則來選擇 f(42, 2.56); ㄅ...
    * 先看 42, f(int, int) 是 exact match，f(double, double = 3.14) 需要轉換
    * 再看 2.56, f(double, double = 3.14) 是 exact match，f(int, int) 需要轉換
* 結論是這個 call 是 ambiguous!
    * Each viable function is a better match than the other on one of the arguments to the call.
    * 根據上面那個 overall best match 的規則，這裡存在了兩個 function，他們各自擁有一個比其 function 都還要 better match 的參數，這樣 compiler 就會沒辦法決定這兩個該 call 哪一個了
* 你可以用 cast 來強迫選其中一個 function，不過這樣寫就代表你的 overloading design 寫爛了
* Casts should not be needed to call an overloaded function. **The need for a cast suggests that the parameter sets are designed poorly.**

### 6.6.1 Argument Type Conversions
* caller 給的某些 argument 的 type，沒有一個 overloaded function 可以 exactly match，但是都可以轉換，這時候怎麼辦?
    * compiler 實際上會對不同形式的 conversion 做排名
* 按照下列規則排名
    1. An exact match. An exact match happens when:
        * The argument and parameter types are identical.
        * The argument is converted from an array or function type to the corresponding pointer type. (§ 6.7 (p. 247) covers function pointers.)
        * A top-level const is added to or discarded from the argument.
    2. Match through a const conversion (§ 4.11.2, p. 162).
        * low-level nonconst 轉成 low-level const
    3. Match through a promotion (§ 4.11.1, p. 160).
    4. Match through an arithmetic (§ 4.11.1, p. 159) or pointer conversion (§ 4.11.2, p. 161).
    5. Match through a class-type conversion. (§ 14.9 (p. 579) covers these conversions.)
* 這個第一次看可能會黑人問號，不過很久以後回來看一定會有 fu

#### Matches Requiring Promotion or Arithmetic Conversion
* 哎喲 Primer 用放大鏡，你也知道這很積八?
* 你本來就不該寫這種 closely-related type 的 overloading，很雷
* In order to analyze a call, **it is important to remember that the small integral types always promote to int or to a larger integral type.** Given two functions, one of which takes an *int* and the other a *short*, **the short version will be called only on values of type short.**

* Even though the smaller integral values might appear to be a closer match, those values are promoted to int, whereas calling the short version would require a conversion:
    ```cpp
    void ff(int); void ff(short);
    ff('a'); // charpromotes to int; calls f(int)
    ```
    * 上面會 call ff(int);
    * 因為 promotion 比 built-in conversion 的 rank 高

* 此外，All the arithmetic conversions are treated as equivalent to each other.
    * 反正轉一次就好了，管你從什麼型別轉成什麼型別，你們的 rank 都是一樣的
    * The conversion from **int to unsigned int** , for example, **does not take precedence over** the conversion from **int to double.**
    ```cpp
    void manip(long);
    void manip(float);
    manip(3.14); // error: ambiguous call
    ```

#### Function Matching and const Arguments
* 當你 API 長這樣:
    ```cpp
    void f(int&);
    void f(const int&);
    int a;
    const int b;
    ```
* 根據上面規則第二條 f(a) 會 call f(int&), f(b) 會 call f(const int&);
    * f(b) 好理解，因為 viable functions 只有一個
    * f(a) 的話，因為用 reference to const 去 bind nonconst obj 需要做 conversion，所以 reference to plain obj 會優先考慮，所以 f(a) 會 call f(int&)

* 指標跟上述同理
* 總之就是 based on the constness of the argument 來決定要 call reference/pointer to const/nonconst 的版本


## 6.7 Pointers to Functions
* ㄜ.. 已經知道惹

#### Using Function Pointers
* When we use the name of a function as a value, **the function is automatically converted to a pointer.**

    ```cpp
    bool (*pf)(const string &, const string &); // uninitialized
    pf = lengthCompare; (一個 type 跟 *pf 一樣的 function)
    pf = &lengthCompare; (跟上一行等價)
    ```
* Moreover, we can use a pointer to a function to call the function to which the pointer points.
    ```cpp
    bool b1 = pf("hello", "goodbye"); // calls lengthCompare
    bool b2 = (*pf)("hello", "goodbye"); // equivalent call
    bool b3 = lengthCompare("hello", "goodbye"); // equivalent call
    ```
    * 要不要對 function pointer 做 dereference 都可以，上面三個都等價

* There is no conversion between pointers to one function type and pointers to another function type.
    * 不過你還是可以把任意 type 的 function pointer assign nullptr 或 0

#### Pointers to Overloaded Functions
* 幹...
* 反正如果你宣告了一個 function pointer pf，然後寫 pf = func，compiler 會根據 pf 的 type 把正確的 function 丟給它指
    * the compiler uses the type of the pointer to determine which overloaded function to use.
    * 因為 function pointer 不能轉換，只能 match exactly。


#### Function Pointer Parameters
* 之前提過的，Just as with arrays (§ 6.2.4, p. 214), we cannot define parameters of function type but **can have a parameter that is a pointer to function.**
* As with arrays, we can write a parameter that looks like a function type, but it will be treated as a pointer:
    * 所以那些在 parameter list 裡面寫起來像是 function 的鬼東西其實都是 function pointer，不要以為是 function!
        * 就好像你可以寫 void f(int arr[10]); 一樣，那個 10 是假的!

    ```cpp
    void useBigger(const string &s1, const string &s2,
        bool pf(const string &, const string &));
    ```
    * 第三個看起來像是 function 的參數其實是 function pointer
    ```cpp
    // equivalent declaration: explicitly define the parameter as a pointer to function void useBigger(const string &s1, const string &s2,
    bool (*pf)(const string &, const string &));
    ```
    * 直接寫成 function pointer 的樣子，跟上面的例子等價

* 你要把 function 當 argument 傳進去，你就直接把名字丟進去就好了
    * useBigger(s1, s2, lengthCompare);
    * function name 會自動轉成 function pointer

* 但這就跟 pointer to array 一樣，你的宣告會變的很噁心，於是就用 alias 或 decltype

    ```cpp
    // Func and Func2 have function type
    typedef bool Func(const string&, const string&);
    typedef decltype(lengthCompare) Func2; // equivalent type
    // FuncPand FuncP2have pointer to function type
    typedef bool(*FuncP)(const string&, const string&);
    typedef decltype(lengthCompare) *FuncP2; // equivalent type
    ```
    * Both Func and Func2 are function types, whereas FuncP and FuncP2 are pointer types.
    * 這邊用 decltype 也是少數幾個 function name 不會被轉成 pointer 的例子之一

    ```cpp
    // equivalent declarations of useBigger using type aliases
    void useBigger(const string&, const string&, Func);
    void useBigger(const string&, const string&, FuncP2);
    ```
    * 放 `Func` 進去可以過就只是因為 compiler 會自動把它當成 pointer type
#### Returning a Pointer to Function
* 醜死...
* 請用 using
    ```cpp
    using F = int(int*, int); // F is a function type, not a pointer
    using PF = int(*)(int*, int); // PF is a pointer type
    ```
* 雖然在參數列表裡面你可以直接寫一個 function type 然後讓 compiler 自動轉換成 function pointer，可是 return type 不會，我們一定要寫成 pointer type

    ```cpp
    PF f1(int); // ok: PF is a pointer to function; f1 returns a pointer to function
    F f1(int); // error: F is a function type; f1 can’t return a function
    F *f1(int); // ok: explicitly specify that the return type is a pointer to function
    ```
* 你也可以直接硬幹 jserv 流
    ```cpp
    int (*f1(int))(int*, int);
    ```
* 也可以用 C++11 的 trailing return
    ```cpp
    auto f1(int) -> int (*)(int*, int);
    ```

#### Using auto or decltype for Function Pointer Types
* 反正就是用 decltype 來寫 return type
* 記得 function 傳進去不會被轉換成 pointer
    ```cpp
    string::size_type sumLength(const string&, const string&);
    string::size_type largerLength(const string&, const string&);
    // depending on the value of its stringparameter, /
    decltype(sumLength) *getFcn(const string &);
    // getFcn returns a pointer to sumLengthor to largerLength
    ```