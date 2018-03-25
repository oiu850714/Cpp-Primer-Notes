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

## 18.2 Namespaces
* 再強調一次這是要解決你用了一狗屁第三方 lib 時的好 solution
* 如果每個 lib 都超大，然後他們又有一些名字是 global 可見的(例如 class，functions, templates)，那你同時使用這些 lib 會有很高的機率發生 name collision
* 我們也說把 name 直接放到 global 的 lib 導致 **namespace pollution.**
* 所以要使用 namespace 提供的機制來避免這個問題，之後介紹
* **Namespaces partition the global namespace.**
### 18.2.1 Namespace Definitions
* keyword `namespace` 加上你要定義的 name，配上一個 {}
	* 裡面可以放各種 type 的 name，class, function, variable 等等
* 例子:
	```C++
	namespace cplusplus_primer { 
		class Sales_data { /* .. . */}; 
		Sales_data operator+(const Sales_data&, const Sales_data&);
		class Query { /* .. . */}; class Query_base { /* .. . */};
	}// like blocks, namespaces do not end with a semicolon
	```
	* 記住 namespace 之後跟的 {} 不用加分號
* namespace 本身的 name 在 scope 內要 unique
* namespace 可以直接定義在 global scope 或者其他的 namespace 內
* **但不能定義在 class/function 內**

#### Each Namespace Is a Scope
* 既然 namespace 定義了一個新的 scope，他裡面的 name 就可以有 scope 的特性
	* 例如 namespace 裡面的 name 要 unique，不能重定義
	* 不同 namespace 內可以有相同名字的 name
* namespace 的某個 name 可以被同 namespace 內的其他 names(或者同 namespace 內再定義的 namespace 內的 names) 給存取
* 而在 namespace 之外要存取 namespace 內的 name 就需要用 scope operator 了
	```C++
	cplusplus_primer::Query q = 
			cplusplus_primer::Query("hello");
	```
* 假設有另一個 namespace `AddisonWesley` 也定義了 `Query`，我們要使用這個 `Query` 可以改成這樣:
	```C++
	AddisonWesley::Query q = AddisonWesley::Query("hello");
	```
#### Namespaces Can Be Discontiguous
* 16.5(p.709) 有說到，**同一個** namespace 可以被定義在好幾個地方:
	```C++
	namespace nsp { 
		// declarations
	}
	```
	* 上面這樣的 code 可以是定義一個新的 namespace，也可以是為一個已經存在的 namespace 增加新成員；如果是後者，我們稱為 **open a namespace**
		* 如果這個 namespace 還不存在就創造一個新的 namespace
		* 如果存在就把你現在新增的 names 加到原本的 namespace
* **因為 namespace 可以不連續的被定義的特性，讓我們可以很容易的把不同的 implementations/interface 定義在不同的 source/header files，但是都屬於同一個 namespace**:
	* 在 header 內定義一個 namespace，然後把你想宣告的 class/function/etc 定義在 namespace 內
	* 然後要用到這些 name 的 source file 就 include header，並且 *open* 這個 namespace，在這個 namespace 內實作 header 宣告的 interface
	* 簡而言之就跟以前把 interface/implementation 分別放在 header/source 概念一樣，只不過現在都塞在 namespace 裡面
	* 這樣做的背後意義只有一個，再強調一次，就是 **partition global scope**

* Best Practice: 你可以把屬於同個 namespace 但是不太相干(或者一堆相干的 types)分別定義在不同的 source files，這樣他們都屬於同個 namespace，但又沒有全部在同個檔案，more readable and modularization

#### Defining the Primer Namespace
* 直接看例子比較好懂...
* 我們的 `cplusplus_primer` namespace 會定義在很多個 files 內
	* 例如定義了 `Sales_data` 的部分會定義在 `Sales_data.h`，
	* 定義 `Query` (15章的東西)的部分會定義在 `Query.h`
	* and so on
	* 對應的 source file 則是 `Sales_data.cc` and `Query.cc`

* 以下是 code 結構:
	```C++
	// ---- Sales_data.h---
	// #includes should appear before opening the namespace 
	#include <string> 
	namespace cplusplus_primer { 
		class Sales_data { /* .. . */}; 
		Sales_data operator+(const Sales_data&, const Sales_data&);
		// declarations for the remaining functions in the Sales_data interface 
	}
	```
	```C++
	// ---- Sales_data.cc ---
	// be sure any #includes appear before opening the namespace 
	#include "Sales_data.h" 
	namespace cplusplus_primer { 
		// definitions for Sales_data members and overloaded operators
	}
	```
