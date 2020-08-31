
# Jenkins 声明式与脚本式语法

## 1. 声明式Pipeline

声明式Pipleine是最近添加到Jenkins流水线的，它在流水线子系统之上提供了一种更简单，更有主见的语法。
所有的声明式Pipeline都必须包含一个   `pipeline`块中，比如：

```groovy
pipeline {
    //run
}
```

在声明式Pipeline中的基本语句和表达式遵循Groovy的语法。但是有以下例外：
- 流水线顶层必须是一个块，特别是pipelin{}。
- 不需要分号作为分割符，是按照行分割的。
- 语句块只能由阶段、指令、步骤、赋值语句组成。例如: input被视为input()。
