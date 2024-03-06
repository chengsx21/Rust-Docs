# 集合类型与生命周期

___

## 动态数组 Vector

+ **创建动态数组** 可以使用关联方法 `Vec::new()`, `Vec::with_capacity()` 或宏 `vec!`.

+ **向动态数组尾部添加元素** 可以使用 `push` 方法.

    ```rust
    let mut v = Vec::new();
    v.push(1);
    
    let mut v = Vec::with_capacity(20);
    
    let mut v = vec![1, 2, 3];
    ```
    
+ **从动态数组中读取元素** 可以使用下标索引访问或 `get` 方法.

    ```rust
    let v = vec![1, 2, 3, 4, 5];
    // 防止越界访问, 返回 Option<&T>
    match v.get(2) {
        Some(third) => println!("第三个元素是 {third}"),
        None => println!("去你的第三个元素"),
    }
    ```

+ **借用动态数组元素** 时不能 **进行元素插入**, 否则当 **旧数组大小不够** 时, 会 **重新分配** 一块更大的内存空间, 之前的引用会 **指向一块无效内存**.

    ```rust
    let mut v = vec![1, 2, 3, 4, 5];
    let first = &v[0];
    v.push(6);
    // println!("The first element is: {first}"); ERROR!
    ```

+ **动态数组的常用方法** 如下.

    ```rust
    let mut v = Vec::with_capacity(10);
    v.extend([1, 2, 3]); // 附加数据
    println!("长度:{}, 容量:{}", v.len(), v.capacity());
    v.reserve(100); // 调整容量
    v.shrink_to_fit(); // 释放剩余容量
    ------------------------------------------------------
    let mut v =  vec![1, 2];
    assert!(!v.is_empty()); // 检查是否为空
    v.insert(2, 3); // 指定索引插入数据
    assert_eq!(v.remove(1), 2); // 移除指定位置元素并返回
    assert_eq!(v.pop(), Some(3)); // 删除并返回尾部元素
    assert_eq!(v.pop(), Some(1)); 
    assert_eq!(v.pop(), None);
    v.clear(); // 清空
    
    let mut v1 = [11, 22].to_vec();
    v.append(&mut v1); // 清空 v1, 所有元素附加到 v 中
    v.truncate(1); // 截断到指定长度
    v.retain(|x| *x > 10); // 保留满足条件元素
    ------------------------------------------------------
    let mut v = vec![11, 22, 33, 44, 55];
    // 删除指定范围的元素, 获取被删除元素的迭代器
    let mut m: Vec<_> = v.drain(1..=3).collect();    
    let v1 = m.split_off(1); // 指定索引处切分
    ```

+ **动态数组排序算法** 分为稳定排序 `sort` 和 `sort_by`, 以及非稳定排序 `sort_unstable` 和 `sort_unstable_by`.

    ```rust
    let mut vec = vec![1, 5, 10, 2, 15];
    vec.sort_unstable();
    ------------------------------------------------------
    let mut vec = vec![1.0, 5.6, 10.3, 2.0, 15f32];    
    vec.sort_unstable_by(|a, b| a.partial_cmp(b).unwrap());
    ```

+ **浮点数** 中存在 `NAN`, 没有实现 **全可比较特性** `Ord`, 而实现了 **部分可比较特性** `PartialOrd`; **实现 `Ord` 特性** 需要实现 `Ord`, `Eq`, `PartialEq`, `PartialOrd`.

+ 为动态数组 **实现 `From<T>` 特征**, 那么 `T` 就可以被 **转换成 `Vec`**.

    ```rust
    // impl From<[T; N]> for Vec
    let arr = [1, 2, 3];
    let v1 = Vec::from(arr);
    let v2: Vec<i32> = arr.to_vec();
    assert_eq!(v1, v2);
    ------------------------------------------------------
    // impl From<String> for Vec
    let s = "hello".to_string();
    let v1: Vec<u8> = s.into();
    
    let s = "hello".to_string();
    let v2 = s.into_bytes();
    assert_eq!(v1, v2);
    ------------------------------------------------------
    // impl<'_> From<&'_ str> for Vec
    let s = "hello";
    let v3 = Vec::from(s);
    assert_eq!(v2, v3);
    ------------------------------------------------------
    // Iterators 可以通过 collect 变成 Vec
    let v4: Vec<i32> = [0; 10].into_iter().collect();
    assert_eq!(v4, vec![0; 10]);
    ```