* 而這時 user code 要用我們的 lib 就會有好處了，他想要用什麼功能就 include 什麼 header，而且這些 name 全部都屬於 `cplusplus_primer`，所以前面全部都只要加上 `cplusplus_primer::` 就行了:
	```C++
	// ---- user.cc ---
	// names in the Sales_data.h header are in the cplusplus_primer namespace 
	#include "Sales_data.h" 
	int main() {
		using cplusplus_primer::Sales_data; 
		Sales_data trans1, trans2; 
		// ... 
		return 0;
	}
	```
* This **program organization** gives the developers and the users of our library **the needed modularity.**
	* 其實這種 practice 就跟 `std` 這個 namespace 的做法一樣，你想要用什麼功能就 include 什麼 standard header，然後 name 全部定義在 `std`
	* Each class is still organized into its own interface and implementation files.
	* A user of one class **need not compile names related to the others.**

#### Defining Namespace Members
* 你在命名空間內可以直接使用其他同空間內的名字，不用加 scope:
	```c++
	#include "Sales_data.h" 
	namespace cplusplus_primer { // reopen cplusplus_primer 
		// members defined inside the namespace may use unqualified names 
		std::istream&
		operator>>(std::istream& in,  Sales_data& s) { /* .. . */}
	}
	```
	* 內部使用 `Sales_data` 時就不用加 scope 了
* 除此之外也可以在命名空間外**定義** 空間內的 name:
	* 不過 name 的**宣告**還是要出現在命名空間內才可以
	```c++
	// namespace members defined outside the namespace must use qualified names 
	cplusplus_primer::Sales_data 
	cplusplus_primer::operator+(const Sales_data& lhs, 
								const Sales_data& rhs)
	{
		Sales_data ret(lhs); // ...
	}
	```
	* 注意看清楚，把 name 定義(或使用)在命名空間外面時，什麼時候必須加 scope，什麼時候不用
		* 大抵上跟在 class 外定義 member function 很類似
* 還有，命名空間的 name 只能定義在空間內，或者包含這個空間的 enclosing namespace；用上面的例子來說，`cplusplus_primer` 這個命名空間的外包空間就是 global namespace，所以你可以把 name 定義在 global；可是你不能把屬於 `cplusplus_primer` 的 name 定義在不相干的命名空間，例如 global scope 的某個 `foobar` namespace 內

#### Template Specializations
* 16.5 有先**直接用過**以下功能: template specialization 一定要跟 template 本身定義在同一個命名空間
* 當時是為了自己寫一個 `Sales_data` 版本的 `hash`，而 `hash` 這個 template 定義在 `std` 內，所以當時寫了一段 code 把 `std` **打開**，然後將 `hash<Sales_data>` specialization 定義在裡面
* specialization 跟前面的例子一樣，只要我們先在空間內**宣告** name，我們還是可以把它定義在空間外面:
	```c++
	// we must declare the specialization as a member of std 
	namespace std { 
		template <> struct hash<Sales_data>;
	}
	// having added the declaration for the specialization to std 
	// we can define the specialization outside the std namespace 
	template <> struct std::hash<Sales_data> {
		size_t operator()(const Sales_data& s) const 
		{
			return hash<string>()(s.bookNo) ^ 
				   hash<unsigned>()(s.units_sold) ^ 
				   hash<double>()(s.revenue); }
	// other members as before
	};
	```
#### The Global Namespace
* 除了定義在 function, class, namespace 內的 name，其他的 name 全部都屬於 global namespace
* 你可以想成全域命名空間是一個 implicitly 宣告的 namespace，只要整個 program 的 source files 有把名字定義在 global scope，就會在全域命名空間內增加名字
* 除此之外，想要直接使用某個全域空間的名字，一樣可以使用 scope operator，但是 operator 左邊不加任何 namespace:
	```c++
	::member_name
	```
