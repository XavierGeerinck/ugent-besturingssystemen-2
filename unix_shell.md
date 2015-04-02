## 2. UNIX shell

* Interpreter
* Scripttaal
* In /etc/passwd staat welke shell er moet worden aangeroepen bij login
* Intern shellcommando: Onderdeel van shell programma en is onderdeel van het shellproces als uitgevoerd.
* **Extern Shellcommando (!)**: Uitvoerbaar bestand of binaire code, kijkt in volgende volgorde voor bestand te vinden bij **geen / pad**:
    1.  Is intern commando?
        1. Is alias?
        2. Is functie?
        3. Is het intern shellcommando?
    2. Staat het in dir in PATH variabele? (Ga elke dir af in PATH)
    3. Als niet gevonden, print: "Command Not Found"

Bij het **interactief opstarten van bash** gaan we ook bepaalde scripts uitvoeren en environment variables zetten:

1. Eerst lezen we /etc/profile uit
2. Daarna zoekt het systeem achter `~/.bash_profile`, `~/.bash_login` en `~/.profile` (Het systeem voert het 1ste gevonden bestand uit.)
3. In de meeste distributies ook gekeken of `~/.bashrc` bestaat, indien dit bestaat wordt dit ook uitgevoerd. (Voert ook `/etc/bashrc` uit als dit bestaat omdat dit de eerste lijn is in `~/.bashrc`)

**Hoe shell woorden expandeert:**

1. Splits commandolijn op in tokens
2. Expandeer de woorden afhankelijk van de metatekens
3. Nadat alle woorden geexpandeerd zijn, neem de aanhalingstekens weg

Er zijn 7 soorten expansie:

Expansie|Beschrijving
--------|-------------
Brace expansie|{} kan willekeurige strings genereren, dit door elke permutatie met wat in de braces staat en buiten de braces staat de doen vb: `echo a{d,c,b}e` geeft `ade ace abe` of `echo a{0..9}` geeft `a0 a1 a2 a3 a4 a5 a6 a7 a8 a9`
Tilde expansie|Alles met tilde ervoor wordt beschouwt als mogelijke login naam vb: `~/.bashrc` --> Geeft current user bashrc of `~gast/.bashrc` --> Geeft gast bashrc terug
Parameter expansie|Als woord met $ begint dan treed variabele of parameter expansie op vb: `$var` of `${var}`
Commando substitutie|Voer `` uit 
Arithmetic expansie|Voer uitdrukking uit: vb `$(( uitdr ))`
Woordsplitsing|Kijkt naar `IFS` variabele en doet de token splitsing
Padnaam expansie|Kijkt naar wildcard tekens zoals `*`, `?`, `[`

### 2.1 Soorten Shells
Shell|Uitvinder
-----|---------
sh|Bourne-shell (Steven Bourne),oorspronkelijke shell
bash|Bourne-again-shell,IEEE-POSIX compatibel en bevat sh, csh en ksh elementen + extra's
csh|C-shell (Bill Joy),Standaard aanwezig op BSD-UNIX
tcsh|TENEX C-Shell (Ken Greer), uitbreiding op csh shell (t =  TENEX)
ksh|Korn-shell (David Korn), heeft alle sh shell eigenschappen + beste elementen C-shell + etc

### 2.2 Bash command, history en editing
Commando | Uitleg
---------|--------
.bash_history|houdt history van commando's bij
PIJL UP|Toon vorige command
PIJL DOWN|Toon volgend commando in die lijst
\<HOME\>|Ga naar begin lijn
\<END\>|Ga naar end lijn
\<ALT\>B of \<ALT\>F|Verplaats cursor woord per woord

### 2.3 Gereserveerde woorden
**! case do done elif else esac fi for function if in select then until while { } time [[ ]]**

### 2.4 Metatekens

Teken | Uitleg
:----:|--------
newline of \<CR\>|Beindigt commandolijn
spatie|Afscheiding element
\>|Stuurt output naar file (vb: `echo "test" > test.txt`)
\>\>|Voegt toe aan file
\<|Haalt input van file (vb: `wc <kladblok`)
\||Verbindt uitvoer commando 1 met invoer commando 2 (vb: `ps aux | grep nginx`)
\<\<string|here document
sleep x|Sleep voor x seconden (vb: `sleep 5`)
*|elke teken van 0 tot n
?|1 Willekeurig teken
[abc]|1 van de tekens
;|Afscheiding commando's (vb: `echo "Hello";echo " World";`)
&|Run in background (vb: `lang-durend-commando &`)
(...)|Groepeer commando (vb: `(date;who) | tee kladblok | wc`)
$1 $2 ...|Vervangt $1 ... $9 door argumenten waarmee script is opgestart
\$\*|Alle argumenten
$#|Aantal argumenten
\$0|Naam van het bestand dat het shellscript bevat
\$?|Exit status van het vorige commando
\$\$|Proces indicatie van het shellscript
\$!|Procesindicatie van het laatste met & opgestart commando
\$@|Toont alle argumenten, maar met "" als gebruikt wordt doort "" (vb: `"$@"` print: `"$1" "$2" "$3"`)
$var|waarde shellvariabele var
${var}|Waarde variabele, voorkomt verwaring als samengevoegd met tekst
\\|Gebruikt k in \k
\`tekst\`|voer commando tekst uit en plaats uitvoer daar
'tekst'|Gebruik tekst zonder interpretatie
"tekst"|Gebruikt tekst zonder interpretatie maar interpreteert wel $
\#|commentaar
var=waarde|Assign variabele
&&|Logische AND
\|\||Logische OR
{ comm; }|**Enkel in bash en ksh**, samengesteld commando dat wordt uitgevoerd in huidige shell.
((uitdr))|**Enkel in bash en ksh**, Uitdrukking die geevalueerd wordt volgens aritmische uitdrukkingen
[ uitdr ]|Verkorte notatie voor test uitdr
[[ uitdr ]]|**Enkel in bash en ksh**, Zoals vorige maar met pattern matching
