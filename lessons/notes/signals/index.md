> [warning]
> Tohle jsou jen poznámky, ne materiály k samostudiu.

# Signály

Signál (angl. *signal*) je způsob, jak poslat „upozornění“ běžícímu procesu.
I když ten proces zrovna dělá něco jiného, může na signál zareagovat.
Vlastně, jak si ukážeme, na signál zareagovat *musí*.

Signály jsou docela minimalistické: obsahují jen jedno malé číslo,
druh signálu.
Větší „zprávy“ se musí procesům posílat jiným způsobem (třeba souborem,
rourou nebo přes síť), ale i v těchto případech se dá signálem upozornit
že si proces má nějakou větší zprávu přečíst.
Například `httpd` čte konfiguraci ze spousty souborů,
ale signálem se dá přimět, aby je všechny načetl znova.
Příkaz `systemctl reload httpd` dělá právě tohle: pošle HTTP serveru signál.


## Přijetí signálu

Když procesu pošleš signál, tak ten proces *přeruší* to co zrovna dělá
a signál zpracuje.
Co přesně se stane závisí na druhu signálu; možnosti jsou tyto:

* Ukončení (angl. *terminate*): proces se ukončí.
* Ignorování (angl. *ignore*): nic se nestane.
* Ukončení s chybou (angl. *core dump*): proces se ukončí. Pokud je na to
  systém nastaven, stav paměti ukončeného procesu se zapíše do speciálního
  souboru, aby bylo jednodušší zjistit co se v momentě ukončení zrovna dělo.
* Pozastavení (angl. *stop*): Proces se pozastaví.
  V shellu tohle dělá zkratka <kbd>Ctrl</kbd>+<kbd>Z</kbd>
* Obnovení (angl. *continue*): Pozastavený proces se znovu spustí.
* Zachycení (angl. *catch*): Po přijetí signálu se zavolá funkce,
  kterou proces nastavil.

Když se proces na signály předem nepřipraví, provede se nějaká výchozí akce.
Ta závisí na druhu signálu, ale nejčastější jsou první dvě: proces se ukončí.


## Druhy signálů

Každý druh signálu má své číslo a zkratkovité jméno.
Jejich seznam najdeš v manuálové stránce `signal(7)`, pomocí příkazu:

```console
$ man 7 signal
```

> [note]
> Pozor, `man signal` zobrazí stránku `signal(2)` – nápovědu pro funkci
> `signal`, nikoli signály obecně.
> To číslo je číslo „sekce“ manuálu; seznam sekcí najdeš v `man man`.

{# XXX seznam signálů #}

Každý signál má i své číslo, které se ale může na různých systémech
a procesorech lišit.
Lepší je proto používat jména – snad kromě čísla 9 pro `SIGKILL`.


## Posílání signálů

Příkaz na posílání signálů už znáš: je to `kill`.
Přepínačem pojmenovaným podle signálu můžeš vybrat, který signál pošleš.

Zkus si to!

V jedom terminálu zadej:

```console
$ sleep 1000
```

A ve druhém třeba:

```console
$ ls -a | grep sleep
 12345678 pts/3   00:00:00 sleep
$ kill -SIGABRT 12345678
```

Signál `SIGABRT` proces dostane když se snaží provést neplatnou operaci –
instrukci, které procesor nerozumí.
To je většinou hodně závažná chyba.
Kdybys při instalaci systému zadal{{a}} „automatické hlášení chyb“,
už by k vývojářům Fedory putovalo hlášení o tom, že v `/usr/bin/sleep`
z balíčku `coreutils` je chyba.
Pravděpodobně takhle ale virtuálku nastavenou nemáš, ale můžeš se aspoň
pomocí `abrt-cli list` podívat, že chyba byla zaznamenána.

Podobně si můžeš vyzkoušet jiné signály, třeba:

* `SIGINT` se posílá když zmáčkneš <kbd>Ctrl</kbd>+<kbd>C</kbd>.
* `SIGTERM` posílá `kill` když mu nezadáš jiný signál.
* `SIGKILL` (9) se posílá když zmáčkneš <kbd>Ctrl</kbd>+<kbd>\</kbd>;
  proces se „natvrdo“ ukončí.
  U tohoto signálu si procesy nemůžou nastavit jiné chování:
  proces se vždy ukončí.
* `SIGSTOP` se posílá když zmáčkneš <kbd>Ctrl</kbd>+<kbd>z</kbd>;
  proces se pozastaví.
  Podobně jako u `SIGKILL` si i u tohoto signálu si procesy nemůžou nastavit
  jiné chování.
* `SIGCONT` posílá příkaz `fg`; pozastavený proces se obnoví.


## Signály v Pythonu

To, že se přijetí signálu přeruší cokoli co proces zrovna dělá,
se dá přirovnat k výjimkám v Pythonu.
Koneckonců, výjimka `KeyboardInterrupt` je jen reakce Pythonu na signál
`SIGINT`: vzniká tak, že so Python na `SIGINT` připravil: řekl operačnímu
systému že pro `SIGINT` nemá ukončit proces,
ale má se zavolat funkce která způsobí výjimku.

Program v Pythonu můžeš „připravit“ na výjimky pomocí `try` a `except`.
Na to, aby se program připravil na přijetí signálu, slouží funkce
`signal` z modulu `signal`:

```python
import signal
import os
import time

print('Moje PID:', os.getpid())

def handler(sig_number, frame):
    print(f'Dostal jsem signál {sig_number}: {signal.strsignal(sig_number)}')

signal.signal(signal.SIGUSR1, handler)
signal.signal(signal.SIGUSR2, handler)
signal.signal(signal.SIGINT, handler)

while True:
    time.sleep(10)
```

Tento program reaguje na `SIGUSR1` a `SIGUSR2`, tedy např. `kill -SIGUSR1 $pid`.

Můžeš si ověřit, že pro `SIGKILL` a `SIGSTOP` funkce `signal.signal` selže.
A taky že předefinování `SIGINT` přenastaví chování
<kbd>Ctrl</kbd>+<kbd>C</kbd> – právě pro takové situace se hodí `SIGKILL`.



## Signály a démoni

Démoni systémových služeb často reagují na signály.
Když se podíváš na `systemctl cat httpd` a `systemctl cat sshd`,
uvidíš řádky jako:

* `KillSignal=SIGWINCH` – služba se ukončuje (`stop`) pomocí signálu
  `SIGWINCH`, po kterém se server ukončí „elegantně“: přestane přijímat
  nová spojení, ta co jsou otevřená zpracuje, a až potom se ukončí.
* `ExecReload=/usr/sbin/httpd $OPTIONS -k graceful` – tady se pro `reload`
  volá speciální příkaz, který ale ve výsledku nedělá nic moc jiného než
  poslání signálu `SIGUSR1`.
  Když server tenhle signál dostane, dokončí otevřená spojení, ale mezitím
  znovu načte konfiguraci a nová spojení začne zpracovávat s novým nastavením.
* `ExecReload=/bin/kill -HUP $MAINPID` – pro `reload` tohoto serveru se pošle
  signál `SIGHUP`, který server interpretuje jako žádost o to, aby znovu načetl
  nastavení.

Co dělá který signál, to se dozvíš v dokumentaci konkrétního serveru.
Signálů je málo, a tak se často „zneužívají“ – třeba `SIGWINCH` by se formálně
měl posílat když se změní velikost terminálu; programy jako `nano` nebo
`top` si po jeho přijetí zjistí novou velikost a překreslí výstup.
Ale Apache `httpd` ho používá pro ukončení.