+ **`&[T]` 切片和 `&Vec<T>`** 是不同的类型, 后者仅仅是 **`Vec` 的引用** 并可以通过 **解引用直接获取 `Vec`**.

------------------

## 哈希映射 HashMap

+ **创建 HashMap** 可使用关联方法 `HashMap::new()`, `HashMap::from()`, `HashMap::with_capacity()` 或 `collect` 方法.

    ```rust
    use std::collections::HashMap;
    let mut my_gems = HashMap::new();
    my_gems.insert("红宝石", 1);
    my_gems.insert("河边捡的误以为是宝石的破石头", 18);
    ------------------------------------------------------
    let teams = [
        ("Chinese Team", 100),
        ("American Team", 10),
        ("France Team", 50),
    ];
    // collect 方法在内部支持多种类型的目标集合, 需要进行类型标注
    let map1: HashMap<_,_> = team_list.into_iter().collect();
    
    let mut map2 = HashMap::new();
    for team in &teams {
        map2.insert(team.0, team.1);
    }
    
    let map3 = HashMap::from(teams);
    ```

+ **集合类型** 都是 **动态、没有固定的内存大小** 的, 底层数据 **存储在堆上**, 通过 **存储在栈中的引用类型** 来访问.

+ **`HashMap` 的所有权规则**:

    + 实现 `Copy` 特征 (如 `&str`), 复制进 `HashMap`, 所有权不发生转移;
    + 未实现 `Copy` 特征 (如 `String`), 所有权转移给 `HashMap` 中.

+ **引用类型** 放入 HashMap 中, 需要确保该引用的 **生命周期至少一样久**.

    ```rust
    let name = String::from("Sunface");
    let mut handsome_boys = HashMap::new();
    handsome_boys.insert(&name, 18);
    std::mem::drop(name);
    // println!("{:?}已经被除名", handsome_boys); ERROR!
    ```

+ **从 HashMap 读取元素** 使用 `get` 方法, 返回 `Option<&T>`; 或使用下标索引.

    ```rust
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    
    let name = String::from("Blue");
    let score: Option<&i32> = scores.get(&name);
    let score_: i32 = scores.get(&name).copied().unwrap_or(0);
    ```

+ **从 HashMap 中循环遍历** `Key`, `Value` 对.

    ```rust
    use std::collections::HashMap;
    
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
    ```

+ **更新 HashMap** 的方法如下.

    ```rust
    let mut scores = HashMap::new();
    scores.insert("Blue", 10);
    
    // 覆盖已有的值
    let old = scores.insert("Blue", 20);
    assert_eq!(old, Some(10));
    
    // 查询新插入的值
    let new = scores.get("Blue");
    assert_eq!(new, Some(&20));
    
    // 查询对应的值, 若不存在则插入新值
    // or_insert 返回 &mut v 引用
    let v = scores.entry("Yellow").or_insert(5);
    assert_eq!(*v, 5);
    let v = scores.entry("Yellow").or_insert(50);
    assert_eq!(*v, 5);
    ```

+ **类型作为 `Key`** 的关键是 **能否进行相等比较** (实现 **`std::cmp::Eq` 特征**).

    + **浮点数** 不可以用作 `HashMap` 的 `Key`.
    + **结构体** 作为 `HashMap` 的 `Key` 需要实现 `PartialEq`, `Eq`, `Hash` 特征.

___

## 生命周期 Lifetime

+ **生命周期** 就是 **引用的有效作用域**, 主要作用是 **避免悬垂引用**.

+ Rust 使用 **借用检查器** 来检查 **借用正确性**.

    ```rust
    {
        let r;                // ---------+-- 'a
        {                     //          |
            let x = 5;        // -+-- 'b  |
            r = &x;           //  |       |
        }                     // -+       |
        println!("r: {}", r); //          |
    }                         // ---------+
    ```

+ 函数的返回值为引用时, 编译器需要知道 **引用的具体对象**, 确保 **调用后的引用生命周期分析**; 生命周期 **标注** 并 **不会改变引用的实际作用域**.

    ```rust
    &i32         // 一个引用
    &'a i32      // 具有显式生命周期的引用
    &'a mut i32  // 具有显式生命周期的可变引用
    ```

