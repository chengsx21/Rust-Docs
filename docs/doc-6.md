# 错误处理与项目结构

___

## `panic!` 深入剖析

+ 对于 **严重影响程序运行** 的 **不可恢复错误**, 需要 **触发 `panic`** 进行解决.

+ 使用 **`panic!` 宏主动触发异常**, 可选择 **直接终止** 或获取 **详细的栈展开信息**.

    ```shell
    > RUST_BACKTRACE=1 cargo run
    
    > thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
    stack backtrace:
    	...
    ```

+ **触发 `panic`** 的线程是 **`main` 线程**, 程序会终止; 是 **其它子线程**, 不会导致整个程序的结束; 如果 **展开线程本身 `panic`**, 那么展开也会随之停止.

___

## 返回值 `Result` 和 `?`

+ 通过 **`Result` 枚举** 提供成功或错误信息.

    ```rust
    use std::fs::File;
    use std::io::ErrorKind::NotFound;
    
    fn main() {
        let f = File::open("hello.txt");
        let f = match f {
            Ok(file) => file,
            Err(err) => match err.kind() {
                NotFound => match File::create("hello.txt") {
                    Ok(fc) => fc,
                    Err(e) => panic!("Failed to create: {:?}", e),
                },
                err => panic!("Failed to open: {:?}", err),
            },
        };
    }
    ```

+ 使用 **`unwrap` 和 `expect`** 简化 **错误处理**, 如果失败就直接 `panic`.

    ```rust
    use std::fs::File;
    fn main() {
        let f = File::open("hello.txt").unwrap();
        let f = File::open("hello.txt").expect("Failed!");
    }
    ```

+  **`?`** 是一个 `Result` 中 **进行错误传播的宏**, 可以 **自动进行类型提升**.

    + 标准库的 **`From` 特征** 有 **方法 `from`** 用于类型转换.
    + `?` 自动调用该方法进行 **隐式类型转换**, 但只能用在 **返回值特殊** 的方法.
    + 只要 `ReturnError` 实现 `From<OtherError>` 特征, 就会自动进行转换.
    + 用一个返回类型 **覆盖所有错误类型**, 并为 **子错误类型实现这种转换**.

    ```rust
    use std::fs::File;
    use std::io::{self, Read};
    
    fn read_username_from_file() -> Result<String, io::Error> {
        let f = File::open("hello.txt");
        let mut f = match f {
            Ok(file) => file,
            Err(e) => return Err(e), // 向上传播
        };
        let mut s = String::new();
        match f.read_to_string(&mut s) {
            Ok(_) => Ok(s),
            Err(e) => Err(e), // 向上传播
        }
    }
    ------------------------------------------------------
    use std::fs::File;
    use std::io;
    use std::io::Read;
    
    fn read_username_from_file() -> Result<String, io::Error> {
        let mut f = File::open("hello.txt")?;
        let mut s = String::new();
        f.read_to_string(&mut s)?;
        Ok(s)
    }
    ------------------------------------------------------
    use std::fs::File;
    use std::io;
    use std::io::Read;
    
    fn read_username_from_file() -> Result<String, io::Error> {
        let mut s = String::new();
        File::open("hello.txt")?.read_to_string(&mut s)?;
        Ok(s)
    }
    ```

+ **`?`** 也可以在 `Option` 中 **进行传播**, 进行 `None` 的返回.

    ```rust
    fn last_char_of_first_line(text: &str) -> Option<char> {
        text.lines().next()?.chars().last()
    }
    ```

+ `map` 和 `and_then` 是常用 **组合器**, 可用于 `Result` 与 `Option`.

    ```rust
    pub fn and_then<U, F>(self, op: F) -> Result<U, E>
    	where F: FnOnce(T) -> Result<U, E>,
    
    pub fn map<U, F>(self, op: F) -> Result<U, E>
    	where F: FnOnce(T) -> U,
    ```

    一些 **应用实例**:

    ```rust
    use std::num::ParseIntError;
    
    fn add_two(n_str: &str) -> Result<i32, ParseIntError> {
    	n_str.parse::<i32>().map(|num| num + 2)
    }
    
    fn add_three(n_str: &str) -> Result<i32, ParseIntError> {
        n_str.parse::<i32>().and_then(|num| Ok(num + 3))
    }
    
    fn main() {
        assert_eq!(add_two("4").unwrap(), 6);
        assert_eq!(add_three("5").unwrap(), 8);
    }
    ------------------------------------------------------
    use std::num::ParseIntError;
    
    fn multiply(n1_str: &str, n2_str: &str) -> Result<i32, ParseIntError> {
        n1_str.parse::<i32>().and_then(|n1| {
            n2_str.parse::<i32>().map(|n2| n1 * n2)
        })
    }
    
    fn main() {
        let twenty = multiply("10", "2");
    }
    ```

