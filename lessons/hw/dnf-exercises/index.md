# Úkoly - Balíčky

## Balíček s příkazem `python`

Balíček který obsahuje příkaz `python3` se jmenuje `python3`.
Máš nainstalovanou jednu jeho verzi; v repozitářích jich může být
k dispozici víc:

```console
$ rpm -q --whatprovides /usr/bin/python3
python3-3.9.7-1.fc34.x86_64
$ dnf repoquery /usr/bin/python3
python3-0:3.9.2-1.fc34.i686
python3-0:3.9.2-1.fc34.x86_64
python3-0:3.9.7-1.fc34.i686
python3-0:3.9.7-1.fc34.x86_64
```

Jak se jmenuje balíček, který obsahuje příkaz `python`, bez `3` na konci?

## Odinstalování příkazu `dnf`

Co se stane, když zkusíš odinstalovat balíček `dnf`?

```console
$ sudo dnf remove dnf
[sudo] heslo pro petr: 
Chyba: 
 Problém: Operace by vedla k odstranění následujících chráněných balíčků: dnf
```

Zkusíš vysvětlit, co tahle chyba znamená?

(Nápověda: kdyby tohle šlo, jak bys tuto operaci vrátil{{a}} zpět?)

Balíček `python3` taky nejde odinstalovat; proč?

## Odinstalování příkazu `python`

Příkaz `python` dělá to samé co příkaz `python3`:

```console
$ python --version
Python 3.9.7
$ ls -l /usr/bin/python
lrwxrwxrwx. 1 root root 9 31. srp 15.36 /usr/bin/python -> ./python3
$ ls -l /usr/bin/python3
lrwxrwxrwx. 1 root root 9 31. srp 15.05 /usr/bin/python3 -> python3.9
```

Můžeš odinstalovat balíček s příkazem `python` (bez 3 na konci)?
Bude pak zbytek systému fungovat?

## Doinstalování novějšího Pythonu

V Pythonu 3.9 se operátor `|` nedá použít na typy:

```console
$ python3
>>> str | int
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for |: 'type' and 'type'
```

Nainstaluj si Python verze 3.10, kde to jde:

```console
$ python3.10
>>> str | int
str | int
```


## Instalace `httpd`

Příště ho budeme potřebovat balíčky `httpd` a `systemd`.
Zajisti, abys je měl{{a}} nainstalované.
