# axlog2

Macros for multi-level formatted logging.
The log macros, in descending order of level, are: error!, warn!, info!, debug!, and trace!.
If it is used in no_std environment, the users need to implement the `LogIf` to provide external functions such as console output.
To use in the std environment, please enable the std feature:

```rust
[dependencies]
axlog = { version = "0.1", features = ["std"] }
```

Cargo features:

+ std: Use in the std environment. If it is enabled, you can use console output without implementing the [LogIf] trait. This is disabled by default.
+ log-level-off: Disable all logging. If it is enabled, all log macros (e.g. info!) will be optimized out to a no-op in compilation time.
+ log-level-error: Set the maximum log level to error. Any macro with a level lower than error! (e.g, warn!, info!, …) will be optimized out to a no-op.
+ log-level-warn, log-level-info, log-level-debug, log-level-trace: Similar to log-level-error.

## Examples

```rust
#![no_std]
#![no_main]

#[macro_use]
extern crate axlog2;

use core::panic::PanicInfo;

#[no_mangle]
pub extern "Rust" fn runtime_main(_cpu_id: usize, _dtb_pa: usize) {
    axlog2::init("debug");
    info!("[rt_axlog2]: ...");
    info!("[rt_axlog2]: ok!");
    axhal::misc::terminate();
}

#[panic_handler]
pub fn panic(info: &PanicInfo) -> ! {
    arch_boot::panic(info)
}

```

## Macros

### `ax_print`

```rust
macro_rules! ax_print {
    ($($arg:tt)*) => { ... };
}
```

Prints to the console.
Equivalent to the ax_println! macro except that a newline is not printed at the end of the message.

### `ax_println`

```rust
macro_rules! ax_println {
    () => { ... };
    ($($arg:tt)*) => { ... };
}
```

Prints to the console, with a newline.

### `debug`

```rust
macro_rules! debug {
    (target: $target:expr, $($arg:tt)+) => { ... };
    ($($arg:tt)+) => { ... };
}
```

Logs a message at the debug level.
**Examples**

```rust
use log::debug;

let pos = Position { x: 3.234, y: -1.223 };

debug!("New position: x: {}, y: {}", pos.x, pos.y);
debug!(target: "app_events", "New position: x: {}, y: {}", pos.x, pos.y);
```

### `error`

```rust
macro_rules! error {
    (target: $target:expr, $($arg:tt)+) => { ... };
    ($($arg:tt)+) => { ... };
}
```

Logs a message at the error level.
**Examples**

```rust
use log::error;

let (err_info, port) = ("No connection", 22);

error!("Error: {err_info} on port {port}");
error!(target: "app_events", "App Error: {err_info}, Port: {port}");
```

### `info`

```rust
macro_rules! info {
    (target: $target:expr, $($arg:tt)+) => { ... };
    ($($arg:tt)+) => { ... };
}
```

Logs a message at the info level.
**Examples**

```rust
use log::info;

let conn_info = Connection { port: 40, speed: 3.20 };

info!("Connected to port {} at {} Mb/s", conn_info.port, conn_info.speed);
info!(target: "connection_events", "Successful connection, port: {}, speed: {}",
      conn_info.port, conn_info.speed);
```

### `trace`

```rust
macro_rules! trace {
    (target: $target:expr, $($arg:tt)+) => { ... };
    ($($arg:tt)+) => { ... };
}
```

Logs a message at the trace level.
**Examples**

```rust
use log::trace;

let pos = Position { x: 3.234, y: -1.223 };

trace!("Position is: x: {}, y: {}", pos.x, pos.y);
trace!(target: "app_events", "x is {} and y is {}",
       if pos.x >= 0.0 { "positive" } else { "negative" },
       if pos.y >= 0.0 { "positive" } else { "negative" });
```

### `warn`

```rust
macro_rules! warn {
    (target: $target:expr, $($arg:tt)+) => { ... };
    ($($arg:tt)+) => { ... };
}
```

Logs a message at the warn level.
**Examples**

```rust
use log::warn;

let warn_description = "Invalid Input";

warn!("Warning! {warn_description}!");
warn!(target: "input_events", "App received warning: {warn_description}");
```

## Functions

### `init`

```rust
pub fn init(level: &str)
```

Initializes the logger.
This function should be called before any log macros are used, otherwise nothing will be printed.

### `print_fmt`

```rust
pub fn print_fmt(args: Arguments<'_>) -> Result
```

Prints the formatted string to the console.

### `set_max_level`

```rust
pub fn set_max_level(level: &str)
```

Set the maximum log level.
Unlike the features such as `log-level-error`, setting the logging level in this way incurs runtime overhead. In addition, this function is no effect when those features are enabled.
level should be one of `off`, `error`, `warn`, `info`, `debug`, `trace`.
