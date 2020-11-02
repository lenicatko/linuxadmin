
# 6. Linuxová administrace - mount

## Archivy

Jak fungují archivy? 

Představ si knížku. Knížka je v podstatě sekvence písmenek. 
Když se chceš podívat na určité místo, nemusíš listovat každou stránku zvlášť. 
Pro vyřešení tohoto úkolu najdeš prvně v knížce **Obsah**, který řekne například: *tato kapitola je na stránce 254*. 
Podobně je vše na disku uloženo jako sekvence bajtů. 
To, že jsou v něm nějaké soubory a adresáře, o nichž jsme se minule bavili, je jen iluze. 
Ve skutečnosti je na disku jedna dlouhá sekvence jedniček a nul. 
V tom by se ale vyhledávalo špatně, a proto má počítač v rámci svých jedniček a nul speciální informace, které podobně jako obsah v knížce říkají: *zde jsou adresáře, v nich jsou tyto soubory, jsou takto velké, takto se jmenují* atd.

Nejlepší si to bude ukázat na příkladu archivu `zip`. 

### ZIP

- https://commons.wikimedia.org/wiki/File:ZIP-64_Internal_Layout.svg

Obsahuje jednu dlouhou sekvenci informaci, na jejíž konci je *centrální adresář*. Ještě za ním je informace o počtu souborů v archivu.
V centrálním adresáři je řečeno: *1. soubor začíná na 0. pozici, 2. soubor začíná na 15. pozici* atd. 
Jsou v něm také informace o jednotlivých souborech: mj. jejich jména a jak jsou dlouhé.

Centrální adresář se nachází v zipu na konci, což má svůj důvod.
Můžeš vzít normální soubor, na konec mu přilepit `zip`, a soubor bude pořád fungovat i jako `zip` i jako soubor / program.

Zip je, zjednodušeně řečeno, souborový systém v souboru.
Soubory v zipu se dají docela jednoduše číst. 
Podíváš se, kde začít a jak dlouho číst, a je to.

Existují i takové formáty souborových systémů v souboru, které jsou dělané nejen na čtení souborů, ale dají se do nich i zapisovat.
To teoreticky jde i v zipu, ale problém vzniká, pokud soubor, který chceš do archivu nahrát, je větší než místo, které je v něm k dispozici.

Je to podstatná myšlenka, zkus se nad ní pořádně zamyslet.
Je to jak s tou knížkou: na určitém, pevně daném místě se nachází obsah, který ti říká, kde se nachází jednotlivé kapitoly. 


## Souborový systém

A protože v UNIXu je všechno soubor, i souborové systémy jsou v souboru. 
Vzpomeň si na ukázku z minulé lekce, jak je myš reprezentovaná souborem. 
Když se soubor otevřel pro čtení, každý pohyb myši se zobrazil v terminálu jako spousta binárních symbolů.

Podívej se do adresáře `/dev/disk` - zde uvidíš soubory, které reprezentují tvůj disk, jak jsme si ukázali minule. 

Soubory pro celý disk:
```console
$ ll /dev/disk
drwxr-xr-x. 2 root root 400 Nov  7 09:41 by-id
drwxr-xr-x. 2 root root  80 Nov  7 09:34 by-partuuid
drwxr-xr-x. 2 root root 120 Nov  7 09:34 by-path
drwxr-xr-x. 2 root root 160 Nov  7 09:41 by-uuid
```
Uvnitř každého souboru je souborový systém. I zde jsou, podobně jako v zipu, jednotlivé soubory a legenda obsahující informace o jejich vlastnostech.

Druhů souborových systémů je spousta.

Na flashce máš pravděpodobně prastarý DOSový formát **FAT32**, který je tak starý, že mu už vypršely patenty, a díky tomu ho může používat kdokoliv. :) 
Na disku, pokud je to Linux, se většinou používá **EXT4**, na Windows **NTFS**.



### mount

Pusť příkaz `mount`. Jeho výstup bude celkem dlouhý.
```console
$ mount
...
název  kde je umístěn
|        |       typ souborového systému
|        |        |        
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=8002676k,nr_inodes=2000669,mode=755)
...
```
`mount` říká, kde jsou připojené jednotlivé souborové systémy.
- např. souborový systém `devtmpfs` je připojen na `/dev`

