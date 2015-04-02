## 1. Filesysteem : Directorystructuur en commando's
```
/
|-- /bin
|-- /dev
|   |-- dsk
|   |-- pts
|   |-- rdsk
|   |-- rmt
|   `-- term
|-- /sbin
|-- /etc
|   |-- auth
|   |-- default
|   |-- init.d
|   |-- rc0.d
|   |-- rc2.d
|   |-- rc3.d
|   `-- skel
|-- /home
|-- /lost+found
|-- /mnt
|-- /proc
|-- /tcb
|-- /tmp
|-- /usr
|   |-- bin
|   |   `-- X11
|   |-- games
|   |-- include
|   |-- lib
|   |-- local
|   |   `-- bin
|   |-- sbin
|   `-- share
|       `-- man
|-- /var
|   |-- adm
|   |-- cron
|   |-- mail
|   |-- netls
|   |-- news
|   |-- opt
|   |-- preserve
|   |-- spool
|   |   |-- cron
|   |   |-- lock
|   |   |-- ip
|   |   |-- mail
|   |   |-- mqueue
|   |   |-- uucp
|   |   `-- uucpublic
|   `-- tmp
`-- /stand
```

Directory|Content
---------|--------
/|Root dir, hier begint alles
/bin|Houdt binaries bij
/boot|Kernel compiled versies
/dev|Device directory,gebruikt voor I/O vam de hardware
/etc|etcetera, vooral config bestanden
/sbin|Binaries van de systeem en administratieve commando's (vb: chown, chmod, …)
/home|Home dir van de users
/lost+found|fouten en verloren bestanden komen hier
/mnt|cdrom's,usb's,….
/proc|processen
/tcb|beveiligingsfaciliteiten
/tmp|temp bestanden, global leesbaar
/var|volatiele gegevens, vb post, crons, ...
/stand|Bevat ook beeld van kernel
/usr|lokaal geplaatste commando's enzo
/usr/adm|Houd bij wat er allemaal verkeerdloopt op het systeem, administratieve dir
/usr/bin|binaries die publiek kunnen worden geactiveerd
/usr/include|C++ headers (include files)
/usr/lib|Libraries (iostream, math, ...)
/usr/local|zelf geinstalleerde software
/usr/share|bestanden die gedeeld worden door iedereen, vb: spellingscontrole, fonts, docs, ...
/usr/man of /usr/share/man|manual pages

het linux filesysteem werkt met inodes, dit zijn eigenlijk pointers naar info van de directories (dus eigenlijk meta verwijzingen). Hierdoor hebben we ook toegang tot `.` en `..` wat ons naar het huidige pad begeeft en naar het parent pad.
