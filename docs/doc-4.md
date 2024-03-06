# 泛型与特征

___

## 方法 Method

+ 使用 `impl` 为 **结构体、枚举或特征** 定义方法.

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

+ `impl` 块内, **`Self`** 指代被实现方法的 **结构体类型**, **`self`** 指代此类型的 **实例**.

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

+ Rust 支持 **自动引用和解引用**, 当使用 `object.something()` 调用方法时, 会自动为 `object` 添加 `&`, `&mut` 或 `*` 以便 **与方法签名匹配**:

    ```rust
    // 两种等价写法
    p1.distance(&p2);
    (&p1).distance(&p2);
    ```

    这种自动引用行为之所以有效, 是因为方法有一个 **明确的接收者 `self` 的类型**.

___

## 泛型 Generics

+ **特征** 的目的是让 **类型实现相应的功能**.

+ **函数** 中使用泛型:

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

+ **结构体** 中使用泛型:

    ```rust
    struct Point<T,U> {
        x: T,
        y: U,
    }
    fn main() {
        let p = Point{x: 1, y :1.1};
    }
    ```

+ **枚举** 中使用泛型:

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

+ **方法** 中使用泛型:

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

+ **`const` 泛型** 是 **针对值的泛型**, 可以用于处理数组长度的问题.

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

+ **`const` 泛型参数** 只能使用以下形式的 **实参**:

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

+ **`const` 泛型表达式** 在编译时就能够确定出结果, 可以 **限制函数参数占用的内存大小**.

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

+ 泛型是 **零成本的抽象**, 使用泛型时 **没有运行时开销**, 获得了性能上的巨大优势, 但损失了 **编译速度** 和增大了 **最终生成文件的大小**; 编译时进行泛型代码的 **单态化** 来保证效率.

___

## 特征 Trait

+ 特征定义了 **一组被共享的行为**, 只要实现了特征, 就能使用这组行为; **定义特征** 是把一些方法组合在一起, 定义一个实现某些目标 **所必需的行为集合**.

+ 特征只定义行为 **看起来是什么样**, 只 **定义特征方法签名** 而 **不进行实现**; 每一个实现这个特征的类型都需要具体实现相应方法.

    ```rust
    pub trait Summary {
        fn summarize(&self) -> String;
    }
    pub struct Post {
        pub title: String,
        pub author: String,
        pub content: String,
    }
    impl Summary for Post {
        fn summarize(&self) -> String {
            format!("文章{}, 作者是{}", self.title, self.author)
        }
    }
    ------------------------------------------------------
    struct Foo;
    struct Bar;
    
    #[derive(PartialEq, Debug)]
    struct BarFoo;
    
    impl std::ops::Sub<Foo> for Bar {
        type Output = BarFoo;
        fn sub(self, _rhs: Foo) -> BarFoo {
            BarFoo
        }
    }
    ```

+ **孤儿规则**: 为类型 `A` 实现特征 `T`, 那么 **`A` 或者 `T` 至少有一个在当前作用域中定义**, 确保其它人编写的代码不会破坏你的代码.

+ 在特征中定义具有 **默认实现** 的方法, 其它类型无需实现, 也可以重载该方法; 默认实现 **允许调用相同特征中的其他方法**, 哪怕这些方法没有默认实现.

    ```rust
    pub trait Summary {
        fn summarize_author(&self) -> String;
        fn summarize(&self) -> String {
            format!("(Read more {}...)", self.summarize_author())
        }
    }
    
    impl Summary for Weibo {
        fn summarize_author(&self) -> String {
            format!("@{}", self.username)
        }
    }
    ```

+ **特征约束**: 特征可以作为 **函数参数**, 在函数体内调用该特征的方法.

    ```rust
    // 特征约束
    fn func(item: &impl Summary) {}
    
    fn func<T: Summary>(item: &T) {}
    ------------------------------------------------------
    // 多重约束
    fn func(item: &(impl Summary + Display)) {}
    
    fn func<T: Summary + Display>(item: &T) {}
    ------------------------------------------------------
    // Where 约束
    fn func<T: Display + Clone, U: Debug>(t: &T, u: &U) -> i32 {}
    
    fn func<T, U>(t: &T, u: &U) -> i32
    where
    	T: Display + Clone, U: Clone + Debug
    {}
    ```

