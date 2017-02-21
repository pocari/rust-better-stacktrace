# macosでcargo runでのスタックトレースの行番号確認

## 検証環境

- macos

```
% cargo --version
cargo-0.18.0-nightly (fdfdb5f 2017-02-16)
```

linux

```
% docker run --rm -v $(pwd):/source jimmycuadra/rust cargo --version
cargo 0.16.0-nightly (6e0c18c 2017-01-27)
```

`src/main.rs` は6行目で`panic`するプログラムだが、これをmacのcargoで動かすと

```
% RUST_BACKTRACE=1 cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/rust-better-stacktrace`
xxx
yyy
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src/libcore/option.rs:323
stack backtrace:
   1:        0x10a9bad59 - std::sys::imp::backtrace::tracing::imp::write::h9559bd7cb15d72ad
   2:        0x10a9bc22e - std::panicking::default_hook::{{closure}}::h4b3f2b69c9ce844d
   3:        0x10a9bbedb - std::panicking::default_hook::h61d415f2381a7336
   4:        0x10a9bc5e7 - std::panicking::rust_panic_with_hook::h8e6300d8e8aca457
   5:        0x10a9bc484 - std::panicking::begin_panic::h08622fbe5a379aac
   6:        0x10a9bc3f2 - std::panicking::begin_panic_fmt::ha00d3aa9db91f578
   7:        0x10a9bc357 - rust_begin_unwind
   8:        0x10a9dca20 - core::panicking::panic_fmt::h58d018e87f211baf
   9:        0x10a9dc924 - core::panicking::panic::h4e2dc358c3b23d48
  10:        0x10a9b4634 - <core::option::Option<T>>::unwrap::hd19ecad304e85b2f
  11:        0x10a9b4808 - rust_better_stacktrace::main::hd6e610e90450cf55
  12:        0x10a9be05a - __rust_maybe_catch_panic
  13:        0x10a9bc8a6 - std::rt::lang_start::he28fbf70e98a2b18
  14:        0x10a9b48f9 - main
```

というように関数名と何かのハッシュ値だけしかスタックトレースに表示されない

一方 linuxで実行すると

```
root@06a3933938e9:/source# RUST_BACKTRACE=1 cargo run
    Finished debug [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/rust-better-stacktrace`
xxx
yyy
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libcore/option.rs:323
stack backtrace:
   1:     0x558b2e4df1aa - std::sys::imp::backtrace::tracing::imp::write::h3188f035833a2635
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libstd/sys/unix/backtrace/tracing/gcc_s.rs:42
   2:     0x558b2e4e10df - std::panicking::default_hook::{{closure}}::h6385b6959a2dd25b
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libstd/panicking.rs:349
   3:     0x558b2e4e0cde - std::panicking::default_hook::he4f3b61755d7fa95
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libstd/panicking.rs:365
   4:     0x558b2e4e14c7 - std::panicking::rust_panic_with_hook::hf00b8130f73095ec
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libstd/panicking.rs:553
   5:     0x558b2e4e1304 - std::panicking::begin_panic::h6227f62cb2cdaeb4
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libstd/panicking.rs:515
   6:     0x558b2e4e1279 - std::panicking::begin_panic_fmt::h173eadd80ae64bec
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libstd/panicking.rs:499
   7:     0x558b2e4e1207 - rust_begin_unwind
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libstd/panicking.rs:475
   8:     0x558b2e506ffd - core::panicking::panic_fmt::h3b2d1e30090844ff
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libcore/panicking.rs:69
   9:     0x558b2e506f34 - core::panicking::panic::h2596388ccef1871c
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libcore/panicking.rs:49
  10:     0x558b2e4d89f4 - <core::option::Option<T>>::unwrap::hdacdd6cb35717f45
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libcore/macros.rs:21
  11:     0x558b2e4d8beb - rust_better_stacktrace::main::hfdf401c58b6d709f
                        at /source/src/main.rs:6
  12:     0x558b2e4e8dea - __rust_maybe_catch_panic
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libpanic_unwind/lib.rs:98
  13:     0x558b2e4e19c6 - std::rt::lang_start::h65647f6e36cffdae
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libstd/panicking.rs:434
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libstd/panic.rs:351
                        at /buildslave/rust-buildbot/slave/stable-dist-rustc-linux/build/src/libstd/rt.rs:57
  14:     0x558b2e4d8d02 - main
  15:     0x7f493fba8b44 - __libc_start_main
  16:     0x558b2e4d8878 - <unknown>
  17:                0x0 - <unknown>
```

というように ファイル名と行番号まで表示され、

```
11:     0x558b2e4d8beb - rust_better_stacktrace::main::hfdf401c58b6d709f
                      at /source/src/main.rs:6
```

main.rsの6行目が直接の原因とわかる。


## とりあえずの対策

rustのissueでも上がっていてその中で`lldb`を使ってスタックトレースを見るほうが紹介されている。

https://github.com/rust-lang/rust/issues/24346

### 方法

```
# lldbで対象のプログラムを起動
$ lldb target/debug/rust-better-stacktrace

# lldb内で rust_begin_unwind に対してブレイクポイントを貼る
(lldb) b rust_begin_unwind

# で実行
(lldb) run

# panicすると上記のブレイクポイントで止まるのでそのタイミングでバックトレース表示
(lldb) bt
```

すると下記のようなバックトレースが表示され、 `main.rs` の6行目で落ちたことが分かる

```
* thread #1: tid = 0x78788, 0x0000000100009324 rust-better-stacktrace`std::panicking::rust_begin_panic + 4 at panicking.rs:464, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x0000000100009324 rust-better-stacktrace`std::panicking::rust_begin_panic + 4 at panicking.rs:464 [opt]
    frame #1: 0x0000000100029a21 rust-better-stacktrace`core::panicking::panic_fmt + 129 at panicking.rs:69 [opt]
    frame #2: 0x0000000100029925 rust-better-stacktrace`core::panicking::panic + 101 at panicking.rs:49 [opt]
    frame #3: 0x0000000100001635 rust-better-stacktrace`core::option::{{impl}}::unwrap<i32>(self=Option<i32> @ 0x00007fff5fbff020) + 85 at macros.rs:21
    frame #4: 0x0000000100001809 rust-better-stacktrace`rust_better_stacktrace::main + 169 at main.rs:6
    frame #5: 0x000000010000b05b rust-better-stacktrace`panic_unwind::__rust_maybe_catch_panic + 27 at lib.rs:98 [opt]
    frame #6: 0x00000001000098a7 rust-better-stacktrace`std::rt::lang_start [inlined] std::panicking::try<(),fn()> + 44 at panicking.rs:429 [opt]
    frame #7: 0x000000010000987b rust-better-stacktrace`std::rt::lang_start [inlined] std::panic::catch_unwind<fn(),()> at panic.rs:361 [opt]
    frame #8: 0x000000010000987b rust-better-stacktrace`std::rt::lang_start + 347 at rt.rs:57 [opt]
    frame #9: 0x00000001000018fa rust-better-stacktrace`main + 42
    frame #10: 0x00007fff8b64e5ad libdyld.dylib`start + 1
```
