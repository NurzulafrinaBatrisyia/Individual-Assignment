#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <string.h>

#define MAX_CHILDREN 10
#define MSG_SIZE 50

void parent_handler(int sig);
void child_handler(int sig);

int main() {
    int i, n_children;
    int pfd[MAX_CHILDREN][2];
    int pid[MAX_CHILDREN];
    char msg[MSG_SIZE];

    // set up signal handlers
    signal(SIGINT, parent_handler);
    signal(SIGUSR1, child_handler);

    // prompt user for number of child processes
    printf("Enter number of child processes to fork (max %d): ", MAX_CHILDREN);
    scanf("%d", &n_children);

    // fork child processes
    for (i = 0; i < n_children; i++) {
        if (pipe(pfd[i]) == -1) {
            perror("pipe");
            exit(1);
        }

        pid[i] = fork();

        if (pid[i] == -1) {
            perror("fork");
            exit(1);
        } else if (pid[i] == 0) { // child process
            close(pfd[i][1]); // close write end of pipe
            signal(SIGINT, SIG_IGN); // ignore interrupt signals
            while (1) {
                char buf[MSG_SIZE];
                int n_read = read(pfd[i][0], buf, MSG_SIZE);
                if (n_read > 0) {
                    buf[n_read] = '\0';
                    printf("\nChild %d received message: %s \n", i, buf);
                }
            }
            exit(0);
        } else { // parent process
            close(pfd[i][0]); // close read end of pipe
        }
    }

    // prompt for message once
    printf("Enter message to send to child processes (or 'quit' to exit): ");

    // parent process prompt for message
    while (fgets(msg, MSG_SIZE, stdin) != NULL) {
        if (strcmp(msg, "quit\n") == 0) {
            break;
        }

        for (i = 0; i < n_children; i++) {
            int n_written = write(pfd[i][1], msg, strlen(msg));
            if (n_written == -1) {
                perror("write");
                exit(1);
            }
        }

     
    }

    // kill child processes
    for (i = 0; i < n_children; i++) {
        kill(pid[i], SIGINT);
    }

    return 0;
}

void parent_handler(int sig) {
    printf("\nParent process received interrupt signal (Ctrl + C).\n");
    exit(0);
}

void child_handler(int sig) {
    printf("\nChild process received interrupt signal (SIGUSR1).\n");
}
