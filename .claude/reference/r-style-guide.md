# R 代码风格指南

基于 [Tidyverse Style Guide](https://style.tidyverse.org/) 的核心要点整理。

## 核心原则

**一致性是首要目标**。虽然某些风格选择确实能提高代码可用性（如缩进匹配程序结构），但许多选择本质上是任意的。标准化约定的主要价值在于减少决策负担。

## 命名约定

### 对象命名
- 只使用小写字母、数字和下划线
- 使用 `snake_case` 分隔单词（如 `day_one`）
- 避免在函数/变量名中使用点号，仅在 S3 方法中使用
- **变量用名词，函数用动词**
- 名称简洁且有意义

```r
# 好的示例
day_one
calculate_streak
user_habits

# 不推荐
DayOne         # 不用驼峰命名
day.one        # 不用点号
calculate.streak  # 函数名不用点号
d1             # 太简短不明确
```

### 函数命名
```r
# 好：动词形式
add_row()
permute()
calculate_streak()

# 不推荐：名词形式
row_adder()
permutation()
streak_calculator()
```

## 语法规则

### 赋值运算符
```r
# 使用 <- 进行赋值
x <- 5

# 不使用 =
x = 5  # 不推荐
```

### 空格规则

**逗号：** 逗号后面加空格，前面不加
```r
# 好
x[, 1]
mean(x, na.rm = TRUE)

# 不好
x[,1]
x[ ,1]
mean(x,na.rm=TRUE)
```

**括号：**
- 常规函数调用内外不加空格
- `if`、`for`、`while` 前后加空格
- 函数参数中，`()` 后加空格

```r
# 好
mean(x, na.rm = TRUE)
if (x > 10) { }
for (i in 1:10) { }
function(x) x + 1

# 不好
mean (x, na.rm = TRUE)
if(x > 10) { }
function (x) x + 1
```

**中缀运算符：** 大多数运算符两侧加空格
```r
# 需要空格的运算符
x <- 5
x == y
a + b
a - b
a * b
a / b
x %in% y

# 不需要空格的高优先级运算符
package::function
x$field
x@slot
x[1]
x[[1]]
x^2
-x
+x
1:10

# 简单公式
~foo

# Bang 运算符
!!x
!!!list
```

### 缩进和行长度

**缩进：** 使用 2 个空格
```r
if (y < 0) {
  message("y is negative")
}

function(x) {
  # 函数体缩进 2 个空格
  x + 1
}
```

**行长度：** 限制在 80 个字符以内

**大括号：**
- 左大括号 `{` 应该在行末
- 右大括号 `}` 应该独占一行

```r
# 好
if (y == 0) {
  log(x)
} else {
  y^x
}

# 不好
if (y == 0) { log(x) }
else { y^x }
```

### 长函数调用
对于长函数调用，每个参数占一行：

```r
# 好
do_something_very_complicated(
  something = "that",
  requires = many,
  arguments = "some of which may be long"
)

# 也可以使用悬挂缩进
do_something_very_complicated(something = "that",
                               requires = many,
                               arguments = "some of which")
```

### 函数参数
- 常用数据参数可省略名称
- 其他参数必须命名
- 避免部分参数匹配
- 不要在函数调用内赋值（除非是副作用函数）

```r
# 好
mean(x, na.rm = TRUE)
plot(x, y, type = "l")

# 不好
mean(x, n = TRUE)        # 部分匹配
mean(x = data$x, na = TRUE)  # data 参数也命名了
mean(x <- 1:10)          # 在函数调用内赋值
```

## 函数编写

### 匿名函数
短小的匿名函数使用现代 lambda 语法：
```r
# 好
map(x, \(x) x + 1)

# 旧式语法（可用但不推荐）
map(x, ~.x + 1)

# 多行或命名函数使用 function()
map(x, function(x) {
  y <- x + 1
  z <- y * 2
  z
})
```

### 返回语句
- 仅在提前退出时使用 `return()`
- 否则让 R 返回最后一个表达式的结果
- `return()` 语句应独占一行

```r
# 好
calculate_value <- function(x) {
  if (x < 0) {
    return(0)
  }

  # 最后一个表达式自动返回
  x^2 + 2*x + 1
}

# 不推荐
calculate_value <- function(x) {
  if (x < 0) {
    return(0)
  } else {
    return(x^2 + 2*x + 1)  # 不需要 return
  }
}
```

### 副作用函数
对于主要用于副作用的函数，使用 `invisible()` 返回第一个参数：
```r
print_and_modify <- function(x) {
  print(x)
  x <- x + 1
  invisible(x)  # 允许在管道中使用
}
```

## 控制流

### 条件语句
- 单行 `if` 语句仅用于简单、无副作用的代码
- 多行 `if` 语句必须使用大括号
- 条件中使用 `&&` 和 `||`，不使用 `&` 或 `|`

```r
# 好：单行简单条件
if (x > 0) y <- 1

# 好：使用大括号
if (x > 0) {
  message("Positive")
  y <- 1
}

# 好：多个条件
if (x > 0 && y > 0) {
  z <- x + y
}

# 不好
if (x > 0 & y > 0) { }  # 不要用向量化的 &
```

### 循环
循环体必须使用大括号表达式：
```r
# 好
for (i in 1:10) {
  print(i)
}

# 不好
for (i in 1:10) print(i)
```

### 控制流语句
`return()`、`stop()`、`break`、`next` 应该在独立的大括号块中：
```r
# 好
if (x < 0) {
  stop("x must be positive")
}

for (i in seq_along(x)) {
  if (x[i] < 0) {
    next
  }
  # 处理正数
}

# 不好
if (x < 0) stop("x must be positive")
```

## 数据类型

### 字符串
优先使用双引号 `"`：
```r
# 好
x <- "Hello, world"

# 也可以，但不优先
x <- 'Hello, world'
```

### 逻辑值
使用 `TRUE`/`FALSE`，不使用 `T`/`F`：
```r
# 好
x <- TRUE
y <- FALSE

# 不好（可被重新赋值）
x <- T
y <- F
```

## 注释

### 注释格式
- 以 `# ` 开始（井号 + 一个空格）
- 使用句首大写
- 只有多句评论才需要句号

```r
# 好：解释"为什么"
# 使用 bootstrap 方法因为样本量小
result <- bootstrap(data)

# 不好：说明"是什么"（代码已经说明了）
# 计算平均值
x <- mean(data)
```

### 注释原则
解释代码背后的"为什么"，而不是"是什么"或"怎么做"。

```r
# 好
# 需要手动指定种子以确保可重复性
set.seed(123)
sample_data <- rnorm(100)

# 不好
# 设置随机种子为 123
set.seed(123)
# 生成 100 个正态分布随机数
sample_data <- rnorm(100)
```

## 其他规则

### 分号
完全避免使用分号：
```r
# 好
x <- 5
y <- 10

# 不好
x <- 5; y <- 10
```

### 额外空格
可以添加额外空格来对齐赋值运算符：
```r
# 允许
very_long_name    <- 1
short_name        <- 2
medium_name       <- 3
```

## 工具支持

使用这些包来自动检查和格式化代码：

```r
# 自动格式化代码
styler::style_pkg()
styler::style_file("R/my-script.R")

# 检查代码风格
lintr::lint_package()
lintr::lint("R/my-script.R")
```

## 快速检查清单

- [ ] 函数名使用动词，变量名使用名词
- [ ] 使用 `snake_case` 命名
- [ ] 使用 `<-` 赋值
- [ ] 逗号后有空格
- [ ] 中缀运算符两侧有空格
- [ ] 使用 2 个空格缩进
- [ ] 行长度不超过 80 字符
- [ ] `{` 在行末，`}` 独占一行
- [ ] 所有导出函数都有 roxygen2 文档
- [ ] 注释解释"为什么"而不是"是什么"
- [ ] 使用 `TRUE`/`FALSE` 而不是 `T`/`F`
- [ ] 条件语句中使用 `&&`/`||` 而不是 `&`/`|`
