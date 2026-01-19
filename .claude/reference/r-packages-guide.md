# R 包开发核心指南

基于 [R Packages (2e)](https://r-pkgs.org/) 的核心概念和最佳实践整理。

## 核心理念

> "包是可重复使用的 R 代码的基本单位。"

包是共享可重用函数、文档和示例数据的基本机制。**不需要从一开始就完美**——专注于创建功能性包，随着时间推移逐步改进。

## 包的组成部分

### 1. 基本结构

```
my-package/
├── R/              # R 代码
├── man/            # 文档（自动生成）
├── tests/          # 测试
├── DESCRIPTION     # 包元数据
├── NAMESPACE       # 导出/导入（自动生成）
├── LICENSE         # 许可证
└── README.md       # 项目说明
```

### 2. DESCRIPTION 文件

包的元数据文件，包含：
- **Package**: 包名（只能包含字母、数字、点号）
- **Version**: 版本号（如 0.1.0）
- **Title**: 单行简短描述
- **Authors@R**: 作者信息
- **Description**: 详细描述（可多行）
- **License**: 许可证
- **Imports**: 依赖的包
- **Suggests**: 建议的包（非必需）
- **Depends**: R 版本要求

示例：
```dcf
Package: habittracker
Version: 0.1.0
Title: Track and Analyze Habit Data
Authors@R: person("Jane", "Doe", email = "jane@example.com", role = c("aut", "cre"))
Description: Provides tools for tracking daily habits and analyzing behavior patterns.
    Includes functions for streak calculation and visualization.
License: MIT + file LICENSE
Encoding: UTF-8
Depends: R (>= 4.0.0)
Imports:
    dplyr (>= 1.0.0),
    ggplot2
Suggests:
    testthat (>= 3.0.0),
    knitr,
    rmarkdown
```

### 3. NAMESPACE

**不要手动编辑！** 使用 roxygen2 自动生成：

```r
# 在函数文档中使用 @export
#' @export
calculate_streak <- function(data) {
  # ...
}

# 运行 devtools::document() 自动生成 NAMESPACE
```

## 依赖管理

### 依赖类型

1. **Imports**: 包运行所必需的
   - 在 DESCRIPTION 中列出
   - 在代码中使用 `package::function()` 调用

2. **Suggests**: 推荐但非必需的（用于示例、测试、vignettes）
   - 使用前检查是否安装
   - 测试框架（testthat）通常放在 Suggests

3. **Depends**: 尽量避免使用
   - 仅用于指定最低 R 版本
   - 或者包是另一个包的扩展

### 使用依赖包

**推荐方式：** 显式命名空间调用
```r
#' @importFrom dplyr filter mutate
calculate_streak <- function(data) {
  # 使用 package::function() 语法
  data <- dplyr::filter(data, completed == TRUE)
  data <- dplyr::mutate(data, streak = calculate_days(date))
  data
}
```

**不推荐：** 使用 @import 导入整个包
```r
# 不推荐：污染命名空间
#' @import dplyr
```

### 添加依赖

```r
# 将包添加到 Imports
usethis::use_package("dplyr")

# 将包添加到 Suggests
usethis::use_package("testthat", type = "Suggests")
```

## R 代码组织

### 文件命名
- 使用 `snake_case.R`
- 有意义的名称反映文件内容
- 通用工具函数放在 `utils.R`
- 包级文档放在 `{package}-package.R`

```
R/
├── utils.R              # 通用辅助函数
├── streak.R             # 计算连续性
├── visualization.R      # 可视化函数
└── habittracker-package.R  # 包级文档
```

### 函数组织原则
- 每个文件包含相关功能
- 一个主要的导出函数 + 相关辅助函数
- 内部辅助函数不使用 @export
- 保持函数简短、单一职责

```r
# streak.R

#' Calculate habit streak
#' @export
calculate_streak <- function(data) {
  validate_streak_data(data)
  compute_streak_length(data)
}

# 内部辅助函数（不导出）
validate_streak_data <- function(data) {
  if (!is.data.frame(data)) {
    stop("data must be a data.frame")
  }
  invisible(data)
}

compute_streak_length <- function(data) {
  # 实际计算逻辑
}
```

## 文档

### 函数文档（roxygen2）

必需标签：
- `@param`: 参数描述
- `@return`: 返回值描述
- `@examples`: 使用示例
- `@export`: 导出函数

推荐标签：
- `@description`: 详细描述
- `@details`: 额外细节
- `@seealso`: 相关函数
- `@references`: 参考文献

```r
#' Calculate current habit streak
#'
#' @description
#' Calculates the number of consecutive days a habit has been
#' completed up to the current date.
#'
#' @param data A data.frame with columns: `date` (Date) and
#'   `completed` (logical)
#' @param current_date The date to calculate up to. Defaults to today.
#'
#' @return An integer representing the current streak length
#'
#' @examples
#' habit_data <- data.frame(
#'   date = seq.Date(Sys.Date() - 10, Sys.Date(), by = "day"),
#'   completed = c(rep(TRUE, 8), FALSE, TRUE, TRUE)
#' )
#' calculate_streak(habit_data)
#' # Returns: 2
#'
#' @seealso [calculate_longest_streak()] for finding the longest
#'   historical streak
#'
#' @export
calculate_streak <- function(data, current_date = Sys.Date()) {
  # Implementation
}
```

### 包级文档

在 `R/{package}-package.R` 中：
```r
#' habittracker: Track and analyze daily habits
#'
#' @description
#' The habittracker package provides tools for tracking daily habits
#' and analyzing behavioral patterns over time.
#'
#' @section Main functions:
#' * [calculate_streak()] - Calculate current streak
#' * [plot_habits()] - Visualize habit completion
#' * [summarize_habits()] - Generate summary statistics
#'
#' @docType package
#' @name habittracker-package
#' @aliases habittracker
#' @keywords internal
"_PACKAGE"
```

### README 文件

使用 README.Rmd 生成 README.md：
```r
# 创建 README.Rmd
usethis::use_readme_rmd()

# 编辑 README.Rmd，然后构建
devtools::build_readme()
```

README 应包含：
- 包的目的
- 安装说明
- 基本使用示例
- 徽章（测试覆盖率、构建状态等）

### Vignettes（长篇文档）

为复杂功能创建详细教程：
```r
# 创建 vignette
usethis::use_vignette("introduction")

# 构建所有 vignettes
devtools::build_vignettes()
```

## 测试

### 测试结构

```
tests/
├── testthat/
│   ├── helper.R          # 测试辅助函数
│   ├── setup.R           # 一次性设置
│   ├── test-streak.R     # 测试 streak 功能
│   └── test-utils.R      # 测试工具函数
└── testthat.R            # 测试运行器
```

### 测试原则

1. **测试所有导出函数**
2. **测试边界条件**
   - 空输入
   - NULL 值
   - 极端值
   - 错误类型

3. **测试错误处理**
   - 验证错误消息
   - 确保适当的错误类型

4. **保持测试独立**
   - 不依赖测试顺序
   - 不共享状态
   - 清理临时文件

### 测试覆盖率

目标：
- **整体覆盖率 > 80%**
- **关键函数覆盖率 = 100%**

```r
# 检查覆盖率
covr::package_coverage()

# 生成报告
covr::report()

# 查看特定文件覆盖率
covr::file_coverage("R/streak.R", "tests/testthat/test-streak.R")
```

## 数据

### 包数据（data/）

为用户提供示例数据：
```r
# 创建示例数据
habit_example <- data.frame(
  date = seq.Date(as.Date("2024-01-01"), as.Date("2024-01-31"), by = "day"),
  habit = "exercise",
  completed = sample(c(TRUE, FALSE), 31, replace = TRUE)
)

# 保存为包数据
usethis::use_data(habit_example, overwrite = TRUE)
```

在 `R/data.R` 中文档化：
```r
#' Example habit tracking data
#'
#' A sample dataset containing 31 days of habit tracking.
#'
#' @format A data frame with 31 rows and 3 variables:
#' \describe{
#'   \item{date}{Date of the record}
#'   \item{habit}{Name of the habit being tracked}
#'   \item{completed}{Logical indicating if habit was completed}
#' }
#'
#' @examples
#' data(habit_example)
#' head(habit_example)
"habit_example"
```

### 原始数据（data-raw/）

用于生成包数据的脚本：
```r
# 设置 data-raw
usethis::use_data_raw("habit_example")

# 在 data-raw/habit_example.R 中：
habit_example <- data.frame(...)
usethis::use_data(habit_example, overwrite = TRUE)
```

## 版本管理

### 版本号规则

使用语义化版本：`major.minor.patch`

- **Major (1.0.0)**: 破坏性变更
- **Minor (0.1.0)**: 新功能（向后兼容）
- **Patch (0.0.1)**: 错误修复

开发版本：
- `0.0.0.9000` - 开发中
- `0.1.0` - 第一个发布版本

```r
# 更新版本
usethis::use_version("patch")  # 0.1.0 -> 0.1.1
usethis::use_version("minor")  # 0.1.1 -> 0.2.0
usethis::use_version("major")  # 0.2.0 -> 1.0.0
usethis::use_version("dev")    # 1.0.0 -> 1.0.0.9000
```

### NEWS.md

记录每个版本的变更：
```r
# 创建 NEWS.md
usethis::use_news_md()
```

格式：
```markdown
# habittracker 0.2.0

## New features
* Added `calculate_longest_streak()` function
* Support for multiple habit tracking

## Bug fixes
* Fixed streak calculation for months with 31 days (#12)
* Corrected timezone handling in date processing

## Breaking changes
* Renamed `calc_streak()` to `calculate_streak()`

# habittracker 0.1.0

* Initial CRAN release
```

## 许可证

### 常见许可证

```r
# MIT 许可证（宽松）
usethis::use_mit_license()

# GPL-3（传染性）
usethis::use_gpl3_license()

# CC0（公共域）
usethis::use_cc0_license()
```

选择建议：
- **MIT/Apache**: 商业友好
- **GPL**: 要求衍生作品开源
- **CC0**: 完全放弃版权

## 发布流程

### 准备发布

1. **完整检查**
```r
devtools::check()  # 必须 0 errors, 0 warnings, 0 notes
```

2. **更新版本和 NEWS**
```r
usethis::use_version("minor")
# 编辑 NEWS.md
```

3. **拼写检查**
```r
devtools::spell_check()
```

4. **发布前检查**
```r
devtools::release_checks()
```

5. **构建网站**
```r
pkgdown::build_site()
```

### CRAN 提交

```r
# 构建源码包
devtools::build()

# 提交到 CRAN
devtools::submit_cran()
```

CRAN 检查点：
- 通过所有自动检查
- 遵循 CRAN 政策
- 有效的 LICENSE 文件
- 所有示例可运行（< 5 秒）
- 无未声明的依赖

## 持续集成

### GitHub Actions

```r
# 设置 R CMD check 工作流
usethis::use_github_action("check-standard")

# 设置测试覆盖率报告
usethis::use_github_action("test-coverage")

# 设置 pkgdown 网站部署
usethis::use_github_action("pkgdown")
```

### 徽章

在 README 中添加：
```r
usethis::use_lifecycle_badge("Experimental")
usethis::use_github_actions_badge()
usethis::use_coverage()  # 需要配置 codecov
```

## 包的生命周期

### 开发阶段

1. **Experimental (实验性)**: 早期开发，API 可能大幅变化
2. **Stable (稳定)**: API 稳定，推荐生产使用
3. **Deprecated (弃用)**: 计划移除，但仍可用
4. **Superseded (被取代)**: 有更好的替代方案
5. **Archived (归档)**: 不再维护

```r
# 标记函数生命周期
#' @lifecycle stable
calculate_streak <- function() { }

#' @lifecycle deprecated
#' @description `calc_streak()` was renamed to `calculate_streak()`.
calc_streak <- function() {
  lifecycle::deprecate_warn("0.2.0", "calc_streak()", "calculate_streak()")
  calculate_streak()
}
```

## 最佳实践总结

1. **从简单开始**：不需要一开始就完美
2. **频繁运行 check**：`devtools::check()` 应该始终通过
3. **编写测试**：目标覆盖率 > 80%
4. **文档化一切**：所有导出函数需要 @examples
5. **使用工具**：devtools、usethis、testthat、roxygen2
6. **版本控制**：使用 Git + GitHub
7. **自动化**：设置 CI/CD 工作流
8. **迭代改进**：逐步完善包的质量

## 快速参考

| 任务 | 命令 |
|------|------|
| 创建新包 | `usethis::create_package("path/to/package")` |
| 添加函数 | `usethis::use_r("function-name")` |
| 添加测试 | `usethis::use_test("function-name")` |
| 添加依赖 | `usethis::use_package("dplyr")` |
| 添加数据 | `usethis::use_data(dataset)` |
| 添加 vignette | `usethis::use_vignette("intro")` |
| 添加 README | `usethis::use_readme_rmd()` |
| 添加许可证 | `usethis::use_mit_license()` |
| 设置 Git | `usethis::use_git()` |
| 设置 GitHub | `usethis::use_github()` |
| 设置 CI | `usethis::use_github_action("check-standard")` |

## 学习资源

- [R Packages (2e)](https://r-pkgs.org/) - 完整指南
- [Writing R Extensions](https://cran.r-project.org/doc/manuals/r-release/R-exts.html) - 官方 CRAN 文档
- [rOpenSci Packages Guide](https://devguide.ropensci.org/) - 高质量包开发指南
