Export a Rust Function to Rhai
=============================

{{#include ../links.md}}


Sometimes only a few ad hoc functions are required and it is simpler to register
individual functions instead of a full-blown [plugin module].


Macros
------

| Macro                   | Signature                                                          | Description                                                     |
| ----------------------- | ------------------------------------------------------------------ | --------------------------------------------------------------- |
| `#[export_fn]`          | apply to rust function defined in a Rust module                    | exports the function                                            |
| `register_exported_fn!` | `register_exported_fn!(&mut `_engine_`, "`_name_`", `_function_`)` | registers the function into an [`Engine`] under a specific name |
| `set_exported_fn!`      | `set_exported_fn!(&mut `_module_`, "`_name_`", `_function_`)`      | registers the function into a [`Module`] under a specific name  |


`#[export_fn]` and `register_exported_fn!`
-----------------------------------------

Apply `#[export_fn]` onto a function defined at _module level_ to convert it into a Rhai plugin function.

The function cannot be nested inside another function - it can only be defined directly under a module.

To register the plugin function, simply call `register_exported_fn!`.  The name of the function can be
any text string, so it is possible to register _overloaded_ functions as well as operators.

```rust
use rhai::plugins::*;       // import macros

#[export_fn]
fn increment(num: &mut i64) {
    *num += 1;
}

fn main() {
    let mut engine = Engine::new();

    // 'register_exported_fn!' registers the function as 'inc' with the Engine.
    register_exported_fn!(engine, "inc", increment);
}
```


Fallible Functions
------------------

To register [fallible functions] (i.e. functions that may return errors), apply the
`#[rhai_fn(return_raw)]` attribute on plugin functions that return `Result<Dynamic, Box<EvalAltResult>>`.

A syntax error is generated if the function with `#[rhai_fn(return_raw)]` does not
have the appropriate return type.

```rust
use rhai::plugins::*;       // import macros

#[export_fn]
#[rhai_fn(return_raw)]
pub fn double_and_divide(x: i64, y: i64) -> Result<Dynamic, Box<EvalAltResult>> {
    if y == 0 {
        Err("Division by zero!".into())
    } else {
        let result = (x * 2) / y;
        Ok(result.into())
    }
}

fn main() {
    let mut engine = Engine::new();

    // Overloads the operator '+' with the Engine.
    register_exported_fn!(engine, "+", double_and_divide);
}
```