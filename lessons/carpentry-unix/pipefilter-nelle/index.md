# Nela a medúzy: Kontrola souborů

Zapni terminál a přepni se do adresáře `data-shell/north-pacific-gyre/2012-07-03`.

Pomocí `ls` se podívej, co v adresáři je:

```console
$ ls
goodiff         NENE01736A.txt  NENE01843A.txt  NENE01978B.txt  NENE02040Z.txt
goostats        NENE01751A.txt  NENE01843B.txt  NENE02018B.txt  NENE02043A.txt
NENE01729A.txt  NENE01751B.txt  NENE01971Z.txt  NENE02040A.txt  NENE02043B.txt
NENE01729B.txt  NENE01812A.txt  NENE01978A.txt  NENE02040B.txt
```

Úkol: Kolik mají řádků datové soubory s `.txt` na konci?

{% filter solution %}
```console
$ wc -l *.txt
```
{% endfilter %}

Všechny by měly mít 300 řádků. Když se ale podíváš na výstup, zjistíš, že tomu tak není.
Představ si, že máš těchto souborů k ověření několik tisíc a procházení *očima* nebude možné.
Jak zjistíš, jestli všechny tyto soubory mají opravdu 300 řádků?

{% filter solution %}

`wc -l *.txt | sort | head` - uvidíš, jestli soubory na začátku jsou OK
`wc -l *.txt | sort | tail` - uvidíš, jestli soubory na konci jsou OK

Podívej se na výstup prvního z těch příkazů:

```
  240 NENE02018B.txt
  300 NENE01729A.txt
  300 NENE01729B.txt
  300 NENE01736A.txt
  300 NENE01751A.txt
  300 NENE01751B.txt
  300 NENE01812A.txt
  300 NENE01843A.txt
  300 NENE01843B.txt
  300 NENE01971Z.txt
```

{% endfilter %}

Soubory končí většinou na písmenko `A` nebo `B,` ale je zde jeden končící `Z`. Spočítej, kolik je souborů končících `Z`.

{% filter solution %}
```console
$ ls *Z.txt | wc -l
2
```
{% endfilter %}

Už podle názvů tyhle soubory nevypadají na to, že by je někdo vytvořil ručně.
Můžeš bezpečně předpokládat, že jsou výstupem nějakého programu na zpracování vzorků.

Podívej se do několika z nich (např. pomocí příkazu `less`), abys zjistil{{a}} jak zhruba vypadají informace v nich obsažené.

Obsahem je relativní výskyt 300 bílkovin v různých hloubkách oceánu.
`A` a `B` jsou dvě hloubky, na nichž bylo prováděno měření; `Z` znamená, že se hloubku z nějakého důvodu nepodařilo zjistit.
Kratší soubor je způsobený restarem počítače během měření.
Nevíme, zda takový soubor bude použitelný pro další zpracování.


## Hranaté závorky: Filtrování písmen

Už znáš hvězdičku a otazník, díky nimž můžeš vytvořit masky souborů.
Existují i další „finty“, jak jména souborů filtrovat. Jednu z nich si teď ukážeme.
Když dáš do hranatých závorek výčet znaků, které chceš hledat, např. `[AB]`, celý výraz v závorkách se nahradí za jedno písmenko – buďto `A` nebo `B`.
Můžeš tak odfiltrovat soubory, které končí na `Z`:

```console
$ ls *[AB].txt
NENE01729A.txt  NENE01751A.txt  NENE01843A.txt  NENE01978B.txt  NENE02040B.txt
NENE01729B.txt  NENE01751B.txt  NENE01843B.txt  NENE02018B.txt  NENE02043A.txt
NENE01736A.txt  NENE01812A.txt  NENE01978A.txt  NENE02040A.txt  NENE02043B.txt
```
V tomto případě dostaneš stejný výstup, když napíšeš příkaz:

```console
$ ls *A.txt *B.txt
```

Výstup se ale bude lišit v případě, kdy neexistuje soubor kterému taková maska
odpovídá.
Představ si, že chceš vypsat všechny soubory, které mají na konci `A`, `B` nebo `C`:

```console
$ ls *[ABC].txt
NENE01729A.txt  NENE01751A.txt  NENE01843A.txt  NENE01978B.txt  NENE02040B.txt
NENE01729B.txt  NENE01751B.txt  NENE01843B.txt  NENE02018B.txt  NENE02043A.txt
NENE01736A.txt  NENE01812A.txt  NENE01978A.txt  NENE02040A.txt  NENE02043B.txt

$ ls *A.txt *B.txt *C.txt
ls: cannot access '*C.txt': No such file or directory
 NENE01729A.txt   NENE01751B.txt   NENE01978A.txt   NENE02040B.txt
 NENE01729B.txt   NENE01812A.txt   NENE01978B.txt   NENE02043A.txt
 NENE01736A.txt   NENE01843A.txt   NENE02018B.txt   NENE02043B.txt
 NENE01751A.txt   NENE01843B.txt   NENE02040A.txt
```

Bash nechá `*C.txt` beze změny, protože takový soubor nenajde.
Program `ls` si pak postěžuje že soubor `*C.txt` nezná.

Zkus si, co v těchto případech dostává příkaz `ls` od Bashe.
To můžeš zjistit tak, že místo `ls` zadáš `echo`:

```console
$ echo *[ABC].txt
$ echo *A.txt *B.txt *C.txt
```
