## Labo's
### Labo 1 + 2
#### Deel 1

1. 
    1. CTRL + C
    2. CTRL Z
    3. CTRL + D (EOF)
    4. CTRL + W
    5. /
   
2. 
    1. /pattern
    2. man -i
    3. 9
    4. man 5 passwd (man -a geeft alle)
    5. -l = long listing format -h = human readable format
    6. /
    7. alias
    8. /
    9. /
    10. / 

#### Deel 2

```bash
cc hello.c -o hello
```

```bash
make hello
```

static include alle libraries, waardoor de output groter wordt.
#### Deel 3
**Valgrind:** 

```bash
valgrind ./prog
```

**time:** 

```bash
time ./test
```

**strace:**

```bash
strace ./prog
strace -c ./prog
```
#### Deel 4
Simulatie `lspci -n`, dit geeft alle apparaten weer op de psci bus.

PCI heeft maximum 256 bussen<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Deze bussen hebben elk maximum 32 apparaten<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Elk apparaat heeft maximum 8 functies<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;En elke functie heeft 256 registers

IndexRegister IN&nbsp;&nbsp;&nbsp;&nbsp; = `0xCF8`<br />
IndexRegister OUT = `0xCFC`

**Structuur van het IN pakket:**

```
1 bit: reserved = 1
7 bit: reserved
8 bit: bus nummer
5 bit: device ID
3 bit: functie ID
8 bit: register nummer
```

**Structuur van het OUT pakket:**

```
16 bit: device ID
16 bit: vendor ID
```

We gaan dus alle functies voor elk apparaat in elke bus afscannen door 3 for loops in elkaar te steken, de eerste varieert van 0 -> 255 (bussen), de 2e van 0 -> 32 (apparaten op bus) en de laatste van 0 -> 7 (functies voor de apparaten)

Gebruikte functies:

Functie|Betekenis
-------|--------
iopl|Krijg toegang tot I/O (Max level = 3), vb: iopl(3)
ioperm|Zet poort I/O permissies
perror|gaat error uitsturen, vb: perror(argv[0])
outl|outl(data, address), vb: outl(5,0xcf8) gaat 5 in het adres 0xcf8 steken.
inl|inl(address), vb: inl 0x278 gaat de inhoud van het adres inlezen.

**Oplossing:**

```c
#include <sys/io.h>
#include <stdint.h>
#include <stdio.h>

int main(int argc, char ** argv) {
	if (iopl(3) < 0) {
		perror(argv[0]);
		return 1;
	}
	
	int busnummer, devicenummer, functienummer;
	
	for (busnummer = 0; busnummer < 256; busnummer++) {
		for (devicenummer = 0; devicenummer < 32; devicenummer++) {
			for (functienummer = 0; functienummer < 8; functienummer++) {
				int getal = (1 << 31) + (busnummer << 16) + (devicenummer << 11) + (functienummer << 8);
				outl(getal, 0xCF8);
				
				int antwoord;
				
				if ((antwoord = inl(0xCFC) != 0xFFFFFFFF) {
					printf("bus = %X device = %X functie = %X vendorID = %X deviceID = %X\n", busnummer, devicenummer, functienummer, antwoord & 0x0000FFFF, antwoord >> 16);
				}
			}
		}
	}
}
```
#### Deel 5
File Input en Output met behulp van de linux aanweizge binaries.

Hier gebruikten we de file descriptors van linux om zo een file uit te schrijven.

De gebruikte commando's waren:

Commando|Beschrijving
--------|------------
open|Open file in de opgegeven mode `open(filename, O_WRONLY | O_CREAT`
read|Lees van file `read(fd, buffer, count)`
write|Schrijf naar file `write(fd, buffer, count)`
close|Sluit de file descriptor `close(fd)`

**1) Code:**

