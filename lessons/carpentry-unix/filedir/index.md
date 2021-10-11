# Navigace v souborech a adresářích

Část operačního systému, která se stará o soubory a adresáře,
je *souborový systém*.
Organizuje data do souborů (angl. *files*), které obsahují informace,
a adresáře (též složky; angl. *directories*, *folders*), které obsahují
soubory nebo další adresáře.

Na vytváření, prohlížení, přesouvání a mazání souborů a adresářů
existuje několik příkazů, na které se teď podíváme.

Otevři si příkazovou řádku s Bashem, jestli ji už nemáš před sebou.

Nejdřív se podívej kde „jsi“, pomocí příkazu `pwd` (zkratka angl.
*print working directory*, vypiš pracovní adresář).
Adresáře jsou jako *místa* a vždycky, když používáš příkazovou řádku,
jsi právně na jednom z těchto míst: v *pracovním* neboli *aktuálním* adresáři.
Příkazy většinou pracují se soubory v aktuálním adresáři („tady“),
takže se hodí vždy vědět, kde to „tady“ je.

> [note]
> Aktuální adresář bývá vypsaný ve výzvě, před znakem `$` kterým se Bash ptá na
> příkaz.
> Ale ne všechny systémy jsou tak nastavené.
> A taky tenhle výpis bývá často různě zkrácený.
> Proto je dobré se na aktuální adresář umět zeptat.

```console
$ pwd
/home/nela
```

Tady se adresář jmenuje `/home/nela` – Nelin *domovský adresář*.

> [note]
> U tebe to bude <code>/home/<var>něco jiného</var></code> – podle tvého
> uživatelského jména.

Co to je ten domovský adresář?
Podívejme se zběžně na to, jak jsou adresáře v počítači organizovány.
Vypadá to zhruba nějak takhle:

```plain
─┬  /
 ├── bin
 ├── etc
 ├── home
 ╰── tmp
```

Na začátku je *kořenový adresář* (angl. *root directory*), který obsahuje
všechno ostatní.
Jmenuje se `/`, lomítko; to je to první lomítko v `/home/nela`.

V tomto adresáři je několik dalších adresářů: `bin` (kde jsou programy),
`etc` (kde je nastavení), `home` (kde jsou data uživatelů) nebo `tmp`
(pro dočasné soubory), a tak dále.

Nelin domovský adresář `/home/nela` je v adresáři `/home`, což je poznat
z toho, že začíná na `/home/`.
Stejně tak `/home` je v kořenovém adresáři, `/`.

Lomítko má tedy ve jménech adresářů dva různé významy: lomítko na začátku
označuje kořenový adresář; lomítko uprostřed odděluje jednotlivé adresáře.

> [note]
> Adresářů jménem `home` může být v počítači víc. Nela může mít třeba adresář
> `/home/nela/Dokumenty/home`, který je v adresáři `/home/nela/Dokumenty`.
> Když napíšu `/home` s lomítkem na začátku, dávám najevo že myslím ten
> `home`, který je přímo v kořenovém adresáři. Ten může být jen jeden.

Pod `home` je několik adresářů – jeden pro každého uživatele, který má
na tomto počítači účet.
Nela zrovna používá sdílený univerzitní systém, takže je tu víc:

```plain
─┬  /
 ├── bin
 ├── etc
 ├─┬ home
 │ ├── imhotep
 │ ├── matyas
 │ ╰── nela
 ╰── tmp
```

Uživatel *imhotep* má domovský adresář `/home/imhotep`,
Matyáš má `/home/matyas`.
A protože v našem příkladu počítač používá Nela, je domovský adresář
`/home/nela`.
Když zapneš Bash, typicky začneš ve svém domovském adresáři.
Proto je `/home/nela` teď i aktuální adresář – to, co vypíše `pwd`.

Teď se pomocí příkazu `ls` podívej co v tomhle adresáři je:

```console
$ ls
Dokumenty  Obrázky  Stažené  Veřejné
Hudba      Plocha   Šablony  Videa
```

