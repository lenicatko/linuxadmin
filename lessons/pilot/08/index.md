
# 8. Linuxová administrace - balíčkovací systémy a httpd

Nainstaluj virtuální stroj ze stránek https://alt.fedoraproject.org/en/ (*Network Installer Fedora Server*).
Vytvoř uživateli `root` při instalaci heslo, které si dokážeš zapamatovat pro účely této hodiny.


## Instalace balíčků
Jakým způsobem instaluješ software na linuxovém systému?

Pravděpodobně máš k dispozici grafický program, kde si můžeš dostupné programy prohlédnout, přečíst jejich popisky, kouknout na obrázky, jak vypadají, a zakliknout tlačítko "Instaluj". 
Grafický instalátor se postará o vše potřebné a zpřístupní ti ikonku nového programu v nabídce aplikací.

My se ale dnes podíváme hlouběji, na systémy, díky kterým můžeš instalovat balíčky jednotným způsobem.

### Balíčkovací systémy
Linuxové systémy používají tak zvané balíčkovací systémy. 
Filozofie pod nimi je přímočará - je to vlastně o tom, že stáhneš balíček kódu a rozbalíš si ho na disk (podobně jak ZIP-ový adresář).
Balíček může obsahovat příkazy, konfigurace apod.
Samotný balíček obsahuje:
- soubory
- metadata - např. jméno, verze balíčku
- závislosti (ang. *dependencies*) - co všechno je potřeba, aby mohl balíček fungovat

Pokud si stáhneš kupříkladu textový editor, bude potřebovat knihovnu na vykreslení okének. 
Tu knihovnu musíš mít nainstalovanou, aby fungoval editor. 
Grafická knihovna pak potřebuje např. standardní knihovnu jazyka C atd.
Instalace jednoho balíčku tak často znamená, že ve skutečnosti stáhneš mnohem víc balíčků - ten "hlavní" a všechny ostatní, na kterých závisí.

Fedora používá balíčkovací systém `rpm`. 
`rpm` umí různé akce s balíčky, mj.: 
- instalaci 
- zjištění obsahu
- odinstalaci

`rpm` neřeší stahování balíčků z internetu. 
Pokud máš balíček a řekneš `rpm`, aby ho nainstaloval, on dokáže říct: *nenainstaluji ho, protože potřebuje k svému běhu ještě 5 dalších, které nejsou nainstalované*.
Je ale na tobě, abys tyto chybějící závislosti vyřešil{{a}}.
A protože je to úkol, se kterým se uživatelé systémů nechtějí měřit sami, vznikla nástavba nad `rpm`, která se jmenuje `dnf`, a která pomáhá řešit závislosti a stahování. 
Když chceš nainstalovat balíček pomocí `dnf`, zachová se trochu jinak. Řekne například: *OK, ale potřebuješ ještě dalších 5 balíčků, zde máš seznam, stáhnu všechny položky a nainstaluji všechny závislosti*.

Pomocí `rpm -q` se zeptáš na informace o balíčku, který už je nainstalovaný na daném systému.
```console
$ rpm -q bash
bash-5.0.7-1.fc30.x86_64
```

`rpm -q -l bash` - zobrazí všechny soubory, jaké balíček instaluje.
Vyzkoušej si to, seznam těch souborů je velký.
Najdeš mezi nimi např. soubory v `/etc/skel` - adresáři, který se zkopíruje do adresářů všech nově vytvořených uživatelů.
Nový uživatel dostane hned v domovském adresáři `.bashrc`, `.bash_logout`...

`rpm -q --requires bash` - jaké balíčky potřebuje `bash`, aby mohl být nainstalován:
- potřebuje `sh`
- konfiguraci
- souborový systém v nějaké verzi
- knihovnu C (libc)
atd.

`sudo dnf install [package]` - instalace balíku
`sudo dnf remove [package]` - odinstalace balíčku
`sudo dnf update` - `dnf` se podívá na internet, zjistí nejnovější verze všech balíčků, které máš na svém počítači a nabídne ty, které je možné aktualizovat 


### Repozitář 
Repozitář je místo na internetu, kde jsou ke stažení balíčky.
Ruzné repozitáře obsahují různé balíčky. 
Pro jednu linuxovou distribuci si jich můžeš přidat několik. 
Když chceš nainstalovat nový balíček, systém se podívá na všechny repozitáře, které má na svém seznamu, jestli v nich najde hledaný balíček.

Na Fedoře se repozitáře nacházejí v `/etc/yum.repos.d/`. 

