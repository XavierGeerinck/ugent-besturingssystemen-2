# Scripting oefeningen
## Tips & Tricks
### Check of commando successvol
```bash
if !find ... ; then
...
else
...
fi
```

### Geen dubbele haakjes bij commando check
```bash
if [[ echo ... ]]; then

fi
```

In bovenstaande if statement hebben we [[]] haakjes, echter zijn deze niet nodig omdat we kijken naar de exit code. Daarvoor is volgend voldoende:

```bash
if echo ... ; then
fi
```

## # 89
### Vraag
Ontwikkel een shellscript dat als (enige) parameter een bestandsnaam heeft. Als output moet het script het gemiddelde aantal karakters per regel en het gemiddelde aantal woorden per regel uitschrijven. Gebruik de output van het commando `wc` en probeer met de diverse methodes die hierboven aangehaald werden (behalve de tweede) om deze output op te splitsen.
Opgelet: de shell behandelt alle getallen als
integers. Niettemin kun je er, met behulp van wat wiskunde, voor zorgen dat delingen correct worden afgerond. Hoe ?

### Oplossing
```bash
#!/bin/bash
regel=0
words=0
chars=0
while read line
do
        word_count=$(echo $line | wc -w)
        char_count=$(echo $line | wc -m)

        regel=$((regel+1))
        words=$((words += word_count))
        chars=$((chars += char_count))
done <$1

echo "AVG Words: "$((words/regel))
echo "AVG Chars: "$((chars/regel))
```

## # 90
### Vraag
Wat moet je doen opdat de read - instructie lijnen niet alleen in woorden zou opsplitsen op basis van spaties, tabs en regeleinden, maar ook op basis van :- tekens?

### Oplossing
```bash
cat /etc/passwd | { IFS=: read -r user x uid gid rest; echo $rest; }
```

## # 91
### Vraag
Ontwikkel een script om de gebruikersnaam (veld 1 van /etc/passwd) te bepalen aan de hand van een gebruikers-
ID (veld 3 van /etc/passwd) dat je als (enige) parameter aan het script meegegeven hebt. Je kunt de output van het commando grep zowel met cut, read, set als met een array analyseren.

### Oplossing
```bash
# Oplossing met cut
grep -E '^.*:.*:172:.*$' /etc/passwd | cut -d ':' -f 1

# Oplossing met read
grep -E '^.*:.*:172:.*$' /etc/passwd | { IFS=: ; read naam rest; echo $naam; }
```

## # 92
### Vraag
Ontwikkel een script dat het commando tail n simuleert. Als eerste argument moet een bestandsnaam opgegeven worden en als tweede argument mag het aantal regels
opgegeven worden. Ontbreekt het tweede argument, dan worden de 10 laatste regels weergegeven. Realiseer dit op twee manieren:

* Gebruik een while-lus met een read-commando om het bestand te overlopen en een array om de gegevens cyclisch op te slaan.
* Gebruik geen array, maar bepaal vooraf het aantal lijnen van het bestand. 


### Oplossing
#### While-lus met read-commando
```bash
#!/bin/bash
# Wees zeker dat n opgegeven, anders zet op 10
n=$2

if [[ -z "$n" ]]; then
        n=10
fi

# Declare the array
declare -a ARRAY

# Read lines into the array
i=0
while read line
do
        ARRAY[i]=$line
        i=$((i+1))
done < $1

# Print n lines
j=$((i-n))
for j in $(seq 1 $n)
do
        echo ${ARRAY[j]}
doen
```

#### Vooraf bepaalde aantal lijnen
```bash
#!/bin/bash
# Wees zeker dat n opgegeven, anders zet op 10
n=$2

if [[ -z "$n" ]]; then
        n=10
fi

LINE_COUNT=`wc -l < $1`
START_PRINT=$(($LINE_COUNT - $n))

# Read lines and print as soon as i = START_PRINT
i=0
while read line
do
        if [[ $i -ge $START_PRINT ]]; then
                echo $line
        fi
        i=$(($i+1))
done < $1
```

## # 93
### Vraag
Hoe kun je met behulp van de while-of until-lus een aantal commando's oneindig lang laten uitvoeren? Onderbreek de uitvoering met Ctrl+C

### Oplossing
```bash
while (true)
do

done
```

