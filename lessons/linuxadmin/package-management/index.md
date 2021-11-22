# Instalace balíčků

Jakým způsobem instaluješ software na linuxovém systému?

Pravděpodobně máš k dispozici grafický program, kde si můžeš dostupné programy prohlédnout, přečíst jejich popisky, kouknout na obrázky, jak vypadají, a zakliknout tlačítko "Instaluj".
Grafický instalátor se postará o vše potřebné a zpřístupní ti ikonku nového programu v nabídce aplikací.

My se ale dnes podíváme hlouběji, na systémy, díky kterým můžeš instalovat balíčky jednotným způsobem.

## Balíčkovací systémy

Linuxové systémy používají takzvané *balíčkovací systémy* (angl. *package managers*).
Filozofie pod nimi je přímočará - je to vlastně o tom, že stáhneš archiv
a rozbalíš si ho na disk podobně jak ZIP-ový adresář.
Balíček může obsahovat příkazy (soubory v `/usr/bin`), konfiguraci (soubory
v `/etc`), knihovny (soubory), manuálové stránky (taky soubory) a tak dále.

Některé věci, např. vytvoření uživatele, nelze přes soubory; balíčky proto
můžou obsahovat malé programy – skriptlety – které podobné věci při instalaci
zajistí.

Kromě souborů a skriptletů balíček obsahuje *metadata* jako např.:
- jméno balíčku
- verzi balíčku
- závislosti (ang. *dependencies*) - co všechno je potřeba, aby mohl balíček fungovat

Pokud si stáhneš kupříkladu textový editor, bude potřebovat knihovnu na vykreslení okének.
Tu knihovnu musíš mít nainstalovanou, aby editor fungoval.
Grafická knihovna pak potřebuje např. standardní knihovnu jazyka C atd.
Instalace jednoho balíčku tak často znamená, že musíš instalovat mnohem víc
balíčků - ten “hlavní” a všechny ostatní, na kterých závisí.

Podobná situace nastane při odinstalování: kromě „hlavního“ odinstalovaného
programu se smažou i dalśi balíčky, které nebudou potřeba.

## Dva systémy

Hodně systémů používá balíčkovací nástroje dva.
Jeden základní zvládá samotnou instalaci a odinstalaci, plus zjišťování
a kontrolu toho, co je zrovna nainstalováno.
Neřeší ale stahování balíčků z internetu.
Na Fedoře je to `rpm`.
Pokud máš nějaký balíček už stažený a řekneš `rpm`, aby ho nainstaloval,
můžeš dostat odpověď: *nenainstaluji ho, protože potřebuje k svému běhu ještě
5 dalších, které nainstalované nejsou*.
Je na tobě, abys tyto chybějící závislosti vyřešil{{a}}.
A protože tohle uživatelé nechtějí dělat sami,
vznikly nadstavby, které navíc řeší stahování balíčků a všech jejich závislostí.
To umí ve Fedoře DNF.

Když instaluješ pomocí `dnf`, zadáš jméno balíčku který chceš a dostaneš
odpověď jako: *OK, ale potřebuješ ještě dalších 5 balíčků, zde máš seznam.
Všechny je stáhnu a nainstaluju.*

Kromě DNF existují i další takové nadstavby: *Gnome Software* je grafická
aplikace, který instaluje další grafické aplikace.
Nenajdeš v něm tedy `bash` nebo `tree`.
Podobně jako `dnf` stahuje a instaluje včetně závislostí.

Volba balíčkovacího systému se v různých Linuxových distribucích liší.
Často to je jedna z mála částí systému, které nejdou snadno vyměnit za jinou.
Tady je malý přehled:

| Distro          | Základní systém             | Nadstavby                    |
| --------------- | --------------------------- | ---------------------------- |
| Fedora          | RPM                         | DNF; yum                     |
| Debian & Ubuntu | APT                         | aptitude; Synaptic; APT      |
| Arch Linx       | pacman                      | pacman                       |
| OpenSUSE        | RPM                         | YaST                         |

Tyto balíčkovací nástroje spravují samotný operační systém:
vše od jádra přes shell až po „uživatelské“ aplikace.
Instalují software ke kterému mají přístup všichni uživatelé systému;
potřebují tedy superuživatelské oprávnění.