```c
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>

int main(int argc, char ** argv) {
	int fd, i;
	srand(time(0));
	
	if ((fd = open("test.txt", O_CREAT | O_WRONLY)) < 0) {
		perror(argv[0]);
		return 1;		
	} else {
		/* Write 10 Mb */
		for (i = 0; i < 10000000; i++) {
			char c = 'a' + (rand() % 26);
			
			if (write(fd, &c, 1) != 1) {
				perror(argv[0]);
				close(fd);
				return 1;
			}
		}
	}

	fclose(fd);
	
	return 0;
}
```


**2) Code:**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <pthread.h>

int main() {
    int i;
    for (i = 1; i < 8192; i <<= 1) {
        char* buffer = malloc(i * sizeof(char));
        double start = clock();
        int fd;

        if ((fd = open("bigfile", O_RDONLY)) < 0) {
            return 1;
        }

	int count;
        if ((count = read(fd, buffer, i)) < 0) {
             free(buffer);
             return 1;
        }

	while (count == i) {
            if ((count == read(fd, buffer, i)) < 0) {
                free(buffer);
                return 1;
            }
	}

	double time = (clock() - start) / CLOCKS_PER_SEC;

        printf("Time: %f\n", time);
        free(buffer);
        close(fd);
    }
}
```

**3) Code:**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <pthread.h>

int main(int argc, char ** argv) {
	int fd, i, j, blok, count;
	srand(time(0));

	if ((fd = open("test.txt", O_CREAT | O_WRONLY)) < 0) {
		perror(argv[0]);
		return 1;		
	} else {
		/* Write 10 Mb */
		for (blok = 1; blok <= 8192; blok <<= 1) {
			// Create buffer
			char *buffer=malloc(sizeof(char) * blok);
			
			// Fill buffer with random data
			for (j=0;j<blok;j++){
				char c = 'a' + (rand() % 26);
				buffer[j] = c;
			}
	
			double start = clock(); // Start counting
			// i = counter of how many bytes have been written
			i=0;	
			// While we have not written more, write 10m bytes
			while ((i + blok) <= 10000000) {	
				// And make sure we actually wrote them, and save the amounts written to count
				if ((count = write(fd, buffer, blok)) < 0) {
					perror(argv[0]);
					free(buffer);
					close(fd);
					return 1;
				}

				i += count;
			}
			
			// Afterwards write the remaining bytes to write
			write(fd,buffer,10000000 - i);
	
			// Print used time			
			double time = (clock() - start) / CLOCKS_PER_SEC;
			printf("BUFFER: %d, TIME: %f\n", blok, time);

			// And clear the buffer
			free(buffer);
			close(fd);
			unlink("test.txt");
		}
	}

	close(fd)
	
	return 0;
}
```

**4):**
Door de O_SYNC flag toe te voegen gaan we ervoor zorgen dat we rechtstreeks naar de schijf gaan schrijven in plaats van eerst te cachen.

> De schijf zelf gaat wel nog caching doen, we kunnen dit voorkomen door `hdparm -W0 /dev/sda` uit te voeren

**5) Simulate CAT Code:**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char** argv) {
        struct stat st;
        int fd, count = 0, total_read = 0;
        int i = 0;

        for (i = 1; i < argc; i++) {
                if (strcmp(argv[i],"-") == 0) {
                        char buffer[BUFSIZ];
                        count = 0;
                        while ((count = read(0, buffer, BUFSIZ)) > 0) {
                                write(1, buffer, count);
                        }
                } else {
                        // Try opening the file
                        if ((fd = open(argv[i], O_RDONLY)) < 0) {
                                printf("Can not open file.");
                                return 1;
                        }

                        // Get info about the file
                        stat(argv[i], &st);

                        // Create buffer for the file
                        //char* buffer = malloc(sizeof(char) * st.st_size);
                        int blok = 8192;
                        char* buffer = malloc(sizeof(char) * blok);

                        while ((total_read + blok) < (st.st_size * sizeof(char))) {
                                if ((count = read(fd, buffer, blok)) < 0) {
                                        free(buffer);
                                        close(fd);
                                        printf("Error, could not read");
                                        return 1;
                                }

                                total_read += count;

                                write(1, buffer, blok);
                        }

                        // Read rest amount of bytes
                        read(fd, buffer, (st.st_size * sizeof(char)) - total_read);
                        write(1, buffer, blok);

                        // Cleanup
                        free(buffer);
                        close(fd);
                }
        }
}
```

**7) Filewatcher**

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <time.h>
#include <string.h>
#include <stdlib.h>

// argv[0] is the string to watch
int main(int argc, char** argv) {
        struct stat st;
        time_t last_modification = NULL;
        printf("Watching file: %s\n", argv[1]);

        while (1) {
                int err = stat(argv[1], &st);

                if (err != 0) {
                        perror("Error");
                        exit(1);
                }

                if (last_modification && st.st_mtime > last_modification) {
                        printf("File %s modified!\n", argv[1]);
                }

                last_modification = st.st_mtime;

                sleep(0.5);
        }
}
```