## # 94
### Vraag
Het bestand ping.out bevat de output van een Windows batch file : ping -n 1 AL005951 ping -n 1 AL005952 ...
Indien toestel xxxxxxxx actief is, wordt het commando
ping beantwoord met regels van de vorm:

    Pinging xxxxxxxx [141.96.126.137] with 32 bytes of data: 
    Reply from 141.96.126.137: bytes=32 time=1322ms TTL=124

Je kunt niet-actieve toestellen herkennen aan het feit dat het commando niet wordt beantwoord zoals bij actieve. De foutboodschappen die dan gegenereerd worden zijn divers. Maak een Bash-script dat uit ping.out een inventaris opmaakt van alle niet- actieve toestellen (één lijn per toestel). Het script moet ook een samenvattende regel weergeven die zowel het aantal actieve als het totaal niet-aantal toestellen vermeldt.
Zorg ervoor dat het script onafhankelijk is van de precieze foutboodschappen die niet-actieve toestellen produceren. Construeer twee oplossingen, al dan niet gebruikmakend van associatieve arrays (enkel beschikbaar in Bash v4). 

### Oplossing
```bash
#!/bin/bash
# Array to keep the NOT active hosts
declare -a ARRAY
i=0 # Index of the array
hosts=0

# Loop through the lines
while read -r p1 p2 p3 p4
do
#       echo "P1: "$p1 "P2: " $p2 "P3: " $p3 "P4: "$p4

        #  If ping start, then add to array of not active hosts
        if [ "$p1" == "C:\WINNT35\system32>ping" ]; then
                ARRAY[i]=$p4
                hosts=$(($hosts + 1))
        fi

        # If p1 = Request timed out, then see this as a non active pc
        if [ "$p1" == "Request" -a "$p2" == "timed" ]; then
                i=$(($i + 1))
        fi
done < $1

active_hosts=$(($hosts - ($i + 1)))
echo "Total Hosts: $hosts"
echo "Active Hosts: $active_hosts"
echo "Not Active Hosts: $(($i + 1))"
```

## # 95
### Vraag
Gebruik (enkel) het bestand /etc/passwd om voor alle groepsnummers het aantal gebruikers met hetzelfde primaire groepsnummer te tellen. Realiseer dit op twee manieren:
* Gebruik een while-lus met een read-commando om het bestand te overlopen en arrays om de gegevens op te slaan.
* Sla eerst de gegevens geordend op in een tijdelijk bestand, en verwerk vervolgens dit bestand. 

### Oplossing
```bash
#!/bin/bash
declare -a ARRAY

while IFS=: read -r username pass uid gid rest 
do
        ARRAY[$gid]=$((ARRAY[$gid] + 1))

done < /etc/passwd

for el in ${!ARRAY[@]}
do
        echo $el : ${ARRAY[$el]}
done
```

## # 96
### Vraag
Gebruik de bestanden /etc/group en /etc/passwd om een overzicht te maken van alle groepen, gevolgd door de volledige lijst van gebruikers die deze groep als primaire groep hebben. Gebruik een while-lus met een read-commando om het bestand /etc/group te overlopen en grep om de gebruikers op te sporen.

### Oplossing
```bash
#!/bin/bash
declare -a ARRAY

while IFS=: read -r name pass gid rest 
do
	# Print de naam van de group
	echo $name

	# Print gebruikers door middel van grep in /etc/passwd
	grep -E ".*:.*:.*:${gid}:.*" /etc/passwd | while IFS=: read -r pname ppass puid pgid prest
	do
		echo "USER: $pname"
	done
	echo ""
done < /etc/passwd
```

## # 97
### Vraag
Ontwikkel een script met juist twee parameters. De eerste parameter is de naam van een directory tree, de tweede parameter stelt een aantal bytes voor. Het script genereert de naam van alle bestanden in de direc tory tree waarvan de grootte de waarde van de tweede parameter overschrijdt. Bovendien wordt het totale aantal bestanden dat aan deze voorwaarde voldoet en het totale aantal bytes in deze bestanden gerapporteerd. Tip: Gebruik het find-commando met passende opties om de individuele bestanden te vinden. Gebruik de optie - printf om de noodzakelijke informatie op te vragen tijdens het zoeken 

