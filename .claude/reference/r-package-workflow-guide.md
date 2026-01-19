# R Package Development Workflow Guide

This comprehensive guide provides detailed instructions, examples, and best practices for R package development in the habit-tracker project. For quick reference, see `CLAUDE.md`.

## Daily Development Cycle

Follow this iterative cycle when developing your package:

### 1. Load and Develop
```r
# Start RStudio in your package directory
devtools::load_all()  # Ctrl+Shift+L
```

This loads your package functions into memory, similar to `library()`, but for development. You can now interactively test your functions in the console.

### 2. Write Code
- Add new functions to files in `R/`
- Follow naming conventions: `snake_case` for everything
- Document as you go using roxygen2 comments (`#'`)

### 3. Document
```r
devtools::document()  # Ctrl+Shift+D
```

This generates documentation from your roxygen2 comments and updates `NAMESPACE`.

### 4. Test
```r
devtools::test()  # Ctrl+Shift+T
```

Run your tests frequently. Write tests as you write code (TDD approach).

### 5. Check
```r
devtools::check()  # Ctrl+Shift+E
```

Run a full R CMD check before committing changes. This catches many common issues.

## Key Principles

### 1. Write Tests First (TDD)
Before writing a new function:
1. Create the test file: `usethis::use_test("function-name")`
2. Write a failing test that describes what you want
3. Implement the function until the test passes
4. Refactor if needed

**Benefits:**
- Clearer function design
- Better edge case handling
- Built-in regression testing

### 2. Document Everything
Use roxygen2 for all exported functions:
```r
#' Brief title
#'
#' @description Longer description of what this does
#'
#' @param param_name Description of parameter
#'
#' @return Description of return value
#'
#' @examples
#' example_code()
#'
#' @export
your_function <- function(param_name) {
  # Implementation
}
```

### 3. Small, Focused Functions
- Each function should do one thing well
- Easier to test, understand, and maintain
- Extract complex logic into helper functions

### 4. Validate Inputs
For exported functions, validate inputs explicitly:
```r
my_function <- function(data) {
  if (!is.data.frame(data)) {
    stop("data must be a data.frame")
  }
  if (!"required_col" %in% names(data)) {
    stop("data must contain 'required_col' column")
  }
  # Rest of function
}
```

### 5. Use Consistent Style
Run these periodically:
```r
styler::style_pkg()    # Auto-format code
lintr::lint_package()  # Check for style issues
```

## Common Workflows

### Adding a New Function

```r
# 1. Create R file (if new area of functionality)
usethis::use_r("new-feature")

# 2. Create test file
usethis::use_test("new-feature")

# 3. Write test first
test_that("new_function does X", {
  expect_equal(new_function(input), expected_output)
})

# 4. Run test (should fail)
devtools::test()

# 5. Write function with roxygen2 docs
#' Title
#' @export
new_function <- function(x) {
  # Implementation
}

# 6. Document
devtools::document()

# 7. Load and test interactively
devtools::load_all()
new_function(test_input)

# 8. Run tests (should pass)
devtools::test()

# 9. Check package
devtools::check()
```

### Fixing a Bug

```r
# 1. Write a failing test that reproduces the bug
usethis::use_test("buggy-function")
test_that("function handles edge case X", {
  expect_equal(buggy_function(edge_case), expected)
})

# 2. Run test to confirm it fails
devtools::test()

# 3. Fix the bug in R/buggy-function.R

# 4. Test until it passes
devtools::test()

# 5. Full check
devtools::check()
```

### Preparing for Release

```r
# 1. Check everything
devtools::check()

# 2. Update version in DESCRIPTION
usethis::use_version()

# 3. Update NEWS.md
usethis::use_news_md()
# Add section for new version with changes

# 4. Check spell
devtools::spell_check()

# 5. Run release checks
devtools::release_checks()

# 6. Build pkgdown site
pkgdown::build_site()

# 7. Final check
devtools::check()
```

## Testing Best Practices

### What to Test
- All exported functions
- Edge cases (empty inputs, NULLs, extreme values)
- Error conditions
- Data type handling

### What NOT to Test
- Third-party package behavior (trust they work)
- Trivial getters/setters
- R base functionality

