# 8. Questions
## Bootstrapping (computer systeem opstarten en klaarmaken voor gebruik)
1. Bootproces gaat instructies laden op ROM, die starten meestal firmware op (BIOS)
2. Deze BIOS herkent de hardware en gaat bootpgoramma lokaliseren en starten
3. Dit programma laadt de kernel in het geheugen
4. Deze initialiseert interne tabellen, detecteert de hardware, laadt de nodige drivers en start spontane processen zoals het init process.
5.  Dit initprocess laadt initialisatiescripts in /etc  dir (zie `/etc/inittab`).
    * Testen integriteit filesystem met `fsck`
    * Mount lokale partities
    * Ken paging areas toe (swap)
    * Clean filesystem (/tmp, /usr/tmp)
    * Start server daemons
    * Start netwerk daemons
    * Start X-Windows
    * Laat gebruikers zich aanmelden
6. Nu zit men in multiuser mode.
7. Meldingen worden gemonitored door syslogd of dmesg en bewaard in een file (/usr/adm/messages, /var/adm/messages, /var/log/messages).

## Verschillende runlevels
|runlevel|naam en betekenis|
|:-:|-|
|0|powerdown, veilig om spanning af te leggen|
|1|systeemadmin mode|
|S of s|single user mode|
|2|multi user mode zonder netwerk|
|3|multi user mode met volledige netwerk|
|4|door de admin te definieren|
|5|firmware state, speciaal onderhoud voor systeemdiagnose|
## Verschillen bootstrapping bij Windows
* Deze gebruikt het MBR, wat het master boot programma bevat
* Deze laat het programma op het bootblok.
* Dit 2de bootprogramma gaat acties uitvoeren om kernel te starten.

Als linux een MBR gebruikt is dit meestal grub of lilo. Windows heeft zijn eigen versie.

## Commando's opzoeken en uitvoeren
1. Als pad, kijk of bestand bestaat en uitvoerbaar is. Als dit het geval is, voer commando uit. Anders toon een foutmelding.
2. Als geen pad, kijk of het een intern commando is.
    * Als alias, voer alias uit
    * Als functie, voer functie uit
    * Als geen alias of functie, zoek op in interne tabel van shell commando's (als bestaat voer uit).
3. Als geen intern commando, overloop $PATH directories.
4. Anders toon "command not found"

## Voeg gebruiker toe zonder adduser / addgroup
1. Voeg lijn toe in /etc/passwd
    * **login:** Gebruiker zijn login naam
    * **x:** `*` of `x` toont aan dat het een geencrypteerd wachtwoord is, anders staat plain hier.
    * **uid:** User id
    * **gid:** Group Id
    * **gecos:** Gebruikers info
    * **home_dir:** Home directory
    * **shell:** Te gebruiken shell, meestal /bin/bash
2. Voeg lijn toe in /etc/shadow (user:password:systeminfo)
    * **user:** login zoals deze in /etc/passwd staat
    * **password:** voorlopig niks, dus een ;
    * **systeminfo:** Kan meerdere lijnen omvatten, eigen aan systeem