Linuxové distribuce se liší svým přístupem k softwaru.
Fedora, kupříkladu, nemá ve svých repozitářích přehrávač videa.
Ten se dá doinstalovat z repozitáře `rpmfusion`, který, technicky vzato, obsahuje dva repozitáře:
- free - balíčky s programy s otevřeným zdrojovým kódem
- nonfree - balíčky s uzávřeným zdrojovým kódem: Skype, Steam etc.
Na stránkách [rpmfusion](https://rpmfusion.org/) najdeš návod, jak přidat repozitář do svého systému.

## Firewall

Ještě než půjdeme na náš dnešní úkol, musíme pozměnit nastavení firewallu na naší Fedoře.
Firewall je síťové zařízení, které povoluje/blokuje připojení podle zadaných parametrů.
Dnes spustíme webový server.
Webové servery poslouchají na portu 80, který musíš povolit. 
Port je číslo, které říká, k jaké službě se chceš připojit. 
Každá služba má přiřazený port, na kterém pracuje
- webserver má port 80 (`http`), 443 (`https`).

Pokud ti běží server (teď ještě ne, ale už za chvilku) a chceš na něho vidět zvenku, z jiného počítače, je třeba otevřít port 80.

Když nainstaluješ nový systém, nainstaluje se ti s ním firewall, který defaultně všechny tyto porty uzavře.
Na konfiguraci firewallu existuje nástroj.
Webový server na virtuálním počítači povolíš příkazem:
```console
# firewall-cmd --add-service=http
success
``` 
A zapiš, že má být povolen port pro `http` po dalším startu:
```console
# firewall-cmd --permanent --add-service=http 
success
```

## Webový server

Už víš, jak a odkud instalovat, tak je vhodný čas na dnešní úkol.
Nainstalujeme si tradiční webový server, `Apache`, neboli `httpd`.
Existují různé alternativy k Apachi, dnes se často používá `nginx`, který je rychlejší a štíhlejší. 
Apache je starší a robustnější. 
Je to dobrý příklad, aby sis vyzkoušel{{a}} fungování démona, procesu, který běží na pozadí a zprostředkovává nějakou službu.
Je to zároveň dobrý příklad toho, jak konfigurovat podobné systémové služby a jak se o takové procesy starat.

### Instalace httpd
Příklady níže předpokládají, že jsi do systému přihlášen{{a}} jako `root`.

Nainstaluj `httpd` pomocí příkazu`dnf install httpd` a potvrď po vyzvání klávesou <kbd>Y</kbd>.

Když se podíváš na všechny soubory, které ti nainstaloval tento balíček, uvidíš dlouhý seznam. Zkus:

```console
$ rpm -q -l httpd
/etc/httpd/conf
/etc/httpd/conf.d/autoindex.conf
/etc/httpd/conf.d/userdir.conf
/etc/httpd/conf.d/welcome.conf
/etc/httpd/conf.modules.d
/etc/httpd/conf.modules.d/00-base.conf
/etc/httpd/conf.modules.d/00-dav.conf
...
```

### Správa služby httpd
Fedora se řídí filozofií, že nově nainstalované balíčky jsou sice k dispozici, ale se automaticky nenakonfigurují ani nespouští.
Správce si je musí nakonfigurovat a spustit sám.
Existují linuxové distribuce, jako Ubuntu, které se zaměřují na to, aby systém vyžadoval malé množství konfigurace, kde by se ta stejná služba rovnou spustila s nějakou výchozí konfigurací. 

#### Start, stop, enable, disable
`httpd` je démon. O démonech jsme si povídali v minulé hodině.
O démony se stará systémový proces, který se jmenuje `systemd`. 
`systemd` dává k dispozici příkaz `systemctl` na spouštění a správu procesů.

Když spustíš jen příkaz `systemctl`, uvidíš velký seznam věcí, co běží na počítači (procesy, zařízení, připojené souborové systémy, logování, nastavení automatického času, internetové připojení, atd.). 

Spusť službu (proces) `httpd`:
```console
# systemctl start httpd
```
Nastartovanou službu máš teď k dispozici. Kdyby proces spadl s chybou, `systemd` ho nastartuje znova. 

Pro vypnutí služby `httpd` existuje příkaz (zatím ji nevypínej, budeme s ní dál pracovat):
```console
# systemctl stop httpd 
```

Zobraz stav služby (jestli běží, na jakém portu poslouchá, seznam procesů, atp.):
```console
# systemctl status httpd
```

Nyní můžeš načíst webovou stránku webového serveru přímo v terminálu virtuálky, pokud zadáš příkaz `curl` a jako argument uvedeš svou IP adresu. Zkus to.
`curl` je příkaz, který umí stahovat webové stránky. 
Jako argument mu můžeš dát i libovolnou www adresu, např. `curl https://github.com`.

Můžeš si dokonce nainstalovat verzi textového prohlížeče do virtuálního počítače, např. `links`: `dnf install links`, spustí se pak příkazem `links <adresa>`.

Zkus zobrazit stránku nainstalovaného webserveru:

```console
$ links IP_adresa_virtualniho_pocitace 
```
Pomocí klávesy <kbd>q</kbd> `links` opustíš.

My ale chceme se na tu webovou stránku podívat z jiného počítače, od sebe, graficky. 
Nyní, pokud zadáš IP adresu virtuálního počítače do prohlížeče na svém počítači, měla by se ti načíst webová stránka. 
Zde je důležité, abys měl{{a}} správně nakonfigurovaný firewall. Pokud máš problém s načtením obsahu stránky, ještě jednou zkontroluj nastavení firewallu.

Webserver budeš mít k dispozici tak dlouho, dokud máš zapnutý virtuální počítač. 
Jakmile ho vypneš, služba se ukončí. 
Po opětovném startu systému se webserver nezapne sám.
Na to potřebuješ ještě jeden příkaz, který znamená, že se služba zapne vždy po startu systému.

```console
# systemctl enable httpd
```

> [note]
> Příkaz `start` se týká jen tohoto běhu počítače, nastartuje službu pro tebe tady a teď. 
> `enable` s tím nesouvisí, ten řídí jen zapínání při startu systému. 
> Pokud zastavíš službu příkazem `stop` a restartuješ počítač, služba bude po restartu opět zapnutá. 

Dobrý test, jestli máš službu korektně nastavenou, je restartování počítače (provedeš příkazem `reboot`). Pokud potom běží tak, jak jsi ji naplánoval{{a}}, je vše v pořádku.

#### Restart a reload

Se službami se pojí ještě dva příkazy, které se hodí znát. Jsou to:
- `systemctl restart httpd` 
- `systemctl reload httpd`
   
Když změníš konfiguraci, server se o tom nedozví automaticky za svého běhu. 
Musíš mu říct, že se má restartovat a při startu si načte aktualizovanou konfiguraci.
Rozdíl mezi `restart` a `reload` je, že v případě `reload` se pouze načtou změny v konfiguračních souborech a připojení k webserveru zůstanou aktivní. 
V případě restartu se ukončí vše.

Kde najdeš nastavení webového serveru? 
Tam, kde bys ho nejspíš čekal{{a}} :).
Je v `/etc/httpd/conf.d/`, kde najdeš několik souborů. 
Můžeš provést změnu v `/etc/httpd/conf.d/welcome.conf`, například vše zakomentovat (s použitím znaku `#`). 
Tento soubor definuje obsah stránky, kterou dostaneš, když se zeptáš na IP adresu počítače s běžícím webovým serverem.
Po zakomentování by se mělo začít zobrazovat něco jiného.
Když se ale teď podíváš do prohlížeče, nebude žádná změna.
Server si zatím změny nevšiml.
Můžeš teď buď vypnout a zapnout server, nebo mu říct, aby si načetl nové nastavení. 
Půjdeme druhou variantou, použij `systemctl reload httpd` (příp. `restart`) pro zobrazení změn.

