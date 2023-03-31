# Ancient Encodings

Este reto viene con dos archivos: output.txt y source.py. Observemos primero `output.txt`

```
0x53465243657a467558336b7764584a66616a4231636d347a655639354d48566664326b786246397a5a544e66644767784e56396c626d4d775a4446755a334e665a58597a636e6c33614756794d33303d
```

Este archivo solo tiene un número en hexadecimal, ahora, el archivo fuente `source.py`

```python
from Crypto.Util.number import bytes_to_long
from base64 import b64encode

FLAG = b"HTB{??????????}"


def encode(message):
    return hex(bytes_to_long(b64encode(message)))


def main():
    encoded_flag = encode(FLAG)
    with open("output.txt", "w") as f:
        f.write(encoded_flag)


if __name__ == "__main__":
    main()
```

En este script la bandera esta siendo codificada (más no cifrada) a base 64 lo pasa a hexadecimal y escribe el resultado en output.txt. Para obtener la bandera en su forma original, simplemente debemos tomar el número en hexadecimal, convertirlo a `bytes` y decodificar en base 64.

```python
>>> from Crypto.Util.number import long_to_bytes
>>> from base64 import b64decode
>>> c = 0x53465243657a467558336b7764584a66616a4231636d347a655639354d48566664326b786246397a5a544e66644767784e56396c626d4d775a4446755a334e665a58597a636e6c33614756794d33303d
>>> b64decode(long_to_bytes(c))
b'HTB{1n_y0ur_j0urn3y_y0u_wi1l_se3_th15_enc0d1ngs_ev3rywher3}'
```
flag `HTB{1n_y0ur_j0urn3y_y0u_wi1l_se3_th15_enc0d1ngs_ev3rywher3}`