#### Nested Namespaces
* 定義在命名空間的命名空間 lol
	```c++
	namespace cplusplus_primer { 
		// first nested namespace: defines the Query portion of the library 
		namespace QueryLib { 
			class Query { /* .. . */}; 
			Query operator&(const Query&, const Query&); 
			// ...
		}
		// second nested namespace: defines the Sales_data portion of the library 
		namespace Bookstore { 
			class Quote { /* ... */}; 
			class Disc_quote : public Quote { /* .. . */}; 
			// ...
		}
	}
	```
	* 一樣，內部的空間的名字會隱藏外部空間的名字
	* 外部空間要用內部空間的 name 一樣要加 scope operator:
	```c++
	cplusplus_primer::QueryLib::Query
	```
#### Inline Namespace
* C++11
* 最大的特點: **可以在某個 inline 命名空間的 enclosing namepsace 內使用 inline 命名空間的名字**
* 用 `inline` keyword 宣告:
	```c++
	inline namespace FifthEd { 
		// namespace for the code from the Primer Fifth Edition
	}
	namespace FifthEd { 
		// implicitly inline 
		class Query_base { /* .. . */}; 
		// other Query-related declarations
	}
	```
	* 注意，如果要把命名空間宣告為 `inline`，則你必須在**第一次定義命名空間時就加上 `inline`
	* 之後如果 reopen 命名空間，可加也可不加 `inline`，但是都是 implicitly `inline`

* Inline namespaces are **often used when code changes from one release of an application to the next.**
* inline 命名空間可以解決 code 版本變更時會出現的一些問題，詳見下面:
* 例如我們可以把第五版的 code 定義在一個 `inline` 命名空間，叫做 `FifthEd`；然後有關前面版本的 code 則是定義成 non-`inline`:
	```c++
	namespace FourthEd { 
		class Item_base { /* ... */}; 
		class Query_base { /* .. . */}; 
		// other code from the Fourth Edition
	}
	```
* 然後在 `cplusplus_primer` 內 **include** 這些命名空間:
	```c++
	namespace cplusplus_primer { 
	#include "FifthEd.h" 
	#include "FourthEd.h"
	}
	```
	* 這樣寫的話第四版跟第五版所定義的命名空間都會是 `cplusplus_primer` 的子空間
	* 然後因為 `FifthEd` 是 inline 命名空間，使用到 `cplusplus_primer::` 的 user code 就可以直接使用到 `FifthEd` 定義的 name
	* 如果我們想要使用舊版的 code，我們就還是照一般的寫法，`cplusplus_primer::FourthEd::Query_Base` 這樣

* 說真的這裡說 inline 命名空間最主要的功能是這個，有點看不懂，可能可以啃一下下面這個:
	* https://stackoverflow.com/questions/11016220/what-are-inline-namespaces-for

#### Unnamed Namespaces
* keyword `namespace` 後面直接跟上一對大括號的 code 就是 unnamed namespace
* 無名空間的 name 存活時間是 static lifetime: They are created before their first use and destroyed when the program ends.
* 無名空間一樣可以不連續，但只能在同一個 source file 內，無法跨越多個 source file
* 換句話說每個 source file 都有自己的一個無名空間(每個 source file 都可以有，**但都是不同的**)
	* 例如兩個 source file 的無名空間都可以**定義**同一個 name，可是這些 name 是對應到不同的實體(entities)
* 你如果把無名空間定義在 header，**這樣的話所有 include 這個 header 的 source 都會有一份「不相干」的無名空間**(只是都會有 header 定義的無名空間定義的 names)；這些不相干的無名空間內的同樣名字的 name 還是會對應到不同的實體
* 定義在無名空間的 name 就能直接用，畢竟沒有命名空間的名字給你用...，所以不可能用 scope operator 來指定無名空間內的 name lol
* 再來，定義在無名空間的 names 的 scope，就是跟定義無名空間的那個 scope:
	* 例如你在 global scope 定義無名空間，則無名空間的 names 的 scope 就是 global scope
* 因此下面的 code 會造成 ambiguous:
	```c++
	int i; // global declaration for i 
	namespace { 
		int i;
	}
	
	int main(){
		// ambiguous: defined globally and in an unnested, unnamed namespace
		i = 10;
	}
	```
	* 你如果要把無名空間定義在 global scope，則空間裡面的 name 不能跟 global scope 內的 name 重複
