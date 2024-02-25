# Rust 学习笔记 - 3

___

## 结构体 Struct

+ 结构体语法:

    + 定义结构体: 字段名称与类型, **不允许将某个字段专门指定为可变的**.

        ```rust
        #[derive(Debug)]
        struct User {
            active: bool,
            username: String,
            email: String,
            sign_in_count: u64,
        }
        ```
    
    + 创建结构体实例: 每个字段都需进行初始化, 顺序不需要和定义顺序一致.
    
        ```rust
        let user1 = User {
            email: String::from("someone@example.com"),
            username: String::from("someusername123"),
            active: true,
            sign_in_count: 1,
        };
        ```
    
    + 缩略语法: **函数参数和结构体字段同名**时, 使用缩略方式进行初始化.
    
        ```rust
        fn build_user(email: String, username: String) -> User {
            User {
                email,
                username,
                active: true,
                sign_in_count: 1,
            }
        }
        ```
    
    + 更新语法: 未显式声明的字段全部自动获取, 所有权发生**部分 `move`**.
    
        ```rust
        let user2 = User {
            email: String::from("another@example.com"), // 赋值
            ..user1 // 更新语法
        };
        println!("{}", user1.active);
        // println!("{:?}", user1); Error!
        ```
    
+ **元组结构体**的字段可以没有名称.

    ```rust
    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
    ```

+ 元组结构体**支持模式匹配**, 会产生所有权的转移 (回顾「部分 `move`」, **结构体和其属性**之间存在一种**分离的所有权关系**).

    ```rust
    struct Point(i32, i32, i32);
    fn main() {
        let v = Point(0, 127, 255);
        check_color(v);
    }
    
    fn check_color(p: Point) { // `p` 发生 `部分 move`
        let Point(x, _, _) = p;
        assert_eq!(x, 0);
        assert_eq!(p.1, 127);
        assert_eq!(p.2, 255);
    }
    ```

+ **单元结构体**没有任何字段和属性, **只关心其行为**.

    ```rust
    struct AlwaysEqual;
    let subject = AlwaysEqual;
    // 为 AlwaysEqual 实现某个特征
    impl SomeTrait for AlwaysEqual { }
    ```

+ **结构体中使用引用**需要**加上生命周期**, 确保结构体作用范围小于借用数据的作用范围 (或者让结构体拥有数据, 而不是借用).

    ```rust
    struct User<'a> {
        username: &'a str,
        email: &'a str,
        sign_in_count: u64,
        active: bool,
    }
    fn main() {
        let user1 = User {
            email: "someone@example.com",
            username: "someusername123",
            active: true,
            sign_in_count: 1,
        };
    }
    ```

___

## 枚举 Enum

+ 枚举类型可作为**函数参数**.

    ```rust
    #[derive(Debug)]
    enum PokerSuit {
        Clubs,
        Spades,
        Diamonds,
        Hearts,
    }
    fn main() {
        let heart = PokerSuit::Hearts;
        print_suit(heart);
    }
    fn print_suit(card: PokerSuit) {
        println!("{:?}",card);
    }
    ```

+ 枚举是一个**结构体集合**, 枚举成员可**关联任何类型的数据信息**.

    ```rust
    enum Message {
        Quit,
        Move{x: i32, y: i32},
        Write(String),
        ChangeColor(i32, i32, i32),
    }
    
    fn main() {
        let m1 = Message::Quit;
        let m2 = Message::Move{x:1,y:1};
        let m3 = Message::ChangeColor(255,255,0);
    }
    ```

+ `Option` 枚举使用泛型参数 `T`, `None` 表示无有效值, 处理值为空的情况.

    ```rust
    enum Option<T> {
        Some(T),
        None,
    }
    let some_number = Some(5);
    let some_string = Some("a string");
    let absent_number: Option<i32> = None;
    ```

+ 使用**模式匹配解构**枚举类.

    ```rust
    let msg = Message::Move{x: 1, y: 2};
    if let Message::Move{x: a, y: b} = msg {
        assert_eq!(a, b);
    } else {
        panic!("不要让这行代码运行!");
    }
    ------------------------------------------------------
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }
    ```

+ 使用 `as` 将**枚举值强转为整数类型**.

    ```rust
    enum Number {
        Zero,
        One,
        Two,
    }
    fn main() {
        assert_eq!(Number::One as u8, 1u8);
    } 
    ```

___

## 数组 Array

+ 连续存放元素, 可以通过**索引访问**; 越界检查在运行时进行.

    ```rust
    let a = [9, 8, 7, 6, 5];
    let first = a[0];
    // `get` 返回 `Option<T>` 类型
    let second = a.get(1).unwrap();
    ```

+ 数组元素为非基础类型时, 调用 `std::array::from_fn ` **创建元素相同的数组**.

    ```rust
    let array: [String; 8] = std::array::from_fn(|_i| String::from("rust is good!"));
    println!("{:#?}", array);
    ```