> [note]
> Existují i balíčkovací nástroje které nespravují samotný systém.
> Ty umožňují přiinstalovat software „navíc“, často jen pro konkrétního
> uživatele.
> Tyhle nástroje jsou často spjaté s určitým jazykem (`pip` pro Python,
> `npm` pro JavaScript, `cargo` pro Rust); některé existují jen pro konkrétní
> operační systém (Chocolatey pro Windows, Google Play Store a F-Droid pro
> Android); některé jsou univerzálnější (Nix, Conda, Homebrew, Flatpak, Snap).

Máme nainstalovanou Fedoru; pojďme se tedy podívat na RPM a DNF.
Nástroje ostatních systémů vesměs umí stené základní operace,
ale příkazy budou vdy vypadat trošku jinak.


## RPM – informace o balíčcích

Pomocí `rpm -q` (z angl. *query*, dotaz) se zeptáš na informace o balíčku,
který máš nainstalovaný.
Samotným přepínačem `-q` dostaneš aktuálně nainstalovanou verzi:

```console
$ rpm -q bash
bash-5.1.0-2.fc34.x86_64
$ rpm -q python3
python3-3.9.7-1.fc34.x86_64
$ rpm -q gnome-terminal
gnome-terminal-3.40.3-1.fc34.x86_64
```

> [note]
> V některých případech může být nainstalovaných verzí víc, např.
> u jádra systému:
>
> ```console
> $ rpm -q kernel
> kernel-5.11.12-300.fc34.x86_64
> kernel-5.14.16-201.fc34.x86_64
> ```
>
> Kdyby „poslední“ verze byla špatná – nepodařilo by se s ní zapnout
> systém – můžeš pořád použít tu předchozí a systém spravit.
> U většiny balíčků lze mít nainstalovanou jen jednu verzi.

Příkaz `rpm -q -l bash` (`l` z angl. *list*, vyjmenovat) zobrazí všechny
soubory které balíček obsahuje.
Vyzkoušej si to: zjistíš že seznam těch souborů je celkem dlouhý.
Najdeš mezi nimi samotný příkaz – soubor `/usr/bin/bash`,
ale např. i soubory v `/etc/skel` - to je adresář,
který se zkopíruje do adresářů všech nově vytvořených uživatelů.
Nový uživatel tedy v domovském adresáři bude mít `.bashrc`, `.bash_logout`
atd.

Příkaz `rpm -q --requires bash` vypíše závislosti – to co `bash` potřebuje,
aby mohl být nainstalován:

```console
$ rpm -q --requires bash
/usr/bin/sh
config(bash) = 5.1.0-2.fc34
filesystem >= 3
libc.so.6()(64bit)
libc.so.6(GLIBC_2.11)(64bit)
libc.so.6(GLIBC_2.14)(64bit)
libc.so.6(GLIBC_2.15)(64bit)
[...]
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
rpmlib(PayloadIsZstd) <= 5.4.18-1
rtld(GNU_HASH)
```

Nejde jen obalíčky, ale i o jiné vlastnosti systému:
- potřebuje příkaz `sh`, tedy soubor `/usr/bin/sh`
- konfiguraci `config(bash)` v konkrétní verzi
- souborový systém (balíček `filesystem`) v konkrétní minimální verzi
- knihovnu C (`libc`) pro konkrétní procesor (`64bit`) s vlastnostmi
  z konkrétních verzí
- určité vlastnosti balíčkovacího systému (`rpmlib`)

Kromě `filesystem` tohle nejsou přímo jména balíčků, ale soubory
a „popisky“ vlastností, které musí systém splňovat.

* `bash` nebo `filesystem` je jméno balíčku.
* `bash-5.1.0-2.fc34.x86_64` je jméno konkrétního balíčku v jedné konkrétní
  verzi. Když napíšeš jen `bash`, RPM to pochopí jako právě nainstalovanou
  verzi Bashe – tenhle dlouhý řetězec.
* `filesystem >= 3` říká, že balíček `filesystem` musí být ve verzi `3`
  nebo vyšší. Pomocí `rpm -q filesystem` si můžeš ověřit, že tohle je splněno.
* `/usr/bin/sh` je soubor – stejně jako jakákoli jiná závislost která
  obsahuje lomítko.
