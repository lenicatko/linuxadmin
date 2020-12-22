# Kontejnery

Když vývojář naprogramuje nový program, na jeho počítači to zpravidla funguje dobře. 
Až ho má hotový a chce se o něj podělit s kolegy, dost často zjistí, že to u kolegů neběží, protože zapomněl říct o změně v `/etc/něco` nebo o potřebných balíčcích.
A nebo mají úplně jiný operační systém.
Tento problém se stává natolik často, že zasloužil si vlastní vtip: *works on my machine*.
Systémovým administrátorům podobné situace vadí a kontejnery mají za cíl to řešit.

Kontejnery řeší **izolaci** prostředí, a taky zajišťují že programy běží ve **sjednocené** prostředí.
Na všech systémech poběží stejně.

Jak je to možné?
Kontejner neobsahuje jen program, ale celý souborový systém – `/` a všechno v něm, se všemi knihovnami a nastavením, zabalený pěkně do jednoho souboru.
Při spuštění kontejneru se pak pro konkrétní porces nastaví *jiný kořenový adresář* (`/`) než pro zbytek systému.
Tento tedy uvidí úplně jiné soubory než zbytek systému.
Co to znamená? Můžeš vzít tento kousek souborového systému, ten obraz disku s programem a poslat ho jinam. 
Nemusíš se dál starat o nastavení `/etc/`, protože to nastavení bude stejné všude, kde někdo spustí daný kontejner.
Tato vlastnost kontejnerů umožňuje jejich **distribuci**.

Pokud nějakému programu nevěříš stoprocentně a nechceš, aby ti napáchal problémy v počítači, můžeš ho spustit v kontejneru, který zařídí, že se ty vrstvy odizolují.
Cokoli se takový proces snaží provést se provede jen v izolovaném prostředí kontejneru.

Kvůli výše popsaným vlastnostem je izolace trochu složitější než jen nastavení jiného kořenového adresáře.
V kontejneru se můžou izolovat nejen soubory a souborové systémy, ale i **uživatelé**.
Uživatel číslo 0, tj. `root` může být jiný pro systém, a jiný pro proces kontejneru.
To znamená, že můžeš říct: uživatel `hanka` ať v kontejneru superuživatel, tj. v tom kontejneru může dělat cokoli, ale ven se nedostane.
Můžeš změnit **síťování** - nastavit, jestli proces může komunikovat s internetem, nebo jen s konkrétními programy/pčítači.
Můžeš určit hranici **zásob**, které proces smí spotřebovat: např. max. 2kB paměti a 30s procesorového času.
Pokud bude chtít víc, tak ho systém ukončí.

Všechna omezení se pak vztahují i na případné další procesy, které ten první spustí.


## Kontejnery vs. virtuální stroje

Teoreticky by tedy kontejnery měly být bezpečné: mají svůj vlastní píseček a neměly by sahat na zbytek systému.
Je tu ale vždy možnost bezpečnostních chyb v samotném kontejnerovém systému.
Mnohem bezpečnější je v tomto ohledu virtuální počítač, který *simuluje* celý stroj – disk, obrazovku, procesor, atd.
Umožňuje ti spustit jiný operační systém, nebo dokonce systém určený pro jiný procesor.
(Znáš-li emulátory herních konzolí: to jsou taky virtuální počítače.)

Virtuální stroje ovšem mají podstatnou nevýhodu: jsou relativně velké (na disku i v paměti) a pomalejší než kontejnery.
Kontejnery totiž sdílejí jádro operačního systému s „hostitelem“.
A mezi sebou toho mohou sdílet víc.


## Sdílení vrstev

Díky kontejneru můžeš vzít program s celým prostředím a pustit ho na jiném systému.
Zde bývá problém, protože operační systémy bývají velké.
Například všechny knihovny – všechno v `/usr/lib*` – které jsou potřeba k běhu programu v kontejneru musí být součástí kontejneru.
Nemůžeš prostě použít základní knihovnu C z hostitelského počítače: všechno kromě jádra se musí nakopírovat do kontejnerů.
Teď si představ, že máš na počítači 10 různých služeb, tři databáze, dva webové servery a aplikaci, a pro každý druh této služby potřebuješ mít zvlášť celý souborový systém. 
Je toho celkem hodně.

