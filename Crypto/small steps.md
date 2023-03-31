# Small Steps

Para este reto uno debe conectarse a una instancia remota, pero tenemos acceso al código fuente del servidor

```python
from Crypto.Util.number import getPrime, bytes_to_long

FLAG = b"HTB{???????????????}"
assert len(FLAG) == 20


class RSA:

    def __init__(self):
        self.q = getPrime(256)
        self.p = getPrime(256)
        self.n = self.q * self.p
        self.e = 3

    def encrypt(self, plaintext):
        plaintext = bytes_to_long(plaintext)
        return pow(plaintext, self.e, self.n)


def menu():
    print('[E]ncrypt the flag.')
    print('[A]bort training.\n')
    return input('> ').upper()[0]


def main():
    print('This is the second level of training.\n')
    while True:
        rsa = RSA()
        choice = menu()

        if choice == 'E':
            encrypted_flag = rsa.encrypt(FLAG)
            print(f'\nThe public key is:\n\nN: {rsa.n}\ne: {rsa.e}\n')
            print(f'The encrypted flag is: {encrypted_flag}\n')
        elif choice == 'A':
            print('\nGoodbye\n')
            exit(-1)
        else:
            print('\nInvalid choice!\n')
            exit(-1)


if __name__ == '__main__':
    main()
```

El servidor nos entrega la bandera cifrada con la llave pública, pero desafortunadamente no tenemos información acerca de la llave privada. Sin embargo, el exponente que usa el servidor es 3, un número muy pequeño.

Para realizar el cifrado RSA usamos la formula

```math
c = m^e \mod N
```

Generalmente el exponente es 0x10001, pero al ser el exponente 3 en este caso, el resultado de $m^3$ puede ser menor al modulo $N$ y basicamente la operación modulo no tiene ningun efecto en el cifrado. Por lo tanto para recuperar el mensaje original

```math
m = \sqrt[3]{c}
```

El servidor nos da la siguiente bandera cifrada

```
70407336670535933819674104208890254240063781538460394662998902860952366439176467447947737680952277637330523818962104685553250402512989897886053
```

usemos python para decifrarla

```python
>>> from Crypto.Util.number import long_to_bytes
>>> c = 70407336670535933819674104208890254240063781538460394662998902860952366439176467447947737680952277637330523818962104685553250402512989897886053
>>> m = pow(c,1/3)
>>> long_to_bytes(m)
b'HTB{5l\xe4\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
```

Nos podemos dar cuenta que no obtenemos la bandera por completo. La razón de esto es por los dígitos de precisión que utiliza la función `pow`. Si queremos mejorar la precisión, tendremos que usar una biblioteca como mpmath o sage. A continuación se muestra la operación usando mpmath.

```
>>> from Crypto.Util.number import long_to_bytes
>>> import mpmath as mp
>>> from mpmath import *
>>> mp.dps = 80 # fijamos los dígitos de presición
>>> c = 70407336670535933819674104208890254240063781538460394662998902860952366439176467447947737680952277637330523818962104685553250402512989897886053
>>> mpf(c) ** mpf('1/3')
mpf('412926389432612660984016953290834154417829082236.99999999999999999999999999999997612')
>>> long_to_bytes(412926389432612660984016953290834154417829082237) # aproximamos
b'HTB{5ma1l_E-xp0n3nt}'
```

flag `HTB{5ma1l_E-xp0n3nt}`
