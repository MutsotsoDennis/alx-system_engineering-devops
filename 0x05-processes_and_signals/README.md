Linux Processes – Environment extern, environ, getenv, setenv
by HIMANSHU ARORA on MARCH 12, 2012
This is the 1st article of a new series on the processes in Linux.

The focus of this series would be on the practical aspects of process environment, process control, process relationships etc.

In this article, we will discuss how to get and set environment variables inside a C program.

Linux Processes Series: part 1 (this article), part 2, part 3

What is a Process?
A process can be thought of as an instance of a program in execution. We called this an ‘instance of a program’, because if the same program is run lets say 10 times then there will be 10 corresponding processes.

Moving ahead, each process has its own unique process ID through which it is identified in the system. Besides it own ID, a parent’s process ID is also associated with a process.

The main() Function
A ‘C’ program always starts with a call to main() function. This is the first function that gets called when a program is run.

The prototype of a main() function is :


int main(int argc, char *argv[]);
In the above prototype :

The return type of the main() function is ‘int’. This is because,, when the main() function exits, the program finishes. And a return type from main() would signify whether the program got executed properly or not. In strict sense we say that if main() returns ‘0’, then the program got executed successfully. Any other return value indicates a failure.
The main() function accepts two arguments. One is the number of command line arguments and the other is the list of all the command line arguments.
Lets take a small example code that explains the points mentioned above.

#include<stdio.h>

int main(int argc, char *argv[])
{
  int count = argc;
  printf("\n The number of arguments passed is [%d] \n", count);

  int c = 0;
  while(c < count)
  {
    printf("\n The argument [%d] is : [%s]\n", c+1, argv[c]);
    c++;
  }
  return 0;
}
The above C code prints the number of command line arguments passed to it, and also prints the value of each argument.

When the program is executed, it displays the following output:

$ ./main abc 1 3

The number of arguments passed is [4]

The argument [1] is : [./main]

The argument [2] is : [abc]

The argument [3] is : [1]

The argument [4] is : [3]
We passed 3 arguments to the program ‘main’, but the log notifies that it received 4 arguments. This is because the name of the program (that we use to execute it) is also treated as a command line argument.

Also, since the above program was run on the terminal, the return value from the main() function is also sent to it. You can use bash shell special parameter $? as shown below to check the return value (0 indicates success).

$ echo $?
0
Coming back to the main function, when a C program is executed by the kernel, an ‘exec’ function is used to trigger the program.
Then in the next step, a typical startup routine is called just before the main() function of the program.
Similarly when a program ends execution then also a typical end routine is called.
If we look at any executable file, we will find that it specifies the start routine and ens routine addresses as the first routine and the last routine to be called.
The startup routine takes the command line arguments, environment etc from the kernel and passes these on to the main() function.
This whole setup comprising of the startup and end routine is done by the linker in the linking stage of the compilation process.
Environment List
Type the command ‘env’ on your Linux prompt and you will get a list of name=value pairs. This represents your shell environment. Similarly, a process also has its environment. There are two ways in which we can access a process environment:

Through the global variable ‘extern char **extern‘
Through the third argument to the main() function ‘char *envp[]’
Regarding envp[] (the 3rd argument), you might ask from where a third argument to the main() function came from as we discussed earlier that main() function has only two arguments.

Well, historically a third argument (the environment array) to the main() function was present. But ISO C specifies that the main() function to be written with only two arguments. Hence we do not use this third argument when we specify main function. But, we could access this envp[] inside the program.

Anyways, coming back to the environment list, the following code snippet specifies how to access the environment from within a process :

#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

extern char **environ;

int main(int argc, char *argv[])
{
  int count = 0;

  printf("\n");
  while(environ[count] != NULL)
  {
    printf("[%s] :: ", environ[count]);
    count++;
  }

  char *val = getenv("USER");
  printf("\n\nCurrent value of environment variable USER is [%s]\n",val);

  if(setenv("USER","Arora",1))
  {
    printf("\n setenv() failed\n");
    return 1;
  }

  printf("\n Successfully Added a new value to existing environment variable USER\n");

  val = getenv("USER");
  printf("\nNew value of environment variable USER is [%s]\n",val);

  while(1)
  {
    sleep(2);
  }
  return 0;
}
In the above code, we have used the global variable ‘environ’ to access all the environment variables. Also we have used two functions :

getenv – Get value of a particular environment variable
setenv – Set a new value to an environment variable
The output of the above program comes out to be :

$ ./environ

[ORBIT_SOCKETDIR=/tmp/orbit-himanshu] :: [SSH_AGENT_PID=1627] :: [TERM=xterm] ::
[SHELL=/bin/bash] :: [WINDOWID=39846040] :: [GTK_MODULES=canberra-gtk-module] ::
[USER=himanshu] :: [SSH_AUTH_SOCK=/tmp/keyring-6kpqGc/ssh] ::
..
..

Current value of environment variable USER is [himanshu]

Successfully Added a new value to existing environment variable USER

New value of environment variable USER is [Arora]
The above output prints the whole environment list to the stdout. The above code snippet also used getenv and setenv to get the USER environment variable and changed its value.
