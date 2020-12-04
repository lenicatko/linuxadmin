# Virtualbox - nastavení sítě

Aby ses mohl{{ a }} "dovolat" (tzv. připojit se na port) mezi virtuálním a hostitelským počítačem s VirtualBoxem, budeš k tomu pravděpodobně potřebovat trochu upravit nastavení sítě. V tomto návodu si ukážeme jak.

1. Spusťte Virtualbox
1. Ještě s vypnutým virtuálním strojem otevři nastavení. Klikni na název virtuálního počítače (v mém případě se jmenuje `fedora`) → *Nastavení* → *Síť* → *Karta 1*

- **Připojena k** změnit na **Síťový most**
- **Název** vyberte síťovou kartu, případně wifi kartu. Budeš-li jich tam mít více, postupně je vyzkoušej. Tu správnou poznáš podle toho, že má ve virtuálním PC budeš mít přiřazenou IP adresu, která bude začínat stejně, jako i tvůj hostitelský počítač (to je ten, na kterém spouštíš Virtualbox).
  Jak ji poznat se dozvíš v následujícím bloku.

  {{ figure(
    img=static('virtualbox-vm-config.png'),
    alt='Screenshot s očíslovaným postupem, kam klikat v nastavení',
  ) }}

## Ta správná síťová karta
Jak ale zjistit, která síťová karta je ta správná? Pokud to neví, otevři si příkazovou řádku Windows (klikni na nabídku *Start* → napiš *cmd* a potvrď <kbd>Enter</kbd>. Spusť příkaz `ipconfig` bez dalších parametrů a měl{{a}} bys vidět podobný výstup:
```console
C:\Users\uzivatel>ipconfig

Windows IP Configuration


Ethernet adapter Ethernet 4:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :

Ethernet adapter Ethernet 2:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :

Ethernet adapter Připojení k místní síti:

   Link-local IPv6 Address . . . . . : fd84::fd7e:9520:3d65:d139%10
   IPv4 Address. . . . . . . . . . . : 192.168.0.114
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.0.1
```

Z výpisu zjistíme, že počítač dostal přiřazenou IP adresu `192.168.0.114`. Hledáme teda takovou síťovou kartu, která nám i ve virtuálním počítači přiřadí podobnou adresu (správně řečeno: IP adresu ze stejného rozsahu).

> [note]
> Hledámte tedy síťovou kartu, která nám ve virtuálním počítači přiřadí IP adresu, která v tomto případě začíná `192.168.0.<číslo>` (poslední číslo za tečkou rozlišuje jednotlivé počítače na stejné síti a proto se bude lišit).

# Kontrola IP adresy ve virtuálním počítači

Spusť virtuální počítač (pokud ještě nemáš) a v terminálu spusť příkaz `ifconfig`, ze kterého můžeš vyčíst aktuálně přidělenou IP adresu (na obrázku to je `192.168.0.145`).

V případě, že adresa začíná jinak, než jakou očekáváš z předchozího kroku, znovu si otevři nastavení tvého virtuálního počítače a vyzkouše jiný síťový adaptér.

Aby si toho všiml i virtuální počítač, je třeba "odpojit a znovu připojit síťový kabel", což se dá udělat i přímo z prostředí Fedory.

1. Klikni do pravého horního rohu na skupinu ikon (nachází se zde mj. symbol pro vypnutí).
1. Klikni na rozbalovací menu u *Drátové je připojeno*.
1. Klikni na *Vypnout*. Tím dojde k "odpojení" síťového kabelu.
1. Znovu se doklikej na to samé místo a tentokráte vyber *Aktivovat*, což způsobí znovupřipojení kabelu a tím i k načtení nové IP adresy.


  {{ figure(
    img=static('fedora-network-reconnect.png'),
    alt='Screenshot s postupem, jak se ve Fedoře připojit/odpojit k síti',
  ) }}


# Jak poznám, že to funguje?

Na virtuálním počítači si otevři terminál a do něj zadej:

```console
$ nc -l -p 5000
```


Na svém **hostitelském počítači** si otevři ideálně Firefox (s jinými prohlížeči to nefunguje tak hezky) na do adresního řádku napiš IP adresu tvého virtuálního počítače (v této ukázce to je `192.168.0.145`) společně s portem `5000`:

```console
192.168.0.145:5000
```

Ta stránka se sice bude tvářit, že se nechce načíst, ale i přesto poslal tvůj prohlížeč do virtuálního počítače požadavek na spojení, které uvidíš v otevřeném terminálu. Zobrazilo se ti něco podobného? Výborně!

```console
$ nc -l -p 5000
GET / HTTP/1.1
Host: 192.168.0.145:5000
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:83.0) Gecko/20100101 Firefox/83.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: cs,sk;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Pragma: no-cache
Cache-Control: no-cache
```

Běžící program `nc` vypneš běžným způsobem, tj. pomocí <kbd>Ctrl</kbd>+<kbd>C</kbd> (pokud se neukončí dřív sám).
