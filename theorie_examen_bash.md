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
