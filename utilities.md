# 9. Utilities
## 9.1. CRONTAB
* Gaat taken periodiek uitvoeren
* Wordt beheerd door de cron daemon
* Crontab files bevinden zich meestal in `/var/spool/cron/crontabs` of `/usr/spool/cron/crontabs`.
* Met `crontab naam` maakt cron een bestaand aan op de juiste plaats.
* Nu geeft de gebruiker de minuten, uur, dagnummer, maand, en dag van de week in om de cron te laten lopen.

**Voorbeeld:**
```bash
0,15,30,45 * * * * (echo; date; echo) > /dev/console
0,10,20,30,40,5 7-18 * * * /usr/lib/atrun
0 0 * * * find / -name "*.bak" -type f -atime +7 -exec rm {} \;
```

## 9.2. Backup en restore
### 9.2.1. Backup Strategie
Er zijn in totaal 3 vormen die gebruikt worden voor back-ups.

* Full-backup: Deze maakt een volledige kopie.
* Incrementele back-up: Dit is een kopie van de dingen die veranderd zijn de laatste tijd.
* Differentiele backup: Tussenvorm, kopieert wijzigingen sinds laatsts backup.

Met een cron kunnen we bijvoorbeeld instellen om het systeem elke weet te laten backuppen.

### 9.2.2. Tar
Dit is afkomstig van tape aarchive. Zie de man page voor de opties.

### 9.2.3. cpio
Dit staat voor copy-input-output. Deze wordt gebruikt voor het zeer gemakkelijk maken van een compleet arbitraire verzameling van files.

### 9.2.4. dump & restore
Het commando dump heeft eigenlijk tot doel een lokaal filesysteem in zijn geheel te kopieren.

Om dit terug te herstellen gebruiken we het commando restore.

### 9.2.5. dd
dd staat voor disk dump, zie man page voor info.
