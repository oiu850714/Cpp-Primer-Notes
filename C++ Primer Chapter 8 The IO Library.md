# C++ Primer Chapter 8 The IO Library

* 十幾頁而已，爽
* 在 C++，IO 操作是以 library 的方式實現的，而不是用 syntax 的方法去操作 IO
* 這章先講基本的 IO 操作，14 17 章會有更深入的介紹

## 8.1 The IO Classes
* 到目前我們使用的 IO object 都是在操作 char data
    * 把 char data 轉成 arithmetic 或 string(>>)
    * 把 arithmetic 或 string 轉成 char data(<<)
* 這些 IO object 預設綁在 stdout, stdin, stderr
* 當然，我們還需要其他形式的 IO，例如讀寫 named files(檔案)，網路之類的
* 除了操作 char data，還會有操作 w_char 的可能
* Standard 為此定義了除了 istream 跟 ostream 之外的一些 type 來讓我們達成上述操作，分別在下述 header 內:
1. iostream
    * istream, wistream reads from a stream 
    * ostream, wostream writes to a stream
    * iostream, wiostream reads and writes a stream 
2. fstream
    * ifstream, wifstream reads from a file 
    * ofstream, wofstream writes to a file
    * fstream, wfstream reads and writes a file
3. sstream
    * istringstream, wistringstream reads from a string 
    * ostringstream, wostringstream writes to a string
    * stringstream, wstringstream reads and writes a string

* w 開頭的是拿來操作 w_char 用的
* 其實有 wcin, wcout, wcerr 這種東西

#### Relationships among the IO Types
* 概念上，我現在要對某個裝置或檔案，或者 string 做 IO 操作，我都希望我使用的操作方式可以不用去理會操作的對象；亦即我可以用一模一樣的寫法來操作這些東西
* C++ **利用繼承來達成**，繼承之後章節會講
* 如同 template，我們一樣可以不用很了解繼承在幹嘛，可是一樣可以使用繼承的好處
* 簡單說，一個 class B 繼承 class A，我們說 class B **也是** class A，**class B 可以用 class A 的方式操作**
    * ifstream 跟 istringstream 繼承 istream
    * ofstream 跟 ostringstream 繼承 ostream
    * fstream 跟 stringstream 繼承 iostream
* 所以 ifstream 跟 istringstream 的物件就可以用 >> 的方式操作(記得 cin 是 ifstream 物件)
* << 同理

* Note: Everything that we cover in the remainder of this section applies equally to plain streams, file streams, and string streams and to the char or wide-character stream versions.
    * 一種寫法，多種適用，爽ㄛ

### 8.1.1 No Copy or Assign for IO Objects
* 之前說過， IO object 不能 copy(雖然我還是不知道怎樣弄才會不能 copy lol)
    ```C++
    ofstream out1, out2;
    out1 = out2; // error: cannot assign stream objects
    ofstream print(ofstream); // error: can’t initialize the 
    ofstream parameter out2 = print(out2); // error: cannot copy stream objects
    ```

* 因為我們不能 copy IO objects，所以不能宣告 function that 接收或回傳 plain IO objects
    * 真的要傳入或回傳 IO object 要用 reference
    * Reading or writing an IO object changes its state, so the reference must not be const.
    * 而且你只要 read 或 write IO object 就會改變 IO object 的 state，所以不能宣告成 reference to const

### 8.1.2 Condition States
* Inherent in doing IO is the fact that errors can occur.
* 其實你可以利用讀取 IO 來練習 error handling 的 code，因為 IO 噴 runtime error 是常態
    * Some errors are recoverable; others occur deep within the system and are beyond the scope of a program to correct.
* **The IO classes define functions and flags, listed in Table 8.2, that let us access and manipulate the condition state of a stream.**
    * IO class 定義了一系列的 interface 可以讓我們看 IO object 對應的 stream 的狀態

* 下面的例子
    ```C++
    int ival; cin >> ival;
    ```
* 如果這時鍵盤打 Boo 再 enter，那 cin >> ival 會 fail
* **As a result, cin will be put in an error state.**
* 相似的，如果這時候打 Eof 對應的 key，那 cin 一樣也會變成 error state
* ***只要 IO object 變成 error state，之後對它做的讀寫全部都會 fail***
* Because a streammight be in an error state, code ordinarily should check whether a stream is okay before attempting to use it.
    * 其實 while(cin >> i) 就是一種檢查惹

