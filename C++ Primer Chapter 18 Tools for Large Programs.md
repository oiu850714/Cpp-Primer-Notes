# C++ Primer Chapter 18 Tools for Large Programs
* exception handling, namespaces, and multiple inheritance
* 大專案會需要這三種需求:
    * 可以處理各獨立子系統之間產生的錯誤
    * 容易使用各種(第三方)library
    * 有辦法 model 架構更複雜的問題
* exception handling, namespaces, and multiple inheritance 就可以處理這些

## 18.1 Exception Handling
* Exception handling allows independently developed parts of a program to communicate about and handle problems that arise at run time.
    * 有夠抽象有夠精簡...
* Exceptions let us **separate problem detection from problem resolution.**
    * 感覺這個不錯: http://www.cnblogs.com/lsgxeva/p/7714712.html
    * One part of the program can **detect a problem and can pass the job of resolving that problem to another part** of the program.
    * **The detecting part need not know anything about the handling part, and vice versa.**

### 18.1.1 Throwing an Exception
* In C++, an exception is **raised** by **throwing** an expression.
    * The **type** of the thrown expression, together with the **current call chain**, **determines which** **handler** will deal with the exception.
    * The selected handler is the one **nearest in the call chain that matches the type of the thrown object.**
        * **Type** and **contents** of that object allow the throwing part of the program to **inform the handling part** about what went wrong.

* When a throw is executed, the statement(s) following the throw are not executed.
    * Instead, **control is transferred from the throw to the matching catch.**
    * That catch **might be local to the same function** *or* might be **in a function that directly or indirectly called the function** in which the exception occurred.

* 上面說的隱含兩件很重要的事:
    * 從你 throw 到 match 的 catch 之間的 functions call chain 可能都會提早結束
    * 那些提早結束的 functions 的 local variable 也會在結束時被破壞(呼叫 dtor)

* 因為 `throw` 會讓接在它後面的 statements 都不執行，throw 感覺有點像 `return`
    * Primer 說實務上可能會把 throw 寫在 function 最後面，或者最後一個 condition 內

#### Stack Unwinding
* 那個從 throw 到找到 match 的 catch 的過程:
    * 如果 `throw` 是寫在 `try` block 內，則先檢查對應的 `catch` block 有沒有 match，如果有就由找到的 match 來 handle
    * 如果沒有，那看看 `try` 是不是 nested 在另一個 `try` 內，如果是就找另一個 try 的 `catch` block，有 match 就 handle；這個過程是遞迴的
    * 如果還是沒找到 match(或者 `throw` 壓根就沒寫在 `try` 內)，則當前 `throw` 的 function 會被終止，接下來往 calling function 找 match
    * 如果 calling function 是把這個被結束的 caller 寫在 `try` 內，則從對應的 `catch` 找 match
    * 如果沒找到一樣看這個 `try` 是否 nested 在另一個 `try`，如果有就往另一個 `try` 內的 `catch` 找 match；一樣是遞迴的；
    * 還是沒找到，那 calling function 也會被終止，並且往它的 caller 找；這過程一樣是遞迴的
* 上面的講解可以看 Primer P.773，總之這叫做 **stack unwinding**
    * continues up the chain of nested function calls until a catch clause for the exception is found, or the main function itself is exited without having found a matching catch.
    * 假設 `catch` 有找到，則執行完那個 `catch` 之後，接著會執行那一堆 `catch`es 的最後一個 `catch` 之後的 statements
    * 如果沒找到 `catch` 則程式就結束了
        * 為什麼要這樣設計，因為 exception 的概念就是**一定要被處理的錯誤**，你不可以不處理它，所以一旦找不到 `catch` 可以處理它，程式就會被結束
    * If no matching catch is found, the program calls the library `terminate` function. As its name implies, terminate stops execution of the program.

