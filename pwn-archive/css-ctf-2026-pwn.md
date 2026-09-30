**CSS-CTF-2026 pwn chall solution..**

[binary](https://github.com/TheGreenOfficial/pwn-archive/blob/main/pwn-archive/binaries/sector01_handout.zip)

*initial checks*

```bash
skid@TheGreen:~/ctfs/old/pwn/public$ ls
chall  Dockerfile  flag.txt
skid@TheGreen:~/ctfs/old/pwn/public$ file chall
chall: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=5dc6c7d1b386968f69614f59403d81e77b22be70, for GNU/Linux 3.2.0, stripped
skid@TheGreen:~/ctfs/old/pwn/public$ pwn checksec chall
[*] '/home/skid/ctfs/old/pwn/public/chall'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
skid@TheGreen:~/ctfs/old/pwn/public$
```

*decompiled view in ghidra*

```
====================================================================================================
FUNCTION: FUN_00401268
ADDRESS : 00401268
====================================================================================================
void FUN_00401268(int param_1,int param_2)
{
  FILE *__stream;
  long in_FS_OFFSET;
  char local_58 [72];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  if ((param_1 == -0x21524111) && (param_2 == -0x35014542)) {
    puts("[+] Access Granted! Here is your flag:");
    __stream = fopen("flag.txt","r");
    if (__stream == (FILE *)0x0) {
      puts("[-] flag.txt missing on server.");
                    /* WARNING: Subroutine does not return */
      exit(1);
    }
    fgets(local_58,0x40,__stream);
    puts(local_58);
    fclose(__stream);
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("[-] Authentication token mismatch.");
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
====================================================================================================
FUNCTION: FUN_00401348
ADDRESS : 00401348
====================================================================================================
void FUN_00401348(void)
{
  undefined1 local_28 [32];
  
  memset(local_28,0,32);
  printf("Tagging operator: ");
  read(0,local_28,33);
  return;
}
====================================================================================================
FUNCTION: FUN_0040139a
ADDRESS : 0040139a
====================================================================================================
void FUN_0040139a(void)
{
  undefined1 local_58 [80];
  
  memset(local_58,0,80);
  printf("[*] Report buffer allocated at: %p\n",local_58);
  printf("Enter report summary: ");
  read(0,local_58,80);
  FUN_00401348();
  puts("[*] Processing report...");
  return;
}
====================================================================================================
FUNCTION: FUN_00401419
ADDRESS : 00401419
====================================================================================================
undefined8 FUN_00401419(void)
{
  long lVar1;
  long in_FS_OFFSET;
  
  lVar1 = *(long *)(in_FS_OFFSET + 0x28);
  FUN_004011b6();
  puts("=== Sector 01: Maintenance Log Terminal ===");
  FUN_0040139a();
  puts("[*] Log finalized. Exiting.");
  if (lVar1 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

**