Příkaz `ls` vypíše názvy všech souborů a adresářů v aktuálním adresáři.
Můžeš zařídit, aby ukázal víc informací: přidej *přepínač*
(angl. *option*, *switch*, nebo *flag*) `-F` (pomlčka F), který příkazu
`ls` řekne, aby k názvu přidal znak podle druhu souboru:

* `/` značí adresáře,
* `@` značí odkazy,
* `*` značí spustitelné soubory,
* bez symbolu jsou normální soubory.

```console
$ ls -F
Dokumenty/  Obrázky/  Stažené/  Veřejné/
Hudba/      Plocha/   Šablony/  Videa/
```

> [note]
> Přepínač od názvu příkazu musíš oddělit mezerou.
> A pozor na velikost písmen: stejně jako v Pythonu na ní záleží i v Bashi.
>
> Napíšeš tedy:
> <kbd>l</kbd> <kbd>s</kbd> <kbd>mezera</kbd> <kbd>pomlčka</kbd>
> <kbd>Shift</kbd>+<kbd>F</kbd> <kbd>↵ Enter</kbd>.
>

Ve Fedoře (v základním nastavení) se druh souboru dá určit i podle barvy,
kterou `ls` použije: adresáře jsou vypsány modře.
Barvy se ale z terminálu špatně kopírují a dál zpracovávají.

V našem příkladu je vidět, že Nelin domovský adresář obsahuje
pouze další adresáře.

> [note] Mazání terminálu
> Kdybys měl{{a}} na terminálu moc textu, můžeš použít
> <kbd>Ctrl</kbd>+<kbd>L</kbd> a obrazovku „vyčistit“.
> Pořád se můžeš vracet k předchozím příkazům pomocí <kbd>↑</kbd> a
> <kbd>↓</kbd>, nebo se i dostat k historii textu rolováním kolečkem myši.
> 
> Příkaz `clear` dělá něco podobného, ale smaže víc historie – rolování
> fungovat nebude, ale <kbd>↑</kbd>/<kbd>↓</kbd> ano.


## Syntax shellového příkazu

Teď zadej příkaz níže; rozebereme si ho:

```console
$ ls -F /
```

Ono `ls` na začátku je *příkaz*, kterému dáváš *přepínač* `-F` a *argument* `/`.

Přepínač `-F` jsme už viděli.
Přepínače obecně nějakým způsobem mění chování příkazu.
Jména přepínačů začínají jednou nebo dvěma pomlčkami (`-` nebo `--`).

Argumenty pak většinou říkají příkazu, s jakým souborem či adresářem má
pracovat.
V tomto případě vypisuješ obsah kořenového adresáře; všimni si, že jeden
z vypsaných adresářu je `home`.

Většině příkazů můžeš dát více přepínačů a více argumentů.

> [note]
> Přepínače a argumenty jsou ekvivalent pozičních, resp. pojmenovaných
> argumentů v Pythonu. Kdyby Bash uměl otevírat soubory jako Python,
> volání `open('soubor.txt', encoding='utf-8')` by se zapsalo zhruba jako
> `$ open --encoding=utf-8 soubor.txt`.

Jednotlivé části jsou oddělené mezerami: kdybys vynechal{{a}} mezeru mezi
`ls` a `-F`, Bash by hledal příkaz `ls-F`, který neexistuje.
Kdybys vynechal{{a}} mezeru mezi `-F`  a `/`, dostal by program `ls` přepínač
`-F/`, což taky nebude fungovat.

A záleží na velikosti písmen. Příkaz `LS` neexistuje.
Příkaz `ls -s` vypíše velikost souborů; `ls -S` soubory podle velikosti seřadí:

```console
$ ls -s /bin
celkem 271152
   64 [
   64 ab
    4 abrt
   36 ac
   24 aconnect
    4 activate-global-python-argcomplete
   40 addr2line
  124 afs5log
    4 alias
...
```

```console
$ ls -S /bin
podman
qemu-system-x86_64
qemu-system-i386
runc
cstool
gnome-control-center
...
```

Jsou to dva různé přepínače, které se dají zkombinovat:

```console
$ ls -s -S /bin
celkem 271152
56432 podman
13000 qemu-system-x86_64
12960 qemu-system-i386
 9960 runc
 4700 cstool
 4564 gnome-control-center
...
```


