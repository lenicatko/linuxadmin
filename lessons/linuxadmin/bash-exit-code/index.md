# Návratová hodnota příkazů

Tak jako funkce v Pythonu, i shellové příkazy mají návratovou hodnotu.
(V angličtine se používá *exit code* spíš než *return value*; v češtině budu
i tady používat termín známý z kurzu Pythonu.)

Návratová hodnota posledního příkazu je k dispozici ve speciální proměnné `$?`
(dolar otazník).
Je to vždy číslo.

> [note]
> Technicky vzato je jméno této proměnné jen otazník, ale používá se vždy
> společně s operátorem `$`. Nejde do ní přiřadit.

Návratová hodnota se většinou používá ke hlášení chyb
Pokud příkaz proběhl v pořádku, vrátí číslo `0`.
Pokud se vyskytne nějaká chyba, vrátí jiné číslo – podle toho,
jaká chyba vznikla.

Vyzkoušej si, že po úspěšném příkazu je v `$?` nula:

```
$ echo Ahoj
Ahoj
$ echo $?      # v `$?` je teď návratová hodnota příkazu `echo Ahoj`
0
$ ls
Dokumenty  Hudba  Obrázky  Plocha  Stažené  Šablony  Veřejné  Videa
$ echo $?      # v `$?` je teď návratová hodnota příkazu `ls`
0
$ echo $?      # v `$?` je teď návratová hodnota příkazu `echo $?`
0
```

Například `cat` vrací hodnotu 1, když se mu nepodaří otevřít soubor;
`bash` vrací hodnotu 127 když nenajde zadaný příkaz:

```console
$ cat neexistujici-soubor
cat: neexistujici-soubor: Adresář nebo soubor neexistuje
$ echo $?      # v `$?` je teď návratová hodnota příkazu `cat ...`
1
$ neexistujici-prikaz
bash: neexistujici-prikaz: command not found...
$ echo $?      # v `$?` je teď kód od Bashe
127
$ echo $?      # v `$?` je teď návratová hodnota příkazu `echo $?`
0
```

Nenulová návratová hodnota ovšem nutně neznamená že příkaz *selhal*.
Třeba hledací příkaz `grep` vrací 1 pokud nic nenajde:

```console
$ ls | grep Plocha
Plocha
$ echo $?
0
$ ls | grep Programy
$ echo $?
1
```

Při “opravdové” chybě použije jinou návratovou hodnotu, např.:

```console
$ grep nejaky-text neexistujici-soubor
grep: neexistujici-soubor: Adresář nebo soubor neexistuje
$ echo $?
2
```