#### Objects Are Automatically Destroyed during Stack Unwinding
* Ordinarily, local objects are destroyed when the block in which they are created is exited. **Stack unwinding is no exception**.
* When a block is exited during stack unwinding, the *compiler guarantees that objects created in that block are properly destroyed.*
    * If a local object is of class type, the *destructor* for that object is called automatically.
    * As usual, the compiler does no work to destroy objects of built-in type.
* If an exception *occurs in a constructor*, then the **object under construction might be only partially constructed.**
    * Even if the object is only partially constructed, **we are guaranteed that the constructed members will be properly destroyed.**

* Similarly, an exception might occur *during initialization of the elements of an array or a library container type.*
    * Again, we are guaranteed that the elements (if any) that were constructed before the exception occurred will be destroyed.

#### Destructors and Exceptions
* The fact that destructors are run—but code inside a function that frees a resource may be bypassed—affects how we structure our programs.
    * 如果在某 function 內，`throw` 後面有 statements 是在 free resources 的，那就完蛋惹

* On the other hand, resources allocated by an object of class type generally will be freed by their destructor.
    * **By using classes to control resource allocation, we ensure that resources are properly freed, whether a function ends normally or via an exception.**
    * 把 allocate resources 寫在 ctor，deallocate resources 寫在 dtor，就不會發生上面說的事情了--傳說中的 RAII
* 然後注意 dtor 不該 throw execption，**如果要丟也是 dtor 自己要把它包在 `try` 內並且自己 `catch`，否則 `terminate` 會被呼叫**
    * terminate is called if two exceptions are propagating at the same time
* 實務上 dtor 都在 free resources，也很難 throw，至少所有 standard library type 都保證 dtor 不會 throw

#### The Exception Object
* 在 `throw` expression 語句時其實是 copy initialize 一個 exception object，這意味著這個 object 必須要有 **complete type**
    * 主要是因為你藥 `throw` 三小都可以，包刮 `int` 之類的
* 如果 object 是 class type，他一定要有可存取的 dtor，以及 copy/move ctor(要可以刪掉跟複製的意思)
* 如果 object 是 array 或 function type，會被 decay 成 pointer
* **exception object 的生存期跟生存空間比較特別，是由 compiler 所管控的**，它被保證可以被對應的 `catch` 給存取；並且 exception 被 handle 之後，exception 物件會被自動破壞
* **這裡就要跟一般的 local object** 做比較了
    * 如果你 throw 一個 ptr to local object，這基本上一定是 UB，因為 throw 之後，call chain 被 unwinding 時 local object 就被破壞了
    * 上述的原因基本上就跟 return ptr to local object 一樣
    * 所以如果要 throw 指標請 deref 它，這樣它指向的物件就會被 copy initialize
* 還有一點是 **throw expression 的 type 一樣是由它的 static type 決定的**
    * **這跟繼承超有關係**，如果你在 `throw` 時是 deref 一個 base* 指標，可是指標指向的卻是 derived object，那這個物件在 copy initialize 時就會被 slice down! 只有 base-class part 會被 `throw`


### 18.1.2 Catching an Exception
* 接下來看怎麼寫 `catch` clause
    * 看起來跟宣告(只能有一個參數的) function 很像，你必須寫一個 exception declaration 在 `clatch` clause 的括號裡面
    * 如果你的 catch clause 沒有需要使用到這個參數，你一樣可以不給參數名字
* **The type of the declaration determines what kinds of exceptions the handler can catch.**
    * 這個參數的型別決定了這個 `catch` clause 可以處理怎麼樣的 exception
    * 還有之前提過了，型別必須是 complete type
    * 還有型別可以是 lvalue reference type，但不能是 rvalue reference

* 當 `catch` 被進入時，它的參數就會被對應的 `throw` 的 object 給初始化
    * 如果參數是 value type 就用 copy
        * 如果是用 base type 來 copy derived type 物件，那物件會被 sliced down
    * 如果是 reference type 就會綁定 throw 的物件
        * 如果是 base 的 ref 來綁定 derived object，則就按照一般的綁定規則，物件也不會被 slice down