### Test Coverage Goals
- Aim for >80% overall coverage
- Critical functions should have 100% coverage
- Check with: `covr::package_coverage()`

### Test Organization
- One test file per R file: `R/utils.R` → `tests/testthat/test-utils.R`
- Group related tests with descriptive `test_that()` names
- Use helper functions in `tests/testthat/helper.R`

## Common Pitfalls

### 1. Forgetting to Document
**Problem:** Function works but has no documentation
**Solution:** Run `devtools::document()` after writing roxygen2 comments

### 2. Not Running Check
**Problem:** Package works locally but fails R CMD check
**Solution:** Run `devtools::check()` regularly, especially before committing

### 3. Tests That Don't Clean Up
**Problem:** Tests leave artifacts or change global state
**Solution:** Use `withr::local_*()` functions or ensure cleanup in tests

### 4. Hardcoded Paths
**Problem:** Tests fail on other machines
**Solution:** Use `testthat::test_path()` and `system.file()` for paths

### 5. Not Testing Edge Cases
**Problem:** Function works for normal inputs but fails on edge cases
**Solution:** Write tests for NULL, empty, NA, extreme values, wrong types

## RStudio Keyboard Shortcuts

Master these for efficient development:
- `Ctrl+Shift+L` - Load all (devtools::load_all())
- `Ctrl+Shift+D` - Document (devtools::document())
- `Ctrl+Shift+T` - Test (devtools::test())
- `Ctrl+Shift+E` - Check (devtools::check())
- `Ctrl+Shift+B` - Build and Reload
- `Ctrl+.` - Go to file/function
- `F2` - Jump to function definition

## Resources

