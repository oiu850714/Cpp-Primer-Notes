# C++ Primer Chapter 5 Statements
* Statements are **executed sequentially.**
* C++ also defines a set of flow-ofcontrol statements that allowmore complicated execution paths.

## 5.1 Simple Statements
* An expression, such as ival + 5, **becomes an expression statement when it is followed by a semicolon**
* Expression statements **cause the expression to be evaluated** and its *result discarded:*
    ```C++
    ival + 5; // rather useless expression statement
    cout << ival; // useful expression statement
    ```
    * first exp statement is useless, addition is doen but result not used
    * More commonly, an expression statement **contains an expression that has a side effect**—such as assigning a new value to a variable, or printing a result—when it is evaluated.
#### Null Statements
```C++
; // null statement
```
* A null statement is useful where the language requires a statement but the program’s logic does not.
    * 用在那些語法上需要一個 statement 但是程式邏輯不需要的地方
    * 例如 loop 要做的事情全部寫在 header 裡面，可是還是需要一個 statement 當作 loop body，這時就加個 ;
        ```C++
        // read until we hit end-of-file or find an input equal to sought 
        while (cin >> s && s != sought)
            ;// null statement
        ```
    * **Null statements should be commented.** That way anyone reading the code can see that the statement was omitted intentionally.
#### Beware of Missing or Extraneous Semicolons
* Null statements 就是 statement，他當然可以出現在 statements 可以出現的地方:
    ```C++
    ival = v1 + v2;; 
    // ok: second semicolon is a superfluous null statement
    ```
    * 上面有**兩個** statements，一個是 exp statement，一個是 null statement
    * 通常無害，但如果不小心在 loop header 後面打了 ; 就很有可能會出事

#### Compound Statements (Blocks)
* A compound statement, usually referred to as **a block**, is a (possibly empty) **sequence of statements** and declarations **surrounded by a pair of curly braces.**

* **A block is a scope (§ 2.2.4, p. 48).**
    *  Names introduced inside a block are accessible only in that block and in blocks nested inside that block.
* Compound statements are used when the language requires a single statement but the logic of our program needs more than one.
    * 例子一樣是 loop
* 你也可以擺 empty block，它等價於 null statement 

## 5.2 Statement Scope
* 我們可以在 if, switch, while, for 的 control structure(也就是它們後面跟著的括號)內定義變數
* 對你沒看錯不只有 for 可以，不過如果是在 while 裡面宣告的話，**每次 loop 都會重新宣告跟初始化**
```C++
while (int i = get_num()) 
// i is created and initialized on each iteration     
    cout << i << endl;
i = 0; // error: iis not accessible outside the loop
```

## 5.3 Conditional Statements

### 5.3.1 The if Statement
* condition must have a type that is convertible to bool
* switch statement

#### Dangling else
* 老生常談
* In C++ the ambiguity is resolved by specifying that each else is matched with the closest preceding unmatched if.
#### Controlling the Execution Path with Braces
* 請用括號表示你的邏輯..

### 5.3.2 The switch Statement
* A switch statement provides a convenient way of selecting among a (possibly large) number of ***fixed*** alternatives.
* 上面提過，switch () 內一樣可以宣告變數，阿斯
* **The expression is converted to integral type.**
* If the expression matches the value of a case label, **execution begins with the first statement following that label.**
    * through the end of switch
    * or until break;
* **case label must be *integral* const expressions!**
    * 不能是非整數 literal
    * 兩個 label 不能有同樣 value
* 還有一個 label 叫做 default
* 你不希望多個 case 被執行，請用 break 跳出 switch
* 通常都會用 break 跳出 switch
    * Omitting a break at the end of a case happens rarely. **If you do omit a break, include a comment explaining the logic.**
#### Forgetting a break Is a Common Source of Bugs
* 講過ㄌ
#### The default Label
* The statements following the default label are executed **when no case label matches the value of the switch expression.**
* **BEST PRACTICE**
    * It can be useful to **define a default label even if there is no work for the default case.** Defining an empty default section indicates to subsequent readers that the case was considered.
* label 不能單獨存在，必須跟在 label 或 statement 的前面(注意這裡有點遞迴定義，總之就是要跟在 statement 前面)，如果整個 switch code 只有 label，你至少要加個 null statement 之類的