####Deel 6
De aanroep van het commando `fork()` gaat een exacte kopie maken van het ouderprocess en dit starten als een kindprocess.

Het pid commando gaat -1 teruggeven voor een fout, 0 als het de child bedoelt, en > 0 als het de parent is.

**vb:**

```c
int main() {
	int pid;
	
	// Create fork, child krijgt 0 terug, parent krijgt pid terug.	
	if ((pid = fork()) < 0) {
		perror(argv[1]);
		exit(1);
	}
	else if (pid == 0) {
		// Wordt uitgevoerd door kind process
	} 
	else {
		// Wordt uitgevoerd door parent process
	}
}
```

`fork()` geeft 0 terug als het een kind is, en een PID als het een parent process is die een kind aanmaakt.

Bij `fork()` zijn er 2 mogelijkheden:

1. Ouderproces is sneller klaar dan kindprocess => De parent gaat stoppen waardoor er childs blijven bestaan zonder parent. Dit wordt een **orphan** genoemd. Deze orphan krijgt het init proces als nieuwe parent.
2. Kindproces is sneller klaar
	1. Ouder wacht
	2. Ouder wacht niet =>  Als een process eindigt, en de parent heeft geen `wait()` gecalled, dan wordt dit een **zombieproces** genoemd.

**1):**
Bij uitvoer van 3x fork() achter elkaar, hoeveel processen worden er aangemaakt?

![http://i.stack.imgur.com/nfmGC.jpg](http://i.stack.imgur.com/nfmGC.jpg)

we gaan dus 1 ouderprocess hebben en 7 kind processen.

> De volgorde waarin deze aangekaamt worden kunnen we niet voorspellen.

**2):**
```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
        int n = 3; // Number of child processes
        int i;
        int pid;

        for (i = 0; i < n; i++) {
                if ((pid = fork()) < 0) {
	        } else if (pid == 0) {
		        printf("%d", getpid());
		        exit(0);
	        }
        }

        waitpid(-1,NULL);
        printf("DONE\n");
}
```

> `execve` gaat een commando uitvoeren in een ander pid. vb: `execve("/bin/ls")`

**3):**

**3.0 Programma `writestring.c`**

```C
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main(int argc, char** argv) {
        printf("ProcessID: %d, argv[0]: %s\n", getpid(), argv[1]);
}
```

**3.1 Roep writestring op in programma van opdracht 2**

We gaan de code voor het oproepen van het writestring programma laten uitvoeren door onze child (daarom steken we dit in if pid == 0). En we laten onze parent wachten op deze PID (dus als fork != 0).
```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(int argc, char **argv) {
        int n = 3; // Number of child processes
        int i;
        int pid;
        char* prog_argv[2];

        for (i = 0; i < n; i++) {
                pid = fork();

                if (pid) {
                        // Wait on this child process to end
                        while(waitid(P_ALL,pid, NULL, WEXITED) != -1) continue;

                        printf("PID: %d\n", pid);
                        continue;
                } else if (pid == 0) {
                        // Child Process
                        prog_argv[1] = "Hello";
                        execv("./writestring", prog_argv);
                        break;
                } else {
                        printf("fork error\n");
                        exit(1);
                }
        }

        waitid(P_ALL,-1,NULL, WEXITED);
        printf("DONE\n");
}
```

