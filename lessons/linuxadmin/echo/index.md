# Echo

Další z velice jednoduchých nástrojů, které při práci v příkazové řádce
využiješ, je `echo`.
Tento příkaz vypíše všechny argumenty, které dostane.

```console
$ echo haló haló
haló haló
```

K čemu je to dobré? Můžeš si takto otestovat, jestli operace které se chystáš
provést na souborech dělají přesně to, co chceš.
Bash může jednotlivé argumenty trochu měnit, takže konkrétní program dostává
trochu jiný seznam argumentů než to, co napíšeš.

Následující program říká Bashi: zavolej program `echo` a dej mu tři argumenty:
`ahoj`, `halo` a `halooo`.
Všimni si, že mezery zde nehrají roli: oddělují jednotlivé argumenty,
které Bash předá dál.
Příkaz `echo` tak dostane jen ty tři slova a vypíše je oddělená mezerami.

```console
$ echo ahoj      halo halooo
ahoj halo haloo
```

V následujícím příkazu `echo` dostane celkem 7 argumentů: `ahoj` a
6 jmen souborů, které Bash vytvoří z `*.pdb`.

```console
$ echo ahoj *.pdb
ahoj cubane.pdb ethane.pdb methane.pdb octane.pdb pentane.pdb propane.pdb
```

Pokud potřebuješ zachovat řetězec i včetně mezer, hvězdiček a dalších znaků,
můžeš použít uvozovky.
Bash bude všechno co je uvnitř jednoduchých uvozovek (`'`) brát jako
jeden argument.

```console
$ echo 'ahoj     *.pdb'
ahoj     *.pdb
```

Doporučuji používat uvozovky vždy, když si nejseš jist{{gnd('ý', 'á')}},
jestli něco Bash nebude považovat za speciální znak:

```console
$ echo '<o.o> je robot #1! :)'
<o.o> je robot #1! :)
```

> [note] Rozdíl mezi `cat` a `echo`
> Tyhle dva příkazy si lidi občas pletou.
>
> `cat` vypisuje *obsah* souboru, jehož jméno dostane jako argument.
> Takový soubor musí existovat.
> Příkaz `echo` vypisuje přímo argumenty které dostane – ať už to jsou jména
> souborů nebo ne:
>
> ```console
> echo soubor.txt
> soubor.txt
> ```

# Dvojité uvozovky

Na rozdíl od Pythonu je v Bashi rozdíl mezi jednoduchými a dvojitými uvozovkami.
Uvnitř dvojitých bude sice Bash zachovávat mezery, ale některé znaky
(jako `$` a <code>`</code>) si speciální význam zachovají.

Zkus si tento příkaz se speciálním znakem `$`:

```console
$ echo $0
bash
```

Místo `$0` Bash doplní jméno aktuálního shellu: `bash`, nebo na některých
systémech `/bin/bash`.
Samotné `echo` nic nepřevádí, pouze vypisuje přesně to, co dostává jako argument.
To Bash nahradí `$0` konkrétním řetězcem.

Ve dvojitých uvozovkách Bash nezpracovává hvězdičku, ale `$0` funguje dál:

```console
$ echo "*"
*
$ echo "$0"
bash
```

Uvnitř jednoduchých ale je speciální znak jen jeden: ukončující `'`.

```bash
$ echo '$0'
$0
$ echo '*'
*
```

Dokud neznáš pravidla o tom, které znaky jsou v `""` speciální,
používej radši jednoduché uvozovky.


## Echo do souboru

`echo` je další způsob, jak vytvářet soubory.
Znáš už `touch` a `nano` (případně jakýkoli jiný textový editor);
teď k nim přidej přesměrování z `echo`:

```console
$ echo haló haló > basnicka.txt
$ cat basnicka.txt
haló haló
```

Přesměrováním se ale původní obsah souboru smaže, takže když budeš chtít dopsat
do básničky další řádek, nepůjde to:

```console
$ echo 'co se stalo?' > basnicka.txt
$ cat basnicka.txt
co se stalo?
```

Pro přidání textu do souboru můžeš použit další druh přesměrování: `>>`.
S ním původní obsah v souboru zůstane a nový text bude přidán na konec:

```console
$ echo haló haló > basnicka.txt
$ cat basnicka.txt
haló haló
$ echo 'co se stalo?' >> basnicka.txt
$ cat basnicka.txt
haló haló
co se stalo?
```

Jiný způsob jak napsat něco do souboru je přesměrovat výstup z `cat`.
Text pak napíšeš přímo do příkazové řádky:

```console
$ cat > basnicka.txt  # (následující řádky napiš ty!)
ahoj ahoj
jak se máš?
$ cat basnicka.txt    # (následující řádky vypíše cat)
ahoj ahoj
jak se máš?
```

Další způsob jak uložit text do souboru je použít `echo` a jednoduché uvozovky.
Bash ti v rámci uvozovek dokonce umožní napsat víc řádků.
Na konci je potřeba řetězec uzavřít a přesměrovat do souboru.

```
$ echo 'halo halo
> co se stalo
> ...' > basnicka.txt
$ cat basnicka.txt 
halo halo
co se stalo
...
```

> [note]
> Když zadáváš víceřádkový příkaz, změní se výzva (prompt): místo `$`
> je poslední znak většinou `>`.
>
> Znamená to že příkaz ještě není ukončený.
> Kdyby se ti ho nedařilo zavřít (zde jednoduchou uvozovkou),
> zmáčkni <kbd>Ctrl</kbd>+<kbd>C</kbd> a začni znovu.

{# XXX cover stuff like >>> somewhere #}
