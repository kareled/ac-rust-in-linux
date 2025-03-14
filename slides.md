---
marp: true
footer: Rust in Linux kernel
---
<!-- Barvou nadpisů první a druhé úrovně je _tmavá zelená_ z vizuálního stylu
používaného do r. 2025. Nová tmavá zelená je na většíně projektorů špatně vidět.
Často málo kontrastuje s bílou -->
<style>
img[alt~="logo"] {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 210px;
}

h1 {
    color: #009645
}

h2 {
    color: #009645
}

pre {
    background: #f8f8f8
}

img[alt~="title-image"] {
  position: absolute;
  top: 500px;
  right: 70px;
  width: 180px;
}
</style>

![logo](img/edhouse_logo.png)

# Rust in Linux kernel

What. Where. And why at all!

---

## Linux, what is it?

![logo](img/edhouse_logo.png)
![bg right:33% width:300px](img/LinuxCon_Europe_Linus_Torvalds_03_(cropped).jpg)

- kernel. Only kernel!
- first version 1991 (Linus Torvalds)
- open source/free software project with GPLv2 license
- last version 6.14-rc6, March 9, 2025
- 25 millions lines of C code
---

## Why Rust for kernel?

because of security related issues. Let's see some statistics (2020)

- 60-70% of security bugs in Ubuntu kernel are memory related (unsafe memory handling)
- 65% of Android and Chrome in Android are use-after-free, double-free and heap buffer overflows
- 71% of all macOS vulnerabilities are caused by memory unsafety
- ~70% of M$ vulnerabilities (CVEs) are memory safety issues

source: https://alexgaynor.net/2020/may/27/science-on-memory-unsafety-and-security/

![logo](img/edhouse_logo.png)


---
## Rust in Linux history

![logo](img/edhouse_logo.png)

- started as a hobby project in 2013
  https://github.com/tsgates/rust.ko  
- serious talks started during 2020 on Linux Plumbers Conference
- first commit of real Rust code happened during 2021
- toolchain issue: Linux kernel likes GNU C, while Rust is based on LLVM
  - solved by hacking kernel to be Clang/LLVM friendly and compilable.
  ```
  $ make LLVM=1
  ```

---
## Rust in Linux as of 2025

![logo](img/edhouse_logo.png)

- 11k lines of code!
- why so little?
  - no drivers (except reference)
  - no internals
  - few examples
  - *just* C to Rust manual *bindings* and *abstractions*!
  - bindgen generated bindings are 180k lines of Rust code!

---
## Rust in Linux development policy

![logo](img/edhouse_logo.png)

```

                                                rust/bindings/
                                               (rust/helpers/)

                                                   include/ -----+ <-+
                                                                 |   |
  drivers/              rust/kernel/              +----------+ <-+   |
    fs/                                           | bindgen  |       |
   .../            +-------------------+          +----------+ --+   |
                   |    Abstractions   |                         |   |
+---------+        | +------+ +------+ |          +----------+   |   |
| my_foo  | -----> | | foo  | | bar  | | -------> | Bindings | <-+   |
| driver  |  Safe  | | sub- | | sub- | |  Unsafe  |          |       |
+---------+        | |system| |system| |          | bindings | <-----+
     |             | +------+ +------+ |          |  crate   |       |
     |             |   kernel crate    |          +----------+       |
     |             +-------------------+                             |
     |                                                               |
     +------------------# FORBIDDEN #--------------------------------+
```


---


![logo](img/edhouse_logo.png)

## Example

```rust
pub struct Page {
    page: NonNull<bindings::page>,
}
[...]
impl Page {
  [...]
  pub fn alloc_page(flags: Flags) -> Result<Self, AllocError> {
    // SAFETY: Depending on the value of `gfp_flags`, this call may sleep. Other than that, it
    // is always safe to call this method.
    let page = unsafe { bindings::alloc_pages(flags.as_raw(), 0) };
    let page = NonNull::new(page).ok_or(AllocError)?;
    // INVARIANT: We just successfully allocated a page, so we now have ownership of the newly
    // allocated page. We transfer that ownership to the new `Page` object.
    Ok(Self { page })
  }
}
```

---

## What do we need for Hello World like example?

![logo](img/edhouse_logo.png)

- compile own kernel! :-)
- let's make it as easy as possible
  - use distribution with up-to-date tools (Debian Testing to the rescue!)
  - install all required tools from distribution repository
  ```
  $ sudo apt install rustc rust-src bindgen rustfmt rust-clippy
  ```    
  - obtain kernel source code, preferably from github.com mirror
  ```
  $ git clone https://github.com/torvalds/linux.git
  ```
  - make coffee, it takes some time, ... to git load 6GB of data


---

![logo](img/edhouse_logo.png)

## Demo Time!

- verify toolchain is there: `# make LLVM=1 rustavailable`
- configuration: `# make LLVM=1 menuconfig`
- compilation: `# make LLVM=1 -j14`
- installation: `# make LLVM=1 modules_install install`

---

![logo](img/edhouse_logo.png)

## Demo Time!

- I'm cheating! Due to time required for compilation (40 minutes!)
  I've already compiled both kernel and modules and also installed it

- hence I'll recompile/reinstall only rust modules here
```
  # make LLVM=1 M=samples/rust
```
- and install
```
  # make LLVM=1 M=samples/rust modules_install
```
---

![logo](img/edhouse_logo.png)

## What API is available?

- no_std only, hence only core and alloc crates are available
  e.g KVec as Vec implementation, KBox as Box implementation etc.
- kernel provides its 'kernel' crate
  https://rust.docs.kernel.org/kernel/

---

## In kernel tree kind of reference drivers

![logo](img/edhouse_logo.png)
- AMCC QT2025 PHY driver -- basically part (physical) of some 10 GigE NICs
- ASIX PHY Driver
- DRM Panic QR code generator -- displays panic info in a form of QR
- Null Block Driver -- minimalistic block device driver

---

## Out of kernel tree drivers

![logo](img/edhouse_logo.png)

- Android Binder driver
- Apple AGX GPU driver -- basically Linux is not able to use GPU on modern Mx based macs unless Rust is compiled in.
- Nova GPU driver -- free implementation of GPU driver for modern Nvidia cards
- NVMe driver -- Samsung's work
- PuzzleFS filesystem driver

---

## Questions?

![logo](img/edhouse_logo.png)