* `libc.so.6()(64bit)` je „virtuální závislost“ – něco, co musí poskytnout
  nějaký jiný balíček. Není to ale přímo jméno balíčku.

Příkaz `rpm -q --provides bash` vypíše „virtuální závislosti“,
které ostatním balíčkům poskytuje `bash`:

```console
$ rpm -q --provides bash
/bin/bash
/bin/sh
bash = 5.1.0-2.fc34
bash(x86-64) = 5.1.0-2.fc34
config(bash) = 5.1.0-2.fc34
```

> [note]
> Konfiguraci `config(bash)` Bash potřebuje (`--requires`), ale zároveň
> si ji poskytuje (`--provides`).
> To je normální.

Když chceš zjistit, který balíček poskytuje některou závislost,
můžeš použít přepínač `--whatprovides`:

```console
$ rpm -q --whatprovides 'libc.so.6()(64bit)'
glibc-2.33-20.fc34.x86_64
```

`libc.so.6()(64bit)`, tedy standardní knihovnu jazyka C,
poskytuje balíček `glibc`.
A které balíčky tuhle věc potřebují?
Ty vypíše přepínač `--whatrequires`, a je jich hodně:

```console
$ rpm -q --whatrequires 'libc.so.6()(64bit)'
[...]
$ rpm -q --whatrequires 'libc.so.6()(64bit)' | wc -l
```


## Informace o Pythonu

Balíček s Pythonem se na Fedoře jmenuje `python3`.
Zkus se podívat, jaké soubory obsahuje:

```console
$ rpm -ql python3
/usr/bin/pydoc
/usr/bin/pydoc3
/usr/bin/pydoc3.9
/usr/bin/python3
/usr/bin/python3.9
/usr/lib/.build-id
/usr/lib/.build-id/f4
/usr/lib/.build-id/f4/aa4b6fb3d014792f7a0be4ed6cbdd514c32d9e
/usr/share/doc/python3
/usr/share/doc/python3/README.rst
/usr/share/man/man1/python3.1.gz
/usr/share/man/man1/python3.9.1.gz
```

To je docela málo!
Hlavní část Pythonu – standardní knihovna – je totiž „schovaná“ v jedné ze
závislostí:

```console
$ rpm -q --requires python3
libc.so.6()(64bit)
libc.so.6(GLIBC_2.2.5)(64bit)
libpython3.9.so.1.0()(64bit)
python3-libs(x86-64) = 3.9.7-1.fc34
rpmlib(CompressedFileNames) <= 3.0.4-1
rpmlib(FileDigests) <= 4.6.0-1
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
rpmlib(PayloadIsZstd) <= 5.4.18-1
rtld(GNU_HASH)
```

Je to `python3-libs(x86-64)`.

Když chceš zjistit, který balíček tuhle poskytuje,
můžeš použít přepínač `--whatprovides`:

```console
$ rpm -q --whatprovides 'python3-libs(x86-64)'
python3-libs-3.9.7-1.fc34.x86_64
```

A opravdu, balíček `python3-libs` verze `3.9.7-1.fc34.x86_64`
poskytuje závislost `python3-libs(x86-64)`:

```console
$ rpm -q --provides python3-libs-3.9.7-1.fc34.x86_64
bundled(libmpdec) = 2.5.0
bundled(mpdecimal) = 2.5.0
libpython3.9.so.1.0()(64bit)
libpython3.so()(64bit)
python-libs = 3.9.7-1.fc34
python3-libs = 3.9.7-1.fc34
python3-libs(x86-64) = 3.9.7-1.fc34
python3.9-libs = 3.9.7-1.fc34
```

A pomocí `rpm -q -l python3-libs` můžeš zobrazit soubory v tomto balíčku.
Tentokrát je jich několik tisíc.


## Repozitář

Příkaz `rpm` nekomunikuje po internetu.
Neumí tedy prohledávat balíčky které ještě nemáš nainstalované
nebo zjišťovat, jestli u některých z nich není k dispozici nová verze.

Takž můžeš zjistit, jak se jmenuje balíček s příkazem `bash` nebo `cat`:

```console
$ rpm -q --whatprovides /usr/bin/bash
bash-5.1.0-2.fc34.x86_64
$ rpm -q --whatprovides /usr/bin/cat
coreutils-8.32-30.fc34.x86_64
```

