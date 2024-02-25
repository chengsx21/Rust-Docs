# Rust 学习笔记 - 4

___

## 方法 Method

+ 使用 `impl` 为**结构体、枚举或特征**定义方法.

    ```rust
    struct Circle {
        x: f64,
        y: f64,
        radius: f64,
    }
    
    impl Circle {
        // new 是关联函数, 因为第一个参数不是self, 且 new 不是关键字
        // 往往用于初始化当前结构体的实例
        fn new(x: f64, y: f64, radius: f64) -> Circle {
            Circle {
                x: x,
                y: y,
                radius: radius,
            }
        }
        // area 是方法, &self 表示借用当前 Circle 结构体
        fn area(&self) -> f64 {
            std::f64::consts::PI * (self.radius * self.radius)
        }
    }
    ```

    ```rust
    #[derive(Debug)]
    enum TrafficLightColor {
        Red,
        Yellow,
        Green,
    }
    impl TrafficLightColor {
    	// Rust 支持自动引用与解引用
        fn color(&self) -> String {
            match self {
                Self::Red => "red".to_string(),
                Self::Yellow => "yellow".to_string(),
                Self::Green => "green".to_string(),
            }
        }
    }
    
    fn main() {
        // assert_eq! 调用类型的 PartialEq 特征
    	// String 实现了对 str 和 &str 类型的 PartialEq 特征
        let c = TrafficLightColor::Yellow;
        assert_eq!(c.color(), "yellow");
        println!("{:?}",c);
    }
    ```

+ `impl` 块内, **`Self`** 指代被实现方法的**结构体类型**, **`self`** 指代此类型的**实例**.

    + `self` 表示所有权转移到该方法中.
    + `&self` 表示该方法对实例的不可变借用.
    + `&mut self` 表示可变借用.

    ```rust
    #[derive(Debug)]
    struct TrafficLight {
        color: String,
    }
    
    impl TrafficLight {
        pub fn new() -> Self {
            Self {
                color: "red".to_string()
            }
        }
        pub fn get_state(&self) -> &str {
            &self.color
        }
    }
    
    fn main() {
        let light = TrafficLight::new();
        assert_eq!(light.get_state(), "red");
    }
    ```

+ Rust 支持**自动引用和解引用**, 当使用 `object.something()` 调用方法时, 会自动为 `object` 添加 `&`, `&mut` 或 `*` 以便**与方法签名匹配**:

    ```rust
    // 两种等价写法
    p1.distance(&p2);
    (&p1).distance(&p2);
    ```

    这种自动引用行为之所以有效, 是因为方法有一个**明确的接收者 `self` 的类型**.

___

## 泛型 Generics

+ **特征**的目的是让**类型实现相应的功能**.

+ **函数**中使用泛型:

    ```rust
    fn add<T: std::ops::Add<Output = T>>(a:T, b:T) -> T {
        a + b
    }
    ------------------------------------------------------
    // 后置式泛型特征, 使用 where 关键字
    fn add<T>(a:T, b:T) -> T 
    where 
        T: std::ops::Add<Output = T> 
    {
        a + b
    }
    ------------------------------------------------------
    fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
        let mut largest = list[0];
        for &item in list.iter() {
            if item > largest {
                largest = item;
            }
        }
        largest
    }
    ------------------------------------------------------
    struct A;         		 // 具体的类型 `A`.
    struct S(A);      		 // 具体的类型 `S`.
    struct SGen<T>(T);		 // 泛型 `SGen`.
    fn reg_fn(_s: S) {}
    fn gen_spec_t(_s: SGen<A>) {}
    fn gen_spec_i32(_s: SGen<i32>) {}
    fn generic<T>(_s: SGen<T>) {}
    
    fn main() {
        // 非泛型函数
        reg_fn(S(A));          // 具体的类型
        gen_spec_t(SGen(A));   // 隐式地指定类型参数  `A`.
        gen_spec_i32(SGen(2)); // 隐式地指定类型参数`i32`.
        // 显式指定类型参数
        generic::<char>(SGen('a'));
        // 隐式指定类型参数
        generic(SGen('a'));
    }
    ```

