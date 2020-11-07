# typescript

1、`react-app-env.d.ts` 声明文件

```ts
/// <reference types="react-scripts" />
```

三斜线指令是包含单个XML标签的单行注释。 注释的内容会做为编译器指令使用。

三斜线指令仅可放在包含它的文件的最顶端。 一个三斜线指令的前面只能出现单行或多行注释，这包括其它的三斜线指令。 如果它们出现在一个语句或声明之后，那么它们会被当做普通的单行注释，并且不具有特殊的涵义。

```ts
/// <reference path="..." />
/// <reference path="..." />指令是三斜线指令中最常见的一种。 它用于声明文件间的 依赖。
```

三斜线引用告诉编译器在编译过程中要引入的额外的文件。

当使用 `--out` 或 `--outFile` 时，它也可以做为调整输出内容顺序的一种方法。 文件在输出文件内容中的位置与经过预处理后的输入顺序一致。


2、`React.FC`

```
FC = Functional Component
SFC = Stateless Functional Component (已弃用)
```