Ale nemůžeš to udělatpro příkaz `ogg123` (pokud ho nemáš nainstalovaný):

```console
$ rpm -q --whatprovides /usr/bin/ogg123
chyba: soubor /usr/bin/ogg123: Adresář nebo soubor neexistuje
```

K tomu je potřeba prohlédnout *repozitář* – seznam všeho softwaru,
který Fedora poskytuje.

K práci s repozitáři slouží příkaz `dnf`, který umí podobné věci jako
`rpm`, ale „ví“ i o balíčcích, které zrovna nejsou nainstalované.

Obdoba `rpm -q` je `dnf repoquery`.
Můžeš se tedy podívat, jaký balíček obsahuje příkaz `ogg123`:

```console
$ dnf repoquery --whatprovides /usr/bin/ogg123
Fedora 34 - x86_64                              6.8 MB/s |  74 MB     00:10    
Fedora 34 openh264 (From Cisco) - x86_64        2.5 kB/s | 2.5 kB     00:01    
Fedora Modular 34 - x86_64                      2.3 MB/s | 4.9 MB     00:02    
Fedora 34 - x86_64 - Updates                    5.5 MB/s |  29 MB     00:05    
Fedora Modular 34 - x86_64 - Updates            3.4 MB/s | 4.2 MB     00:01    
vorbis-tools-1:1.4.2-2.fc34.x86_64
```

Poslední řádek je jméno balíčku, který obsahuje `/usr/bin/ogg123`.
Co jsou ale ty řádky nad tím?

Když `dnf` pustíš poprvé, stáhne si informace o repozitářích (tj. o softwaru
který sedá nainstalovat).
Když ho pustíš znovu, místo aktualizace jen vypíše poznámku o tom, že jsou
naposledy stažené informace dost aktuální:

```console
$ dnf repoquery --whatprovides /usr/bin/ogg123
Poslední kontrola metadat: před 0:00:48, Po 22. listopadu 2021, 14:53:50.
vorbis-tools-1:1.4.2-2.fc34.x86_64
```

> [note]
> Informace o stahování a aktuálnosti se vypsaly na standardní chybový výstup.
> Když ten přesměruješ do `/dev/null`, dostaneš jen samotné jméno balíčku:
>
> ```console
> $ dnf repoquery --whatprovides /usr/bin/ogg123 2>/dev/null
> vorbis-tools-1:1.4.2-2.fc34.x86_64
> ```




## Instalace a odinstalace

RPM umí i instalovat (`-i`) a mazat (`-e`) balíčky, ale protože neřeší
závislosti a stahování, hodí se to jen v hodně zvláštních situacích, např:

* Spravuješ superbezpečný armádní systém, který není připojený k internetu,
  a balíčky musíš stahovat {{gnd('sám', 'sama')}} a nechat si je vždy schválit.
* Z nějakého důvodu chceš závislosti ignorovat. (Instalovaný program pak
  nejspíš nebude správně fungovat, ale když jsi `root`, můžeš vše.)

Na instalaci je lepší použít příkaz DNF:

* <code>dnf install <var>balíček</var></code> - instalace balíčku
  (včetné stažení a závislostí)

* <code>dnf remove <var>balíček</var></code> - odinstalace balíčku
  (větně nepotřebných závislostí)

* `dnf update` - aktualizace – `dnf` se podívá na internet,
   zjistí nejnovější verze všech balíčků, které máš nainstalované a nabídne
   novější verze.

Na rozdíl od `dnf repoquery` a `rpm -q` tyhle příkazy modifikují systém,
a tak je potřeba je pouštět s právy superuživatele – tedy pomocí `sudo`.

Zkus si nainstalovat balíček `vorbis-tools`:

