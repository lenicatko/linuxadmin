
# Zápis – Syscalls, kontejnery

## "šipečky", `>>`

Přesměrování a přidávání už znáš:
```
$ echo "Ahoj" > pozdrav
$ cat pozdrav
Ahoj
$ echo "Zdravim" >> pozdrav
$ cat pozdrav
Ahoj
Zdravim
```

## heredoc

Slouží k zapsání (zpravidla) z víceřádkového textu jako hodnoty.
Za dvě šipečky napiš `END`, pak můžeš psát víceřádkový text, a až ho budeš chtít předat na standardní vstup `cat`, napiš `END` a zmáčkni klávesu <kbd>Enter</kbd>. 
Místo `END` můžeš použít libovolný řetězec, který pak zopakuješ na konci zadané hodnoty.
```console
$ cat << END
...
ABCD
efgh
END
```
Existují i tři šipečky: To, co za ně napíšeš, se předá na standardní vstup programu.
```console
$ cat <<< "Abcdef"
Abcdef
```
Je to výhodné, pokud chceš předat na standardní vstup proměnnou prostředí.
```console
$ cat <<< "$HOME"
/home/user
```

# syscalls

**Linuxové jádro**, anglicky *kernel*
* přepíná mezi běžícími procesy, aby to vypadalo, že běží všechny najednou
* zprostředkovává např. otevírání souborů
	* komunikace probíhá voláním funkcí, zavolá se funkce *otevírání souborů*, a vrátí číslo (*file descriptor*)
	* dostane vstup programu: "chci přečíst 20 bajtů ze souboru 3" - pošle je.

```
+--------+    +--------+
|        |    |        |
| proces |    | proces |
|        |    |        |
+---+----+    +----+---+
    ^              ^                +------------------+
    |              |                |                  |
    |              |                | Soubory na disku |
    v              v                |                  |
+---+--------------+---+            +------------------+
|                      |                   |
|        jádro         |-------------------+
|                      |
+----------------------+

```
Nakresleno pomocí: http://asciiflow.com/

## sledování syscalls pomocí `strace`

`strace` vypíše všechna systémová volání, které daný program používá.
Nástroj `strace` se používá se jménem nějakého programu, použij ho s příkazem `echo`.

```console
$ strace echo
execve("/usr/bin/echo", ["echo"], 0x7ffed87ac530 /* 59 vars */) = 0
brk(NULL)                               = 0x564fe1290000
arch_prctl(0x3001 /* ARCH_??? */, 0x7ffd4c975070) = -1 EINVAL (Invalid argument)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=109457, ...}) = 0
mmap(NULL, 109457, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fdbbd5a9000
close(3)     
...
+++ exited with 0 +++
```

Na začátku výpisu je `execve` (spusť program) mimo jiné s argumenty:
- jméno spustitelného souboru
- argumenty příkazové řádky

Trochu dál následuje volání `openat`/`open`, kde dostane to jméno souboru, mód otevření jako argumenty, a návratová hodnota je *file descriptor* (číslo souboru).
`read(3, [...])`, `[...]` dostane jako argument jméno souboru a místo v paměti, které se má použít. 
Jádro to místo v paměti přepíše obsahem souboru, a vrátí počet přečtených bajtů.
Nakonec se zavolá `close()`, s číslem souboru k uzavření, kde `1` je standardní výstup, a `2` je chybový výstup.
Je to ta nízká úroveň, na které procesy komunikují s jádrem.

Můžeš se i podívat na něco složitějšího jako `strace python`.
Výstup je o dost delší. 
Pokud zmáčkneš nějaké klávesy, ve výstupu bude vidět volání `read`, jak se načítají jednotlivé znaky z terminálu.
Zmáčknutím <kbd>CTRL</kbd> + <kbd>C</kbd> se zruší `strace`.

K čemu to může být dobré? 
Občas se hodí vědět například, které soubory program otevírá.


# Kontejnery

Když vývojář naprogramuje nový program, na jeho počítači to zpravidla funguje dobře. 
Až ho má hotový a chce se o něj podělit s kolegy, dost často zjistí, že to u kolegů neběží, protože v `/etc/něco` zapomněl poslat určitý konfigurační soubor. 
Tento problém se stává natolik často, že zasloužil si vlastní vtip: *works on my machine*.
To systémovým administrátorům vadí a kontejnery mají za cíl to řešit.