A proto kontejnery používají *vrstvy*.
Ukazovali jsme si, jak fungují souborové systémy (TODO: link na lekci o FS) a že existuje spousta jejich druhů.

Jeden ze způsobů, jak zařídit souborový systém, je `overlayfs`.
Je zvláštní v tom, že má data uložená ve dvou *jiných* souborových systémech, tzv. vrstvách:

* V první je základní souborový systém, který je pouze pro čtení.
* Do druhé se ukládají změny – rozdíl oproti první vrstvě.

Soubor se hledá nejprve ve druhé vrstvě, a pokud se tam najde, přečte se z ní.
Pokud se nenajde, přečte se z první vrstvy.
Základ je jen pro čtení a nemění se, a všechny zápisy jdou do druhé vrstvy.

 > Paralela se změnami v zákonech: Existuje základní zákon, který se nemění.
 > Pokud vznikne potřeba změnit určitý odstavec, zveřejňuje se novela – aktualizace konkrétního paragrafu.
 > Pokud chceš znát *aktuální znění* zákona, musíš se podívat do obou dokumentů.

K čemu to je dobré?
Představ si že máš systém, na němž má běžet 5 kontejnerů: 1 databáze a 4 webové servery,
můžeš mít jeden základ pro všechny – základní systém.
Potom vytvoříš dva `overlayfs`, které *oba* používají tuto základní vrstvu, ale v jednom doinstaluješ webový server a ve druhém databázi.
Teď si představ, že dva z webových serverů potřebují knihovnu Flask a zbylé dva Django.
Vezmeš vrstvu se základním webovým serverem a nad ním vytvoříš další dva `overlayfs`: jeden pro Flask a druhý pro Django.
Společné části systému tak můžeš sdílet mezi všemi kontejnery, které je potřebují.

```plain
                 +------------------+         +---------------+
                 |                  |         |               |
                 |   Flask          |         |   Django      |
                 |                  |         |               |
                 +-+--------------+-+         +--+----------+-+
                   |              |              |          |
               +---v-+            |              |         +v----+
               |     |            |              |         |     |
               +-----+            |              |         +-----+
                                  |              |
    +-------------------+      +--v--------------v-+
    |                   |      |                   |
    | Databáze          |      |        Web        |
    |                   |      |                   |
    +-+--------------+--+      +-----+----------+--+
      |              |               |          |
      |              |               |          |
      |              |               |          |
+-----v-+          +-v---------------v--+     +-v------+
|       |          |                    |     |        |
|       |          |  Základní systém   |     |        |
|       |          |                    |     |        |
+-------+          +--------------------+     +--------+
```

Univerzální základní vrstvy, které jsou užitečné více lidem,
se dokonce dají sdílet po internetu.


## Docker Hub

Docker je jedna z implementací kontejnerů.
Je to první produkt, který se úspěšně prosadil v oblasti kontejnerizace a pro spoustu uživatelů je dnes synonymem pro kontejnery.
Existují ale i jiné technologie, které dělají v podstatě to samé.

