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
|fork|maakt een kind proces aan|pid_t fork(void);|
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

### Bash