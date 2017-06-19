# Google CTF 2017 : Exploit 150

**Category:** pwn |
**Name:** Inst Prof |
**Solves:** 79 / 1998 paticipants |
**Description:** Please help test our new compiler micro-service

___
## Write-up
Despite there wasn't a real attack vector InstProf was a very interesting challenge, because I made a lot of mistakes as always but after a lot of computer meditation I SOLVED it.

## Abstract
This solution doesn't involve ASLR leaking and libc is not used. This solution allocates two pages using code reuse, one page to stack pivot and the other page to execute a shellcode... text pointer dereferencing to bypass ASLR. [exploit.py](/exploit.py)

##  Step0: War of information

We were given a x86_64 linux binary, PIE and NX were active, the binary read reapeatedly 4 bytes from stdin and executed them as asm bytecode.
Below a description of the binary execution:
```
Read four bytes from stdin (read)
Allocate one page (mmap) .. one page = 4096 bytes
Write our four bytes on the allocated page (mov)
Set the page executable (mprotect)
Execute our four bytes 4096 times in a loop (call rbx)
Replay all again
```
We give input to the program and he evaluates them 4 bytes per 4 bytes, using this vector we have to take full control of the remote server. The funny thing is that the number of asm instructions fitting in only 4 bytes are kind of limited so we will have hack a bit.

## Step42: DEEEEEP dive

Let me describe how the magic happened for me.
First, I used unused registers **r13, r14, r15** to create persistence between all different successive evaluations of 4 bytes, these registers allowed us to load a text pointer (**saved_rip**)  from the stack **pop r14; push r14**, and then we dereference it to point to a useful code **"inc r14; ret" * offset_to_code_reuse**. Because the binary was packaged with helpful functions like **allocate_page** and **make_page_executable** we pretty much only needed to call them **call r14** to create two pages. Then we fill the two pages, the first with a shellcode and the other with one ROP gadget to ease the call of **make_page_executable**. Once the shellcode is executable we just need to **ret** in it. There is more to the story, for example all instructions are executed 4096 times but every inconvenience seen from the right angle is helpful. It's time for you to read some python [exploit.py](/exploit.py)

This is the instruction use to fill pages with the data we want:
![mov_r15](/mov_r15.PNG)

## Tips & Tools
This section regroups random valuable informations specific to pwning:

- NX: a page can't be executable and writable at the same time
- ALSR: some sections of the binary are randomized
- PIE: all sections of the binary are randomized
- stack pivot: create a fake stack frame to ease ROP chaining
- pointer dereferencing: use of registers to resolve runtime offset

Reverse Tools:
- GDB: main tool of a linux reverser
- pwntools: greatest python library
- IDA: most used/complete gui reversing tool
- Hopper: best pseudo code
- Binary Ninja: most sexy and future mainstream tool
- radare2: yoda tool
- [ARM NOW](https://github.com/nongiach/arm_now) : easiest/lightest ARM setup
- checksec, peda, plasma, afl-fuzz, frida, dynamorio, voltron, lldb, strace, ltrace, qemu, objdump, nm, hexdump, file, angr, manticore, z3, miasm: just check them out

#### PS: Google feel free to send me goodies :)
----
[@chaign\_c](https://twitter.com/chaign_c) from [HexpressoTeam](http://hexpresso.github.io/)