Když připojíš flashku a spustíš `mount` znovu, přibude na konci nový řádek. 
A říká: 
 - `/dev/něco` s obsahem celé flashky je přípojen na `/run/media/něco`

což znamená, že v tomto místě najdeš souborový systém své flashky. 
Systém interpretuje souborový systém na flashce a zpřístupní ti ho formou adresářů a souborů k prohlížení a měnění.
Každá flashka, každé záznamové medium funguje trochu jako zip archiv: je v něm víc souborů naskládaných za sebou s *obsahem*, který je umožňuje identifikovat a měnit.

Když se chceš podívat, jaké disky máš k dispozici, je na to příkaz `lsblk`. 
Flashka níže je připojená jako `sdb`.
```
$ lsblk
NAME                             MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
nvme0n1                          259:0    0 238.5G  0 disk  
├─nvme0n1p1                      259:1    0     1G  0 part  /boot
└─nvme0n1p2                      259:2    0 237.5G  0 part  
  ├─fedora_dhcp--0--123-root     253:0    0    50G  0 lvm   
  │ └─luks-e28e6e1b-f122-43d6-99d7-049dc034656f
  │                              253:2    0    50G  0 crypt /
  ├─fedora_dhcp--0--123-swap     253:1    0   7.7G  0 lvm   [SWAP]
  └─fedora_dhcp--0--123-home     253:3    0 179.8G  0 lvm   
    └─luks-0147c4fc-0301-4d1e-9cba-b0c17c700b48
                                 253:4    0 179.7G  0 crypt /home
sdb                                8:16   1  29,3G  0 disk 
└─sdb1                             8:17   1  29,3G  0 part  /run/media/guest/pendrive
```

Když flashku odpojíš, `sdb` zmizí.

Pevný disk může být rozdělen na části, z nichž každá smí mít jiný souborový systém. 
Můžeš mít na počítači nainstalované zároveň operační systémy Windows a Linux, které používají jiné souborové systémy, ale dokud každý kouká na svoje místo na disku, proběhne to bez problému.

Disk může být i zašifrovaný, což funguje tak, že Linux vytvoří ještě jeden soubor s obsahem celého disku. 
Když někdo zapíše něco na disk, soubor s obsahem disku se zašifruje a uloží na ten opravdový disk. 
Když chceš potom z něho číst, jde to opačným směrem.
Jsou to abstrakce dány na sebe. 
**Soubor znamená něco, na čem můžeš provádět operace jako čtení a zápis.** 

Jak se čte soubor?
Systém se podívá do "legendy", kde je řečeno, kde se na disku se soubor nachází. 
Až ho najde, zeptá se přímo ovladačů pevného disku a pošle tam elektrické impulzy, aby se přečetly nuly a jedničky, a pak to reprezentuje jako souborový systém. 
Mezi tím může být vrstva šifrování, nebo vrstva, která komunikuje s jiným serverem. 
Například přístupy v univerzitní síti vypadají tak, že domovská složka je sdílená někde na serveru, a je úplně jedno, na který z dostupných počítačů se přihlásíš. 
Vždy, když chceš něco přečíst nebo zapsat, probíhá mezi počítačem a serverem síťová komunikace - pro tebe se to ale celou dobu chová jako normální soubor.

Souborový systém se připojuje na určité místo v adresářovém systému (např. `/` nebo `/run/media`) - to místo se jmenuje *mount point* (přípojný bod). 
Tím říkáš: vše, co je pod danou cestou, se bude číst jako daný souborový systém.

Když provedeš příkaz `mount`, dostaneš informace o všem, co je přípojené na jaký mount point.

Ve výpisu `lsblk` můžeš uvidět ještě dvě zajímavůstky.
- Část pevného disku může být uvedená jako `/boot` nebo `/efi`.
    - část disku určená pro to, aby se z ní spustil operační systém.
    - po zapnutí počítače se BIOS podívá na začátek pevného disku a načte instrukce pro provedení dalších kroků
    - z historických důvodů je tato část spíš malá (disky nebývaly velké) - efi bývá větší než boot.
- Může tam být i `[SWAP]`.
    - část disku určená pro zápis mezivýsledků běžících programů z paměti
    - odkládají se věci, které není potřeba mít zrovna v paměti (například zapomenutá instance prohlížeče...)


