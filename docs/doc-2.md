# 所有权与进阶类型

___

## 所有权 Ownership

+ 编译器在 **编译期** 进行所有权检查, 对程序 **运行期没有性能损失**.

+ 实现了 [RAII](https://docs.microsoft.com/en-us/cpp/cpp/object-lifetime-and-resource-management-modern-cpp?view=msvc-170) (Resource Acquisition Is Initialization programming) 原则.

+ 规则:

    + **每个值被一个变量拥有, 称为值的所有者.**
    + **一个值同时只被一个变量拥有.**
    + **所有者离开作用域范围时, 值将被丢弃.**

+ **字符串 `String`** 由存储在栈中的堆指针、字符串长度、字符串容量组成; 字符串数组分配到堆上, 动态伸缩, 存储编译时大小未知的文本; **拷贝后所有权转移**, 称为 **移动 (`move`)**.

    ```rust
    let mut s = String::from("hello"); // `::` 是调用操作符.
    s.push_str(", world!");
    println!("{}", s);
    ----------------------------------------------------------
    let x = String::from("hello, world");
    // 防止所有权从 x 转移到 y.
    let y = x.clone()/x.as_str()/&x;
    ```

+ **字符串字面值 `&str`** 被硬编码到全局内存区, 不可变; 变量绑定了引用的所有权, 指向字符串的内存地址, **拷贝时无需所有权转移**; 生命周期持续到程序结束, 由操作系统回收内存.

    ```rust
    let x: &str = "hello, world";
    let y = x; // 引用同一个字符串
    println!("{},{}", x, y);
    ```

+ 整数是基本数据类型, 在栈中大小固定, 通过值拷贝完成赋值, 无需所有权转移, 称为 **拷贝 (`copy`)**.

    ```rust
    let x = 5;
    let y = x;
    ```

+ **克隆 (深拷贝)** 支持深度复制, 无需所有权转移; 任何自动的复制都不是 **克隆 (`clone`)**, 对运行时性能影响较小.

+ **拷贝 (浅拷贝)** 只发生在栈上, 性能很高; 无需分配内存或形式资源的类型 (拥有 `Copy` 特征) 可以拷贝, 包括 **整数、布尔、浮点数、字符、元组、不可变引用** 等类型.

    ```rust
    let x = (1, 2, (), "hello");// 可以直接拷贝元组
    let y = x;
    println!("{:?}, {:?}", x, y);
    ```

+ **将值传递给函数**, 也会发生 `move` 或 `copy`, 导致 **所有权转移**.

+ 可变性: 当 **所有权转移** 时, **可变性** 也可以随之改变.

    ```rust
    let s = String::from("hello, ");
    let mut s1 = s;
    s1.push_str("world")
    ```

+ **部分 `move`**: 解构一个变量时, **同时使用 `move` 和引用模式绑定** 的方式. 转移了变量一部分的所有权, 获取了另一部分的引用. **原变量无法使用**, 但 **没转移所有权的部分仍可以使用**.

    ```rust
    #[derive(Debug)]
    struct Person {
        name: String,
        age: Box<u8>,
    }
    let person = Person {
        name: String::from("Alice"),
        age: Box::new(20),
    };
    
    // person.name 的所有权转移给新的变量 `name`.
    // ref 的使用相当于: let age = &person.age .
    let Person { name, ref age } = person;
    // Error!
    // println!("The person struct is {:?}", person);
    println!("The person's age is {}", person.age);
    ----------------------------------------------------------
    let t = (String::from("hello"), String::from("world"));
    let (ref s1, ref s2) = t;
    println!("{:?}, {:?}, {:?}", s1, s2, t);
    ```

___

## 引用 Reference

+ 规则:

    - 同一时刻, 只能拥有 **一个可变引用**, 或 **任意多个不可变引用**.
    - **引用必须总是有效的.**

+ 常规引用是一个 **指针类型**, 指向对象存储的 **内存地址**.

+ We call **the action** of creating a reference (引用) borrowing(借用).

+ 不允许比较整数与引用, 必须使用 **解引用运算符** 解出引用指向的值.

    ```rust
    let x = 5;
    let y = &x;
    assert_eq!(5, x);
    assert_eq!(5, *y);
    ```

+ **引用作为参数传递给函数**, 允许 **使用值**, 但 **不获取所有权**.

+ 声明 **可变类型** 后, 可以创建 **可变引用** 和接受可变引用参数的函数; **`.` 运算符自动为左端操作数进行解引用**. 

    ```rust
    fn main() {
        let mut s = String::from("hello");
        change(&mut s);
    }
    fn change(some_string: &mut String) {
        some_string.push_str(", world");
    }
    ```

+ 进行可变引用后, 不能通过非可变引用或者直接访问这个值 (即 **可变引用的使用范围** 不能 **与其同源引用的使用范围存在交集**).

    ```rust
    let mut a = String::from("123");
    let b = &mut a;
    println!("a = {}, b = {}", a, b); // Error!
    ----------------------------------------------------------
    let mut x = 1;
    let y = &mut x;
    x = 2; // Error!
    *y = 3;
    ```

+ **同一作用域只能有一个可变引用**; 编译器通过 Non-Lexical Lifetimes 优化行为, 找到引用在作用域结束前就不再被使用的位置.

    ```rust
    fn main() {
        let mut s = String::from("hello");
        let r1 = &s;
        let r2 = &s;
        println!("{} and {}", r1, r2); // 新编译器, r1, r2 作用域结束
        let r3 = &mut s;
        println!("{}", r3);
    }   // 老编译器, r1, r2, r3 作用域结束
        // 新编译器, r3 作用域结束
    ```

+ 编译器确保引用 **不会变成悬垂状态**, 释放数据前必须先停止其引用的使用; 需要考虑生命周期才可正常返回引用;

    ```rust
    fn main() {
        let reference_to_nothing = dangle();
    }
    fn dangle() -> &String {
        let s = String::from("hello");
        &s // 引用会指向一个无效的 `String`.
    }
    ```

+ 可以 **使用 `ref` 获取一个值的引用**.

    ```rust
    let c = '中';
    let r1 = &c;
    let ref r2 = c;
    assert_eq!(*r1, *r2);
    ```

___

## 切片 Slice

+ 切片引用集合中部分连续的元素序列, 而非引用整个集合.

    ```rust
    // 对于 String 而言, 切片是对某一部分的引用.
    // String 切片的类型标识是 &str.
    let s = String::from("hello world");
    let hello = &s[0..5]; // 等价: let hello = &s[..5];
    let world = &s[6..11]; // 等价: let world = &s[6..];
    ----------------------------------------------------------
    // 和字符串切片的工作方式一样.
    let a = [1, 2, 3, 4, 5];
    let slice = &a[1..3];
    assert_eq!(slice, &[2, 3]);
    ```

+ **切片的长度无法在编译期得知**, 无法直接使用.

+ 使用 **切片特指切片引用**, 占用 **2 个字的内存空间**, 即指向数据的指针和切片长度.

    ```rust
    let arr: [char; 3] = ['中', '国', '人'];
    let slice = &arr[..2];
    assert!(std::mem::size_of_val(&slice) == 16);
    ```


___

## 字符串 String

+ **字符是 Unicode 类型**, 每个字符占据 4 个字节内存空间.

+ **`str`, `&str`, `String` 是 UTF-8 字符串类型**, 字符所占字节数是变化的, 降低了占用的内存空间.

+ 使用 `&` 可以将 `Box<str>` 转换为 `&str` 类型.

    ```rust
    fn main() {
        let s: Box<str> = "hello, world".into();
        greetings(&s)
    }
    fn greetings(s: &str) {
        println!("{}",s)
    }
    ```

+ **`String` 底层存储格式是 `Vec<u8>`**, 不允许索引 (否则需遍历来定位合法字符).

    ```rust
    let hello = "中国人";
    let s = &hello[0..2];
    ----------------------------------------------------------
    > thread 'main' panicked at 'byte index 2 is not a char boundary; it is inside '中' (bytes 0..3) of `中国人`', src/main.rs:4:14
    > note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
    ```

+ `String` 字符串:

    + **`&String` 可以被隐式地转换成 `&str` 类型.**

    + **追加 - Push:**

        + 使用 `push()` 追加字符, 使用 `push_str()` 追加字符串字面量.
        + 不会返回新字符串, 字符串变量必须由 `mut` 关键字修饰.

    + **插入 - Insert:**

        + 使用 `insert()` 插入字符, `insert_str()` 插入字符串字面量.
        + 不会返回新字符串, 字符串变量必须由 `mut` 关键字修饰.

    + **替换 - Replace:**

        + `replace()` 替换所有匹配的字符串, 返回一个新字符串.
        + `replacen()` 控制替换的匹配字符串的个数, 返回一个新字符串.
        + `replace_range()` 控制替换字符串的范围, 直接操作原字符串, 不会返回新字符串, 字符串变量必须由 `mut` 关键字修饰.

    + **删除 - Delete:**

        + `pop() ` 删除并返回最后一个字符, 直接操作原来的字符串, 返回值是 `Option` 类型 (字符串为空则返回 `None`).
        + `remove() ` 删除并返回指定位置的字符, 直接操作原来的字符串.
        + `truncate() ` 删除从指定位置开始到结尾的字符, 直接操作原来的字符串, 无返回值.
        + `clear() ` 清空字符串, 直接操作原来的字符串, 无返回值.

    + **连接 - Concatenate:**

        + 使用 `+` 或者 `+=` 连接字符串.
        + `+` 返回一个新字符串, 变量声明不需要 `mut` 关键字修饰.
        + `&str` 添加到 `String`, 返回新 `String`, 原 `String` **所有权转移**.

        ```rust
        impl Add<&str> for String {
            type Output = String;
            #[inline]
            fn add(mut self, other: &str) -> String {
                self.push_str(other);
                self
            }
        }
        ------------------------------------------------------
        let s1 = String::from("tic");
        let s2 = String::from("tac");
        let s3 = String::from("toe");
        // String = String + &str + &str + &str + &str
        let s = s1 + "-" + &s2 + "-" + &s3;
        ```

    + `chars()` 方法以 **Unicode 字符** 的方式遍历字符串.

    + `bytes()` 返回字符串的底层 **字节数组** 表现形式

        ```rust
        for c in "中国人".chars() {
            println!("{}", c);
        }
        ------------------------------------------------------
        for b in "中国人".bytes() {
            println!("{}", b);
        }
        ```
        
    + `String::with_capacity()` 返回具有 **指定初始大小容量** 的字符串.

+ **字节字符串** 是 **非 UTF-8 类型**.

    ```rust
    use std::str;
    fn main() {
        let bytestring: &[u8; 21] = b"this is a byte string";
        
        // 字节数组没有实现 `Display` 特征, 只能使用 `Debug` 方式打印
        println!("A byte string: {:?}", bytestring);
        
        // 字节数组可以使用转义, 不支持 unicode 转义
        let escaped = b"\x52\x75\x73\x74 as bytes";
        // let escaped = b"\u{211D} is not allowed";
        println!("Some escaped bytes: {:?}", escaped);
    
        // raw string
        let raw_bytestring = br"\u{211D} is not escaped here";
        println!("{:?}", raw_bytestring);
        // 字节数组转成 `str` 类型可能会失败
        if let Ok(my_str) = str::from_utf8(raw_bytestring) {
            println!("And the same as text: '{}'", my_str);
        }
    
        // 字节数组可以不是 UTF-8 格式
        let shift_jis = b"\x82\xe6\x82\xa8\x82\xb1\x82\xbb"; // "ようこそ" in SHIFT-JIS
        // 但是转成 `str` 类型可能会失败
        match str::from_utf8(shift_jis) {
            Ok(my_str) => println!("Successful: '{}'", my_str),
            Err(e) => println!("Failed: {:?}", e),
        };
    }
    ```
    
+ 使用 **`utf8_slice`** 按照 **字符的自然索引方式** 对 **UTF-8 字符串** 进行切片访问.

    ```rust
    > cargo add utf8_slice
    ------------------------------------------------------
    use utf8_slice;
    fn main() {
       let s = "The 🚀 goes to the 🌑!";
       let rocket = utf8_slice::slice(s, 4, 5);
       // Will equal "🚀"
       println!("{}", rocket)
    }
    ```
    
+ 使用 **`from_utf8`** 将 **UTF-8 数组** 转化为 **UTF-8 字符串**.

    ```rust
    fn main() {
        let mut s = String::new();
        s.push_str("hello");
    
        let v = vec![104, 101, 108, 108, 111];
    	let s1 = String::from_utf8(v).unwrap();
        
        assert_eq!(s, s1);
        println!("Success!")
    }
    ```
    
+ `String` 是一个 **存储在栈上** 的 **智能指针** 结构体, 指向 **堆上的字符串底层数据**.

    ```rust
    let story = String::from("Rust By Practice");
    // Prevent automatically dropping the String's data
    let mut story = std::mem::ManuallyDrop::new(story);
    
    let ptr = story.as_mut_ptr();
    let len = story.len();
    let capacity = story.capacity();
    
    // Re-build a String out of ptr, len, and capacity.
    let s = unsafe { String::from_raw_parts(ptr, len, capacity) };
    assert_eq!(*story, s);
    ```

---

## 元组 Union

+ 使用 **模式匹配** 解构元组, 支持 **解构式赋值**.

    ```rust
    // 解构: 用同样的形式匹配出一个复杂对象的值
    let tup = (500, 6.4, 1);
    let (x, y, z) = tup;
    println!("The value of y is: {}", y);
    ------------------------------------------------------
    let (x, y, z);
    (y,z,x) = (1, 2, 3);
    assert_eq!(x, 3);
    assert_eq!(y, 1);
    assert_eq!(z, 2);
    ```

+ 使用 **`.` 操作符获取元组值**, 不通过 `[]` 索引 (数组元素类型一致, 存储空间大小一致, 移动指针容易计算; 元组各元素类型不一致).

    ```rust
    // 索引: 只访问某个特定元素
    let x: (i32, f64, u8) = (500, 6.4, 1);
    let five_hundred = x.0;
    let six_point_four = x.1;
    let one = x.2;
    ```

+ 元组可以作为 **函数参数或返回值**.

    ```rust
    fn main() {
        let s1 = String::from("hello");
        let (s2, len) = calculate_length(s1); // 模式匹配
        println!("The length of '{}' is {}.", s2, len);
    }
    fn calculate_length(s: String) -> (String, usize) {
        // 接收字符串所有权, 计算长度, 返回所有权和长度
        let length = s.len();
        (s, length)
    }
    ------------------------------------------------------
    fn main() {
        let (x, y) = sum_multiply((2,3));
        assert_eq!(x, 5);
        assert_eq!(y, 6);
    }
    fn sum_multiply(nums: (i32, i32)) -> (i32, i32) {
        (nums.0 + nums.1, nums.0 * nums.1)
    }
    ```
    
+ 过长的元组无法以 `Debug` 模式被打印输出 (长度限制为 12).

    ```rust
    // let tuple = (0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12);
    let tuple = (0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11);
    println!("long tuple: {:?}", tuple);
    ```
