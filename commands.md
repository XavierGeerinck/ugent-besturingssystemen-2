## 3. Commando's
Commando | Uitleg
---------|--------
**su permissies**|**==================================================================================**
su| wissel gebruiker naar root, niet met alle instellingen die je normaal hebt
su -|wisselt volledig naar nieuwe gebruiker
su -l|zelfde als su -
su --login|zelfde als su -
**man pages**|**==================================================================================**
man <doc>|Opent documentatie, vb: man whereis
man 1 <doc> … man 8 <doc>|opent specifieke documentatie categorie voor gegeven command, vb: man 1 whereis, man 5 passwd|Opent man page van 5e categorie voor passwd
man -i|Opent zonder case matching
**Directory manipulatie**|**==================================================================================**
.|Huidig pad
..|Parent pad
pwd|Print working directory
cd|change directory
ls|list informatie over bestand of directory
du|disk usage, geeft omvang van bestanden
mkdir|maakt directory aan
rmdir|verwijderd dir (kan ook met rm -R)
cp <bron> <doel>|kopieert file van bron naar doel
du * \| sort|Geef disk usage van alle bestanden en sorteer, we willen echter getallen gebruiktn
du * \| sort -rn|fixt bovenstaand commando!
lpr <bestandenlijst>|lineprinter, print bestanden af
**Permissies**|**==================================================================================**
chmod|Verandert permissies file of dir
chmod u+x|Geeft execute permissies aan de gebruiker
**Vind files**|**==================================================================================**
whereis <bestandsnaam>|vind het gegeven bestandsnaam, vb: whereis docker
find <padnamen> <opties>|zoekt bestanden
find .|zoekt alle bestanden met .
find /home/gvh -size +10k|zoekt bestanden groter dan +10kb in de gvh home dir
find / -user gast|zoekt bestanden voor gebruiker
find . -name "*.cc"|zoek bestanden met extensie .cc
find . -size +10k -exec ls -l {} \;|vind bestanden groter dan 10k en print met ls -l
**File content**|**==================================================================================**
cat <file1> <file2>|toont inhoud bestand
cat /etc/passwd|print passwd file
diff <file1> <file2>|geeft verschil tussen bestanden
**Date**|**==================================================================================**
date|print datum, kent vele opties
**Mail**|**==================================================================================**
mail|Zend mail, (vb: `mail -vv -S smtp=smtp.hogent.be -a /etc/passwd -r sender@hogent.be receiver@ugent.be`)
**FTP**|**==================================================================================**
ftp|Opent ftp en allow voor commando's (vb: `ftp -n computernaam <invoer_voor_ftp` OF `ftp -n computernaam <<EOF user gast wachtwoordvangast dir put file _xyz delete file_xyz dir EOF`)
**Systeem info**|**==================================================================================**
stty -a|Geeft config STTY zoals line ending, EOF shortcut, ...
**Gebruiker info**|**==================================================================================**
who|geeft ingelogde gebruiker
who am i|zelfde als who
**Bash scripting**|**==================================================================================**
echo|print string, vb echo $PATH
echo \a|Roept het belsignaal ook op (alerts) (vb: `echo -e "Testing 123 \a"`)
echo \c|Geen nieuwe lijn na echo
echo \n|Extra newline
echo \\|Backslash
pr|Formatteer tekst (vb: `(pr cursustekst | lpr) &`)
lpr|Druk uitvoer pr af (vb: `(pr cursustekst | lpr) &`)
time|Berekent uitvoeringstijd (intern bash commando): (vb: `time -p wc /usr/man/man1/perltoc.1`) real = tijd eindig - opstart, user = tijd cpu gewerkt gebruiker, sys tijd cpu gewerkt OS
2>bestandsnaam|Piped STDERR naar bestand (vb: `time -p wc /usr/man/man1/* >wc.out 2>time.out`)
2>&1|Zet STDERR op STDOUT kanaal (vb: `time -p wc /usr/man/man1/perltoc.1 >wc.out 2>&1`)
alias|Verkort uitgebreid commando tot enkel woord (vb: `alias ll='ls -l' --color=tty` geeft ll als commando)
unalias <naam>|Verwijdert alias
unalias -a|Deactiveer alle aliassen
declare -F|Toont gedefinieerde functies
typeset -F|Toont gedefinieerde functies
declare -f|Toont functies + implementatie
typeset -f|Toont functies + implementatie
unset -f functienaam|Verwijdert functienaam
set|Gaat de variabele zetten op uitvoer vorige lijn (vb: ```date;set `date`)```
unset|Cleared variabele
IFS|Is Internal Field Seperator, gaat strings splitten op gegeven waarde (vb: `IFS=:` dan gaat deze splitten op : en komen alle substrings in $ argumenten, zo wordt date;set date;echo $1 Sat Oct 1 06)
export|Zet global value, is niet lokaal!
\.|Voert commando uit in dit shellscript door de huidige shell en niet door een sub-shell
\$HOME|Home dir van de shell
\$PWD|Current working dir
\$IFS|Split char var
\$PATH|PATH Variabele
\$PS1|Prompt string
\$PS2|String voor secundaire prompt, normaal >
**Bash scripting**|**==================================================================================**
passwd|Veranderd wachtwoord gebruiker
gpasswd|Veranderd wachtwoord groep
id|Geeft id van de gebruiker die het uitvoert, geeft uid en groepen terug (vb: uid=501(xanrin) gid=20(staff) groups=20(staff))
newgrp|Stel nieuwe actieve groep in.