## Nápověda

Příkaz `ls` má spoustu dalších přepínačů.
Prakticky nikdo si je nepamatuje všechny.

Jsou dva časté způsoby, jak zjistit jak se ten který příkaz používá:

1. Většina příkazů vypíše nápovědu když dostane přepínač `--help`:

   ```console
   ls --help
   ```

2. Příkaz `man` ukáže manuál k danému příkazu:

   ```console
   man ls
   ```

### Přepínač `--help`

Spousta programů psaných pro shell zná přepínač `--help`, který vypíše
zhuštěné informace o tom, jaké přepínače příkaz umí zpracovat.
Zkontroluj si, že `ls` už pokryl skoro celou abecedu:

```console
$ ls --help
Použití: ls [PŘEPÍNAČ]… [SOUBOR]…
Vypisuje informace o SOUBORECH (implicitně z aktuálního adresáře). Jestliže
není zadán žádný z přepínačů -cftuvSUX nebo --sort, výstup bude seřazen
abecedně.

Povinné argumenty dlouhých přepínačů jsou také povinné u odpovídajících
krátkých přepínačů.
  -a, --all                  vypíše i soubory začínající tečkou
  -A, --almost-all           vypíše všechny soubory kromě souborů „.“ a „..“
      --author               spolu s -l vypíše autora každého souboru
  -b, --escape               negrafické znaky escapuje ve stylu jazyka C
      --block-size=VELIKOST  spolu s -l vypisuje velikosti v násobcích VELIKOSTI,
                             např. „--block-size=M“; popis formátu VELIKOSTI
                             je uveden níže
  -B, --ignore-backups       nevypisuje soubory končící na ~
  -c                         s -lt: řadí podle ctime a vypisuje ctime (čas
                             poslední změny i-uzlových informací);
                             s -l: vypisuje ctime, řadí podle názvu souboru;
                             jinak: řadí podle ctime, vypisuje od nejnovějších
  -C                         vypisuje položky ve sloupcích
      --color[=KDY]          obarví výstup; KDY smí být „always“ (vždy,
                             výchozí hodnota), „auto“ nebo „never“ (nikdy),
                             podrobnosti níže
  -d, --directory            vypíše názvy adresářů místo jejich obsahu
  -D, --dired                generuje výstup formátovaný pro Emacsový
                             mód „dired“
...
```

> [note] Nepodporované přepínače
> Když zadáš přepínač, který daný příkaz nezná, dostaneš většinou chybovou
> hlášku, která tě navede na `--help`:
> ```console
> $ ls -j
> ```

### Příkaz `man`

Druhá možnost jak se něco dozvědět o příkazu `ls` je:

```console
$ man ls
```

To z terminálu udělá „čtečku“ (*pager*) s podrobnějším popisem příkazu a
jeho přepínačů.

K navigaci v dokumentu můžeš použít šipky <kbd>↑</kbd> a <kbd>↓</kbd>,
případně <kbd>mezerník</kbd> či <kbd>PgUp</kbd>/<kbd>PgDn</kbd> na skákání
po stránkách.
Můžeš tu i hledat: zmáčkni <kbd>/</kbd>, hledané slovo a <kbd>Enter</kbd>.
Najde-li se víc výsledků, můžeš mezi nimi přepínat pomocí <kbd>n</kbd>
a <kbd>N</kbd> (tedy <kbd>n</kbd> a <kbd>Shift</kbd>+<kbd>n</kbd>).

Na **zavření** manuálové stránky použij klávesu <kbd>Q</kbd>.

> [note] Anglická nápověda
> Občas narazíš na nápovědu nebo manuálnovou stránku napsanou v češtině.
> O překlad se starají dobrovolníci: nemusí být vždy úplně aktuální
> a někdy o neaktuálnosti najdeš i varování.
>
> Základní funkčnost příkazů jako `ls` se nemění desítky let,
> takže neaktuálnost často zas tolik nevadí. Kdybys ale chtěl{{a}} oficiální
> nápovědu v angličtině, napiš na začátek příkazu `LANG=en_US ` (bez mezer kolem
> rovnítka, s mezerou za `en_US`):
>
> ```bash
> $ LANG=en man ls
> ```