+ 通过 **函数签名** 指定 **生命周期参数** 时, 并 **没有改变** 传入引用或者返回引用的 **真实生命周期**, 而是告诉 **编译器** 当不满足约束条件时 **拒绝编译通过**.

    ```rust
    // 生命周期标注说明参数和返回值至少和 'a 活得一样久
    // 返回值的生命周期是参数中作用域较小的那个
    fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
        if x.len() > y.len() {
            x
        } else {
            y
        }
    }
    ```

+ 如果函数 **返回引用类型**, 其生命周期来源于 **参数** 或 **函数体中某新建引用(悬垂)**.

+ **结构体使用引用**, 需要为 **每个引用标注生命周期**, **每个引用** 需要 **比结构体活得久**.

    ```rust
    struct NoCopyType {}
    struct Example<'a, 'b> {
        a: &'a u32,
        b: &'b NoCopyType
    }
    
    fn fix_me<'a>(foo: &Example<'_, 'a>) -> &'a NoCopyType {
        foo.b
    }
    
    fn main() {
        let no_copy = NoCopyType {};
        let example = Example { a: &1, b: &no_copy };
        fix_me(&example);
    }
    ```

+ 函数或方法中, 参数的生命周期称为 **`输入生命周期`**, 返回值的生命周期称为 **`输出生命周期`**. 介绍 **生命周期消除规则** (编译器发现三条规则都不适用就会报错):

    + 每个 **引用参数** 都会获得 **独自的生命周期**.
    + **只有一个输入生命周期**, 那么该生命周期会被 **赋给所有输出生命周期**.
    + **存在多个输入生命周期**, 其中一个是 **`&self` 或 `&mut self`**, 则 `&self` 的生命周期被 **赋给所有输出生命周期**.

+ 为具有生命周期的结构体 **实现方法**, 语法和 **泛型参数语法** 很相似.

    ```rust
    struct ImportantExcerpt<'a> {
        part: &'a str,
    }
    
    // 生命周期标注也是结构体类型的一部分
    impl<'a> ImportantExcerpt<'a> {
        // 方法签名往往无需标注生命周期, 得益于第一和第三消除规则
        fn level(&self) -> i32 {
            3
        }
    }
    ```

+ **生命周期约束语法** 跟泛型约束相似, 用于说明 `'a` 必须比 `'b` 活得久.

    ```rust
    impl<'a: 'b, 'b> ImportantExcerpt<'a> {
        fn rtn_part(&'a self, announcement: &'b str) -> &'b str {
            println!("Attention please: {}", announcement);
            self.part
        }
    }
    ```

+ **`'static` 生命周期** 表示和程序活得一样久 (**硬编码字符串字面量** 和 **特征对象**).

    ```rust
    fn main() {
        let result;
        {
            // 写法一 ×
            // 字符串在 `}` 结束后 drop 了, 引用失效
            // let s1 = String::from("a");
            // let s2 = String::from("abc");
            // result = longest(s1.as_str(), s2.as_str());
        
            // 写法二 √
            // 字符串字面量本身的生命周期是 `'static`
            let s1 = "abc";
            let s2 = "a";
            result = longest(s1, s2);
        }
        println!("result: {}", result);
    }
    ```

+ 综合运用实例: 实现一个 **简单的字节缓冲区**.

    ```rust
    struct Buffer<'a> {
        buf: &'a [u8],
        pos: usize,
    }
    
    impl<'b, 'a: 'b> Buffer<'a> {
        fn new(b: &'a [u8]) -> Buffer {
            Buffer {
                buf: b,
                pos: 0,
            }
        }
    
        fn read_bytes(&'b mut self) -> &'a [u8] {
            self.pos += 3;
            &self.buf[self.pos-3..self.pos]
        }
    }
    
    fn print(b1 :&[u8], b2: &[u8]) {
        println!("{:#?} {:#?}", b1, b2)
    }
    
    fn main() {
        let v = vec![1, 2, 3, 4, 5, 6];
        let mut buf = Buffer::new(&v);
        let b1 = buf.read_bytes();
        let b2 = buf.read_bytes();
        print(b1, b2)
    }
    ```
