# Janken

Este reto viene acompañado de un archivo comprimido, que a su vez tiene un ejecutable binario, el cual asumimos es como se comporta el servidor. El objetivo es ganarle al servidor 100 veces consecutivas en un juego parecido al de piedra, papel o tijeras, si lo logramos, obtendremos la bandera.

Empecemos por analizar el binario

```
$ file janken 
janken: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter ./.glibc/ld-linux-x86-64.so.2, BuildID[sha1]=56b54cdae265aa352fe2ebb016f86af831fd58d3, for GNU/Linux 3.2.0, not stripped
```

Usando el comando file vemos que se trata de un binario en linux, y también nos informa que es `not stripped` esto quiere decir que al momento de compilar el programa, no se suprimió la información de depuración que hace que el analisis de ingeniería inversa sea más sencillo.

Ahora pasemos a Ghidra para ver la decompilación, dentro de la función `main`, se llama a la función `game`

```c
void game(void)

{
  int iVar1;
  time_t tVar2;
  ushort **ppuVar3;
  size_t sVar4;
  char *pcVar5;
  long in_FS_OFFSET;
  ulong local_88;
  char *local_78 [4];
  char *local_58 [4];
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  tVar2 = time((time_t *)0x0);
  srand((uint)tVar2);
  iVar1 = rand();
  local_78[0] = "rock";
  local_78[1] = "scissors";
  local_78[2] = "paper";
  local_38 = 0;
  local_30 = 0;
  local_28 = 0;
  local_20 = 0;
  local_58[0] = "paper";
  local_58[1] = &DAT_0010252a;
  local_58[2] = "scissors";
  fwrite(&DAT_00102540,1,0x33,stdout);
  read(0,&local_38,0x1f);
  fprintf(stdout,"\n[!] Guru\'s choice: %s%s%s\n[!] Your  choice: %s%s%s",&DAT_00102083,
          local_78[iVar1 % 3],&DAT_00102008,&DAT_0010207b,&local_38,&DAT_00102008);
  local_88 = 0;
  do {
    sVar4 = strlen((char *)&local_38);
    if (sVar4 <= local_88) {
LAB_001017a2:
      pcVar5 = strstr((char *)&local_38,local_58[iVar1 % 3]);
      if (pcVar5 == (char *)0x0) {
        fprintf(stdout,"%s\n[-] You lost the game..\n\n",&DAT_00102083);
                    /* WARNING: Subroutine does not return */
        exit(0x16);
      }
      fprintf(stdout,"\n%s[+] You won this round! Congrats!\n%s",&DAT_0010207b,&DAT_00102008);
      if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
        __stack_chk_fail();
      }
      return;
    }
    ppuVar3 = __ctype_b_loc();
    if (((*ppuVar3)[*(char *)((long)&local_38 + local_88)] & 0x2000) != 0) {
      *(undefined *)((long)&local_38 + local_88) = 0;
      goto LAB_001017a2;
    }
    local_88 = local_88 + 1;
  } while( true );
}
```

Esta función es la encargada de la logíca del juego y esta condición decide si el jugador gana o pierde

```c
// ...
read(0,&local_38,0x1f);
// ...
pcVar5 = strstr((char *)&local_38,local_58[iVar1 % 3]);
if (pcVar5 == (char *)0x0) {
  fprintf(stdout,"%s\n[-] You lost the game..\n\n",&DAT_00102083);
              /* WARNING: Subroutine does not return */
  exit(0x16);
}
```

La función `strstr` busca alguna coincidencia del segundo parametro (needle) en el primero (haystack), si la encuetra devuelve el puntero de la coincidencia, en caso contrario, devuelve un puntero nulo.

Hay que tener presente que el segundo parametro, que es la jugada del servidor, es completamente aleatorea y el jugador debe adivinar la opción del servidor. Por ejemplo tenemos

- Jugador: paper. Servidor: rock -> gana el servidor
- Jugador: scissors. Servidor: scissors -> gana el jugador
- Jugador: rock. Servidor: scissors -> gana el servidor

Ahora, volviendo a la función `strstr` nos fijamos que busca la cadena del servidor en la cadena del jugador y por otro lado, la entrada del jugador puede ser cualquier cadena. ¿Qué pasaría si el jugador envía `rockscissorspaper`?

- Jugador: rockscissorspaper. Servidor: rock -> la cadena rock esta en rockscissorspaper. Gana el jugador
- Jugador: rockscissorspaper. Servidor: scissors -> la cadena scissors esta en rockscissorspaper. Gana el jugador
- Jugador: rockscissorspaper. Servidor: paper -> la cadena paper esta en rockscissorspaper. Gana el jugador

Entonces simplemente debemos enviar 100 veces `rockscissorspaper` cuando el servidor lo requiera. Para esto tenemos el siguiente programa en python usando pwntools

```python
from pwn import *

context.log_level = 'debug'
#io = process("./janken")
io=remote("206.189.112.129",31880)

io.sendafter(b">> ", b"1")
for i in range(100):
    result = io.sendafter(b">> ", b"rockscissorspaper")
    print(result)
result = io.recv()
print(result)
io.close()
```

y al final tenemos de la ejecución la bandera

```
[DEBUG] Received 0xdf bytes:
    00000000  0a 5b 21 5d  20 47 75 72  75 27 73 20  63 68 6f 69  │·[!]│ Gur│u's │choi│
    00000010  63 65 3a 20  1b 5b 31 3b  33 31 6d 73  63 69 73 73  │ce: │·[1;│31ms│ciss│
    00000020  6f 72 73 1b  5b 31 3b 33  36 6d 0a 5b  21 5d 20 59  │ors·│[1;3│6m·[│!] Y│
    00000030  6f 75 72 20  20 63 68 6f  69 63 65 3a  20 1b 5b 31  │our │ cho│ice:│ ·[1│
    00000040  3b 33 32 6d  72 6f 63 6b  73 63 69 73  73 6f 72 73  │;32m│rock│scis│sors│
    00000050  70 61 70 65  72 1b 5b 31  3b 33 36 6d  0a 1b 5b 31  │pape│r·[1│;36m│··[1│
    00000060  3b 33 32 6d  5b 2b 5d 20  59 6f 75 20  77 6f 6e 20  │;32m│[+] │You │won │
    00000070  74 68 69 73  20 72 6f 75  6e 64 21 20  43 6f 6e 67  │this│ rou│nd! │Cong│
    00000080  72 61 74 73  21 0a 1b 5b  31 3b 33 36  6d 1b 5b 31  │rats│!··[│1;36│m·[1│
    00000090  3b 33 32 6d  5b 2b 5d 20  59 6f 75 20  61 72 65 20  │;32m│[+] │You │are │
    000000a0  77 6f 72 74  68 79 21 20  48 65 72 65  20 69 73 20  │wort│hy! │Here│ is │
    000000b0  79 6f 75 72  20 70 72 69  7a 65 3a 20  48 54 42 7b  │your│ pri│ze: │HTB{│
    000000c0  72 30 63 6b  5f 70 34 70  33 52 5f 35  74 72 35 74  │r0ck│_p4p│3R_5│tr5t│
    000000d0  72 5f 6c 30  67 31 63 5f  62 75 47 7d  0a 0a 0a     │r_l0│g1c_│buG}│···│
    000000df
```

flag `HTB{r0ck_p4p3R_5tr5tr_l0g1c_buG}`