#### Interrogating the State of a Stream
* 只用上面 condition 的方式檢查 stream 只能看到 stream 是否 valid，我們常常需要更進一步了解 stream 處於什麼 state，**如果 invalid，為什麼會 invalid**
    * For example, what we do after hitting end-of-file is likely to differ from what we’d do if we encounter an error on the IO device.
    * The IO library **defines a machine-dependent integral type named *iostate*** that it uses to c**onvey information about the state of a stream.**

* iostate, a collection of bits
* The IO classes *define four constexpr values (§ 2.4.4, p. 65) of type iostate that represent particular bit patterns.*
* These values are **used to indicate particular kinds of IO conditions**. They can be **used with the bitwise operators (§ 4.8, p. 152) to test or set multiple flags** in one operation.

* library 特別定義了四個 iostate 物件會有的 constexpr 的 value，分別叫做 badbit, failbit, eofbit, goodbit，可以更詳細地用來確認 stream 的狀態
    * batbit 代表 stream 發生了不可復原的悲劇，可能是系統等級的錯誤，例如 uncoverable 的 read/write error
        * 通常 badbit 一旦發生就不可能再使用這個 stream 了
    * failbit 在那種可復原的錯誤發生之後會被 set，例如要吃一個數字結果 input 卻是字串
        * 通常可以把問題解決然後重新使用對應的 stream
    * Reaching end-of-file **sets both eofbit and failbit**
    * goodbit 標準保證它的 value 是 ，並且 stream 是在 goodbit 時，stream 沒有 error
* **If any of badbit, failbit,or eofbit are set, then a condition that evaluates that stream will fail.**

* Library 除了定義這些 constexpr value 之外，還有定義一系列的 member function 來得知 stream 的狀態
    * good() return true 如果 stream 正長
    * bad(), fail(), 跟 eof() return true 如果對應的 iostate value 發生的話
        * 此外，如果 bad 是 true，那 fail 也會是 true

* By implication, the right way to **determine the overall state** of a stream is to use either good or fail.
* 其實把一個 stream 當 condition 就如同寫成 !stream_obj.fail() 一樣
* 如果真的 fail 了再去檢查到底是因為 eof 還是普通 fail 還是更慘，bad
    * fail 在 failbit 或 badbit set 是都會 return true；奇怪，阿不是說 EOF 時 fail 也會 true? 可是 fail 沒看 eofbit 啊?
    * 那是因為 EOF 是會把 eofbit **跟 failbit** 都設起來!

#### Managing the Condition State
* rdstate 回傳 stream 當前 state 對應的 iostate value
* setstate 有點像 bitwise OR，把給定的(多個) state(s) 打開
* clear 有兩種，一種是不給參數，把 state reset 成 good，第二種是吃一個參數，把 state 設成跟參數一樣的 value
    ```C++
    // remember the current state ofcin
    auto old_state = cin.rdstate();
    // remember the current state of
    cin
    cin.clear(); // make cin valid
    process_input(cin); // use cin    
    cin.setstate(old_state); // now reset cin to its old state
    ```
    * 上面又在一次 demo 用 auto 的好處，你可以不用記得標準定義的物件或是 value 是什麼型態
    * 不過我對上面的 code 一點感覺都沒有.. 如果 old_state 已經 fail，不管它 fail 直接進行 input 操作不會出事ㄇ

* 你還可以用 bitwise operator 來設定你想要的特定的 state
    ```C++
    // turns offfailbitand badbit but all other bits unchanged
    cin.clear(cin.rdstate() & ~cin.failbit & ~cin.badbit);
    ```
    * 上面的 code 只把 failbit 跟 badbit 關掉，但是不會動到 eofbit 原本的狀態


### 8.1.3 Managing the Output Buffer
* 挺重要ㄉ，尤其是 debug 的時候
* **Each output stream manages a buffer**, which it uses to hold the data that the program reads and writes.
* 對 buffer 不熟自己去看
* 總之把資料留在 buffer 可以讓多個小的 user level IO operations 變成一個大的 system level IO operation，可以增加效能
    * 不過這只是欲設行為，這個小節會說怎麼改變這個預設行為