##### DocumentRoot a UserDir
Apache servíruje obsah adresáře určitým způsobem.
V souboru `/etc/httpd/conf.d/autoindex.conf` je nastavení toho, jak se generují výpisy adresářů.
V `/etc/httpd/conf/httpd.conf` je základní nastavení a to hlavní v něm je `DocumentRoot`, kde by mělo být řečeno, kde jsou uloženy dokumenty, kterými disponuje webový server.
Typicky je to `/var/www/html/`.
Vytvoř si v něm soubor `hello.txt` a napiš do něj *Hello, World!*
Podívej se na stránku a zkontroluj, že se ten soubor objeví v prohlížeči.
Obsah `/var/www/html` už není nastavení serveru. Když přijde dotaz, co je na té hlavní stránce, zobrazí se obsah toho adresáře. Tohle bude fungovat i bez reloadu serveru, stačí obnovení stránky (klávesou <kbd>F5</kbd>.

V `/etc/httpd/conf/httpd.conf` je ještě jedno nastavení, a to `UserDir`. 
Nastavuje, že pokud má uživatel vytvořený adresář `public_html`, pak tento adresář lze zobrazit pod `/~user/`, kde *user* je jméno uživatelského účtu. 
Na prvních unixových systémech to bylo skvělé, každý si mohl jednoduše dát na web, co chtěl.
Pak se zjistilo, že je to bezpečnostní riziko, protože se podle toho dá velmi jednoduše zjistit, jestli uživatel daného jména existuje na tom počítači a zkoušet se na ně připojit.
Z tohoto důvodu je to standardně vypnuté, ale v případě potřeby můžeš tuto funkcionalitu opět zapnout .


## DÚ
- vytvořit si nového uživatele
- Povolit uživatelské webové stránky
- Zveřejnit nějaký obsah pomocí webového serveru

Dokumentace je v komentářích v `/etc/httpd/userdir.conf`

