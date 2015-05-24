# 8. Questions
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