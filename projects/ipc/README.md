## Hello this is a simple application used for chatting
### Both the users use the same memory space 

# Server:


    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <pthread.h>
    #include <signal.h>
    #define PRINT_INFO(MSG, ...) { \
        printf("%s INFO %d:%d %ld : " MSG ";;\n", \
        "TODO_PRINT_TIME", getpid(), getppid(), pthread_self()); \
    }

    #define PRINT_ERROR(MSG, ...) { \
        printf("%s ERROR %d:%d %ld %s %s %d : [%d] " MSG ";;\n", \
        "TODO_PRINT_TIME", getpid(), getppid(), pthread_self()); \
    }

    #define SHARED_MEMORY_KEY 1234
    #define SHARED_MEMORY_SIZE 1024

    struct shared_memory {
        int number;
        int is_ready;
    };

    int main()
    {
        int shmid;
        struct shared_memory *shared_mem_ptr;
        // Create shared memory
        shmid = shmget(SHARED_MEMORY_KEY, SHARED_MEMORY_SIZE, 0666 | IPC_CREAT);
        if (shmid < 0) {
            perror("shmget");
            exit(1);
        }

        // Attach shared memory
        shared_mem_ptr = (struct shared_memory *)shmat(shmid, NULL, 0);
        if (shared_mem_ptr == (struct shared_memory *)(-1)) {
            perror("shmat");
            exit(1);
        }

        // Send a number to the client
        shared_mem_ptr->number = 42;
        shared_mem_ptr->is_ready = 1;

        // Wait for the client to read the number
        while (shared_mem_ptr->is_ready) {
            usleep(100000); // Sleep for 100 milliseconds
        }

        // Detach shared memory
        shmdt(shared_mem_ptr);

        // Delete shared memory
        shmctl(shmid, IPC_RMID, NULL);

        return 0;
    }



# Client.c



    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <pthread.h>
    #include <signal.h>
    #define PRINT_INFO(MSG, ...) { \
        printf("%s INFO %d:%d %ld : " MSG ";;\n", \
        "TODO_PRINT_TIME", getpid(), getppid(), pthread_self()); \
    }

    #define PRINT_ERROR(MSG, ...) { \
        printf("%s ERROR %d:%d %ld %s %s %d : [%d] " MSG ";;\n", \
        "TODO_PRINT_TIME", getpid(), getppid(), pthread_self()); \
    }

    #define SHARED_MEMORY_KEY 1234
    #define SHARED_MEMORY_SIZE 1024

    struct shared_memory {
        int number;
        int is_ready;
    };

    int main()
    {
        int shmid;
        struct shared_memory *shared_mem_ptr;

        // Get the shared memory ID
        shmid = shmget(SHARED_MEMORY_KEY, SHARED_MEMORY_SIZE, 0666);
        if (shmid < 0) {
            perror("shmget");
            exit(1);
        }

        // Attach shared memory
        shared_mem_ptr = (struct shared_memory *)shmat(shmid, NULL, 0);
        if (shared_mem_ptr == (struct shared_memory *)(-1)) {
            perror("shmat");
            exit(1);
        }



Hope you understood this 
## Thank You
