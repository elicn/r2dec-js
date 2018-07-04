[![Build Status](https://travis-ci.org/wargio/r2dec-js.svg?branch=master)](https://travis-ci.org/wargio/r2dec-js)

r2dec
=====

Converts asm to pseudo-C code.

# Software Requirements

Requires radare2 version 2.6.9 or newer.

# Install

Follow the following steps to install r2dec via r2pm

    r2pm init
    r2pm install r2dec

done

# Usage

* open with radare2 your file
* analize the function you want to disassemble (`af`)
* run the plugin via `pdd`
* done.

# Arguments

```
[0x00000c60]> pdd?
Usage: pdd [args] - core plugin for r2dec
 pdd   - decompile current function
 pdd?  - show this help
 pdda  - decompile current function with side assembly
 pddu  - install/upgrade r2dec via r2pm
 pddi  - generates the issue data
[0x00000c60]> pdd --help
r2dec [options]
       --help     | this help message
       --colors   | enables syntax colors
       --assembly | shows pseudo next to the assembly
       --offset   | shows pseudo next to the offset
       --casts    | shows all casts in the pseudo code
       --issue    | generates the json used for the test suite
       --debug    | do not catch exceptions
       --html     | outputs html data instead of text
       --xrefs    | shows all xrefs in the pseudo code
```

# Radare2 Evaluable vars

You can use these in your `.radare2rc` file.

```
r2dec.casts         | if false, hides all casts in the pseudo code.
r2dec.asm           | if true, shows pseudo next to the assembly.
r2dec.offset        | if true, shows pseudo next to the offset.
r2dec.xrefs         | if true, shows all xrefs in the pseudo code.
r2dec.theme         | defines the color theme to be used on r2dec.
e scr.html          | outputs html data instead of text.
e scr.color         | enables syntax colors.
```

# Report an Issue

* open with radare2 your file
* analize the function you want to disassemble (`af`)
* give the data to the plugin via `pddi` or `pdd --issue`
* insert the JSON returned by the previous command into the issue (you can also upload the output)
* done.

# Supported Arch

    arm
    avr
    mips
    ppc
    sparc
    v850
    wasm
    x86 (intel)

# Developing on r2dec

[Read DEVELOPERS.md](https://github.com/wargio/r2dec-js/blob/master/DEVELOPERS.md)

## Example

This example shows a possible dump of the plugin.

### Source Code

```c
#include <stdio.h>

int main(int argc, char const *argv[]) {
    int var = 0;
    while(var < 0x90) {
        if(var < 0x10) {
            var += 0x50;
        }
        var += 0x10;
    }
    return 0;
}
```

### radare2 view


```
            ;-- main:
╭ (fcn) main 50
│   main ();
│           ; var int local_20h @ rbp-0x20
│           ; var int local_14h @ rbp-0x14
│           ; var int local_4h @ rbp-0x4
│              ; DATA XREF from 0x0000050d (entry0)
│           0x000005fa      55             push rbp
│           0x000005fb      4889e5         mov rbp, rsp
│           0x000005fe      897dec         mov dword [local_14h], edi
│           0x00000601      488975e0       mov qword [local_20h], rsi
│           0x00000605      c745fc000000.  mov dword [local_4h], 0
│       ╭─< 0x0000060c      eb0e           jmp 0x61c
│       │      ; JMP XREF from 0x00000623 (main)
│      ╭──> 0x0000060e      837dfc0f       cmp dword [local_4h], 0xf   ; [0xf:4]=0x3e000300
│     ╭───< 0x00000612      7f04           jg 0x618
│     │⁝│   0x00000614      8345fc50       add dword [local_4h], 0x50  ; 'P'
│     │⁝│      ; JMP XREF from 0x00000612 (main)
│     ╰───> 0x00000618      8345fc10       add dword [local_4h], 0x10
│      ⁝│      ; JMP XREF from 0x0000060c (main)
│      ⁝╰─> 0x0000061c      817dfc8f0000.  cmp dword [local_4h], 0x8f  ; [0x8f:4]=0x23800
│      ╰──< 0x00000623      7ee9           jle 0x60e
│           0x00000625      b800000000     mov eax, 0
│           0x0000062a      5d             pop rbp
╰           0x0000062b      c3             ret
```

### r2dec pseudo-C code

```c
#include <stdint.h>

int32_t main () {
    *((int32_t*) local_14h) = edi;
    *((int64_t*) local_20h) = rsi;
    *((int32_t*) local_4h) = 0;
    while (*((int32_t*) local_4h) > 0x8f) {
        if (*((int32_t*) local_4h) <= 0xf) {
            *((int32_t*) local_4h) += 0x50;
        }
        *((int32_t*) local_4h) += 0x10;
    }
    eax = 0;
    return eax;
}
```
