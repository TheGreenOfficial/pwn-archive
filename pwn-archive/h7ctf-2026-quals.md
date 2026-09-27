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

**cont..**
