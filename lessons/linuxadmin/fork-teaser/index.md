# Funkce, které ovládají proces

Vytvoř si nový pythonní soubor v textovém editoru: `procesy.py`.
V něm si vyzkoušíme několik funkcí, které ovládají aktuálně běžící proces.

## Ukončení: exit

```python
print(len('abc'))
```
Normální funkce jako `len` a `print` se po zavolání vrátí;
když jsou hotové, program pokračuje dál.
Dokonce i `time.sleep(10000000)` se časem vrátí, jen to dlouho trvá.

Existují ale i funkce které se nevrací: třeba ty které vyvolají výjimku.
Program který na výjimku není připravený skončí.

Funkce `sys.exit()` vyhodí výjimku, která – když ji nezpracuješ – ukončí
program s danou návratovou hodnotou.

```python
import os
try:
    os._exit(7)
finally:
    print('Tohle se vypíše')
print('Tohle už ne')
```

### Tvrdší přístup

Speciální funkce `os._exit()` jde ještě dál: okamžitě ukončí proces.
Python nedostane možnost zpracovat výjimky, zavřít otevřené soubory,
ukončit síťové spojení a podobně.
Prostě se ukončí, hned, bez výjimek a bez diskuse.

```python
import os
try:
    os._exit(7)
finally:
    print('Tohle se nevypíše')
```

Program po zavolání `os._exit()` nepokračuje dál; tahle funkce se nevrací.
Argument funkce `_exit` je opět návratová hodnota procesu.

## Mutace: exec

Další funkce po které program nepokračuje dál je *exec*,
která aktuálně prováděný program nahradí jiným.

Existuje v několika variantách (podle toho, jak se hledá nový program);
nejjednodušší je asi:

```python
import os
os.execvp('echo', ['echo', '123'])
print('Tohle se nevypíše')
```

Proces ve kterém program běží začne místo Pythonu provádět příkaz `echo 123`.
Jako u `os._exit` Python nedostane možnost zpracovat výjimky nebo jinak
„uklidit“.
Prostě se pustí od začátku jiný program se stejným číslem (PID).


## Klonování: fork

Když máme funkce, které se nevrátí, a taky funkce, které se vrátí jednou,
můžou existovat funkce, které se vrátí víckrát? Zkus:

```python
import os
os.fork()
print("Ahoj!")
```

Hláška se vypíše dvakrát.

Funkce `fork` totiž umí *naklonovat* aktuální proces.
Proces se rozdvojí a každá kopie pokračuje, jako by se funkce `fork` právě vrátila.

Tyto dva procesy se liší jen v jedné jediné věci - tím, co vrátí `os.fork()`.

```python
import os
os.fork()
hodnota = os.fork()
print("Fork vrací:", hodnota)
```

```console
(venv)$ python procesy.py
Fork vrací: 1827
Fork vrací: 0
```

V jednom procesu je návratová hodnota `0`, ve druhém je to vyšší číslo.

Jeden proces, tzv. „rodič“, totiž dostal PID druhého procesu,
„potomka", aby ho mohl ovládat.
Druhý proces dostal 0, aby poznal že je „potomek".

Většinou se `fork()` volá v podmínce `if`u, aby každý proces dělal
něco jiného:

```python
import os
os.fork()
pid = os.fork()
if pid == 0:
    print("já jsem dítě")
else:
    print("já jsem rodič")
```

Spousta programů před forkem často vytvoří rouru, což je speciální soubor
který má dva „konce“: jeden pro čtení a druhý pro zápis.

Existuje na to funkce `os.pipe()`.
Ta rouru vytvoří a vrátí dvojicí deskriptorů (čísel souborů):
první pro čtení a druhý pro zápis.

```python
import os

r, w = os.pipe()

pid = os.fork()
if pid == 0:
    print("já jsem dítě, moje PID je", os.getpid())
    os.close(r)
    os.write(w, b'Ahoj!')
    os.close(w)
else:
    print("já jsem rodič, dítě je", pid)
    os.close(w)
    pozdrav = os.read(r, 5)
    print('dítě říká:', pozdrav)
    os.close(r)
```

Co dělá tento kód?
„Potomek“ vypíše že je dítě a zavře konec roury pro čtení. 
Pak napíše pozdrav "Ahoj" do konce pro zápis.
Nakonec zavře i tento konec.

„Rodič“ vypíše že je rodič, zavře rouru pro zápis
a přečte prvních 10 bajtů z konce pro čtení, a zavře i ten.

Takto si můžou procesy „povídat“ – jeden z nich vytvoří rouru, kterou
„zdědí“ jeho „potomek“.


## A k čemu to je?

To si řekneme příště.