+ 为 `Result` 使用 **类型别名** 来 **简化代码**.

    ```rust
    use std::num::ParseIntError;
    type Res<T> = Result<T, ParseIntError>;
    
    fn multiply(n1_str: &str, n2_str: &str) -> Res<i32> {
    	// ...    
    }
    ```

+ `main` 函数可以 **返回 `Result` 类型**, 发生错误时会 **返回该错误并打印信息**.

    ```rust
    fn main() -> Result<(), Box<dyn std::error::Error>> {
        let n_str = "10";
        let num = match n_str.parse::<i32>() {
            Ok(num)  => num,
            Err(e) => return Err(Box::new(e))
        };
        println!("{}", num);
        Ok(())
    }
    ```



## Crate 和 Package

+ `Crate` 是 **独立的可编译单元**, 编译后生成 **可执行文件** 或者 **库**, 使用 `use` 引入.

+ `Package` 包含 **独立的 `Cargo.toml` 文件**, 以及 **组织在一起的数个包**.

+ **库类型** `Package` 只能作为 **第三方库** 被其它项目引用, 而 **不能独立运行**.

+ **典型** 的 `Package` 会包含 **多个二进制包**:

    ```shell
    ├── Cargo.toml
    ├── Cargo.lock
    ├── src
    │   ├── main.rs
    │   ├── lib.rs
    │   └── bin
    │       └── main1.rs
    │       └── main2.rs
    ├── tests
    │   └── some_integration_tests.rs
    ├── benches
    │   └── simple_bench.rs
    └── examples
        └── simple_example.rs
    ```

___

## 模块 Module

+ 如果模块 `A` 包含模块 `B`, 那么 `A` 是 `B` 的 **父模块**, `B` 是 `A` 的 **子模块**.

    ```rust
    /* src/lib.rs */
    mod front_of_house {
        mod hosting {
            fn add_to_waitlist() {}
            fn seat_at_table() {}
        }
        mod serving {
            fn take_order() {}
            fn serve_order() {}
            fn take_payment() {}
        }
    }
    
    pub fn eat_at_restaurant() {
        // 绝对路径
        crate::front_of_house::hosting::add_to_waitlist();
        // 相对路径
        front_of_house::hosting::add_to_waitlist();
    }
    ```

    其中 `front_of_house` 是 `hosting` 和 `serving` 的 **父模块**. 对应的 **模块树**:

    ```rust
    crate
     └── eat_at_restaurant
     └── front_of_house
         ├── hosting
         │   ├── add_to_waitlist
         │   └── seat_at_table
         └── serving
             ├── take_order
             ├── serve_order
             └── take_payment
    ```

+ 模块定义了 **代码的私有化边界**, **父模块无法访问** 子模块中的私有项, 但 **子模块可以访问** 父模块等的私有项. 进行修改:

    ```rust
    mod front_of_house {
        pub mod hosting {
            pub fn add_to_waitlist() {}
        }
    }
    
    pub fn eat_at_restaurant() { /*--- snip ----*/ }
    ```

+ `super` 代表 **父模块为开始** 的引用方式, 类似文件系统中的 `../a/b` 语法.

    ```rust
    fn serve_order() {}
    
    mod back_of_house {
        fn fix_incorrect_order() {
            cook_order();
            super::serve_order();
        }
        fn cook_order() {}
    }
    ```

+ `self` 直接 **引用自身模块** 中的项, 类似文件系统中的 `./a/b` 语法.

    + `use self::x` 表示加载当前模块中的 `x`, 此时 `self` 可省略.
    + `use x::{self, y}` 表示加载当前路径下模块 `x` 本身以及其下的 `y`.

    ```rust
    fn serve_order() {
        self::back_of_house::cook_order()
    }
    
    mod back_of_house {
        fn fix_incorrect_order() {
            cook_order();
            crate::serve_order();
        }
        pub fn cook_order() {}
    }
    ```

+ 将 **结构体** 设置为 `pub`, 它的 **所有字段依然私有**.

+ 将 **枚举** 设置为 `pub`, 它的 **所有字段将对外可见**.

+ 当 **模块有较多子模块时**, 可以通过 **文件夹的方式** 来组织子模块.

+  **`main` 函数** 应该包含的功能:

    - 解析命令行参数.
    - 初始化其它配置.
    - 调用 `lib.rs` 中的 `run` 函数, 启动逻辑代码运行.
    - 如果 `run` 返回错误, 需要进行错误处理.
