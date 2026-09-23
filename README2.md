```

There's a lot to learn hhhhhhhh... list..

════════════════════════════════════════════════════════════════════
🧱 MEMORY CORRUPTION
════════════════════════════════════════════════════════════════════

Buffer Overflow → overwrite data beyond a buffer boundary
Stack Buffer Overflow → overwrite stack data / saved return addresses
Heap Buffer Overflow → overwrite adjacent heap objects / metadata
Global/BSS Overflow → overwrite global or BSS data
Heap Underflow → corrupt memory before a heap allocation
Off-by-One → overwrite exactly one byte past a boundary
Off-by-Null → overwrite a byte with NULL / 0x00
Out-of-Bounds Read → read memory outside intended bounds
Out-of-Bounds Write → write memory outside intended bounds
Arbitrary Read → read attacker-chosen memory
Arbitrary Write → write attacker-chosen memory
Pointer Corruption → modify pointers to redirect memory access
Function Pointer Corruption → redirect indirect function calls
Stack Corruption → damage stack variables / control data
Heap Corruption → damage heap objects / allocator metadata
Metadata Corruption → corrupt allocator bookkeeping
Uninitialized Memory → use memory before it is initialized
Null Pointer Dereference → access address 0 / NULL
Wild Pointer → use an invalid or uncontrolled pointer

Use-After-Free (UAF) → use an object after it has been freed
Double-Free → free the same allocation more than once
Use-After-Realloc → use an object after its allocation changed
Type Confusion → treat an object as the wrong type


════════════════════════════════════════════════════════════════════
📚 STACK EXPLOITATION
════════════════════════════════════════════════════════════════════

Stack Buffer Overflow → overwrite stack memory
Saved RBP Corruption → corrupt the saved frame pointer
RIP/EIP Overwrite → directly control instruction execution
Return Address Overwrite → replace the saved return address
Partial Return Address Overwrite → modify only part of RIP/EIP
Stack Canary Bypass → bypass stack-smashing protection
Stack Pivot → move the stack to attacker-controlled memory
leave; ret → pivot/control the stack using a common gadget
Stack Alignment → satisfy ABI stack-alignment requirements

ret2win → return directly to a hidden win function
ret2text → return to code inside the main executable
ret2plt → call functions through the PLT
ret2libc → call libc functions such as system()
ret2csu → use __libc_csu_init gadgets for register control
ret2syscall → construct a syscall directly
ret2dlresolve → abuse the dynamic linker to resolve a function
ret2vdso → use functions/code inside the VDSO
ret2shellcode → return to attacker-controlled shellcode
one_gadget → jump to a libc gadget that can spawn execution


════════════════════════════════════════════════════════════════════
🔗 ROP / CODE-REUSE
════════════════════════════════════════════════════════════════════

ROP (Return-Oriented Programming) → chain existing return-oriented gadgets
Gadget Hunting → find useful instruction sequences
ROP Chain → combine gadgets into an execution chain
Stack Pivot → move ROP execution to controlled memory
Partial ROP → use only limited available gadgets

JOP (Jump-Oriented Programming) → chain indirect jumps
COP (Call-Oriented Programming) → chain indirect calls
SROP → use sigreturn to control registers / execution state
BROP → perform ROP without knowing the binary beforehand
Blind Gadget Discovery → discover gadgets through crashes / behavior

ROP + libc Leak → leak libc then build a libc-based ROP chain
ROP + PIE Leak → leak the binary base then use PIE-relative gadgets
ROP + Stack Leak → use leaked stack addresses
ROP + Format String → combine format-string primitives with ROP


════════════════════════════════════════════════════════════════════
🖨️ FORMAT STRING EXPLOITATION
════════════════════════════════════════════════════════════════════

Format String Bug → attacker controls printf-style formatting
Format String Leak → leak stack / heap / libc / PIE addresses
Stack Leak → read values from the stack
Arbitrary Read → read attacker-selected memory
Arbitrary Write → write attacker-selected memory

%n → write the number of printed bytes
%hn → perform a 2-byte write
%hhn → perform a 1-byte write
%ln → perform a larger integer write

GOT Overwrite → overwrite a GOT function pointer
Return Address Overwrite → overwrite saved control-flow data
Function Pointer Overwrite → redirect indirect calls
printf Abuse → exploit attacker-controlled format strings
snprintf Abuse → exploit formatting operations with controlled input

Format String + libc Leak → obtain libc base
Format String + PIE Leak → obtain executable base
Format String + ROP → turn the primitive into code execution


════════════════════════════════════════════════════════════════════
🧠 HEAP EXPLOITATION
════════════════════════════════════════════════════════════════════

Heap Overflow → overwrite adjacent heap data
Heap Underflow → corrupt data before an allocation
Heap Metadata Corruption → corrupt allocator bookkeeping
Heap UAF → use a freed heap object
Double-Free → free the same chunk multiple times
Use-After-Realloc → use stale heap references

Fastbin Attacks → exploit fastbin allocator behavior
Fastbin Dup → abuse duplicate fastbin entries
Fastbin Dup into Stack → return a chunk pointing toward stack memory
Fastbin Dup into Arbitrary Location → redirect allocation to controlled memory

Tcache Poisoning → corrupt tcache freelist pointers
Tcache Dup → create duplicate tcache entries
Tcache Stashing → abuse tcache stashing behavior
Tcache Stashing Unlink → abuse small-bin/tcache interactions
Tcache House → techniques involving tcache manipulation
Safe-Linking Bypass → defeat modern encoded freelist pointers

Unsorted Bin Attack → abuse unsorted-bin metadata
Small Bin Attack → manipulate small-bin structures
Large Bin Attack → manipulate large-bin structures
Unlink Attack → abuse chunk unlink operations
Unsafe Unlink → corrupt pointers through unlink
Top Chunk Corruption → corrupt the top heap chunk
Wilderness Exploitation → exploit the top/wilderness chunk
malloc_consolidate Abuse → exploit heap consolidation

House of Spirit → forge a fake chunk
House of Force → manipulate the top chunk
House of Lore → exploit small-bin freelists
House of Einherjar → exploit off-by-one / consolidation
House of Orange → exploit top-chunk / FILE behavior
House of Rabbit → heap freelist manipulation
House of Botcake → tcache/small-bin exploitation
House of Apple → FILE-oriented exploitation
House of Kiwi → FILE/libc exploitation
House of Corrosion → advanced heap corruption


════════════════════════════════════════════════════════════════════
🏠 GLIBC / MALLOC INTERNALS
════════════════════════════════════════════════════════════════════

Chunk → basic unit used by glibc malloc
Chunk Header → allocator metadata attached to a chunk

prev_size → size information for the previous chunk
size → current chunk size / flags
fd → forward freelist pointer
bk → backward freelist pointer
fd_nextsize → large-bin forward size pointer
bk_nextsize → large-bin backward size pointer

Fastbins → freelists for small fast allocations
Tcache → per-thread cache for recently freed chunks
Unsorted Bin → temporary bin for freed chunks
Small Bins → bins for small-sized chunks
Large Bins → bins for larger chunks

main_arena → main glibc heap allocator arena
Top Chunk → chunk representing the top of the heap
Last Remainder → leftover chunk from splitting
Consolidation → merge adjacent free chunks

malloc() → allocate heap memory
free() → release heap memory
calloc() → allocate zeroed memory
realloc() → resize an allocation

Safe-Linking → protects freelist pointers
Safe-Unlinking → validates unlink operations
Malloc Checks → allocator consistency checks
Arena Manipulation → exploit/manipulate allocator arenas


════════════════════════════════════════════════════════════════════
🏛️ GOT / PLT / ELF
════════════════════════════════════════════════════════════════════

GOT → table containing dynamically resolved function addresses
PLT → stubs used to call dynamically linked functions

GOT Leak → leak addresses from the Global Offset Table
GOT Overwrite → replace a GOT function address
PLT Hijacking → redirect calls through modified resolution data
ret2plt → call functions through PLT entries

Lazy Binding → dynamically resolve functions when first called
Dynamic Linker Abuse → manipulate runtime symbol resolution
ret2dlresolve → force the linker to resolve an attacker-chosen symbol
_dl_runtime_resolve → internal dynamic-linker resolution mechanism

ELF Relocation Abuse → manipulate relocation information
ELF Section Abuse → exploit writable / useful ELF sections

.fini_array Overwrite → hijack termination callbacks
.init_array Abuse → abuse initialization callbacks
Destructor Overwrite → redirect destructor execution
Function Pointer Overwrite → redirect indirect function calls

No RELRO → GOT remains writable
Partial RELRO → some relocation protections are enabled
Full RELRO → GOT becomes read-only after relocation


════════════════════════════════════════════════════════════════════
🧬 LIBC EXPLOITATION
════════════════════════════════════════════════════════════════════

Libc Leak → obtain an address inside libc
Libc Base Calculation → calculate libc base from a leaked address
ret2libc → call libc functions using controlled execution
system("/bin/sh") → use libc system() to execute a shell
execve() → execute a program directly
one_gadget → jump to a suitable libc execution gadget

__free_hook → historical libc function-hook target
__malloc_hook → historical libc allocation-hook target
__realloc_hook → historical libc reallocation-hook target
__exit_funcs → manipulate functions executed during exit

Libc Function Pointer Corruption → redirect libc-controlled calls
Libc Address Overwrite → replace important libc pointers


════════════════════════════════════════════════════════════════════
📁 FILE / FSOP
════════════════════════════════════════════════════════════════════

_IO_FILE → glibc internal structure used by stdio streams
FILE Corruption → corrupt FILE structures
Fake FILE → construct attacker-controlled FILE objects
Fake Vtable → replace FILE virtual-function tables
FILE Vtable Corruption → redirect FILE operations

FSOP → File Stream Oriented Programming
_IO_list_all → linked list of active FILE structures
_IO_str_jumps → FILE operation jump table
_IO_wfile_jumps → wide-character FILE operations
_IO_wdefault_xsputn → wide-character output operation

Wide-Character FILE Attacks → exploit wide FILE structures
stdin Corruption → manipulate standard input
stdout Corruption → manipulate standard output
stderr Corruption → manipulate standard error
Exit-Time FILE Exploitation → trigger corruption during program exit

House of Apple → advanced FILE-oriented exploitation


════════════════════════════════════════════════════════════════════
🧩 C++ / OBJECT EXPLOITATION
════════════════════════════════════════════════════════════════════

C++ UAF → use a freed C++ object
Vtable Overwrite → replace virtual-function table
Vptr Corruption → corrupt an object's virtual table pointer
Fake Vtable → construct an attacker-controlled virtual table

Object Confusion → confuse one object with another
Type Confusion → interpret an object as the wrong type
Destructor Abuse → abuse object destruction
Virtual Function Hijacking → redirect virtual calls

Multiple-Inheritance Confusion → exploit complex C++ object layouts
RTTI Abuse → manipulate runtime type information
C++ Heap Exploitation → combine object bugs with heap corruption


════════════════════════════════════════════════════════════════════
🔢 INTEGER / SIZE / LOGIC BUGS
════════════════════════════════════════════════════════════════════

Integer Overflow → arithmetic exceeds the integer's maximum value
Integer Underflow → arithmetic goes below the minimum value
Signed Integer Overflow → overflow involving signed integers
Signed/Unsigned Confusion → incorrect interpretation of signed values
Integer Truncation → larger integer reduced to a smaller type
Integer Conversion → unsafe integer type conversion

size_t Bugs → incorrect handling of allocation sizes
Length-Field Corruption → manipulate length values
Negative-Size Bugs → use negative values to bypass checks
Multiplication Overflow → overflow during size calculations
Allocation-Size Overflow → cause an undersized allocation

Comparison Bypass → manipulate values to defeat checks
Logic Bug → exploit incorrect program logic
State Confusion → place the program into an unintended state


════════════════════════════════════════════════════════════════════
🎯 EXPLOITATION PRIMITIVES
════════════════════════════════════════════════════════════════════

Arbitrary Read → read attacker-selected memory
Arbitrary Write → write attacker-selected memory
Relative Read → read memory relative to a known address
Relative Write → write memory relative to a known address

Stack Read → leak stack memory
Stack Write → modify stack memory
Heap Read → leak heap memory
Heap Write → modify heap memory
Libc Read → leak libc memory
Libc Write → modify libc memory

Pointer Overwrite → replace a pointer
Function Pointer Overwrite → redirect a function call
GOT Overwrite → redirect dynamically linked functions
Return Address Overwrite → redirect control flow
Vtable Overwrite → redirect C++ virtual calls


════════════════════════════════════════════════════════════════════
🌎 INFORMATION LEAKS / ASLR / PIE
════════════════════════════════════════════════════════════════════

Information Leak → disclose addresses or sensitive memory
Stack Leak → leak stack addresses
Heap Leak → leak heap addresses
Libc Leak → leak libc addresses
PIE Leak → leak executable addresses
Binary-Base Leak → calculate the PIE base

GOT Leak → leak dynamically resolved addresses
Return-Address Leak → leak code addresses from the stack
environ Leak → leak a stack address through libc
__libc_start_main Leak → leak a libc-related address

Format-String Leak → leak memory through format strings
Uninitialized-Memory Leak → disclose uninitialized data
Partial Pointer Leak → disclose only part of an address

ASLR Bypass → defeat randomized memory locations
PIE Bypass → defeat randomized executable base


════════════════════════════════════════════════════════════════════
🛡️ MITIGATIONS / BYPASSES
════════════════════════════════════════════════════════════════════

NX → prevents execution from non-executable memory
NX Bypass → use ROP / ret2libc / other code-reuse techniques

ASLR → randomizes memory addresses
ASLR Bypass → obtain leaks and calculate addresses

PIE → randomizes the executable base
PIE Bypass → leak the executable base

Stack Canary → detects stack corruption
Canary Leak → obtain the canary value
Canary Bypass → avoid or overwrite the canary correctly

RELRO → protects relocation/GOT structures
RELRO Bypass → find another writable control target

FORTIFY_SOURCE → adds checks to certain libc operations
FORTIFY Bypass → exploit weaknesses outside the protected operation

CET → hardware-assisted control-flow protection
Shadow Stack → protects return addresses
CET Bypass → find alternative control-flow primitives

Pointer Mangling → encodes sensitive pointers
Pointer-Mangling Bypass → recover or manipulate encoded pointers

Safe-Linking → protects heap freelist pointers
Safe-Linking Bypass → recover/control encoded freelist pointers


════════════════════════════════════════════════════════════════════
🐚 SHELLCODE
════════════════════════════════════════════════════════════════════

Shellcode → attacker-controlled machine code
Stack Shellcode → execute shellcode placed on the stack
Heap Shellcode → execute shellcode placed on the heap
.bss Shellcode → execute shellcode stored in BSS
RWX Shellcode → execute from read/write/execute memory

mprotect Shellcode → make memory executable
mmap Shellcode → allocate executable memory
execve Shellcode → execute a program
ORW Shellcode → Open → Read → Write a file

open/read/write → directly perform file operations
Null-Free Shellcode → avoid NULL bytes
Alphanumeric Shellcode → restrict shellcode to alphanumeric bytes
Encoded Shellcode → encode payload bytes
Decoder Stub → decode shellcode at runtime
Staged Shellcode → load the real payload in stages
Polymorphic Shellcode → change shellcode representation


════════════════════════════════════════════════════════════════════
⚙️ SYSCALL-ORIENTED EXPLOITATION
════════════════════════════════════════════════════════════════════

ret2syscall → directly invoke syscalls through ROP
SROP → use sigreturn to control registers

execve → execute a program
execveat → execute through a file descriptor
read → read data into controlled memory
write → write data somewhere
open → open a file
openat → open a file relative to a directory/file descriptor
dup2 → duplicate file descriptors
mmap → allocate memory
mprotect → change memory permissions
prctl → manipulate process behavior

Syscall Chaining → chain multiple syscalls
ORW → open → read → write
Seccomp Interaction → work within syscall restrictions


════════════════════════════════════════════════════════════════════
🥷 BLIND / REMOTE PWN
════════════════════════════════════════════════════════════════════

BROP → exploit a remote binary without having its binary
Blind Memory Disclosure → discover memory through program behavior
Crash Oracle → use crashes as information
Byte-by-Byte Leak → recover data one byte at a time

Remote Offset Discovery → discover offsets remotely
Blind Gadget Discovery → discover useful gadgets remotely
Service Fingerprinting → identify the remote service
Remote libc Identification → determine the libc version remotely


════════════════════════════════════════════════════════════════════
🧪 RACE CONDITIONS
════════════════════════════════════════════════════════════════════

Race Condition → exploit timing between concurrent operations
TOCTOU → Time-of-Check to Time-of-Use race
File Race → race filesystem operations
Heap Race → race heap operations
Thread Race → race multiple threads

Double-Fetch → data changes between two reads
Concurrent UAF → trigger UAF through concurrent execution
Signal Race → exploit signal timing
Race-Based Privilege Escalation → race security-sensitive operations


════════════════════════════════════════════════════════════════════
🧵 THREAD / CONCURRENCY EXPLOITATION
════════════════════════════════════════════════════════════════════

Thread UAF → use an object after another thread frees it
Thread Race → exploit concurrent thread execution
Double-Fetch → exploit inconsistent data between accesses
Lock Corruption → corrupt synchronization structures
pthread Object Corruption → corrupt pthread-related objects

TLS Corruption → corrupt Thread Local Storage
Thread-Local Storage Exploitation → abuse TLS-controlled data
Thread Stack Corruption → corrupt another thread's stack


════════════════════════════════════════════════════════════════════
🔐 SANDBOX / SECCOMP
════════════════════════════════════════════════════════════════════

Seccomp → restrict allowed system calls
Seccomp Analysis → determine permitted syscalls
Seccomp Bypass → achieve execution within restrictions
Allowed-Syscall Abuse → use permitted syscalls creatively

ORW under Seccomp → read files using allowed syscalls
SROP under Seccomp → manipulate registers through sigreturn
openat Tricks → use permitted openat operations
read/write Chains → use allowed I/O syscalls
mmap/mprotect Chains → construct memory permissions using allowed calls


════════════════════════════════════════════════════════════════════
🖥️ KERNEL PWN
════════════════════════════════════════════════════════════════════

Kernel Buffer Overflow → overwrite kernel memory
Kernel Heap Overflow → corrupt kernel heap objects
Kernel UAF → use freed kernel objects
Kernel OOB → access kernel memory outside bounds
Kernel Double-Free → free kernel memory twice
Kernel Race → exploit kernel concurrency bugs

Arbitrary Kernel Read → read arbitrary kernel memory
Arbitrary Kernel Write → write arbitrary kernel memory
Kernel Info Leak → leak kernel addresses
Kernel ROP → execute ROP inside the kernel
Kernel Stack Pivot → move kernel execution to controlled memory

cred Overwrite → modify process credentials
task_struct Corruption → corrupt Linux process structures
Kernel Heap Spraying → place controlled objects in kernel memory

KASLR Bypass → defeat kernel address randomization
SMEP Bypass → bypass supervisor-mode execution protection
SMAP Bypass → bypass supervisor-mode access protection
KPTI Bypass → bypass kernel page-table isolation
ret2usr → redirect kernel execution toward user memory

msg_msg Spraying → use System V messages for heap manipulation
pipe_buffer Exploitation → manipulate Linux pipe buffer structures
seq_operations → exploit sequence-operation objects
tty_struct → exploit TTY structures
timerfd_ctx → exploit timerfd objects
userfaultfd → control page-fault handling for exploitation
Dirty-Pipe-Style Primitive → exploit pipe-buffer corruption primitives


════════════════════════════════════════════════════════════════════
🏗️ DATA-ORIENTED / CONTROL-FLOW TECHNIQUES
════════════════════════════════════════════════════════════════════

ROP → manipulate control flow using return instructions
JOP → manipulate control flow using indirect jumps
COP → manipulate control flow using indirect calls
SROP → manipulate execution through sigreturn

DOP (Data-Oriented Programming) → modify data without hijacking control flow
COOP → Counterfeit Object-Oriented Programming
FSOP → control execution through FILE structures

Function Pointer Hijacking → redirect indirect calls
Vtable Hijacking → redirect C++ virtual calls
Callback Hijacking → redirect callback execution


════════════════════════════════════════════════════════════════════
📱 ARCHITECTURES / ABIs
════════════════════════════════════════════════════════════════════

x86 → 32-bit Intel architecture
x86-64 → 64-bit Intel/AMD architecture
ARM → ARM architecture
AArch64 → 64-bit ARM architecture
MIPS → MIPS architecture
RISC-V → RISC-V architecture
PowerPC → PowerPC architecture

Calling Convention → determines how arguments/returns are passed
ABI → defines binary-level calling and data conventions
Register Control → control architecture-specific registers
Stack Layout → understand architecture-specific stack organization


════════════════════════════════════════════════════════════════════
🧨 SPECIALIZED / "HOUSE OF" TECHNIQUES
════════════════════════════════════════════════════════════════════

House of Spirit → fake heap chunk technique
House of Force → top-chunk manipulation
House of Lore → small-bin manipulation
House of Einherjar → off-by-one / consolidation technique
House of Orange → top-chunk + FILE exploitation
House of Rabbit → freelist manipulation
House of Botcake → tcache + small-bin technique
House of Apple → FILE exploitation
House of Kiwi → FILE/libc exploitation
House of Corrosion → advanced heap exploitation

Poison Null Byte → corrupt heap metadata using NULL-byte overwrite
Unsafe Unlink → exploit corrupted unlink metadata
Fastbin Dup → duplicate fastbin chunks
Tcache Poisoning → corrupt tcache freelists
Unsorted Bin Attack → manipulate unsorted-bin metadata
Large Bin Attack → manipulate large-bin metadata

FILE-Oriented Programming → exploit FILE objects
Data-Oriented Programming → manipulate program data without direct control flow
Control-Flow Hijacking → redirect execution
Code-Reuse Attack → reuse existing executable code


════════════════════════════════════════════════════════════════════
🧰 EXPLOIT DEVELOPMENT / PWN WORKFLOW
════════════════════════════════════════════════════════════════════

Recon → understand the binary and its attack surface
Reverse Engineering → understand program behavior
Bug Identification → find the vulnerability
Crash Analysis → determine what can be controlled
Offset Finding → determine exact overwrite positions

Primitive Building → turn the bug into read/write/control
Information Leak → obtain useful addresses
Base Calculation → calculate PIE/libc/heap bases
Mitigation Analysis → understand NX/PIE/ASLR/RELRO/Canary/etc.
Gadget Hunting → find useful ROP gadgets
Payload Construction → build the exploit
Local Exploitation → develop against the local binary
Remote Exploitation → adapt the exploit to the remote service

Libc Identification → determine the correct libc
Environment Matching → reproduce the target environment
Debugging → inspect registers / stack / heap / memory
Reliability → make the exploit work consistently

Final Goal → arbitrary code execution / shell / flag


PWN
│
├── 1. MEMORY BUGS
│   ├── Stack Overflow
│   ├── Heap Overflow
│   ├── UAF
│   ├── Double Free
│   ├── OOB Read/Write
│   ├── Format String
│   ├── Integer Bugs
│   └── Type Confusion
│
├── 2. STACK
│   ├── ret2win
│   ├── ret2libc
│   ├── ret2plt
│   ├── ret2csu
│   ├── ret2syscall
│   ├── ret2dlresolve
│   ├── Stack Pivot
│   └── Canary Bypass
│
├── 3. ROP
│   ├── ROP
│   ├── JOP
│   ├── COP
│   ├── SROP
│   └── BROP
│
├── 4. FORMAT STRING
│   ├── Leak
│   ├── Arbitrary Read
│   ├── Arbitrary Write
│   └── GOT / RIP / Function Pointer overwrite
│
├── 5. HEAP
│   ├── Fastbin
│   ├── Tcache
│   ├── Unsorted Bin
│   ├── Small Bin
│   ├── Large Bin
│   ├── UAF
│   ├── Double Free
│   ├── Heap Metadata
│   └── House of ███
│
├── 6. LIBC
│   ├── Libc Leak
│   ├── ret2libc
│   ├── one_gadget
│   ├── Hooks
│   └── FILE / FSOP
│
├── 7. ELF
│   ├── GOT
│   ├── PLT
│   ├── RELRO
│   ├── .fini_array
│   ├── .init_array
│   └── ret2dlresolve
│
├── 8. LEAKS
│   ├── Stack
│   ├── Heap
│   ├── PIE
│   ├── Libc
│   └── GOT
│
├── 9. MITIGATIONS
│   ├── NX
│   ├── ASLR
│   ├── PIE
│   ├── Canary
│   ├── RELRO
│   ├── FORTIFY
│   ├── CET
│   ├── Safe-Linking
│   └── Seccomp
│
├── 10. SHELLCODE / SYSCALLS
│   ├── Shellcode
│   ├── ORW
│   ├── ret2syscall
│   └── SROP
│
├── 11. CONCURRENCY
│   ├── Race Conditions
│   ├── TOCTOU
│   ├── Double Fetch
│   ├── Thread UAF
│   └── TLS
│
├── 12. SANDBOX
│   ├── Seccomp
│   ├── ORW
│   ├── SROP
│   └── Syscall Restrictions
│
├── 13. C++ / OBJECTS
│   ├── UAF
│   ├── Type Confusion
│   ├── Vtable
│   ├── Vptr
│   └── Fake Objects
│
├── 14. KERNEL PWN
│   ├── Kernel UAF
│   ├── Kernel Heap
│   ├── Kernel OOB
│   ├── KASLR
│   ├── SMEP
│   ├── SMAP
│   ├── KPTI
│   └── Kernel ROP
│
└── 15. ARCHITECTURES
    ├── x86
    ├── x86-64
    ├── ARM
    ├── AArch64
    ├── MIPS
    └── RISC-V


```