### Oplossing
```bash
#!/bin/bash
echo "Searching for files in $1 and bigger than $2 bytes"

find $1 -size +$2c -printf "%p:%s\n" | { while IFS=: read -r file size 
do
	((total_size+=size))
	((total_files++))

printf "SIZE %10d FILE %s\n" $size $file
done

echo "TOTAL SIZE: $total_size"
echo "TOTAL_FILES: $total_files"
}
```

> Zie de { } voor het sub shell gebruik!

## # 98
### Vraag
Ontwikkel een script met als parameters een bestandnaam en een willekeurig aantal strings(minstens één). Alle stringparameters die voorkomen in het bestand moeten regel voor regel naar standaarduitvoer worden weggeschreven; de volgorde is hierbij niet van belang.

### Oplossing
```bash
#!/bin/bash
echo "Finding words in $1 param count = $#"

file=$1

shift 1
for ARG in $*
do
	echo "Finding lines with $1"

	IFS=$'\n'

	for line in `grep -E "^.*$1.*$" $file`
	do
		printf "Line: $line\n\n"
	done
	
	shift 1

	printf "\n\n"
done
```

## # 99
### Vraag
Bepaal voor een groep, waarvan het groepsnummer als enige parameter wordt meegegeven, de volledige lijst van gebruikersaccounts die behoren tot deze groep (ook als niet - primaire groep). Construeer twee oplossingen: 
* Schrijf eerst alle gebruikersnamen weg naar een tijdelijk bestand; dubbels zijn voorlopig toegestaan. Filter vervolgens de dubbels hieruit en schrijf de resulterende gebruikerslijst uit. 
* Gebruik een associatieve array, met de gebruikersnamen als sleutels

### Oplossing
```bash

```

## # 100
### Vraag
Ontwikkel een script dat alle parameters uitschrijft die meer dan één keer voorkomen in de argumentenlijst van het script. De volgorde waarin de minstens dubbel voorkomende parameters worden uitgeschreven heeft geen belang (sorteren mag), maar je moet er wel voor zorgen dat parameters die meer dan twee keer voorkomen toch slechts eenmaal weggeschreven worden. Laat als eerste parameter ook eventuele opties -i of -I (van ignore case ) toe, die desgewenst aangeven dat er geen onderscheid mag gemaakt worden tussen hoofdletters en kleine letters. Tip: denk terug aan instructies uit voorgaande oefeningen!

### Oplossing
```bash

```

## # 101
### Vraag
Ontwikkel een script dat een beperkte versie van het commando wc simuleert. Het script moet het aantal regels en de bestandsnaam afdrukken van elk bestand dat als parame ter meegegeven wordt. Het script mag enkel interne Bash - instructies ( if , for , case , let , while , read enz.) gebruiken, en behalve echo geen externe commando's; het gebruik van awk , sed , perl en wc in het bijzonder is niet toegelaten. Je zult bijgevolg elk bestand regel voor regel moeten inlezen en deze tellen. Het script moet bovendien een samenvattende regel weergeven met het totale aantal regels. Indien geen enkele parameter meegegeven wordt, neem je alle bestanden in de huidige werkdirectory in beschouwing. Los dit zo beknopt mogelijk op met de speciale notaties voor shell - variabelen.

### Oplossing
```bash
#!/bin/bash
lines=0

for ARG in $*
do
	# Read lines
	while IFS=$'\n' read -r line
	do
		((lines++))
	done < $1

	echo "$lines $1"

	# Next
	lines=0
	shift 1
done
```

## # 102
### Vraag
Ontwikkel een script dat een directory maakt waarvan het pad als (enige) parameter
aan het script meegegeven wordt. Indien tussenliggende directory's ook nog niet zouden bestaan, moeten deze eveneens gecreëerd worden. Het script simuleert bijgevolg mkdir -p. Het mag enkel interne Bash-instructies (if, for, case, let, while, read enz.) gebruiken, en bovendien het commando mkdir, zij het zonder de optie -p. Zorg ervoor dat zowel absolute als relatieve (t.o.v. de huidige directory) padnamen worden ondersteund. 

Tip: gebruik / als scheidingsteken. 