* There are several conditions that cause the buffer to be flushed—**that is, to be written—*to the actual output device* or file:**
    * 程式正常關閉時，也就是 return from main 時
    * buffer 滿了的時候
    * 把 endl 這種 IO manipulator 丟給 cout 時
    * We can use the **unitbuf** *manipulator* to set the stream’s internal state to empty the buffer after each output operation. By default, unitbuf is set for cerr,so that writes to cerr are flushed immediately.
    * An output stream might be **tied to** another stream.
        * In this case, the output stream is flushed whenever the stream to which it is tied is read or written.
        * 一個 output stream 可以綁住(tie) 另一個 stream，當這個 stream 有做讀寫時，output stream 就會 flush
        * 預設 cout 被 tie 在 cin 跟 cerr 上，所以如果 cin 有讀取或者 cerr 有寫入那 cout 就會 flush

#### Flushing the Output Buffer
* 目前只用過 endl 這個 manipulator
* 還有另外兩個，flush 跟 ends
    * flush 不會輸出字元到 stream，就只是單純說要 flush
    * ends 會塞一個 null character 到 stream 然後 flush

#### The unitbuf Manipulator
* 我們想要從某個時間點開始都不要把資料 buf 住的話，可以用 unitbuf manipulator
    * ostream_obj << unitbuf;
* nounitbuf 會把 stream 的 buffer 機制重設成預設方式

* CAUTION: BUFFERS ARE NOT FLUSHED IFTHE PROGRAM CRASHES
    * 如果你的程式不正常結束，那些還在 buffer 的資料是不會被 flush 的!
    * **當你在 debug 的時候記得把 buffer 調整成會立即 flush，這樣才不會有以為程式沒正常執行殊不知只是資料待在 buffer 沒印出來的情況**

#### Tying Input and Output Streams Together
* 幹我看不懂 Primer 說的 "tie" 這個動詞怎麼用.. 到底誰 tie 誰==
* 如果你把一個 input stream tie 到一個 output stream，每次你讀東西到 input stream，**C++ 會把跟 output stream 有關聯的 stream 都 flush**
    * 請注意看我到底是寫誰會被 flush
    * 預設 cin 跟 cout 綁在一起
    * 所以類似 cin >> ival; 的操作就會 flush 跟 cout tie 在一起的物件

* Note: Interactive systems usually should tie their input stream to their output stream. *Doing so means that all output, which might include prompts to the user, will be written before attempting to read the input.*

* tie 有兩種 overloaded member functions
    * 一個不吃參數，回傳 pointer to output stream，指向目前 tie 住的 stream(如果有的話)，否則 return nullptr
    * 第二個是給一個 pointer to ostream 參數，然後**把自己 tie 到那個 ostream**；一樣回傳舊的 tie 著的 stream，如果有的話
        * That is, x.tie(&o) ties the streamx to the output stream o.
* 我們要把 i 或 ostream tie 到 ostream 都可以

```C++
cin.tie(&cout); // illustration only: the library ties cin and cout for us
// old_tie points to the stream (if any) currently tied to cin
ostream *old_tie = cin.tie(nullptr); // cin is no longer tied
// ties cin and cerr; not a good idea because cin should be tied to cout
cin.tie(&cerr); // reading cin flushes cerr, not cout
cin.tie(old_tie); // reestablish normal tie between cinand cout
```
* 如果要把一個 stream tie 到一個新的 ostream，我們就要 call stream_obj.tie(&ostream_obj);
* 如果要把某個 stream 給解 tie，我們就要傳入 nullptr
    * stream_obj.tie(nullptr);
* 還記得競程超愛寫的 cin.tie(0); 嗎? 0 等價 nullptr(或 NULL)，所以就是把 cin 解 tie 的意思!

* 一個 stream 同時最多只能 tie 一個 ostream
* 但是可以同時有多個 stream tie 住一個 ostream

## 8.2 File Input and Output
* ifstream: read from file
* ostream: write to file
* fstream: both

