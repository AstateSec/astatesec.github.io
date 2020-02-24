# Introduction au heap overflow 

Bonjour à tous ! Vous avez peut-être dejas exploiter un buffer overflow sur la stack, je vous propose dans cette article de découvrire cette attaque mais cette fois sur la heap. L'objectif est de mener une exploitation simple pour introduire les heap overflows on va désactiver pas mal de sécurité. La différence majeur entre un stack overflow et un heap overflow est le fais que sur notre heap on a pas le couple saved rbp/saved rip, parfois on en pourra pas rediriger le flux de notre programme, tout depend de ce qui se trouve sur la heap...

Bon, depuis le debut je parle de heap mais c'est quoi ça ? Je ne vais pas trop tarder sur ça car il y a plein d'articles d'articles expliquant très bien le fonctionnement de la heap. 

Alors la heap est une partie de la mémoire comme la stack, contrairement à la stack la heap est alloué dynamiquement, c'est à dire que n'importe quel bloc de la heap peux être alloué ou libéré comme on veux, ce n'est pas comme la stack qui est un structure FIFO (First In First Out). Bon maintenant comment allouer de la mémoire dans la heap ? Eh bien il existe plusieur fonction comme ptmalloc, dlmalloc, malloc et free pour désallouer. Toute ces fonctions font appelle au syscall mmap et brk. On va rester sur malloc, je suppose que vous avez du C, je ne vais pas m'attarder là, passons donc à notre "méthodologie"

## Méthodologie 

Alors voilà vous êtes devant un heap overflow comment vous allez faire ? 

1. On trouve ou commence la heap avec la commande info proc map gdb
2. On pose un breakpoint après le malloc, et après le heap overflow
3. On fais juste un x/(un nombre)x addresse_du_debut_de_la_heap
4. On regarde les addresse après le buffer et on regarde à quoi ces addresse correspondent 
5. Il est temps d'être créatif et de voir que pouvons nous faire en changant ces addresses !

## Un première exploitation !

Très bien, on va prendre un code exemple (largement inspiré du dernier challenge heap du célèbre Protostar)

Le code:
```c
#include <stdlib.h>
#include <stdio.h>

void winner() {
  printf(" Bravo ! \n");
}

int main(void) {
  char *a, *b, *c;
  a = malloc(32);
  b = malloc(32);
  c = malloc(32);

  gets(a);
  gets(b);
  gets(c);

  free(c);
  free(b);
  free(a);

  return 0;
}
```

On compile le code avec
```gcc -no-pie -o heap heap.c```

Comme vous vous en doutez, notre objectif va être d'appeler la fonction ```winner()```, resortons ce bon vieux pwndbg !