+ **数组遍历**需要考虑**所有权的转移**.

    ```rust
    let one = [1, 2, 3];
    let two = [4, 5, 6];
    let arrays: [[u8; 3]; 2]  = [one, two];
    println!("{:p} {:?}", &arrays, arrays);
    
    for t in &arrays { // 引用借用 => 代价低
        println!("{:p} {:?}", t, t);
        for k in t {
            println!("{:p} {:?}", k, k);
        }
    }
    for t in arrays { // 数组 Copy => 深度拷贝, 代价高
        println!("{:p} {:?}", &t, t);
        for k in t {
            println!("{:p} {:?}", &k, k);
        }
    }
    for t in arrays.iter() { // 迭代器, 和 (1) 等价 => 代价低
        println!("{:p} {:?}", t, t);
        for k in t {
            println!("{:p} {:?}", k, k);
        }
    }
    ```

___

## 流程控制 Control Flow

+ **`if` 语句块是表达式**, 赋值时保证每个分支返回的类型一样.

    ```rust
    let n = 5;
    let big_n =
        if n < 10 && n > -10 {
            println!(" 数字太小，先增加 10 倍再说");
            10 * n
        } else {
            println!("数字太大，我们得让它减半");
            n / 2
        };
    println!("{} -> {}", n, big_n);
    ```

+ **`for` 循环**往往使用**集合的引用形式**, 否则所有权会**发生 `move`**; 实现了 **`copy` 特征**的集合会**直接进行拷贝**, 不发生所有权转移, 循环后可以正常使用; 不使用索引访问集合, 更安全简洁, 避免运行时边界检查, 性能更高.

  | 方式                          | 等价方式                                          | 所有权     |
    | ----------------------------- | ------------------------------------------------- | ---------- |
    | `for item in collection`      | `for item in IntoIterator::into_iter(collection)` | 转移所有权 |
    | `for item in &collection`     | `for item in collection.iter()`                   | 不可变借用 |
    | `for item in &mut collection` | `for item in collection.iter_mut()`               | 可变借用   |

  ```rust
  let a = [4, 3, 2, 1];
  // `.iter()` 方法把 `a` 数组变成一个迭代器.
  // `.enumerate()` 方法获取元素的索引.
  for (i, v) in a.iter().enumerate() {
      println!("第{}个元素是{}", i + 1, v);
  }
  ```

+ **`continue` 语句**跳过当前当次的循环, 开始下次的循环.

+ **`break` 语句**可以直接跳出当前整个循环; **可以带返回值**, 类似 `return`.

    ```rust
    for i in 1..5 {
        if i == 2 {
            continue;
        }
        if i == 4 {
            break i;
        }
        println!("{}", i);
    }
    ```
    
+ **`while` 语句**也能用来实现 **`for` 循环**的功能.

    ```rust
    let mut n = 0;
    while n <= 5  {
        println!("{}!", n);
        n = n + 1;
    }
    println!("我出来了！");
    ```

+ **`loop` 循环**适用面最高, 适用于所有循环场景, 在内部通过 `break` 关键字控制循环结束逻辑; **`loop` 是一个表达式**, 可以返回一个值.

    ```rust
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };
    println!("The result is {}", result);
    ```

+ **多层 `loop` 循环**可以使用 `continue` 或 `break` 控制外层循环, 外部循环必须拥有一个**标签 `'label`**, 并在 `break` 或 `continue` 时指定该标签.

    ```rust
    let mut count = 0;
    'outer: loop {
        'inner1: loop {
            if count >= 20 {
                break 'inner1; // 跳出 inner1 循环
            }
            count += 2;
        }
        count += 5;
        'inner2: loop {
            if count >= 30 {
                break 'outer;
            }
            continue 'outer;
        }
    }
    assert!(count == 30)
    ```

___

## `match` 和 `if let`

+ `match` 匹配将**一个值**与**一系列模式**相比较, 必须穷举所有可能; 每一个分支都是表达式, 返回值类型必须相同.

    ```rust
    enum Direction {
        East,
        West,
        North,
        South,
    }
    
    fn main() {
        let dire = Direction::South;
        match dire {
            Direction::East => println!("East"),
            Direction::North | Direction::South => {
                println!("South or North");
            },
            _ => println!("West"),
        };
    }
    ```

+ `match` 是一个**表达式**, 可以用来赋值; 也可以从模式中取出绑定的值.

    ```rust
    enum IpAddr {
       Ipv4,
       Ipv6
    }
    fn main() {
        let ip1 = IpAddr::Ipv6;
        let ip_str = match ip1 {
            IpAddr::Ipv4 => "127.0.0.1",
            _ => "::1",
        };
        println!("{}", ip_str);
    }
    ```

    ```rust
    enum UsState {
        Alabama,
        Alaska,
        // --snip--
    }
    enum Coin {
        Penny,
        Nickel,
        Dime,
        Quarter(UsState),
    }
    fn main() {
        let coin = Coin::Dime;
        let value = match coin {
            Coin::Penny => 1,
            Coin::Nickel => 5,
            Coin::Dime => 10,
            Coin::Quarter(state) => {
                println!("State quarter from {:?}!", state);
                25
            },
        }
    }
    ```

