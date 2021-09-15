# Navigace v souborech a adresářích

Část operačního systému, která se stará o soubory a adresáře,
je *souborový systém*.
Organizuje data do souborů (angl. *files*), které obsahují informace,
a adresáře (též složky; angl. *directories*, *folders*), které obsahují
soubory nebo další adresáře.

Na vytváření, prohlížení, přesouvání a mazání souborů a adresářů
existuje několik příkazů, na které se teď podíváme.
Otevři si příkazovou řádku s Bashem.

Nejdřív se podívej, kde „jsi“, pomocí příkazu `pwd` (zkratka angl.
*print working directory*, vypiš pracovní adresář).
Adresáře jsou jako *místa* a vždycky, když používáš příkazovou řádku,
jsi právně na jednom z těchto míst: v *pracovním* neboli *aktuálním* adresáři.
Příkazy většinou pracují se soubory v aktuálním adresáři („tady“),
takže se hodí vždy vědět, kde jsi.

> [note]
> Aktuální adresář bývá vypsaný ve výzvě, před znakem `$` kterým se Bash ptá na
> příkaz, ale ne všechny systémy jsou tak nastavené.
> Je dobré se na to Bashe umět zeptat.

```console
$ pwd
/home/nela
```

Tady se adresář jmenuje `/home/nela` – Nelin *domovský adresář*.

> [note]
> U tebe to bude <code>/home/<var>něco jiného</var></code> – podle tvého
> uživatelského jména.

Co to je ten domovský adresář?
Podívejme se zběžně na to, jak jsou organizovány složky v počítači.
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

> [note] Lomítka
> Pozor na dva významy, které má lomítko: na začátku jména souboru/adresáře
> označuje kořenový adresář; uprostřed odděluje jednotlivé adresáře.

Pod `home` je několik adresářů – jeden pro každého uživatele, který má
na tomto počítači účet:

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

Teď se podívej, co v tomhle adresáři je, pomocí příkazu `ls`:

```console
$ ls
Dokumenty  Obrázky  Stažené  Veřejné
Hudba      Plocha   Šablony  Videa
```

Příkaz `ls` vypíše názvy všech souborů a adresářů v aktuálním adresáři.
Můžeš zařídit, aby ukázal víc informací: přidej *přepínač*
(angl. *option*, *switch*, nebo *flag*) `-F`, který příkazu `ls` řekne,
aby k názvu přidal znak podle druhu souboru:

* `/` značí adresáře,
* `@` značí odkazy,
* `*` značí spustitelné soubory,
* bez symbolu jsou normální soubory.

```console
$ ls -F
Dokumenty/  Obrázky/  Stažené/  Veřejné/
Hudba/      Plocha/   Šablony/  Videa/
```

Ve Fedoře (v základním nastavení) se druh souboru dá určit i podle barvy,
kterou `ls` použije – ale barva se špatně kopíruje a dál zpracovává.

V našem příkladu je vidět, že Nelin domovský adresář obsahuje
pouze podadresáře.

> [note] Mazání terminálu
> Kdybys měl{{a}} na terminálu moc textu, můžeš použít
> <kbd>Ctrl</kbd>+<kbd>L</kbd> a obrazovku „vyčistit“.
> Pořád se můžeš vracet k předchozím příkazům pomocí <kbd>↑</kbd> a
> <kbd>↓</kbd>, nebo se i dostat k historii textu rolováním kolečkem myši.
> 
> Příkaz `clear` dělá něco podobného, ale smaže víc historie – rolování
> fungovat nebude, ale <kbd>↑</kbd>/<kbd>↓</kbd> ano.


## Syntax shellového příkazu

Příkaz níže použijeme jako příklad příkazu, který si rozebereme:

```console
$ ls -F /
```

Ono `ls` je *příkaz*, kterému dáváš *přepínač* `-F` a *argument `/`.
Přepíač `-F` jsme už viděli.
Obecně jména přepínačů začínají jednou pomlčkou (`-`) nebo dvěma (`--`).
Nějakým způsobem mění chování příkazu.
Argumenty pak většinou říkají příkazu, s jakým souborem či adresářem má
pracovat.
Příkazům můžeš dát více přepínačů a více argumentů, ale někdy se obejdeš i
bez nich.

> [note]
> Přepínače a argumenty jsou ekvivalent pozičních, resp. pojmenovaných
> argumentů v Pythonu: kdyby Bash uměl otevírat soubory jako Python,
> volání `open('soubor.txt', encoding='utf-8')` by se zapsalo zhruba jako
> `$ open --encoding=utf-8 soubor.txt`.