- [R Packages (2e)](https://r-pkgs.org/) - THE guide for package development
- [testthat documentation](https://testthat.r-lib.org/) - Testing details
- [roxygen2 documentation](https://roxygen2.r-lib.org/) - Documentation syntax
- [Tidyverse Style Guide](https://style.tidyverse.org/) - Code style
- [Writing R Extensions](https://cran.r-project.org/doc/manuals/r-release/R-exts.html) - Official CRAN guide

## Complete R Command Reference

### Setup & Dependencies
```r
# Install development tools (run once)
install.packages(c("devtools", "usethis", "roxygen2", "testthat", "pkgdown"))

# Install package dependencies
devtools::install_deps()
devtools::install_deps(dependencies = TRUE)
```

### Development Commands
```r
devtools::load_all()     # Load package (Ctrl+Shift+L)
devtools::document()     # Generate docs (Ctrl+Shift+D)
devtools::test()         # Run tests (Ctrl+Shift+T)
devtools::check()        # Full check (Ctrl+Shift+E)
```

### Code Quality
```r
styler::style_pkg()      # Auto-format code
lintr::lint_package()    # Check for style issues
```

### Testing & Coverage
```r
covr::package_coverage()                        # Check coverage
devtools::test_file("tests/testthat/test-*.R") # Test specific file
devtools::test_coverage()                       # Coverage for package
covr::report()                                  # Coverage report
```

### Documentation
```r
devtools::build_readme()      # Build README from .Rmd
pkgdown::build_site()         # Build pkgdown website
devtools::build_vignettes()   # Build vignettes
```

### Build & Install
```r
devtools::build()                # Build source package
devtools::build(binary = TRUE)   # Build binary package
devtools::install()              # Install locally
```

### Release Preparation
```r
devtools::release_checks()       # Pre-release checks
devtools::spell_check()          # Spell check
usethis::use_version()           # Bump version
usethis::use_news_md()           # Update NEWS.md
```

## Roxygen2 Documentation Template

```r
#' Brief function title (one line)
#'
#' @description
#' More detailed description of what the function does.
#' Can span multiple lines.
#'
#' @param param_name Description of the parameter
#' @param another_param Description of another parameter
#'
#' @return Description of what the function returns
#'
#' @examples
#' # Example usage
#' result <- my_function(data)
#'
#' @export
my_function <- function(param_name, another_param = NULL) {
  # Validate inputs
  if (!is.data.frame(param_name)) {
    stop("param_name must be a data.frame")
  }

  # Implementation
  result <- # your code

  return(result)
}
```

## Error Handling Patterns

### Input Validation
```r
validate_habit_data <- function(data) {
  if (!is.data.frame(data)) {
    stop("data must be a data.frame", call. = FALSE)
  }

  required_cols <- c("date", "completed")
  missing_cols <- setdiff(required_cols, names(data))

  if (length(missing_cols) > 0) {
    stop(
      "Missing required columns: ",
      paste(missing_cols, collapse = ", "),
      call. = FALSE
    )
  }

  invisible(data)
}
```

### Warnings for Non-Fatal Issues
```r
calculate_streak <- function(habit_data) {
  if (nrow(habit_data) == 0) {
    warning("Empty habit data provided, returning streak of 0")
    return(0)
  }
  # Implementation
}
```

### Messages for User Information
```r
process_data <- function(data, verbose = FALSE) {
  if (verbose) {
    message("Processing ", nrow(data), " records...")
  }
  # Implementation
}
```

## Data Management

### Package Data
```r
# Create example dataset
habit_example <- data.frame(
  date = seq.Date(as.Date("2024-01-01"), as.Date("2024-01-31"), by = "day"),
  habit = "exercise",
  completed = sample(c(TRUE, FALSE), 31, replace = TRUE)
)

# Save as package data
usethis::use_data(habit_example, overwrite = TRUE)

# Document in R/data.R
#' Example habit tracking data
#'
#' @format A data frame with 31 rows and 3 variables:
#' \describe{
#'   \item{date}{Date of the record}
#'   \item{habit}{Name of the habit}
#'   \item{completed}{Logical indicating completion}
#' }
"habit_example"
```

### Raw Data Scripts
```r
# Setup data-raw folder
usethis::use_data_raw("habit_example")

# Creates: data-raw/habit_example.R
```

### User Data Storage
```r
#' Get user data directory
#' @keywords internal
get_user_data_dir <- function() {
  dir <- rappdirs::user_data_dir("habittracker")
  if (!dir.exists(dir)) {
    dir.create(dir, recursive = TRUE)
  }
  dir
}
```

## Testing Examples

### Test Structure (Arrange-Act-Assert)
```r
test_that("calculate_streak returns correct values", {
  # Arrange
  habit_data <- data.frame(
    date = seq.Date(Sys.Date() - 5, Sys.Date(), by = "day"),
    completed = c(TRUE, TRUE, TRUE, FALSE, TRUE, TRUE)
  )

  # Act
  result <- calculate_streak(habit_data)

  # Assert
  expect_equal(result, 2)
  expect_type(result, "integer")
})
```

### Edge Case Testing
```r
test_that("calculate_streak handles edge cases", {
  # Empty data
  expect_equal(calculate_streak(data.frame()), 0)

  # Single row
  single <- data.frame(date = Sys.Date(), completed = TRUE)
  expect_equal(calculate_streak(single), 1)

  # All FALSE
  all_false <- data.frame(
    date = seq.Date(Sys.Date() - 5, Sys.Date(), by = "day"),
    completed = rep(FALSE, 6)
  )
  expect_equal(calculate_streak(all_false), 0)

  # NA values
  with_na <- data.frame(
    date = Sys.Date(),
    completed = NA
  )
  expect_warning(calculate_streak(with_na))
})
```

### Error Testing
```r
test_that("calculate_streak validates inputs", {
  expect_error(
    calculate_streak(NULL),
    "data must be a data.frame"
  )

  expect_error(
    calculate_streak(data.frame(x = 1)),
    "Missing required columns: date, completed"
  )

  expect_error(
    calculate_streak(list()),
    "data must be a data.frame"
  )
})
```

### Snapshot Testing
```r
test_that("summary output format is stable", {
  result <- summarize_habits(habit_example)
  expect_snapshot(result)
})
```

## Questions?

When stuck:
1. Check the error message carefully
2. Run `devtools::check()` to see all issues
3. Search for the error on [Stack Overflow](https://stackoverflow.com/questions/tagged/r)
4. Check the [R Packages book](https://r-pkgs.org/)
5. Ask on the [RStudio Community](https://community.rstudio.com/)
