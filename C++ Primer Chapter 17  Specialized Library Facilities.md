# C++ Primer Chapter 17 Specialized Library Facilities
* 介紹 `tuple`s,` bitset`s, random-number generation, 還有 regular expressions.
* 再來是 IO library revisit，介紹一些更奇葩的功能
* C++ library 的部分還有增加很多東西，不可能全部介紹，這裡只介紹夠 general 的功能

## 17.1 The `tuple` Type
* 很像 `pair` 的東西，也是 `template`
	* `pair` 只能有兩個 members，但 members 的 type 可以不一樣
	* 一個 `tuple` 可以有任意個 members，不過這個數量是 `tuple` 型態的一部分；members 的型態可以不同
* `tuple` 最有用的地方是，你想要把一堆 data 綁到一個 object 上，可是又不想為這個 object 定義一個 data structure。
	* Note: A tuple can be thought of as a "quick and dirty" data structure.
	* 笑死 LOL
* table 17.1 列出 `tuple` 的 API；tuple 定義在 `<tuple>` header 內
* ![](https://i.imgur.com/NZVrOGD.png)
### 17.1.1 Defining and Initializing `tuple`s
* 看例子比較快...:
	```c++
	tuple<size_t, size_t, size_t> threeD; // all three members set to 0
	tuple<string, vector<double>, int, list<int>>
		 someVal("constants", {3.14, 2.718}, 42, {0,1,2,3,4,5});
	```
	* `tuple` 的 default ctor 會值初始化 members
* 如果要餵 initializer 給 member，記得 ctor 是 `explicit`:
	```c++
	tuple<size_t, size_t, size_t> threeD = {1,2,3}; // error
	tuple<size_t, size_t, size_t> threeD{1,2,3}; // ok
	```
* (如果你還記得，)有 `make_pair`，當然也有 `make_tuple`:
	```c++
	// tuple that represents a bookstore transaction: ISBN, count, price per book
	auto item = make_tuple("0-999-78345-X", 3, 20.00);
	```
	* 反正配合 `auto` + `make_tuple` 你就不用打超級長的型別惹
	* 上面 `item` 的型別是 `tuple<const char*, int, double>`

#### Accessing the Members of a `tuple`
* `tuple` 不像 `pair` 永遠只有兩個 members，所以不可能用 `pair` 那套 `first`, `second` 的方法給你 access members
* STL 用 `get` 這個 template 讓你存取 members，template parameter 是 size_t，也就是要存取第幾個 member
* 看例子...:
	```c++
	auto book = get<0>(item); // returns the first member of item
	auto cnt = get<1>(item); // returns the second member of item
	auto price = get<2>(item)/cnt; // returns the last member of item
	get<2>(item) *= 0.8; // apply 20% discount
	```
	* 餵給 `get` 的參數一定要是 integral constant expression
* 如果你想要知道一個 `tuple` 的確切型別或者個別 members 的型別可以用以下 templates:
	```c++
	typedef decltype(item) trans; // trans is the type of item
	// returns the number of members in object’s of type trans
	size_t sz = tuple_size<trans>::value; // returns 3
	// cnt has the same type as the second member in item
	tuple_element<1, trans>::type cnt = get<1>(item); // cnt is an int
	```
	* 覺得這 API 實在有點鳥...
	* `tuple_size` 有一個 `public` data member `value`，會回傳 member 的數量，是 constant expression
	* `tuple_element` 則是吃一個 index 跟 `tuple` type，他有一個 type member `type`，型別就是吃個 `tuple` type 對應 index 的型別
		* 有夠繞口令但很好懂?

#### Relational and Equality Operators
* 要能用 relational operators，lhs 跟 rhs 一定要有相同數量的 members
* 然後對應的 member 必須要存在合法的 relational operators
* 看例子...:
	```c++
	tuple<string, string> duo("1", "2");
	tuple<size_t, size_t> twoD(1, 2);
	bool b = (duo == twoD); // error: can’t compare a size_t and a string
	tuple<size_t, size_t, size_t> threeD(1, 2, 3);
	b = (twoD < threeD); // error: differing number of members
	tuple<size_t, size_t> origin(0, 0);
	b = (origin < twoD); // ok: b is true
	```
* Note: 因為 `tuple` 定義了 `<` 跟 `==`，我們可以把 sequence of `tuple`s 丟給 STL algo 用，也可以把 `tuple` 當成 associative container 的 key type

### 17.1.2 Using a `tuple` to Return Multiple Values
* 這是一種 common use
* 這裡舉一個 usecase，在 p.721 頁，沿用之前寫的 `Sales_data`:
	* 假設現在不只一家書店，而是一堆連鎖書店；
	* 其中每一家店都會有一個 transaction file，顧名思義，裡面記錄了這家店的所有交易紀錄
		* 其中同一本書的所有交易紀錄會被放在一起(grouped together)
	* 再來我們假設已經存在一個 function，他可以幫我們把 transaction file 轉換成 `vector<Sales_data>` ，也就是說一家店的所有交易紀錄都存在這個物件內
		* 再假設所有書店的 transaction file 就會被轉換成一個`vector<vector<Sales_data>>`，就叫他 `files`
	* 我們真正要寫的 function 的功能是，給定一本書，然後 function 會搜尋 `files`，把這本書在每一家店的交易紀錄撈出來
		* 如果某一間店裡有這本書的交易紀錄，我們就創造一個 `tuple`，他有三個 member，第一個是這家店的 index，剩下兩個是這本書的交易紀錄的頭尾 iterators
			* 記得同本書的交易紀錄會 grouped together，所以只要用兩個 iterators 就可以表示這段區間了
			* 講的具體一點，`files` 的第 i 個 element 就代表第幾家店，如果這家店有對應的交易紀錄，就用兩個 iterators 把它標示出來，iterator type 就會是 `vector<Sales_data>::iterator`
#### A Function That Returns a (vector of)`tuple`s
* 這 function 就吃 `files` 以及要找的書，然後如果某書店有這筆書的交易紀錄，就為它創造對應的 `tuple`，塞到一個暫時的 vector 內；整個 `files` 掃完後，就 return vector:
	```c++
	// matches has three members: an index of a store and iterators into that store’s vector
	typedef tuple<vector<Sales_data>::size_type,
			      vector<Sales_data>::const_iterator,
			      vector<Sales_data>::const_iterator> matches;
	// files holds the transactions for every store
	// findBook returns a vector with an entry for each store that sold the given book
	vector<matches>
	findBook(const vector<vector<Sales_data>> &files,
		     const string &book)
	{
		vector<matches> ret; // initially empty
		// for each store find the range of matching books, if any
		for (auto it = files.cbegin(); it != files.cend(); ++it) {
			// find the range of Sales_data that have the same ISBN
			auto found = equal_range(it->cbegin(), it->cend(),
									 book, compareIsbn);
			if (found.first != found.second) // this store had sales
				// remember the index of this store and the matching range
				ret.push_back(make_tuple(it - files.cbegin(),
								found.first, found.second));
		}
		return ret; // empty if no matches found
	}
	```
	* 首先先把那噁心的 `tuple` 型別 alias 成 `matches`
	* 再來是我們用了 `std::equal_range`，他吃一個區間，還有要找的 value，跟一個 call object；
		* 回顧一下，因為 `equal_range` 預設是用 `operator<`，而 `Sales_data` 沒有，所以用 `compareIsbn` 代替
		* `equal_range` 回傳的是一個 pair of iterators，代表擁有這個 value 的區間
			* 如果沒找到有這個值的區間，則 `first == second`
#### Using a `tuple` Returned by a Function
* 接下來就只是把找到的 `tuple`s 一個一個印出來:
	```c++
	void reportResults(istream &in, ostream &os,
			           const vector<vector<Sales_data>> &files)
	{
		string s; // book to look for
		while (in >> s) {
			auto trans = findBook(files, s); // stores that sold this book
			if (trans.empty()) {
				cout << s << " not found in any stores" << endl;
				continue; // get the next book to look for
		}
		for (const auto &store : trans) // for every store with a sale
			// get<n> returns the specified member from the tuple in store
			os << "store " << get<0>(store) << " sales: "
			   << accumulate(get<1>(store), get<2>(store),
							 Sales_data(s))
			   << endl; }
	}
	```
	* 先確認有沒有此書的交易紀錄，如果沒有就印 not found message，並且搜尋下一本書
	* 如果有就一筆一筆印出來
	* 把 `get<1>(store), get<2>(store)` 傳給 `std::accumulate` 就把這本書在這間 store 的所有交易都加起來
		* 因為 `Sales_data` 有定義 `operator+` 才可以直接用  `std::accumulate`
		* 然後餵給 `std::accumulate` 拿來加的起始 value 是直接用書本名稱初始化一個 `Sales_data`；記得這樣初始化，物件內所表示的 price 等等的會是 0
	* 注意一些小細節，例如哪裡該用 `const`/ref 之類的

## 17.2 The `bitset` Type
* 4.8 介紹過 bitwise operator，把 integral type 當成一堆 bits 來操作
* C++ 標準還有定義 `bitset` 這個 class template，可以讓 bitwise 操作更容易，而且可以讓被操作的 bits 數量超過最大 integral type 的數量
### 17.2.1 Defining and Initializing `bitset`s
* table 17.2 是初始化 `bitset` 的方法總結:
* ![](https://i.imgur.com/rDOlBf9.png)
* 首先他跟 `array` 一樣，bits 的數量是固定的，定義 `bitset` 物件時要指定 bit 數量；數量為型別的一部份:
	```c++
	bitset<32> bitvec(1U); // 32bits; low-order bit is 1, remaining bits are 0
	```
	* 給定的數量當然要是 constant expression
	* 上面的 code 定義了一個可以塞 32 個 bit 的 `bitset`
* `bitset` 的好處之一就是可以用 `operator[]` 來取的對應位置的 bit
	* 注意從第 0 個 bit 來看，bit 是 **low order bit**，從最高位 bit 來看就是 **high order bit**
#### Initializing a `bitset` from an unsigned Value
* 當你丟一個 integral value 當作 initializer 時，**這個 integral value 會先被轉成 `unsigned long long`** 然後被當作是一個 bit pattern 來看待；
	* 之後在被初始化的 `bitset` 內存的 bit pattern 就是這個 `unsigned long long` 的 bit pattern
* 如果 `bitset` 長度大於 `unsigned long long` 的長度，則 high order bits 會補 0
* 如果長度小於 `unsigned long long` 的長度，則這個 ull 的 high order bits 會被無視，`bitset` 的 bit pattern 只會把 ull 的第 0 到 第 `bitset` 的 `size-1` 個 bits 存起來

* 看例子...:
	```c++
	// bitvec1 is smaller than the initializer; high-order bits from the initializer are discarded
	bitset<13> bitvec1(0xbeef); // bits are 1111011101111
	// bitvec2 is larger than the initializer; high-order bits in bitvec2 are set to zero
	bitset<20> bitvec2(0xbeef); // bits are 00001011111011101111
	// on machines with 64-bit longlong 0ULL is 64 bits of 0, so ~0ULL is 64 ones
	bitset<128> bitvec3(~0ULL); // bits 0... 63 are one; 63. ..127 are zero
	```
#### Initializing a bitset from a string
* 我們也可用 `std::string` 或 C string 來
* 看例子:
	```c++
	bitset<32> bitvec4("1100"); // bits 2 and 3 are 1, all others are 0
	```
	* 注意這個例子，bit 3 跟 2 是 `1`，bit 1 跟 0 是 `0`，總之就跟人類的表示方式一樣...
* 如果 string 的長度低於 `bitset` 的 bit 數量，則 `bitset` 的 high order bits 也會補 0
* 當然除了用整個 string 來初始化，`bitset` 的 API 也跟 `string` 自己的一樣有彈性，看 code...:
	```c++
	string str("1111111000000011001101");
	bitset<32> bitvec5(str, 5, 4); // four bits starting at str[5], 1100
	bitset<32> bitvec6(str, str.size()-4); // use last four characters
	```
	* 這裡有幾點要注意，首先因為要塞入的 bit 數量都小於 `bitset` 的 bit 數量，所以會補 0；但到底是 LSB 還是 MSB 會補靈? 答案是 MSB，這也符合人類直覺... 如果你看不懂，看下面這張圖:
		* ![](https://i.imgur.com/V7WFSHq.png)
### 17.2.2 Operations on `bitset`s
* 一樣來個 table，對 `bitset` 可以做的 operations 做總結:
	* ![](https://i.imgur.com/LRiY541.png)
	* 主要就是 4.8 講的 bitwise operator，以及一些特定的 API 跟 IO
	* 注意 `operator[]` 回傳的 type 會根據 `bitset` 物件本身是否為 `const` 而 return value/reference
	* **還有他的 reference 跟一般的 reference 有差異，但是功能概念上是一樣的，就是可以改對應 bit 的值**
		* 這在 `vector<bool>` 也有類似情形，詳情可以去查 `vector<bool>` 跟其他型別的 vector 有什麼區別

* `count`, `size`, `all`, `any`, 還有 `none` 都只是屬於回傳 `bitset` 狀態的 API，並不會改變 `bitset` 的內容
* `set`, `reset`, 還有 `flip` 就會改變 `bitset` 的狀態
	* 改變 `bitset` 狀態的 API 有 overload: 如果是有接收「位置參數」的版本就是改變參數對應位置的 bit，沒有的話就是對整個 `bitset` 做操作
* 看例子zzz:
	```c++
	bitset<32> bitvec(1U); // 32bits; low-order bit is 1, remaining bits are 0
	bool is_set = bitvec.any(); // true, one bit is set
	bool is_not_set = bitvec.none(); // false, one bit is set
	bool all_set = bitvec.all(); // false, only one bit is set
	size_t onBits = bitvec.count(); // returns 1
	size_t sz = bitvec.size(); // returns 32
	bitvec.flip(); // reverses the value of all the bits in bitvec
	bitvec.reset(); // sets all the bits to 0
	bitvec.set(); // sets all the bits to 1
	```
	* 注意，`all` 這個實用的 API 是 C++11 才有的 LOL
	* 然後 `size` 是 constant expression

* 再多看一點例子...:
	```c++
	bitvec.flip(0); // reverses the value of the first bit
	bitvec.set(bitvec.size() - 1); // turns on the last bit
	bitvec.set(0, 0); // turns off the first bit
	bitvec.reset(i); // turns off the ith bit
	bitvec.test(0); // returns false because the first bit is off
	```
* 再來介紹 `operator[]`，他有做 `const` overloading:
	* `const` 版本會回傳 value，如果給定的位置的 bit 是 `true`，false otherwise
	* non`const` 版本則是會回傳一種特殊的 reference type，認讓我們可以藉由這個 reference 來操作對應位置的 bit
	* 看例子:
	```c++
	bitvec[0] = 0; // turn off the bit at position 0
	bitvec[31] = bitvec[0]; // set the last bit to the same value as the first bit
	bitvec[0].flip(); // flip the value of the bit at position 0
	~bitvec[0]; // equivalent operation; flips the bit at position 0
	// 注意上面這行 code，看起來沒有 assign，實際上會更改 bit!!! API有點鳥
	bool b = bitvec[0]; // convert the value of bitvec[0] to bool
	```
#### Retrieving the Value of a `bitset`
* `to_ulong` 和 `to_ullong` 會回傳對應的 integral value，其 bit pattern 跟 `bitset` 內的一樣
	* 注意只有當你 `bitset` 的 size 小於這兩種 integral typ(`unsigned long` 或 `unsigned long long`) 的 bit 長度時才可使用:
	```c++
	unsigned long ulong = bitvec3.to_ulong();
	cout << "ulong = " << ulong << endl;
	```
	* 如果 `bitset` 長度超過這兩個對應的 member functions 回傳的型別，就會 `throw` `overflow_error`
#### `bitset` IO Operators
* input operator 會先將 input characters 讀到一個 temp `std::string` 中，直到下列幾種情況時會停止:
	* 讀了跟 `bitset` size 一樣多的字元
	* 遇到不是 `0` `1` 的字元
	* `EOF` 或者 input error
* 然後 `bitset` 就會用這個 temp `string` 來建構 bit pattern
	* 一樣，如果 `string` 比 `bitset` size 短，那就在 `bitset` high order bits 補 0
		* 然後不會發生比較長的問題，因為這邊的 `string` 最多跟 `bitset` 一樣長
* output operator 就把 bit pattern 印粗乃
* 看 code:
	```c++
	bitset<16> bits;
	cin >> bits; // read up to 16 1 or 0 characters from cin
	cout << "bits: " << bits << endl; // print what we just read
	```

#### Using `bitset`s
* 直接重新實作 4.8 的 grading code lol，如果你還記得那是什麼鬼的話:
	* 簡單說就是紀錄 30 個學生的 pass/fail 狀態，然後這裡提供 4.8 的 code 跟用 `bitset` 實作的 code:
	```c++
	bool status; // version using bitwise operators
	unsigned long quizA = 0; // this value is used as a collection of bits
	quizA |= 1UL << 27; // indicate student number 27 passed
	status = quizA & (1UL << 27); // check how student number 27 did
	quizA &= ~(1UL << 27); // student number 27 failed
	// equivalent actions using the bitset library
	bitset<30> quizB; // allocate one bit per student; all bits initialized to 0
	quizB.set(27); // indicate student number 27 passed
	status = quizB[27]; // check how student number 27 did
	quizB.reset(27); // student number 27 failed
	```
	* 好吧看起來真的比較好懂...
