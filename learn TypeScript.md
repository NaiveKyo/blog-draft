## Summary

Reference：

- [TypeScript ](https://www.typescriptlang.org/)

- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [TypeScript Deep Dive Chinese Edition](https://jkchao.github.io/typescript-book-chinese/)
- [TypeScript Community](https://github.com/TypeStrong)



## TypeScript Deep Dive

TypeScript is just JavaScript with docs.

TypeScript 可以看作是 JavaScript 的 Superset，JavaScript 是它的一部分，同时 TypeScript 还提供更多的特性和语法。

TypeScript 是强类型的语言，且可以利用编译器提前发现程序中的错误。

### JavaScript

#### Equality

> 1、Equality

JavaScript 中 `==` 和 `===` 两种运算符，前者会进行强制类型转换，转为相同类型后进行比较，而在 TypeScript 中这就属于 compile time errors。

注意：除了 null 值的比较外，都推荐使用 `===` 和 `!==`。



> 2、Structural Equality

