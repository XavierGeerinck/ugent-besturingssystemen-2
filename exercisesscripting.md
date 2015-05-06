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

### Oplossing
```bash

```

## # 99
### Vraag

### Oplossing
```bash

```

## # 100
### Vraag

### Oplossing
```bash

```