* 在所有其他方面，無名空間內的 name 就跟一般的 entities 沒什麼兩樣
* 無名空間也跟其他命名空間一樣，可以 nest 在其他命名空間內
* 如上這樣定義的話，code 就會長類似這樣:
	```c++
	namespace local { 
		namespace { 
			int i;
		} 
	}
	// ok: i defined in a nested unnamed namespace is distinct from global i
	local::i = 42;
	```
	* 總之就是少了一層 scope operator

#### UNNAMED NAMESPACES REPLACE FILE STATICS
* 這裡很ㄎㄧㄤ，這裡在說 C 在 global scope 宣告 name 成 `static`，以達到其他 file 看不到這個 name 的效果，這叫做 *file statics*
* 這在 C++ 被認為是 deprecated，我們應該要用無名空間來達到這樣的功能(你定義在無名空間的 name 其他 file 看不到)
	* 這個看看就好啦，你很多情況還是要接 C code

### 18.2.2 Using Namespace Members
* 一直在那邊用 scope operator 取得命名空間內的 name 有時候真的很煩
* 解決方法有三個，下面介紹:
	* `using` declaration
	* namespace aliases
	* `using` directives
#### Namespace Aliases
* 就為某個命名空間取個別名的意思:
	```c++
	namespace cplusplus_primer { /* ... */};
	namespace primer = cplusplus_primer;
	```
* It is an **error** if the original namespace name has not already been defined as a namespace.
* 你也可以直接對某個空間的 nested 空間取別名:
	```c++
	namespace Qlib = cplusplus_primer::QueryLib; 
	Qlib::Query q;
	```
	`Qlib::Query` 其實是 `cplusplus_primer::QueryLib::Query`
* 總之別名宣告後都可以交互使用

#### `using` Declarations: A Recap
* 一次引入某個命名空間的*一個* name
* 某種程度上來說是以最小最精細的單位引入 name，以防止 collision

* Names introduced in a using declaration obey normal scope rules:
	* They are **visible from the point of the using declaration to the end of the scope** in which
the declaration appears.
	* hidden outer scope
	* 被 using 宣告的 name 只能在當前 scope 或 nested scope 直接使用
	* 一旦出當前 scope，你還是要用 scope operator 去指定 name
* using declaration 可以出現在 global, local, namespace, or class scope.
	* **注意在 class 內，using declaration 只能 refer to base class member**
	* Primer 說 15 章有提 LOL，可是那邊明明只有說可以 using base 的 member，沒有說不能 using 其他東西

#### `using` Directives
* 一次直接引入整個命名空間內的 names
	```c++
	using namespace namespace_name;
	```
* It is an error if the name is not a previously defined namespace name.
* using directive 可以出現在 global, local, or namespace scope, 但不能出現在 class 內
* These directives **make all the names from a specific namespace visible without qualification.**
* 直到使用 using directive 的 scope 結束為止你都可以不用使用 scope operator 取得 name
#### **Warning: 盡量不要用 using directives, 很容易撞名，撞的你不要不要的，尤其是用在那種我們不能控制的 namespace，比如說 `std`**

#### `using` Directives and Scope
* 用 `using` directives 引入的那些 names 的 scope 比用 `using` declaration 更複雜
* 如果你在某個地方使用了 `using` directives，則被使用的命名空間內的 names 的 scope，會被*擴展*到同時包含那個 namespace 以及使用 `using` directives 的地方
* 為什麼會做這件事情? **因為 namespace 內可能會有 local scope 不能放入的 member**
	* 還記得可以在 function 內用 `using` directives 嗎? 如果假設 `using` directives 是把被使用的命名空間的 scope 拉到使用 `using` directives 的地方，然後那個地方只是某個 ordinary function，這樣不就代表我們在 function 內定義了一個 class 嗎?
	* 所以解決方法就是把被 using 的 namespace 的 names 丟到同時包含 namespace 跟 `using` directive 的地方
* 舉例，假設我們有 namespace `A` 跟 function `f`，如果在 `f` 內寫 `using namespace A;` 的話，這樣在 `f` 內就宛如 `A` 的 members 是在 global scope，也就是包含 `A` 跟 `f` 的 scope:
	```c++
	// namespace A and function f are defined at global scope 
	namespace A { 
		int i, j;
	}
	void f() {
		using namespace A; // injects the names from A into the global scope 
		cout << i * j << endl; // uses i and j from namespace A 
		// ...
	}
	```
