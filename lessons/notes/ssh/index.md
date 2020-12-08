# SSH - vzdálená administrace

Dnes se podíváme na to, jak spravovat servery přes síť.

Po troše nastavování to bude podobné jako většina toho, co jsme zatím dělali.
Jen bez grafických okýnek – připojíme se čistě přes příkazovou řádku.

> [note]
> Samozřejmě je možné sdílet obrazovku; nástroje na to existují i na Linuxu.
> Na serverech ale často nebývá grafické prostředí vůbec nainstalováno,
> protože vykreslování celkem zbytečně zabírá výkon.
> A každý software navíc taky zvyšuje šanci, že se na systému vyskytne
> bezpečnostní chyba.

Technicky to není složité: terminál je jen soubor, do kterého se zapisuje to,
co napíšeš na klávesnici a ze kterého lze číst výstup shellu a spouštěných
programů.
Dokonce i vychytávky jako barvičky nebo hrátky s kurzorem (které používá
editor `nano` nebo skrolovátko `less`) se dají posílat jako text.

Nejsložitější na celé věci je komunikaci *zabezpečit*.
Součástí posílaného textu – toho, co napíšeš na klávesnici – je i
přihlašovací heslo.
Kdysi, když se administrátorům počítačových sítí dalo věřit (protože stejně
měli přístup ke všem počítačům ve firemní či univerzitní síti), se informace
posílaly „jen tak“.
Dnes, když informace putují internetem přes spoustu zařízení, z nichž každé
si může posílané informace přečíst, se ale musí šifrovat.
Díky efektivním nástrojům se šifruje všechno, co se pošle, nikoli jen hesla.

> [note]
> Nezabezpečené připojení se stále používají ke konfiguraci
> zařízení, které jsou k ovládacímu počítači připojeny přímo,
> ne přes síť/internet.
> Máš-li zkušenosti s MicroPythonem, vzpomeň jak probíhala komunikace
> s „destičkou“.

K zabezpečenému připojení použijeme protokol SSH (*secure shell*).
Podobně jako HTTP existuje server – služba SSHD – který je potřeba nastavit
a spustit.
K počítači, na kterém běží SSHD, je možné se vzdáleně připojit.

