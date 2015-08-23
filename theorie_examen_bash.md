# Scripts
## Script 1a

```
#!/bin/bash
# ========================================================================
# Aanroep: scriptnaam groepsnaam
# ========================================================================
# Gevraagd:
# ========================================================================
# Schrijf een script met 1 argument, nl. de groepsnaam.
# Toon van elk lid van de groep de 5 grootste files.
# Toon eveneens de totale grootte van die 5 files.
# Voer aparte controles uit op volgende zaken, geef een error indien fout:
# - wordt er een argument meegegeven ?
# - is de groepsnaam geldig ?
# - zitten er gebruikers in de groep?
# Maak tevens verplicht gebruik van:
# - cut
# - find
# - reguliere expressies

# Reset IFS
IFS=

# Kijk of argument meegegeven
if [ -z $1 ]; then
	echo "Geen argument meegegeven"
	echo "Gebruik: $0 <groepsnaam>"
	echo "Vb: $0 _postgres"
	exit
fi

GROUP_NAMES=(`grep -v '^#' /etc/group | cut -f 1 -d ':'`)

# Kijk of groepsnaam geldig, de grep verwijderd #, de cut fetched de naam
if ! [[ ${GROUP_NAMES[@]} =~ $1 ]]; then
	echo "Groepsnaam is niet geldig"
	exit
fi

# Kijk of gebrukers in die groep
GROUP=`grep -E "$1:" /etc/group`
GROUP_USERS=( $(IFS=","; echo $GROUP | cut -f 4 -d ':') )

echo "Info: $GROUP"
echo "Gebruikers: $GROUP_USERS"

if [[ -z ${#GROUP_USERS[@]} ]]; then
	echo "Geen gebruikers in deze groep"
else
	echo "#Gebruikers in deze groep: ${#GROUP_USERS[@]}"
fi

# Toon de 5 grootste files van deze gebruikers
for gebruiker in ${GROUP_USERS[@]}; do
	IFS="
	"
	FILES=( `find /bin -type f -user $gebruiker -exec ls -al {} \; | sort -nr -k5 | head -n 5` )
	IFS=
	
	TOT_GROOTTE=0
	for file in ${FILES[@]}; do
		# Toon file
		echo $file

		# Voeg de grootte toe aan het totaal
		FILE_GROOTTE=`echo $file | cut -f 8 -d ' '`
		let "TOT_GROOTTE+=FILE_GROOTTE"
	done

	echo "Totaal: $TOT_GROOTTE"
done
```

## Script 2

```
#!/bin/bash
# ========================================================================
# Aanroep: scriptnaam naam telefoonboek [nummer]
# ========================================================================
# Gegeven:
# ========================================================================
# Een bestand telefoonboek met daarin verschillende lijnen van het type naam:adres:info:tel1:tel2:tel3
# ========================================================================
# Gevraagd:
# ========================================================================
# Schrijf de telefoonnummer uit (op basis van het meegegeven nummer).
# Indien er geen nummer is meegegeven, neem je standaard 1.
# Schrijf vervolgens uit hoeveel nummers die persoon in het bestand heeft zitten

# Reset IFS
IFS=""

# Kijk of argumenten meegegeven
if [[ -z $1 || -z $2 ]]; then
	echo "Geen argument meegegeven"
	echo "Gebruik: $0 <naam> <telefoonboekfile> [nummer]"
	echo "Vb: $0 \"Xavier Geerinck\" ./tel.txt 2"
	exit
fi

# Zet standaard op 1
if [[ -z $3 ]]; then
	NUMMER=1
else
	NUMMER=$3
fi

TELEFOONBOEK=$2
NAAM=$1
INFO=`grep -E "$NAAM:" $TELEFOONBOEK`

if [[ ! -z `echo $INFO | cut -f 4 -d ':'` ]]; then
	NUMMERS[0]=`echo $INFO | cut -f 4 -d ':'`
fi

if [[ ! -z `echo $INFO | cut -f 5 -d ':'` ]]; then
	NUMMERS[1]=`echo $INFO | cut -f 5 -d ':'`
fi

if [[ ! -z `echo $INFO | cut -f 6 -d ':'` ]]; then
	NUMMERS[2]=`echo $INFO | cut -f 6 -d ':'`
fi

# Schrijf telefoonnummer uit
echo "Telefoonnummer: ${NUMMERS[$(($NUMMER - 1))]}"
echo "Aant nummers: ${#NUMMERS[@]}"
```

## Script 3a

```

```

## Script 3b

```

```

## Script 4

```

```

## Script 5

```

```