Jednotlivé části jsou oddělené mezerami: kdybys vynechal{{a}} mezeru mezi
`ls` a `-F`, Bash by hledal příkaz `ls-F`, který neexistuje.
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

Ale zpátky k `ls -F /`: tento příkaz vypíše soubory a adresáře v kořenovém
adresáři `/` a označí je podle typu. Například:

```console
 ls -F /
 bin@    lib@          opt/    srv/
 boot/   lib64@        proc/   sys/
 dev/    lost+found/   root/   tmp/
 etc/    media/        run/    usr/
 home/   mnt/          sbin@   var/
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
Usage: ls [OPTION]... [FILE]...
List information about the FILEs (the current directory by default).
Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all                  do not ignore entries starting with .
  -A, --almost-all           do not list implied . and ..
      --author               with -l, print the author of each file
  -b, --escape               print C-style escapes for nongraphic characters
      --block-size=SIZE      with -l, scale sizes by SIZE when printing them;
                               e.g., '--block-size=M'; see SIZE format below
  -B, --ignore-backups       do not list implied entries ending with ~
  -c                         with -lt: sort by, and show, ctime (time of last
                               modification of file status information);
                               with -l: show ctime and sort by name;
                               otherwise: sort by ctime, newest first
  -C                         list entries by columns
      --color[=WHEN]         colorize the output; WHEN can be 'always' (default
                               if omitted), 'auto', or 'never'; more info below
  -d, --directory            list directories themselves, not their contents
  -D, --dired                generate output designed for Emacs' dired mode
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
případně <kbd>mezerník/kbd> či <kbd>PgUp</kbd>/<kbd>PgDn</kbd> na skákání
po stránkách.
Můžeš tu i hledat: zmáčkni <kbd>/</kbd>, hledané slovo a <kbd>Enter</kbd>.
Najde-li se víc výsledků, můžeš mezi nimi přepínat pomocí <kbd>n</kbd>
a <kbd>N</kbd> (tedy <kbd>Shift</kbd>+<kbd>n</kbd>).

Na **zavření** manuálové stránky použij <kbd>Q</kbd>.


### Web

Třetí možnost, jak dostat informace o příkazu, je samozřejmě Internet.
Když k hledanému výrazu připíšeš `unix command` nebo `unix man page`,
dostaneš relevantnější výsledky.

Projekt GNU spravuje zevrubnou dokumentaci
k [základním příkazům][manuals-coreutils]
i [ostatním GNU projektům][manuals-gnu].

[manuals-coreutils]: https://www.gnu.org/software/coreutils/manual/coreutils.html
[manuals-gnu]: https://www.gnu.org/manual/manual.html


## Další přepínače pro `ls`

Co dělá přepínač `-l`?
Co se stane, když `-l` zkombinuješ s `-h`?

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


## Kombinování příkazů

Jednopísmenné přepínače, které začínají jednou pomlčkou, se dají kombinovat
dohromady: `ls -h -l` můžeš zkrátit na `ls -hl`, `ls -t -r -l` na `ls -trl`.

U „dlouhých“ přepínačů jako `--help` to nefunguje.


## Obsah dalších adresářů

Příkaz `ls` umí zobrazit obsah jakéhokoli adresáře, ne jen toho aktuálního.
Podívej se teď do svého adresáře `Dokumenty`, a to s přepínačem `-F`, který
označí druhy souborů.

```console
$ ls -f Dokumenty
data-shell.zip   data-shell/
```

Měl{{a}} bys vidět všechny soubory ve svých Dokumentech: archiv, který jsi
před chvílí stáhl{{a}} a adresář, kam se rozbalil.
A možná i nějaké jiné.

K adresáři `data-shell`, který je v Dokumentech, se můžeš dostat dvěma způsoby.

První je zadat *cestu* k němu, tedy `Dokumenty/data-shell`.

```console
$ ls -f Dokumenty/data-shell
creatures/          molecules/          notes.txt           solar.pdf
data/               north-pacific-gyre/ pizza.cfg           writing/
```

Příkaz `ls` se podíval do adresáře `Dokumenty` a v něm pak do adresáře
`data-shell`.

Druhý způsob je tou cestou projít pomocí příkazu `cd`, který změní aktuální
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

* `..`, tedy nadřazený adresář (zde tedy Dokumenty)
* `.`, což je jméno pro *aktuální* adresář (zde tedy `data-shell`)
* `.bash-profile`, což je *skrytý soubor*. Všechny soubory, jejichž jméno
  začíná tečkou, nástroje jako `ls` normálně neukazují.
  (Znáš-li Git, vzpomeneš si možná, že tečkou začíná adresář `.git` a
  soubor `.gitignore`.)

Adresáře `.` a `..` nejsou jen nějaká zkratka příkazu `cd`: jsou to opravdové
jména příslušných adresářů.
Můžeš je použít kdykoli, když pojmenováváš adresář, dokonce i jako součást
cesty. Zkus si:

```console
$ ls ..
$ ls creatures/../data/
```


Tak, teď víš, jak se orientovat v souborovém systému pomocí příkazů `cd`,
`ls` a `pwd`.
Pojďme se ještě podívat na několik variací.
Co se stane, když napíšeš samotné `cd`?

```console
$ cd
```

Jestli to není jasné z výzvy, zkontroluj kde jsi pomocí `pwd`.
Dostaneš se do svého domovského adresáře!

Příkazy shelly mají *výchozí chování*, které nastane když jim nepředáš
argument.
Příkaz `ls` vypíše aktuální adresář; chová se tedy stejně jako `ls .`.
Kdyby se ale `cd` choval jako `cd .`, nic by se nestalo.
A tak přejde radši do adresáře domovského.

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


## Další zkratky

Shell dává speciální význam znaku `~`.
Když nějaký argument začíná `~/` (nebo je jen `~`), shell místo něj doplní tvůj
domovský adresář (a vytvoří tak absolutní cestu).
Můžeš tedy psát:

* `ls ~` místo `ls /home/nela`
* `ls ~/Dokumenty/data-shell` místo `ls /home/nela/Dokumenty/data-shell`
* `cd ~` místo `cd`

> [note]
> Toto je zkratka *shellu*: vlnovka není opravdové jméno adresáře.
> Nemůžeš např. v Pythonu použít `open('~/data.txt')`.
> Oproti tomu `open('../data.txt')` fungovat bude; `..` a `.` jsou jména adresářů
> se vším všudy.
>
> Shell za `~` doplňuje i když argument není jméno souboru.
> Například příkaz `echo` svůj argument vypíše;
> vyzkoušej si, do dělá `echo Ahoj!` a `echo ~/Ahoj!`

Další zkratka je `cd -`, což tě vrátí do naposledy navštíveného adresáře.
Rozdíl mezi `cd ..` a `cd -` je ten, že `..` tě dostane do *nadřazeného*
adresáře, který je pro každý adresář daný, kdežto `-` používá historii
shellu, která se s každým dalším `cd` mění.

> [note]
> Toto je pro změnu zkratka *příkazu `cd`* – pro jiné příkazy, jako `ls`,
> pomlčka fungovat nebude. Nebo bude fungovat jinak.



(XXX: Další cvičení – viz [SC](http://swcarpentry.github.io/shell-novice/02-filedir/index.html#absolute-vs-relative-paths))


## Nela a medúzy – Doplňování

Nela teď ví spoustu nových věcí o adresářích a chce si zorganizovat
soubory, které bude vytvářet.
Vytvořila si proto adresář `north-pacific-gyre` (severní tichomořský vír)
aby věděla, odkud data jsou.
Uvnitř si vytvořila adresář `2012-07-03` – datum, kdy začala zpracovávat.
Kdysi používala jména jako `clanek` a `opravene-vysledky`, ale takové názvy
po pár letech pestaly dávat smysl. (Poslední kapka byl tenkrát adresář
`opravene-opravene-vysledky-3`. Git ještě Nela nezná.)

> [notes] Řazení kalendáře
> Nela pojmenovává adresáře <var>rok</var>-<var>měsíc</var>-<var>den</var>
> s dvouciferným měsícem a dnem (tj. s případnou nulou na začátku).
> Když takové názvy seřadí podle abecedy (např. pomocí `ls`), seřadí se
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
Je tu 19 možností, které nezačínají stejně.
Když ale Nela zmáčkne <kbd>Tab</kbd> podruhé, všechny tyto možnosti se vypíšou
a může si z nich vybrat.

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

To je docela otrava, ale většinou to nevadí, protože v Linuxu se dá
kopírovat i snadnějším způsobem: kdykoli myší *označíš* nějaký text,
zkopíruje se do zvláštní schránky, kterou pak můžeš vložit pomocí
*prostředního tlačítka myši*.
Když si na to zvykneš, není cesty zpět :)

Občas můžeš dokonce využít toho, že máš k dispozici dvě schránky – jednu
s výběrem, druhou s textem zkopírovaným
<kbd>Ctrl</kbd>(+<kbd>Shift</kbd>)+<kbd>C</kbd>.
