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
        printf ( "%s INFO %d:%d %ld : " MSG ";;\n", \
        "TODO_PRINT_TIME", getpid(), getppid(), pthread_self()); \
    }

    #define PRINT_ERROR(MSG, ...) { \
        printf ( "%s ERROR %d:%d %ld %s %s %d : [%d] " MSG ";;\n", \
        "TODO_PRINT_TIME", getpid(), getppid(), pthread_self());	\
        }

    #define CONNECT_CHANNEL_ID 100
    #define COMMUNICATION_CHANNEL_ID 200
    #define REGISTER_REQUEST 1
    #define UNREGISTER_REQUEST 2
    #define CALCULATION_REQUEST 3
    #define MAX_SHM_SIZE 1024

    int connect_shmid, communication_shmid;

    struct connect_channel {
        int client_id;
        int response;
    };

    struct communication_channel {
        int request_type;
        int client_id;
        int data[3]; // Added a third element for operation type
    };

    void handle_sigint(int sig)
    {
        printf("Exiting server, closing connect channel\n");
        // Close connection Shared memory
        shmctl(connect_shmid, IPC_RMID, NULL);
        shmctl(communication_shmid, IPC_RMID, NULL);
        exit(0);
    }

    int main(int argc, char **argv) {
        connect_shmid = shmget(CONNECT_CHANNEL_ID, MAX_SHM_SIZE, 0666 | IPC_CREAT);
        int count;
        if (connect_shmid < 0) {
            perror("shmget (connect channel)");
            exit(1);
        }
        int client_id_ifsame;
        communication_shmid = shmget(COMMUNICATION_CHANNEL_ID, MAX_SHM_SIZE, 0666 | IPC_CREAT);
        if (communication_shmid < 0) {
            perror("shmget (communication channel)");
            exit(1);
        }

        struct connect_channel *connect_channel_ptr = (struct connect_channel *) shmat(connect_shmid, NULL, 0);
        struct communication_channel *communication_channel_ptr = (struct communication_channel *) shmat(communication_shmid, NULL, 0);

        signal(SIGINT, handle_sigint);

        int client_id = 0;
        //int array=[]; // add client id here
        while (1) {
            if (connect_channel_ptr->client_id != client_id) {
                client_id = connect_channel_ptr->client_id;
                printf("Client %d registered : %d, %d , %ld \n", client_id,getpid(), getppid(), pthread_self());
                //printf("Client %d registered\n", client_id);
                connect_channel_ptr->response = 1;
                count = 0;
            }
            if (communication_channel_ptr->request_type == CALCULATION_REQUEST) {
                int num1 = communication_channel_ptr->data[0];
                int num2 = communication_channel_ptr->data[1];
                int operation = communication_channel_ptr->data[2];
                int result;
                if (operation == 1) {
                    result = num1+num2;

                } else if (operation == 2) {
                    result = num1 - num2;

                } else if (operation == 3) {
                    result = num1*num2;

                } else if (operation == 4) {
                    result = num1 / num2;

                } else {
                    printf("Invalid operation type\n");
                    continue;
                }

                printf("Client %d requested calculation of %d %d %d %d %ld\n", client_id, num1, num2,getpid(), getppid(), pthread_self());

                communication_channel_ptr->request_type = 0; // Reset request type
                communication_channel_ptr->data[0] = result;

                printf("Sending result %d to client %d %d %d %ld \n", result, client_id,getpid(), getppid(), pthread_self());

                if (client_id_ifsame = client_id){
                    count = count + 1;
                    printf("Count is :");
                    printf("%d\n",count);
                }
            }
        }

        shmdt(connect_channel_ptr);
        shmdt(communication_channel_ptr);

        shmctl(connect_shmid, IPC_RMID, NULL);
        shmctl(communication_shmid, IPC_RMID, NULL);

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

    #define PRINT_INFO(MSG, ...) { \
        printf ( "%s INFO %d:%d %ld : " MSG ";;\n", \
        "TODO_PRINT_TIME", getpid(), getppid(), pthread_self()); \
    }

    #define PRINT_ERROR(MSG, ...) { \
        printf ( "%s ERROR %d:%d %ld %s %s %d : [%d] " MSG ";;\n", \
        "TODO_PRINT_TIME", getpid(), getppid(), pthread_self());	\
        }


    #define CONNECT_CHANNEL_ID 100
    #define COMMUNICATION_CHANNEL_ID 200
    #define REGISTER_REQUEST 1
    #define UNREGISTER_REQUEST 2
    #define CALCULATION_REQUEST 3
    #define EVEN_OR_ODD 4
    #define IS_PRIME 5
    #define IS_NEGATIVE 6
    #define MAX_SHM_SIZE 1024

    struct connect_channel {
        int client_id;
        int response;
    };

    struct communication_channel {
        int request_type;
        int client_id;
        int data[3];
    };

    int main(int argc, char **argv) {
        int connect_shmid = shmget(CONNECT_CHANNEL_ID, MAX_SHM_SIZE, 0666 | IPC_CREAT);
        if (connect_shmid < 0) {
            perror("shmget (connect channel)");
            exit(1);
        }

        int communication_shmid = shmget(COMMUNICATION_CHANNEL_ID, MAX_SHM_SIZE, 0666 | IPC_CREAT);
        if (communication_shmid < 0) {
            perror("shmget (communication channel)");
            exit(1);
        }

        struct connect_channel *connect_channel_ptr = (struct connect_channel *) shmat(connect_shmid, NULL, 0);
        struct communication_channel *communication_channel_ptr = (struct communication_channel *) shmat(communication_shmid, NULL, 0);

        int client_id = getpid();
        connect_channel_ptr->client_id = client_id;

        printf("Client %d registered\n %d %d %ld\n", client_id, getpid(), getppid(), pthread_self());

        while (1) {
            int request_type;
            printf("Enter request type (1=register, 2=unregister, 3=calculation): ");
            while(scanf("%d", &request_type)!=1);

            if (request_type == REGISTER_REQUEST) {
                connect_channel_ptr->client_id = client_id;
                while (connect_channel_ptr->response != 1)
                {
                    printf("Waiting for response\n");
                    /* Wait for response */
                }
                connect_channel_ptr->response = 0;
               // PRINT_INFO("Client registered %d \n", client_id);
                printf("Client %d registered : %d, %d , %ld \n", client_id,getpid(), getppid(), pthread_self());
            } else if (request_type == UNREGISTER_REQUEST) {
                connect_channel_ptr->client_id = 0;
                //PRINT_INFO("Client %d unregistered\n", client_id);
                printf("Client %d unregistered   %d  %d  %ld  \n", client_id,getpid(), getppid(), pthread_self());
            } else if (request_type == CALCULATION_REQUEST) {
                int num1, num2, operation;
                printf("Enter two numbers: ");
                while(scanf("%d %d", &num1, &num2) != 2);
                printf("Enter operation type (1=add, 2=subtract, 3=multiply, 4=divide): ");
                while(scanf("%d", &operation) != 1);
                communication_channel_ptr->request_type = CALCULATION_REQUEST;
                communication_channel_ptr->client_id = client_id;
                communication_channel_ptr->data[0] = num1;
                communication_channel_ptr->data[1] = num2;
                communication_channel_ptr->data[2] = operation;
                printf("Sending calculation request to server...\n");
                sleep(1);
                if (connect_channel_ptr->client_id == 0) {
                    printf("Server is not available\n");
                    //PRINT_ERROR("SIGCHILD Caught.... Code {%d}", sig);
                    continue;
                }
                //PRINT_INFO("Received result from server: %d\n", communication_channel_ptr->data[0]);
                printf("Received result from server: %d  %d  %d  %ld \n", communication_channel_ptr->data[0],getpid(), getppid(), pthread_self());
            } else {
                printf("Invalid request type\n");
            }
        }
        shmdt(connect_channel_ptr);
        shmdt(communication_channel_ptr);
        return 0;
    }


Hope you understood this 
## Thank You