* again，跟 function 一樣的是，parameter 的 **static type** 決定了你可以對這個 parameter 做的操作
* 題外話，上面說的有關繼承的一些問題，可以看下面的連結:
    * https://stackoverflow.com/questions/2522299/c-catch-blocks-catch-exception-by-value-or-reference
    * **Throw by value, Catch by reference**
    * 這樣就不會導致複製時因為繼承的關係物件被 slice down
    * Primer 其實也有提到

#### Best Practice: Ordinarily, a catch that takes an exception of a **type related by inheritance ought to define its parameter as a *reference*.**

#### Finding a Matching Handler
* **first match, not best match**
* 所以最特別的 `catch` clause 要放在所有 `catch` clauses 的第一個
* 例如你要接一個 hierarchy 內的 exception，那你應該把最特別，也就是 hierarchy 最底層的 derived type，擺在第一個 handler，最後再擺 base class handler
    * 不然你把 base 的 clause 擺在 derive clause 的前面，derived clause 永遠不會被執行LOL
    * 或者你可以看 IDE/compiler 丟給你的 warning，它很有可能告訴你有些 handler 永遠不會被執行

* catch parameter 的 match 機制比 function parameter 還要嚴格，除了下面的例外其他 type 都要 exact match，具體如下:
    * Conversions from nonconst to const are allowed.
        * 可用 const ref 來綁 nonconst object
        * 當然如果是 copy value，top-level 一樣是無視
    * Conversions from derived type to base type are allowed.
    * An array is converted to a pointer to the type of the array; a function is converted to the appropriate pointer to function type.

* No other conversions are allowed to match a catch. In particular, *neither the standard arithmetic conversions nor conversions defined for class types are permitted.*
    * 其他轉型皆不會發生，具體來說就是 arithmetic conversion，跟 class defined 的 conversion 都不會
    * **包括從 string literal 轉成 `std::string` 之類的**，handler 這邊都不會轉型:
    ```C++
    void f() { throw "wow!"; }
    int main() {
         try {
             f();
        } catch (int i) {
            cout << "const int catch int!";
        } catch (double d) {
            cout << "double can be after the int";
        } catch (string s) {
            cout << "can catch string literal??\\n";
        }
    }
    ```
    * 上面這段 code 會 terminate，`string` 不能接 string literal

#### Rethrow
* 有時候某個 handler 抓到了 exception，可是他只能處理某些錯誤，其他錯誤它無法解決，這時 handler 可以 **rethrow** exception
    ```C++
    throw;
    ```
* empty `thrwo` 只能出現在 `catch` 或 `catch` clause 直接或間接呼叫的 function 內；
    * 如果在其他地方出現 empty `throw` 結果現在整個 program 沒有在 exception 狀態，`terminate` 會被呼叫
* empty `throw` 會把「當前」 exception 繼續往 call chain 傳遞
    * 注意這個「當前」是指 compiler 幫你 maintain 的那個 exception 物件，而不是你的 `catch` 的 parameter
        * 你的 parameter 如果是 reference type，然後有對綁定的物件做更改，那你 rethrow 時這些更改就會傳遞下去，因為你綁定的物件就是那個 compiler 幫你維護的「當前」物件
        * 可是如果你的 parameter 是 value type，那你就不會改到 compiler 維護的那個物件，而是更改一份那個物件的 copy，所以你 rethrow 時 compiler 維護的那個物件的內容不會被改變
    * 還是看例子...:
    ```C++
    catch (my_error &eObj) {     // specifier is a reference type 
        eObj.status = errCodes::severeErr; // modifies the exception object     
        throw; // the status member of the exception object is severeErr
    } catch (other_error eObj) { // specifier is a nonreference type eObj.status = errCodes::badErr; // modifies the local copy only 
        throw; // the status member ofthe exception object is unchanged
    }
    ```
    
#### The Catch-All Handler
*  have the form `catch(...)`, matches any type of exception.
* Primer 說這種用法的 `catch` clause 通常會 rethrow:
	```C++ 
	void manip() { 
		try {
			// actions that cause an exception to be thrown 
		}
		catch (...) { 
			// work to partially handle the exception
			throw;
		}
	}
	```