Kontejnery řeší **izolaci** prostředí, a taky zajišťují **sjednocené** prostředí pro tvůj program. 
Na všech systémech poběží stejně.

Jak je to možné?
Kontejner ti nedodá jen program, on dodává celý souborový systém se všemi knihovnami, zabalený pěkné do jednoho souboru.
Potom řekne linuxovému jádru: *já jsem speciální proces a můj kořenový adresář (/) nebude celý ten disk, ale jen určený adresář*. 
Pro tento proces adresář `/` bude obsahovat úplně jiné soubory, než pro zbytek systému. 
Podle toho, jaký má kořenový adresář, interpretuje si ostatní cesty, o které si tvůj program řekne. 
Co to znamená? Můžeš vzít tento kousek souborového systému, ten obraz disku s programem a poslat ho jinam. 
Nemusíš se dál starat o nastavení `/etc/`, protože to nastavení bude stejné všude, kde někdo spustí daný kontejner.
Tato vlastnost kontejnerů umožňuje jejich **distribuci**.

Má to další využití!
Pokud nějakému programu nevěříš stoprocentně a nechceš, aby ti napáchal problémy v počítači, můžeš ho spustit v kontejneru, který zařídí, že se ty vrstvy odizolují. 
Takový proces, cokoli by se snažil provést, provede jen v izolovaném prostředí kontejneru.

Kvůli výše popsaným vlastnostem je to i dost složitější.
V kontejneru se můžou izolovat nejen **soubory** a souborové systémy, ale i **uživatelé**. 
Uživatel číslo 0, tj. `root` může být jiný pro systém, a jiný pro proces kontejneru. 
To znamená, že můžeš zařídit: *uživateli hanka dej č. 0, dej mu všechna práva v tom kontejneru a ten si může dělat systémová nastavení, ale ven se nedostane*.
Můžeš změnit **síťování** - nastavit, že proces nemůže komunikovat s Internetem, ale jen s nějakými jinými programy, se kterými mu to dovolíš. 
Můžeš určit hranici **zásob**, které proces smí spotřebovat, např. max. 2kB paměti, a pokud bude chtít víc, tak ho systém ukončí. 
Nebo spustit program na maximálně 5 vteřin, a pokud bude chtít víc, ukončí se.

Teoreticky by tedy kontejnery měly být bezpečné: mají svůj vlastní píseček a neměly by sahat na zbytek systému. 

Díky kontejneru můžeš vzít program a dát ho na další systém. 
Zde bývá problém, protože operační systémy bývají velké.
Představ si, že všechny knihovny, co máš na počítači, potřebuješ mít stejně, aby program fungoval, tzn. musí být součástí kontejneru.
Nemůžeš prostě použít základní knihovnu C z hostitelského počítače: všechno kromě jádra se musí nakopírovat do kontejnerů. 
Teď si představ, že máš na počítači 10 různých služeb, tři databáze, dva webové servery a aplikaci, a pro každý druh této služby potřebuješ mít zvlášť celý souborový systém. 
Je toho celkem hodně. 


## `overlayfs`
A proto `docker` používá tzv. `overlayfs`. 
Ukazovali jsme si, jak fungují souborové systémy (TODO: link na lekci o FS) a že existuje spousta jejich druhů.

Jeden ze způsobu, jak zařídit souborový systém, je `overlayfs`. Je zvláštní v tom, že má data uložená ve dvou souborových systémech pod ním. 
(Analogie se šifrovaným diskem).

`overlayfs` funguje tak, že jsou dvě vrstvy:
* první - základní souborový systém (např. adresář na disku), pouze pro čtení
* druhá - zde se ukládají změny (rozdíl oproti základnímu systému)

Soubor se hledá nejprve na druhé vrstvě, a pokud se tam najde, přečte se z ní. 
Pokud se nenajde, přečte se z první vrstvy.
Základ je jen pro čtení a nemění se, a všechny zápisy jdou do druhé vrstvy.

 > Paralela se změnami v zákonech: Existuje základní zákon, který se nemění. 
 > Pokud vznikne potřeba změnit určitý paragraf v určitém odstavci, zveřejňuje se aktualizace toho paragrafu. 
 > A pokud se chceš tímto zákonem řídit, musíš se podívat do obou dokumentů. 