#### Variable Definitions inside the Body of a switch
* C++ 不准你寫這樣的 code:
```C++
case true:
// this switch statement is illegal because these initializations might be bypassed
string file_name; // error: control bypasses an implicitly initialized variable 
int ival = 0; // error: control bypasses an explicitly initialized variable 
int jval;// ok: because jvalis not initialized 
break;

case false: 
// ok: jval is in scope but is uninitialized 
jval = next_num(); // ok: assign a value to jval
if (file_name.empty()); // file_name is in scope but wasn’t initialized
```
* 假設這 code legal，那 fila_name 跟 ival 這些在 true label 被初始化的變數就有可能因為跳過 true label 而沒被初始化，那這樣你在執行 false label 時不會ㄎㄧㄤ掉嗎?
* 簡單說如果你從還沒宣告某變數並且初始化的的地方跳到已經宣告了某變數，並且初始化"之後"的地方，這樣就叫做從 variable out of scope 跳到 variable in scope，這樣不合法
* 不過你的變數是 built-in 並且沒有初始化，那這樣合法
    * 可是這樣真的是糞code...
* 解法是，如果你有一個變數會在兩個 label 以上使用到，請你宣告並初始化在 switch 外面
    * 如果你只會在一個 label 內用到，你可以宣告在某 label 的 code 內，但是必須把那個 label 的 code 用 block 包起來，這樣一跳出那個 label，裡面宣告的變數就會 out of scope 了，不會有上面說的從沒宣告的地方跳到宣告之後的地方，因為宣告之後的地方不存在(一超過那個 label，變數就已經 out of scope 了)


## 5.4 Iterative Statements
### 5.4.1 The while Statement
* while 的 condition 一樣可以塞宣告(必須給 initializer)，這樣每次執行 condition，變數都會宣告並初始化
* 如果你沒給 initializer，VScode 會噴 variable declaration in condition must have an initializer
* Variables defined in a while condition or while **body are created and destroyed on each iteration.**
* Using Pattern:
    * when we want to **iterate indefinitely**

### 5.4.2 Traditional for Statement
* 
    ```C++
    for (init-statement condition; expression) statement
    ```
    * *init-statement* must be a declaration statement, an expression statement, or a null statement.
### 5.4.3 Range for Statement
    ```C++
    for (declaration : expression) statement
    ```
* 作用在 container 或 sequence 上
    * 看起來好像是 built-in array，brace initializer，還有支援 begin() end() 的 STL 都行
    * declaration 宣告的型態一定要是 expression 的 element 可以轉過去的型態
        * 能用 auto 就用!
* On each iteration, **the control variable is defined and initialized by the next value in the sequence**, after which statement is executed.
* 來點例子
    ```C++
    for (auto &r : v) // for each element in v r *=2;
    // double the value ofeach element in v
    ```
* 傳統的等價 loop
    ```C++
    for (auto beg = v.begin(), end = v.end(); beg != end; ++beg) 
    { 
        auto &r = *beg; // r must be a reference so we can change the element 
        r *=2;
        // double the value ofeach element in v
    }
    ```
    * 注意看 reference 的部分，如果在傳統 loop 內把 initializer 寫成 reference 就沒辦法更動了，所以是寫在 body 內。
* Now we can understand why we said in § 3.3.2 (p. 101) that we cannot use a range for to add elements to a vector
    * In a range for,the value of end() is cached.
    * If we add elements to (or remove them from) the sequence, the value of end might be invalidated(§ 3.4.1, p. 110). 
    * 第九章才會講到底為什麼會被 invalidated

### 5.4.4 The do while Statement
```C++
do 
    statement
while (condition);
```
* statement 不管 condition 如何至少會執行一次
* 記得 condition 後面有分號
* 還有，**condition 內用到的變數一定要宣告在 do while 外面**！宣告在 do 的 body 內會噴 error
* 再來，do while 的 condition 就不像其他 loop 一樣可以宣告變數惹
    * 你想想看如果可以宣告，然後你的 body 又用到這個變數，那不就是在宣告之前就用了變數?!

## 5.5 Jump Statements
* break, continue, goto, return(chapter 6)

### 5.5.1 The break Statement
* A break statement terminates the nearest enclosing **while, do while, for,or switch** statement.
* A break affects only the nearest enclosing loop or switch

### 5.5.2 The continue Statement
* terminates the **current iteration of the nearest enclosing loop**
* 如果你在一個被包在 loop 內的 switch 用 continue，比用 break 跳的還遠耶www
* In the case of a while or a do while, execution continues by evaluating the condition. In a traditional for loop, execution continues at the expression inside the for header. In a range for, execution continues by initializing the control variable from the next element in the sequence.


### 5.5.3 The goto Statement
* Label identifiers
    * independent of names used for variables and other identifiers.
* goto 要跳的地方必須在同個 function 內
* As with a switch statement, **a goto cannot transfer control from a point where an initialized variable is out of scope to a point where that variable is in scope:**
    * 跟 switch 的 label 有 87%像
* A jump backward over an already executed definition is okay. **Jumping back to a point before a variable is defined destroys the variable and constructs it again:**

## 5.6 try Blocks and Exception Handling
* run-time anomalies
* 這裡就看看就好.. 這噁心的東西等到該學的都學完再來好好補