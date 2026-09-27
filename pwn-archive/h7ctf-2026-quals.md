## First wave contained 5 pwn challenge.. solved em.. here's how..

**1st one**:  [download](https://github.com/TheGreenOfficial/pwn-archive/blob/main/pwn-archive/binaries/vault.zip)

```Read-Readme
Target: Ubuntu 24.04, glibc 2.39 (2.39-0ubuntu8.9), x86-64.

Files:
  vault                    the challenge binary (also served live on your instance)
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

****
