COIS 3380 lab 6
=====

#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>

#define MAXSIZE 15
int factorial(int n)
{
    if(n!=1)
     return n*factorial(n-1);
	else 
		return 1;
}


int main(int argc, char* argv[])
{
	char *input;
	char *output;
	int status, inval, outval, count;
	int bpipe[2];
	pid_t pid;
	
	if (argc != 2 )
	{               
		printf("*** Error: requires only 1 argument\n");
		exit(1);
	}
	
	if (argv[1] < 0)
	{
		printf("***Error: argument has to be positive\n");
		exit(1);
	}

	if( pipe(bpipe) < 0 )
	{						  
		perror("pipe error\n");
		exit(1);
	}

	if( fcntl(bpipe[0], F_SETFL, O_NONBLOCK) < 0 )
	{						  
		perror( "fcntl\n");
		exit( 2 );
	}
	
	input = argv[1];
	pid = fork();

	if(pid == 0)            
	{
		printf("test1");
		read(bpipe[0], &input, MAXSIZE);

		printf("test2");
		sleep(2);

		printf("test3");
		inval = atoi(input);
		/*outval = factorial (inval);*/
		for(count=1;count<=inval;++count)    
		{
			outval*=count;              
		}
		sprintf(output,"%d",outval);

		printf("The answer from the child is %d\n", output);

		write(bpipe[1], &output, MAXSIZE);

		printf("Child closing pipe ...\n");
		close(bpipe[0]);
		close(bpipe[1]);

	} else if (pid > 0)       /*parent process*/
	{
		write(bpipe[1], &input, MAXSIZE);
		
		while(waitpid(pid, &status, WNOHANG) == 0);                        
		{
			printf("Waiting for the child ...\n");
			sleep(1);
		} 

		read(bpipe[0], &output, MAXSIZE);
		
		printf("The result from the parent is %s\n", output);

		printf("parent is closing pipe ...\n");
		close(bpipe[0]);
		close(bpipe[1]);
	}else
	{
		perror("fork failed\n");
		exit(2);
	}
	return 0;
}