**3.2 Vervang execv door execl**

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(int argc, char **argv) {
        int n = 3; // Number of child processes
        int i;
        int pid;
        char* prog_argv[2];

        for (i = 0; i < n; i++) {
                pid = fork();

                if (pid) {
                        // Wait on this child process to end
                        while(waitid(P_ALL,pid, NULL, WEXITED) != -1) continue;

                        printf("PID: %d\n", pid);
                        continue;
                } else if (pid == 0) {
                        // Child Process
                        prog_argv[1] = "Hello";
                        execv("./writestring", prog_argv);
                        break;
                } else {
                        printf("fork error\n");
                        exit(1);
                }
        }

        waitid(P_ALL,-1,NULL, WEXITED);
        printf("DONE\n");
}
```

**4) Multi threaded watchfile**

**watchfile.txt**

	file1.txt
	file2.txt
	file3.txt

**watchfiled.cpp**

```c++
#include <map>
#include <iostream>
#include <fstream>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(int argc, char** argv) {
        std::map<std::string, int> filePidMapping;
        std::ifstream in("watchfile.txt");
        std::string line;
        int pid;

        if (in.is_open()) {
                // Keep repeating
                while (1) {
                        // Loop through all the lines
                        while (getline(in,line)) {
                                std::cout << "Line read: " << line << std::endl;
                                // if we got no PID yet, then start the watcher and save the PID
                                if (!filePidMapping[line]) {
                                        if ((pid = fork()) < 0) {
                                                std::cerr << "Error" << std::endl;
                                                exit(1);
                                        } else if(pid == 0) {
std::cout << "Called fork and is in child" << std::endl;
                                                execl("./watchfile", "./watchfile", line.c_str(), 0);
                                                filePidMapping[line] = pid;
                                        } else {
                                                //waitpid(pid, NULL, 0);
                                        }
                                }
                        }

                        // Start reading from the beginning again
                        in.clear(); // Clear EOF flag
                        in.seekg(0, std::ios::beg);

                        // Delay for a sec
                        sleep(1);
                }
        }
}
```

####Deel 6 IPC
Als we gaan kijken naar hoe het commando `# wc /etc/passwd | less` werkt, dan zien we dat we een fork() krijgen op het bash uitvoeringsprocess. Daar wordt dan wc op uitgevoerd.

Het `|` teken is een anonieme pipe (=unnamed pipe) of een named pipe. waardoor less in dit voorbeeld een kindproces is.

Dus zodra het pipe teken er staat worden er 2 processen aangemaakt.