### Oplossing
```bash
#!/bin/bash
# Simulate mkdir -p
ABSOLUTE_PATH=0

if [[ ${1:0:1} == "/" ]];
    then
    ABSOLUTE_PATH=1
fi

echo "Path: ${ABSOLUTE_PATH}"

# If absolute, cd to /
if [[ $ABSOLUTE_PATH == 1 ]];
    then
    cd /
fi

# Start processing the path creation
IFS='/' read -ra dir <<< $1
for i in "${dir[@]}"; do
    # If empty, skip
    if [[ $i == "" ]];
        then
        continue
    fi

    # Determine if dir exists, if not create it
    if [[ ! -d $i ]];
        then
        mkdir "$i"
    fi

    # Descend into the new dir
    cd "$i"
done
```

## # 103
### Vraag
Enerzijds kun je met behulp van het commando ps -e informatie opvragen over alle
processen die actief zijn. De vier kolommen in de output tonen respectievelijk het
proces-ID (PID), de TTY device file van de (pseudo-)terminal, de CPU time, en het
commando dat het proces opgestart heeft. Anderzijds kun je met behulp van het
commando kill -KILL pid een proces met willekeurig proces-ID afbreken. Ontwikkel een
script dat alle processen afbreekt waarvan het commando één van de strings bevat die
als parameters bij het oproepen van het script meegegeven wordt. Indien geen enkele
parameter meegegeven wordt, moet het script een gesorteerde lijst weergeven van alle
unieke commandonamen van actieve processen. Behalve de interne instructies (if, for,
case, let, while, read enz.) mag je ook de externe commando's grep, sort en uniq
gebruiken. Om problemen te vermijden, schrijf je bij het testen de kill-opdracht uit naar standaarduitvoer i.p.v. deze daadwerkelijk uit te voeren.

### Oplossing
```bash

```

## # 104
### Vraag
Ontwikkel een script dat (zonder gestopt te gebruiken) alle opties die aan het script worden meegegeven naar standaarduitvoer wegschrijft, één per regel. Je moet dus alle karakters die voorkomen in parameters die beginnen met een minteken verzamelen, en deze één voor één verwerken. Bekommer je niet om opties die meerdere keren zouden voorkomen. Voor de argumentenlijst -Ec -rq /etc/passwd -a moet het script dus als uitvoer E, c, r, q en a produceren. Tip: gebruik een lus en stringoperatoren.

### Oplossing
```bash
#!/bin/bash
regex="-([a-zA-Z]+)"

IFS=' ' read -ra input <<< $1
for i in ${input[@]}
do
    # Regex
    [[ $i =~ $regex ]]

    params="${BASH_REMATCH[1]}"

    if [[ $params != "" ]]; then
        for ((i = 0; i <${#params}; i++)); do
            printf "%s " ${params:$i:1}
        done
    fi
done
```

## # 105
### Vraag
Ontwikkel een script dat als eerste parameter een bestandsnaam heeft, en als tweede parameter de naam van een HTML-tag (em, strong, code enz.). Geef een overzicht van alle strings in het bestand die tussen de opgegeven tag staan. Je mag ervan uitgaan dat de tag maximaal één keer voorkomt per regel (zowel de open- als sluittag) en dat de sluittag op dezelfde regel staat als de opentag. Zorg er ook voor dat elke string slechts één keer wordt weergegeven.

### Oplossing
```bash
#!/bin/bash
regex="<$2>(.*)</$2>"

while IFS="\n" read -r line; do
    [[ $line =~ $regex ]]

    if [[ "${BASH_REMATCH[1]}" != '' ]]; then
        echo "${BASH_REMATCH[1]}"
    fi
done < $1
```

## # 106
### Vraag

### Oplossing
```bash
IFS=":"
tel=0
while read account passwd uid gid rest ; do		#regels van /etc/passwd inlezen
    (( tot[gid]++ ))					#frequentietabel opstellen per gid
done < /etc/passwd

for gid in ${!tot[@]} ; do				#alle indices overlopen van de array
    echo "Groep $gid bevat ${tot[gid]} gebruikers"
done
```

## # 107
### Vraag
Ontwikkel een script dat in een directory tree (de eerste parameter) op zoek gaat naar alle bestanden waarvan de naam voldoet aan een bepaald patroon (de tweede parameter) en met behulp van grep in de inhoud van deze bestanden op zoek gaat naar een reguliere expressie (de derde parameter). Van zodra een van de bestanden in een bepaalde directory de expressie bevat, moet de zoektocht in deze directory worden beëindigd. Gebruik een geneste for-lus, waarbij de buitenste for-lus recursief op zoek gaat naar alle directory's, en de binnenste for-lus alle bestanden in een specifieke directory afloopt (niet recursief). In beide lussen wordt de woordenlijst samengesteld op basis van een find-commando met geschikte opties.

