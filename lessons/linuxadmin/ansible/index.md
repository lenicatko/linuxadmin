# Automatizace administrace – lehký úvod k Ansible

Představ si, že jsi {{gnd('systémový administrátor', 'systémová administrátorka')}}
na serverové farmě.
Staráš se o 500 počítačů. Každý z nich má nastavený časovač na zálohy.
Třetina má nastavený webový server, další třetina je velký cluster databází a poslední třetina spravuje e-maily.
Tvůj úkol je zajistit, aby se na všech těch serverech záloha dělal{{a}} dvakrát častěji.

Co uděláš?
Připojíš se přes `ssh` k prvnímu serveru, pozměníš nastavení časovače
a pustíš `systemctl daemon-reload`.
Hurá! Jen 499 a můžeš jít domů!

Jiná varianta je, že si napíšeš v Bashi cyklus, který se sám připojí ke
každému počítači a interval změní.
Budeš mít z toho skvělý pocit a práce bude odvedená během chvilky.
...Jenže pak zjistíš, že u serveru číslo 54 se to jaksi nepovedlo.
Což znamená, že bys měl{{a}} zjistit,
jestli to u ostatních serverů dopadlo dobře, nebo se to taky nepovedlo.

Teď si představ, že na tvé farmě právě jeden z tvých serverů shořel.
Nakoupíš nový počítač a chceš aby fungoval úplně stejně jako ten předchozí.
Potřebuješ určitou verzi databáze s určitou konfigurací, a taky tam potřebuješ
nastavit časovač včetně poslední změny frekvence záloh.

Změny konfigurace na vícero strojích se dnes už nedělají tak,
že jdeš k počítači, přímo na něm upravíš konfigurační soubor
a následně restartuješ příslušnou službu.
Používají se na to systémy, kde si do souboru napíšeš,
jak má tvůj systém vypadat a pustíš program,
který se připojí na těch 500 serverů a každý nastaví podle toho seznamu.
Kdž pak server shoří, vyměníš ho za jiný a pustíš ten program znova.

Velká výhoda takového řešení je to, že se všechno nastavení dá ukládat do Gitu,
takže na něm můžeš spolupracovat v týmu – stejně jako programátoři
spolupracují na kódu.

Drobná nevýhoda je, že je to další systém, který se musíš naučit konfigurovat.
Dnes je to ovšem čím dál rozšířenější a systémoví administrátoři s Ansiblem
nebo podobnými systémy běžně pracují.

Systémů na správu konfigurace je více: například Ansible, Chef, Puppet,
SaltStack, nebo Terraform.
Každý funguje trochu jinak, ale základní koncepty bývají podobné.
Dnes si ukážeme Ansible.


## Deklarativní zápis

Jeden z konceptuálních rozdílů mezi klasickou administrací a Ansible je ten,
že Ansible je *deklarativní*.
Místo toho, že bys spouštěl{{a}} příkazy které něco udělají,
popíšeš jaký má být výsledný stav nastavovaného systému.
Úkol Ansible je potom zkontrolovat, že je systém ve správném stavu – a pokud
není, tak ho do toho stavu dostat.

V praxi se tyto přístupy tolik neliší (`sudo dnf install httpd` taky nedělá
nic, pokud je `httpd` už nainstalovaný), ale jde o užitečný úhel pohledu.


## Idempotence

A když už jsme u cizích slov: úkoly pro Ansible by měly být *idempotentní*,
když Ansible pustíš dvakrát za sebou, ve druhém běhu se nic nestane.
Například „zajisti, že je nainstalovaný `httpd`“ je idempotentní – pokud to
provedeš několikrát za sebou, výsledek je stejný jako kdybys to provedl{{a}}
jen jednou.

Oproti tomu „vytvoř nový šifrovací klíč pro `ssh`“ idempotentní není – pokaždé
se vytvoří jiný klíč.
Idempotentní úkol by byl „zajisti, že je vytvořený šifrovací klíč pro `ssh`“.

> [note]
> Ansible nezaručuje, že všechny úkoly jsou opravdu idempotentní.
> Lze tak například úkol „Napiš zprávu, že je vše OK“, který zapíše novou
> zprávu při každém spuštění.
> Dobře napsané úkoly ale idempotentní jsou.
> Aspoň v těch důležitých aspektech.


## Instalujeme program

Nejlepší způsob jak se naučit pracovat s Ansiblem je praxe.
Zkusme si nainstalovat balíček a nakonfigurovat ho.

Než začneš, musíš nainstalovat samotní Ansible.
(Na serverové farmě to je potřeba udělat jen na jednom počítači – na tom,
který bude řídit ty ostatní.)

```console
$ sudo dnf install ansible
```

Pak bude potřeba napsat konfigurační soubory.
Bude jich víc, tak si pro tuto lekci udělej si nový adresář.

