# Cron

Najdi si na systému nějaký zvuk a přehraj si ho:

```console
$ locate .mp3
$ locate .ogg
$ mplayer <cesta/k/některému/zvuku>
```

Pokud nemáš nainstalovaný `mplayer` z RPMFusion, použij místo toho
`ogg123` – ten lze nainstalovat ze systémových repozitářů.


## Přehráváme zvuky z příkazové řádky

Proč se to hodí přehávat z příkazové řádky?

Protože to můžeme **automatizovat**. Třeba spouštět stále dokola.

```console
$ while ; do mplayer <cesta/k/tvemu/zvuku>; done
```

(až to budeš chtít zrušit, zmáčni `Ctrl+C`).


## Spouštění zvuku podle času

Taky si vyzkoušíme, jak spustit zvuk v zadaný čas.

O spouštění programů podle času se stará démon `cron`.

```console
$ man crontab
```

Tahle manuálová stránka je o příkazu `crontab`.
Nedočteme se tu ale, jak vypadá konfigurace.

`man` má více *kapitol*.
V kapitole `1` jsou popsané příkazy: jak se zadaný příkaz spouští a jaké má parametry.
V kapitole `5` jsou popisy konfiguračních souborů.
Občas se stane, že je ve dvou kalpitolách sekce stejného jména, jako v případě
`crontab(1)` a `crontab(5)`.
Většinou je to zmíněno v sekci `SEE ALSO`.

Sekci o `crontab` z páté kapitoly zobrazíš pomocí:

```console
$ man 5 crontab
```

Příkaz `crontab` je tedy nástroj, kterým můžeš editovat soubor
`/etc/crontab` nebo `/var/spool/cron/<jméno uživatele>`.
Na rozdíl od normálního editoru po uložení zkontroluje,
jestli je soubor v pořádku.

```console
$ crontab -e
```

Otevře se nám editor, do kterého napíšeme:

```console
* * * * * mplayer /desta/k/zvuku
```
Každá z hvězdiček vyjadřuje jinou část času (hodina, minuta, ...). Abychom si to ale nemuseli pokaždé číst znova, doporučuji si na první řádek dá komentář, co která pozice říká.

```console
# min hod den mesic den_v_tydnu prikaz
```

Když to uložíte a zavřete, tak by se měl nainstalovat.

A teďka vám to každou minutu přehraje zvuk. Jupí! :)

Toto nastavení vydrží i po vypnutí počítače. Zkrátka dokud si to zas nevypneš.

Teďka si zkusíme snížit frekvenci, jak často se nám to bude přehrávat. Třeba každou 50. minutu.

```
$ crontab -e
50 * * * * mplayer /cesta/ke/zvuku
```

`*` znamená každou/každý

Každý druhý den se napíše takto:
```
$ crontab -e
0 0 /2 * * mplayer /cesta/ke/zvuku
```

Znamená 2. den v měsíci
```
$ crontab -e
0 0 2 * * mplayer /cesta/ke/zvuku
```

[nakopírovat kus z EXAMPLES]

Cron může posílat emaily. Email je věc starší, než je internet. Původně se emaily posílaly v rámci jednoho počítače

Příkazem `mail` si je můžeme zobrazit. Po čase je dobré se podívat, jestli je nám `cron` neposílá, což by nám po čase mohlo 