![IPC2](https://lh5.googleusercontent.com/-zjGWHAYlXMo/VQk5RqRxluI/AAAAAAAAKmg/Oq-DME0IkDM/s0/img9.gif "img9.gif")

Let op, de pipe die in bovenstaande afbeelding staat zijn eigenlijk 2 pipes, wie hieronder.

![IPC3](https://lh6.googleusercontent.com/--AXKmgr3tRM/VQk6LZ8XTDI/AAAAAAAAKm0/X4bL9kOGhbw/s0/cs111-scribe-notes-img1.jpg "cs111-scribe-notes-img1.jpg")

fd[0] gaat reading end fd bevatten van de pipe
fd[1] gaat write end fd bevatten van de pipe 

> Note: we hebben 2 pipes nodig per proces
> Note 2: We sluiten de filedescriptor die we niet gebruiken, dit om deadlocks tegen te gaan.

in code wordt dit:

```c++
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <stdlib.h>
#include <stdio.h>

pid_t processes[6];

int main(int argc, char **argv){
        pid_t pid;

        if (argc!=2){
                printf("One argument expected!\n");
                exit(1);
        } else {
                pid_t process_ids[atoi(argv[1])];
                int fd[2];
                int i;
                for(i=0;i<atoi(argv[1]);i++){
                        if (pipe(fd)<0){
                                perror(argv[0]);
                                exit(1);
                        }

                        // Create fork, check on error
                        if ((pid=fork())<0){
                                perror(argv[0]);
                        }

                        // Child process
                        if (pid==0){
                                // Close read end of pipe, we only write in the child
                                close(fd[0]);

                                // Generate random number
                                srand(getpid());
                                int number=rand();

                                // Write the random number on the pipe
                                if ((write(fd[1],&number,sizeof(int))!=sizeof(int))<0){
                                        perror(argv[0]);
                                        exit(1);
                                }

                                // When done writing, kill child
                                exit(0);
                        }
                        // Parent process 
                        else {
                                // Close writing end of the pipe, parent only reads
                                close(fd[1]);

                                // Wait for all the childs to exit
                                if (wait(NULL)==pid){
                                        // Read the number from the pipe and displays it
                                        int number;
                                        read(fd[0],&number,sizeof(int));
                                        printf("pid=%d value=%d!\n",pid,number);
                                }
                        }
                }
        }

        return 0;
}
```

> IPC INFO: http://www.read.cs.ucla.edu/111/notes/lec16b

**6)**
```c++
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <stdlib.h>
#include <stdio.h>

struct MappingPid {
        int number;
        int pid;
        int fd_write;
};

pid_t processes[6];

int main(int argc, char **argv){
        pid_t pid;
        pid_t wpid;
        int status;

        if (argc!=2){
                printf("One argument expected!\n");
                exit(1);
        } else {
                pid_t process_ids[atoi(argv[1])];
                int fd[2];
                int fd_response[2];
                int i;
                struct MappingPid mappings[atoi(argv[1])];
                int counter = 0;

                for(i=0;i<atoi(argv[1]);i++){
                        // Create first pipe to send the numbers
                        if (pipe(fd)<0){
                                perror(argv[0]);
                                exit(1);
                        }

                        // Create second pipe to send the response
                        if (pipe(fd_response)<0) {
                                perror(argv[0]);
                                exit(1);
                        }

                        // Create fork, check on error
                        if ((pid=fork())<0){
                                perror(argv[0]);
                        }

                        // Child process
                        if (pid==0){
                                // Close read end of pipe, we only write in the child
                                close(fd[0]);
                                close(fd_response[1]);

                                // Generate random number
                                srand(getpid());
                                int number=rand();

                                // Write the random number on the pipe
                                if ((write(fd[1],&number,sizeof(int))!=sizeof(int)) < 0){
                                        perror(argv[0]);
                                        exit(1);
                                }

                                // Wait on the response
                                struct MappingPid winner;
                                if (read(fd_response[0],&winner,sizeof(struct MappingPid)) != sizeof(struct MappingPid)) {
                                        exit(1);
                                }

                                printf("child: %d says: winner is: %d with pid: %d\n", getpid(), winner.number, winner.pid);

                                // When done writing, kill child
                                exit(0);
                        }
                        // Parent process 
                        else {
                                close(fd[1]); // Close writing end of the pipe
                                close(fd_response[0]); // Close reading pipe

                                // Read the number from the pipe and displays it
                                int number;
                                read(fd[0],&number, sizeof(int));
                                printf("pid=%d value=%d!\n",pid,number);

                                struct MappingPid mp;
                                mp.number = number;
                                mp.pid = pid;
                                mp.fd_write=fd_response[1];

                                mappings[counter].number = mp.number;
                                mappings[counter].pid = mp.pid;
                                mappings[counter].fd_write=fd_response[1];

                                // Increment created children
                                counter++;
                        }
                }

                // Find max
                struct MappingPid max;
                max= mappings[0];

                for (i = 1; i < atoi(argv[1]); i++) {
                        if (mappings[i].number > max.number) {
                                max = mappings[i];
                        }
                }

                // Found max, send it to the child
                for (i = 0; i < atoi(argv[1]); i++) {
                        if ((write(mappings[i].fd_write,&max,sizeof(struct MappingPid)) != sizeof(struct MappingPid)) < 0) {
                                perror(argv[0]);
                                exit(0);
                        }
                }

                printf("\nMax number is: %d\n", max);

                // Wait on all the childs to exit
                while ((wpid = wait(&status)) > 0)
                {
                        printf("Exit status of %d was %d (%s)\n", (int)wpid, status, (status > 0) ? "accept" : "reject");
                }
        }

        return 0;
}

```

####Deel 7
**1)**
```c++
#include<stdio.h>
#include<string.h>
#include<pthread.h>
#include<stdlib.h>
#include<unistd.h>

#define N_THREADS 4
pthread_t tid[N_THREADS]; // Container for 4 threads

void* handleThread(void* arg) {
        pthread_t id = pthread_self();
        int i = 0;
        int number = *arg;

        // Every thread does the same, so just print a number 100 times, else check on thread id with a case!
        for (i = 0; i < 100; i++) {
                printf("%d\n", number);
        }

        return NULL;
}

int main(int argc, char** argv) {
        srand(time(NULL));

        int i, err;
        int rand_number = rand() % 100;

        for (i = 0; i < N_THREADS; i++) {
                err = pthread_create(&(tid[i]), NULL, &handleThread, &rand_number);

                if (err != 0) {
                        printf("Can't create thread\n");
                        exit(1);
                }
        }

        for (i = 0; i < N_THREADS; i++) {
                pthread_join(tid[i], NULL);
        }

        return 0;
}
```