```console
$ sudo dnf install vorbis-tools
Poslední kontrola metadat: před 0:42:11, Po 22. listopadu 2021, 14:35:00.
Závislosti vyřešeny.
============================================================================================
 Balíček                Architektura     Verze                       Repozitář        Velikost
============================================================================================
Instalování:
 vorbis-tools           x86_64           1:1.4.2-2.fc34              fedora           216 k

Shrnutí transakce
============================================================================================
Instalovat  1 balíček

Celková velikost ke stažení: 216 k
Velikost po nainstalování: 769 k
Je to ok [a/N]: a
Stahování balíčků:
vorbis-tools-1.4.2-2.fc34.x86_64.rpm                        1.1 MB/s | 216 kB     00:00    
--------------------------------------------------------------------------------------------
Celkem                                                      298 kB/s | 216 kB     00:00     
Spouští se kontrola transakce
Kontrola transakce byla úspěšná
Probíhá test transakce
Test transakce byl úspěšný.
Transakce probíhá
  Příprava        :                                                                     1/1 
  Instalování     : vorbis-tools-1:1.4.2-2.fc34.x86_64                                  1/1 
  Probíhá skriplet: vorbis-tools-1:1.4.2-2.fc34.x86_64                                  1/1 
  Ověřuje se      : vorbis-tools-1:1.4.2-2.fc34.x86_64                                  1/1 

Nainstalováno:
  vorbis-tools-1:1.4.2-2.fc34.x86_64

Hotovo!
```

Příkaz `dnf` předpokládá, že ho použtí člověk
Kdybys ho spouštěl{{a}} automaticky, ze skriptu, můžeš použít přepínač `-y`
(z angl. *yes*, ano) – `dnf` se pak na takové otázky nebude ptát a bude
předpokládat kladnou odpověď. Zkus si to na odinstalaci:

```console
$ sudo dnf remove -y vorbis-tools
Závislosti vyřešeny.
============================================================================================
 Balíček                Architektura     Verze                      Repozitář         Velikost
============================================================================================
K odstranění:
 vorbis-tools           x86_64           1:1.4.2-2.fc34             @fedora           769 k

Shrnutí transakce
============================================================================================
Odstranit  1 balíček

Uvolněné místo: 769 k
Spouští se kontrola transakce
Kontrola transakce byla úspěšná
Probíhá test transakce
Test transakce byl úspěšný.
Transakce probíhá
  Příprava        :                                                                     1/1 
  K odstranění    : vorbis-tools-1:1.4.2-2.fc34.x86_64                                  1/1 
  Probíhá skriplet: vorbis-tools-1:1.4.2-2.fc34.x86_64                                  1/1 
  Ověřuje se      : vorbis-tools-1:1.4.2-2.fc34.x86_64                                  1/1 

Odstraněno:
  vorbis-tools-1:1.4.2-2.fc34.x86_64

Hotovo!
```

A teď nainstalovat znova – ale trošku jinak.
Protože DNF přístup k repozitáři, umí vybrat správný balíček i podle souboru,
který zadáš.
Když tedy chceš nainstalovat opříkaz `ogg123`, můžeš zadat:

```console
$ sudo dnf install /usr/bin/ogg123
[...]
Hotovo!
```

A teď si štěknutí můžeš přehrát.
(Doufám že na virtuálním počítači to půjde slyšet.)

```console
$ locate bark.ogg
/usr/share/sounds/gnome/default/alerts/bark.ogg
sh-5.1$ ogg123 -d alsa /usr/share/sounds/gnome/default/alerts/bark.ogg
```


## Repozitáře

Repozitář je místo na internetu, kde jsou ke stažení balíčky.
Ruzné repozitáře obsahují různé balíčky.
Můžeš si jich přidat několik.
Když DNF instaluje nový balíček, kontroluje všechny.

Na Fedoře se repozitáře nacházejí v `/etc/yum.repos.d/`.
(Dnes by to bylo `dnf.repos.d`, ale DNF tu používá stejný formát jako starší
nástroj `yum`, tak zůstává i jméno adresáře.)

Proč si přidávat další repozitáře?

Jednotlivé linuxové distribuce – a jednotlivé repozitáře – se liší svým
přístupem k softwaru.
Fedora ve svých repozitářích nemá přehrávače videa nebo MP3,
protože v Americe jsou tyto technologie patentované a musí se za jejich
použití platit.
To u distribuce která je volně ke stažení nejde jednoduše zařídit.

Evropské právo ale softwarové patenty v takovém rozsahu neuznává,
a tak můžeš s klidným svědomím přidat repozitáře evropského projektu
`rpmfusion`.
Tento projekt spravuje dva repozitáře:
- free - balíčky s programy s otevřeným zdrojovým kódem
- nonfree - balíčky s uzávřeným zdrojovým kódem: např. Steam

