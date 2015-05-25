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