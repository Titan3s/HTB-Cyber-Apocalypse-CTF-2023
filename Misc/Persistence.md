# Persistence

> Thousands of years ago, sending a GET request to /flag would grant immense power and wisdom. Now it's broken and usually returns random data, but keep trying, and you might get lucky... Legends say it works once every 1000 tries.

En este reto el objetivo es enviar al menos 1000 peticiones al servidor con la ruta /flag para que responda con la bandera. Un simple script en python sera suficiente.

```python
import requests

for i in range(1000):
    r = requests.get('http://46.101.73.33:32109/flag')
    if r.text.startswith('HTB'):
        print(r.text)
        break
```

flag `HTB{y0u_h4v3_p0w3rfuL_sCr1pt1ng_ab1lit13S!}`