* 注意 `catch(...)` 一定要寫在最後一個，不然它之後的 `catch` clause 永遠不會執行
### 18.1.3 Function try Blocks and Constructors
* exception 可以發生在 program 的任何地方，包刮 ctor
* 可是要怎麼 handle ctor 內的 exception? ctor init list 呢?
	* 首先你仔細想一下，你在 ctor 內寫的 try block 是沒有辦法接收 ctor init list 噴的 exception 的，因為在執行 init list 階段時，ctor body 根本還沒執行
* 所以要用一種特別的語法叫做 function try blocks
* 語法直接看例子比較快...:
	```C++
	template <typename T> Blob<T>::Blob(std::initializer_list<T> il) 
	try : 	
		data(std::make_shared<std::vector<T>>(il)) {
			/* empty body */
		} 
		catch(const std::bad_alloc &e) 
		{ handle_out_of_memory(e); }
	```
	* 要在 ctor init list 的冒號之前寫上一個 `try`，然後在 ctor body 之後寫上 `catch`，這樣不管是發生在 ctor init list 或者 ctor body 內的 exception 都可以被 catch 接到
* 同樣的寫法也可以用在 dtor，這裡就不多講

* 還有一點很重要，**在初始化 ctor parameter 的階段也有可能會發生 exception**，不過這個時間點發生的 exception 就不是 ctor 可以處理的了，就算你用 function try block 的寫法也處理不到；這是 caller 那邊會處理的(你可以想像成，如果你是 copy init parameter 之類的，那就是 parameter 自己的 ctor 要處理 exception，總之不會是當前的 ctor 要處理)
	* As with any other function call, **if an exception occurs during parameter initialization, that exception is part of the calling expression and is handled in the caller’s context.**

### 18.1.4 The `noexcept` Exception Specification
* It can be helpful both to users and to the compiler to know that a function will not throw any exceptions.
	* Knowing that a function will not throw simplifies the task of writing code that calls that function.
	* 去查 reference 時也會跟你說哪個 API 有沒有 `noexcept`
	*  Moreover, if the compiler knows that no exceptions will be thrown, *it can (sometimes) perform optimizations that must be suppressed if code might throw.*
* 語法:
	```C++
	void recoup(int) noexcept; // won’t throw 
	void alloc(int); // might throw
	```
	* We say that `recoup` has a **nonthrowing specification**.
* The noexcept specifier must appear on all of the declarations and the corresponding definition of a function or on none of them.
	* 要馬宣告跟定義都要寫 `noexcept` 要馬都不要寫
	* `noexcept` 要放在 `const`/ref qualifier 後面，`final`/`override`/`=0` 的前面
#### Violating the Exception Specification
* 注意，你還是可以在 `noexcept` 的 function 內 `throw`，這是合法的，只是 compiler 可能會噴 warning:
	```C++
	// this function will compile, even though it clearly violates its exception specification 
	void f() noexcept // promises not to throw any exception
	{ 
		throw exception(); // violates the exception specification
	}
	```
* **If a noexcept function does throw, terminate is called,** thereby enforcing the promise not to throw at run time.
* **It is unspecified whether the stack is unwound.**
* `noexcept` 通常用在兩種情況:
	* 我們很確信 function 不會 `throw`
	* 或者會 `throw`，可是根本不能處理:
		* 例子有你在 function 內創造一個 `vector` 結果他卻噴 `bad_alloc` 之類的，這種 error 根本無法處理，因為沒有記憶體了
		* 可是你還是可以標記成 `noexcept`，因為這種 error 幾乎等於 program 無法繼續正常執行，被 terminate 也是合理

* `noexcept` 還有一個「概念」: 它免除了 caller 必須處理 exception 的責任，就算最後 program 被 terminate，也不會是 caller 的錯

