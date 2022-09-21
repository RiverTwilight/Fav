> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7061556434692997156)

本文以 [Typescript 4.5](https://link.juejin.cn?target=https%3A%2F%2Fwww.typescriptlang.org%2Fdocs%2Fhandbook%2Frelease-notes%2Ftypescript-4-5.html "https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html") 及以上版本为基础，于 2022 年 02 月 07 日在掘金首发

本文要实现一种类型工具

`type result = Add<"9007199254740991", "9007199254740991">`

能计算出两个数字字符串类型的和，即 `"18014398509481982"`

本文代码见：[github.com/kawayiLinLi…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FkawayiLinLin%2Ftypescript-lodash%2Ftree%2Fmaster "https://github.com/kawayiLinLin/typescript-lodash/tree/master")

中文文档：[kawayilinlin.github.io/typescript-…](https://link.juejin.cn?target=https%3A%2F%2Fkawayilinlin.github.io%2Ftypescript-lodash%2F "https://kawayilinlin.github.io/typescript-lodash/")

作为一名花里胡哨的前端，看到最近这不是冬奥会开始了，咱们得为运动健儿们呐喊助威，还想着能不能当一次前端届的体操运动员，和奥运健儿们一起感受在冬天运动的魅力

下面，我们就开始做操吧！（如果你不想看这么多，欢迎到评论区点在线尝试的链接去尝试一下吧）

TS 类型体操基本原理
-----------

### if 和 else

条件类型，条件类型冒号左边为 `if` 右边为 `else`

```
type A = 1
type B = 2
type Example = A extends B ? true : false // false
复制代码

```

`type Example = A extends B ? true : false` 中的 `true` 和 `false` 即可以理解成它们分别为 `if` 分支和 `else` 分支中要写的代码

而 `if` 中的条件即为 `A extends B`，`A` 是否可以分配给 `B`

要实现 `else if` 则需要多个这样的条件类型进行组合

### 模式匹配

```
type A = [1, 2, 3]
type ExampleA = A extends [infer First, ...infer Rest] ? First : never // 1
type B = "123"
type ExampleB = B extends `${infer FirstChar}${infer Rest}` ? FirstChar : never // '1'
复制代码

```

模式匹配是我们要利用的最有用的 `ts` 特性之一，之后我们要实现的字符串的增删改查和元组的增删改查都要基于它

如果你想知道更多，可以参考这篇文章：[模式匹配 - 让你 ts 类型体操水平暴增的套路](https://juejin.cn/post/7045536402112512007 "https://juejin.cn/post/7045536402112512007")

关于条件类型中 `infer` 的官方文档：[Inferring Within Conditional Types](https://link.juejin.cn?target=https%3A%2F%2Fwww.typescriptlang.org%2Fdocs%2Fhandbook%2F2%2Fconditional-types.html%23inferring-within-conditional-types "https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types")

### 与或非

基于条件类型可以轻松实现与或非

```
// C 意为 Condition，条件
// common
// 与，即 C1，C2 同为真
type And<C1 extends boolean, C2 extends boolean> = C1 extends true
  ? C2 extends true
    ? true
    : false
  : false

// common
// 与，即 C1，C2 有一个为真
type Or<C1 extends boolean, C2 extends boolean> = C1 extends true
  ? true
  : C2 extends true
  ? true
  : false

// common
// 非，即反转 C 的真假状态
type Not<C extends boolean> = C extends true ? false : true
复制代码

```

`ts` 目前不支持动态个数的泛型参数，因此如果有多个条件，我们需要定义多个不同的，比如

```
// common
// 有三个条件的情况
type And3<C1 extends boolean, C2 extends boolean, C3 extends boolean> = And<
  And<C1, C2>,
  C3
>

// common
// 有四个条件的情况
type And4<
  C1 extends boolean,
  C2 extends boolean,
  C3 extends boolean,
  C4 extends boolean
> = And<And3<C1, C2, C3>, C4>
复制代码

```

现在，我们已经封装了若干个类型工具 `And Or Not`，要达成基于 `ts` 的类型系统实现加法器的目标

我们需要很多个这样的类型工具

为了方便管理，我们需要给它分模块，比如上面的与或非，我们划分在 `common` 里

我们还需要 `function、array、number、object、string` 这另外的五个，用于处理函数类型、元组类型、数字类型、对象类型、字符类型

### 判断相等

在 js 的运算操作符中，有 `==` 和 `===`

在 ts 类型系统中，也可以实现类似的判断

```
// common
// 判断左侧类型是否可以分配给右侧类型
type CheckLeftIsExtendsRight<T extends any, R extends any> = T extends R
  ? true
  : false

// common
// 判断左侧类型是否和右侧类型一致
type IsEqual<A, B> = (<T>() => T extends A ? 1 : 2) extends <
  T1
>() => T1 extends B ? 1 : 2
  ? true
  : false
复制代码

```

`CheckLeftIsExtendsRight` 即校验左侧类型是否可分配给右侧类型，和 `==` 不同的是，`==` 会进行类型转换后的比较，而条件类型 `Left extends Right ? xxx : xxx` 只会进行结构性兼容的检查

如

```
type Example1 = { a: 1; b: 2 } extends { a: 1 } ? true : false // true
type Example2 = 1 | 2 extends 1 ? true : false // true
复制代码

```

虽然两个类型长的不一样，但是可以通过约束校验

`IsEqual` 参考 [github - typescript issue：[Feature request]type level equal operator](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fmicrosoft%2FTypeScript%2Fissues%2F27024%23issuecomment-510924206 "https://github.com/microsoft/TypeScript/issues/27024#issuecomment-510924206")

### toString

要实现 ts 的数学运算过于麻烦，因为数字是无法进行 infer 的，如何判断它（一个数字类型）为整型，浮点型？还是正数，或者负数？是不是仅仅有一个数字类型没有任何办法？

这都需要基于字符类型（或元组类型）的模式匹配

```
// string
// 将类型转为字符串有一定的限制，仅支持下面的类型
type CanStringified = string | number | bigint | boolean | null | undefined

// string
// 将支持的类型转化为字符串
type Stringify<T extends CanStringified> = `${T}`
复制代码

```

效果

```
type Example1 = Stringify<0> // "0"

type Example2 = Stringify<-1> // "-1"

type Example3 = Stringify<0.1> // "0.1"

type Example4 = Stringify<"0.2"> // "0.2"
复制代码

```

### 循环

在 js 中，我们可以通过 `for、while、do...while` 等循环进行可迭代对象的遍历，这些都离不开一个东西，那就是循环条件

比如在 for 循环中 `for (初始化; 条件; 循环后逻辑)` 我们一般用一个变量 `i` ，每一次循环后进行自增，循环条件一般是将 `i` 与另一个数比较大小

那么我们还需要实现一个数字类型大小比较，数字类型累加的工具类型

`ts` 中的循环可以通过递归来实现，`ts` 的类型系统中不存在类型赋值的概念

```
type Example = 1
Example = 2 // 没有这种写法
复制代码

```

只有通过每次递归时，把当前泛型参数处理后，当做下一次递归的泛型参数，终止递归时，返回当前某一个泛型参数的类型

通过一个最简单的递归类型例子来看一下这个过程

```
type Example<
  C extends boolean = true,
  Tuple extends unknown[] = [1]
> = C extends true ? Example<false, [...Tuple, 1]> : Tuple

type Result = Example // [1, 1]

// Example 的两个泛型参数
复制代码

```

如上示例，`Result` 得到的类型是 `[1, 1]`

第一次：C 是默认类型 true， 则会走到 `Example<false, [...Tuple, 1]>`，其中 `[...Tuple, 1]` 的结果为 `[...[1], 1]`，即 `[1, 1]`

第二次：C 传入了 false，会走到 `Tuple`，Tuple 的值为上次传入的值 `[1, 1]`，最后的返回类型为 `[1, 1]`

除了递归，还有两种方式可以循环，一种是分布式条件类型，还有一种是映射类型，但是它们都很难传递类型

```
// 分布式条件类型，当泛型参数 T 为联合类型时，条件类型即为分布式条件类型，会将 T 中的每一项分别分发给 extends 进行比对
type Example1<T> = T extends number ? T : never

type Result1 = Example1<"1" | "2" | 3 | 4> // 3 | 4

// 映射类型，固定写法，in 操作符会分发 T 成为新对象类型的键
type Example2<T> = {
  [Key in T]: Key
}
type Result2 = Example2<"1" | "2" | 3 | 4> // { 1: "1"; 2: "2"; 3: 3; 4: 4; }
复制代码

```

### 基本的数学运算

#### 判断正负

在部分场景下，我们可以兼容 number 类型的数字，也可以兼容 string 类型的数字，定义其为 `NumberLike`

```
type NumberLike = number | `${number}`
// 可分配给 NumberLike 的类型示例：number、`${number}`、1、-1、0.1、-0.1、"1"、"-1" 等
复制代码

```

判断为 0

```
// N 意为 Number 数字
// number
// number类型是否为0，判断 N 是否可分配给 0 | "0"
type IsZero<N extends NumberLike> = common.CheckLeftIsExtendsRight<N, 0 | "0">

// number
// number类型是否大于0，泛型类型有限制 NumberLike，所以它一定是个数或者由数字构成的字符串，将其转为字符串后，判断最前面是不是 -，如果不是，就是大于零
type IsOverZero<N extends NumberLike> = IsZero<N> extends true
  ? false
  : common.CheckLeftIsExtendsRight<
      string.Stringify<N> extends `${"-"}${infer Rest}` ? Rest : never,
      never
    >

// number
// number类型是否小于0，对上面 IsOverZero 的结果取反
type IsLessZero<N extends NumberLike> = common.Not<IsOverZero<N>>
复制代码

```

#### 两数相加

在上面 `循环` 章节，我们讲到了，可以通过递归传递修改后的泛型参数，来创建复杂的工具类型

此场景下，我们可以生成动态的类型，最常见的有这几种，元组类型，模板字符串类型，联合类型

而元组类型的长度是可以访问的 如 `[0, 1, 2]['length']` 结果为 `3`，且元组类型是可以拼接的，如 `[...[0, 1, 2], ...[0]]['length']` 的长度为 `4`

那么我们可以动态生成两个指定长度的元组类型，然后拼接到一起，获取拼接后的元组长度，就可以得到正整数（和 0）的加法了

参考：[juejin.cn/post/705089…](https://juejin.cn/post/7050893279818317854#heading-8 "https://juejin.cn/post/7050893279818317854#heading-8")

```
// array
// 构造长度一定（Length）的元组
type GetTuple<Length extends number = 0> = GetTupleHelper<Length>

type GetTupleHelper<
  Length extends number = 0,
  R extends unknown[] = []
> = R["length"] extends Length ? R : GetTupleHelper<Length, [...R, unknown]>
复制代码

```

```
type IntAddSingleHepler<N1 extends number, N2 extends number> = [
  ...array.GetTuple<N1>,
  ...array.GetTuple<N2>
]["length"]

// number
// 正整数（和0）加法，T1，T2最大999
type IntAddSingle<N1 extends number, N2 extends number> = IntAddSingleHepler<
  N1,
  N2
> extends number
  ? IntAddSingleHepler<N1, N2>
  : number
复制代码

```

#### 比较大小

如果想要实现元组类型的排序，那就必须要能够比较数字大小

如何实现数字类型大小的比较呢？

还是得基于元组

基于两个数 `N1`、 `N2`，创建不同的元组 `T1`、`T2`，依次减少两个元组的长度（删除第一位或最后一位），当有一个元组长度为 0 时，就是这个元组对应的数字类型，比另一个数字类型小（或相等，所以也要先判断是否不相等才进行比较）

去掉数组最后一位的实现：[juejin.cn/post/704553…](https://juejin.cn/post/7045536402112512007#heading-2 "https://juejin.cn/post/7045536402112512007#heading-2")

基于模式匹配，匹配出最后一项，和剩余项，并返回剩余项

类型系统中没有改变原类型的概念，因此元组类型的增删改查都应该直接返回修改后的类型，而不是修改后的变化值

如在 js 中，`[1, 2, 3].shift()` 会返回 `1`，`[1, 2, 3].pop()` 会返回 `3`，但是 ts 类型系统中，这样返回是没有意义的，`Pop<[1, 2, 3]>` 应该得到类型 `[1, 2]`

```
// array
// 去掉数组的最后一位
type Pop<T extends unknown[]> = T extends [...infer LeftRest, infer Last]
  ? LeftRest
  : never
复制代码

```

```
// T 意为 Tuple 元组
type CompareHelper<
  N1 extends number,
  N2 extends number,
  T1 extends unknown[] = array.GetTuple<N1>,
  T2 extends unknown[] = array.GetTuple<N2>
> = IsNotEqual<N1, N2, true> extends true
  ? common.Or<IsZero<T1["length"]>, IsZero<T2["length"]>> extends true
    ? IsZero<T1["length"]> extends true
      ? false
      : true
    : CompareHelper<array.Pop<T1>["length"], array.Pop<T2>["length"]>
  : false

// number
// 比较两个数字类型大小
type Compare<N1 extends number, N2 extends number> = CompareHelper<N1, N2>
复制代码

```

#### 两数相减

两个数字类型相减的逻辑与两个数字类型比较大小的逻辑类似，但是返回类型时，会返回剩余长度多的元组的长度

这个实现受到元组类型长度的限制，只能得到正数（或 0），即结果的绝对值

且用在其他工具类型中时，会出现 `类型实例化过深，并可能无限（Type instantiation is excessively deep and possibly infinite）` 的报错，参考 [github issue](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fmicrosoft%2FTypeScript%2Fissues%2F37613 "https://github.com/microsoft/TypeScript/issues/37613")

即：目前有 50 个嵌套实例的限制，可以通过批处理规避限制 （20210714）

```
// 批处理示例
type GetLetters<Text> =
  Text extends `${infer C0}${infer C1}${infer C2}${infer C3}${infer C4}${infer C5}${infer C6}${infer C7}${infer C8}${infer C9}${infer Rest}`
    ? C0 | C1 | C2 | C3 | C4 | C5 | C6 | C7 | C8 | C9 | GetLetters<Rest>
    : Text extends `${infer C}${infer Rest}`
    ? C | GetLetters<Rest>
    : never
复制代码

```

减法实现

```
type IntMinusSingleAbsHelper<
  N1 extends number,
  N2 extends number,
  T1 extends unknown[] = array.GetTuple<N1>,
  T2 extends unknown[] = array.GetTuple<N2>
> = IsNotEqual<N1, N2, true> extends true
  ? common.Or<IsZero<T1["length"]>, IsZero<T2["length"]>> extends true
    ? IsZero<T1["length"]> extends true
      ? T2["length"]
      : T1["length"]
    : IntMinusSingleAbsHelper<array.Pop<T1>["length"], array.Pop<T2>["length"]>
  : 0

// number
// 两个数字类型相减，得到绝对值
type IntMinusSingleAbs<
  N1 extends number,
  N2 extends number
> = IntMinusSingleAbsHelper<N1, N2>
复制代码

```

虽然有嵌套深度的限制，写好的减法不能用，但是加法是很好用的，有加法我们一样可以写出很多逻辑

参考 js 封装工具类型
------------

工具类型可能相互依赖，如果遇到没见过的，请跳转到对应章节查看

### 封装 string 工具类型

#### Stringify - 将类型字符串化

```
/**
 * 将支持的类型转化为字符串
 * @example
 * type Result = Stringify<0> // "0"
 */
type Stringify<T extends CanStringified> = `${T}`
复制代码

```

原理：TS 内置的模板字符串类型

#### GetChars - 获取字符

```
/**
 * @exports
 * 获取模板字符串类型中的字符
 * @see https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html
 * @example
 * type Result = GetChars<'abc'> // 'a' | 'b' | 'c'
 */
type GetChars<S> = GetCharsHelper<S, never>

/**
 * 以 尾递归 tail-recursive 的方式优化 GetChars，不导出为工具类型
 */
type GetCharsHelper<S, Acc> = S extends `${infer Char}${infer Rest}`
  ? GetCharsHelper<Rest, Char | Acc>
  : Acc
复制代码

```

原理：通过模板字符串类型的模式匹配，使用 `GetCharsHelper` 匹配出字符串类型的第一个字符和剩余字符，然后将剩余字符继续放入 `GetCharsHelper` 中进行处理

每次匹配的结果通过 `Acc` 参数传递，当 `S` 为空字符串时，`S` 不能分配给 `${infer Char}${infer Rest}`，走到 `false` 分支中，结束递归，即返回 `Acc` 类型

#### Split - 分割字符串

```
type SplitHelper<
  S extends string,
  SplitStr extends string = "",
  T extends string[] = []
> = S extends `${infer Char}${SplitStr}${infer Rest}`
  ? SplitHelper<Rest, SplitStr, array.Push<T, Char>>
  : S extends string
  ? S extends ""
    ? T
    : array.Push<T, S>
  : never

/**
 * 拆分字符串变为一个元组
 * @example
 * type Result = Split<'1,2,3', ','> // [1, 2, 3]
 */
type Split<S extends string, SplitStr extends string = ""> = SplitHelper<
  S,
  SplitStr
>
复制代码

```

原理：分割字符串类型，是把字符串类型转为元组类型，参数中需要设置一个元组类型用作返回结果

模板字符串类型的模式匹配是从左往右的，如果字符类型 `S` 为 `'1,2,3'`，`${infer Char}${','}${infer Rest}` 中 `Char` 即为 `'1'`，`Rest` 即为 `'2,3'`，同理，如果 `S` 为 `'2,3'`，`${infer Char}${','}${infer Rest}` 中 `Char` 即为 `'2'`，`Rest` 即为 `'3'`

这样的话，我们只需要把每次匹配出的 `Char` 放到元组类型参数 `T` 中的最后一项，在匹配结束后，返回 `T` 的类型即可

注：`array.Push` 见下文

#### GetStringLength - 获取字符串长度

```
/**
 * 获取字符串的长度
 * @example
 * type Result = GetStringLength<"123"> // 3
 */
type GetStringLength<S extends string> = Split<S>["length"]
复制代码

```

原理：元组的长度是可以获取的，通过上文的 `Split` 可以将字符串类型按照 `''` 分割成元组类型，再取元组的 `length` 即为字符串类型的长度

#### CharAt - 获取字符串在索引位 I 下的 字符

```
/**
 * 获取字符串在索引位 I 下的 字符
 * @example
 * type Result = CharAt<"123", 1> // "2"
 */
type CharAt<S extends string, I extends number> = Split<S>[I]
复制代码

```

原理：元组类型可以进行索引访问，可以将字符串类型按照 `''` 分割成元组类型，然后通过 `索引访问`，得到索引位 `I` 处的字符

#### Concat - 拼接两个字符串

```
/**
 * 拼接两个字符串
 * @example
 * type Result = Concat<"123", "456"> // "123456"
 */
type Concat<S1 extends string, S2 extends string> = `${S1}${S2}`
复制代码

```

原理：TS 模板字符串类型用法

#### Includes - 判断字符串是否包含子串

```
/**
 * 判断字符串是否包含子串
 * @example
 * type Result = Includes<"123", "12"> // true
 */
type Includes<
  S1 extends string,
  S2 extends string
> = S1 extends `${infer Left}${S2}${infer Right}` ? true : false
复制代码

```

原理：模式匹配可判断字符串类型中是否含有子串

#### StartsWith - 判断字符串是否以子串为起始

```
/**
 * 判断字符串是否以子串为起始
 * @example
 * type Result = StartsWith<"123", "12"> // true
 */
type StartsWith<
  S1 extends string,
  S2 extends string
> = S1 extends `${S2}${infer Right}` ? true : false
复制代码

```

原理：模式匹配时，左侧不写 `infer Left`，代表左侧只包含空字符串，不存在任何有长度的子串，即 `StartsWith`

#### EndsWith - 判断字符串是否以子串为结束

```
/**
 * 判断字符串是否以子串为结束
 * @example
 * type Result = EndsWith<"123", "23"> // true
 */
type EndsWith<
  S1 extends string,
  S2 extends string
> = S1 extends `${infer Left}${S2}` ? true : false
复制代码

```

原理：模式匹配时，右侧不写 `infer Right`，代表右侧只包含空字符串，不存在任何有长度的子串，即 `EndsWith`

#### IndexOf - 从左往右查找子串的位置

```
type IndexOfHelper<
  S1 extends string,
  S2 extends string,
  Len1 extends number = GetStringLength<S1>,
  Len2 extends number = GetStringLength<S2>
> = common.Or<
  number.Compare<Len1, Len2>,
  number.IsEqual<Len1, Len2>
> extends true
  ? S1 extends `${infer Left}${S2}${infer Right}`
    ? GetStringLength<Left>
    : -1
  : -1

/**
 * 从左往右查找子串的位置
 * @example
 * type Result = IndexOf<"123", "23"> // 1
 */
type IndexOf<S1 extends string, S2 extends string> = IndexOfHelper<S1, S2>
复制代码

```

原理：匹配出 `${infer Left}${S2}${infer Right}` 中 `Left`，求其长度，则索引位即为 `Left` 的长度，如果匹配不到，返回 -1

可以先比较父串和子串的长度，如果子串比父串还长，那就不需要匹配了，直接返回 -1

#### LastIndexOf - 从右往左查找子串的位置

```
type LastIndexOfHelper<
  S1 extends string,
  S2 extends string,
  Index extends number = -1 /** 当前从左往右匹配最大的值，匹配不到以后，上一次匹配的索引就是从右往左第一个的索引 */,
  AddOffset extends number = 0 /** 每次从左往右匹配并替换成空串后，下次循序需要累加的值 */
> = S1 extends `${infer Left}${S2}${infer Right}`
  ? LastIndexOfHelper<
      Replace<S1, S2, "">,
      S2,
      number.IntAddSingle<GetStringLength<Left>, AddOffset>,
      number.IntAddSingle<AddOffset, GetStringLength<S2>>
    >
  : Index

/**
 * 从右往左查找子串的位置
 * @example
 * type Result = LastIndexOf<"23123", "23"> // 3
 */
type LastIndexOf<S1 extends string, S2 extends string> = LastIndexOfHelper<
  S1,
  S2
>
复制代码

```

原理：模板字符串类型的模式匹配是从左往右的，而 `LastIndexOf` 是从右往左的，所以在匹配时，仍然基于从左往右匹配，但是每次匹配后，替换掉匹配过的子串为空串

然后把删掉的部分的长度累计起来，结果就是模拟从右往左匹配到的索引值

注：`Replace` 见下文

#### Replace - 在字符串中查找并替换一处子串

```
/**
 * 在字符串中查找并替换一处子串
 * @example
 * type Result = Replace<"23123", "23", "xx"> // "xx123"
 */
type Replace<
  S extends string,
  MatchStr extends string,
  ReplaceStr extends string
> = S extends `${infer Left}${MatchStr}${infer Right}`
  ? `${Left}${ReplaceStr}${Right}`
  : S
复制代码

```

原理：基于模板字符串的模式匹配，匹配到了就用 `ReplaceStr` 换掉 `MatchStr`

#### ReplaceAll - 在字符串中查找并替换所有子串

```
/**
 * 在字符串中查找并替换所有子串
 * @example
 * type Result = Replace<"23123", "23", "xx"> // "xx1xx"
 */
type ReplaceAll<
  S extends string,
  MatchStr extends string,
  ReplaceStr extends string
> = Includes<S, MatchStr> extends true
  ? ReplaceAll<Replace<S, MatchStr, ReplaceStr>, MatchStr, ReplaceStr>
  : S
复制代码

```

原理：基于 `Replace`，递归进行替换，替换掉所有 `MatchStr`，终止条件是 `S` 是否包含 `MatchStr`

#### Repeat - 重复 Times 次数的字符串

```
type RepeatHelper<
  S extends string,
  Times extends number,
  OriginStr extends string = S,
  Offset extends number = 1
> = Times extends 0
  ? ""
  : number.IsEqual<Times, Offset> extends true
  ? S
  : `${OriginStr}${RepeatHelper<
      S,
      Times,
      OriginStr,
      number.IntAddSingle<Offset, 1>
    >}`

/**
 * 重复 Times 次数的字符串
 * @example
 * type Result = Repeat<"1", 5> // "11111"
 */
type Repeat<S extends string, Times extends number = 1> = RepeatHelper<S, Times>
复制代码

```

原理：当重复次数 `Times` 为 `0` 时，直接返回空字符串

在参数中传递循环条件 `Offset` （每次传递时加 1，即 `number.IntAddSingle<Offset, 1>`），当循环条件 `Offset` 和循环次数 `Times` 相等时，结束递归

每次递归中，都在字符串的起始位置插入一个字符串 `S`，即

```
`${第一次的S}${`${第二次的S}${`${第三次的S}${剩下的...}`}`}`
复制代码

```

注：`number.IntAddSingle、number.IsEqual` 见下文

#### PadStart - 在字符串前面填充

```
type PadHelper<
  S extends string,
  N extends number = 0,
  FillS extends string = " ",
  IsStart extends boolean = true,
  Len extends number = GetStringLength<S>,
  Offset extends number = Len
> = number.Compare<N, Len> extends true
  ? number.IsEqual<N, Offset> extends true
    ? S
    : PadHelper<
        `${IsStart extends true ? FillS : ""}${S}${IsStart extends false
          ? FillS
          : ""}`,
        N,
        FillS,
        IsStart,
        Len,
        number.IntAddSingle<Offset, 1>
      >
  : S

/**
 * 当字符串不满足给定的长度时，在字符串前面填充使其满足长度
 * @example
 * type Result = PadStart<'0123', 10> // '      0123'
 */
type PadStart<
  S extends string,
  N extends number = 0,
  FillS extends string = " "
> = PadHelper<S, N, FillS>
复制代码

```

原理：比较指定的长度和当前字符串类型的长度相等，如果满足长度，直接返回 `S`，每次递归时，给 `S` 左侧添加指定的字符，直到 `S` 的长度满足指定的长度时，终止递归

#### PadEnd - 在字符串后面填充

```
/**
 * 当字符串不满足给定的长度时，在字符串后面填充使其满足长度
 * @example
 * type Result = PadStart<'0123', 10> // '0123      '
 */
type PadEnd<
  S extends string,
  N extends number = 0,
  FillS extends string = " "
> = PadHelper<S, N, FillS, false>
复制代码

```

原理：比较给定的长度和当前字符串类型的长度相等，如果满足长度，直接返回 `S`，每次递归时，给 `S` 右侧添加指定的字符，直到 `S` 的长度满足给定的长度时，终止递归

#### TrimLeft - 去掉字符串前面的空格

```
/**
 * 去掉字符串类型左侧的空格
 * @see https://juejin.cn/post/7045536402112512007#heading-5
 * @example
 * type Result = PadStart<'   0123'> // '0123'
 */
type TrimLeft<S extends string> = S extends `${
  | " "
  | "\t"
  | "\n"}${infer RightRest}`
  ? TrimLeft<RightRest>
  : S
复制代码

```

原理：每次匹配 `${一个空格}${剩余字符}` 然后让 `剩余字符` 继续匹配，直到不符合 `${一个空格}${剩余字符}` 的规则时，终止递归，返回 `S`

#### TrimRight - 去掉字符串后面的空格

```
/**
 * 去掉字符串类型右侧的空格
 * @example
 * type Result = PadStart<'0123   '> // '0123'
 */
type TrimRight<S extends string> = S extends `${infer LeftRest}${
  | " "
  | "\t"
  | "\n"}`
  ? TrimRight<LeftRest>
  : S
复制代码

```

原理：每次匹配 `${剩余字符}${一个空格}` 然后让 `剩余字符` 继续匹配，直到不符合 `${剩余字符}${一个空格}` 的规则时，终止递归，返回 `S`

#### Trim - 去掉字符串的空格

```
/**
 * 去掉字符串类型两侧的空格
 * @example
 * type Result = PadStart<'   0123   '> // '0123'
 */
type Trim<S extends string> = TrimLeft<TrimRight<S>>
复制代码

```

原理：先用 `TrimRight` 去掉右侧的空格，再把前者结果交给 `TrimLeft` 去掉左侧的空格

#### ToUpperCase - 字符串转大写

```
/**
 * 字符串转大写
 * @example
 * type Result = ToUpperCase<'abc'> // 'ABC'
 */
type ToUpperCase<S extends string> = Uppercase<S>
复制代码

```

原理：TS 内置

#### ToLowerCase - 字符串转小写

```
/**
 * 字符串转小写
 * @example
 * type Result = ToUpperCase<'ABC'> // 'abc'
 */
type ToLowerCase<S extends string> = Lowercase<S>
复制代码

```

原理：TS 内置

#### SubString - 截取 start（包括）到 end（不包括）之间的字符串

```
type SubStringHelper<
  S extends string,
  Start extends number,
  End extends number,
  Offset extends number = 0,
  Cache extends string[] = []
> = number.IsEqual<Offset, End> extends true
  ? array.Join<Cache, "">
  : SubStringHelper<
      S,
      Start,
      End,
      number.IntAddSingle<Offset, 1>,
      common.And3<
        common.Or<number.Compare<Offset, Start>, number.IsEqual<Offset, Start>>,
        common.Or<number.Compare<End, Offset>, number.IsEqual<Offset, End>>,
        CharAt<S, Offset> extends string ? true : false
      > extends true
        ? array.Push<Cache, CharAt<S, Offset>>
        : Cache
    >
/**
 * 截取start（包括）到end（不包括）之间的字符串
 * @example
 * type Result = SubString<'123', 0, 1> // '1'
 */
type SubString<
  S extends string,
  Start extends number,
  End extends number
> = SubStringHelper<S, Start, End>
复制代码

```

原理：遍历字符串类型的每一个字符，如果当前索引大于等于 `Start`，并且小于等于 `End`，就把当前字符 push 到元组中，最后用 `array.Join`，将元组转为字符串类型

注：`array.Join` 见下文

#### SubStr - 在字符串中抽取从开始下标到结束下标的字符

```
/**
 * 在字符串中抽取从 开始 下标开始的指定数目的字符
 * @example
 * type Result = SubStr<'123', 1, 2> // '23'
 */
type SubStr<
  S extends string,
  Start extends number,
  Len extends number
> = SubStringHelper<S, Start, number.IntAddSingle<Start, Len>>
复制代码

```

原理：`SubString` 需要起始和结束，有 `Start` 和 `Len` 就可以先算出 `End`，就可以使用 `SubString` 了

### 封装 array 工具类型

#### GetTuple - 构造指定长度的元组

```
/**
 * 构造长度一定（Length）的元组
 * @example
 * type Result = GetTuple<3> // [unknown, unknown, unknown]
 */
type GetTuple<Length extends number = 0> = GetTupleHelper<Length>

type GetTupleHelper<
  Length extends number = 0,
  R extends unknown[] = []
> = R["length"] extends Length ? R : GetTupleHelper<Length, [...R, unknown]>
复制代码

```

#### ArraySet - 更改元组中指定索引位的类型

```
type SetHelper<
  T extends unknown[],
  Index extends number,
  Value,
  Offset extends number = 0,
  Cache extends unknown[] = []
> = Offset extends T["length"]
  ? Cache
  : SetHelper<
      T,
      Index,
      Value,
      number.IntAddSingle<Offset, 1>,
      Push<Cache, Offset extends Index ? Value : T[Offset]>
    >

/**
 * 更改元组中指定索引位的类型
 * @example
 * type Result = ArraySet<[1, 2, 3], 2, 4> // [1, 2, 4]
 */
type ArraySet<T extends unknown[], Index extends number, Value> = SetHelper<
  T,
  Index,
  Value
>
复制代码

```

原理：遍历元组类型，如果 `Offset` 等于给定的索引，则该索引对应的类型替换为给定的类型，否则用原类型

#### TupleToUnion - 从元（数）组类型构造联合类型

```
/**
 * 从元（数）组类型构造联合类型
 * @example
 * type Result = TupleToUnion<[1, 2, 3]> // 1 | 2 | 3
 */
type TupleToUnion<T extends unknown[]> = T[number]
复制代码

```

原理：元组（数组）类型的索引访问会得到联合类型

#### Pop - 去除元组类型的最后一位

```
/**
 * 去掉元组的最后一位
 * @see https://juejin.cn/post/7045536402112512007#heading-2
 * @example
 * type Result = Pop<[1, 2, 3]> // [1, 2]
 */
type Pop<T extends unknown[]> = T extends [...infer LeftRest, infer Last]
  ? LeftRest
  : never
复制代码

```

原理：基于元组的模式匹配，提取最后一项，返回剩余项

#### Shift - 去除元组类型的第一位

```
/**
 * 去掉数组的第一位
 * @example
 * type Result = Shift<[1, 2, 3]> // [2, 3]
 */
type Shift<T extends unknown[]> = T extends [infer First, ...infer RightRest]
  ? RightRest
  : never
复制代码

```

原理：与 `Pop` 同理

#### UnShift - 在元组前面插入一位

```
/**
 * 在元组前面插入一位
 * @example
 * type Result = UnShift<[1, 2, 3], 0> // [0, 1, 2, 3]
 */
type UnShift<T extends unknown[], Item> = [Item, ...T]
复制代码

```

原理：`[]` 中直接写类型可以构建新元组类型，其中写 `...Tuple`，与 js 中的扩展运算符效果一致

#### Push - 在元组后面插入一位

```
/**
 * 在元组最后插入一位
 * @example
 * type Result = Push<[1, 2, 3], 4> // [1, 2, 3， 4]
 */
type Push<T extends unknown[], Item> = [...T, Item]
复制代码

```

原理：同 `UnShift`

#### Concat - 合并两个元组类型

```
/**
 * 合并两个元组类型
 * @example
 * type Result = Concat<[1, 2, 3], [4]> // [1, 2, 3, 4]
 */
type Concat<T extends unknown[], R extends unknown[]> = [...T, ...R]
复制代码

```

原理：见 `UnShift`

#### Join - 将元组类型拼接成字符串类型

```
/**
 * 将元组类型拼接成字符串类型
 * @example
 * type Result = Join<[1, 2, 3]> // "1,2,3"
 */
type Join<
  T extends string.CanStringified[],
  SplitStr extends string.CanStringified = ""
> = T["length"] extends 0
  ? ""
  : T extends [infer Left, ...infer RightRest]
  ? Left extends string.CanStringified
    ? RightRest extends string.CanStringified[]
      ? `${Left}${T["length"] extends 1 ? "" : SplitStr}${Join<
          RightRest,
          SplitStr
        >}`
      : never
    : never
  : never
复制代码

```

原理：每次递归时提取元组第一个类型，然后将此类型放到模板字符串类型的第一个位置 `${第一个位置}${第二个位置}${第三个位置}`

第二个位置即转为字符串用来分隔的子串，如果元组的长度为 0，则为空串

第三个位置则是剩下部分的逻辑，即重复最开始的逻辑

#### Every - 校验元组中每个类型是否都符合条件

```
type EveryHelper<
  T extends unknown[],
  Check,
  Offset extends number = 0,
  CacheBool extends boolean = true
> = T["length"] extends Offset
  ? CacheBool
  : EveryHelper<
      T,
      Check,
      number.IntAddSingle<Offset, 1>,
      common.And<common.CheckLeftIsExtendsRight<T[Offset], Check>, CacheBool>
    >

/**
 * 校验元组中每个类型是否都符合条件
 * @example
 * type Result = Every<[1, 2, 3], number> // true
 */
type Every<T extends unknown[], Check> = T["length"] extends 0
  ? false
  : EveryHelper<T, Check>
复制代码

```

原理：初始类型 `CacheBool` 为 true，依次将元组中每个类型与初始类型进行 `与` 操作，如果元组长度为 0，则返回 `false`

注：`common.And、common.CheckLeftIsExtendsRight` 见下文

#### Some - 校验元组中是否有类型符合条件

```
type SomeHelper<
  T extends unknown[],
  Check,
  Offset extends number = 0,
  CacheBool extends boolean = false
> = T["length"] extends Offset
  ? CacheBool
  : SomeHelper<
      T,
      Check,
      number.IntAddSingle<Offset, 1>,
      common.Or<common.CheckLeftIsExtendsRight<T[Offset], Check>, CacheBool>
    >

/**
 * 校验元组中是否有类型符合条件
 * @example
 * type Result = Every<['1', '2', 3], number> // true
 */
type Some<T extends unknown[], Check> = SomeHelper<T, Check>
复制代码

```

原理：初始类型 `CacheBool` 为 false，依次将元组中每个类型与初始类型进行 `或` 操作，如果元组长度为 0，则返回 `false`

注：`common.Or、common.CheckLeftIsExtendsRight` 见下文

#### Fill - 以指定类型填充元组类型

```
type FillHelper<
  T extends unknown[],
  F,
  Offset extends number = 0
> = T["length"] extends 0
  ? F[]
  : Offset extends T["length"]
  ? common.IsEqual<T, F[]> extends true /** any[] -> T[] */
    ? T
    : F[]
  : FillHelper<array.Push<array.Shift<T>, F>, F, number.IntAddSingle<Offset, 1>>


/**
 * 以指定类型填充元组类型
 * @example
 * type Result = Fill<['1', '2', 3, any], 1> // [1, 1, 1, 1]
 */
type Fill<T extends unknown[], F = undefined> = FillHelper<T, F>
复制代码

```

原理：如果原元组长度为 0，则直接返回由新类型构成的元组 `F[]`

如果是数组类型如：`any[]`、`never[]`、`number[]`，也应该直接替换成 `T[]`

否则，每次在原元组中删除第一个，然后在最前面添加一个新类型，直到循环条件与 `T` 的长度一致时，终止递归

注：`commom.IsEqual` 见下文

#### Filter - 过滤出元组类型中符合条件的类型

```
type FilterHelper<
  T extends unknown[],
  C,
  Strict extends boolean,
  Offset extends number = 0,
  Cache extends unknown[] = []
> = Offset extends T["length"]
  ? Cache
  : FilterHelper<
      T,
      C,
      Strict,
      number.IntAddSingle<Offset, 1>,
      common.And<Strict, common.IsEqual<T[Offset], C>> extends true
        ? array.Push<Cache, T[Offset]>
        : common.And<
            common.Not<Strict>,
            common.CheckLeftIsExtendsRight<T[Offset], C>
          > extends true
        ? array.Push<Cache, T[Offset]>
        : Cache
    >

/**
 * 过滤出元组类型中符合条件的类型
 * @example
 * type Result = Filter<['1', '2', 3, any, 1], 1, true> // [1]
 */
type Filter<
  T extends unknown[],
  C,
  Strict extends boolean = false
> = FilterHelper<T, C, Strict>
复制代码

```

原理：严格模式，即 `any` 只能为 `any`，而不能为 `1`、`unknown` 这样的其他类型

如果是严格模式就用 `common.IsEqual` 进行约束校验，否则用 `common.CheckLeftIsExtendsRight` 进行约束校验

每次递归时，如果满足上述条件，则放入新的元组类型中

如果循环条件 `Offset` 等于 `T` 的长度是，终止循环，返回新的元组类型 `Cache`

注：`common.Not` 见下文

#### MapWidthIndex - 将元组类型映射为带索引的元组类型

```
interface IndexMappedItem<Item, Index extends number, Tuple extends unknown[]> {
  item: Item
  index: Index
  tuple: Tuple
}

type MapWidthIndexHelper<
  T extends unknown[],
  Offset extends number = 0,
  Cache extends unknown[] = []
> = T["length"] extends Offset
  ? Cache
  : MapWidthIndexHelper<
      T,
      number.IntAddSingle<Offset, 1>,
      Push<Cache, IndexMappedItem<T[Offset], Offset, T>>
    >

/**
 * 将元组类型映射为带索引的元组类型
 * @example
 * type Result = MapWidthIndex<[1, 2]> // [{ item: 1; index: 0;tuple: [1, 2]; }, { item: 2; index: 1;tuple: [1, 2]; }]
 */
type MapWidthIndex<T extends unknown[]> = MapWidthIndexHelper<T>
复制代码

```

原理：声明一个接口用于构造新的元组类型中的项，然后每次递归都向 `Cache` 中添加一个经过 `IndexMappedItem` 处理后的类型

注：由于 TS 中实现不了回调的效果，因为带泛型参数的工具类型不能直接当做类型传递，必须要先传了泛型参数才能用，所以暂时无法实现 `js` 中 `Array.prototype.map` 的效果

#### Find - 找到元组类型中第一个符合条件的类型

```
type FindHelper<
  T extends unknown[],
  C,
  Offset extends number = 0
> = Offset extends number.IntAddSingle<T["length"], 1>
  ? null
  : common.CheckLeftIsExtendsRight<T[Offset], C> extends true
  ? T[Offset]
  : FindHelper<T, C, number.IntAddSingle<Offset, 1>>
/** */
type Find<T extends unknown[], C> = FindHelper<T, C>
复制代码

```

原理：遍历在元组中找，如果找到了匹配的，则返回该类型，否则返回 `null` 类型

#### Reverse - 反转元组

```
type ReverseHelper<
  T extends unknown[],
  Offset extends number = 0,
  Cache extends unknown[] = []
> = Cache["length"] extends T["length"]
  ? Cache
  : ReverseHelper<T, number.IntAddSingle<Offset, 1>, UnShift<Cache, T[Offset]>>
/** */
type Reverse<T extends unknown[]> = ReverseHelper<T>
复制代码

```

原理：遍历老元组类型，每次在新元组类型 `Cache` 的前面插入当前类型

#### FindLast - 找到元组类型中最后一个符合条件的类型

```
/** */
type FindLast<T extends unknown[], C> = Find<Reverse<T>, C>
复制代码

```

原理：反转老元组类型，然后通过 `Find` 查找

#### FindIndex - 找到元组类型中第一个符合条件的类型的索引

```
type FindIndexHelper<
  T extends unknown[],
  C,
  Strict extends boolean = false,
  Offset extends number = 0
> = Offset extends number.IntAddSingle<T["length"], 1>
  ? -1
  : common.And<common.IsEqual<T[Offset], C>, Strict> extends true
  ? Offset
  : common.And<
      common.CheckLeftIsExtendsRight<T[Offset], C>,
      common.Not<Strict>
    > extends true
  ? Offset
  : FindIndexHelper<T, C, Strict, number.IntAddSingle<Offset, 1>>

/** */
type FindIndex<
  T extends unknown[],
  C,
  Strict extends boolean = false
> = FindIndexHelper<T, C, Strict>
复制代码

```

原理：严格模式，参考上文 `Filter`，遍历元组，符合约束校验时，返回当前 `Offset`，否则结束后返回 -1

#### FindLastIndex - 找到元组类型中最后一个符合条件的类型的索引

```
type FindLastIndexHelper<
  T extends unknown[],
  C,
  Item = Find<Reverse<MapWidthIndex<T>>, IndexMappedItem<C, number, T>>
> = Item extends IndexMappedItem<C, number, T> ? Item["index"] : -1

type FindLastIndex<T extends unknown[], C> = FindLastIndexHelper<T, C>
复制代码

```

原理：通过 `MapWidthIndex` 将索引记录到元组的每个类型中，使用 `Find` 匹配反转后的元组，匹配到时，返回该类型的 `Item['index']` 值即为结果

#### Flat - 扁平化元组

```
type FlatHelper<
  T extends unknown[],
  Offset extends number = 0,
  Cache extends unknown[] = []
> = Offset extends T["length"]
  ? Cache
  : FlatHelper<
      T,
      number.IntAddSingle<Offset, 1>,
      T[Offset] extends unknown[]
        ? Concat<Cache, T[Offset]>
        : Push<Cache, T[Offset]>
    >

type Flat<T extends unknown[]> = FlatHelper<T>
复制代码

```

原理：遍历元组类型，如果当前类型不满足 `unknown[]` 的约束，则将它 `Push` 进新元组中，否则将它 `Concat` 进去

#### Includes - 元组类型中是否存在一个符合条件的类型

```
type Includes<T extends unknown[], C> = common.CheckLeftIsExtendsRight<
  C,
  TupleToUnion<T>
>
复制代码

```

原理：将元组转为联合类型，如果约束条件 `C` 可以分配给该联合类型，那么就是 `true`

#### Slice - 提取元组类型中指定起始位置到指定结束位置的类型构造新元组类型

```
type SliceHelper<
  T extends unknown[],
  Start extends number,
  End extends number,
  Offset extends number = 0,
  Cache extends unknown[] = []
> = number.IsEqual<Offset, End> extends true
  ? Cache
  : SliceHelper<
      T,
      Start,
      End,
      number.IntAddSingle<Offset, 1>,
      common.And3<
        common.Or<number.Compare<Offset, Start>, number.IsEqual<Offset, Start>>,
        common.Or<number.Compare<End, Offset>, number.IsEqual<Offset, End>>,
        common.Or<
          number.Compare<T["length"], Offset>,
          number.IsEqual<T["length"], End>
        >
      > extends true
        ? array.Push<Cache, T[Offset]>
        : Cache
    >

type Slice<
  T extends unknown[],
  Start extends number,
  End extends number
> = SliceHelper<T, Start, End>
复制代码

```

原理：和字符串裁剪的类似，遍历老元组，当循环条件 `Offset` 大于等于 `Start` 或 小于等于 `End` 时，将这些类型 `Push` 到新元组中

#### Sort - 排序

```
type SortHepler2<
  T extends number[],
  Offset extends number = 0,
  Offset1 extends number = 0,
  Offset1Added extends number = number.IntAddSingle<Offset1, 1>,
  Seted1 extends unknown[] = ArraySet<T, Offset1Added, T[Offset1]>,
  Seted2 extends unknown[] = ArraySet<Seted1, Offset1, T[Offset1Added]>
> = number.IntAddSingle<
  number.IntAddSingle<Offset, Offset1>,
  1
> extends T["length"]
  ? SortHepler1<T, number.IntAddSingle<Offset, 1>>
  : SortHepler2<
      number.Compare<T[Offset1], T[Offset1Added]> extends true
        ? Seted2 extends number[]
          ? Seted2
          : never
        : T,
      number.IntAddSingle<Offset1, 1>
    >

type SortHepler1<
  T extends number[],
  Offset extends number = 0
> = Offset extends T["length"] ? T : SortHepler2<T, Offset>

type Sort<T extends number[]> = SortHepler1<T>
复制代码

```

原理：最简单的冒泡排序，每次排序时，将大的与小的置换

注：受到嵌套实例深度的限制，只能排两个类型的元组

### 封装 number 工具类型

#### IsZero - 判断类型为 0

```
/**
 * number类型是否为0
 * @example
 * type Result = IsZero<0> // true 
 */
type IsZero<N extends NumberLike> = common.CheckLeftIsExtendsRight<N, 0 | "0">
复制代码

```

注：原理见上文，`NumberLike` 见上文

#### IsOverZero - 是否大于 0

```
/**
 * number类型是否大于0
 * @example
 * type Result = IsOverZero<2> // true 
 */
type IsOverZero<N extends NumberLike> = IsZero<N> extends true
  ? false
  : common.CheckLeftIsExtendsRight<
      string.Stringify<N> extends `${"-"}${infer Rest}` ? Rest : false,
      false
    >
复制代码

```

注：原理见上文

#### IsLessZero - 是否小于 0

```
/**
 * number类型是否小于0
 * @example
 * type Result = IsLessZero<-2> // true 
 */
type IsLessZero<N extends NumberLike> = common.Not<IsOverZero<N>>
复制代码

```

#### IsFloat - 是否为浮点型

```
/**
 * number类型是否是小数
 * @example
 * type Result = IsFloat<1.2> // true 
 */
type IsFloat<
  N extends NumberLike,
  OnlyCheckPoint extends boolean = true
> = string.Stringify<N> extends `${infer Left}${"."}${infer Right}`
  ? OnlyCheckPoint extends true
    ? true
    : common.Not<array.Every<string.Split<Right>, "0">>
  : false
复制代码

```

原理：转为字符串类型后判断是否小数点

#### IsInt - 是否为整型

```
/**
 * number类型是否是整数
 * @example
 * type Result = IsInt<1> // true 
 */
type IsInt<
  N extends NumberLike,
  OnlyCheckPoint extends boolean = true
> = common.Not<IsFloat<N, OnlyCheckPoint>>
复制代码

```

原理：`IsFloat` 取反

#### IsEqual - 数字类型是否相等

```
/**
 * 两个number类型是否相等
 * @example
 * type Result = IsEqual<1, 1> // true 
 */
type IsEqual<
  L extends NumberLike,
  R extends NumberLike,
  Strict extends boolean = true
> = Strict extends true
  ? common.CheckLeftIsExtendsRight<L, R>
  : common.CheckLeftIsExtendsRight<string.Stringify<L>, string.Stringify<R>>
复制代码

```

原理：见上文

#### IsNotEqual - 数字类型是否不相等

```
/**
 * 两个number类型是否不相等
 * @example
 * type Result = IsNotEqual<1, 2> // true 
 */
type IsNotEqual<
  L extends NumberLike,
  R extends NumberLike,
  Strict extends boolean = true
> = common.Not<IsEqual<L, R, Strict>>
复制代码

```

原理：`IsEqual` 取反

#### IntAddSingle - 整数相加

```
type IntAddSingleHepler<N1 extends number, N2 extends number> = [
  ...array.GetTuple<N1>,
  ...array.GetTuple<N2>
]["length"]

/**
 * 正整数（和0）加法，A1，A2最大999
 * @see https://juejin.cn/post/7050893279818317854#heading-8
 * @example
 * type Result = IntAddSingle<1, 2> // 3 
 */
type IntAddSingle<N1 extends number, N2 extends number> = IntAddSingleHepler<
  N1,
  N2
> extends number
  ? IntAddSingleHepler<N1, N2>
  : number
复制代码

```

原理：见上文

#### Compare - 比较大小

```
type CompareHelper<
  N1 extends number,
  N2 extends number,
  A1 extends unknown[] = array.GetTuple<N1>,
  A2 extends unknown[] = array.GetTuple<N2>
> = IsNotEqual<N1, N2, true> extends true
  ? common.Or<IsZero<A1["length"]>, IsZero<A2["length"]>> extends true
    ? IsZero<A1["length"]> extends true
      ? false
      : true
    : CompareHelper<array.Pop<A1>["length"], array.Pop<A2>["length"]>
  : false

/**
 * 比较大小
 * @example
 * type Result = Compare<1, 2> // false 
 */
type Compare<N1 extends number, N2 extends number> = CompareHelper<N1, N2>
复制代码

```

原理：见上文

#### IntMinusSingleAbs - 两数相减

```
type IntMinusSingleAbsHelper<
  N1 extends number,
  N2 extends number,
  A1 extends unknown[] = array.GetTuple<N1>,
  A2 extends unknown[] = array.GetTuple<N2>
> = IsNotEqual<N1, N2, true> extends true
  ? common.Or<IsZero<A1["length"]>, IsZero<A2["length"]>> extends true
    ? IsZero<A1["length"]> extends true
      ? A2["length"]
      : A1["length"]
    : IntMinusSingleAbsHelper<array.Pop<A1>["length"], array.Pop<A2>["length"]>
  : 0

/**
 * 两数相减
 * @example
 * type Result = IntMinusSingleAbs<2, 1> // 1 
 */
type IntMinusSingleAbs<
  N1 extends number,
  N2 extends number
> = IntMinusSingleAbsHelper<N1, N2>
复制代码

```

原理：见上文

#### GetHalf - 获取当前数字类型的一半

```
type GetHalfHelper<N extends number, Offset extends number = 0> = IsEqual<
  IntAddSingle<Offset, Offset>,
  N
> extends true
  ? Offset
  : IsEqual<IntAddSingle<IntAddSingle<Offset, Offset>, 1>, N> extends true
  ? IntAddSingle<Offset, 1>
  : GetHalfHelper<N, IntAddSingle<Offset, 1>>

/**
 * 获取当前数字类型的一半
 * @example
 * type Result = GetHalf<4> // 2 
 */
type GetHalf<N extends number> = GetHalfHelper<N>
复制代码

```

原理：循环，当 `Offset` + `Offset` 等于 `N` 时，或 `Offset + 1` + `Offset` 等于 `N` 时，`Offset` 即为结果

#### ToNumber - 字符串类型转数字类型

```
/** @see https://juejin.cn/post/6999280101556748295#heading-68 */
type Map = {
  "0": []
  "1": [1]
  "2": [...Map["1"], 1]
  "3": [...Map["2"], 1]
  "4": [...Map["3"], 1]
  "5": [...Map["4"], 1]
  "6": [...Map["5"], 1]
  "7": [...Map["6"], 1]
  "8": [...Map["7"], 1]
  "9": [...Map["8"], 1]
}

type Make10Array<T extends any[]> = [
  ...T,
  ...T,
  ...T,
  ...T,
  ...T,
  ...T,
  ...T,
  ...T,
  ...T,
  ...T
]

type ToNumberHelper<
  S extends string,
  L extends any[] = []
> = S extends `${infer F}${infer R}`
  ? ToNumberHelper<
      R,
      [...Make10Array<L>, ...(F extends keyof Map ? Map[F] : never)]
    >
  : L["length"]

/**
 * 字符串类型转数字类型
 * @example
 * type Result = ToNumber<"100"> // 100
 */
type ToNumber<S extends string> = ToNumberHelper<S>
复制代码

```

原理：创建一个字符与元组的映射表，我们通过前文的探究已知数字类型可由元组的长度得到，那么就根据每个字符依次构建不同长度的元组即可得到结果

`Make10Array` 是将上一次的结果 * 10

见 [juejin.cn/post/699928…](https://juejin.cn/post/6999280101556748295#heading-68 "https://juejin.cn/post/6999280101556748295#heading-68")

#### Add - 数字相加

见下文

### 封装 object 工具类型

#### KeysToUnion - 对象类型的所有键转联合类型

```
type KeysToUnion<T> = keyof T
复制代码

```

#### Values - 获取对象类型的值构成的联合类型

```
type Values<T> = T[KeysToUnion<T>]
复制代码

```

#### KeysToTuple - 获取对象类型键够成的元组类型

```
type KeysToTuple<T> = KeysToUnion<T>[]
复制代码

```

#### ExtractValues - 过滤出符合类型 V 的属性

```
type ExtractValues<T, V> = {
  [Key in keyof T as T[Key] extends V ? Key : never]: T[Key]
}
复制代码

```

#### ExcludeValues - 过滤出不符合类型 V 的属性

```
type ExcludeValues<T, V> = {
  [Key in keyof T as T[Key] extends V ? never : Key]: T[Key]
}
复制代码

```

#### GetterSetterPrefix - 向对象类型中添加 get 和 set 前缀

```
type GetterSetterPrefix<T> = {
  [Key in keyof T as Key extends string ? `get${Capitalize<Key>}` : never]: {
    (): T[Key]
  }
} & {
  [Key in keyof T as Key extends string ? `set${Capitalize<Key>}` : never]: {
    (val: T[Key]): void
  }
} & T
复制代码

```

#### Proxify - 将对象类型的每个属性值转为 get 和 set 形式

```
type Proxify<T> = {
  [P in keyof T]: {
    get(): T[P]
    set(v: T[P]): void
  }
}
复制代码

```

#### NullableValue - 将对象类型的每个属性值转为可为空的

```
type NullableValue<T> = {
  [Key in keyof T]?: common.Nullable<T[Key]>
}
复制代码

```

#### Include - 提取出符合类型 U 的键名构造新的对象类型

```
type Include<T extends object, U extends keyof any> = {
  [Key in keyof T as Key extends U ? Key : never]: T[Key]
}
复制代码

```

#### ChangeRecordType - 将对象类型的属性值填充为类型 T

```
type ChangeRecordType<K, T = undefined> = {
  [P in keyof K]?: T
}
复制代码

```

#### Mutable - 变为可写类型

```
type Mutable<T> = {
  -readonly [P in keyof T]: T[P]
}
复制代码

```

#### ReadonlyPartial - 变为只读且可选的

```
type ReadonlyPartial<T> = {
  readonly [P in keyof T]?: T[P]
}
复制代码

```

#### DeepPartial - 将对象类型的所有属性转为可选

```
type DeepPartial<T> = {
  [Key in keyof T]?: T[Key] extends object ? DeepPartial<T[Key]> : T[Key]
}
复制代码

```

#### ChainedAccessUnion - 查找对象类型的所有路径

```
type ChainedAccessUnion<T extends object> = ChainedAccessUnionHelper<T>

type ChainedAccessUnionHelper<
  T,
  A = {
    [Key in keyof T]: T[Key] extends string ? never : T[Key]
  },
  B = {
    [Key in keyof A]: A[Key] extends never
      ? never
      : A[Key] extends object
      ?
          | `${Extract<Key, string>}.${Extract<keyof A[Key], string>}`
          | (ChainedAccessUnionHelper<A[Key]> extends infer U
              ? `${Extract<Key, string>}.${Extract<U, string>}`
              : never)
      : never
  }
> = T extends object
  ? Exclude<keyof A | Exclude<Values<B>, never>, never>
  : never
复制代码

```

### 封装 common 工具类型

#### Not - 非

```
type Not<C extends boolean> = C extends true ? false : true
复制代码

```

#### And - 与

```
type And<C1 extends boolean, C2 extends boolean> = C1 extends true
  ? C2 extends true
    ? true
    : false
  : false
复制代码

```

#### Or - 或

```
type Or<C1 extends boolean, C2 extends boolean> = C1 extends true
  ? true
  : C2 extends true
  ? true
  : false
复制代码

```

#### CheckLeftIsExtendsRight - 约束校验

```
type CheckLeftIsExtendsRight<T extends any, R extends any> = T extends R
  ? true
  : false
复制代码

```

#### IsEqual - 类型严格相等

```
/**
 * https://github.com/microsoft/TypeScript/issues/27024#issuecomment-510924206
 */
type IsEqual<A, B> = (<T>() => T extends A ? 1 : 2) extends <
  T1
>() => T1 extends B ? 1 : 2
  ? true
  : false
复制代码

```

#### IsAny - 类型是否为 any

```
type IsAny<T> = 0 extends (1 & T) ? true : false
复制代码

```

#### Diff - 差异

```
type Diff<T, C> = Exclude<T, C> | Exclude<C, T>
复制代码

```

#### SumAggregate - 并集

```
type SumAggregate<T, U> = T | U
复制代码

```

#### Nullable - 可为空

```
type Nullable<T> = T | null | undefined
复制代码

```

### 封装 function 工具类型

#### Noop - 普通函数类型

```
type Noop = (...args: any) => any
复制代码

```

#### GetAsyncFunctionReturnType - 获取异步函数返回值

```
type GetAsyncFunctionReturnType<F extends Noop> = Awaited<ReturnType<F>>
复制代码

```

#### GetFunctionLength - 获取参数长度

```
type GetFunctionLength<F extends Noop> = F extends (...args: infer P) => any
  ? P["length"]
  : never
复制代码

```

实现一个加法器！
--------

### 分析

让我们回忆一下，在小学三年级的数学计算中，10 进制的小数相加要怎么做？

如 `1.8 + 1.52`

是不是要先把小数点对齐

```
  1.8
+ 1.52
——————
  2.32
复制代码

```

然后从右往左依次计算，如果当前位的结果大于等于 10，则需要往左边进一位

那么 ts 要如何知道 1 + 1 = 2，1 + 2 = 3 呢？

我们可以定义一个映射表，二维元组，用索引访问的方式得到一位整数加法的结果和它的进位情况

### 实现

#### 什么是一个形如数字的字符串类型

如 `"0.1"`、`"1"` 即是，用 ts 表示即 `${number}`

但是这样的数字也是符合这个条件的：`"000.1"`

如果想限制这样的数，用正则表达式很好处理，如何在 ts 类型系统中限制呢？

我们可以定义除了小数点前面有多个零的情况的类型

```
// 每位数
type Numbers = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9

// 开头不能是多个0
type AdvancedNumericCharacters =
  | `${0}.${number}`
  | `${Exclude<Numbers, 0>}${number | ""}.${number}`
  | `${Exclude<Numbers, 0>}${Numbers | ""}.${number}`
  | `${Exclude<Numbers, 0>}${number}`
  | `${Numbers}`
复制代码

```

#### 定义加法表

我们如果是 1 + 1，且加法表为 `AddMap`，我们想这样使用 `AddMap[1][1]`，得到相加的结果和需要进位的结果

即：

```
type AddMap = [
    [/* 0 + 0 */ { result: 0, add: 0 }, /* 0 + 1 */ {/* ... */} /* ... */]
    // ...
]
复制代码

```

#### 数据处理

小学三年级数学中的加法，要从右往左算，要以小数点对齐

但是在上文中我们实现的字符串或元组处理工具中，从右往左的都很麻烦，所以从左往右更简单，且元组操作要比字符串操作更方便，那么在实际的加法运算中

我们真实处理的数据应该是一个被反转的元组

1.  按小数点对齐

即按小数点分割，然后用 PadStart 和 PadEnd 补 0

```
// type Result = SplitByPoint<"1.02"> // ["1", "02"]
// 如果没有小数点，则小数位补 0 
type SplitByPoint<S extends AdvancedNumericCharacters> = string.Includes<
  S,
  "."
> extends true
  ? string.Split<S, ".">
  : [S, "0"]

// type Result = AddHelperSplitToArr<"1.02", "0.123"> // [["1", "02"], ["10", "123"]]
// 这里需要一起分割两个数字嘛
type AddHelperSplitToArr<
  S1 extends AdvancedNumericCharacters,
  S2 extends AdvancedNumericCharacters,
  Result = [SplitByPoint<S1>, SplitByPoint<S2>]
> = Result extends [[`${number}`, `${number}`], [`${number}`, `${number}`]]
  ? Result
  : never
复制代码

```

```
// type Result = AddFillZeroHelper<[["1", "02"], ["10", "123"]]> // [["01", "020"], ["10", "123"]]
// 对上面的结果用 PadStart 和 PadEnd 补 0
type AddFillZeroHelper<
  Data extends [[`${number}`, `${number}`], [`${number}`, `${number}`]],
  Result = [
    [
      string.PadStart<Data[0][0], string.GetStringLength<Data[1][0]>, "0">,
      string.PadEnd<Data[0][1], string.GetStringLength<Data[1][1]>, "0">
    ],
    [
      string.PadStart<Data[1][0], string.GetStringLength<Data[0][0]>, "0">,
      string.PadEnd<Data[1][1], string.GetStringLength<Data[0][1]>, "0">
    ]
  ]
> = Result extends [[`${number}`, `${number}`], [`${number}`, `${number}`]]
  ? Result
  : never
复制代码

```

2.  转元组后反转方便从左往右计算

```
// type Result = AddFillZeroHelper<[["1", "02"], ["10", "123"]]> 
// [[["1", "0"], ["0", "2", "0"]], [["0", "1"], ["3", "2", "1"]]]
type AddReverseData<
  Data extends [[`${number}`, `${number}`], [`${number}`, `${number}`]],
  Result = [
    [
      array.Reverse<string.Split<Data[0][0]>>,
      array.Reverse<string.Split<Data[0][1]>>
    ],
    [
      array.Reverse<string.Split<Data[1][0]>>,
      array.Reverse<string.Split<Data[1][1]>>
    ]
  ]
> = Result extends [
  [`${Numbers}`[], `${Numbers}`[]],
  [`${Numbers}`[], `${Numbers}`[]]
]
  ? Result
  : never
复制代码

```

#### 开始计算

单独计算小数位或整数位，减少复杂度，如果有进位，就在元组的最前面添加 `"10"`

```
type StepAdderHelper<
  DataLeft extends `${Numbers}`[], // 整数部分
  DataRight extends `${Numbers}`[], // 小数部分
  Curry extends `${Numbers}` = `${0}`, // 当前是否有进位
  Offset extends number = 0, // 循环的偏移量
  ResultCache extends `${number}`[] = [], // 用于缓存结果
  NextOffset extends number = number.IntAddSingle<Offset, 1>, // 偏移量加1
  Current extends AddMap[Numbers][Numbers] = AddMap[DataLeft[Offset]][DataRight[Offset]], // 当前的结果
  CurrentWidthPreCurry extends `${Numbers}` = AddMap[Current["result"]][Curry]["result"] // 当前的实际结果（加上进位）
> = DataLeft["length"] extends DataRight["length"]
  ? `${Offset}` extends `${DataLeft["length"]}`
    ? ResultCache
    : StepAdderHelper<
        DataLeft,
        DataRight,
        Current["add"],
        NextOffset,
        common.And<
          number.IsEqual<Current["add"], "1">,
          number.IsEqual<`${NextOffset}`, `${DataLeft["length"]}`>
        > extends true
          ? array.Push<["10", ...ResultCache], CurrentWidthPreCurry>
          : array.Push<ResultCache, CurrentWidthPreCurry>
      >
  : never
复制代码

```

#### 拼接结果

```
type NumbersWidthCurry = Numbers | 10

type MergeResultHelper<
  Data extends [
    [`${Numbers}`[], `${Numbers}`[]],
    [`${Numbers}`[], `${Numbers}`[]]
  ], // 处理后的的数据
  LeftInt extends `${Numbers}`[] = Data[0][0], // 加数的整数部分
  LeftFloat extends `${Numbers}`[] = Data[0][1], // 加数的小数部分
  RightInt extends `${Numbers}`[] = Data[1][0], // 被加数的整数部分
  RightFloat extends `${Numbers}`[] = Data[1][1], // 被加数的小数部分
  FloatAdded extends `${NumbersWidthCurry}`[] = StepAdderHelper<
    LeftFloat,
    RightFloat
  >, // 小数部分加法，附带进位的反序元组的结果
  FloatHasCurry extends boolean = FloatAdded[0] extends "10" ? true : false, // 小数是否有进位
  DeleteCurryFloatResult extends unknown[] = FloatHasCurry extends true
    ? array.Shift<FloatAdded>
    : FloatAdded, // 小数部分删除进位后的可以直接用的反序元组结果
  IntAdded extends `${NumbersWidthCurry}`[] = StepAdderHelper<
    LeftInt,
    RightInt,
    FloatHasCurry extends true ? `1` : "0"
  >, // 整数部分加法，初始会附带上小数部分的进位，结果会带上自己的进位
  IntHasCurry extends boolean = IntAdded[0] extends "10" ? true : false, // 整数部分是否还有进位
  DeleteCurryIntResult extends unknown[] = IntHasCurry extends true
    ? array.Shift<IntAdded>
    : IntAdded, // 整数部分删除进位后的可以直接用的反序元组结果
  ResultReversed = array.Reverse<
    LeftFloat["length"] extends 0
      ? DeleteCurryIntResult
      : array.Concat<
          [...DeleteCurryFloatResult, "."],
          [...DeleteCurryIntResult]
        >
  >, // 将整数小数（小数点）加入结果，并将反序的元组还原
  FloatResult = array.Join<
    ResultReversed extends string[]
      ? IntHasCurry extends true
        ? ["1", ...ResultReversed]
        : ResultReversed
      : never,
    ""
  > // 转字符串，并处理整数的进位
> = FloatResult

// 最终结果
type Add<
  S1 extends AdvancedNumericCharacters,
  S2 extends AdvancedNumericCharacters
> = MergeResultHelper<
  AddReverseData<AddFillZeroHelper<AddHelperSplitToArr<S1, S2>>>
>
复制代码

```

### 最终代码

```
type Numbers = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9

type AdvancedNumericCharacters =
  | `${0}.${number}`
  | `${Exclude<Numbers, 0>}${number | ""}.${number}`

type AddMap = [
  [
    { result: "0"; add: "0" }, // 00
    { result: "1"; add: "0" }, // 01
    { result: "2"; add: "0" }, // 02
    { result: "3"; add: "0" }, // 03
    { result: "4"; add: "0" }, // 04
    { result: "5"; add: "0" }, // 05
    { result: "6"; add: "0" }, // 06
    { result: "7"; add: "0" }, // 07
    { result: "8"; add: "0" }, // 08
    { result: "9"; add: "0" } // 09
  ],
  [
    { result: "1"; add: "0" }, // 10
    { result: "2"; add: "0" }, // 11
    { result: "3"; add: "0" }, // 12
    { result: "4"; add: "0" }, // 13
    { result: "5"; add: "0" }, // 14
    { result: "6"; add: "0" }, // 15
    { result: "7"; add: "0" }, // 16
    { result: "8"; add: "0" }, // 17
    { result: "9"; add: "0" }, // 18
    { result: "0"; add: "1" } // 19
  ],
  // ....
  [
    { result: "8"; add: "0" }, // 80
    { result: "9"; add: "0" }, // 81
    { result: "0"; add: "1" }, // 82
    { result: "1"; add: "1" }, // 83
    { result: "2"; add: "1" }, // 84
    { result: "3"; add: "1" }, // 85
    { result: "4"; add: "1" }, // 86
    { result: "5"; add: "1" }, // 87
    { result: "6"; add: "1" }, // 88
    { result: "7"; add: "1" } // 89
  ],
  [
    { result: "9"; add: "0" }, // 90
    { result: "0"; add: "0" }, // 91
    { result: "1"; add: "1" }, // 92
    { result: "2"; add: "1" }, // 93
    { result: "3"; add: "1" }, // 94
    { result: "4"; add: "1" }, // 95
    { result: "5"; add: "1" }, // 96
    { result: "6"; add: "1" }, // 97
    { result: "7"; add: "1" }, // 98
    { result: "8"; add: "1" } // 99
  ]
]

type SplitByPoint<S extends AdvancedNumericCharacters> = string.Includes<
  S,
  "."
> extends true
  ? string.Split<S, ".">
  : [S, "0"]

type AddHelperSplitToArr<
  S1 extends AdvancedNumericCharacters,
  S2 extends AdvancedNumericCharacters,
  Result = [SplitByPoint<S1>, SplitByPoint<S2>]
> = Result extends [[`${number}`, `${number}`], [`${number}`, `${number}`]]
  ? Result
  : never

type AddFillZeroHelper<
  Data extends [[`${number}`, `${number}`], [`${number}`, `${number}`]],
  Result = [
    [
      string.PadStart<Data[0][0], string.GetStringLength<Data[1][0]>, "0">,
      string.PadEnd<Data[0][1], string.GetStringLength<Data[1][1]>, "0">
    ],
    [
      string.PadStart<Data[1][0], string.GetStringLength<Data[0][0]>, "0">,
      string.PadEnd<Data[1][1], string.GetStringLength<Data[0][1]>, "0">
    ]
  ]
> = Result extends [[`${number}`, `${number}`], [`${number}`, `${number}`]]
  ? Result
  : never

type AddReverseData<
  Data extends [[`${number}`, `${number}`], [`${number}`, `${number}`]],
  Result = [
    [
      array.Reverse<string.Split<Data[0][0]>>,
      array.Reverse<string.Split<Data[0][1]>>
    ],
    [
      array.Reverse<string.Split<Data[1][0]>>,
      array.Reverse<string.Split<Data[1][1]>>
    ]
  ]
> = Result extends [
  [`${Numbers}`[], `${Numbers}`[]],
  [`${Numbers}`[], `${Numbers}`[]]
]
  ? Result
  : never

type StepAdderHelper<
  DataLeft extends `${Numbers}`[],
  DataRight extends `${Numbers}`[],
  Curry extends `${Numbers}` = `${0}`,
  Offset extends number = 0,
  ResultCache extends `${number}`[] = [],
  NextOffset extends number = number.IntAddSingle<Offset, 1>,
  Current extends AddMap[Numbers][Numbers] = AddMap[DataLeft[Offset]][DataRight[Offset]],
  CurrentWidthPreCurry extends `${Numbers}` = AddMap[Current["result"]][Curry]["result"]
> = DataLeft["length"] extends DataRight["length"]
  ? `${Offset}` extends `${DataLeft["length"]}`
    ? ResultCache
    : StepAdderHelper<
        DataLeft,
        DataRight,
        Current["add"],
        NextOffset,
        common.And<
          number.IsEqual<Current["add"], "1">,
          number.IsEqual<`${NextOffset}`, `${DataLeft["length"]}`>
        > extends true
          ? array.Push<["10", ...ResultCache], CurrentWidthPreCurry>
          : array.Push<ResultCache, CurrentWidthPreCurry>
      >
  : never

type NumbersWidthCurry = Numbers | 10

type MergeResultHelper<
  Data extends [
    [`${Numbers}`[], `${Numbers}`[]],
    [`${Numbers}`[], `${Numbers}`[]]
  ],
  LeftInt extends `${Numbers}`[] = Data[0][0],
  LeftFloat extends `${Numbers}`[] = Data[0][1],
  RightInt extends `${Numbers}`[] = Data[1][0],
  RightFloat extends `${Numbers}`[] = Data[1][1],
  FloatAdded extends `${NumbersWidthCurry}`[] = StepAdderHelper<
    LeftFloat,
    RightFloat
  >,
  FloatHasCurry extends boolean = FloatAdded[0] extends "10" ? true : false,
  DeleteCurryFloatResult extends unknown[] = FloatHasCurry extends true
    ? array.Shift<FloatAdded>
    : FloatAdded,
  IntAdded extends `${NumbersWidthCurry}`[] = StepAdderHelper<
    LeftInt,
    RightInt,
    FloatHasCurry extends true ? `1` : "0"
  >,
  IntHasCurry extends boolean = IntAdded[0] extends "10" ? true : false,
  DeleteCurryIntResult extends unknown[] = IntHasCurry extends true
    ? array.Shift<IntAdded>
    : IntAdded,
  ResultReversed = array.Reverse<
    LeftFloat["length"] extends 0
      ? DeleteCurryIntResult
      : array.Concat<
          [...DeleteCurryFloatResult, "."],
          [...DeleteCurryIntResult]
        >
  >,
  FloatResult = array.Join<
    ResultReversed extends string[]
      ? IntHasCurry extends true
        ? ["1", ...ResultReversed]
        : ResultReversed
      : never,
    ""
  >
> = FloatResult

type Add<
  S1 extends AdvancedNumericCharacters,
  S2 extends AdvancedNumericCharacters
> = MergeResultHelper<
  AddReverseData<AddFillZeroHelper<AddHelperSplitToArr<S1, S2>>>
>

type add = Add<"9007199254740991", "9007199254740991">
复制代码

```

> 相关文章推荐
> 
> [本文原文](https://link.juejin.cn?target=https%3A%2F%2Fyzl.xyz%2Flin%2F2022%2F01%2F%25E6%258E%25A5%25E8%25BF%2591%25E5%25A4%25A9%25E8%258A%25B1%25E6%259D%25BF%25E7%259A%2584TS%25E7%25B1%25BB%25E5%259E%258B%25E4%25BD%2593%25E6%2593%258D-%25E7%259C%258B%25E6%2587%2582%25E4%25BD%25A0%25E5%25B0%25B1%25E8%2583%25BD%25E7%258E%25A9%25E8%25BD%25ACTS%25E4%25BA%2586%2Fc7c8c77c59d1.html "https://yzl.xyz/lin/2022/01/%E6%8E%A5%E8%BF%91%E5%A4%A9%E8%8A%B1%E6%9D%BF%E7%9A%84TS%E7%B1%BB%E5%9E%8B%E4%BD%93%E6%93%8D-%E7%9C%8B%E6%87%82%E4%BD%A0%E5%B0%B1%E8%83%BD%E7%8E%A9%E8%BD%ACTS%E4%BA%86/c7c8c77c59d1.html")
> 
> [掘金：TypeScript 类型体操姿势合集 <通关总结>-- 刷完](https://juejin.cn/post/6999280101556748295 "https://juejin.cn/post/6999280101556748295")
> 
> [掘金：Ts 高手篇：22 个示例深入讲解 Ts 最晦涩难懂的高级类型工具](https://juejin.cn/post/6994102811218673700 "https://juejin.cn/post/6994102811218673700")
> 
> [掘金：来做操吧！深入 TypeScript 高级类型和类型体操](https://juejin.cn/post/7039856272354574372 "https://juejin.cn/post/7039856272354574372")
> 
> [掘金：模式匹配 - 让你 ts 类型体操水平暴增的套路](https://juejin.cn/post/7045536402112512007 "https://juejin.cn/post/7045536402112512007")