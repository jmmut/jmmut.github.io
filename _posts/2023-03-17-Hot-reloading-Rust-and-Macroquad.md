---
tags: programming
excerpt_separator: <!--more-->
---

# Hot-reloading Rust and Macroquad

I came across [this blog post](https://fasterthanli.me/articles/so-you-want-to-live-reload-rust)
thanks to [this other post](https://johnaustin.io/articles/2022/hot-reloading-rust). The first one
is a great walk-through to create dynamic libraries in C and Rust, loading them with dlopen,
and reloading them in a loop.

Later I found [this article about hot-lib-reloader](https://robert.kra.hn/posts/hot-reloading-rust/),
that presents a library to do hot reloading and mentions that Macroquad is tricky to hot-reload
because it has global state. It briefly mentions that it can be reloaded if the state is kept
in the main binary and the reloaded code uses a decorated Macroquad handle.

With all those ideas, I did a proof of concept for a hot-reloading Macroquad program, and then
added hot-reloading to the Macroquad game I'm writing,
[Bioengineer](https://jmmut.itch.io/bioengineer). The idea is not new, and I'm kind of embarrassed I
didn't try this before.

<!--more-->

### The basic hot-reloading idea

See this simple rust library:
```
# hot-reload-rust-lib/Cargo.toml
[package]
name = "hot-reload-rust-lib"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]
```

```rust
// hot-reload-rust-lib/src/lib.rs
#[no_mangle]
pub extern "C" fn multiply(n: i64) -> i64 {
    n * 10
}
```

It defines a function `multiply` that will be compiled in a C compatible way (`#[no_mangle]`,
 `extern "C"`) into a dynamic library (`crate-type = ["cdylib"]`) called `libhot_reload_rust_lib.so`.

Now see this other rust program that loads it at runtime. The important part is how `dynamic_load()`
is called inside a loop, and it calls `dlopen`, `dlsym` and `transmute` to get the `multiply`
function from inside the dynamic library, and later we call `dlclose` (also inside the loop). See [Faster Than Lime's article](https://fasterthanli.me/articles/so-you-want-to-live-reload-rust) for the step-by-step reasoning to get to this code:
```
# hot-reload-rust-bin/Cargo.toml
[package]
name = "hot-reload-rust-bin"
version = "0.1.0"
edition = "2021"
```
```rust
// hot-reload-rust-bin/src/main.rs
use std::env::current_dir;
use std::error::Error;
use std::io::BufRead;
use std::{ffi::c_void, ffi::CString, os::raw::c_char, os::raw::c_int};

type AnyError = Box<dyn Error>;

#[link(name = "dl")]
extern "C" {
    fn dlopen(path: *const c_char, flags: c_int) -> *const c_void;
    fn dlsym(handle: *const c_void, name: *const c_char) -> *const c_void;
    fn dlclose(handle: *const c_void);
}

// taken from dlfcn.h
pub const RTLD_LAZY: c_int = 0x00001;

const LIB_PATH: &'static str = "../hot-reload-rust-lib/target/debug/libhot_reload_rust_lib.so";
const FUNCTION_NAME: &'static str = "multiply";
type FunctionType = extern "C" fn(n: i64) -> i64;

fn main() -> Result<(), AnyError> {
    let mut line = String::new();
    let stdin = std::io::stdin();

    loop {
        let (multiply, lib_handle) = dynamic_load(LIB_PATH, FUNCTION_NAME)?;

        let input = 2;
        let result = multiply(input);
        println!("input was {}, result was {}", input, result);
        
        unload(lib_handle);
        
        println!("-----------------------------\nPress Enter to go again, Ctrl-C to exit...");
        stdin.lock().read_line(&mut line)?;
    }
}

fn dynamic_load(
    lib_path: &str,
    function_name: &str,
) -> Result<(FunctionType, *const c_void), AnyError> {
    let lib_path_c = CString::new(lib_path).unwrap();
    let lib = unsafe { dlopen(lib_path_c.as_ptr(), RTLD_LAZY) };
    if lib.is_null() {
        let dir = current_dir()?;
        let dir = dir.display();
        return Err(format!("could not open library at {} from {}", lib_path, dir).into());
    }

    let function_name_c = CString::new(function_name).unwrap();
    let function_raw = unsafe { dlsym(lib, function_name_c.as_ptr()) };
    if function_raw.is_null() {
        return Err(format!("could not load function {}", function_name).into());
    }
    let function: FunctionType = unsafe { std::mem::transmute(function_raw) };
    Ok((function, lib))
}

fn unload(lib: *const c_void) {
    unsafe {
        dlclose(lib);
    }
}
```

if you run that program from the `hot-reload-rust-bin` folder, you can edit 
 `hot-reload-rust-lib/src/lib.rs` and recompile it, press enter on the program and see the changes,
even though the program is still the same process as before.

<div id="simple-reload"></div>
<script src="/js/asciinema-player.min.js"></script>
<script>
    AsciinemaPlayer.create('/assets/hot-reloading/simple.cast',
                           document.getElementById('simple-reload'));
</script>

---
---

### Hot-reloading Macroquad

Now, reloading Macroquad has the extra difficulty that it can not be used directly in the library
part, because that will compile some global state that is not initialized, as initialization will
happen on the global copy from the main binary.

For the lib part, the Cargo.toml is the same (note it doesn't need the Macroquad dependency), and
the code can look like this for our simple example:
```
# hot-reload-macroquad-lib/Cargo.toml
[package]
name = "hot-reload-macroquad-lib"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]
```

```rust
// hot-reload-macroquad-lib/src/lib.rs
pub trait Drawer {
    fn draw_text(&self, text: &str, x: f32, y: f32, font_size: f32);
}

pub struct Screen {
    pub drawer: Box<dyn Drawer>,
}

pub type DrawFrameFunction = extern "C" fn(&Screen);

#[no_mangle]
pub extern "C" fn hot_reload_draw_frame(screen :&Screen) {
    screen.drawer.draw_text("text that will change", 20.0, 60.0, 30.0);
}
```

Below there's the main program. The main difference is we are passing a `Screen` to the library,
and the Macroquad global state is kept in the main binary, configured properly by the main entry
point.

```
# hot-reload-macroquad-bin/Cargo.toml
[package]
name = "hot-reload-macroquad-bin"
version = "0.1.0"
edition = "2021"

[dependencies]
macroquad = "=0.3.16"
```

```rust
use macroquad::prelude::*;
use std::error::Error;
use std::ptr::null;
use std::{ffi::c_void, ffi::CString, os::raw::c_char, os::raw::c_int};

type AnyError = Box<dyn Error>;

// taken from dlfcn.h
pub const RTLD_LAZY: c_int = 0x00001;

#[link(name = "dl")]
extern "C" {
    fn dlopen(path: *const c_char, flags: c_int) -> *const c_void;
    fn dlsym(handle: *const c_void, name: *const c_char) -> *const c_void;
    fn dlclose(handle: *const c_void);
}

const LIB_PATH: &'static str =
    "../hot-reload-macroquad-lib/target/debug/libhot_reload_macroquad_lib.so";
const FUNCTION_NAME: &'static str = "hot_reload_draw_frame";
pub type DrawFrameFunction = extern "C" fn(&Screen);

pub struct Screen {
    drawer: Box<dyn Drawer>,
}

pub trait Drawer {
    fn draw_text(&self, text: &str, x: f32, y: f32, font_size: f32);
}

pub struct DrawerImpl;

impl Drawer for DrawerImpl {
    fn draw_text(&self, text: &str, x: f32, y: f32, font_size: f32) {
        macroquad::prelude::draw_text(text, x, y, font_size, BLACK);
    }
}

#[macroquad::main("BasicShapes")]
async fn main() -> Result<(), AnyError> {
    let drawer: Box<dyn Drawer> = Box::new(DrawerImpl {});
    let screen = Screen { drawer };
    let (mut draw_frame, mut lib_handle) = load()?;
    while !is_key_pressed(KeyCode::Escape) {
        clear_background(LIGHTGRAY);

        if is_key_down(KeyCode::R) {
            draw_text(
                "About to reload when you release",
                20.0,
                20.0,
                30.0,
                DARKGRAY,
            );
        }
        if is_key_released(KeyCode::R) {
            (draw_frame, lib_handle) = reload(lib_handle)?;
        }
        draw_line(60.0, 40.0, 120.0, 200.0, 15.0, BLUE);
        draw_frame(&screen);
        next_frame().await
    }
    unload(lib_handle);
    Ok(())
}

fn reload(lib: *const c_void) -> Result<(DrawFrameFunction, *const c_void), AnyError> {
    unload(lib);
    load()
}

fn load() -> Result<(DrawFrameFunction, *const c_void), AnyError> {
    let lib_name_c = CString::new(LIB_PATH).unwrap();
    let lib = unsafe { dlopen(lib_name_c.as_ptr(), RTLD_LAZY) };
    if lib.is_null() {
        let dir = std::env::current_dir()?;
        let dir = dir.display();
        return Err(format!("could not open library at {} from {}", LIB_PATH, dir).into());
    }

    let function_name = CString::new(FUNCTION_NAME).unwrap();
    let function = unsafe { dlsym(lib, function_name.as_ptr()) };
    if function.is_null() {
        return Err(format!("could not load function {}", FUNCTION_NAME).into());
    }
    let transmuted_function: DrawFrameFunction = unsafe { std::mem::transmute(function) };
    Ok((transmuted_function, lib))
}

fn unload(lib: *const c_void) {
    if !lib.is_null() {
        unsafe {
            dlclose(lib);
        }
    }
}
```

### Hot-reloading Bioengineer

I'm not going to post the code here because it's a bit more complex, but the idea is the same.
Faster Than Lime also added a cool feature about watching the folder and reloading when the lib is
recompiled, and I totally stole the idea.

You can [read the code](https://github.com/jmmut/bioengineer/blob/d02bf929435a7552fa31d664d9c8a3350a7e1a4e/src/bin/hot_reload_bioengineer.rs)
if you are really interested in how it's done in a more non-trivial game. I decorated the calls for
drawing textures and UI too.