3. Stel wachtwoord in met passwd
4. Maak home directory aan
5. Kopieer /etc/skel/* naar home directory
6. Zorg dat gebruiker directory owned door `chown login:group`
7. Zorg dat gebruiker toegang heeft door `chmod 700`
8. Toevoegen `umask 077` aan `.bash_profile` (umask wordt chmod 700)

## Aan welke voorwaarden moet er zijn voldaan om als gebruiker het passwd commando te kunnen gebruiken?
* Admin moet toestaan dat je wachtwoord mag veranderen
* Je mag enkel je eigen wachtwoord veranderen

## Leg cron uit
* Voert scripts uit op bepaalde tijdstippen (vb logs inkorten, backups, ...)
* Men kan resultaat doorsturen naar admin via email
* Uitvoeren administratieve taken zonder interactie met gebruiker
* Gecontroleerd door de cron daemon:
    * Controleert periodiek cron bestanden (/vars/spool/cron/crontabs)
        * Bijgehouden per gebruiker
        * Gebruiker kan deze files aanpassen.
    * 1 file per gebruiker, bestandsnaam is de username van de gebruiker
    * Syntax: minuten uren dag-van-de-maand maand weekdag commando 
        * Vb: `0,15,30,45 * * * * (date;echo)`
        * Een `*` betekent niet ingevuld
        * Een `#` is een commentaarlijn (als lijn hiermee begint)
    * Gebruikers verhinderen cron te gebruiken: 
        * Bestanden `cron.allow` en `cron.deny` in `/var/spool/cron`
        * Gebruiker in allow => toelaten
        * Gebruiker in deny => verboden
        * Geen gebruikers in beide => root enkel
    * Verwijderen gebeurt door `cron -r`

## Verklaar speciale toegangsrechten
* **Sticky bit**
    * **Vroeger:** save text mode, indien gezet houd bestand in geheugen (bv: vi)
    * **Nu:** 
        * Geplaatst op directory
        * Enkel eigenaar van bestanden mag deze bestanden verwijderen.
        * Voorbeeld: `/tmp` 
        * Aanzetten: `chmod u+t /tmp`
        * Zien: `ls -ld /tmp` ==> `drwxrwxrwt 2 root root 8704 ...`
        * In recente linux distributies: `chmod o+t /tmp` of `chmod 1777 /tmp`
* **set-user-id (SUID) / set-group-id (SGID) op file**
    *  Bij opstarten process, krijgt dit effective user id / group id het id van de gebruikerseigenaar.
    *  Dit laat gewone gebruikers commando's uitvoeren waar ze normaal geen rechten op hebben (vb: `passwd`)
    *  Gebruik: `chmod u+s binbestand` of `chmod g+s binbestand`
    *  Gebruik: `chmod 4xxx binbestand`
    *  Gaat enkel voor binaire bestanden, niet voor scripts (kan via compile bestand en exec)
* **set-group-id (SGID) op directory**
    * Bestanden gemaakt in directory met dit bit, krijgen groepseigenaar van de directory 
    * Interessant voor bijvoorbeeld www (`chmod g+s dir`)

## Verklaar variabelen in de shell
* Gewone variabelen
    * Stelt geheugenplaats die naam heeft gekregen voor.
    * Naam begint met een letter of underscore en bevat letters, cijfers of underscores.
    * Shell variabelen zijn altijd tekenstrings, dit omdat deze geen variabeltypes kent.
    * Waarde wordt gezet door toekenning (=)
    * Variabele wordt omgezet in waarde door prefix $ (expanderen d.m.v. parameter of woordexpansie)
    * $1,$2,... zijn argumenten op commando regel
    * Environment variabelen (enkel hoofdletters):
        *  Elke variabele gekopieerd naar kind shell.
        *  Gebruik het commando `set` om deze te zien 
        *  Voorbeelden: `PATH`, `HOME`
        *  Zetten door: `PATH=/bin:/usr/bin:/urs/local/bin`
    *  Verwijderen variabele door: `unset varname`
    *  Opsplitsen van parameters door IFS, standaard spatie, tab, new line. Zet deze door: `IFS=:`
* Arrays
    * `table[32]="value"`
    * `declare -a tablename` → Dit maakt tabel aan zonder elementen
    * `declare -A tablename` → Maakt associatieve array aan
    * `table=(val1 val2 val3)` → Zet tabel elementen
    * `table=(val1 [2]=val2 val3)` → Zet index 2 op waarde
    * `echo ${table[32]}` → Toont waarde op plaats 32
    * `echo ${#table[*]}` → Toont aantal elementen
    * `echo ${table[@]}` of `echo ${table[*]}` → toont alle elementen
    * `unset tablename[idx]` of `unset tablename` → verwijdert tabel index of tabel
* Speciale notering: foutboodschap
    * `echo ${var?}` → Als var gezet print, anders error
    * `echo ${var?error happened\?}` → Toont error happened als niet gezet
* Default waarden
    * `echo ${var='default value'}` → Wijst default waarde toe 
* Anti Default waarden
    * `echo ${var+'some val'}` → Dit voegt some val toe als var gezet
* Bereik van variabelen
    * `sh` opent nieuwe shell
    * Environment variabelen worden daarnaar gekopieerd, andere variabelen niet
    * Enkel indien ze geexporteerd zijn door: `export varname`
    * Kind shell kan geen waarde teruggeven aan oudershell (schrijven in file)
* Speciale variabelen
    * `$#` → aantal / lengte
    * `$*`, $@` → Alle argumenten van het shellscript
    * `$0` → Naam script
    * `$?` → Exit status
    * `$$` → Process ID
    * `$!` → Process identification laatste met & opgestart commando

### Bij toekenning aan variabele, waarom geen spatie voor en na het = teken?
* Bash ziet assignatie anders als een commando met parameters achter
* Gaat zoeken achter commando en gaat dit mss niet vinden → error
* Wel vinden zal toeval zijn, en gaat rare uitkomst geven

### Waarom kan variabele alleen van ouder --> kind en niet andersom? 
* Als kind process aangemaakt, enkel env vars gekopieerd
* Dit is een subshell, en heeft dus een volledige andere omgeving
* Ouder kan variabelen kopieren door `export var` te doen, kind kan dit niet naar ouder.

## Wat zijn wildcards en hoe worden ze door de shell verwerkt?
|Wildcard|Betekenis|
|--------|---------|
|-|-|
|`*`|Duid een willekeurig karakter aan|
|`?`|Wijst op 1 willekeurig karakter|
|`/`|Duid newline aan|
|`#`|Commentaar|
|`[a-z]`|1 karakter die aan het patroon binnen [] voldoet, hier van a - z|

De betekenis van wildcards kan ongedaan worden gemaakt door:

* enkele aanhalingstekens te zetten (letterlijke interpretatie)
* Dubbele aanhalingstekens, $, \n en \ worden wel nog geinterpreteerd
* Aanhalingstekens enkel rond de wildcard kan ook
* Na interpretatie worden de aanhalingstekens verwijderd
* Als geen resultaat met wildcard, dan toont echo het patroon.

## Op pagina 105, de figuur. Wat zit er in de tabel en naar waar verwijst deze. Wat is single, double, triple, quadruple, ..?
* De tabel bevat adressen die verwijzen naar datablokken van het fileobject.
* Single, double, triple, .. staat voor het indirect blok. 
* Hebben we bijvoorbeeld een blokomvang van 1KB, en een adres 32 bit is (4 bytes), dan kan een blok 256 adressen bevatten.
	* 10 directe blokken geeft dan 10KB
	* 1 indirect blok met 256 directe blokken geeft 256KB
	* 1 dubbel-indirect blok met 256 directe blokken geeft dan 256^2 = 64MB
	* 1 driedubbel-indirect blok met 256 dubbel-indirecte blokken geeft 16GB

## Beschrijf de STIN, STDOUT, STDERR
* Deze 3 zijn gekoppeld aan bestanden (filedescriptors). Kunnen doorverbonden worden met een file of een pipe.
* 0 = STDIN: Input descriptor
* 1 = STDOUT: Output descriptor
* 2 = STDERR: Laat errors altijd op scherm komen, verwittigd gebruikers.
* We kunnen deze samenvoegen door bv: `/usr/bin/time -p wc /usr/man/man1/perltoc.1 >wc.out 2>&1` dit gaat stderr oo zelfde file zetten als stdout.

## Wat is een 'HERE' document?
* Invoer specifieren voor een commando, direct na het commando.
* Vb: `ftp -n naam <<EOF user gast wachtwoordvangast ..... EOF`
* Alles wordt als 1 regel gezien wat na de << komt.
* Invoer direct, niet via een apart bestand.
* $ en `...` worden geinterpreteerd, tenzij tussen aanhalingstekens of voorafgegaan door \

## Beschrijf de werking van het filesystem fysisch, hoe verschillende worden fysische media in het systeem beheerd?
### Logische boomstructuur verdelen over verschillende fysische diske of partities
* Kan zelfs over verschillende computers
* Meestal magnetische schijf
*  Bij grote systemen worden takken ondergebracht op verschillende schijven of verschillende partities.
	* Als er een fout gebeurt op 1 van de disks is niet alle data aangetast.
	* Moeilijker onderhoudbaar (wat indien filesystem te klein?)

### mount, umount
* Filesystems op verschillende devices worden gekoppeld aan boomstructuur
	* Hierna weet de kernel dat device bestaat
	* Er bestaat 1 logische structuur vanaf /
	* Vb: `mount -t type device dir` of `mount -t vfat /dev/fd0 floppy`
	* Meestal enkel voor root login
	* Best monteren op lege dir.
* Mount toont alle gemonteerde filesystems
* umount gaat schijf van logische boom halen
* umount moet gebeuren voor schijf te demonteren
* vb: `umount /dev/fd0` of `umount /mnt/floppy`

### Filesystem configureren door middel van `/etc/fstab`
* Bestand dat aantal lijnen bevat, 1 per device.
* Elke lijn beschrijft welk device er waar wordt gemonteerd gebruik makend van opties.

**Voorbeeld:**
```
/dev/hda3	 /					ext2    defaults   1 1
/dev/hda2	 swap				swap    defaults   0 0
/dev/hda1	 /mnt/windows98	vfat    defaults   0 2
/dev/cdrom /mnt/zip		   iso9660 noauto, ro 0 0
```

**Opties zijn:**

* **defaults:** dit zijn rw, suid, dev, exec, auto, nouser en async
* **rw:** read write
* **ro:** read only
* **suid:** laat toe dat de SUID en SGID bits effect hebben
* **dev:** interpreteert speciale devicefiles
* **exec:** laat de uitvoering toe van binaire files
* **auto:** kan worden gemonteerd met `mount -a` bij starten van systeem
* **nouser:** verbiedt een gewone gebruiker het filesysteem te monteren
* **async:** alle I/O naar het filesysteem gebeurt asynchroon
* **user:** laat een gewone gebruiker toe het filesysteem te monteren (impliceert wel noexec, nosuid en nodev)
* **noauto:** het bestandssysteem kan alleen expliciet worden gemonteerd.

**Gebruik fstab:**

* `mount -a` Alle devices in fstab worden gemount, tenzij die met noauto
* monteren van devices vermeld in fstab is het genoeg met ofwel dir ofwel device (`mount /dev/fd0` of `mount /mnt/floppy`.
* Devices met "user" optie kunnen worden gemount door gewone users (ook umount)
* `df` toont vrije ruimte

### Filesystem blocks
* Logische blokken van 512, 1024, 2048, .... (afhankelijk van systeem)
	* Grootte van blokken waarmee OS data leest of schrijft
	* Grote blokken zijn sneller maar geven slechtere diskusage.
* Layout logische blokken:
	1. **Boot blok:** 1e sector van diskpartitie, bevat opstart code (init OS)
	2. **Super blok:** Beschrijft status filesysteem 
		* omvang, namen van filesystemen, naam volumes, aantal files die het kan bevatten, time laatste update en backup, vrije data blokken, lijst vrije inodes.
		* `tune2fs -l device`: Geeft overzicht van superblok
		* Gereserveerde blokken zijn niet in te nemen door gebruiker.
	3. **Inode lijst:** pointer naar datablok
	4. **Data blokken:** Bevatten eigenlijke gegevens, 1 datablok kan maar tot 1 fileobject behoren.

## Wat zijn signalen? Hoe worden ze verwerkt? Hoe kan een applicatie er mee omgaan?
Signalen worden gebruikt om processen te stoppen

### Signalen

|Signaal ID|Betekenis|
|:-|-|
|1 (HUP)|Hangup, verbreekt connectie tussen terminel en process|
|2 (INT)|Interrupt, zelfde als CTRL+C|
|3 (QUIT)|Zelfde als DEL-toets, neerslag van geheugeninhoud|
|9 (KILL)|Kill process (kan worden opgevangen) kan niet gestopt worden|
|15 (TERM)|Terminate, zo snel mogelijk afsluiten na opkuis|
|24 (STOP)|Zelfde als CTRL+Z (jobcontrole stop)|

Gaat ook alle kinderen van deze shell stoppen

### Process die niet gekilled kunnen worden, zelfs niet met kill
1. Zombies: hebben nog geen bevestinging van parent maar bron is al opgeruimd. Neemt geen system resources in beslag maar worden wel vermeld.
2. Processen die wachten op niet beschikbare NFS-bronnen
3. Processen die wachten op device (bv: terugspoelen tape-drive)

### Signalen opvangen in shell:
`trap rij-commands lijst-signaal nummers`, vb: `trap 'rm -f $new $old; exit 1' 1 2 15`.

## Bespreek device files en geef een beknopt overzicht van de andere soorten files met hun functionaliteit
### Device files
* Verzorgen input en output met alle devices
* Staan in `/dev`
* Je kan bestanden op een schijf aanspreken alsof de schijf een bestand is
* 2 soorten speciale files:
	* Character special files (I/O op karaktergeorienteerde of ruwe niet gebufferde basis.)
	* Block special files (I/O verloopt in blokken, data wordt getransfered in blokken van vaste lengte)
* Namen sterk afhankelijk van de implementatie
* Harde schijf: hda
	* hda1 = eerste partitie
* Floppydrive: fd
* `ls -l`: geeft i.p.v. omvang 2 getallen, wordt gebruikt door devices manager in kernel (major --> zegt welke devicedriver, minor --> zegt hoe te gebruiken)
* Aanmaken door: `mknode [c|b] majornr minornr filename`
* Staan ook vermeld in fstab    

### Andere soorten files
* **regular files:** normale bestanden (ascii tekst, binaire info, ...)
* **dirs:** binaire file met lijst van andere fileobjecten (lijst bevat paren met naam van file en nummer inode)
* **links:** verschillende filenaam refereren naar bestand, 2 soorten: hard link en soft link of symbolic link.
* **sockets:** File dat wordt gebruikt ter communicatie tussen processen
* **named pipes:** Pijplijnen die geopend werden op naam, meestal in /dev. Andere manier van IPC.

## Bespreek de uitvoer van het ps-commando en bespreek de bijhorende procesattributen
* Toont informatie over de processen die actief zijn
* Veel opties -> man pages
* Hoofdingen van de uitvoer:	
	* PRI: Geeft frequentie in Hz van timeslice waarmee proces aan bod komt
	* NI: Nice number
	* SIZE: Omvang proces in virtueel geheugen
	* STAT: Status (R=run, S=sleep, D=dnd, T=gestopt, Z=zombie)
* Complexere vorm kan ook tonen hoe commando's van elkaar afhankelijk zijn.
* Comando top: scherm wordt om de 5 seconden vernieuwd --> geeft beel hoe zwaar pc belast is. 

## Bespreek logische en numerieke uidrukkingen in de shell, herhaling, voorwaarde, selectie, ...
### while
```
while commando
do
	# CODE
done
```

* geeft exit status 0 als geslaagd
* while, do en done worden door shell herkent, dus nieuwe regel of voorafgegaan door ';'

### if
```
if commando
then
	# CODE
elif commando
then
	# CODE
else
	# CODE
fi
```

### test
```
test
# of
[ uitdrukking ]
of
[[ uittdrukking ]] #extra functies: patronen, std numeriek vgl, ...
```

### for
```
for commando
do
	# CODE
done
```

### case
```
case teststring in
	patroon1)
		# code
		;;
	patroon2)
		# code
		;;
esac
```

### shift
doet een pop op de argumenten, verwijder 1e argumenten en schuif de rest 1 plaats op.

## Wat is het verschil tussen ; en & en wat voor invloed heeft het op de shell?
* `&`: voert uit op achtergrond, krijgt onmiddelijk na uitvoering controle over command prompt.
* `;`: Toevoegen op einde commando, staat toe meerdere commando's op 1 lijn.

## leg uit: commandolijn, variabelen, metatekens, omleiden, groeperen, ...
### commandolijn
* Woorden gescheiden door |, |, &, (, ), spatie, tab met carriage return erna.
* Interne of externe commando's:
    * Interne: Deel van shellprocess
    * Externe: Naam van binair uitvoerbaar programma of shell script
* Command completion door TAB te gebruiken
* Command History:
    * Bijgehouden per gebruiker
    * .bash_history
    * Pijltjes omhoog / omlaag
* Gereserveerde woorden
    * Speciale betekenis voor de shell
    * !, case, do, done, elif, else, esac, fi, for, function, if, in, select, then, until, while, {, }, time, [[, ]]

### opties en argumenten
* Geeft informatie aan een commando
* Verander iets aan de werking van het commando
    * POSIX: Meestal beginnend met '-' en is  een letter
    * NEW POSIX: Meestal beginnend met '--' gevolgd door woord.

### metatekens
* Speciale betekenis in de shell
* Voorbeelden: >, >>, >, |, >>string, *, ?, [kkk], ;, &, (...), $1 $2 $3, $var, ${var}, \, "tekst", var=waarde, &&, ||, { comm; }, ((uitdr)), [[ uitdr ]], [ uitdr ]

### pipen
* date | wc
* Gaat uitvoer 1e commando met '|' doorpipen naar invoer 2e commando.
* Kan meerdere keren na elkaar.

### redirecten
* Gaat de uitvoer naar een bestand sturen
* Vb: `date >> datefile.txt`

### commando's groeperen
* Gebeurt door (..)
* Vb: `(date; who) | wc`