### Oplossing
```bash
#!/usr/bin/bash
(( $# == 3 )) || { echo "Gebruik: $0 directory pattern regexp" >&2 ; exit 1 ; }
[[ -d "$1" ]] || { echo "$1 is geen directory" >&2 ; exit 1 ; }

for d in $( find "$1" -type d 2>/dev/null ) ; do					#voor alle subdirs d in directory $1
    for f in $( find "$d" -maxdepth 1 -type f -name "$2" 2>/dev/null ) ; do		#voor alle files in directory d die matchen met pattern $2
        grep -E $3 "$f" > /dev/null 2>&1 && { echo $f ; break ; }			#zoek regex $3 in file f en indien gevonden: echo en break uit binnenste for-lus
    done
done
```

## # 108
### Vraag
Een vaak voorkomend probleem is een groot aantal gebruikersaccounts aanmaken, waarbij alle informatie over deze gebruikers in een bestand te vinden is. Een eerste stap bestaat erin om voor elke gebruiker een unieke gebruikersnaam te genereren, op basis van voor de gebruiker specifieke informatie. Het bestand stud.txt bevat per student zijn/haar studentennummer (bestaande uit acht cijfers), volledige naam (bv. Piet Van Hee), postnummer, type student (A, B …), woonplaats, geboortedatum en klascode. Genereer voor elke student uit dit bestand een gebruikersnaam, rekening houdend met volgende regels:
* Neem de eerste letter van de voornaam, veronderstellend dat die steeds uit
één woord bestaat.
* Neem de eerste significante letter van de familienaam toe. Negeer de
prefixen van, de, den, der.
* Neem ten slotte de laatste 6 cijfers van het studentennummer
Je moet bijgevolg onder meer volgende gebruikersnamen bekomen: TB952571,
SW952641 en BC952578.

### std.txt

### Oplossing
```bash
naam_regex='([a-zA-Z\-]+)\s?([a-zA-Z\-]+)?\s?([a-zA-Z\-]+)?\s?([a-zA-Z\-]+)?\s?([a-zA-Z\-]+)?\s?([a-zA-Z\-]+)?'
while IFS="\n" read -r line; do
    while IFS=":" read -r studentnummer naam postnummer geboortedatum typestudent; do
        # Eerste letter naam
        result=${naam:0:1}

        # Significante letter familienaam
        [[ $naam  =~ $naam_regex ]]

        matches=`echo $naam | grep -o -E '([a-zA-Z\-]+)?' | wc -l`
        result="$result${BASH_REMATCH[matches]:0:1}"

        # laatste 6 cijfers studentennummer
        studentnummer_lengte=`echo $studentnummer | wc -c`
        result="$result${studentnummer:studentnummer_lengte - 7:studentnummer_lengte}"

        printf "%10s %s\n" "$result" "$naam"
    done <<< $line
done < 'stud.txt'
```

## # 109
### Vraag

### Oplossing
```bash

```

## # 110
### Vraag

### Oplossing
```bash

```

## # 111
### Vraag
Ontwikkel een script dat een recursieve versie van het UNIX commando wc simuleert. Behalve de interne instructies (if, for, case, let, while, read enz.) mag je ook de externe commando's echo, wc en find gebruiken. Het script kan opgeroepen worden met 0 tot 3 opties (-l, -w en -c, niet noodzakelijk in die volgorde), gevolgd door een willekeurig aantal parameters. Indien er geen enkele optie meegegeven wordt, wordt aangenomen dat alledrie de opties werden vermeld. De parameters kunnen zowel bestandsnamen als namen van directory's zijn; worden geen parameters meegegeven, dan neemt het script de werkdirectory in beschouwing. De optie -l staat voor het afdrukken van het aantal regels, -w voor het aantal woorden en -c voor het aantal karakters. Deze aantallen worden berekend 
* voor elk bestand dat als parameter meegegeven wordt,
* voor elk bestand in de tree van een directory die als parameter meegegeven
wordt en
* tot slot voor alle bestanden samen. 

