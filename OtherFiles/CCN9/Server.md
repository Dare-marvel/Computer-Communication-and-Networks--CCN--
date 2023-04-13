### [Chat application using client server](https://www.geeksforgeeks.org/simple-chat-room-using-python/)

## Brief Description:

## Key Insights:

## Code:
```c
// Include standard C libraries
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

// Initialize client count to 0
int clientCount = 0;

// Initialize mutex and conditional variables for thread synchronization
static pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
static pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

// Structure to hold client details
struct client
{
    int index;
    int sockID;
    struct sockaddr_in clientAddr;
    int len;
};

// Create an array of client structures and an array of thread IDs
struct client Client[1024];
pthread_t thread[1024];

// Networking function to be executed by each thread
void *doNetworking(void *ClientDetail)
{
    // Convert the void pointer argument to a client structure pointer
    struct client *clientDetail = (struct client *)ClientDetail;
    int index = clientDetail->index;
    int clientSocket = clientDetail->sockID;
    // Print a message indicating that the client has connected
    printf("Client %d connected.\n", index + 1);

    // Loop to handle incoming messages from the client
    while (1)
    {
        // Buffer to hold received message data
        char data[1024];
        // Receive data from the client socket
        int read = recv(clientSocket, data, 1024, 0);
        data[read] = '\0';

        // Buffer to hold output message data
        char output[1024];

        // Check if the received message is a request to list connected clients
        if (strcmp(data, "LIST") == 0)
        {
            // Initialize the length variable for building the output message
            int l = 0;
            // Loop through all connected clients (except the current client)
            for (int i = 0; i < clientCount; i++)
            {
                if (i != index)
                    // Add a line to the output message containing the client index and socket ID
                    l += snprintf(output + l, 1024, "Client %d is at socket %d.\n", i + 1, Client[i].sockID);
            }
            // Send the output message back to the client
            send(clientSocket, output, 1024, 0);
            // Continue listening for more messages
            continue;
        }

        // Check if the received message is a request to send a message to another client
        if (strcmp(data, "SEND") == 0)
        {
            // Receive the ID of the target client
            read = recv(clientSocket, data, 1024, 0);
            data[read] = '\0';
            int id = atoi(data) - 1;
            // Receive the message to send
            read = recv(clientSocket, data, 1024, 0);
            data[read] = '\0';
            // Send the message to the target client
            send(Client[id].sockID, data, 1024, 0);
        }
    }
    return NULL;
}

int main()
{
    // Create a new server socket
    int serverSocket = socket(PF_INET, SOCK_STREAM, 0);
    // Initialize the server address structure
    struct sockaddr_in serverAddr;
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(8080);
    serverAddr.sin_addr.s_addr = htons(INADDR_ANY);

    // Bind the server socket to the specified address and port
    if (bind(serverSocket, (struct sockaddr *)&serverAddr, sizeof(serverAddr)) == -1)
        return 0;

    // Listen for incoming connections on the server socket
    if (listen(serverSocket, 1024) == -1)
        return 0;

    // Print a message indicating that the server has started listening on the specified port
    printf("Server started listenting on port 8080 ...........\n");

    // Continuously accept incoming client connections and spawn threads to handle them
    while (1)
    {
        // Accept a new client connection on the server socket
        Client[clientCount].sockID = accept(serverSocket, (struct sockaddr *)&Client[clientCount].clientAddr, &Client[clientCount].len);

        // Set the index of the new client in the Client array
        Client[clientCount].index = clientCount;

        // Spawn a new thread to handle networking for the new client
        pthread_create(&thread[clientCount], NULL, doNetworking, (void *)&Client[clientCount]);

        // Increment the client count for the next incoming client
        clientCount++;
    }

    // Join all threads and exit the program
    for (int i = 0; i < clientCount; i++)
        pthread_join(thread[i], NULL);
}
```
