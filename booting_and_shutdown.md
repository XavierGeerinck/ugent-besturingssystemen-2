# 8. Booting and Shutdown
## 8.1. Het opstartproces
### 8.1.1. Van power-on tot init
1. lees instructies van ROM om boot te starten
2. Laad firmware in, dit is communicatie met de hardware onderdelen.
3. De BIOS lokaliseert het boot programma en start verder op. (Staat op bootable device)
4. Boot laadt de kernel in het geheugen.
5. Er wordt een master boot ingeladen, op windows is dit voorzien. Op linux is dit lilo of grub.
6. Bij het starten van linux verwijst de bootloader direct naar de echte fysieke locatie op schijf van de kernel (Windows gaat programma in bootblok van eerste actieve partitie laden en dan het 2de bootpgoramma uitvoeren die deze doen starten.)
7. Linux start het proces init op (PID=1).

### 8.1.2. Taken van init
* Voert taken uit die beschreven staan in `/etc/inittab`. Deze bevinden zich in /etc of in /sbin.
* Voorbeelden van taken: Mounten disks, toekennen paging areas, schoonmaken disk, testen disk, opstarten x-windows, ...
* Systeemtoestand is meestal multiuser, echter kan men ook opstarten in single user.

### 8.1.3. Meldingen bij het opstarten.
Alle opstart meldingen die op het console verschijnen worden bijgehouden. De functionaliteit die deze controleerd is de *syslogd* daemon en/of de *dmesg* utility. Deze meldingen vinden we terug in */usr/adm/messages* of in */var/adm/messages* of */var/log/messages*.

## 8.2. Systeemstatus of runlevel
Er zijn 3 systeemstatussen:
* uitgeschakeld
* multi user mode
* single uer mode

Ook zijn er verschillende runlevels, deze bepalen wat er kan gebeuren op het systeem.


|runlevel|Betekenis|
|:-:|-|
|0|powerdown: veilige conditie om spanning af te leggen|
|1|systeemadminstratie modus|
|S of s|single user mode|
|2|multi user mode zonder netwerk|
|3|multi user met netwerk|
|4|door admin te definieren status, niet gebruikt standaard|
|5|firmware state: speciale onderhoudsmodus|
|6|shutdown en reboot status: gebruikt om opnieuw op te starten.|

De betekenis van deze runlevels staat meestal vermeld in het bestand */etc/inittab*.

## 8.3. Activering en organisatie van de opstartscripts
### 8.3.1. Configuratie van init /etc/inittab
Deze file bepaalt de activiteiten die het init-proces zal uitvoeren. Voor de betekenis zie deze file.

De file wordt geformatteerd in `label:runlevel:actie:proces` volgorde.

### 8.3.2. Hierarchie
De scripts die worden geladen on boot vinden we of terug in */etc/rc\** of in */etc/rd.c*. Door gebruik te maken van het commando `ls  -Rp rc.d` in de */etc* dir kunnen we een recursieve lijst bekomen met alle scripts.