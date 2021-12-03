# 模块

Rust 中一个文件 即一个模块，也可以通过  `mod` 来创建模块。多个文件放到同一个目录下，也可以成为一个模块。

模块相关有三个概念：

1. `mod`是 Rust 代码的“骨架”。
2. `use` 则是用来决定使用或导出哪个模块中的具体的类型或方法。
3. `Path`，则是一个命名系统，类似于命名空间。

---
<!-- toc -->
---

## P.MOD.01    合理控制对外接口和模块之间的可见性

### 【描述】

Rust提供强大的模块（module）系统，并且可以管理这些模块之间的可见性（公有（public）或私有（private））。

1、对于提供给其他crate使用的对外函数、结构体、trait等类型需要严格控制对外pub的范围，避免将内部成员对外提供。

2、对于crate内部，mod之间可见的类型，需要添加上`pub(crate) `。

3、对于mod内部私有的类型，不要添加`pub(crate) `或者`pub`。

【正例】

```rust
// lib.rs
pub mod sha512;
pub use sha512::Sha512;

// sha512.rs
pub struct Sha512 {
    inner: Sha512Inner, // inner作为内部结构体，不添加pub描述
}
```

---

## G.MOD.01   使用导入模块中的类型或函数，在某些情况下需要带模块名前缀

### 【级别：建议】

建议按此规范执行

### 【Lint 检测】

| lint name | Clippy 可检测 | Rustc 可检测 | Lint Group | 是否可定制 |
| --------- | ------------- | ------------ | ---------- | ---------- |
| _         | no            | no           | _          | yes        |

### 【描述】

对于标准库中，很多人都熟知的类型 ，比如 `Arc`/ `Rc`/ `Cell`/ `HashMap` 等 ， 可以导入它们直接使用。

但是对于可能引起困惑的函数，比如 `std::ptr::replace` 和 `std::mem::replace` ，在使用它们的时候，就必须得带上模块前缀。

使用一些第三方库中定义的类型或函数，也建议带上crate或模块前缀。如果太长的话，可以考虑使用 `as`  或  `type` 来定义别名。

以上考虑都是为了增强代码的可读性、可维护性。

【正例】

```rust
use std::sync::Arc;
let foo = Arc::new(vec![1.0, 2.0, 3.0]); // 直接使用 Arc
let a = foo.clone();

// 需要带上 ptr 前缀
use std::ptr;
let mut rust = vec!['b', 'u', 's', 't'];
// `mem::replace` would have the same effect without requiring the unsafe
// block.
let b = unsafe {
    ptr::replace(&mut rust[0], 'r')
};
```



## G.MOD.02   如果是作为库供别人使用，在 `lib.rs`中重新导出对外类型、函数和 trait 等

### 【级别：规则】

按此规范执行

### 【Lint 检测】

| lint name | Clippy 可检测 | Rustc 可检测 | Lint Group | 是否可定制 |
| --------- | ------------- | ------------ | ---------- | ---------- |
| _         | no            | no           | _          | yes        |

### 【描述】

这样使用方在使用的时候，就不需要`use crate::mod::mod::struct`，可以直接使用`use crate::struct`，好处是使用方`use`的时候会比较方便和直观。

【正例】

```rust
// From syn crate
pub use crate::data::{
    Field, Fields, FieldsNamed, FieldsUnnamed, Variant, VisCrate, VisPublic, VisRestricted,
    Visibility,
};
```



## G.MOD.03   导入模块不要随便使用 通配符`*` 

### 【级别：建议】

建议按此规范执行。

### 【Lint 检测】

| lint name                                                    | Clippy 可检测 | Rustc 可检测 | Lint Group | level |
| ------------------------------------------------------------ | ------------- | ------------ | ---------- | ----- |
| [wildcard_imports](https://rust-lang.github.io/rust-clippy/master/#wildcard_imports) | yes           | no           | pedantic   | allow |

该 lint 可以通过 clippy 配置项 `warn-on-all-wildcard-imports = false` 来配置，用于是否禁用 `prelude`/ `super` (测试模块中) 使用通配符导入， 默认是 `false`。

### 【描述】

使用通配符导入会污染命名空间，比如导入相同命名的函数或类型。

【正例】

```rust
use crate1::foo; // Imports a function named foo
foo(); // Calls crate1::foo
```

【反例】

```rust
use crate2::*; // Has a function named foo
foo(); // Calls crate1::foo
```

【例外】

```rust
use prelude::*;

#[test]
use super::*
```

## G.MOD.04    一个项目中应该避免使用不同的模块布局风格

### 【级别：建议】

建议按此规范执行。

### 【Lint 检测】

| lint name                                                    | Clippy 可检测 | Rustc 可检测 | Lint Group  | level |
| ------------------------------------------------------------ | ------------- | ------------ | ----------- | ----- |
| [self_named_module_files](https://rust-lang.github.io/rust-clippy/master/#self_named_module_files) | yes           | no           | restriction | allow |
| [mod_module_files](https://rust-lang.github.io/rust-clippy/master/#mod_module_files) | yes           | no           | restriction | allow |

### 【描述】

Rust 支持两种 模块布局，文件夹内使用 `mod.rs` 或者是使用跟文件夹同名的文件名，来组织模块。

但是项目里如果混合这两种模块布局，是比较让人困惑的，最好统一为同一种风格。

 上面两种  lint ，选择其中一种用于检查是否存在不同的模块布局。

【正例】

```rust
// 使用 `self_named_module_files`，允许下面模块布局
src/
  stuff/
    stuff_files.rs
    mod.rs
  lib.rs

// 使用 `mod_module_files`，允许下面模块布局
src/
  stuff/
    stuff_files.rs
  stuff.rs
  lib.rs
```

【反例】

```rust
// 使用 `self_named_module_files`，不允许下面模块布局
src/
  stuff/
    stuff_files.rs
  stuff.rs
  lib.rs

// 使用 `mod_module_files`，不允许下面模块布局

src/
  stuff/
    stuff_files.rs
    mod.rs
  lib.rs
```



## G.MOD.05    不要在私有模块中 设置其内部类型或函数方法 为 `pub(crate)`

### 【级别：建议】

建议按此规范执行。

### 【Lint 检测】

| lint name                                                    | Clippy 可检测 | Rustc 可检测 | Lint Group | level |
| ------------------------------------------------------------ | ------------- | ------------ | ---------- | ----- |
| [redundant_pub_crate](https://rust-lang.github.io/rust-clippy/master/#redundant_pub_crate) | yes           | no           | nursery    | allow |

注意：此 lint 为 nursery，意味着有 Bug。

### 【描述】

如果在私有模块中设置  `pub(crate)` 可能会让使用者产生误解。建议用 `pub` 代替。

【正例】

```rust
mod internal {
    pub fn internal_fn() { }
}
```

【反例】

```rust
mod internal {
    pub(crate) fn internal_fn() { }
}
```

