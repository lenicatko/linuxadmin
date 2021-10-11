# Bash

První krok na cestě za porozuměním Linuxu bude práce s příkazovou řádkou;
konkrétně *shellem*.
Nejdřív si ale přibližme, co to vlastně je.


## Trocha kontextu, názvosloví a historie

### Jádro a GNU/Linux

Technicky vzato, [Linux](https://www.linuxfoundation.org/projects/linux/)
je název *jádra* operačního systému.
Jádro (angl. *kernel*) je ta část, která zařizuje aby ostatní programy mohly
běžet, a zajišťuje a „moderuje“ přístup ke sdíleným zařízením – procesoru,
paměti, klávesnici, obrazovce, atd.

Jádro samo o sobě je nepoužitelné: podobně jako k motoru auta je potřeba mít
volant, kola a karoserii, tak k jádru operačního systému potřebujeme
programy jako shell (a samozřejmě fyzický počítač).

To, čím se „náš“ Linux liší od jiných systémů používající Linuxové jádro
(jako Android a spousta „chytrých“ zařízení) jsou třeba příkazy jako `cd`
a `mkdir`, které zadáváme do příkazové řádky.
Ty – nebo aspoň jejich nejpoužívanější verze – jsou vyvíjeny jako součást
projektu [GNU](https://www.gnu.org/).

> [note]
> Někteří dokonce korektnější celému systému říkat GNU/Linux, protože
> Linux je jen jedniná část.
> Autor tohoto textu považuje pojmenování systému podle *dvou* částí
> jen o málo lepší než podle jediné, takže zůstaneme u krátkého „Linux“.


### Shell

Slovo *shell* znamená v angličtině *skořápka*, např. ořechu – tedy „obal“
kolem jádra.
Pro nás je to program, který na *základě přání uživatele spouští ostatní programy*.
Tedy: zjistí, jaký program chce uživatel spustit, a řekne jádru aby ho spustil.

U systémů jako MS Windows nebo macOS je shell „grafický“ a zakomponovaný do
systému, takže ho jako uživatel nemůžeme „oddělit“ od jádra.

I Linuxové systémy mají grafické shelly. Ten který používáme my je
[GNOME Shell](https://wiki.gnome.org/Projects/GnomeShell).
Hlavní úkol shellu je spouštět ostatní programy, ale má i jiné funkce:
 např. ukazuje čas nebo tě umí odhlásit.

> [note]
> GNOME Shell je jen jedna část prostředí GNOME; v rámci tohoto projektu
> se kromě shellu vyvíjí např. správce souborů
> [*Files* (`nautilus`)](https://wiki.gnome.org/Apps/Files),
> terminál [`gnome-terminal`](https://wiki.gnome.org/Apps/Terminal),
> nebo textový editor [`gedit`](https://wiki.gnome.org/Apps/Gedit).

Původní *shelly* ale byly textové: než počítače uměly vykreslovat okýnka,
když ještě komunikovaly přes psací stroj a tiskárnu, byl *shell* program,
který se zeptal jaký program spustit, spustil ho, a po skončení se zeptal
znovu.
Takový textový shell – příkazová řádka – byl hlavní způsob komunikace
s počítačem.

Většina shellových příkazů spouští další programy, i když má takový shell
– podobně jako ten grafický – i další funkce navíc.

Shell, který budeme používat my, je
[GNU Bash](https://www.gnu.org/software/bash/).
Jeho první verze vyšla v roce 1989 a je z velké části kompatibilní
s původním `sh` z roku 1979.
Nemá vychytávky novějších shellů jako [`fish`](http://fishshell.com/) nebo
[`xonsh`](https://xon.sh/), ani není nejrychlejší nebo nejbezpečnější,
ale je dost dobrý na to, aby byl stále nejpopulárnější – najdete ho téměř
na každém „linuxovém“ počítači (a dokonce i na macOS),
většinou jako výchozí volbu.


### Terminál

Samotný textový shell neumí *vykreslovat* písmenka.
To v sedmdesátých letech dělala teletypová tiskárna, později videoterminál
(obrazovka s klávesnicí).
Dnes to na to máme grafické okýnko programu, kterému se říká
[*emulátor terminálu*](https://cs.wikipedia.org/wiki/Emul%C3%A1tor_termin%C3%A1lu)
nebo jen “terminál” či „okýnko s příkazovou řádkou“.
V prostředí GNOME použijeme program *Terminál*.
Jeho funkcí je předávat písmenka napsaná na klávesnici určitému programu,
a vykreslovat to, co tento program chce vypsat.

Když Terminál spustíš, spustí se „v něm“ Bash.


### Proč text?

Proč ovládat počítač textově – psát příkazy na klávesnici místo klikání myší?

Je to složitější na naučení: příkazy si typicky musíš pamatovat nebo dohledat,
místo toho abys je {{gnd('našel', 'našla')}} v menu.
Ale jakmile to zvládneš, je textové ovládání často rychlejší.
Poslední příkaz zopakuješ pomocí <kbd>↑</kbd><kbd>Enter</kbd>.
Když si několik příkazů zaznamenáš v souboru, můžeš je rovnou zkopírovat
do shellu.
A z takového souboru s příkazy není daleko ke *skriptu*, kdy sadu příkazů
pojmenuješ a můžeš ji spouštět jako jediný příkaz – podobně jako si můžeš
v Pythonu několik příkazů pojmenovat a vytvořit si funkci.
Až budeš potřebovat zkopírovat třetí řádek z každého z tisíce souborů, nebo
každé pondělí provést analýzu aktuálního počasí, začne se ti znalost Bashe
vyplácet.

A když se ztratíš, můžeme ti mnohem lépe pomoct stylem „co vypíše příkaz
`foobar status`?“ místo „klikni na zelené tlačítko vpravo dole a pošli
mi *screenshot*“.

A nakonec – spousta důležitých počítačů nemá ani připojenou obrazovku, ani
nainstalovanou grafickou kartu, která umí vykreslovat okýnka.
Příkazy se na ně posílají textově přes síť; administrátorův notebook funguje
jako původní terminály: nástroje k zadávání a zobrazování textu.
Až tedy budeš spravovat *microservices* v *cloudu*, nebo *clustery*
v superpočítačovém centru, bude tvá klávesnice mnohem ošoupanější než myš.

Pojďme s tím začít.


## Pouštění příkazů

Když Bash pustíš, ukáže se ti *prompt* neboli *výzva* – Bash se ptá na další
příkaz.

```console
$ 
```

Bash většinou používá `$ `, i když se dá nastavit i jiný symbol.
Před dolarem jsou většinou nějaké další informace, typicky aktuální adresář
a jméno uživatele.

Důležité je, že prompt není součást příkazu: když tu uvidíš ukázku příkazu,
dolar nepíšeš ty, vždy píšeš až to za ním.
Stejně tak nepíšeš výstup – typicky to, co je na řádcích které dolarem
nezačínají. I to vypisuje počítač sám.

Zkus si zadat příkaz `ls` (zkratka z angl. *list*), který vypíše obsah
aktuálního adresáře. Zadej <kbd>l</kbd><kbd>s</kbd><kbd>Enter</kbd>:

```console
$ ls
Dokumenty  Obrázky  Stažené  Veřejné
Hudba      Plocha   Šablony  Videa
```

Příkaz `ls` není součást Bashe – je to samostatný program, který shell
spustí, když zadáš jeho jméno.
Zatímco `ls` běží, má přístup k terminálu: může vypisovat výsledky.
(Nebo může zpracovávat vstup z klávesnice, např. u funkce `input` Pythonu.)
Když doběhne, vrátí se „ke slovu“ opět Bash a výzvou s dolarem se zeptá
na další příkaz.


##  Nenalezený příkaz

Když Bash nenajde program který by měl spustit, vypíše chybovou hlášku:
```console
$ lss
bash: lss: Příkaz nebyl nenalezen...
Podobný příkaz je: 'ls'
```

To se stane, když uděláš překlep nebo když daný program není nainstalovaný.
Některé systémy umí v rámci chybové hlášky i napovědět podobný příkaz
(jako výše), nebo doporučit instalaci:

```console
$ blender
bash: blender: Příkaz nebyl nenalezen...
Nainstalovat balíček „blender“, který poskytuje příkaz „blender“? [N/y]
```

Občas se stane, že hledání podobného příkazu trvá příliš dlouho.
Zastavit hledání můžeš pomocí <kbd>Ctrl</kbd>+<kbd>C</kbd>.


## Složitější programy

Většina tohoto kurzu bude o spouštění malých prográmků, které umí jen jednu
věc, a o tom jak je spojovat dohromady.
Ale Bash umí pouštět jakýkoli program, který máš nainstalovaný.
Zkus si:

```console
$ firefox
$ gedit
$ nautilus
```

Kdyby některý z těchto programů nepustil po zavření zpátky k příkazové řádce,
zavři ho pomocí <kbd>Ctrl</kbd>+<kbd>C</kbd>.
(Některé programy zůstávají po zavření okna běžet; je to pro případ, že bys
chtěl{{a}} za chvilku otevřít okno nové.)

A některé grafické programy sice pustí Bash zpátky „ke slovu“ rychle,
ale běží dál a dokonce píšou do terminálu hlášky, které by tě mohly mást.
Doporučuji proto teď terminálové okno zavřít a otevřít nové.


## Nela a medúzy

Jako příklad pro použití Bashe si projdeme kurz od *The Carpentries*
– organizaci, která učí vědce.
Jejich příklad je praktický; možná ne pro tebe, ale úkoly které by byly
praktické pro všechny se hledají špatně a hry (jako v začátečnickém kurzu
Pythonu) se k Bashi moc nehodí.

Tady je příběh:

Nela Nemo, mořská bioložka, se právě vrátila z šestiměsíční výpravy
na severní Tichý oceán, kde studovala měkké organismy
v [Tichomořské odpadkové skvrně](https://cs.wikipedia.org/wiki/Velk%C3%A1_tichomo%C5%99sk%C3%A1_odpadkov%C3%A1_skvrna).
Nasbírala 150 vzorků, ve kterých její zařízení měřilo relativní zastoupení
300 proteinů.
Teď má 1500 souborů, které musí zpracovat speciálním programem `goostats`,
který kdysi napsal někdo z její univerzity.
A samozřejmě musí o výsledcích napsat článek.
Do konce měsíce.

Kdyby `goostats` pouštěla ručně, klikáním, musela by ho 1500× otevřít
a vybrat soubor.
To by jí zabralo as 12 hodin čistého času, kdyby přitom neudělala chybu
– nemluvě o dopadech na její karpální tunel.
A tak radši tuhle práci zautomatizuje pomocí Bashe a bude se soustředit
na ten článek.

V dalších lekcích se zaměříme na to, jak to Nela může zautomatizovat.
Přesněji řečeno, jak pustit program `goostats` na každý z tisíců souborů.

Vzniklý *skript* pak navíc půjde použít znovu, až Nela nasbírá víc dat.


K tomuto příběhu se váže sada souborů.
Stáhni si [archiv](static/data-shell.zip) do svého adresáře `Dokumenty`.
A potom ho rozbal. To se dělá následujícími příkazy:

```console
$ cd Dokumenty
$ unzip data-shell.zip
$ cd
```