#### using Directives Example
* 看更複雜的例子"
	```c++
	namespace blip { 
		int i = 16, j = 15, k = 23; 
		// other declarations
	}
	int j = 0; // ok: j inside blip is hidden inside a namespace 
	void manip() {
		// using directive; the names in blip are ‘‘added’’ to the global scope 
		using namespace blip; 
		// clash between ::j and blip::j 
		// detected only if j is used
		++i; // sets blip::ito 17
		++j; // error ambiguous: global j or blip::j?
		++::j; // ok: sets global j to 1
		++blip::j; // ok: sets blip::j to 16 
		int k = 97; // local k hides blip::k 
		++k; // sets local kto 98
	}
	```
	* The using directive in manip makes all the names in blip directly accessible;
	* 注意在 `using` directive 時已經有 name collision，不過只要沒使用就不會有事
	* 注意 `manip` 內宣告的 `k` 比較特別
		* 假設現在在 global 宣告 k(並且 `blip` 也有 `k`)，也不會噴 error，因為 `manip` 內宣告的 k 跟另外兩個 k 的 scope 都不同
* 題外話，我們說把 name 提到同時包含 `using` 跟 namespace 的地方叫做 injected in namespace
* 上面也提過了，這會導致 name collision
	*  Such conflicts are permitted, but to use the name, we **must explicitly indicate which version is wanted.**

#### Headers and using Declarations or Directives
* 如果你在 header 的 top-level scope 使用 `using` directives，則 include header 的 source file 都會在 top-level scope 看到 `using` directives 的 namepace 定義的所有 names
* 總之不要在 header 的 top-level scope 用 `using` declaration/directives...

#### CAUTION:AVOID USING DIRECTIVES
* 總之盡量不要用，當你專案一大，用的 library 一多，一定會 name collision
* 而且這種問題可能會開發到一半才發生，例如你用了新的 lib 才 name collision
* 還有，由於 `using` directives 導致的 ambiguity 只會在你真的有使用到對應的 name 時 compiler 才會噴 error
	* 一樣，一開始不會噴，當你用到才會噴，這樣很機掰
* 如果你真的想減少 code 量，要用也是用 `using` declaration，這樣會減少 collision 的機會，*並且你在使用 `using` declaration 時有撞名的話就會馬上噴 error 了
* `using` directive 真的要用的話也是用在 implementation files

#### 18.2.3 Classes, Namespaces, and Scope
* 看例子比較快，反正 namespace 的 look up rule 就跟以前一樣:
	```c++
	namespace A { 
		int i;
		namespace B { 
			int i; // hides A::i within B
			int j; 
			int f1() {
				int j; // j is local to f1 and hides A::B::j 
				return i; // returns B::i
			}
		} // namespace B is closed and names in it are no longer visible 
		int f2() { return j; 
			// error: j is not defined
		} 
		int j = i;  // initialized from A::i
	}
	```
	* 總之先在當前 scope 找 name，沒有就往 enclosing scope 找

* 當 class 定義在 namespace ，一樣還是用 normal lookup
	* 某 member function 使用 name 時，先找 function 內的 name；再找 class 內的 member(包含 base class)，再來是 enclosing scope:
	```c++
	namespace A { 
		int i; 
		int k; 
		class C1 { 
		public:
			C1(): i(0), j(0) { } // ok: initializes C1::i and C1::j 
			int f1() { return k; } // returns A::k 
			int f2() { return h; } // error: h is not defined 
			int f3();
		private: 
			int i;  // hides A::i within C1
			int j; 
		};
		int h = i; // initialized from A::i
	} 
	// member f3 is defined outside class C1 and outside namespace A
	int A::C1::f3() { return h; } // ok: returns A::h
	```
* 除了 class member function 找 name 時會把 class 內的所有 member 看過以外(7.4.1, p.283, 總之就是在說 class definition 會先把宣告都看完才編譯 member 定義)，其他的 scope rule 都是從你使用某個 name 的某行 code 開始「**往上**」找
	* 也就是說在使用某個 name 之前必須要看到他們被宣告
	* 所以 `f2` 不能編譯，因為在看 `f2` 的定義的時候 normal lookup 找不到 `h`
	* 但是 `f3` **可以**編譯，因為在看 `f3` 的定義的時候，`h` 已經被定義了

