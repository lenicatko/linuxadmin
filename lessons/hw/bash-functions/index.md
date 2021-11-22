# Úkoly - Spouštění procesů v Bashi

*Neboj se experimentovat. Správným řešením těchto úkolů si systém nerozbiješ a kdyby bylo hodně špatně, virtuální počítač si můžeš vždycky připravit nový.*

### Bashová funkce na pozadí

Na lekci jsme si říkali, že jedna z výhod funkcí v Bashi je to, že se vykoná v procesu Bashe, který ho spouští. To ale není pravda úplně vždycky.
Zkus vysvětlit rozdíl mezi těmito příkazy:

```console
$ pust-ps () { ps; } ; pust-ps
    PID TTY          TIME CMD
2496176 pts/8    00:00:00 bash
2496266 pts/8    00:00:00 ps
```

```console
$ pust-ps () { ps; } ; pust-ps &
[1] 2496277
...$     PID TTY          TIME CMD
2496176 pts/8    00:00:00 bash
2496277 pts/8    00:00:00 bash
2496278 pts/8    00:00:00 ps

```

Nakresli pro oba příkazy diagramy toho, jak se jednotlivé procesy navzájem spouštějí.
(Ukázky diaramů jsou i v [materiálech](https://naucse.python.cz/2020/linux-admin/linuxadmin/process-execution/). Stačí je samozřejmě jen načrtnout tužkou.)


### Výbušná změť symbolů

(Tohle zadání navazuje na předchozí úkol. A je asi dost složité; klidně přeskoč.)

Existuje jeden známý příkaz, který *nespouštěj*. Tvým úkolem bude zjistit, co dělá.
To můžeš zkusit udělat třemi způsoby:

* Najdeš to na internetu, čímž se ochudíš o to, že na řešení přijdeš sama :)
* Příkaz spustíš. To nedělej: příkaz je známý tím, že umí „shodit“ celý systém. Některé novější systémy se proti tomu umí více či méně bránit, ale stejně – nepokoušej štěstí.
  (Na druhou stranu – nestane se nic horšího než budeš muset vypnout virtuální počítač.)
* Projdeš si poznámky o tom, co v Bashi dělají různé symboly, a příkaz dešifruješ.

Příkaz je `:(){ :|:& };:` – s tím že znak`:` není v Bashi speciální, funguje stejně jako třeba písmeno `f`. Nakresli diagram toho, jak se po spuštění tohoto příkazu spouštějí procesy.

#### Nápověda/doplnění

Velice podobný program v Pythonu, který *taky nespouštěj*, je:


```python
# Nespouštět – počítač zamrzne!
import os
while True:
    os.fork()
```
