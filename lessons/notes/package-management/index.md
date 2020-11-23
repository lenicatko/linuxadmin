
# Instalace balíčků
Jakým způsobem instaluješ software na linuxovém systému?

Pravděpodobně máš k dispozici grafický program, kde si můžeš dostupné programy prohlédnout, přečíst jejich popisky, kouknout na obrázky, jak vypadají, a zakliknout tlačítko "Instaluj".
Grafický instalátor se postará o vše potřebné a zpřístupní ti ikonku nového programu v nabídce aplikací.

My se ale dnes podíváme hlouběji, na systémy, díky kterým můžeš instalovat balíčky jednotným způsobem.

## Balíčkovací systémy

Linuxové systémy používají tak zvané *balíčkovací systémy* (angl. *package managers*).
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

Hodně systémů používá balíčkovací nástroje dva.
Jeden základní zvládá instalaci, zjištění obsahu a odinstalaci, ale neřeší
stahování balíčků z internetu.
Na Fedoře je to `rpm`.
Pokud máš nějaký balíček stažený a řekneš `rpm`, aby ho nainstaloval,
můžeš dostat odpověď: *nenainstaluji ho, protože potřebuje k svému běhu ještě
5 dalších, které nejsou nainstalované*.
Je na tobě, abys tyto chybějící závislosti vyřešil{{a}}.
A protože je to úkol, se kterým se uživatelé systémů nechtějí měřit sami,
vznikly nástavby, které řeší stahování balíčků a jejich závislostí.
To umí ve Fedoře DNF.

Když instaluješ balíček pomocí `dnf`, zachová se trochu jinak.
Řekne například: *OK, ale potřebuješ ještě dalších 5 balíčků, zde máš seznam,
stáhnu všechny položky a nainstaluji všechny závislosti*.

Kromě DNF existují i další takové nástavby: *Gnome Software* je grafická
aplikace, který instaluje další grafické aplikace.
(Nenajdeš v něm tedy `bash` nebo `tree`.)
Podobně jako `dnf` je stahuje a instaluje včetně závislostí.

Volba balíčkovacío systému je něco, co se hodně liší v různých Linuxových
distribucích. Například:

| Distro          | Základní systém             | Použitelná nástavba          |
| --------------- | --------------------------- | ---------------------------- |
| Fedora          | RPM                         | DNF                          |
| Debian & Ubuntu | APT                         | apt-get; aptitude            |
| Arch Linx       | pacman                      | pacman                       |
| OpenSUSE        | RPM                         | YaST                         |

Grafický *Gnome Software* umí pracovat s více systémy – RPM, APT a dalšími.


## RPM – informace o balíčcích

Pomocí `rpm -q` (z angl. *query*, dotaz) se zeptáš na informace o balíčku,
který máš nainstalovaný:

```console
$ rpm -q bash
bash-5.0.7-1.fc30.x86_64
```

`rpm -q -l bash` - zobrazí všechny soubory, jaké balíček instaluje.
Vyzkoušej si to, seznam těch souborů je velký.
Najdeš mezi nimi např. soubory v `/etc/skel` - adresáři, který se zkopíruje do adresářů všech nově vytvořených uživatelů.
Nový uživatel dostane hned v domovském adresáři `.bashrc`, `.bash_logout`...

`rpm -q --requires bash` - co `bash` potřebuje, aby mohl být nainstalován.
Nejde jen obalíčky, ale i o jiné vlastnosti systému:
- potřebuje příkaz `sh`
- konfiguraci
- souborový systém (balíček `filesystem`) v nějaké verzi
- knihovnu C (`libc`)
- určité vlastnosti balíčkovacího systému (`rpmlib`)

RPM umí i instalovat (`-i`) a mazat (`-e`) balíčky, ale protože neřeší
závislosti, hodí se to jen v hodně zvláštních situacích, např:

* Spravuješ superbezpečný armádní systém, který není připojený k internetu.
* Z nějakého důvodu chceš závislosti ignorovat. Instalovaný program pak nejspíš
  nebude správně fungovat, ale když jsi `root`, můžeš vše.


## DNF – instalace a odinstalace

<code>sudo dnf install <var>balíček</var></code> - instalace balíčku (včetné stažení a závislostí)
<code>sudo dnf remove <var>balíček</var></code> - odinstalace balíčku
`sudo dnf update` - aktualizace – `dnf` se podívá na internet,
 zjistí nejnovější verze všech balíčků, které máš nainstalované a nabídne
 novější verze.


## Repozitáře

Repozitář je místo na internetu, kde jsou ke stažení balíčky.
Ruzné repozitáře obsahují různé balíčky.
Pro jednu linuxovou distribuci si jich můžeš přidat několik.
Když chceš nainstalovat nový balíček, systém se podívá na všechny repozitáře, které má na svém seznamu, jestli v nich najde hledaný balíček.

Na Fedoře se repozitáře nacházejí v `/etc/yum.repos.d/`.

Linuxové distribuce se liší svým přístupem k softwaru.
Fedora, kupříkladu, nemá ve svých repozitářích přehrávač videa nebo MP3,
protože v Americe jsou tyto technologie patentované.

Evropské právo softwarové patenty neuznává, a tak můžeš s klidným svědomím
přidat evropský repozitář `rpmfusion`.
Tento projekt spravuje dva repozitáře:
- free - balíčky s programy s otevřeným zdrojovým kódem
- nonfree - balíčky s uzávřeným zdrojovým kódem: Skype, Steam etc.

> [note]
> Software v obou je k dispozici zdarma.
> „Free“ je tu od „freedom“ – volnost software používat, zkoumat a měnit.

Na stránkách [rpmfusion](https://rpmfusion.org/) se můžeš doklikat k návodu,
jak přidat repozitář do svého systému:

```console
$ sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
```

Toto nainstaluje balíček `rpmfusion-free-release`, který přidá nastavení
repozitáře do `/etc/yum.repos.d/`:

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

A pak můžeš nainstalovat přehrávač videa/hudby:

```console
$ sudo dnf install mplayer
$ mplayer /usr/share/sounds/gnome/default/alerts/bark.ogg
```

> [note]
> Zvuky ve formátu `.ogg` jde přehrávat i bez přidání `rpmfusion`.
> `mplayer` ale funguje i s `.mp3` a ostatními.


## Co je nainstalováno?

Pomocí `dnf` můžeš instalovat jak grafické aplikace (na které funguje
i grafický nástroj *Gnome Software* tak i programy pro příkazovou řádku
nebo systémové knihovny.
Taky se můžeš např. podívat jaký  software máš nainstalovaný
(což jde i bez `sudo`):

```
$ sudo dnf list installed
```

Taky můžeš systém aktualizovat:

```console
$ sudo dnf update
```