+ **特征约束** 可以在 **指定类型 + 指定特征** 的条件下去 **实现方法**.

    ```rust
    // `PartialOrd` trait 依赖于 `PartialEq` trait
    #[derive(Debug, PartialEq, PartialOrd)]
    struct Unit(i32);
    
    struct Pair<T> {
        x: T,
        y: T,
    }
    impl<T: Debug + PartialOrd> Pair<T> {
        fn cmp_display(&self) {
            if self.x >= self.y {
                println!("The largest member x = {:?}", self.x);
            }
        }
    }
    
    fn main() {
        let pair = Pair{ x: Unit(1), y: Unit(3) };
        pair.cmp_display();
    }
    ```

+ **特征约束** 中使用 **`Fn` 特征** 支持 **接受闭包类型作为参数**.

    ```rust
    struct Cacher<T>
        where T: Fn(u32) -> u32,
    {
        calculation: T,
        value: Option<u32>,
    }
    
    impl<T> Cacher<T>
        where T: Fn(u32) -> u32,
    {
        fn new(calculation: T) -> Cacher<T> {
            Cacher {
                calculation,
                value: None,
            }
        }
    
        fn value(&mut self, arg: u32) -> u32 {
            match self.value {
                Some(v) => v,
                None => {
                    let v = (self.calculation)(arg);
                    self.value = Some(v);
                    v
                },
            }
        }
    }
    
    let mut cacher = Cacher::new(|x| x+1);
    assert_eq!(cacher.value(20), 21);
    assert_eq!(cacher.value(25), 21);
    ```

+ **特征** 通过 `impl Trait` 作为 **函数返回值,** 声明 **返回的类型实现了某个特征**; 当函数返回多个类型时, 需要引入 **特征对象**.

    ```rust
    fn returns_summarizable() -> impl Summary {
        Weibo {
            username: String::from("sunface"),
            content: String::from(
                "m1 max太厉害了，电脑再也不会卡",
            )
        }
    }
    ```

+ **`derive` 派生特征** 的对象 **自动实现对应的默认特征代码**, 还可以手动重载实现.

+ **调用特征方法** 需要将特征 **引入当前作用域** (**运算符** 是 **特征方法调用** 的语法糖).

    ```rust
    use std::convert::TryInto;
    
    fn main() {
        let a: i32 = 10;
        let b: u16 = 100;
        let c = b.try_into().unwrap();
        if a < c {
        println!("Ten is less than one hundred.");
        }
    }
    ```

___

## 特征对象 Trait Object

+ **特征对象** 指向 **实现了某特征的类型实例**, 这种映射关系 **存储在一张表中**, 在 **运行时** 通过特征对象 **找到具体调用的类型方法**.

+ **特征对象** 大小不固定, 但其 **引用类型** 由指针 `ptr` 和 `vptr` 组成, **大小固定**.

+ 通过 **`&` 引用** 或者 **`Box<T>` 智能指针** 的方式创建特征对象; **`dyn` 关键字** 只用在特征对象的 **类型声明** 上, 在创建时无需使用.

    ```rust
    trait Draw {
        fn draw(&self) -> String;
    }
    impl Draw for u8 {
        fn draw(&self) -> String {
            format!("u8: {}", *self)
        }
    }
    impl Draw for f64 {
        fn draw(&self) -> String {
            format!("f64: {}", *self)
        }
    }
    
    // 实现 Draw 特征, 传入的 Box<T> 可隐式转换成 Box<dyn Draw>
    // 实现 Deref 特征, Box 智能指针会自动解引用为包裹的值
    fn draw1(x: Box<dyn Draw>) {
        x.draw();
    }
    fn draw2(x: &dyn Draw) {
        x.draw();
    }
    
    fn main() {
        let x = 1.1f64;
        let y = 8u8;
        draw1(Box::new(x));
        draw1(Box::new(y));
        draw2(&x);
        draw2(&y);
    }
    ```

+ **特征对象** 的优势是 **无需在运行时检查一个值是否实现了特定方法**, 否则会 **产生编译期错误**.

+ **静态分发 (static dispatch)**: 泛型是在 **编译期完成处理**, 编译器为每个泛型参数对应的具体类型生成一份代码, 对于 **运行期性能完全没有影响**.

+ **动态分发 (dynamic dispatch)**: 关键字 `dyn` 在 **运行时确定** 需要调用什么方法.

+ **特征对象的安全性**: 方法 **返回类型不能是 `Self`**; 方法 **没有泛型参数** (特征对象不关心实现该特征的具体类型).

___

## 深入了解特征 Dive into Trait