> [note]
> Software v obou je k dispozici zdarma.
> „Free“ je tu od „freedom“ – volnost software používat, zkoumat a měnit.

Na stránkách [rpmfusion](https://rpmfusion.org/) se můžeš doklikat k návodu,
jak přidat repozitář do svého systému. Pro *free* variantu to je:

```console
$ sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
```

Při první instalaci z nového repozitáře se DNF zeptá, jestli věříš
šifrovacím klíčům, kterýmijsou balíčky podepsané:

```console
Importuje se GPG klíč 0xD651FF2E:
Uživatelské ID : "RPM Fusion free repository for Fedora (2020) <rpmfusion-buildsys@lists.rpmfusion.org>"
Otisk: E9A4 91A3 DE24 7814 E7E0 67EA E06F 8ECD D651 FF2E
Zdroj : /etc/pki/rpm-gpg/RPM-GPG-KEY-rpmfusion-free-fedora-34
Je to ok [a/N]:
```

Vypsaný *otisk* (angl. *fingerprint*) můžeš zkontrolovat na adrese
[rpmfusion.org/keys](https://rpmfusion.org/keys).
Jestli sedí, je balíček podepsaný od správců RPM Fusion.

Když se instalace povede, budeš mít na systému nový balíček
`rpmfusion-free-release`.
Ten přidává nastavení repozitáře do `/etc/yum.repos.d/`:

```console
$ rpm -ql rpmfusion-free-release
/etc/pki/rpm-gpg/RPM-GPG-KEY-rpmfusion-free-fedora-2020
/etc/pki/rpm-gpg/RPM-GPG-KEY-rpmfusion-free-fedora-32
/etc/pki/rpm-gpg/RPM-GPG-KEY-rpmfusion-free-fedora-32-primary
/etc/pki/rpm-gpg/RPM-GPG-KEY-rpmfusion-free-fedora-33
/etc/pki/rpm-gpg/RPM-GPG-KEY-rpmfusion-free-fedora-34
/etc/pki/rpm-gpg/RPM-GPG-KEY-rpmfusion-free-fedora-latest
/etc/pki/rpm-gpg/RPM-GPG-KEY-rpmfusion-free-fedora-rawhide
/etc/yum.repos.d/rpmfusion-free-updates-testing.repo
/etc/yum.repos.d/rpmfusion-free-updates.repo
/etc/yum.repos.d/rpmfusion-free.repo
```

Od teď můžeš nainstalovat přehrávače videa/hudby jako `mplayer` nebo `vlc`:

```console
$ sudo dnf install mplayer
$ mplayer /usr/share/sounds/gnome/default/alerts/bark.ogg
```

> [note]
> Zvuky ve formátu `.ogg` jde přehrávat pomocí `ogg123`,
> tedy i bez přidání `rpmfusion`.
> `mplayer` ale funguje i s `.mp3` a ostatními.


## Další repozitáře

Podobný „dodatečný“ repozitář si může založit kdokoli.
Kdyby ses chtěla do tvoření balíčků ponořit, výsledky své práce můžeš
distribuovat na stránkách [COPR](https://copr.fedorainfracloud.org/).
Když ti někdo řekne aby sis z COPR něco nainstaloval{{a}}, je na tobě jestli
mu budeš věřit.
Balíčky nainstalované pomocí `dnf` mají plný přístup k celému systému.

> [note]
> Na Ubuntu existuje podobný koncept „uživatelských“ repozitářů,
> kam si může kdokoli nahrát svůj software, pod jménem **PPA**
> (Personal Package Archive).
>
> Pro systémy CentOS, RHEL atd. existuje sada repozitářů **EPEL**
> (Extra Packages for Enterprise Linux), který má podobná pravidla
> jako Fedora.


## Co je nainstalováno?

Pomocí `dnf` můžeš instalovat jak grafické aplikace (na které funguje
i grafický nástroj *Gnome Software* tak i programy pro příkazovou řádku
nebo systémové knihovny.
Taky se můžeš např. podívat jaký  software máš nainstalovaný
(což jde i bez `sudo`):

```
$ sudo dnf list installed
```

Taky můžeš celý systém aktualizovat – DNF najde balíčky pro které
je v repozitářích k dispozici novější verze, stáhne je a nahradí jimi
to co máš nainstalováno:

```console
$ sudo dnf update
```