Pro Ansible se píšou tzv. *playbooky*, což jsou soubory ve formátu
[YAML](https://en.wikipedia.org/wiki/YAML).
Podobně jako Python tenhle formát vyžaduje odsazování a bude si stěžovat,
pokud ho nedodržíš.
Pozor tedy na mezery a pomlčky na začátcích řádků.

Pro první playbook si vytvoř soubor s názvem `setup.yml`.

První věc, kterou nastavíme, jsou informace o počítačích které chceš
konfigurovat: které to jsou a jak se k nim připojit.
Protože máš teď k dispozici jen jeden virtuální počítač,
nastav `hosts` na `localhost` a `connection` na `local`.

```yaml
# setup.yml

- hosts: localhost
  connection: local
```

V praxi se tahle sekce většinou píše do samostatného souboru,
[inventáře](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html),
kde jsou uvedeny i další informace – třeba které ze strojů mají být webové servery
a které databázové.

Abys mohl{{a}} něco nainstalovat, je třeba býti `root`em.
To je potřeba zapsat; později pak řekneme ať k tomu Ansible použije `sudo`
a heslo.

```yaml
   become_user: root
```

Dále se píše stav, do jakého chceš dostat svůj systém.
K tomu slouží seznam `tasks`.
V YAML se seznamy definují pomlčkou u každého prvku.

Každá úloha musí mít jméno.
Nainstalujeme si třeba `htop`, což je vylepšený `top`:
program na sledování stavu systému.

```yaml
   tasks:
   - name: Install htop
```

Pro instalaci musíš být `root`, což musíš zadat i v úloze.
:

```yaml
     become: yes
```

Tohle `become` piš pod `name`: je to další záznam o téhle úloze.
Pomlčku na začátku řádku, která označuje nový prvek seznamu `tasks`,
potřebuješ jen před `name`. Celý blok `tasks` bude teď vykadat takhle:

```yaml
   tasks:
   - name: Install htop
     become: yes
```

Přichází čas na instrukci, jak balíček nainstalovat.
Na Fedoře se instaluje pomocí pluginu `dnf`, kterému řekneš,
že balíček `htop` chceš nainstalovat v poslední dostupné verzi:

```yaml
     dnf:
        state: latest
        name:
        - htop
```


Celý soubor nakonec vypadá takto. Pozor na odsazení:
```yaml
- hosts: localhost
  connection: local
  become_user: root
  tasks:
   - name: Install htop
     become: yes
     dnf:
        state: latest
        name:
        - htop
```

Teď to zkus spustit:

```console
$ ansible-playbook -K setup.yml
BECOME password: 
[WARNING]: provided hosts list is empty, only localhost is available. Note that
the implicit localhost does not match 'all'

PLAY [localhost] ***************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Install htop] ************************************************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

Parametr `-K` říká, aby se tě Ansible nejdřív zeptal na heslo pro `sudo`
a s tím pak spouští úkoly označené `become: yes`.


Když ten samý příkaz pustíš znova, Ansible ohlásí že je vše v pořádku;
nic se nezměnilo.


## A co dál?

Co všechno se dá dělat s Ansiblem?
- instalovat balíčky (to jsme si právě zkusili)
- kopírovat soubory - např. konfigurační soubor pro `cron`.
- přidávat uživatele, skupinu uživatelů
- a mnohem víc...

Pro každý druh úlohy existuje tzv. *pluginy*, který danou úlohu řeší.
Ansible obsahuje základní nabídku pluginů, další se dají doinstalovat,
a když ani to nestačí, můžeš si v Pythonu napsat další.

Tedy je malá ukázka toho, co je možné; další možnosti najdeš v dokumentaci.
(Nebo na dalším kurzu?)

```yaml
- hosts: localhost
  connection: local
  become_user: root
  tasks:
   - name: Install packages
     become: yes
     dnf:
        state: latest
        name:
        - htop
        - httpd
   - name: Uninstall packages
     become: yes
     dnf:
        state: removed
        name:
        - cowsay
   - name: Add a user
     become: yes
     user:
        name: testuser1
   - name: Add a file to serve
     become: yes
     copy:
        dest: /var/www/html/index.html
        # Znak `|` v YAML uvozuje odsazený blok s víceřádkovým textem
        content: |
            Hello world!
            Welcome to my server!
   - name: Start & enable httpd
     become: yes
     systemd:
        name: httpd
        state: started
        enabled: true

  - name: Set up RPM fusion
    become: yes
    shell: |
      dnf install -y https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
    args:
      creates: /etc/yum.repos.d/rpmfusion-free.repo
```


Tento příklad se připojuje pouze k aktuálnímu počítači. 
Pokud bys to chtěl{{a}} pouštět i na jiných počítačích,
je třeba je zapsat do takzvaného *inventáře* (*inventory* file)
a použít pomocí přepínače `ansible-playbook -i`.
Návody a příklady opět najdeš v dokumentaci Ansible.


## Ansible na laptopu

Automatizovaná konfigurace počítačů je nejužitečnější pro správu více
počítačů. Ty ale k nějaké serverové farmě bohužel přístup hned tak nedostaneš.

Můžeš si ale Ansible zkoušet i na vlastním laptopu.
Když si oblíbené nastavení uložíš do *playbooku*, můžeš ho sdílet mezi dvěma
počítači (např. osobním a firemním), nebo ho použít když si budeš nastavovat
nový počítač.
