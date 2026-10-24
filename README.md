# anoushka_dsa_mini_project
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
 
#define MAX_USERS 50
 
// Represents a link to a friend in your network graph
typedef struct Node {
    int userId;
    struct Node* next;
} Node;
 
// Represents a user profile in the social app database
typedef struct {
    char name[50];
    int exists;      // Flag to check if account is active
    Node* head;      // Head of the adjacency list (friend list)
} User;
 
// Global database for our social platform
User network[MAX_USERS];
 
// Function declarations
void addUser();
void addFriend();
void displayNetwork();
void findMutualFriends();
void findDegreesOfSeparation();
void recommendFriends();
void freeGraph();
Node* createNode(int id);
 
int main() {
    int i, choice;
 
    // Setting up our empty database at startup
    for (i = 0; i < MAX_USERS; i++) {
        network[i].exists = 0;
        network[i].head = NULL;
    }
 
    do {
        printf("\nCONNECTIFY - SOCIAL NETWORK\n");
        printf("1. Create New Profile (Register)\n");
        printf("2. Send Friend Request (Connect)\n");
        printf("3. View Social Feed / Network Graph\n");
        printf("4. Find Mutual Friends\n");
        printf("5. Find Degrees of Separation (Shortest Path)\n");
        printf("6. Friend Recommendations (People You May Know)\n");
        printf("7. Exit App\n");
        printf("Select an option (1-7): ");
 
        if (scanf("%d", &choice) != 1) {
            printf("Oops! Please enter a valid menu number.\n");
            while(getchar() != '\n');
            continue;
        }
 
        switch (choice) {
            case 1: addUser(); break;
            case 2: addFriend(); break;
            case 3: displayNetwork(); break;
            case 4: findMutualFriends(); break;
            case 5: findDegreesOfSeparation(); break;
            case 6: recommendFriends(); break;
            case 7:
                printf("Logging off... Cleaning up app memory. Goodbye!\n");
                freeGraph();
                break;
            default:
                printf("Invalid choice. Please pick between 1 and 7.\n");
        }
    } while (choice != 7);
 
    return 0;
}
 
// Helper: Dynamically creates a new friend connection node
Node* createNode(int id) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->userId = id;
    newNode->next = NULL;
    return newNode;
}
 
// FEATURE 1: REGISTER USER (O(1) Array Indexing)
void addUser() {
    int id;
    size_t len;
 
    printf("Enter unique User ID (0 to 49): ");
    scanf("%d", &id);
 
    if (id < 0 || id >= MAX_USERS) {
        printf("Error: ID is out of range! Keep it under 50.\n");
        return;
    }
    if (network[id].exists) {
        printf("Error: This ID is already taken by %s!\n", network[id].name);
        return;
    }
 
    // Clear buffer before taking full name string input
    while(getchar() != '\n');
 
    printf("Enter your Full Name: ");
    fgets(network[id].name, sizeof(network[id].name), stdin);
 
    // Remove trailing newline character from fgets
    len = strlen(network[id].name);
    if (len > 0 && network[id].name[len - 1] == '\n') {
        network[id].name[len - 1] = '\0';
    }
 
    network[id].exists = 1;
    network[id].head = NULL;
 
    printf("Success! Welcome to Connectify, %s (ID: %d)!\n", network[id].name, id);
}
 
// Helper: Adds a two-way edge in the graph for mutual friendship
void insertEdge(int u, int v) {
    // Add v to u's friend list
    Node* newNode1 = createNode(v);
    newNode1->next = network[u].head;
    network[u].head = newNode1;
 
    // Add u to v's friend list (since friendship is mutual)
    Node* newNode2 = createNode(u);
    newNode2->next = network[v].head;
    network[v].head = newNode2;
}
 
// FEATURE 2: ADD FRIEND CONNECTION (Graph Edge Insertion)
void addFriend() {
    int id1, id2;
    Node* temp;
 
    printf("Enter your User ID: ");
    scanf("%d", &id1);
    printf("Enter the User ID of the person to add: ");
    scanf("%d", &id2);
 
    if (id1 < 0 || id1 >= MAX_USERS || id2 < 0 || id2 >= MAX_USERS || !network[id1].exists || !network[id2].exists) {
        printf("Error: One or both user profiles do not exist.\n");
        return;
    }
    if (id1 == id2) {
        printf("You cannot add yourself as a friend!\n");
        return;
    }
 
    // Check if they are already connected in the adjacency list
    temp = network[id1].head;
    while (temp != NULL) {
        if (temp->userId == id2) {
            printf("You are already friends with %s!\n", network[id2].name);
            return;
        }
        temp = temp->next;
    }
 
    insertEdge(id1, id2);
    printf("Success! %s and %s are now connected friends.\n", network[id1].name, network[id2].name);
}
 
// FEATURE 3: DISPLAY NETWORK (Graph Adjacency List Traversal)
void displayNetwork() {
    int i, found;
    Node* temp;
 
    printf("\nCONNECTIFY NETWORK DIRECTORY\n");
    found = 0;
 
    for (i = 0; i < MAX_USERS; i++) {
        if (network[i].exists) {
            found = 1;
            printf("[%d] %s -> Friends: ", i, network[i].name);
            temp = network[i].head;
            while (temp != NULL) {
                printf("%s (ID:%d)", network[temp->userId].name, temp->userId);
                temp = temp->next;
                if (temp != NULL) printf(", ");
            }
            printf("\n");
        }
    }
    if (!found) printf("No profiles found on the network yet.\n");
}
 