#### Argument-Dependent Lookup and Parameters of Class Type
## 超級重要
* 看下面這段 code:
	```d++
	std::string s; 
	std::cin >> s;
	```
* 第二行會呼叫 `operator>>(std::cin, s);`
* 這個版本的 `operator>>` 是 `std::string` 定義的，**換句話說是在 `std` 這個命名空間定義的**
	* **但是我們卻不用使用 `std::operator>>` 或用 using declaration 來取得 `std` 內的這個 name**
* 這是一個特殊例外:
	* **當我們把某個 class object 傳入 function 時，compiler 除了按照一般的 lookup rule 找 name 以外，還會在那個 class 定義的的 namespace 內找 name**
	* 直接傳入 class object, ref/ptr to class object，都會觸發 argument dependent lookup
* 在上面的例子，當 compiler 看到 `std::cin >> s`，也就是 function call 時，他會從 current scope 開始找 name，找不到就往 enclosing scope and so on，也就是一般的 normal lookup；
* 除此之外，因為 function call 傳入的 `std::cin` 跟 `s` 是 class object，compiler 也會去搜尋定義這些 object 的 class 的 scope，也就是 `std`，然後就會找到對應的 `operator>>`
* 這個例外就可以讓那些在 namespace 內，構成 class interface，但是是 nonmember function 的 name 可以直接被使用，例如這邊的 `operator>>` 就是 nonmember function，但是構成 `iostream` 的 interface

* 如果沒有這個例外，那你就不能可能用直接使用 operator 的寫法(`std::cin >> s`)，而是要用 `std::operator>>(std::cin, s)`，或者要先用 using declaration(`using std::operator>>;`)，不管哪一種都更複雜(想想每個構成 interface 的 nonmember function 都要這樣搞的情況)

