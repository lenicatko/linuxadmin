# Úkol: Vlastní příkaz

Níže je funkce v Pythonu, která bere řetězec a vrátí "obrácený" řetězec: znaky jsou v něm pozpátku a nahrazené podle slovníku.

Tvůj úkol je napsat příkaz pro příkazovou řádku, který bude radost použít v Bashi.
Detaily níže.

```python
obracena_pismena = {
    'a': 'ɐ', 'b': 'q', 'c': 'ɔ', 'd': 'p', 'e': 'ǝ', 'f': 'ɟ', 'g': 'ƃ',
    'h': 'ɥ', 'i': 'ᴉ', 'j': 'ɾ', 'k': 'ʞ', 'l': 'l', 'm': 'ɯ', 'n': 'u',
    'o': 'o', 'p': 'd', 'q': 'b', 'r': 'ɹ', 's': 's', 't': 'ʇ', 'u': 'n',
    'v': 'ʌ', 'w': 'ʍ', 'x': 'x', 'y': 'ʎ', 'z': 'z', 'A': '∀', 'B': 'B',
    'C': 'Ɔ', 'D': 'D', 'E': 'Ǝ', 'F': 'Ⅎ', 'G': 'פ', 'H': 'H', 'I': 'I',
    'J': 'ſ', 'K': 'ʞ', 'L': '˥', 'M': 'W', 'N': 'N', 'O': 'O', 'P': 'Ԁ',
    'Q': 'Q', 'R': 'R', 'S': 'S', 'T': '┴', 'U': '∩', 'V': 'Λ', 'W': 'M',
    'X': 'X', 'Y': '⅄', 'Z': 'Z', '0': '0', '1': 'Ɩ', '2': 'ᄅ', '3': 'Ɛ',
    '4': 'ㄣ', '5': 'ϛ', '6': '9', '7': 'ㄥ', '8': '8', '9': '6', ',': "'",
    '.': '˙', '?': '¿', '!': '¡', '"': '„', "'": ',', '`': ',', '(': ')',
    ')': '(', '[': ']', ']': '[', '{': '}', '}': '{', '<': '>', '>': '<',
    '&': '⅋', '_': '‾',
}

def obrat(text):
    return ''.join(obracena_pismena.get(c, c) for c in reversed(text))
```

Příklad použití v Pythonu:

```pycon
>>> obrat("Ahoj, PyLadies!")
"¡sǝᴉpɐ˥ʎԀ 'ɾoɥ∀"
```

Program pro příkazovou řádku by měl být spustitelný (zařiď pomocí `chmod`).
Po zkopírování do správného adresáře by měl jít spustit pomocí příkazu `obrat`.

Když nedostane žádný argument, obrátí standardní vstup:

```console
$ echo Ahoj | obrat
ɾoɥ∀
```

Jednotlivé řádky by měl program zpracovávat samostatně:

```console
$ echo 'Ahoj,' > pozdrav.txt
$ echo 'PyLadies' >> pozdrav.txt
$ cat pozdrav.txt
Ahoj,
PyLadies
$ cat pozdrav.txt | obrat
'ɾoɥ∀
¡sǝᴉpɐ˥ʎԀ
```

Pokud `obrat` dostane argumenty, zpracuje dané soubory:

```console
$ obrat pozdrav.txt
'ɾoɥ∀
¡sǝᴉpɐ˥ʎԀ
$ obrat pozdrav.txt pozdrav.txt
'ɾoɥ∀
¡sǝᴉpɐ˥ʎԀ
'ɾoɥ∀
¡sǝᴉpɐ˥ʎԀ
```

Argument `-` znamená standardní vstup (tedy ne soubor jménem `-`).

```console
$ echo Ahoj | obrat -
ɾoɥ∀
$ echo haha | obrat pozdrav.txt - pozdrav.txt
'ɾoɥ∀
¡sǝᴉpɐ˥ʎԀ
ɐɥɐɥ
'ɾoɥ∀
¡sǝᴉpɐ˥ʎԀ
```

Pokud program dostane přepínač `--help`, vypíše krátkou nápovědu a ignoruje ostatní argumenty.

Je-li použit jiný přepínač (začínající `-`) a ne `--help`, program uživateli vynadá (na chybovém výstupu), vrátí chybovou návratovou hodnotu (`exit(1)`) a ignoruje ostatní argumenty.

```console
$ obrat -x
Přepínač -x neexistuje, viz --help
$ obrat --cislovani-radku > vystup.txt
Přepínač --cislovani-radku neexistuje, viz --help
$ echo $?
1
```

Nakonec program změň tak, aby když některý znak chybí ve slovníku, vracel chybovou návratovou hodnotu `2`. Pořád ale zpracuje celý vstup.

```console
$ echo Ahoj | obrat
ɾoɥ∀
$ echo $?
0
$ echo Čau | obrat
nɐČ
$ echo $?
2
```

### Nápověda k Pythonu

Naimportuješ-li `sys` a `os`, pak:

* `sys.argv` je seznam argumentů (včetně jména programu)
* `sys.stdin` je *už otevřený* soubor se std. vstupem (netřeba `with` či `close`)
* Podobně `sys.stdout` je soubor se standardním výstupem (tam píše `print`) a `sys.stderr` je soubor chybovým výstupem.
* `os.environ` je slovník* s proměnnýma prostředí
* `exit(1)` ukončí program s danou hodnotou

(* přesněji řečeno, objekt který se chová jako slovník)

> [note]
> Na zpracování argumentů existují knihovny jako `argparse` (součást Pythonu),
> nebo `click` (kterou je potřeba doinstalovat, ale je jednodušší na použití a
> má spoustu vychytávek navíc).
> Ažbudeš psát „opravdový“ program pro příkazovou řádku, nastuduj si jak
> se používají.
>
> Přímo na seznam `sys.argv` se dívají jen ty nejjednodušší skripty,
> programátoři kteří se chtějí naučit jak to funguje (ty!), a taky autoři
> knihoven jako `click`.