Na druhé straně budeme potřebovat *klienta*, program, který se k SSH serveru
umí připojit.
Na Linuxu a macOS existuje program `ssh`, na Windows použijeme grafický
`puTTY`, který si nainstaluj ze [stránek PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

> [note]
> PuTTY je k dispozici i pro ostatní systémy. Na Fedoře je v repozitářích,
> můžeš si ho nainstalovat přes `sudo dnf install putty`.


## Instalace OpenSSH

SSH server není na Fedoře v balíčku `sshd`.
Na rozdíl od `httpd` totiž takových serverů bylo historicky více.
Dnes se většinou používá OpenSSH, který si nainstaluj následovně:

```console
$ sudo dnf install openssh-server
```

Služba samotná už se ovšem jmenuje `sshd`:

```console
$ sudo systemctl start sshd
```

Možná budeš potřebovat i otevřít firewall.
Protokolu `ssh`, přes který budeme komunikovat, kterému přísluží port 22:

```console
$ sudo firewall-cmd --add-service=ssh
```


## Připojení

Teď se můžeš přihlásit!
Zkus to nejdřív z virtuálního počítače.
Nebude to zatím správa vzdáleného stroje, ale bude to jednodušší:

```console
$ ssh localhost
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is SHA256:dzya/6SYp058Apl4++46lt8IMPNTuIAznjUGlrI1ymU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
petr@localhost's password: 
Last login: Mon Dec  7 15:41:10 2020 from 192.168.122.1
[petr@localhost ~]$ 
```

Program `ssh` tě po připojení upozorní, že k tomuto počítači se ještě nikdy
nepřipojoval a zeptá se, jestli je to opravdu počítač ke kterému se chceš
připojit.
Kdyby ses totiž připojoval{{a}} přes internet, někdo „mezi“ tvým a cílovým
počítačem by se mohl docela jednoduše vydávat za server,
na který se chceš připojit.
Proto až „o něco půjde“, vždycky si ověř že daný „otisk“ – v mém případě
`SHA256:dzya/6SYp058Apl4++46lt8IMPNTuIAznjUGlrI1ymU` – odpovídá počítači,
ke kterému se připojuješ.
Za chvíli to uděláme; teď u `localhost` prostě napiš `yes`.
Dále zadej heslo a jsi připojen{{a}}!

Klient `ssh` se standardně připojuje pod tvým uživatelským jménem.
Kdybys na vzdáleném počítači měla účet pojmenovaný jinak, nebo se chtěl{{a}}
odděl uživatelské jméno od jména serveru znakem `@`.
Máš-li stále účet `test1`, zkus to – ale pak se zase odhlaš,
ať můžeme pokračovat:

```console
$ ssh test1@localhost
[test1@localhost ~]$ exit
$
```

Tentokrát se `ssh` na bezpečnostní ověření neptá.
Jakmile ses totiž k tomuto počtači připojil{{a}}, `ssh` si adresu `localhost`
spojil s daným klíčem.
Zaznamenal si to do souboru `~/.ssh/known_hosts`:

```console
$ cat ~/.ssh/known_hosts
localhost ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBC9n7rIp76I0uxvicyZyRzUeh2PGElLwLV2Vw/AtrxOQkXWK/yfY/n5WlFiRblrOHf01tiH52C/zy1CiVLgl+bc=
```

> [note]
> Klíč v souboru je delší a bezpečnější verze „otisku“, na který se tě `ssh`
> ptal.
> Nebylo by ale praktické nutit člověka, aby porovnával takhle dlouhý klíč.

Kdyby se klíč na serveru změnil, může to znamenat dvě věci:

* Administrátor heslo změnil, třeba kvůli bezpečnosti.
* Nepřipojuješ se k počítači, ke kterému ses připojoval{{a}} minule – a
  kdyby ses připojil{{a}}, pravděpodobně bys novému počítači jako první věc
  poslala přihlašovací heslo.

První variantu si můžeš vyzkoušet příkazem.
Nebudu ho vysvětlovat do detailů; stačí vědět že vytvoří nový šifrovací klíč
pro SSHD:

```console
$ sudo ssh-keygen -f /etc/ssh/ssh_host_ecdsa_key -N '' -t ecdsa
```

Zkus se pak připojit:

```console
$ ssh localhost
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
...
Host key verification failed.
```

Když takovou zprávu uvidíš, ověř si že jde o první variantu výše – pokud od
administrátora např. nepřišel důvěryhodný mail, tak mu radši třeba zavolej.

Je-li vše v pořádku, smaž se souboru `~/.ssh/known_hosts` starý záznam
a připoj se znovu.


## Vzdálené připojení – ssh

A teď se připoj vzdáleně!
Máš-li na svém *opravdovém* počítači Linux nebo macOS, zjisti IP adresu
virtuálního počítače (např. příkazem `ifconfig`), na opravdovém počítače
si otevři příkazovou řádku a zadej <code>ssh <var>jmeno</var>@<var>adresa</var></code>:

```console
$ ssh petr@192.168.122.133
The authenticity of host '192.168.122.133 (192.168.122.133)' can't be established.
ECDSA key fingerprint is SHA256:0LNQwY8++rCDzynAAQx2MNP9OMupnpsps0loay8P3NM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? 
```

Tentokrát si otisk opravdu ověř.
Na virtuálním počítači zadej:

```console
$ ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
256 SHA256:0LNQwY8++rCDzynAAQx2MNP9OMupnpsps0loay8P3NM root@localhost.localdomain (ECDSA)
```

A zkontroluj, že otisk sedí.
Pro plnou bezpečnost je potřeba zkontrolovat každé písmenko,
ne jen začátek nebo konec.

Je li vše OK, zadej `yes`, napiš svoje heslo – a máš vzdálený přístup
k virtuálnímu počítači!


## Vzdálené připojení – PuTTY

Další SSH klient je grafický PuTTY.
Máš-li ho nainstalovaný, spusť ho a do políčka *Host Name (or IP address)*
napiš IP adresu virtuálního počítače.
*Port* nech na 22 a *Connection Type* na SSH.

{{ figure(
    img=static('putty-config.png'),
    alt='Diagram příkazu `cat`',
) }}

Pak klikni na *Open*, zadej své jméno a heslo – a máš vzdálený přístup
k virtuálnímu počítači!


## Jméno serveru

Podobně se připojit k jakémukoli jinému serveru, kam máš přístup.

Aby se servery nepletly, je dobré společně se službou SSHD nastavit i jméno
počítače.
To se dělá pomocí příkazu `hostname`, který bez argumentu jméno zobrazí
a s argumentem ho nastaví:

```console
$ hostname
localhost
$ sudo hostname virtualka
[sudo] heslo pro petr: 
$ hostname
virtualka
```

Programy většinou jméno počítače čtou jen při spuštění.
Když se teď *znovu přihlásíš* (přes SSH, nebo otevřeš nový terminál),
ve výzvě najdeš nově nastavené jméno.

Ale jako spousta jiného nastavení se tahle změna zruší když počítač vypneš
a zase zapneš.
Trvalé nastavení – tedy to, podle kterého se jméno nastavuje při startu
systému – je v souboru `/etc/hostname`.

```console
$ cat /etc/hostname
localhost.localdomain
[petr@localhost ~]$ echo virtualka | sudo tee /etc/hostname
virtualka
```


## SSH klíč

Zatím ses k virtuálnímu počítači přihlašoval{{a}} heslem.
To ovšem není nejbezpečnější a nejkomfortnější varianta.

Lepší je si vytvořit soubor s *klíčem*.
Vlastně to jsou soubory dva: *veřejný klíč*, který můžeš s kýmkoli sdílet,
a *soukromý klíč*, který je naopek jen tvůj – funguje jako heslo.

Svůj *veřejný klíč* dáš na server, ke kterému se budeš připojovat,
a nastavíš že mu server bude „věřit“.

Pokud pracuješ na Windows, vytvoř si klíče na svém virtuálním počítači.
Jinak to udělej na tom opravdovém.

Klíče se tvoří příkazem `ssh-keygen`.
Ten je normálně ukládá jako `/home/petr/.ssh/id_rsa`,
kde je potom očekává `ssh`.
Na otázku, kam je chceš uložit, odpověz <kbd>Enter</kdb> – tím vybereš
výchozí nastavení.

Pokud soubor už existuje, znamená to že klíč už máš.
Nevytvářej si nový; na otázku `Overwrite (y/n)?` odpověz `n`.

Další otázka je na heslo, kterým můžeš svůj soukromý klíč chránit.
Je tu pro situaci, kdy si nezamkneš počítač a někdo ti soubor s klíčem
ukradne – pokud uhodne heslo, nebo pokud není nastavené,
může se pak přihlašovat pod tvým účtem.

```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/petr/.ssh/id_rsa): 
Created directory '/home/petr/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/petr/.ssh/id_rsa
Your public key has been saved in /home/petr/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:RihRspvWIrPti3X1x/HAYZ0N5CcqOTp1aaolCw7qZP4 petr@virtualka
```


## Nastavení SSHD

XXX