### Oplossing
```bash
#!/usr/bin/bash
unset tel_lijnen     # Aantal regels tellen? als boolean gebruiken 0/1
unset tel_woorden    # Aantal woorden tellen? als boolean gebruiken 0/1
unset tel_karakters  # Aantal karakters tellen? als boolean gebruiken 0/1
totaal_lijnen=0
totaal_woorden=0
totaal_karakters=0

while true ; do									# Verwerk en verwijder opties, typisch met een case en een oneindige while + break
    case $1 in
        -c)
            tel_karakters=1
            shift								# optie -c verwijderen uit argumenten
            ;;
        -l)
            tel_lijnen=1
            shift								# optie -l verwijderen uit argumenten
            ;;
        -w)
            tel_woorden=1
            shift								# optie -w verwijderen uit argumenten
            ;;
        -*)
            echo "Gebruik: $0 [-c][-l][-w] [bestand|directory ...]" >&2		# Foutmelding bij optie -x wanneer dit geen -c, -l of -w is
            exit 1
            ;;
        *)
            break  								# Beëindig lus bij eerste niet-optie
            ;;
    esac
done

if [[ -z "$tel_lijnen" && -z "$tel_woorden" && -z "$tel_karakters" ]] ; then 	#indien geen opties meegegeven, dan wordt alles uitgevoerd
    tel_lijnen=1
    tel_woorden=1
    tel_karakters=1
fi

TMP=$( mktemp XXXXXX )  							# Tijdelijk bestand voor uitvoer wc
for parm in "${@:-.}" ; do							# ${variable:-value} indien $@ leeg is (na wissen opties) of ongedefinieerd (niets meegegeven als argument), dan wordt als argumenten '.' genomen (de huidige werkdirectory)
    [[ -f "$parm" ]] && bestanden=$parm						# indien argument een file is, dan beschouwen we die file

    # Invullen met namen van alle bestanden in directory tree
    [[ -d "$parm" ]] && bestanden=$( find "$parm" -type f )			# indien argument een directory is, dan beschouwen we alle files in die directory

    [[ -n "$bestanden" ]] && wc $bestanden 2>/dev/null | grep -v " total$" >>$TMP   # TMP bestand aanvullen met uitvoer van wc voor bestanden
										# Enkel uitvoeren indien minstens één bestand
										# Indien meerdere bestanden verschijnt ook "total", dus verwijder dit
done

										# Haal interessante kolommen uit TMP en bereken totalen
while read lijnen woorden karakters bestandsnaam ; do
    [[ "$tel_lijnen" ]]    && { (( totaal_lijnen += lijnen ))       ; printf "% 10d" $lijnen ; }
    [[ "$tel_woorden" ]]   && { (( totaal_woorden += woorden ))     ; printf "% 10d" $woorden ; }
    [[ "$tel_karakters" ]] && { (( totaal_karakters += karakters )) ; printf "% 10d" $karakters ; }
    echo "    $bestandsnaam"
done < $TMP

										# Druk totalen af
[[ "$tel_lijnen" ]]    && printf "% 10d" $totaal_lijnen
[[ "$tel_woorden" ]]   && printf "% 10d" $totaal_woorden
[[ "$tel_karakters" ]] && printf "% 10d" $totaal_karakters
echo "    total"

rm -f $TMP
```

## # 112
### Vraag
Het bestand pagefile.out bevat de uitvoer van een Windows batch file:
dir \\AL005951\c$\pagefile.sys
dir \\AL005952\c$\pagefile.sys
…
Elk van de 2.901 dir-opdrachten werd beantwoord met regels van de vorm:
C:\WINNT\system32>dir \\AL005951\c$\pagefile.sys
Volume in drive \\AL005951\c$ is WINNT
Volume Serial Number is D0A0-4386
Directory of \\AL005951\c$
02/08/00 02:12 146.800.640 pagefile.sys
 1 File(s) 146.800.640 bytes
 577.850.880 bytes free
 
### Oplossing
```bash

```

TIPS
1.
echo ${tab[@]} | xargs -n1

Neem het 1ste element van standaard uitvoer en echo

2.
parallel doet hetzelfde als xargs, maar gebruikt alle CPUs'
