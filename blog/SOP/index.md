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