+ **结构体**中使用泛型:

    ```rust
    struct Point<T,U> {
        x: T,
        y: U,
    }
    fn main() {
        let p = Point{x: 1, y :1.1};
    }
    ```

+ **枚举**中使用泛型:

    ```rust
    // 常用于函数的返回值类型
    enum Option<T> {
        Some(T),
        None,
    }
    ------------------------------------------------------
    enum Result<T, E> {
        Ok(T),
        Err(E),
    }
    ```

+ **方法**中使用泛型:

    ```rust
    struct Point<T> {
        x: T,
        y: T,
    }
    // impl<T> 是泛型声明, Point<T> 是完整的结构体类型
    impl<T> Point<T> {
        fn x(&self) -> &T {
            &self.x
        }
    }
    // 针对特定的具体类型进行方法定义
    impl Point<f32> {
        fn distance_from_origin(&self) -> f32 {
            (self.x.powi(2) + self.y.powi(2)).sqrt()
        }
    }
    fn main() {
        let p = Point { x: 5, y: 10 };
        println!("p.x = {}", p.x());
    }
    ------------------------------------------------------
    struct Point<T, U> {
        x: T,
        y: U,
    }
    // 方法中定义额外的泛型参数
    impl<T, U> Point<T, U> {
        fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
            Point {
                x: self.x,
                y: other.y,
            }
        }
    }
    fn main() {
        let p1 = Point { x: 5, y: 10.4 };
        let p2 = Point { x: "Hello", y: 'c'};
        let p3 = p1.mixup(p2);
        println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
    }
    ```

+ **`const` 泛型**是**针对值的泛型**, 可以用于处理数组长度的问题.

    ```rust
    fn display<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {
        println!("{:?}", arr);
    }
    fn main() {
        let arr: [i32; 3] = [1, 2, 3];
        display(arr);
    
        let arr: [i32; 2] = [1, 2];
        display(arr);
    }
    ```

+ **`const` 泛型参数**只能使用以下形式的**实参**:

    + **单独的 `const` 泛型参数.**
    + **字面量.**
    + **`const` 表达式 (不含泛型参数).**

    ```rust
    fn foo<const N: usize>() {}
    
    fn bar<T, const M: usize>() {
        foo::<M>(); // ok: 第一种
        foo::<2021>(); // ok: 第二种
        foo::<{20 * 100 + 20 * 10 + 1}>(); // ok: 第三种
        
        foo::<{ M + 1 }>(); // error: 违背第三种
        foo::<{ std::mem::size_of::<T>() }>(); // error: 违背第三种
    
        let _: [u8; M]; // ok: 第一种
        let _: [u8; std::mem::size_of::<T>()]; // error: 违背第三种
    }
    ```

+ **`const` 泛型表达式**在编译时就能够确定出结果, 可以**限制函数参数占用的内存大小**.

    ```rust
    // 目前只能在 nightly 版本下使用
    #![allow(incomplete_features)]
    #![feature(generic_const_exprs)]
    
    pub enum Assert<const CHECK: bool> {}
    pub trait IsTrue {}
    impl IsTrue for Assert<true> {}
    
    fn something<T>(val: T)
    where // 这里是一个 const 表达式
        Assert<{ core::mem::size_of::<T>() < 768 }>: IsTrue,
    {
        //
    }
    
    fn main() {
        something([0u8; 0]); // ok
        something([0u8; 512]); // ok
        something([0u8; 1024]); // 编译错误, 数组长度超过参数长度限制
    }
    ```

+ 泛型是**零成本的抽象**, 使用泛型时**没有运行时开销**, 获得了性能上的巨大优势, 但损失了**编译速度**和增大了**最终生成文件的大小**; 编译时进行泛型代码的**单态化**来保证效率.

___

## 特征 Trait