* BACKWARD COMPATIBILITY: EXCEPTION SPECIFICATIONS
	* 早期可以寫成 `void recoup(int) throw(blablabla);` 來說明這個 function 會 `throw` blablabla 這些 types，不過這種寫法已經是 deprecated 而且幾乎沒人在用；但有一個寫法例外，而且常常用:
	```C++
	void recoup(int) noexcept; // recoup doesn’t throw 
	void recoup(int) throw(); // equivalent declaration
	```
	* 如果你的 `throw()` 內沒東西，代表這個 function 不會 `throw`，因此上面兩個寫法是等價的

#### Arguments to the noexcept Specification
* `noexcept` 可以吃一個 argument，必須要能轉成 `bool`，如果是 `true`，那 function 不會 `throw` 反之則會:
	```C++
	void recoup(int) noexcept(true); // recoup won’t throw
	void alloc(int) noexcept(false); // alloccan throw
	```
#### The noexcept Operator
* 注意這裡講的 `noexcept` 是一種 operator! 它吃一個 operand，會 return `true`/`false` 的 **const expression**
* 而且它跟 `sizeof` 一樣，**不會 evaluate 它的 operand**
* 舉例...:
	```C++
	noexcept(recoup(i)) // true if calling recoupcan’t throw, falseotherwise
	```
	* 如果 `recoup(i)` 不會 throw，則 `noexcept(recoup(i))` 就會是 `true`，而因為我們把 `recoup` 宣告成 `noexcept`(注意這裡是 specifier，不是 operator)，所以這個情況是 return `true`
* 更一般來說:
	```C++
	noexcept(e)
	```
	* 會 return `true`，如果:
		* 被 e 呼叫的所有 function 都有 `noexcept` specifier
		* e 自己本身也是 `noexcept`
		* 否則 return `false`
* 我們可以寫出這種很莫名其妙(?)的東西:
	```C++
	void f() noexcept(noexcept(g())); // f has same exception specifier as g
	```
	* 如果 `g` 是 `noexcept`，則 `f` 也是 `noexcept`

* 總之 `noexcept` 有兩個意思，一個是 specifier，一個是 operator

#### Exception Specifications and Pointers, Virtuals, and Copy Control
* 注意，`noexcept` specifier 不是 function type 的其中一部份；但它卻會影響 function 怎麼被使用
* 首先先說，`noexcept` 也可以用在 function pointer 的宣告......
* 沒宣告成 `noexcept` 的 ptr 可以指向的 function，要不要宣告 `noexcept` 都可以，可是宣告成 `noexcept` 的 ptr 只能指向有宣告 `noexcept` 的 function，指向沒宣告的會噴 error:
	```C++
	// both recoup and pf1 promise not to throw 
	void (*pf1)(int) noexcept = recoup;
	// ok: recoup won’t throw; it doesn’t matter that pf2 might 
	void (*pf2)(int) = recoup;
	pf1 = alloc; // error: alloc might throw but pf1said it wouldn’t
	pf2 = alloc; // ok: both pf2 and alloc might throw
	```
* 再來，如果 base class 的 virtual function 不會 `throw`，則 derived override 時一樣也不能 `throw`
	* 如果 base 沒有 `throw`，則 derived 要不要 `throw` 都可以:
	```C++
	class Base { 
	public:
		virtual double f1(double) noexcept; // doesn’t throw 
		virtual int f2() noexcept(false); // can throw 
		virtual void f3(); // can throw
	};
	class Derived : public Base {
	public: 
		double f1(double); // error: Base::f1 promises not to throw
		int f2() noexcept(false); // ok: same specification as Base::f2
		void f3() noexcept; // ok: Derived f3 is more restrictive
	};
	```
* 再來是有關 copy-control 的 member
	* compiler 幫我們和成的時候也會指定 `noexcept` specifier
	* 如果 copy-control member 使用到的 operation(例如 copy ctor 會用 data member 的 copy ctor)全部都是 `noexcept`，則合成出來的 copy-control 就會是 `noexcept`
	* 反之就是 `noexcept(false)`
	* 還有，就算我們的 dtor 有自定義，可是沒有指定 `noexcept` specifier，compiler 還是會幫我們合成，而合成出來的結果會跟 compiler 自己合成一個 dtor 的 specifier 一致

