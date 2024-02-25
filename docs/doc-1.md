# Rust 学习笔记 - 1

___

## 认识 Cargo

+ 包管理工具, 使任何用户拿到代码都能正常运行.

+ 项目分为两个类型 `bin` 和 `lib`, 前者是**可运行项目**, 后者是**依赖库项目**; 默认创建 `bin` 类型项目.

    ```shell
    > cargo new world_hello
    ```

+ 高性能代码.

    ```shell
    > cargo build/run --release
    ```

+ 快速检查代码编译通过.

    ```shell
    > cargo check
    ```

+ 引入项目依赖项.

    ```shell
    > cargo add num
    num = "0.4.1"
    ----------------------------------------------------------
    hammer = { version = "0.5.0"}
    color = { git = "https://github.com/bjz/color-rs" }
    geometry = { path = "crates/geometry" }
    ```

___

## `println!`

+ 原生支持 **UTF-8 编码**的字符串.
+ **宏操作符**, 是一种**特殊类型函数**.
+ 自动识别输出数据的类型, 使用 `{}` 输出占位符.
+ 实现 `Display` 特征, 以使用 `{}` 方式打印结构体.
+ 实现 `Debug` 特征, 以使用 `{:?}` 或 `{:#?}` 方式打印结构体.

+ `dbg!` 宏输出到 `stderr`, 而 `println!` 输出到 `stdout`.

    ```rust
    #[derive(Debug)]
    struct Rectangle {
        width: u32,
        height: u32,
    }
    fn main() {
        let scale = 2;
        let rect1 = Rectangle {
            width: dbg!(30 * scale),
            height: 50,
        };
        dbg!(&rect1);
    }
    ------------------------------------------------------
    > [src/main.rs:10] 30 * scale = 60
    > [src/main.rs:14] &rect1 = Rectangle {
        width: 60,
        height: 50,
    }
    ```

___

## 变量绑定 Binding

+ 可变变量为编程提供**灵活性**, 不可变变量提供**安全性**.
+ 变量使用 `let` 关键字声明, 默认情况下是不可变的.
+ 常量使用 `const` 关键字声明, 值的类型必须标注.
+ 使用**下划线开头**或 `#[allow(unused_variables)]`, **忽略**未使用变量.
+ 使用 `mut` 声明的变量可以**修改同一内存地址的值**, 不会发生内存对象再分配, 性能更好; 使用 `let` 声明的变量**只拥有同样名称**, 涉及内存对象的再分配.

___

##  变量解构 Deconstruction

+ 使用 `let` 表达式进行**复杂变量解构**, **匹配部分内容**.
+ 支持使用**元组、切片和结构体模式**进行变量解构.
    ```rust
    let (a, mut b): (bool, bool) = (true, false);
    ```

___

## 数值类型 Scalar Types

+ 通常根据变量值和上下文**自动推导**变量类型, 某些情况需进行**手动标注**.

+ 允许在复杂类型上定义运算符, 被称为**运算符重载**.

+ **浮点数**采用 **IEEE-754 标准**, 但是没有实现 `std::cmp::Eq` 特征, 存在**精度问题**, 无法作为 `HashMap` 的 `Key` 类型.

    ```rust
    assert!(0.1_f32 + 0.2_f32 == 0.3_f32);
    assert!((0.1_f64 + 0.2 - 0.3).abs() < 0.001);
    ```

+ **序列**只允许数字或字符类型; 如 `1..5` 生成 1 到 4 的连续数字, `1..=5` 生成 1 到 5 的连续数字.

    ```rust
    assert_eq!((1..5), Range{start: 1, end: 5});
    assert_eq!((1..=5), RangeInclusive::new(1, 5));
    ```

+ **类型转换**必须是**显式进行**的.

    ```rust
    let n: u16 = 38_u8 as u16;
    ----------------------------------------------------------
    for c in 'a'..='z' {
        println!("{}", c as u8);
    }
    ```

+ 可以在**数值上使用方法**.

    ```rust
    let w: f32 = 13.14_f32.round();
    ```

___

## 字符、布尔、单元类型 Char/Boolean/Union Types

+ **字符类型**支持 `Unicode` 编码, 占用内存的大小为 **4 个字节**.

    ```rust
    let x: char = '中';
    println!("{}", std::mem::size_of_val(&x));
    ----------------------------------------------------------
    for c in '你'..='我' {
        println!("字符 c = {}", c as u32);
        println!("汉字 c = {}", c);
    }
    ```

+ 布尔类型值为 `true` 和 `false`, 占用内存的大小为 1 个字节.

+ **单元类型**作为 `main` 函数、`println!` 宏等的返回值或 `map` 的值 (此时 `map` 相当于 `set`), 表示**不分配任何内存**, 只用来**占位**.

    ```rust
    let unit: () = ();
    println!("{}", std::mem::size_of_val(&unit));
    ```

+ 单元类型便于**统一有返回值和无返回值的函数**, 以便将函数当做对象使用.

___

## 语句和表达式 Statement and Expression

+ `let` 是语句, 不返回值, 不能给其它变量赋值.

    ```rust
    let y = {
        let x_squared = x * x;
        let x_cube = x_squared * x;
        // 下面表达式的值将被赋给 `y`
        x_cube + x_squared + x
    };
    ----------------------------------------------------------
    let z = {
        // 分号让表达式变成了语句, 返回的是语句的值 `()`
        2 * x;
    };
    ```

+ 调用**函数、宏**是**表达式**, 会进行求值并返回值, 或隐式返回一个单元类型.

    ```rust
    // `{}` 是一个块作用域, 没有返回值, 因此隐式地返回 `()`
    assert_eq!((), {});
    ```

+ **基于表达式**是**函数式语言**的重要特征.

___

## 函数 Function

+ **Rust** 是**强类型语言**, 需要为函数参数都标识出具体类型.

+ **函数是表达式**, 返回值是函数体最后一条表达式, 可以**使用 `return` 提前返回**.

    ```rust
    fn plus_or_minus(x:i32) -> i32 {
        if x > 5 {
            return x - 5;
        }
        x + 5
    }
    ```

+ 函数体最后一条不是表达式且没有 `return` 关键字时, 不会返回任何值, 相当于返回类型为 `()`.

    ```rust
    fn report<T: std::fmt::Debug>(item: T) {
    	println!("{:?}", item);
    }
    ----------------------------------------------------------
    fn clear(text: &mut String) -> () {
    	*text = String::from("");
    }
    ```

+ 用 `!` 作函数返回类型, 表示函数**永不返回**, 称为**发散函数**.

    ```rust
    fn dead_end() -> ! {
    	panic!("崩溃吧");
    }
    ----------------------------------------------------------
    fn never_return() -> ! {
        loop {
            println!("I return nothing");
            std::thread::sleep(std::time::Duration::from_secs(1))
        }
    }
    ```

+ 发散函数也可以**用于 `match` 表达式匹配**, 替代任何类型的值.

    ```rust
    let _v = match b {
        true => 1,
        false => {
            println!("Success!");
            panic!("No value for `false` but panic")
        }
    };
    println!("Exercise Failed!");
    ```
