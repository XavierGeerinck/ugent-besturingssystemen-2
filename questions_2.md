# Questions 2
##  Verschil hard links en soft links
### Hard Links
* Associeert 2+ filenamen met dezelfde inode.
* Ze delen dezelfde datablokken op schijf maar tonen wel als 2 verschillende directories.
* Aanmaken door: `ln brief hlink`

### Soft Links
* Deze zijn pointer files die een bestand een andere naam geven in een andere directory.
* Is een nieuw bestand met een nieuwe inode en een nieuw datablok. 
* Aanmaken door: `ln -s brief slink`
* Krijgt **l** voor de rechten: `lrwxrwxrwx 1 ghv users 5 Nov 17 17:56 slink -> brief`
* Als doelbestand verwijderd, dan foutboodschap

## Bij schijfindeling, wat moet er staan indien het de enige partitie op de schijf is?
* bootblok
* partitieblok
    * super blok
    * blokken met inodes
    * blokken met data

## Wat is een superblok?
* Eerste blok op elk filesysteem
* Beschrijft status van filesysteem en bevat:
    * omvang
    * naam filesysteem
    * naam volume
    * aantal files die het kan bevatten
    * tijdsaanduidingen over laatste update en backup
    * lijst met vrije datablokken
    * lijst met vrije inodes

## Welke info bevat een inode?
* Identificatie van eigenaar
* Type object (regular file, dir, special file, softlink, socket, pipe, ...)
* Toegangsrechten
* 3 tijdstippen: access time, modify time, change time
* Aantal links naar file
* Omvang file
* tabel met adressen van de datablokken
* Check met: `ls -i file`

## Wat is de bedoeling van de init.d en rcX.d mappen en bestanden?
* Bij verandering runlevel gaat /etc/rc.d/rc worden uitgevoerd met het runlevel nummer
* de bestanden in de rcX.d map uitgevoerd worden met X de nummer van het runlevel
* In init.d staan dus de init bestanden (opstart systeem roept /etc/rc.d/rc.sysinit op)