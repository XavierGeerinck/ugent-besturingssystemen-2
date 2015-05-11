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
|pthread_create|Maakt nieuwe thread aan|`int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) void( *), void *arg);`|
|pthread_join|Deze functie wacht voor een gegeven thread tot deze klaar is.|`int pthread_join(pthread_t thread, void **retval);`|

> **IMPORTANT NOTES:**
> * Bij pipes, sluit niet gebruikt einde! (close(fd[0]) or close(fd[1]))
> * Bij pipes, gebruik 2 verschillende fd's als moet kunnen schrijven en lezen.
> * read en write kunnen ook naar pipes readen en writen.
> * Zorg zeker dat we een while != sizeof() gebruiken bij read!
> * Bij threads, zorg zeker voor arg struct als meerdere args

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

#### 4. Piping

    // 1 argument: Number of child process to generate.
    // Every child process generates a number and sends it to the parent.
    // The parent then finds the biggest of the numbers and sends that back to the children
    // When the children are done writing this number to the screen the parent exits.
    // ==> 2 pipes, one for reading and one for writing
    
    #include <stdlib.h> // srand, rand
    #include <unistd.h> // pipe, close, write
    #include <stdio.h> // printf
    #include <sys/types.h> // waitpid
    #include <sys/wait.h> // waitpid
    
    typedef struct process {
        int pid;
        int fd_write;
    } process;
    
    int winner_id = 0;
    int winner_value = 0;
    
    int main(int argc, char ** argv) {
        if (argc != 2) {
            printf("Expected 1 parameter.\n");
            exit(1);
        }
    
        process processes[atoi(argv[1])]; // our process table
        int received_numbers = 0;
        int fd1[2];
        int fd2[2];
    
        // Create the processes
        for (int i = 0; i < atoi(argv[1]); i++) {
            // Create the pipe for this process
            if (pipe(fd1) < 0 || pipe(fd2) < 0) {
                perror(argv[0]);
                exit(1);
            }
    
            // Create the child process
            if ((processes[i].pid = fork()) < 0) {
                perror(argv[0]);
                exit(1);
            } else if (processes[i].pid == 0) {
                // CHILD
                close(fd1[0]); // Close unused read end
                close(fd2[1]); // Close unused write end
    
                // Generating the random number
                srand(getpid());
                int number = rand();
    
                // Send this number to the parent
                if ((write(fd1[1], &number, sizeof(int)) != sizeof(int)) < 0) {
                    perror(argv[0]);
                    exit(1);
                };
    
                // Wait for the response
                int winner_pid;
                while (read(fd2[0], &winner_pid, sizeof(int)) != sizeof(int));
    
                // Determine if the process is the winner or not
                if (winner_pid == getpid()) {
                    printf("I'm the winner!\n");
                } else {
                    printf("Process %d is the winner\n", winner_pid);
                }
    
                // Exit child
                exit(0);
            } else {
                // PARENT
                close(fd1[1]); // Close unused write end
                close(fd2[0]); // Close unused read end
    
                processes[i].fd_write = fd2[1];
    
                // Wait for the processes to close
                int number;
                if (read(fd1[0], &number, sizeof(int)) != sizeof(int)) {
                    perror(argv[1]);
                    exit(1);
                }
    
                printf("Number Received: %-10d from process: %d\n", number, processes[i].pid);
    
                // Determine winner
                if (winner_value < number) {
                    winner_id = i;
                }
            }
        }
    
        // Send the winner back
        for (int i = 0; i < atoi(argv[1]); i++) {
            if ((write(processes[i].fd_write, &processes[winner_id].pid, sizeof(int)) != sizeof(int)) < 0) {
                perror(argv[1]);
                exit(1);
            }
        }
    
        // Wait for the childs to exit
        waitpid(-1, NULL, 0);
    
        return 0;
    }


### Bash