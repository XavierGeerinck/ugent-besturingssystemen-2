## 5 Gebruikers en het Filesystem intern
### 5.1 Gebruikers en groepen
Toegang is bepaald door gebruikersnaam en bijbehorend wachtwoord. Maar ook de groep speelt een rol of een gebruiker toegang heeft tot bepaalde bestanden.

#### 5.1.1 /etc/passwd
Dit bestand slaagt alle gegevens op van de gebruiker om toegang te kunnen hebben. Het `:` teken is de separator. 

Veld|Betekenis
username|Inlog naam, root en andere zijn door het systeem bepaald. Deze naam kan men veranderen in het passwd file.
wachtwoord|* als men niet kan inloggen, x als het wachtwoord in de shadow file staat
userId|Nummer van de gebruiker, userId 0 is root.
groupId|GroepId, dit is de primary group. kijk in /etc/group voor meer info over de groepen.
Info over gebruiker|Telefoon, Description, naam, ...
HomeDir|Home dir gebruiker
StartProgram|Welk programma op te starten na boot. Als /sbin/nologin, dan kan met niet interactief inloggen (of ook /usr/bin/false)

**vb:**

```bash
nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
root:*:0:0:System Administrator:/var/root:/bin/sh
daemon:*:1:1:System Services:/var/root:/usr/bin/false
```

#### 5.1.2 Shadow en Encryptie
Alle wachtwoorden zijn geencrypteerd met one way encryptie, men kan deze enkel kraken door middel van brute-force attacks. Deze worden opgeslagen in het /etc/shadow bestand welke enkel voor root zichtbaar is. Dit bestand bevat een aantal kolommen, de gebruikersnaam en het wachtwoord en nog andere i.v.m. wachtwoord policy (hoe vaak veranderen, min lengte, ...)

**vb:**

```bash
root:$1$CQoPk7Zh$370xDLmeGD9m4aF/ciIlC.:14425:0:99999:7:::
bin:*:14425:0:99999:7:::
daemon:*:14425:0:99999:7:::
```

#### 5.1.3 Groepenbestanden
Weeral kolommen gesplitst door `;`, dit bevat de groepnaam, encrypted wachtwoord (meestal leeg), groepnummer en lijst van namen van gebruikers.

```bash
nobody:*:-2:
nogroup:*:-1:
wheel:*:0:root
daemon:*:1:root
certusers:*:29:root,_jabber,_postfix,_cyrus,_calendar,_dovecot
```
### 5.2 Toegang tot fileobjecten
Elk bestand heeft zijn eigen gebruiker- en groepseigenaar. Deze zijn volledig onafhankelijk van elkaar. De groepseigenaar is dezelfde als de primaire groep van de gebruiker. Het is handig om groepen te gebruiken om users toegang te geven tot bepaalde files. Zo wordt bijvoorbeeld de groep `www-data` vaak gebruikt voor website data zodat een nginx of apache server hier toegang tot hebben.

Wanneer een file of directory wordt aangemaakt dan is de gebruiker die dit aanmaakt de eigenaar hier van, samen met zijn groep. Om dit te veranderen kan men gebruik maken van het `chown` (change owner) en het `chgrp` (change group) commando.

We kunnen ook specifieke veranderingen aan de files en folders doen voor permissies door het aanpassen van de write (w), read (r) en execute (e) flags. (777, 775, 776, ...).

Met de write flag kunnen we ook bestanden verwijderen in directories, dus hier best mee opletten.

We kunnen rap rechten aanpassen door bijvoorbeeld `chmod u+w` uit te voeren wat in dit geval write rechten gaat geven aan de users. Zie onderstaande tabel voor meer info.

|toegangsklasse|operator|toegangstype|
|-|-|-|
|**u** (user)<br />**g** (group)<br />**o** (other)<br />**a** (all)|**+** (voeg recht toe)<br />**-** (verwijdert het recht)<br />**=** (zet het recht)|**r** (read)<br />**w** (write)<br />**x** (execute)|

We kunnen ook de **standaardwaarde** instellen van welke rechten een nieuw fileobject krijgt. We doen dit door gebruik te maken van **`umask`**. vb: `umask 022` let op, het nummer dat we hier zien is hetgeen we krijgen als we het aftrekken van `777`, dus in dit geval `755`. Ander voorbeeld: `umask 077` geeft dan `700`.

### 5.3 Aanmaken van gebruikers (manueel)
1. Voeg lijn toe in het shadowbestand (als groep nog niet bestaat, voeg dan ook toe in groups file) (wachtwoord doen we met passwd)
2. Maak home directory aan als deze niet bestaat
3. Kopieer sjablonen van profielbestanden en directory's (/etc/skel of /etc/security)
4. Stel aangepaste rechten in op bijvoorbeeld het .bash_profile bestand.
5. Stel correcte permissies in voor home diretory
6. Zorg er ook voor dat enkel deze gebruiker er toegang tot heeft.

### 5.4 Speciale toegangstypes

|Bit|Betekenis|
|-|-|
|sticky bit **(op file)**|save text mode, houd uitvoerbaar kopie in geheugen, gebruikt bij veel geopende programma's zoals vi) (zet via `chmod o+t`, geeft kleine t als ook execute, grote T als geen execute, via octaal: `chmod 1777 /tmp`)|
|sticky bit **(op dir)** |als het op dir gezet wordt, kan men enkel bestanden verwijderen waarvan men eigenaar is) (zet via `chmod o+t`, geeft kleine t als ook execute, grote T als geen execute, via octaal: `chmod 1777 /tmp`
|set-user-id bit (SUID)|process van binair bestand krijgt id van gebruikersnaam (effective user id), als SGID dan groupsid, zo kunnen we bijvoorbeeld met passwd het passwd bestand aanpassen ook al hebben we geen schrijfrechten tot /etc/shadow. vb flag: `chmod 4000` of `chmod u+s` of `chmod g+s`. Gaat enkel voor binaire bestanden!|
|set-group-id bit (SGID)|als deze gezet, dan is de group gelijk aan group van directory en niet van gebruiker, `chmod 2000`