### Web

Třetí možnost, jak dostat informace o příkazu, je samozřejmě Internet.
Když k hledanému výrazu připíšeš `unix command` nebo `unix man page`,
dostaneš relevantnější výsledky.

Nevěříš-li často náhodným výsledkům vyhledávačů, můžeš kouknout na
stránky projektu GNU, kde najdeš oficiální zevrubnou dokumentaci
k [základním příkazům][manuals-coreutils]
i [ostatním GNU projektům][manuals-gnu].

[manuals-coreutils]: https://www.gnu.org/software/coreutils/manual/coreutils.html
[manuals-gnu]: https://www.gnu.org/manual/manual.html


## Další přepínače pro `ls`

Co dělá přepínač `-l`?
Co se stane, když `-l` zkombinuješ s `-h`?

Už víš kde najít nápovědu, tak zkus zjistit odpověď.

{% filter solution %}
Přepínač `-l` zapne „dlouhý“ formát, kde je vidět spousta dalších informací
o souborech: velikost, datum poslední změny, nebo vlastníka.
Když přidáš `-h` (tedy: `ls -h -l`), velikost souboru se zobrazí ve formátu
„pro lidi“, zaokrouhleně — např. místo `5369` se vypíše `5.3K`.
{% endfilter %}

Příkaz `ls` normálě řadí soubory podle abecedy (A-Z).
Příkaz `ls -t` je seřadí podle data poslední změny.
Příkaz `ls -r` je seřadí podle abecedy, ale opacně (Z-A)

Tipneš si, co dělá příkaz `ls -t -r`?

{% filter solution %}
`ls -t -r` seřadí soubory podle data poslední změny, od nejnovějšího.
To se může hodit až budeš hledat kde jsi naposledy udělal{{a}} změnu,
nebo jestli se objevil nový výstupní soubor.
{% endfilter %}


## Kombinování přepínačů

Jednopísmenné přepínače, které začínají jednou pomlčkou, se dají kombinovat
dohromady:
* `ls -h -l` můžeš zkrátit na `ls -hl`,
* `ls -t -r -l` na `ls -trl`.

U „dlouhých“ přepínačů jako `--help` to nefunguje.

> [note]
> Takhle funguje naprostá většina příkazů, ale některé – často starší nebo
> nějak specializované – používají jiné konvence.


## Obsah dalších adresářů

Příkaz `ls` umí zobrazit obsah jakéhokoli adresáře, ne jen toho aktuálního.
Podívej se teď do svého adresáře `Dokumenty`, a to s přepínačem `-F`, který
označí druhy souborů.

```console
$ ls -F Dokumenty
data-shell.zip   data-shell/
```

Měl{{a}} bys vidět všechny soubory ve svých Dokumentech: archiv, který jsi
před chvílí stáhl{{a}} a adresář, kam se rozbalil.
A možná i nějaké jiné.

Na adresář `data-shell`, který je v Dokumentech, se můžeš podívat několika způsoby.

První je zadat *cestu* k němu, tedy `Dokumenty/data-shell`.

```console
$ ls -F Dokumenty/data-shell
creatures/          molecules/          notes.txt           solar.pdf
data/               north-pacific-gyre/ pizza.cfg           writing/
```

Příkaz `ls` se podíval do adresáře `Dokumenty` a v něm pak do adresáře
`data-shell`.

