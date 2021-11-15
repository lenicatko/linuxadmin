# Souborové systémy – mini úvod

Pojďme si přiblížit, jakým způsobem Linux zapisuje informace – soubory
a adresáře – na disk.

## Obsah

Představ si knížku. Knížka je v podstatě sekvence písmenek.
Když ale v téhle dlouhé sekvenci chceš najít třetí kapitolu,
nemusíš procházet každou stránku zvlášť.
Knížka má totiž na začátku *Obsah*, který ti řekne že kapitola 3 je na stránce 254.

Podobně jsou informace uloženy na disku.
To, že jsou na něm nějaké soubory a adresáře, o nichž jsme se minule bavili, je jen iluze.
Obsah disku je ve skutečnosti jedna dlouhá sekvence jedniček a nul,
podobně jako je knížka sekvence písmenek.
V tom by se ale vyhledávalo špatně,
a některé z těch jedniček a nul fungují podobně jako obsah v knížce.
Říkají: *zde jsou adresáře, v nich jsou tyto soubory, jsou takto velké, takto se jmenují* atd.

Nejlepší si to bude ukázat na příkladu archivu Zip.


### Archivy

- https://commons.wikimedia.org/wiki/File:ZIP-64_Internal_Layout.svg

Archiv *zip* obsahuje jednu dlouhou sekvenci informaci,
na jejíž konci je *centrální adresář*.
To že je zrovna na konci, je specifické pro Zip – u jiného druhu archivu
může být klidně třeba na začátku.

Centrální adresář obsahuje počet souborů v archivu,
a pro každý soubor několik dalších informací:

* jak se soubor jmenuje,
* kde v archivu začíná jeho obsah (data), a
* jak je ten obsah dlouhý.

Soubory v zipu se dají docela jednoduše číst.
Podíváš se, kde začít a jak dlouho číst, a pak je přečteš.

Na rozdíl od kapitol v knížce jsou názvy souborů pouze v centrálním adresáři;
neopakuje se na začátku každého kousku s daty.
Bez centrálního adresáře je proto archiv nepoužitelný.


## Souborový systém

Podobně jako archiv Zip fungují *souborové systémy* – způsob,
kterým počítač ukládá a spravuje spoustu souborů a adresářů na disku.
Na určitých částech disku jsou uložena data jednotlivých souborů;
na jiných je uložen seznam souborů v adresářích;
jinde jsou popisy souborů obsahující informace o jejich vlastnostech.

Na rozdíl od zipu je „opravdový“ souborový systém složitější – například
„adresář“ není jen jeden centrální, ale bývá pro každou složku zvlášť.
„Opravdový“ souborový systém je taky dělaný nejen na čtení souborů,
ale i na snadný zápis.
(To teoreticky jde i v Zipu, ale pokud je soubor který
chceš do archivu nahrát větší než místo, které je na něj v archivu vyhrazené,
je postup složitější.)

Základní myšlenka ale zůstává stejná – souborový systém je jako archiv,
jen mnohem složitější.
