1. The fork() system call. 
2. The wait() , waitpid() system call.
3. The execvp() system call
4. the pipe() system call.


# Some programs to understand more about these system calls.


Using a fork() system Call.
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int main(int argc, char *argv[])
{
    printf("hello world(pid:%d)\n", (int) getpid());
    
    int rc = fork();
     
    if (rc < 0) { // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child (new process)
        printf("hello, I am child (pid:%d)\n", (int)getpid());
    } else { // parent goes down this path (main)
         int wc = wait(NULL);
         printf("hello, I am parent of %d (wc:%d)(pid:%d)\n", rc, wc, (int) getpid());
    }
    return 0;
}
```





Creating two processes , one for generating a random integer and then passing its output to other process for calculating its square using pipe() system call. 
``` c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <time.h>

int main() {
    int pipefd[2]; 
    pid_t pid1, pid2;

    // 1. Create the pipe
    if (pipe(pipefd) == -1) {
        perror("Pipe failed");
        return 1;
    }

    // --- CHILD 1: The Generator ---
    pid1 = fork();
    if (pid1 == 0) {
        // Close the unused read end
        close(pipefd[0]);

        // Seed random number generator (using PID to ensure randomness)
        srand(time(NULL) ^ getpid());
        int random_num = rand() % 20 + 1; // Random number between 1 and 20

        printf("[Child 1] Generated number: %d\n", random_num);
        printf("[Child 1] Sending to pipe...\n");

        // Write the raw integer to the pipe
        write(pipefd[1], &random_num, sizeof(int));

        close(pipefd[1]); // Close write end after finishing
        exit(0);
    }

    // --- CHILD 2: The Calculator ---
    pid2 = fork();
    if (pid2 == 0) {
        // Close the unused write end
        close(pipefd[1]);

        int received_num;
        
        // Read from the pipe. This will BLOCK until Child 1 writes something.
        read(pipefd[0], &received_num, sizeof(int));

        int square = received_num * received_num;

        printf("[Child 2] Received number: %d\n", received_num);
        printf("[Child 2] Calculated square: %d\n", square);

        close(pipefd[0]); // Close read end after finishing
        exit(0);
    }

    // --- PARENT ---
    // Parent must close its own copies of the pipe ends
    // otherwise Child 2 might not detect EOF properly
    close(pipefd[0]);
    close(pipefd[1]);

    // Wait for both children to ensure clean exit
    waitpid(pid1, NULL, 0);
    waitpid(pid2, NULL, 0);

    printf("[Parent] All tasks finished.\n");
    return 0;
}
```
  
Other system Call include , using execvp() , explore it from the internet.





 