### Labo Bash
#### Deel VII

|Commando|Uitleg|
|--------|------|
|mktemp| gaat temporary lege file aanmaken met random naam|
|touch| gaat bestand aanmaken of modify date zetten|
|dd| staat voor convert en copy vb: `dd if=/etc/passwd of=passwd`, dit kopieert passwd naar een lokale passwd file. We kunnen if of of weglaten wat cat gaat simuleren of schrijven naar file. |
|`dd if=/dev/zero of=disk.img bs=1000M count=1`|dit gaat 1 blok van 1000 bytes van /dev/zero en schrijft dit naar disk.img|
|`od -tx1 disk.img`|gaat de inhoud van dit bestand printen (orthodump)|
|`xxd disk.img`| gaat ook een hexadecimale output geven.|
|`od -tc disk.img`|gaat dit als chars printen.|
|`mkfs.ext4 disk.img`|gaat bestandssysteem aanmaken op deze img|
|`mount disk.img /mnt`|gaat disk.img mounten op /mnt|
|`umount /mnt`|gaat unmounten|
|`dd if=/dev/sda of=mbr.img count=1`|gaat /dev/sda (wat mbr is) kopieren naar mbr.img. We kunnen dit dan bekijken met `od -tc mbr.img \| less`|
|`strings mbr.img`|Dit gaat de strings weergeven die men hier kan in vinden, kan ook een directory zijn.|
|`dd if=/dev/sda1 \| ssh root@pcname dd of=/root/test.img`|Dit gaat /dev/sda1 naar STDIO sturen, wat de pipe gaat opvangen en doorsturen over ssh, de ssh gaat dit dan met dd of opslaan in test.img.|
|`du -h /root/test.img`|Gaat de disk usage printen van dit bestand in human readable format.|
|`systemctl enable sendmail`|Gaat de sendmail opstarten tijdens boot|
|`systemctl disable sendmail`|Gaat de sendmail disablen|
|`tail -f`|Gaat het bestand streamen|
|`systemctl list-dependencies`|Gaat de startup services tonen|
|`journalctl --unit sendmail.service`|Gaat alle logboodschappen tonen van de sendmail unit|
|`rm -- -rf`|Verwijdert de file met de name -rf|
|`touch -- -rf`|Gaat bestand -rf aanmaken, dit beschouwt alles achter -- als letterlijke tekst|
|`wc -c /etc/passwd`|Print de wordcount van het /etc/passwd bestand|
|`wc -c < /etc/passwd`|Gaat enkel de wordcount printen en niet het /etc/passwd erachter|
|`man builtin`|Gaat alle inwendige commando's printen|
|`set\|less`|Gaat de variabelen tonen die geset zijn|
|`env`|Print environment variables|
|`sync`|Gaat de file system buffers clearen, wat alles naar disk gaat schrijven wat nog in het geheugen zit.|
|`fdisk -l`|Gaat info van disks tonen|
|`find / -ctime 1`|Vind aanpassingen van exact 24u geleden|
|`find / -ctime -1`|Vind aanpassingen van minder dan 24u geleden|
|`find / -ctime +1`|Vind aanpassingen van meer dan 24u geleden|
|`genisoimage -J --rock -o etc.iso /etc /etc`|Rockridge extensies en andere wat een iso maakt van /etc met goede compatibiliteit.|
|`printf "%x\n" {0..256..2}`|Print alle even nummers van 0 tot 256 in hex|
|echo \{\{0..9\},\{a..f\}\}\{\{0..9\},\{a..f\}\}|Doet hetzelfde als bovenstaande, maar deze keer met enkel braces|
|echo \{\{0..9\},\{a..f\}\}\{\{0..9\},\{a..f\}\}|Print alle hex nummers van 00 tot FF|
|`set -C noclobber`|Laat niet toe om met > files te overschrijven.|
|`echo ok >\| test`|Gaat bij noclobber toch wegschrijven naar file|
|`set +C noclobber`|Gaat noclobber disablen en dan kunnen we files terug overschrijven met >|
|`du -h -s /etc`|Dit gaat de grootte van de directory (-s = summary) /etc nemen|
|`du -h /etc`|Dit gaat alle folders in /etc apart tonen|
|`\>\>`|Gaat appenden, vb: echo "" >> test.txt|
|`du -h / >test 2>>test2`|Stuurt output naar test en error append aan test2|
|`du -h / > /dev/null`|Redirect output naar null en toont enkel errors|
|`cat /dev/null > test`|maakt bestand test leeg|
|`fuser /var/log/messages`|Gaat pid geven van process dat dit bestand gebruikt|
|`ps -aux \| grep -E "^2198"`|Grepped process output en ziet wat dit pid gebruikt|
|`ln -s /dev/null test`|symlink test naar /dev/null wat dit bestand leeg houdt, symlinks nemen enkel een inode in beslage|
|`du / >test 2>&1`|Dit gaat alles aan test toevoegen en errors ook, let op de volgorde|
|`du / &>test`|Hetzelfde als bovenstaand|
|`( echo Hallo; du /etc; ) > test 2>&1`|Gaat eerst echo uitvoeren en daarna hallo; dan gaat men deze allebei groeperen en naar de file test sturen. fouten ook.|
|`{ echo Hallo; du /etc; } > test 2>&1`|Dit gaat een subshell opstarten voor dit commando.|
|`du / 2>&1 >/dev/null \| wc -c`|Dit gaat de fouten naar less sturen en de normale output weggooien.|
|`head -n 3 /etc/passwd | tail -n 1`|Gaat lijn 3 tonen|
|` if [ `wc -l < /etc/passwd` -gt 3 ^C then head -n3 | tail -n1; fi;`|Gaat kijken of de file input genoeg lijnen heeft, anders print hij niets|
|`if (( `wc -l < /etc/passwd` > 3 )); then head -n3 /etc/passwd | tail -n1; fi;`|Doet hetzelfde als bovenstaand, enkel met de nieuwere syntax|
|`dos2unix`|Gaat de DOS <CR><LF> vervangen door unix <LF>|
|`unix2dos`|Gaat de unix <LF> vervangen door windows <CR><LF>|
|`tr a-z A-Z < /etc/passwd`|tr staat voor translate, dit accepteerd geen bestand op de input lijn. we gebruiken dus input redirection. Dit gaat het eerste set veranderen door de tweedes set, dit is dus touppercase|
|`tr -d '\r' < test`|Dit verwijdert de \r's|
|`file /bin/printf`|Gaat allemaal info tonen over de file, als het executable is gaat dit commando bijvoorbeeld tonen dat het een elf32 file is, of deze linked is, gestripped (dat debug code weg is), sha checksum, ...|
|`uniq -d /etc/passwd | sort -u`|We gebruiken sort om de lijnen te sorteren, dit omdat uniqid verwacht dat de lijnen aanhangend zijn.|
|`grep -E "(C\-[a-z] ).*\1" TUTORIAL`|Regex met grep|
|`last | less`|Toont wie er ingelogd was|
|`w`|Toont info over huidige gebruiker|
|`history -c`|Cleared history|
|`sort -t : -k 4,4 -n /etc/passwd`|Gaat splitsen op : en sorteren op het 4de veld, dit is numeriek door de -n optie|
|` cut -d : -f1 /etc/passwd`|Gaat alle gebruikersnamen van /etc/passwd tonen|
|`cut -d : -f1 /etc/passwd | tee test`|Gaat uitschrijven naar scherm en dankzij tee direct naar een bestand.|
|`find /etc -name pass\*`|Gaat alle bestanden vinden die met pass beginnen in de /etc dir, merk op: we gaan dit escapen omdat de shell dit anders expandeert!|
|`find / -type d -name sh\*`|Gaat dirs zoeken die met de naam sh beginnen|
|`find /usr/ -type f -size +1M`|Gaat bestanden zoeken in /usr die groter zijn dan 1M|
|`find /usr/ -type f -size +1M -printf "%s %p\n"`|Toont alle bestanden van vorig commando en print grootte en de naam.|
|`find ~/ -type f -mtime -14 -printf "%p\t%s\t%t\n"`|Toont bestanden die laatst gewijwigd zijn|
|`find /usr -type f -name "*.h" -printf "%h\n" \| sort \| uniq`|Toont alle directories zonder duplicaten die header files bevatten.|
|`find / -type d 2>&1 >/dev/null | cut -d : -f2`|Toont alle bestanden waar we geen toegang tot hebben|
|`find /bin -type f -perm -u=r,-u=w,-u=x`|Toont bestanden waarvoor de huidige gebruiker write, execute en read mogelijkheden heeft.|
|Difference between ${} and $()|${} will use the variable and $() will execute the command within the braces|
|`een=random;twee=een;echo ${!twee}`|Dit gaat random tonen, dit omdat twee verwijst naar een.|
|`echo $SECONDS`|Aantal seconden dat het script aan het draaien is|
|`echo $PWD`|Print Working Directory|
|`echo $RANDOM`|Random getal|
|`echo $PS1`|Current prompt (hoe het er uit ziet)|
|`echo $PS2`|Continuation prompt (hoe het er uit ziet)|
|`cd -`|Switch tussen 2 directories! (GODLY)|
|`read a b c d`|Laat ons toe om meerdere variabelen in te lezen, **note: ** hier gebruiken we dus spaties tussen elke ingevoerde waarde.|
|`set | grep ^IFS`|Toont IFS waarde.|
|`t=$(du /etc/passwd)`|Dit gaat de STDOUT van commando toekennen aan een variabele.|
|t=``du /etc/passwd``|Dit gaat de STDOUT van commando toekennen aan een variabele.|
|`echo $((5 + 5))`|Dit gaat 10 printen, $(()) staat voor Arithmetic Expansion en gaat de rekensom uitvoeren.|
|`SECONDS=0;ls -IR /> /dev/null 2>&1;sleep 5;echo "This took: $SECONDS seconds"`|Gaat uitvoeringstijd berekenen met $SECONDS|
|`shift N`|Gaat input params opschuiven met N, vb: 10 input parameters $1 - $10, $5 wordt $1, $6 $2, $7 $3, ... als shift 4|
|x='dit is een <b>eenvoudige</b> en <b>nuttige</b> oefening'||