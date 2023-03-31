# Remote Computation

Observemos que índica la instancia remota

```
$ nc 178.62.9.10 32166
[-MENU-]
[1] Start
[2] Help
[3] Exit
> 2

Results
---
All results are rounded
to 2 digits after the point.
ex. 9.5752 -> 9.58

Error Codes
---
* Divide by 0:
This may be alien technology,
but dividing by zero is still an error!
Expected response: DIV0_ERR

* Syntax Error
Invalid expressions due syntax errors.
ex. 3 +* 4 = ?
Expected response: SYNTAX_ERR

* Memory Error
The remote machine is blazingly fast,
but its architecture cannot represent any result
outside the range -1337.00 <= RESULT <= 1337.00
Expected response: MEM_ERR
```

El objetivo de este reto es calcular las 500 operaciones que pide el servidor correctamente con dos dígitos de precisión. Además, debemos tener presente los errores de dividir por cero, sintaxis y memoria.

Para este reto usamos pwntools, una biblioteca muy útil ya que es más fácil recibir y enviar información en vez de usar sockets directamente.
El script final quedaría así

```python
from pwn import *

def build_response(exp_str):
    try:
        print(exp_str)
        r = eval(exp_str)
        print(r)
        if -1337.0 <= r and r <= 1337.0:
            return str(r).encode()
        else:
            return b"MEM_ERR"
    except ZeroDivisionError:
        return b"DIV0_ERR"
    except SyntaxError:
        return b"SYNTAX_ERR"

context.log_level = 'debug'
io = remote("178.62.9.10",32166)
io.sendafter(b"> ", b"1\n")
recv_exp = io.recv()
exp = recv_exp[44:]
exp = exp[:-6]
exp_str = "round("+exp.decode("ascii")+",2)"
response = build_response(exp_str)
io.send(response+b'\n')
for i in range(499):
    recv_exp = io.recv()
    exp = recv_exp[6:]
    exp = exp[:-6]
    exp_str = "round("+exp.decode("ascii")+",2)"
    response = build_response(exp_str)
    io.send(response+b'\n')
io.recv()
io.close()
```
Las operaciones se realizan en la función `build_response`. Allí, la función `eval` se encarga de ejecutar una cadena de código válido, que en este caso son solo operaciones aritméticas. Si tenemos una división por cero o un error de sintaxis  como por ejemplo `2 +* 9`, `eval` arrojará la excepción correspondiente.

Al ejecutar este script obtenemos la bandera

```
[DEBUG] Received 0x2d bytes:
    00000000  1b 5b 33 32  6d 5b 2a 5d  20 47 6f 6f  64 20 6a 6f  │·[32│m[*]│ Goo│d jo│
    00000010  62 21 20 48  54 42 7b 64  31 76 31 64  33 5f 62 59  │b! H│TB{d│1v1d│3_bY│
    00000020  5f 5a 33 72  30 5f 33 72  72 30 72 7d  0a           │_Z3r│0_3r│r0r}│·│
```

flag `HTB{d1v1d3_bY_Z3r0_3rr0r}`