### 18.1.5 Exception Class Hierarchies
![](https://i.imgur.com/rQGSuoq.png)
* 這是 standard 定義的 exception hierarchy
* base class `exception` 定義 copy ctor, copy assign, virtual dtor, 跟一個 member `what`，會回傳 const char*；然後 `exception` 本身 guarantee 不會再噴 exception
 * `exception`, `bad_cast`, `bad_alloc` 也定義了 default ctor
 * `runtime_error`, `logic_error` 沒有 default ctor，不過有定義兩個 ctor，一個是 C style string, 一個吃 `std::string`
	 * Those arguments are intended to give additional information about the error.
* 這些 class 的 `what` member 基本上都是回傳初始化 exception object 時給的 message
* 然後如果用 catch by reference to `exception` 就可以用 `virtual` 的特性呼叫對應 dynamic type 的 `what`

#### Exception Classes for a Bookstore Application
* 通常要寫有 exception handling 的應用時會定義一個 class，來繼承 hierarchy 的某個 class(你要直接繼承 `exception` 或者他的 derived 都可以，看你想幹嘛)
* 甚至繼承之後，自己定義的 exception 一樣有一個 hierarchy，超噁
* 假設真的寫一個 bookstore 應用，而不是 Primer 書內介紹的 toy example，可能會定義這樣的 exception:
	```C++
	// hypothetical exception classes for a bookstore application 
	class out_of_stock: public std::runtime_error {
	public:
		explicit out_of_stock(const std::string &s): std::runtime_error(s) { }
	};
	class isbn_mismatch: public std::logic_error {
	public:
		explicit isbn_mismatch(const std::string &s): std::logic_error(s) { }
		isbn_mismatch(const std::string &s, 
			const std::string &lhs, const std::string &rhs):
			std::logic_error(s), left(lhs), right(rhs) { } 
		const std::string left, right;
	};
	```
* 這裡就有點 design pattern 了，總之你寄成越 base 的 class 就會代表越 general 的 error，例如直接繼承 `exception` 就代表 "something went wrong"
* 標準的 hierarchy 第二層就分成兩大類: `runtime_error` 跟 `logic_error`
	* Run-time errors represent things that **can be detected only when the program is executing.**
	* Logic errors are, in principle, **errors that we could have detected in our application.**

* 例如我們定義的 `out_of_stock` 是繼承 `runtime_error`，因為它概念上就是 runtime 才有可能發生的事情，訂單不能滿足之類的
* 而 `isbn_mismatch` 是繼承 `logic_error`，因為代表這個東西會影響你程式的邏輯(不能輸入一個非法的 ISBN 之類的)

#### Using Our Own Exception Types
* 接下來就是在我們寫的 `Sales_data` 內使用我們定義的 exception 惹
* 記得，program 的某 part 會丟 exception，某個 part 則是 handle exception
* 舉例: `operator+=`:
	```C++
	// throws an exception if both objects do not refer to the same book 
	Sales_data&
	Sales_data::operator+=(const Sales_data& rhs) {
		if (isbn() != rhs.isbn()) 
			throw isbn_mismatch("wrong isbns", isbn(), rhs.isbn());
		units_sold += rhs.units_sold; 
		revenue += rhs.revenue; 
		return *this;
	}
	```
* 而如果你還記得的話，我們在加總 revenue 的 code 可以這樣改寫:
	```C++
	// use the hypothetical bookstore exceptions 
	Sales_data item1, item2, sum; 
	while (cin >> item1 >> item2) { // read two transactions 
		try {
			sum = item1 + item2; // calculate their sum 
			// use sum
		} catch (const isbn_mismatch &e) { 
			cerr << e.what() << ": left isbn(" << e.left 
				 << ") right isbn(" << e.right << ")" << endl;
		}
	}
	```
	* 基本上就是把輸入的部份給用 try 包起來