#### Lookup and std::move and std::forward
* 很多 C++ programmer 根本不知道 ADL
* 先來看這個情境: 當 user code 定義了一個 name，這個 name 在 standard 內也有定義，怎以下兩件事情其中一個會發生:
	* 正常的 overload 會決定要呼叫哪個 name(比方說你在你自定義的 namespace 內定義 `operator>>`
	* 或者 user code 永遠只會想要用自己定義的 name，不是 lib 定義的
* 現在我們來看 `std::move` 跟 `std::forward`，*他們都是 function templates*，而且吃一個 rvalue ref；
* 16.2.6 p.690 說過，這樣的 template 可以 match 任何 type
* **如果我們的 user code 也定義了 `move` 或 `forward`，則不管 parameter type 是什麼，都會跟 `std::move/forward` 撞到
* 所以這種單純吃一個 rv ref 的 function 因為很容易撞到，最好不要用 using declaration
	* 而且他們的使用情況通常也都很特別，user code 通常不會想要覆蓋掉他們的功能
* 所以很久以前才會說 `move`/`forward` 不要用 using declaration，原因就是這個
#### Friend Declarations and Argument-Dependent Lookup
* 記得在 class 內做 friend declaration 時，主要是在指定 friend 有對 class private member 的存取權限而已，並不是一般的宣告(7.2.1 p.270)
* **但是在 class 內宣告的 friend，會被認為是包含這個 class 的最近的 namespace 的 member**
* 這個規則跟 ADL 結合起來會有意想不到的結果:
	```c++
	namespace A { 
		class C { 
			// two friends; neither is declared apart from a friend declaration 
			// these functions implicitly are members of namespace A 
			friend void f2(); // won’t be found, unless otherwise declared 
			friend void f(const C&); // found by argument-dependent lookup
		};
	}
	```
	* `f` 跟 `f2` 都是 `A` 的 member
	* 如果這樣寫:
	```c++
	int main() {
		A::C cobj; 
		f(cobj); // ok: finds A::f through the friend declaration in  A::C 
		f2(); // error: A::f2not declared
	}
	```
	* 在呼叫 `f` 的情況下，如果沒有額外宣告 `f`，我們就可以透過 ADL 找到 `C::f`
	* 但是呼叫 `f2` 就會噴 error，因為他沒有 parameter XD

### 18.2.4 Overloading and Namespaces
* namespace 當然會影響 function matching，影響方法就是使用了 using declaration/directives，這樣會增加 function 到 candidate set
* 另一個就是 ADL
#### Argument-Dependent Lookup and Overloading
* 之前已經講了，ADL 會讓 name lookup 時，讓 function call 給的 class type 的參數所定義的 namespace 也被考慮進去
	* 這個 rule 也會影響到 name lookup 所看到的 candidate set
* 總的來說，候選 functions 有:
	* 所有 class type 參數定義的 namespaces 內的同名 functions
	* 這些參數的 base class 定義的 namespaces 內的同名 functions...
* 看以下例子，**就算 base class 這時根本不可見，可是 base class 所在的 namespace 還是會被搜尋，超ㄎㄧㄤ**(不過我認為這應該100%是糞code啦):
	```c++
	namespace NS {
		class Quote { /* .. . */}; 
		void display(const Quote&) { /* ... */}
	}
	// Bulk_item’s base class is declared in namespace NS 
	class Bulk_item : public NS::Quote { /* .. . */}; 
	int main() { 
		Bulk_item book1; 
		display(book1); 
		return 0;
	}
	```
	* 在 `main` 裡面根本看不到 `NS::Quote`，但是 ADL 卻可以讓他找到 `NS::display`....

#### Overloading and `using` Declarations
* 首先要注意，**`using` 是宣告一個*名字*，不是一個 function**:
	```c++
	using NS::print(int); // error: cannot specify a parameter list 
	using NS::print; // ok: using declarations specify names only
	```
* 如果用 `using` declaration 引入 name 是 function，那所有版本的 functions 都會被引入到當前 scope
	* 這是一個合理的設計原則，因為這些同名的 functions 都構成一個 interface，interface 作者會想要 overload 肯定是有他的理由，所以要馬就是把所有版本都引入，要馬就都不要；如果可以讓 user code 自行選用所有版本內的特定版本 function，這樣說不定會有作者想不到的非預期行為，例如 function matching 時有些該 match 到的版本因為沒有引入結果就沒 match 到之類的

* **`using` declaration 最重要的意義就是增加當前 scope 的 candidate sets**
	* 它一樣有 hide outer scope name 的功能
	* 如果引入的 overload instance 內有跟當前 scope 的 instance 一樣的 function parameter list，則噴 error

#### Overloading and using Directives
* 上面是講 `using` declaration 對 overload 的影響，這裡則是講 `using` directives 對 overload 的影響
* 總之就是會把被引入的命名空間的 names 全部加到當前 scope，該 overload 的就 overload
	* 不過就如同很有以前前面說的，用 `using` directive 的話，一般的 name collision 只有在真的使用到這個 name 時才會噴 error，function parameter list 的 collision 也是，只有在真的使用到這個 function 時才會噴 ambiguous error，很靠北很機掰
* 看例子...:
	```c++
	namespace libs_R_us { 
		extern void print(int); 
		extern void print(double);
	}
	// ordinary declaration 
	void print(const std::string &);
	// this using directive adds names to the candidate set for calls to print: 
	using namespace libs_R_us; 
	// the candidates for calls to print at this point in the program are: 
	// print(int) from libs_R_us 
	// print(double) from libs_R_us 
	// print(const std::string &) declared explicitly 
	void fooBar(int ival) {
		print("Value: "); // calls global print(const string &) 
		print(ival); // calls libs_R_us::print(int)
	}
	```
#### Overloading across Multiple `using` Directives
* 如果用了多次 `using` directives，則每個被引入的命名空間內的同名 functions 都會被加入 candidates functions
	```c++
	namespace AW { int print(int);
	}
		namespace Primer { 
			double print(double);
		}
	// using directives create an overload set off unctions from different namespaces 
	using namespace AW; 
	using namespace Primer; // 靠，這行有夠雞巴，是 depends on 上一行，導致看的到 Primer 之後才可以 using namespace Primer 的
	long double print(long double); 
	int main() { 
		print(1); // calls AW::print(int) 
		print(3.1); // calls Primer::print(double) 
		return 0;
	}
	```
	* 總之按照上面的 `using` directive 寫之後，global scope 就有三個 `print` 版本當作 candidates function 了
	* 再強調一次這樣亂 `using` 是糞 code

## 18.3 Multiple and Virtual Inheritance