## gparted
Program, který umí modifikovat struktury na disku - dávej velký pozor při spouštění.
Práva na spuštění má pouze superuživatel, protože nesprávnými změnami může dojít k zrušení systému.
Po spuštění bez argumentů `gparted` uvidíš grafické zobrazení rozdělení svého disku na oddíly a kolik z nich je zaplněno daty. 
Tento program můžeš použít na smazání oddílů a vytvoření několika nových - menších, nebo jiné uspořádání místa. Nedělej to, dokud si nejsi stoprocentně jistý/á, že víš, co chceš provést :).


## ISO
Jiný druh souborového systému je ISO, který byl vyvinutý na to, aby soubory bylo možno dávat na CD. 
Nemůžeš je proto modifikovat, jen z nich číst.

*Další část vyžaduje stažený .iso soubor (v našem případě s Fedorou, ale obsah není důležitý, může to být cokoli jiného).*

Podívej se, co máš v adresáři `/mnt`. Nemělo by tam být nic.
Vytvoř v něm soubor (v kořenovém adresáři musíš mít právo superuživatele, proto použij `sudo`). 
```console
$ ls /mnt
$ sudo touch /mnt/soubor
```
Adresář `/mnt` slouží k připojení různých zařízení jako flashky, disky atp.

Připojíme si souborový systém z ISO souboru tak, jako bychom vložili CD do počítače. 

Stažený soubor připojíš takto:
```console
sudo mount cesta/k/souboru.iso /mnt
```

Nyní můžeš otevřít složku `/mnt` a najdeš v ní další adresáře a soubory.
```console
$ ls /mnt
EFI   images   isolinux
```

Pokud máš v `/mnt` cokoli jiného před připojením (jak náš `soubor`), pak se k těmto souborům nyní nedostaneš, protože data směřují někam jinam - vhodné je např. vytvořit složku v `/mnt` a připojit zařízení tam.

Když se teď podíváš na výstup příkazu `mount`, na posledním řádku bys měl{{a}} najít připojené iso.

Pro odpojení připojeného prostoru napiš:

```console
$ umount /mnt
```

Pozor, když se o to pokusíš přímo v adresáři `/mnt`, operace nebude úspěšná.
Pro odpojení se musíš přepnout mimo adresář `/mnt`.

Úplně nakonec můžeš po sobě uklidit a smazat soubor.

```console
sudo rm /mnt/soubor
```

Souborový systém si můžeš představit jako Pythonní třídu. 
Je to objekt, který má nějakou funkčnost (umí číst soubory, zapisovat, zjistit, kdo vlastní soubor atd.) a má nějaká data (na disku, v souboru, v paměti...). 
Adresář `/proc` má například soubory obsahující hodnoty vypočítané v aktuálně běžících procesech. Nejsou to fyzické soubory na disku.
Adresář `/tmp` je dnes většinou jen v paměti, jeho obsah se ani neukládá na disk. 

Pokud tě zajímá, jak doopravdy funguje souborový systém, existuje knihovna [python-fuse](https://github.com/libfuse/python-fuse) (*file system in user space*), s jejiž pomoci si můžeš napsat Pythonní třídu, ve které nadefinuješ souborové operace, a namountovat si ji do systému.

### Vychytávky
Při náhlém odpojení počítače/souboru je možné, že se souborový systém poškodí. 
Je možné se jej pokusit opravit pomocí `fsck`.
Detaily najdeš v manuálové stránce toho programu.

Můžeš mít několik fyzických disků spojených dohromady jako jeden souborový systém. Tímto se jednoduše zvětšuje nebo zmenšuje kapacita disku.
Aby fungovaly jako jeden souborový systém, je nad nimi vrstva, která funguje jako jeden disk, a podle toho na které místo přistoupíš, bude vědět, na který konkrétní disk se má obrátit.

Takto můžeš dělat zálohy, že když něco zapíšeš na disk, rozdistribuuje se to na dvě nebo víc míst. 
Nejčastěji používaná technologie k duplikování dat se jmenuje [RAID](https://en.wikipedia.org/wiki/RAID). 
Tuto bys měl{{a}} rozhodně znát, pokud se chystáš pracovat jako systémový administrátor.