+ **关联类型** 是在特征定义的语句块中申明的 **自定义类型**, 可以在 **特征的方法签名中** 使用该类型, 目的是 **代码的可读性** (所以 **不使用泛型**).

    ```rust
    pub trait Iterator {
        type Item;
        // 通过 Self::type 使用该类型
        fn next(&mut self) -> Option<Self::Item>;
    }
    
    impl Iterator for Counter {
        type Item = u32;
        fn next(&mut self) -> Option<Self::Item> {
            // --snip--
        }
    }
    
    fn main() {
        let c = Counter{..}
        c.next()
    }
    ------------------------------------------------------
    trait Contains<A, B> {
        fn contains(&self, _: &A, _: &B) -> bool;
        fn first(&self) -> i32;
        fn last(&self) -> i32;
    }
    
    impl Contains<i32, i32> for Container {
        fn contains(&self, a: &i32, b: &i32) -> bool {
            (&self.0 == a) && (&self.1 == b)
        }
        fn first(&self) -> i32 { self.0 }
        fn last(&self) -> i32 { self.1 }
    }
    
    fn difference<A, B, C: Contains<A, B>>(container: &C) -> i32 {
        container.last() - container.first()
    }
    ------------------------------------------------------
    trait Contains {
        type A;
        type B;
        fn contains(&self, _: &Self::A, _: &Self::B) -> bool;
        fn first(&self) -> i32;
        fn last(&self) -> i32;
    }
    
    impl Contains for Container {
        type A = i32;
        type B = i32;
        fn contains(&self, a: &Self::A, b: &Self::B) -> bool {
            (&self.0 == a) && (&self.1 == b)
        }
        fn first(&self) -> i32 { self.0 }
        fn last(&self) -> i32 { self.1 }
    }
    
    fn difference<C: Contains>(container: &C) -> i32 {
        container.last() - container.first()
    }
    ```

+ 特征 **使用泛型类型参数** 时, 可以 **指定一个默认的具体类型**.

    ```rust
    trait Add<RHS=Self> {
        type Output;
        fn add(self, rhs: RHS) -> Self::Output;
    }
    ------------------------------------------------------
    #[derive(Debug, PartialEq)]
    struct Point {
        x: i32,
        y: i32,
    }
    
    impl Add for Point {
        type Output = Point;
        fn add(self, other: Point) -> Point {
            Point {
                x: self.x + other.x,
                y: self.y + other.y,
            }
        }
    }
    ------------------------------------------------------
    struct Millimeters(u32);
    struct Meters(u32);
    
    impl Add<Meters> for Millimeters {
        type Output = Millimeters;
        fn add(self, other: Meters) -> Millimeters {
            Millimeters(self.0 + (other.0 * 1000))
        }
    }
    ```

+ **类型和特征** 定义了 **同名方法** 时, 编译器 **默认调用类型中定义的方法**, **调用特征的方法** 需要使用 **显式调用语法**.

    ```rust
    let person = Human;
    person.fly(); // 调用 Human 类型自身的方法
    Pilot::fly(&person); // 调用 Pilot 特征上的方法
    Wizard::fly(&person); // 调用 Wizard 特征上的方法
    ```

+ **方法没有 `self` 参数 (关联函数)** 时, 需要使用 **完全限定语法** `<Type as Trait>::function(receiver_if_method, next_arg, ...)`.

    ```rust
    Dog::baby_name(); // 调用 Dog 类型自身的方法
    <Dog as Animal>::baby_name(); // 调用 Animal 特征上的方法
    ```

+ **`supertrait`**: 需要让特征 A 使用特征 B 的功能 (另一种形式的 **特征约束**), 要 **为类型实现特征 A 和特征 B**.

    ```rust
    use std::fmt::Display;
    
    trait OutlinePrint: Display {
        fn outline_print(&self) {
            let output = self.to_string();
            let len = output.len();
            println!("{}", "*".repeat(len + 4));
            println!("*{}*", " ".repeat(len + 2));
            println!("* {} *", output);
            println!("*{}*", " ".repeat(len + 2));
            println!("{}", "*".repeat(len + 4));
        }
    }
    ```

+ **`newtype` 模式**: 绕过 **孤儿规则**, 创建一个 **元组结构体** 新类型, 该结构体 **封装了希望实现特征的具体类型**, 相当于 **一层装饰器**.

    ```rust
    // 为 Vec<T> 实现 Display 特征
    // 没有什么是加一层解决不了的, 如果不行那就再加一层
    struct Wrapper(Vec<String>);
    
    impl Display for Wrapper {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            write!(f, "[{}]", self.0.join(", "))
        }
    }
    
    impl Deref for Wrapper {
    	type Target = Vec;
        fn deref(&self) -> &Self::Target {
            &self.0
        }
    }
    
    fn main() {
        let w = Wrapper(vec![1, 2, 3, 4, 5]);
    	println!("w = {}", w);
    }
    ```