K čemu to je dobré? 
Když máš ten systém, na němž má běžet 10 kontejnerů, z nichž 3 jsou databáze a 2 webové servery, můžeš mít jeden základ pro všechny, a pak nad tím můžeš použít několik `overlays`. 
Představ si, že jedna z webových aplikací je napsaná v knihovně Flask a druhá v Djangu. 
Vezmeš tak základní webový server, postavíš nad tím `overlay` a nainstaluješ si tam Django. 
Pak vezmeš ten základ, a postavíš něco jiného. 
Základ můžeš sdílet mezi několika kontejnery.

// zde byl obrázek na tabuli, ale nemám ho k dispozici :(//

## Docker
Docker je jedna z implementací kontejnerů.
Je to první produkt, který se úspěšně prosadil v oblasti kontejnerizace a pro spoustu společností je dnes synonym pro kontejnery. 
Existují ale i jiné technologie, které dělají v podstatě to samé.

Podívejme se webovou stránku jménem [Docker Hub](https://hub.docker.com/). 
Jsou zde ke stažení obrazy kontejnerů. 

Podíváme se na kontejner s `httpd`.

 - Jaký je rozdíl mezi instalaci programu do systému (např. z repozitáře linuxové distribuce) a instalaci jako obraz Docker?
 
Instalace do systému samozřejmě bude fungovat. 
Ale když si to nainstaluješ do jiného systému, tak to bude fungovat jinak. 
Základní nastavení programu na Fedoře je jiné než na Ubuntu.
Když pak budeš chtít ten program odinstalovat ze systému, on po sobě uklidí nebo ne - záleží, jak je napsaný. 
Za to když to spustíš z kontejneru, je tam vždy stejný operační systém a celé nastavení bude stejné, ať už ho spustíš na Fedoře, Ubuntu nebo jiném systému.
Odstranění kontejneru zajistí, že se se vždy uklidí všechno, co s tím programem souviselo.

Má to blízko k *virtuálnímu počítači*. 
Hlavní rozdíl je v tom, že virtuální počítač používá procesy, které se navenek tváří jako procesor, disk atd. Má vlastní systém a vlastní procesy.
U Dockeru se ty věci používají v rámci systému, na kterém to spouštíš. 
V tomhle se Docker spíš podobá virtuálnímu prostředí (jako to, které používáš pro izolaci Pythonu na svém počítači).


## Instalace `docker`

Pokud používáš Fedoru 31, podívej se na `podman` v návodu níže.

Pro ostatní distribuce platí tento postup.
Nainstaluj si balíček `docker` a pusť `docker daemon`.
*Daemon* se pustí:
```console
sudo systemctl start docker
```
Když budeš spouštět příkazy `docker ...`, bude po tobě pravděpodobně chtít to spustit jako uživatel `root`. 
Ten pak komunikuje s `docker daemonem` .

Raději se ale přidej do skupiny `docker`.
```console
$ sudo usermod -a -G docker $(whoami)
$ su - $(whoami)
```

S nainstalovaným dockerem můžeš stáhnout obraz `httpd` z Docker Hubu:
```console
$ docker pull httpd
(... stahování vrstev ...)
```


## Instalace `podman`

Jedná se o implementaci typickou pro Fedoru.
Na rozdíl od `docker` nepotřebuje práva roota a pro naši potřebu to bude fungovat stejně.
```console
$ sudo dnf install podman
$ podman pull httpd
```


## Obraz a kontejner

Příkaz `pull` stáhl spoustu *blobů*. 
To jsou jednotlivé vrstvy. 
Pokud už nějakou máš staženou z dřívějška, rovnou se použije, nebude se stahovat podruhé.

Než se podíváme na `httpd`, začneme s jiným operačním systémem.
Stáhni například Ubuntu. Pokud sis instaloval{{a}} podman, vyměň `docker` v příkazech níže za `podman`.

```console
$ docker run -it ubuntu
root@934745e0585a:/# 
``` 
Příkaz výše stáhne a spustí příkazovou řádku z kontejneru Ubuntu.

Tam si zkus spustit:
```console
root@934745e0585a:/# cat
abc
abc
```
Vše funguje, jak má.

Otevři si v jiném terminálu (svého systému) příkaz `htop` a pomocí <kbd>F5</kbd> si zobraz stromový výpis. 
Ve výpisu najdeš běžící proces `cat` z toho kontejneru.

Pokud bys to samé udělal{{a}} ve virtuálním počítači, najdeš jen jeden proces, který dělá virtuální procesor, ale už ne procesy, které se na něm spouští.

Proto je možné např. ukončit proces v kontejneru z hlavního systému.
Kontejner ale vidí jiný souborový systém - podívej se do `/home` - bude nejspíš prázdný. 
Když se pokusíš použit příkaz `ping` v kontejneru, zjistíš, že tady není a musí se doinstalovat. Na obyčejném Ubuntu by ovšem byl k dispozici.
Systém určený pro kontejnery je „ořezaný“ – mnohem menší než "plná" distribuce. Všechno, co systémový administrátor potřebuje, aby mu běžely konkrétní služby, ale může bez problémů doinstalovat.
Údržba takového systému je pak mnohem méně náročná než velkého „kombajnu“. Čím méně nainstalovaných programů v kontejneru, tím bezpečnější by to mělo být.

Pro vyskočení z ubuntí příkazové řádky stačí ukončit terminál, například příkazem `exit`.

Vypiš seznam všech stažených obrazů:
```console
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
httpd               latest              2ae34abc2ed0        13 days ago         165MB
ubuntu              latest              775349758637        5 weeks ago         64.2MB
```
*Image*, neboli obraz, je obraz disku nějakého toho systému, tzn. `overlay`. 
Kdy uděláš nový kontejner, tzn. použiješ příkaz `run`:
 - vezmou se informace z toho obrazu, 
 - nad tím se vytvoří vrstva na to, co se uloží na tom kontejneru, kdybys změnil{{a}} nějaké soubory 
 - pustí se nějaký program, v našem případě *bash*.


Vypiš seznam všech právě běžících kontejnerů. Pokud ti právě běží Ubuntu, uvidíš ve výpisu něco podobného: 
```console
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
c9f835211bbb        ubuntu              "/bin/bash"         39 seconds ago      Up 38 seconds                                   focused_khayyam
```

Vypiš seznam všech kontejnerů, které máš na disku (i těch, které právě neběží):
```
$ docker ps --all
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
4a523c41344d        ubuntu              "/bin/bash"         10 minutes ago      Exited (0) 2 minutes ago                        nostalgic_williams
697cccb2c307        ubuntu              "/bin/bash"         12 minutes ago      Exited (0) 11 minutes ago                       jovial_keller
```

Kontejnery můžeš klidně smazat pomocí: 
```console
$ docker rm <container-id>
```

Obdobně smažeš obrazy:
```console
$ docker image rm <image-id>
```

Jaký je rozdíl mezi kontejnerem a *image*?

 - *image* je obraz disku, nedá se změnit
 - *kontejner* je proces, běží s informacemi, které obsahuje image; navíc může zapisovat data, může se změnit; je instanci obrazu

*Image* můžou být sdílené mezi vícero kontejnery.
*Image* se mohu smazat jedině pokud neběží žádný kontejner, který je na nich založeny.
*Image* může být založen i na jiném *image*, slouží k tomu vrstvy, aby základ systému se dal sdílet a neplýtvalo se místem na disku.


## Webový server v kontejneru
```console
$ docker run -it httpd
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
[Sun Dec 13 15:35:46.045992 2020] [mpm_event:notice] [pid 1:tid 139871236002944] AH00489: Apache/2.4.41 (Unix) configured -- resuming normal operations
[Sun Dec 13 15:35:46.057586 2020] [core:notice] [pid 1:tid 139871236002944] AH00094: Command line: 'httpd -D FOREGROUND'
```

Image `httpd` je nastaven tak, že nepustí `bash`, ale spustí přímo server `httpd` hned na popředí. 
Nejsou třeba žádní démoni.
Jediný úkol toho kontejneru je pustit webový server, a přesně to dělá. 
Když zrušíš kontejner, všechno v něm se smaže. 
Říká se, že ideálně v jednom kontejneru by měl běžet jeden proces.

Kontejner můžeš taky spustit s přepínačem `-d`, čili *detached*, což znamená, že se spustí na pozadí.
Můžeš tak spustit dokonce těch kontejnerů víc.

```console
$ docker run -d httpd
$ docker run -d httpd
$ docker run -d httpd
$ docker run -d httpd
```
Až je budeš chtít smazat, použij příkazy 
```console
$ docker stop <container-ip>
$ docker rm <container-ip> 
```

My se ale chceme podívat do webového serveru, takže spustíme v kontejneru `bash`:
```console
$ docker run -it httpd bash
root@f0a4bf04ccc6:/usr/local/apache2# 
```
`-it` říká, že chceš kontejner spustit interaktivně a na popředí, což se nám hodí pro `bash`.

Podívej se do adresáře `htdocs`. Je v něm soubor `index.html`, který obsahuje větu *It works!*

```console
root@8933505e2865:/usr/local/apache2# ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  logs	modules
root@8933505e2865:/usr/local/apache2# cd htdocs/
root@8933505e2865:/usr/local/apache2/htdocs# ls
index.html
root@8933505e2865:/usr/local/apache2/htdocs# cat index.html 
<html><body><h1>It works!</h1></body></html>
```

Kontejner teď zavři, napiš do příkazové řádky `exit`.

Na tu stránku s *It works* se chceme podívat ze svého opravdového počítače. 
Proto spusť `httpd` s přepínačem `-p`, který nadefinuje porty:

```console
$ docker run -p 8080:80 httpd
```

Port `80` je port na straně kontejneru, který se zpřístupní na portu `8080` na tvém (hostitelském) počítači.
Pokud tedy zadáš do svého prohlížeče adresu `localhost:8080` uvidíš stránku *It works!*.

Dále si zkusíme připojit nějaký vlastní adresář dovnitř souborového systému v kontejneru.
Zruš běžící `httpd` kontejner.

Nejdříve si vytvoř prázdný adresář a v něm soubor `index.html`.
```console
$ echo Ahoj! > index.html
```
A teď spusť kontejner s přepínačem `-v`:

```console
$ docker run -p 8080:80 -v "$PWD":/usr/local/apache2/htdocs/:Z httpd
```
Někdy na Fedoře je potřeba použít na konci cesty `:Z`, což ti zpřístupní soubory z domovského adresáře (standardně to totiž je zakázané).

Když teď obnovíš stránku v prohlížeči, uvidíš **Ahoj!**.


## Dockerfile

Existují způsoby, jak udělat nový kontejner, do kterého místo standardního `index.html` dáš vlastní soubor.
Jeden z nich je pomocí souborů Dockerfile.

Vytvoř si `Dockerfile` (musí být psané s velkým písmenem na začátku). 
Do toho souboru si zapíšeme, jak takový kontejner vyrobit.
Má své vlastní příkazy, jako: *začni s kontejnerem A, na tohle místo dej soubor B, doinstaluj balíček C atd*.

Uděláme si Dockerfile z `httpd`, a v něm spustíme jeden příkaz:
```
FROM httpd
RUN echo Ahoj > /usr/local/apache2/htdocs/index.html
```
Tím vznikne nový obraz disku, jehož většina je stejná jako u `httpd`. 

Soubor ulož a vytvoř nový image příkazem `build` s několika argumenty:
 - výsledek se otaguje (`-t`) jménem `mujhttpd`, 
 - tečka na konci příkazu znamená místo, ve kterém se nachází Dockerfile, který se má použít k vytvoření nového image.
Co to udělá? Provedou se postupně všechny příkazy z Dockerfile.

```console
$ docker build -t mujhttpd .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM httpd
 ---> 2ae34abc2ed0
Step 2/2 : RUN echo Ahoj > /usr/local/apache2/htdocs/index.html
 ---> Using cache
 ---> 47e1060cde54
Successfully built 47e1060cde54
Successfully tagged mujhttpd:latest
```

Když teď spustíš svůj kontejner stejně jako před chvíli oficiální `httpd`, v prohlížeči na svém počítači uvidíš stránku s textem **Ahoj!**
```console
$ docker run -p 8080:80 mujhttpd
```

Takový hotový kontejner se dá pak nahrát na Docker Hub.

Jde to dokonce spustit na Windows, kde není linuxové jádro.
Vytvoří se v něm virtuální stroj s Linuxem, na němž je možné kontejnery bez problému spustit.