Firma Docker, Inc. provozuje portál [Docker Hub](https://hub.docker.com/),
kde jsou ke stažení obrazy kontejnerů – ony užitečné základní vrstvy.
Kdokoli si může nějakou vytvořit a na Docker Hubu ji sdílet.

Podívej se na stránku [kontejner s `httpd`](https://hub.docker.com/_/httpd):
uvidíš popis, odkazy na dokumentaci a různé verze, a podobně.


## Docker a Podman

Docker můžeš nainstalovat z balíčku `docker`.
Po instalaci je potřeba spustit službu stejného jména.

Jiná implementace kontejnerů se jmenuje *Podman*.
Sice není tak populární, ale na Fedorře funguje trošku líp.
Výhodou je, že nepotřebuje běžící službu (démona) a k jeho ovládání
nepotřebuješ superuživatelská práva.
Jinak se ale používá úplně stejně.
Nainstaluj si tedy ten:

```console
$ sudo dnf install podman
```

Všechny příkazy níže budou fungovat i s `docker` místo `podman`,
pokud máš na systému `docker` nastavený.


> [note] Instalace Dockeru
>
> Pro ostatní distribuce platí tento postup.
> Nainstaluj si balíček `docker` a pusť `docker daemon`.
> *Daemon* se pustí:
> ```console
> sudo systemctl start docker
> ```
> Když budeš spouštět příkazy `docker ...`, bude po tobě pravděpodobně chtít to spustit jako uživatel `root`. 
> Ten pak komunikuje s `docker daemonem` .
>
> Raději se ale přidej do skupiny `docker`.
> ```console
> $ sudo usermod -a -G docker $(whoami)
> $ su - $(whoami)
> ```
>
> S nainstalovaným dockerem můžeš stáhnout obraz `httpd` z Docker Hubu:
> ```console
> $ docker pull httpd
> (... stahování vrstev ...)
> ```


## Stažení obrazu

Jedná se o implementaci typickou pro Fedoru.
Na rozdíl od `docker` nepotřebuje práva roota a pro naši potřebu to bude fungovat stejně.
```console
$ podman pull httpd
```

Příkaz `pull` stáhl spoust tzv. *blobů*.
To jsou jednotlivé vrstvy: základní systém, aktualizace k němu, webový server, atd.
Pokud už nějakou máš staženou z dřívějška (třeba pro jiný kontejner), rovnou se použije, nebude se stahovat podruhé.

Než se podíváme na `httpd`, začneme s jiným operačním systémem.
Spusť například Ubuntu. Pokud sis instaloval{{a}} podman, vyměň `docker` v příkazech níže za `podman`.

```console
$ podman run -it ubuntu
root@934745e0585a:/# 
```

Příkaz výše stáhne kontejner Ubuntu a rovnou ho spustí.

Tam si zkus spustit:
```console
root@934745e0585a:/# echo Ahoj!
Ahoj!
root@934745e0585a:/#  id    
uid=0(root) gid=0(root) groups=0(root)
root@934745e0585a:/# ls -a /home
.  ..
root@934745e0585a:/# cat
```
Vše funguje, jak má!
Jsi `root`, uživatel číslo 0.
Aldresář `home` je prázdný – jsi v izolovaném prostředí.

Otevři si v jiném terminálu (svého systému) příkaz `htop` a pomocí <kbd>F5</kbd> si zobraz stromový výpis. 
Ve výpisu najdi (<kbd>F3</kbd>) běžící proces `cat` z toho kontejneru.
Všimni si, že běží pod *tvým* uživatelským účtem, i když uvnitř kontejneru je tento účet `root`.

V kontejneru běží normální procesy; izolace spočívá v tom, že mají nastavený jiný adresář `/`, přenastavené uživatelské účty, a podobně.

> [note]
> Pokud bys to samé udělal{{a}} pro virtuálním počítač, najdeš na hostitelském systému jen jeden proces,
> který simulije virtuální procesor, disk, jádro systému i procesy v něm.
> Procesy, ve virtuálním stroji tedy na hostitelském systému nenajdeš – izolace je úplnější.

Proto je možné např. ukončit proces v kontejneru z hlavního systému.
(V `htop` přes <kbd>k<kbd><kbd>Enter<kbd>, což pošle signál `SIGTERM`.)
Když se pokusíš použit příkaz `ping` v kontejneru, zjistíš, že tady není a musí se doinstalovat.
To se na Ubuntu dělá jinak než na Fedoře:

```console
# apt-get update
# apt-get install iputils-ping
```

Na „normální“ verzi Ubuntu, nainstalované pro uživatele by ovšem `ping` byl k dispozici.
Systémy určené pro kontejnery jsou „ořezané“ – mnohem menší než "plné" distribuce.
Všechno, co systémový administrátor potřebuje, aby mu běžely konkrétní služby, si musí doinstalovat.
(Nebo použít vrstvu, kde už je to doinstalované , jako `httpd`.)
Údržba takového systému je pak mnohem méně náročná než velkého „kombajnu“.
Čím méně nainstalovaných programů v kontejneru, tím bezpečnější by to mělo být.

Pro vyskočení z ubuntí příkazové řádky stačí ukončit terminál, například příkazem `exit`.

Vypiš seznam všech stažených obrazů:
```console
$ podman images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
httpd               latest              2ae34abc2ed0        13 days ago         165MB
ubuntu              latest              775349758637        5 weeks ago         64.2MB
```
*Image*, neboli obraz, je obraz disku nějakého toho systému, tzn. `overlay`.
Kdy uděláš nový kontejner, tzn. použiješ příkaz `run`, tak:
 - se vezmou informace z toho obrazu,
 - nad tím se vytvoří nová vrstva na to, co se uloží na tom kontejneru, když změníš nějaké soubory, a
 - pustí se nějaký program – v případě Ubuntu je výchozí `bash`.

Vypiš seznam všech právě běžících kontejnerů.
Pokud ti právě v jiném terminálu běží Ubuntu, uvidíš něco jako:

```console
$ podman ps
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

Jaký je rozdíl mezi kontejnerem a *image*?

 - *image* je obraz disku, nedá se změnit
 - *kontejner* je proces, běží s informacemi, které obsahuje image; navíc může zapisovat data, může se změnit; je instanci obrazu

*Image* můžou být sdílené mezi vícero kontejnery.
*Image* se mohu smazat jedině pokud neběží žádný kontejner, který je na nich založeny.
*Image* může být založen i na jiném *image*, slouží k tomu vrstvy, aby základ systému se dal sdílet a neplýtvalo se místem na disku.

Kontejnery můžeš klidně smazat pomocí: 
```console
$ podman rm <container-id>
```

Obdobně smažeš obrazy (ale funguje to jen pokud příslušný obraz nic nepoužívá):
```console
$ podman image rm <image-id>
```


## Webový server v kontejneru
```console
$ podman run -it httpd
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
$ podman run -d httpd
$ podman run -d httpd
$ podman run -d httpd
$ podman run -d httpd
```
Až je budeš chtít smazat, použij příkazy 
```console
$ podman stop <container-ip>
$ podman rm <container-ip> 
```

My se ale chceme podívat do webového serveru, takže spustíme v kontejneru `bash`:
```console
$ podman run -it httpd bash
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
$ podman run -p 8080:80 httpd
```

Port `80` je port na straně kontejneru, který se zpřístupní na portu `8080` na tvém (hostitelském) počítači.
Pokud tedy zadáš do svého prohlížeče adresu `localhost:8080` uvidíš stránku *It works!*.

Dále si zkusíme připojit nějaký vlastní adresář dovnitř souborového systému v kontejneru.
Zruš běžící `httpd` kontejner.

Nejdříve si vytvoř prázdný adresář a v něm soubor `index.html`.
```console
$ mkdir ~/container_htdocs
$ cd ~/container_htdocs
$ echo Ahoj! > index.html
```
A teď spusť kontejner s přepínačem `-v`:

```console
$ podman run -p 8080:80 -v ~/container_htdocs:/usr/local/apache2/htdocs/:Z httpd
```

> [note]
> Někdy na Fedoře je potřeba použít na konci cesty `:Z`,
> což ti zpřístupní soubory z domovského adresáře (standardně to totiž je
> z bezpečnostních důvodů zakázané).
> Nejde takto zpřístupnit *celý* domovský adresář – opět z bezpečnostních důvodů.

Když teď obnovíš stránku v prohlížeči, uvidíš **Ahoj!**.

Jak to funguje?
Pusť si opět Bash, ale se zpřístupněným adresářem, a v něm `findmnt`:

```console
$ podman run -p 8080:80 -v ~/container_htdocs:/usr/local/apache2/htdocs/:Z -it httpd bash
# findmnt
```

Zjistíš, že na cestě `/usr/local/apache2/htdocs` je *připojený* speciální
souborový systém, který dává kontejneru k dispozici adresář
`/home/petr/container_htdocs` z hostitelského systému.

Tenhle soubor můžeš dokonce změnit a změny se v hostitelském systému projeví.

```console
# echo ahoj ahoj>/usr/local/apache2/htdocs/index.html
```


## Dockerfile

Existují způsoby, jak udělat nový kontejner, do kterého místo standardního `index.html` dáš vlastní soubor.
Jeden z nich je pomocí souborů Dockerfile.

Vytvoř si soubor `Dockerfile`, nebo neutrálnější `Containerfile`.
(Soubor musí být přesně toto jméno, i s velkým písmenem na začátku).
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
$ podman build -t mujhttpd .
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
$ podman run -p 8080:80 mujhttpd
```

Takový hotový kontejner se dá pak nahrát na Docker Hub.

Jde to dokonce spustit na Windows, kde není linuxové jádro.
Vytvoří se v něm virtuální stroj s Linuxem, na němž je možné kontejnery bez problému spustit.