+ `if let` 只**匹配一个条件**, 忽略其它条件.

    ```rust
    match fields[1].parse::<f32>() {
        Ok(length) => println!("{} cm", length),
        _ => (),
    }
    ------------------------------------------------------
    // 执行成功会返回 `Ok(f32)` 类型, 执行失败会返回 `Err(e)` 类型.
    if let Ok(length) = fields[1].parse::<f32>() {
        println!("{} cm", length);
    }
    ```

+ `matches!` 宏**将一个表达式和模式进行匹配**, 然后返回匹配的结果.

    ```rust
    enum MyEnum {
        Foo,
        Bar
    }
    fn main() {
        let v = vec![MyEnum::Foo,MyEnum::Bar];
        let foos = v.iter().filter(|x| matches!(x, MyEnum::Foo)).collect();
    }
    ------------------------------------------------------
    let bar = Some(4);
    assert!(matches!(bar, Some(x) if x > 2));
    ```

+ 如果没有**实现 `Copy` 特征**, 匹配成功会**转移所有权**, 不成功则不转移所有权.

    ```rust
    let item = Some(String::from("item"));
    match item {
        // item 不是 Copy 的, 匹配消耗了 Option, 获取里面的值
        Some(it) => println!("{it}"),
        None => unreachable!(),
    }
    ------------------------------------------------------
    let item = Some(String::from("item"));
    match item {
        // ref 引用 Option 里的值, 并没有消耗 item
        Some(ref it) => println!("{it}"),
        None => unreachable!(),
    }
    ------------------------------------------------------
    let item = Some(String::from("item"));
    match &item {
        // 引用 item, 不会消耗所有权
        Some(it) => println!("{it}"),
        None => unreachable!(),
    }
    ------------------------------------------------------
    let item = Some("item");
    match item {
        // item 是 Copy 的, match 使用 item 的副本, item 未被消耗
        Some(it) => println!("{it}"),
        None => unreachable!(),
    }
    ```

+ `Option<T>` 常用作**处理 `map` 中可能的空值**.

    ```rust
    let x: Option<i32> = None;
    println!("{}", x
        .map(|x| x + 1)
        .map(|x| x * 2)
        .expect("报错了，他是空的")
    )
    ```

___

## 模式匹配 Pattern Match

+ `match` 分支:

    ```rust
    let x = 1;
    // 字符和数字值是仅有的可以判断是否为空的类型
    match x {
        1 | 2 => println!("one or two"),
        3..=7 => println!("three to seven"),
        _ => println!("anything"),
    }
    ```

+ `if let` 分支:

    ```rust
    if let PATTERN = SOME_VALUE {
    	STATEMENT;
    }
    ```

+ `while let` 条件循环:

    ```rust
    let mut stack = Vec::new();
    stack.push(1);
    stack.push(2);
    stack.push(3);
    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
    ------------------------------------------------------
    // `vec` 的类型是 `[Option]`
    let vec = [Some(1), Some(2), Some(3), Some(4), None, Some(6)];
    // `.pop()` 返回的类型是 `Option<Option>`
    while let Some(tp) = vec.pop().unwrap() {
        println!("case success {:?}", tp);
    }
    ```

+ `for` 循环:

    ```rust
    let v = vec!['a', 'b', 'c'];
    for (index, value) in v.iter().enumerate() {
        println!("{} is at index {}", value, index);
    }
    ```

+ `let` 语句:

    ```rust
    // 变量名也是一种模式
    let x = 5;
    ------------------------------------------------------
    let (x, y, z) = (1, 2, 3);
    ```

+ 函数参数:

    ```rust
    fn print_coordinates(&(x, y): &(i32, i32)) {
        println!("Current location: ({}, {})", x, y);
    }
    fn main() {
        let point = (3, 5);
        print_coordinates(&point);
    }
    ```

+ **不可驳模式匹配**: `let`, `for` 和 `match` 要求完全覆盖匹配才能通过编译.

+ **可驳模式匹配**: `if let` 允许匹配一种而忽略其余的模式.

+ 使用 `|` 匹配多个值, 使用 `..=` 匹配一个闭区间的数值序列.

+ **`@` 运算符**将一个**与模式相匹配的值绑定到新的变量上**.

    ```rust
    enum Message {
        Hello { id: i32 },
    }
    let msg = Message::Hello { id: 5 };
    match msg {
        Message::Hello { id: id_variable @ 3..=7 } => {
            println!("Found an id in range: {}", id_variable)
        },
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        },
        Message::Hello { id } => {
            println!("Found some other id: {}", id)
        },
    }
    ------------------------------------------------------
    struct Point {
        x: i32,
        y: i32,
    }
    let p = Point { x: 0, y: 10 };
    match p {
        Point { x, y: 0 } => println!("x axis at {}", x),
        Point { x: 0..=5, y: y@ (10 | 20) } => {
            println!("y axis at {}", y),
        }
        Point { x, y } => println!("neither: ({}, {})", x, y),
    }
    ```

