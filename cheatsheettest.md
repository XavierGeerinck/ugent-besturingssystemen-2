# CheatSheetTest
## C++
|Commando|Betekenis|vb|
|-|-|-|
|cc hello.c -o hello|Compileer hello.c||
|make hello|Compileer hello.c||
|iopl|Veranderd het I/O privilege level||
|perror|Print een system error bericht||
|outl|low-level port input en output||
|read|lees in van een file descriptor|`ssize_t read(int fd, void *buf, size_t count);`|
|write|Schrijf naar een file descriptor|`ssize_t write(int fd, const void *buf, siwe_t count);`|
|open|open en mogelijks create een bestand of device|`int open(const char*pathname, int flags, mode_t mode)`|
|clock|vraagt de tijd op|`clock()`|
|CLOCKS_PER_SEC|clock ticks per seconde global variabele||
|malloc|assign geheugen, vergeet free niet te doen||
|fork|maakt een kind proces aan|`pid_t fork(void)`|
|getpid|vraagt pid op van het proces, gebruikt om pid kind te weten.|`pid_t getpid(void)`|
|execve|voert een programma uit|`int execve(const char*filename, char *const argv[], char *const envp[])`|
|waitid, waitpid|Wacht op en process om van state te veranderen|`waitid(P_ALL, pid, NULL, WEXITED);`|
|pipe|maakt pipe aan, pipefd[0] = lees kant, pipefd[1] = schrijf kant|`int pipe(int pipefd[2]);`|

> **IMPORTANT NOTES:**
> * Bij pipes, sluit niet gebruikt einde! (close(fd[0]) or close(fd[1]))
> * Bij pipes, gebruik 2 verschillende fd's als moet kunnen schrijven en lezen.

## Bash

## Voorbeelden
### C++
#### 1.
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <fcntl.h>
    #include <unistd.h> // read
    #include <stdlib.h> // rand, srand
    #include <time.h> // time
    #include <stdio.h> // perror
    #include <pthread.h> // malloc
    
    int main(int argc, char** argv) {
    
    
        // Read 10Mb in decrementing BUF_SIZES into the buffer
        for (int i = 1; i < 8192; i <<= 1) {
            // Create buffer to load it in
            char *buffer = (char *)malloc(sizeof(char) * i);
    
            // Start timing
            double start = clock();
    
            // Open file in write only, create if not exist and always truncate mode.
            int fd;
    
            // Open file
            if ((fd = open("test.txt", O_RDONLY)) < 0) {
                free(buffer);
                perror(argv[0]);
                exit(1);
            };
    
            // Keep reading the count while count == i, this means we have still remaining bytes
            int count;
    
            if ((count = read(fd, buffer, i)) < 0) {
                free(buffer);
                perror(argv[0]);
                exit(1);
            }
    
            while (count == i) {
                if ((count == read(fd, buffer, i)) < 0) {
                    free(buffer);
                    perror(argv[0]);
                    exit(1);
                }
            }
    
            double end = (clock() - start) / CLOCKS_PER_SEC;
            printf("BUF_SIZE=%6d Time=%4f\n", i, end);
    
            free(buffer);
    
            if (close(fd) < 0) {
                free(buffer);
                perror(argv[0]);
                exit(1);
            };
    
        }
    
        return 0;
    }
    
#### 2. Forking

    #include <unistd.h>
    #include <sys/types.h>
    #include <stdio.h>
    #include <stdlib.h>

    int main(int argc, char** argv) {
        pid_t pid;
    
        // Create 3 childs
        for (int i = 0; i < 3; i++) {
            if ((pid = fork()) < 0) {
                // Error
                perror(argv[0]);
            } else if (pid == 0) {
                // Child
                printf("pid: %d\n", getpid());
                exit(0); // Exit child
            } else {
                // Parent
            }
        }
    
        return 0; // exit
    }

#### 3. Waitid

    #include <stdlib.h> // exit
    #include <unistd.h> // fork, execv
    #include <stdio.h> // perror
    #include <sys/types.h> // waitid
    #include <sys/wait.h> // waitid
    #include <sys/stat.h>
    
    int main(int argc, char** argv) {
        pid_t pid;
    
        if ((pid = fork()) < 0) {
            perror(argv[0]);
            exit(1);
        } else if (pid == 0) {
            // CHILD
            // call writestring with argument hello
            char *args[] = {"/takeoff/ugent/c++/writestring_externcall", "HelloWorld", 0};
    
            if (execv("writestring", args) < 0) {
                perror(argv[0]);
                exit(1);
            }
    
            exit(0);
        } else {
            // parent
            waitid(P_ALL, pid, NULL, WEXITED);
        }
    
        printf("DONE\n");
    
        return 0;
    }


### Bash