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

```

## # 91
### Vraag
Ontwikkel een script om de gebruikersnaam (veld 1 van /etc/passwd) te bepalen aan de hand van een gebruikers-
ID (veld 3 van /etc/passwd) dat je als (enige) parameter aan het script meegegeven hebt. Je kunt de output van het commando grep zowel met cut, read, set als met een array analyseren.

### Oplossing
```bash

```

## # 92
### Vraag


### Oplossing
```bash

```

## # 93
### Vraag


### Oplossing
```bash

```

## # 94
### Vraag
### Oplossing
```bash

```