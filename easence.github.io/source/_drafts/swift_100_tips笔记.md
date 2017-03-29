---
title: swift 100 tips笔记.md
categories:
- Apple Development
- iOS开发笔记
tags:
- swift
---
## 笔记来源
来自喵神的[《swifter》][1]的要点记录。

## 要点
1. 柯里化，即将接受**多个参数**的方法转换成接受**一个参数**的的方法。
2. Struct `Mutating`关键字，由于Struct是immutable的数据结构，如果想**在方法体中修改局部变量的值**，则需要在func 前面加上`Mutating`关键字。
3. Protocol方法的`Mutating`关键字，为了**能使Struct、enum能在方法中修改它们的局部变量的值**，多考虑在func 前面加上`Mutating`关键字。`Class`方法的前面不需要加`Mutating`关键字。
4. 传入的闭包，为了避免引用循环，可以在参数列表前加上[weak self, weak obj]这样的标注。
5. 当作为参数的闭包，在这个函数执行完之后，依然还会被其他地方使用，可以使用`@escaping`关键字修饰这个闭包参数。例如：在网络请求中，传输的结果处理闭包，往往是异步调用的，因此应当使用`@escaping`修饰传入的闭包。


---
[1]: https://www.amazon.cn/Swifter-100个Swift-2开发必备Tip-王巍/dp/B019CRN7TW/ref=sr_1_1?ie=UTF8&qid=1477880062&sr=8-1&keywords=swifter

