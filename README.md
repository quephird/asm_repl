# asm_repl
A [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) for assembly.

Type some assembly instructions and immediatly see which registers were changed.

Currently only supports i386 and x86_64 on OS X.

Special instructions
==
This fork includes a very minor addition as well as different instructions for building on OS X Mojave, since it introduced a few monkey wrenches. Not only will this project build with XCode 10.x but you will need to go through some extra steps to be able to properly codesign the executable.

* First get a hold of an older version of XCode, 9.4.1 exactly, which you can get from https://developer.apple.com/download/more/.

* Once you've installed it, you'll need to switch your environment to use it and its own command line tools:

```
xcode-select -s /Applications/Xcode\ 9.4.1.app/Contents/Developer
```

* Next, install the command line tools:

```
xcode-select --install
```

* Install radare as documented here: https://github.com/radareorg/radare2

* Run `make`. (This will take a while.)

* See that you have a new executable in the project root, `asm_repl`.

* If you try to run it, you'll see the following message after being challenged for the System password:

```
task_for_pid() failed!
Either codesign asm_repl or run as root.
Failed to read
```

* To get around this you'll need to create a new certificate; follow the directions [here](https://gcc.gnu.org/onlinedocs/gnat_ugn/Codesigning-the-Debugger.html,) on how to do that.

* The original directions below will tell you to simply use the `codesign` utility next with the new certificate, but unfortunately that doesn't quite work on OS X Mojave. You'll need to also create an XML file that declares permissions that need to be explicitly listed and granted to the binary to get around the requirement to run as root. It should look like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.cs.debugger</key>
    <true/>
</dict>
</plist>
```

You can read more about this oddity here: https://lapcatsoftware.com/articles/debugging-mojave.html.

* Now you can codesign the binary with the following:

```
codesign -f -s  "task_for_pid" --entitlements asm_repl.xml ./asm_repl
```

* Finally, you should be able to run the executable without being bothered with passwords again except the first time.

Screenshot
==
[![Screenshot x86_64](http://i.imgur.com/Eb8Bz15.png)](http://i.imgur.com/Eb8Bz15.png)

Also see [https://asciinema.org/a/19605](https://asciinema.org/a/19605).

Running
==

* Install [radare2](https://github.com/radare/radare2).
* `make`
* `./asm_repl` (`make run32` or `make run64` to choose a specific architecture)

You need to codesign `asm_repl` binary or run it as root as we have to access the process we're running the assembly code in. You can codesign the binary so it can use `task_for_pid` without root by creating a certificate named `task_for_pid` using the guide [here](https://gcc.gnu.org/onlinedocs/gnat_ugn/Codesigning-the-Debugger.html) and then running `make`.

Commands
==

```
Valid input:
  Help:
    ?      - show this help
    ?[cmd] - show help for a command

  Commands:
    .set      - change value of register
    .read     - read from memory
    .write    - write hex to memory
    .writestr - write string to memory
    .alloc    - allocate memory
    .regs     - show the contents of the registers
    .show     - toggle shown register types

Any other input will be interpreted as x86_64 assembly
```

`.set`
--

```
Usage: .set register value
Changes the value of a register

  register - register name (GPR, FPR or status)
  value    - hex if GPR or FPR, 0 or 1 if status
```

`.read`
--

```
Usage: .read address [len]
Displays a hexdump of memory starting at address

  address - an integer or a register name
  len     - the amount of bytes to read
```

`.write`
--

```
Usage: .write address hexpairs
Writes hexpairs to a destination address

  address  - an integer or a register name
  hexpairs - pairs of hexadecimal numbers
```

`.writestr`
--

```
Usage: .writestr address string
Writes an ascii string to a destination address

  address - an integer or a register name
  string  - an ascii string
```

`.alloc`
--

```
Usage: .alloc len
Allocates some memory and returns the address

  len - the amount of bytes to allocate
```

`.regs`
--

```
Usage: .regs
Displays the values of the registers currently toggled on
```

`.show`
--

```
Usage: .show [gpr|status|fpr_hex|fpr_double]
Toggles which types of registers are shown

  gpr        - General purpose registers (rax, rsp, rip, ...)
  status     - Status registers (CF, ZF, ...)
  fpr_hex    - Floating point registers shown in hex (xmm0, xmm1, ...)
  fpr_double - Floating point registers shown as doubles
```

Todo
==

* Use a library (libr?) for assembling instead of reading the output of running `rasm2`.
* Support more architectures (arm).
* Support more platforms (linux).
* Arithmetic for commands (`.read rip-0x10`).
* Variables to specific memory addresses (`.alloc 4` => `.write $alloc 12345678`).
