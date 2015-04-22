## 4. Scripting
### 4.1 Propere comment block

```bash
#!/bin/bash
#============================================================
#          FILE:  install.sh
#         USAGE:  ./install.sh sitename
#   DESCRIPTION: This script will install the site with the given configuration
#
#       OPTIONS:  $1 The Imagename for the installed DockerContainer
#       OPTIONS:  $2 The name to call it after installation
#
#  REQUIREMENTS:  /
#        AUTHOR:  Xavier Geerinck (thebillkidy@gmail.com)
#       COMPANY:  /
#       VERSION:  1.3.0
#       CREATED:  18/08/13 20:12:38 CET
#      REVISION:  1.0 - Base structure
#                 1.1 - Added Port Forwarding
#============================================================
```

### 4.2 Oproepen met argumenten

**commando:** 

```bash
./test.sh testing123
```

**Inhoud:**

```bash
#!/bin/bash
echo "You entered: $1"
```

**Uitvoer:** 

```bash
You entered: testing123
```

### 4.3 Controleer argumenten
**commando:** 

```bash
./test.sh
```

**Inhoud:**

```bash
#!/bin/bash
# Check parameters
if [ -z "$1" -o -z "$2" ]; then
   echo "Usage: YOUR USAGE"
	echo "Example: AN EXAMPLE"

	exit 0
fi
```

**Uitvoer:** 

```bash
Usage: YOUR USAGE
Example: AN EXAMPLE
```

### 4.4 VB met argumenten en grep
**commando:** 

```bash
./1207 anne
```

**Inhoud:**

```bash
#!/bin/bash
telefoonboek=$(cat <<END
leon            220
kris            230
dany            783
mario           802
anne frank      563
anne deman      236
END
)

echo $telefoonboek | grep $1
```

**Uitvoer:** 

```bash
anne frank 563
anne deman 236
```

### 4.5 Arrays
**Inhoud:**

```bash
#!/bin/bash
tabel[32]="Tabel waarde" #Zet tabel index 32 op string
tabel2=(waarde1 waarde2 waarde3) #tabel2[0], tabel[1] en tabel[2] zijn set
tabel3=(waarde1 [2]=waarde2 "waarde3 met spatie" #tabel3[0] = waarde1, tabel[1]=undefined, tabel[2]=waarde2, tabel[3]=waarde3
echo ${tabel[0]} # Print index 0
echo ${tabel[*]} # Print alle waarden
echo ${#tabel[*]} # Print lengte van aant ingevulde elementen
echo ${#tabel[0]} # Print lengte van elementwaarde
```

### 4.6 Expanderen van variabele
```bash
# Vb 1:
var1='een twee drie'
een=1
v=var$een
echo $v			# Outputs: var1
echo ${!v}		# Outputs: een twee drie (Expandeer eerst in ${var1})

# Vb 2:
set a b c
echo $#			# Outputs: 3	(Print aant. arg)
echo ${!#}		# Outputs: c	(Eerst expanden in ${3} want aant. arg is 3)
echo ${!een}	# Outputs: a (Expand in ${1} wat de waarde 1 heeft)
```

### 4.7 Rekenen in bash
Hier gebruiken we het `expr` commando, dit was beschikbaar in de Bourne-shell en heeft beperkte mogelijkheden, welke uitgebreid zijn in bash.

```bash
tel=1
tel=`expr '(' $tel + 1 ')' \* 5`	# Tel 1 op bij tel en vermenigvuldig met 5
echo $tel								# 10
expr 25 \* 6							# 150
expr $tel '<' 12						# 1 (True)
echo $?								# 0 (Geen exit status)
echo $tel '>' 12						# 0 (False)
echo $?								# 1 (Wel een exit status)

# Kijk of numeriek is
a=tekst
expr $a + 1	# Geeft non-numeric argument
echo $?		# Print 3

# Of
expr $a + 1 >/dev/null
echo $?		# Print terug 3
```

in bash hebben we echter toegang tot het `let` commando, dit commando kent alle operatoren van C++

```bash
a=5
((b=(12*(4+2)-a)/2))	# We kunnen (( uitdr )) gebruiken ipv let. (korter)
echo $b					# Prints 33
echo $((5*$b))			# Prints 165 (Plaats $ voor uitdrukking om direct te expanderen)
```

### 4.8 While
```bash
# Commando uitleg
while commando
	do
		# Do dit
	done

# Vb
tel=1
while $expr $tel '<' 10 >/dev/null
	do echo -n $tel; tel=`expr $tel + 1`
done

# Prints 123456789
```

### 4.9 IF
```bash
# Commando uitleg
if command
	then
		# commando's
	elif
		then
			# Commando's
	else
		# Commando's
fi

if command
```