* 17 章才會講怎麼同時讀寫一個 file..
* 再提醒一次，你可以用操作 cin cout 的方式來操作 fstream，讚
* 實際上就是，整個 code 有用到 iostream 的地方你都可以用 fstream 物件來代替，這94繼承的威力
* 還有一些 fstream 獨有的 member function 可以用
    * ![](https://i.imgur.com/ghFNkGU.png)

### 8.2.1 Using File Stream Objects
* We define a file stream object and associate that object with the file.
* Each file stream class defines a member function named open *that does whatever system-specific operations are required to locate the given file* and open it for reading or writing as appropriate
* 可以宣告 object 時就給 filename，這樣 open 會自動被 call
* 或者 default init，之後再用 open member function
* C++11 之前 filename 只能是 C string，超ㄎㄧㄤ，現在可以給 string

#### Using an fstream in Place of an iostream&
* 之前講過，因為繼承的特性，我們可以把 fstream 物件放到需要 iostream 的地方
* 還記得很久以前為了 Sales_data 寫過 print 跟 read 嗎? 我們這時候可以傳 fstream 給他們，這樣就可以從檔案讀寫啦!
* 下面的 code 假設 input 跟 output file 當成 command line arg 傳進來
    ```C++
    ifstream input(argv[1]); // open the file of sales transactions 
    ofstream output(argv[2]); // open the output file
    Sales_data total;           // variable to hold running sum
    if(read(input, total)){
        Sales_data trans;    // variable to hold next transaction
        while(read(input, trans)) {
            if(total.isbn() == trans.isbn())
                total.combine(trans);
            else {
                print(output, total) << endl;
                total = trans;
            }
        }
        print(output, total) << endl;
    }
    else
        cerr << "No data?!" << endl;
* Aside from using named files, this code is nearly identical to the version of the addition program on page 255.

#### The open and close Members
* When we define an empty file stream object(沒給 string 參數的), we can subsequently associate that object with a file by calling open:
    ```C++
    ifstream in(ifile);
    ofstream out;
    out.open(ifile + ".copy");

* 別忘了 fstream 是繼承 iostream 的，所以那些檢查 stream 狀態的功能都可以用
* 例如如果檔案開啟失敗，failbit 會被 set
* 挖操，感覺比 FILE* == NULL 好用(ry
    ```C++
    if (out)
        ;   // check that the opensucceeded // the opensucceeded, so we can use the file
    ```
* 直接這樣的 code 就可以確認 fstream 的狀態，讚

* Once a file stream has been opened, *it remains associated with the specified file.*
* Indeed, calling open on a file stream that is already open will fail and set failbit.
* 你要開新檔案，必須先 call close() 這個 member function

#### Automatic Construction and Destruction
* 下面的 code 是一個 fstream 可能的使用場景
```C++
// for each file passed to the program
for (auto p = argv + 1; p != argv + argc; ++p) { 
    ifstream input(*p); // create input and open the file 
    if (input) {
        process(input); } 
    else
        cerr << "couldn’t open: " + string(*p);
}// input goes out of scope and is destroyed on each iteration
```
* As usual, we check that the open succeeded.
* Because input is defined inside the block that forms the for body, **it is created and destroyed on each iteration (§ 6.1.1, p. 205).**
* When an fstream object **goes out of scope, the file it is bound to is *automatically closed*.**
    * 所以不會發生表面上看到的連續 open 多次
    * 還有這裡又再次強調，在 for 裡面宣告的變數，每次 iteration 會 in scope，然後又馬上被消滅，又 in scope，and so on

#### 8.2.2 File Modes
* Each stream has an associated file mode that represents how the file may be used.
* ![](https://i.imgur.com/wJQ3fWY.png)
* We can supply a file mode whenever we open a file—either when we call open or when we indirectly open the file when we initialize a stream from a file name.
* 這些 file mode 有一些限制
    * out 只能用在 ofstream 跟 fstream
    * in 只能用在 ifstream 跟 fstream
    * trunc 只有在 out 有給的時候才能給
    * app 在 trunc 給的時候不能給，亦即 app 跟 trunc 只能選一個
    * 預設行為，當檔案用 out mode 開啟時也會被 truncated 就算我們沒有指定 trunc
        * 要改變這個預設行為就要額外給 app，記得上面講的 app trunc 不能同時出現，所以給了 app trunc 就不會開啟；但是這樣我們就只能從檔案尾巴開始寫資料了
        * 或者同時給 in，不過同時給 in 跟 out 在 17章才會詳細講
    * ate 跟 binary 只能配其他的一起用

* 這些 stream type 都有預設的 file mode，每個有不同
* fstream 是 in|out
* ifstream 是 in
* ofstream 是 out


#### Opening a File in out Mode Discards Existing Data
```C++
// file1 is truncated in each of these cases 
ofstream out("file1"); // out and trunc are implicit ofstream 
out2("file1", ofstream::out); // trunc is implicit ofstream 
out3("file1", ofstream::out | ofstream::trunc);
// to preserve the file’s contents, we must explicitly specify app mode
ofstream app("file2", ofstream::app); // out is implicit
ofstream app2("file2", ofstream::out | ofstream::app);
```
* 注意上面那個 mode 只有給 app 的宣告還是合法的，因為 out 會隱性的給

#### File Mode *Is Determined Each Time* open Is Called

```C++
ofstream out; // no file mode is set
out.open("scratchpad"); // mode implicitly outand trunc
out.close(); // close outso we can use it for a different file 
out.open("precious", ofstream::app); // mode is outand app
out.close();
```
* 每次開檔案時的 file mode 不會繼承之前開檔案時的 file mode，每次 mode 都是獨立的

## 8.3 string Streams

* The sstream header defines three types to support in-memory IO; these types read from or write to a string **as if the string were an IO stream.**
* istringstream read from string
* ostringstream write to string
* stringstream both
* 繼承 iostream，所有 operation 都可以用
* In addition to the operations they inherit, **the types defined in sstream addmembers to manage the string associated with the stream.**
* 注意雖然 stringstream 跟 fstream 跟 iostream 有相同 interface，但他們也只有這層關聯而已，沒有其他共同的地方惹
    * 比方說 stringstream 沒有 open，fstream 沒有 str
* stringstream 獨有的 interface
    * ![](https://i.imgur.com/z9bnEpc.png)

### 8.3.1 Using an istringstream
* An istringstream is often used when we have some work to do on an entire line, and other work to do with individual words within a line.
    * 先用 getline 把一行資料讀到 string，再用 istringstream 讀 string

* 例子: 現在有一份檔案，每一行是一筆資料，每筆資料第一個字是人名，之後會有一筆或多筆字是電話號碼
    morgan 2015552368 8625550123 
    drew  9735550130
    lee 6095550132 2015550175 8005550000
* 我們用一個 struct 來存每一筆資料
    ```C++
    struct PersonInfo { 
        string name;
        vector<string> phones;
    };
    ```
* 可以這樣寫 code
    ```C++
    string line, word; // will hold a line and word from input, respectively
    vector<PersonInfo> people; // will hold all the records from the input
    // read the input a line at a time until cin hits end-of-file (or another error)
    while (getline(cin, line)) {
        PersonInfo info; // create an object to hold this record’s data
        istringstream record(line); // bind record to the line we just read     
        record >> info.name; // read the name     
        while (record >> word) // read the phone numbers
            info.phones.push_back(word); // and store them
        people.push_back(info); // append this record to people
    }
    ```
    
* 一種 practice 是，你的資料是以什麼樣的單位擺放的，最外層的迴圈就以這樣的單位來讀取檔案
    * 例如這邊是以行為單位擺放，那最外層就一次讀一行，內層迴圈再對這行慢慢處理
    * 不要發生那種明明是以行為單位，卻一次讀一個字的狀況，這樣你程式的"狀態機"會很難定義，而且很難懂
* 注意上面使用的一些專屬 stringstream 的 member function
    * 先創一個 istringstream，把 line 丟進去，這樣就可以用 >> 把 word 從 istringstream object 裡面讀出來
    * When the string has been completely read, **"end-of-file" is signaled** and the next input operation on record will fail.
    
### 8.3.2 Using ostringstreams
* An ostringstream is useful when we need to build up our output a little at a time but do not want to print the output until later.
* 例如上面 phone number 的例子，我們可能需要對 number 做一些格式上的驗證，然後再寫到新的檔案裡，不符格式的那一行就不輸出
* 這樣我們定要把一整行的所有電話都確認完之後才能輸出到新檔案
* 可以把 output 內容先塞到 ostringstream，最後再輸出到檔案(輸出 osstream_obj.str())
```C++
for (const auto &entry : people) { // for each entry in people
    ostringstream formatted, badNums; // objects created on each loop
    for (const auto &nums : entry.phones) { // for each number
        if (!valid(nums)) {
            badNums << " " << nums; // string in badNums
        } else // "writes" to formatted’s string
            formatted << " " << format(nums);
    }
    if (badNums.str().empty()) // there were no bad numbers
        os << entry.name << " " // print the name << formatted.str() << endl; // and reformatted numbers
    else    // otherwise, print the name and bad numbers
        cerr << "input error: " << entry.name
            << " invalid number(s) " << badNums.str() << endl;
}