// FEATURE 4: FIND MUTUAL FRIENDS (List Intersection)
void findMutualFriends() {
    int id1, id2, count;
    Node* t1;
    Node* t2;
 
    printf("Enter first User ID: ");
    scanf("%d", &id1);
    printf("Enter second User ID: ");
    scanf("%d", &id2);
 
    if (!network[id1].exists || !network[id2].exists) {
        printf("Error: Invalid user IDs provided.\n");
        return;
    }
 
    printf("\nMutual friends linking %s and %s:\n", network[id1].name, network[id2].name);
    count = 0;
 
    // Compare user 1's friend list with user 2's friend list
    t1 = network[id1].head;
    while (t1 != NULL) {
        t2 = network[id2].head;
        while (t2 != NULL) {
            if (t1->userId == t2->userId) {
                printf("  - %s (ID: %d)\n", network[t1->userId].name, t1->userId);
                count++;
            }
            t2 = t2->next;
        }
        t1 = t1->next;
    }
 
    if (count == 0) printf("  (No mutual connections found)\n");
}
 
// FEATURE 5: DEGREES OF SEPARATION (BFS Shortest Path)
void findDegreesOfSeparation() {
    int start, target, i, found, front, rear, curr, adjUser, pathLen;
    int visited[MAX_USERS];
    int distance[MAX_USERS];
    int parent[MAX_USERS];
    int queue[MAX_USERS];
    int path[MAX_USERS];
    Node* temp;
 
    printf("Enter your User ID: ");
    scanf("%d", &start);
    printf("Enter target User ID to check connection: ");
    scanf("%d", &target);
 
    if (!network[start].exists || !network[target].exists) {
        printf("Error: Invalid user IDs.\n");
        return;
    }
 
    if (start == target) {
        printf("That's you! Distance is 0 degrees.\n");
        return;
    }
 
    // Reset tracking arrays for traversal
    for (i = 0; i < MAX_USERS; i++) {
        visited[i] = 0;
        distance[i] = 0;
        parent[i] = -1;
    }
 
    front = 0;
    rear = 0;
 
    // Standard Breadth-First Search (BFS) Queue Initialization
    queue[rear++] = start;
    visited[start] = 1;
    distance[start] = 0;
 
    found = 0;
    while (front < rear) {
        curr = queue[front++]; // Pop from queue
 
        if (curr == target) {
            found = 1;
            break;
        }
 
        // Check all adjacent friends
        temp = network[curr].head;
        while (temp != NULL) {
            adjUser = temp->userId;
            if (!visited[adjUser]) {
                visited[adjUser] = 1;
                distance[adjUser] = distance[curr] + 1;
                parent[adjUser] = curr; // Track path backwards
                queue[rear++] = adjUser; // Push to queue
            }
            temp = temp->next;
        }
    }
 
    if (found) {
        printf("\nDegrees of Separation: %d step(s)\n", distance[target]);
 
        // Trace back the connection chain
        pathLen = 0;
        curr = target;
        while (curr != -1) {
            path[pathLen++] = curr;
            curr = parent[curr];
        }
 
        printf("Connection Path: ");
        for (i = pathLen - 1; i >= 0; i--) {
            printf("%s (ID:%d)", network[path[i]].name, path[i]);
            if (i > 0) printf(" -> ");
        }
        printf("\n");
    } else {
        printf("\nNo connection path found. You are in separate social islands.\n");
    }
}
 
// FEATURE 6: RECOMMEND FRIENDS (Friends of Friends Algorithm)
void recommendFriends() {
    int id, recommendedCount;
    int isFriend[MAX_USERS];
    Node* temp;
    Node* fof;
    int friendId;
    int candidate;
    int i;
 
    printf("Enter your User ID for recommendations: ");
    scanf("%d", &id);
 
    if (!network[id].exists) {
        printf("User profile does not exist.\n");
        return;
    }
 
    // Mark yourself and current direct friends so we don't re-recommend them
    for (i = 0; i < MAX_USERS; i++) {
        isFriend[i] = 0;
    }
    isFriend[id] = 1;
 
    temp = network[id].head;
    while (temp != NULL) {
        isFriend[temp->userId] = 1;
        temp = temp->next;
    }
 
    printf("\nPeople You May Know (Friend Recommendations for %s):\n", network[id].name);
    recommendedCount = 0;
 
    // Traverse through your friends' friend lists (Friends of Friends)
    temp = network[id].head;
    while (temp != NULL) {
        friendId = temp->userId;
        fof = network[friendId].head;
        while (fof != NULL) {
            candidate = fof->userId;
            // If they aren't already a friend or yourself, recommend them!
            if (!isFriend[candidate]) {
                printf("  - %s (ID: %d):[Mutual Friend: %s]\n",
                       network[candidate].name, candidate, network[friendId].name);
                isFriend[candidate] = 1; // Prevent duplicate suggestions in this session
                recommendedCount++;
            }
            fof = fof->next;
        }
        temp = temp->next;
    }
 
    if (recommendedCount == 0) {
        printf("  (No recommendations right now. Try adding more friends!)\n");
    }
}
 
void freeGraph() {
    int i;
    Node* current;
    Node* next;
 
    for (i = 0; i < MAX_USERS; i++) {
        current = network[i].head;
        while (current != NULL) {
            next = current->next;
            free(current);
            current = next;
        }
    }
}