> [note]
> Cesta `Dokumenty/data-shell` nezačíná lomítkem, vychází tedy z aktuálního
> adresáře. Můžeš použít „celou“ cestu, která lomítkem začíná,
> ale v té musíš uvést i svůj domovsý adresář. Například:
> 
> ```console
> $ ls -F /home/nela/Dokumenty/data-shell`
> ```

Další způsob je tou cestou projít pomocí příkazu `cd`, který změní aktuální
adresář (z angl. **c**hange **d**irectory).
(To jméno je trošku zavádějící: v adresáři se nic nezmění,
jen příkazová řádka začne pracovat v jiném adresáři.
Jako bys na ten adresář klikl{{a}} v grafickém prohlížeči souborů.)

```console
$ cd Dokumenty
$ cd data-shell
$ cd data
```

Každý příkaz `cd` změní tvůj aktuální adresář, takže celá tahle série příkazů
tě postupně provede přes `Dokumenty` a `Dokumenty/data-shell` až do
`Dokumenty/data-shell/data`.

Všimni si že samotný příkaz `cd ` nic nevypisuje.
To je normální.
Když se něco nepovede, dostaneš chybovou hlášku; když je všechno v pořádku,
není potřeba tě otravovat.

Aktuální adresář by se ti měl (ve zkrácené podobě) ukázat ve výzvě (před `$`),
ale kdyby to nestačilo, můžeš se vždycky zeptat kde jsi:

```console
$ pwd
/home/nela/Dokumenty/data-shell/data
```

Příkaz `ls -F` teď bude ukazovat obsah nového aktuálního adresáře:

```console
$ ls -F
amino-acids.txt   elements/     pdb/	        salmon.txt
animals.txt       morse.txt     planets.txt     sunspot.txt
```

Když teď zkusíš „jít“ do adresáře `data-shell`, nepůjde to.
V aktuálním adresáři žádný `data-shell` neexistuje:

```console
$ cd data-shell
-bash: cd: data-shell: Adresář nebo soubor neexistuje
```

Na disku samozřejmě nějaký adresář `data-shell` máš, a dokonce docela „blízko“,
ale `cd` – a ostatní příkazy shellu – hledají pouze v aktuálním adresáři,
a nikde jinde.

Aby ses dostal{{a}} do „nadřazeného“ adresáře, použij dvě tečky, `..`:

```console
$ cd ..
```

Dvě tečky jsou vždy jméno *nadřazeného* adresáře (angl. *parent directory*),
tedy adresáře, který obsahuje ten aktuální.
Můžeš si to ověřit pomocí `pwd`:

```console
$ pwd
/home/nela/Dokumenty/data-shell/
```

Speciální adresář `..` se normálně neukazuje, ale můžeš ho zobrazit pomocí
přepínače `-a` (nebo ekvivalentního `--all`):

```console
$ ls -aF
./   .bash_profile  data/       north-pacific-gyre/  pizza.cfg  thesis/
../  creatures/     molecules/  notes.txt            solar.pdf  writing/
```

Vida! objevilo se několik položek, které předtím byly skryté:

* `..`, tedy nadřazený adresář (zde tedy `/home/nela/Dokumenty`)
* `.`, což je jméno pro *aktuální* adresář (zde tedy `/home/nela/Dokumenty/data-shell`)
* `.bash-profile`, což je *skrytý soubor*. Všechny soubory, jejichž jméno
  začíná tečkou, nástroje jako `ls` normálně neukazují.
  (Znáš-li Git, vzpomeneš si možná, že tečkou začíná adresář `.git` a
  soubor `.gitignore`.)

Adresáře `.` a `..` nejsou jen nějaká zkratka příkazu `cd`: jsou to opravdová
jména příslušných adresářů.
Můžeš je použít kdykoli, když pojmenováváš adresář, dokonce i jako součást
cesty. Zkus si:

```console
$ ls ..
$ ls creatures/../data/
```

> [note]
> Teď je doufám vidět, proč se pojmenování adresáře (nebo souboru)
> s použitím podadresářů se říká *cesta* (angl. *path*): cesta říká, jak se
> „dostat“ na nějaké místo.
> Příklad `creatures/../data/` to krásně ukazuje: říká že se z aktuálního
> adresáře (`data-shell`) “vydáš” do adresáře `creatures`, tam se ale obrátíš
> a jdeš zpátky do `data-shell`, odkud zamíříš do `data`.
>
> Skončíš na stejném místě, kam vede i kratší cesta `data`.

Tak, teď víš, jak se orientovat v souborovém systému pomocí příkazů `cd`,
`ls` a `pwd`.
Pojďme se ještě podívat na několik variací.
Co se stane, když napíšeš samotné `cd`?

```console
$ cd
```

Jestli to není jasné z výzvy, zkontroluj kde jsi pomocí `pwd`.
Dostaneš se do svého domovského adresáře!

Příkazy shellu mají *výchozí chování*, které nastane když jim nepředáš
argument.
Příkaz `ls` vypíše aktuální adresář; chová se tedy stejně jako `ls .`.
Kdyby se ale `cd` choval jako `cd .`, nic by se nestalo.
A tak tvůrci tohoto příkazu zvolili lepší výchozí chování: samotné `cd` přejde
do domovského adresáře.

> [note]
> Občase se stane, že samotné `cd` zadáš nechtěně.
> Pak se ti bude hodit další zkratka: příkaz `cd -` (`cd` pomlčka) tě vrátí do
> naposledy navštíveného adresáře.
>
> Některé příkazy zapnou nějakou speciální funkci, když jim na místě
> přepínače nebo argumentu zadáš samotnou pomlčku. Detaily najdeš vždy
> v nápovědě daného příkazu.
> Některé příkazy (jako `ls -`) si ale pomlčku vyloží jako normální argument.

Jsi-li teď v domovském adresáři, zkus `cd` dát celou cestu – místo tří
příkazů `cd` použij jeden, s jmény jednotlivých adresářů oddělenými `/`:

```console
$ cd Dokumenty/data-shell/data
```

A pomocí `pwd` zkontroluj, že jsi na stejném místě jako po sekvenci
`cd Dokumenty`; `cd data-shell`; `cd data`, tedy:

```console
$ pwd
/home/nela/Dokumenty/data-shell/data
```


## Relativní a absolutní cesty

Do domovského adresáře se teď můžeš dostat pomocí několika způsobů:

* `cd` bez argumentu
* třikrát `cd ..`
* `cd ../../..`

Další způsob je použití *absolutní cesty* (angl. *absolute path*).
Je to *celé* jméno adresář, včetně počátečního lomítka:

```console
$ cd /home/nela
```

Absolutní cesta označuje daný adresář, ať už jsi kdekoli.
Některé příklady absolutních cest jsou:

* `/home/nela`
* `/home/nela/Dokumenty`
* `/home/nela/Dokumenty/data-shell/data/../creatures`

Opak absolutní cesty je *relativní cesta*, která vždy „začíná“ v aktuálním
adresáři.
Jakákoli cesta bez počátečního lomítka je relativní:

* `Dokumenty`
* `Dokumenty/data-shell/creatures`
* `..`
* `../../../`

Ať už je cesta relativní nebo absolutní, lomítko *na konci* značí,
že jde o adresář.
Tohle koncové lomítko můžeš vynechat.


## Vlnovková zkratka

Shell dává speciální význam znaku `~` (vlnovka).
Když nějaký argument začíná `~/` (nebo je jen `~`), shell místo něj doplní tvůj
domovský adresář (a vytvoří tak absolutní cestu).
Můžeš tedy psát:

* `ls ~` místo `ls /home/nela`
* `ls ~/Dokumenty/data-shell` místo `ls /home/nela/Dokumenty/data-shell`
* `cd ~` místo `cd`

> [note]
> Toto je zkratka *shellu*: vlnovka není opravdové jméno adresáře.
> Nemůžeš např. v Pythonu použít `open('~/data.txt')`.
> Oproti tomu `open('../data.txt')` fungovat bude, protože `..` a `.` jsou
> jména adresářů se vším všudy.
>
> Shell za `~` doplňuje i když argument není jméno souboru.
> Například příkaz `echo` svůj argument vypíše;
> vyzkoušej si, do dělá `echo Ahoj!` a `echo ~/Ahoj!`


(XXX: Další cvičení – viz [SC](http://swcarpentry.github.io/shell-novice/02-filedir/index.html#absolute-vs-relative-paths))


## Nela a medúzy – Doplňování

Nela teď ví spoustu nových věcí o adresářích a chce si zorganizovat
soubory, které bude vytvářet.
Vytvořila si proto adresář `north-pacific-gyre` (severní tichomořský vír)
aby věděla, odkud data jsou.
Uvnitř si vytvořila adresář `2012-07-03` – datum, kdy začala zpracovávat.
Kdysi používala jména jako `data` a `opravene-vysledky`, ale takové názvy
po pár letech pestaly dávat smysl. (Poslední kapka byl tenkrát adresář
`opravene-opravene-vysledky-3`. Git ještě Nela nezná.)

> [notes] Řazení kalendáře
> Nela pojmenovává adresáře ve tvaru <var>rok</var>-<var>měsíc</var>-<var>den</var>
> s dvouciferným měsícem a dnem (tj. s případnou nulou na začátku).
> Když takové názvy seřadí podle “abecedy” (např. pomocí `ls`), seřadí se
> zároveň podle data.

Soubor pro každý vzorek má pojmenovaný podle konvence své laboratoře,
desetipísmenným identifikátorem jako `NENE01729A`.
Pod tímhle jménem si uložila místo, hloubku a další vlastnosti vzorku,
a tak stejné jméno použije i pro jméno souboru.
Protože jeji přístroje zapisují textové soubory, ukládá je jako
`NENE01729A.txt`, `NENE01812A.txt`, atd.
Všech 1520 souborů přijde do stejného adresáře.

Teď, když je v adresáři `data-shell`, se na soubory podívá příkazem:

```console
$ ls -F north-pacific-gyre/2012-07-03
```

To je docela dost písmenek, ale může si to ulehčit: napíše jen:

```console
$ ls -F nor
```

… a zmáčkne <kbd>Tab</kbd>. Shell automaticky doplní jméno adresáře, který
začíná na `nor`:

```console
$ ls -F north-pacific-gyre/
```

A pak zmáčkne <kbd>Tab</kbd>. Shell automaticky doplní jméno jediného adresáře
– `2012-07-03/`.
(Kdyby tu byl ještě, řekněme, `2019-09-12`, shell by doplnil jen `201` a další
písmenko by nechal Nelu dopsat.)

Výsledek je:

```console
$ ls -F north-pacific-gyre/2012-07-03/
```

Další <kbd>Tab</kbd> už neudělá nic.
Je tu 19 možností, které nezačínají všechny stejně.
Když ale Nela zmáčkne <kbd>Tab</kbd> podruhé, všechny tyto možnosti se vypíšou
a může si z nich vybrat – dopsat další písmenko a znovu zmáčknout <kbd>Tab</kbd>.

Tomuhle se říká *doplňování taulátorem* (angl. *tab completion*) a velice
to zjednodušuje práci se shellem.


## Další triky: Kopírování

Když už jsme u zkratek, pojďme se podívat ještě na jednu: vkládání.

Známá kombinace <kbd>Ctrl</kbd>+<kbd>C</kbd>, <kbd>Ctrl</kbd>+<kbd>V</kbd>
v příkazové řádce *nefunguje*.
Už v době, kdy ještě neexistovala grafická okýnka, se zkratka
<kbd>Ctrl</kbd>+<kbd>C</kbd> používala na *ukončení běžícího programu*.
A když se shelly přesunuly do grafických okýnek, bylo pozdě na to, aby se
zkratka předefinovala.

Proto do terminálu musíš vkládat pomocí
<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>V</kbd> a kopírovat z něj
pomocí <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>C</kbd>.
Případně kliknout pravým tlačítkem a vybrat *Kopírovat* nebo *Vložit*.

To je docela otrava, ale většinou to nevadí, protože v Linuxu se dá
kopírovat i snadnějším způsobem: kdykoli myší *označíš* nějaký text,
zkopíruje se do zvláštní schránky, kterou pak můžeš vložit pomocí
*prostředního tlačítka myši*.
Když si na to zvykneš, není cesty zpět!
(Bohužel to ale nefunguje u některých laptopových klávesnic – pak potřebuješ
opravdovou myš.)

Občas můžeš dokonce využít toho, že máš k dispozici dvě schránky – jednu
s výběrem, druhou s textem zkopírovaným
<kbd>Ctrl</kbd>(+<kbd>Shift</kbd>)+<kbd>C</kbd>.

> [note]
> Na macOS a klávesnicích od Apple kombinace jako <kbd>Command</kbd>+<kbd>C</kbd>
> atd. fungují – tyhle klávesnice mají příkazovou řádku rády.
> I tady ale prostřední tlačítko zrychluje práci.
