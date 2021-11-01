# Fork a exec

> [note]
> Tahle sekce je tu pro ty, kdo chtějí jít víc do hloubky.

Spouštění příkazů funguje pomocí dvou operací: `fork` a `exec`.
Podívejme se na ně v opačném pořadí.

## Exec

Operace `exec`, pro kterou existuje i příkaz Bashe, *nahradí* aktuální
proces jiným.
Právě prováděný proces (Bash) se přestane provádět a místo toho se začne
provádět jiný – ovšem se stejnými otevřenými soubory a proměnnými prostředí,
jaké měl ten původní.

Zkus si to. V jednom terminálu zjisti číslo procesu Bashe:

```console
$ echo $$
```

A ve druhém si vypiš příslušný proces.
Místo `ps aux | grep <číslo>` můžeš použít zkratku:

```console
$ ps --pid <číslo procesu>
    PID TTY          TIME CMD
    ... pts/0    00:00:00 bash
```

Potom v původním terminálu zadej:

```console
$ exec top
```

Aktuální proces, ve kterém běží Bash, se „přepíše“ programem `top`.
Zkontroluj to ve druhém terminálu:

```console
$ ps --pid <číslo procesu>
    PID TTY          TIME CMD
    ... pts/0    00:00:00 top
```

Stejný proces, jiný program!

Když teď `top` ukončíš (pomocí <kbd>Q</kbd>), zavře se okýnko s terminálem.
Proces v terminálu skončil, tak jako kdybys v Bashi napsal{{a}} `exit`.

Pro dlouhé zimní večery přikládám k prostudování tento program v Pythonu:

```python
import os

# Vytvoření roury.
# Finkce `pipe` vrací dva deskriptory souborů pro dva "konce" roury:
# `r` je konec pro čtení, `w` je konec pro zápis.
r, w = os.pipe()

# Přesměrování vstupu.
# Soubor `r` tedy konec roury ze kterého se dá číst, nastavíme jako
# standardní vstup (0) tohoto procesu.
os.dup2(r, 0)

# Zápis do roury.
# Do konce určeného pro zápis, `w`, zapíšeme bajty odpovídající
# textu "Ahoj!".
os.write(w, b'Ahoj!\n')

# Uzavření konce pro zápis.
# Když ze program, který z roury bude číst, dostane po 'Ahoj!\n' až sem,
# dostane signál konce souboru.
os.close(w)

# Nahrazení aktuálního procesu programem `cat`.
# (První argument je jméno programu, které se hledá v $PATH; druhý je celý
# seznam argumentů, jako sys.argv – tedy i se jménem programu na začátku.)
# Zkus 'cat' nahradit za 'wc' nebo dokonce 'less'!
os.execvp('cat', ['cat'])

print('Tohle se nevypíše.')
```

> [note]
> Tohle je hodně nízkoúrovňový způsob pouštění programů!
> Kdybys chtěl{{a}} z Pythonu pouštět programy, použij
> [`subprocess.run`](https://docs.python.org/3/library/subprocess.html#subprocess.run),
> který podobné věci dělá za tebe (a navíc funguje i na Windows,
> kde je všechno jinak).


## Fork

Ještě zajímavější operace je `fork`, která běžící proces *rozdělí* na
dvě stejné kopie.
Ta nemá ekvivalent v Bashi, ale v Pythonu ano.

Dva procesy, které pomocí `fork` vzniknou, se liší v jednom detailu:
návratové hodnotě funkce `fork`.
Ta je v novém procesu `0` a ve starém je to PID nového procesu.

Zkus si to v Pythonu:

```python
print('Toto je proces', os.getpid())

r = os.fork()
pid = os.getpid()

print('V procesu', pid, 'je výsledek', r)
```


## Fork a exec

A jak tedy Bash pouští programy?

Napřed se „naklonuje“ pomocí `fork`.
V novém procesu (který z `fork` dostal 0) pak přesměruje vstupy a výstupy,
ponastavuje proměnné prostředí, pozavírá soubory které v novém programu nejsou
potřeba atd. – a když je vše nastavené, pomocí `exec` spustí daný program.
Ve starém procesu (který z `fork` dostal PID) pak čeká, až proces s daným
PID skončí.

V reálu je situace trochu složitější (Bash např. funguje i na systémech které
`fork` nemají), ale dává-li ti `fork` a `exec` smysl, zjednoduší ti to
přemýšlení nad tím co Bash vlastně dělá.


  {{ figure(
    img=static('fork-exec.svg'),
    alt='Diagram spouštění `ps`',
  ) }}

