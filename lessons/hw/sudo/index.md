# Úkoly – sudo

## Jak se pouští sudo

Nakresli diagram toho, jak se pouští procesy pro příkaz `sudo ps`:

```console
$ sudo ps
    PID TTY          TIME CMD
2501386 pts/9    00:00:00 sudo
2501388 pts/9    00:00:00 ps
```


## Sudo zapomíná

*(`sudo` ti dává oprávnění, ale i zodpovědnost. Nauč se před každým použitím `sudo` přečíst spouštěný příkaz a zamyslet se, jestli ho opravdu chceš spustit. Ale opět, neboj se experimentovat – na virtuálním počítači není co zkazit.)*

Co by měl dělat tento příkaz?

```console
ll /var/db/sudo/lectured/
```

Příkaz ale pod běžným uživatelským účtem nefunguje; potřebuješ `sudo ll /var/db/sudo/lectured/`. To ale taky nefunguje. Proč? Jak to spravit?

* Jaké soubory jsou v `/var/db/sudo/lectured/`? Komu patří? Jak jsou velké?
* Jaké soubory jsou v `/var/run/sudo/ts`? Komu patří? Jak jsou velké? Kdo k nim má jaká práva?
* Jak by vypadal příkaz, který smaže „tvůj“ soubor v `/var/run/sudo/ts`?

Když spustíš příkaz `sudo`, zeptá se tě na heslo. Když ho spustíš podruhé (ve stejném terminálu), už se neptá – pamatuje si, že jsi heslo před chvilkou zadala.

Vyzkoušej si ale, že tohle nefunguje s příkazem pro smazání „tvého“ souboru v `/var/run/sudo/ts`. Když ho pustíš několikrát za sebou, `sudo` se vždycky znovu zeptá na heslo.

* Proč?
* Zadej tyhle příkazy:
  * Smaž „svůj“ soubor z `/var/db/sudo/lectured/`.
  * Smaž „svůj“ soubor z `/var/run/sudo/ts`.
  * Spusť `sudo echo`.
 
  A odpověz:
  * Co je u `sudo echo` jinak?
  * K čemu slouží soubory ve `/var/db/sudo/lectured/`?