### 4.10 test commando of []
uitdrukking|beschrijving
-----------|-----------
-r|Kijkt of bestand leesbaar is
-w|Kijjkt of bestand schrijfbaar is
-f|Kijkt of bestand bestaat
-d|Kijkt of directory bestaat
-s|Kijkt of bestand bestaat en > 0 bytes
-x|Kijkt of bestand uitvoerbaar is
-z|Kijkt of inhoud niet leeg is
-n|Kijkt of inhoud uit meer dan 1 teken bestaat
tekst1 = tekst1|Kijkt of inhoud gelijk is van de 2 tekst vars
tekst1 != tekst1|Kijkt of tekst ongelijk is
getal1 -eq getal2|Kijkt of equal
getal1 -ne getal2|Not equal
getal1 -gt getal2|Greater than
getal1 -ge getal2|Greater than or equal
getal1 -lt getal2|Lesser than
getal1 -le getal2|Lesser then or equal
-o|OR
-a|AND
!|Not

```bash
if [ -z "$1" -o -z "$2" ]; then
	# Do something
fi
```

### 4.11 Lezen STDIN met read
```bash
# Vb1
echo -n "Type iets: "
read iets
echo -e "\nDit heb je ingetypt: $iets\n\a"

# Vb2
read iets
# Wacht op <CTRL>D
echo $iets

# Vb3
while read var1 var2 var3
	do
		# Verwerking var1 var2 var3
	done
```

### 4.12 FOR
```bash
# Commando uitleg
for var in lijst-van-woorden
do
	# Commando's die $var gebruiken
done

# Vb
for file in *
do
	echo $file
done
```

### 4.13 CASE
```bash
# Commando uitleg
case test-string in
	patroon1)
		# Do something
		;;
	patroon2)
		# Do something
		;;
	patroon3)
		# Do something
		;;
	...
esac

# Vb
$commando=d
case $command in
	d|D) date						;;
	w|W) who						;;
	l|L) ls -al 					;;
	p|P) pwd						;;
	a|A) echo "Something"			;;
	*) echo "Verkeerde keuze"		;;
esac
```

### 4.14 Kijk of gebruiker ingelogd
```bash
if [$# = 0]
then
	echo "Geef user-id's op!!" 1>&2; exit 1
fi

for uid in $*
do
	if who|grep $uid >/dev/null
	then
		echo "$uid is ingelogd"
	else
		echo "$uid is niet ingelogd"
	fi
done
```

### 4.15 Lees /etc/passwd
```bash
maxuid=0
IFS=:
while read username password userid rest
	do
		echo $username
		if [$userid -gt $maxuid ]
			then maxuid=$userid
		fi
	done < /etc/passwd
echo $maxuid
```


### 4.16 Speciale Variabelen Patterns
|Pattern|Bash Min. Versie|Betekenis|
|-------|-----------|---------|
|`$(#variabele)`|Bash v1|Geeft lengte van de waarde van de variabele in karakters|
|`$(variabele#pattern)`|Bash v1|Verwijdert de korst mogelijke substring vooraan de variabele die aan het pattern voldoet|
|`${variabele##pattern}`|Bash v1|Verwijdert de langst mogelijke substring vooraan de variabele die aan het pattern voldoet|
|`${variabele%pattern}`|Bash v1|Verwijdert de korst mogelijke substring achteraan de variabele die aan het pattern voldoet|
|`${variabele%%pattern}`|Bash v1|Verwijdert de langst mogelijke substring achteraan de variabele die aan het pattern voldoet|
|`${variabele:offset}`|Bash v2|Geeft de deelstring vanaf positie offset|
|`${variabele:offset:length}`|Bash v2|Geef de deelstring vanaf positie offset met length tekens|
|`${variabele/pattern/string}`|Bash v2|Geeft de string die ontstaat als je in de variabele de eerste en langste substring die aan het pattern voldoet, vervangt door string|
|`${variabele//pattern/string}`|Bash v2|Geeft de string die ontstaat als je in de variabele alle substring die aan het pattern voldoen, vervangt door string|
|`${variabele/#pattern/string}`|Bash v2|Geeft de string die ontstaat als je de langste substring vooraan de variabele, die aan pattern voldoet, vervangt doot string.|
|`${variabele/%pattern/string}`|Bash v2|Geeft de string die ontstaat als je de langste substring achteraan de variabele, die aan pattern voldoet, vervangt door string.|
|`${variabele^}`|Bash v4|Converteert de eerste letter van de string naar uppercase|
|`${variabele,}`|Bash v4|Converteert de eerste letter van de string naar lowercase|
|`${variabele^^}`|Bash v4|Converteert de volledige string naar uppercase|
|`${variabele,,}`|Bash v4|Converteert de volledige string naar lowercase|
