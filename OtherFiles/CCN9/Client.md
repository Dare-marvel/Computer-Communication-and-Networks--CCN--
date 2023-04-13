## Brief Description:

## Key Insights:

## Code:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <pthread.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <arpa/inet.h>

// Function to handle recieving messages from the server
void * doRecieving(void * sockID){

    // Extract the socket ID from the parameter passed to the function
    int clientSocket = *((int *) sockID);

    while(1){

        // Recieve data from the server
        char data[1024];
        int read = recv(clientSocket,data,1024,0);

        // Terminate the data string with a null character
        data[read] = '\0';

        // Print the received message to the console
        printf("%s\n",data);

    }

}

int main(){

    // Create a socket for the client
    int clientSocket = socket(PF_INET, SOCK_STREAM, 0);

    // Initialize the server address structure
    struct sockaddr_in serverAddr;
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(8080);
    serverAddr.sin_addr.s_addr = htonl(INADDR_ANY);

    // Connect to the server
    if(connect(clientSocket, (struct sockaddr*) &serverAddr, sizeof(serverAddr)) == -1) return 0;

    // Print a message to confirm that the connection was established
    printf("Connection established ............\n");

    // Create a new thread to handle recieving messages from the server
    pthread_t thread;
    pthread_create(&thread, NULL, doRecieving, (void *) &clientSocket );

    // Wait for user input and send messages to the server
    while(1){

        // Get input from the user
        char input[1024];
        scanf("%s",input);

        // Send the "LIST" command to the server
        if(strcmp(input,"LIST") == 0){

            send(clientSocket,input,1024,0);

        }

        // Send a message to another client
        if(strcmp(input,"SEND") == 0){

            // Send the "SEND" command to the server
            send(clientSocket,input,1024,0);

            // Get the ID of the client to send the message to
            scanf("%s",input);
            send(clientSocket,input,1024,0);

            // Get the message to send to the client
            scanf("%[^\n]s",input);
            send(clientSocket,input,1024,0);

        }

    }

}
```
