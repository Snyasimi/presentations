# Shell Scripting

## Introdction 

In unix, the shell is a command line interface that allows a user to interact with the operating system.
The shell is a macro-processor, int that it expands text and symobls to create *larger expressions*.

The shell can be used interactively or non-interactively. In interactive mode, the shell accepts input from the keyboard. 
When executing in non-interactive mode, the shell executes commands read from a file or a string.

There are different types of shells in unix, these include 
- Korn shell
- bash shell
- c-shell

We'll focus on the bash shell.

The bourne again shell (BASH) is the commandline interpretor for the GNU OS. It implements features of  k-shell and c-shell.

## Shell Scripting

A shell script is a file containing shell commands.
To make a shell script executable, you need to use the `chmod +x {name of script}` command to turn on the executable bit.

A script should begin with the `#!` ~ shebang! ~ . This is the common way to tell which interpretor should be used to interprate/execute the program. You can have a file having 
`#! /usr/bin/python3`, In this case, the program willl use the python interpretor to execute the script.

### Why we need shell scripts

The main reason we need shell scripts, is to minimize the amount of times you need to recall and run a comand, especially if its something done occassionally. With shell scripts, you can modify these commands to do something complex and schedule them to run at specified times using cronjobs.

To further understand why we need them, we will craft a simple command to help us scrape our logfiles,especially the auth files. Say we needed to see which commands needed authentication as sudo and which commands ran. To do this we can write a simple command as follows
    
```bash
    $ cat /var/log/auth 
        
    Apr 13 16:05:01 wild-wolf CRON[78403]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
    Apr 13 16:05:01 wild-wolf CRON[78403]: pam_unix(cron:session): session closed for user root
    Apr 13 16:09:01 wild-wolf CRON[79427]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
    Apr 13 16:09:01 wild-wolf CRON[79427]: pam_unix(cron:session): session closed for user root
    Apr 13 16:15:01 wild-wolf CRON[79776]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
    Apr 13 16:15:01 wild-wolf CRON[79776]: pam_unix(cron:session): session closed for user root
```
    

As we can see,  we get alot of info that's mostly not relevant for us, so we can add in an additional filter to filer for keywords

    ` $ cat /var/log/auth.log | grep -e 'auth\|authentication\|root\|Failed password' `

With this, our command now returns lines containing the strings `authentication, root, Failed password auth and auth `.

Since we are mostly interested in commands that ran a s sudo and the commandas that ran, now we modify our command to 

    ```bash
        $ cat /var/log/auth.log.1 /var/log/auth.log | grep -e 'auth\|authentication\|root\|Failed password' | grep "sudo" | grep -i "COMMAND"
    ```
Our command takes as input the 2 log files and concats them then applies the filters.

As we can see, it's already alot. The command is long. Imagine having to type that every single day.

That's where shell scripts come in. To solve for all this.

But for us to understand shell scripts, we must learn its structure and basic building blocks. Thus we will cover

- [Output](#output)
- [Input](#input) 
- [Variables](#variables)
- [shell arguments](#shell-arguments)
- Control statements
    - [loops](#loops)
    - if statements
    - if-else statements
    - if-elif-else statements
    - switch statements

## Building Blocks

First off, we will start with printing to standard out put. To print, we will use the `echo` command.

### Running shell scripts

To run a shell script, you need to make it executable then run it. To run a shell script called `myscript` in the current directory, we run the following 

```bash
     $ chmod +x myscript

     $ ./myscript
```
        


### Output 

To print something to the console, create a newfile and name it however you want and put the following content inside

```bash

    echo "Hello world"
``` 
save the file and [run the script](#running-shell-scripts)

On running you should see 

    ```bash
        $ chmod +x myscript
        $ ./myscript
          Hello world
        $

    ```
    

### Input 

To get input from a user, you need to use the `read` command after a prompt. Below is an example of how you can obtain a user name and print it 

```bash
#! /bin/bash

echo "Hello guys!"
sleep 2

echo "what's your name?"
read username

echo "hello $username"

exit 0	

```

### Variables 

In bash, you can declare and initialize variables. The  most common way is to declare and initialize then use it later.
To declare variables we just put a `$` before the variable name. eg `$MY_VAR`. Its common convention to declare variable using upper case
Below is an example of how variables work

```bash
# shellscr.sh
#! /usr/bin/env bash
   
   NAME
   
   echo "whats your name?"
   read NAME #reads from terminal and set name to input
   
   echo "What's your pet's name?"
   read PET_NAME #creates and initializes PET_NAME to users input
   
   echo "$NAME, where's your pet $PET_NAME ?"
   
   exit 0

```
### Shell arguments

For your programs to be able to read input from command line arguments, you need to read the shell arguments by using a special character combination that expands to the actual argements given when the script was run.

we can go over these special characters

```bash
$# #this contains the number of arguments passed to the script [./script *arg1* *arg2*]
$@ #This holds the actual arguments passed
$0 #This translates to the name of the script
$1 #this translates to the first argument passed to the script

```

Below is an example of how we can make use of shell arguments

```bash

    #! /bin/bash
  
   
   if [[ $# < 1 ]] ; then
           echo 'Please tell me what to do/say\n USAGE: COMMAND [OPTIONS]'
           exit 1;
   fi
   
   if [[ $1 == "Strongest avenger" ]] ; then
           echo "Tonny Stark"
   else
          echo "Thanos Cooked Multiple MCU Characters :( "
   fi
  
  exit 0;


  #Running the script 

  #$ ./tellme.sh strongest avenger
  #Tonny stark
  #$

```

### Control structures

To effeciently use our scripts, we need to define some order of operation and add repetitions and some decision making.
This is made availible through 

- ##### loops
    - [for loops](#Forloops)
    - [while loops](#while-loops)
    - [until loops](#until-loops)

- ##### conditional statements
    - if
    - if-else
    - if-elif-else
    - case


#### Loops

Loops enable us to repeat a block of statements until a particular condition is satisfied

##### For loops

In bash, the for loop can be written in two forms  
` for name [ [in words ...] ; ] do commands; done `

The other for is similar to the c style for `for loops`

`for (( expr1 ; expr2 ; expr3 )) [;] do commands ; done`

Example program 

```bash

   #! usr/bin/env bash 
   
   for name in bob alice spike
   do
           echo "Enter a number: "
           read $name
   done

#------------------------------------
   #! usr/bin/env bash 
   
   for i in 1 2 3
   do
           echo "Enter a number: "
           read number_$i
   done

#-----------------------------------
# second form 

 #! /usr/bin/env bash
   
   for (( i=1 ; i < 10 ; i++ ))
   do
           echo "$i"
   done
                                                                                                                                                     
          
```

##### While loops

While loops run until the the a condition becomes false, or has an exit status not 0

syntax

`while test-commands; do consequent-commands; done`

example usage 

```bash
 #! /usr/bin/env bash
   
   
   while [[ $i -lte 15]] do
           echo "$i"
   done 

```

##### Until loops

In bash we have another loop known as the until loop, this loops runs *until* a test-command returns a non-zero value.
It runs as long as the test command has an exit status of 0

syntax

`while test-commands; do consequent-commands; done`

Example usage: The loop will terminate once it recieves skynet as input

```bash

  #! usr/bin/env bash
   
   echo "We are doing intil looop, it'll terminate once you type skynet"
   
   str=""
   
   until [[ "$str" == "skynet" ]] ; do
           echo "Enter easter egg"
           read str
   done
  
  echo -e "You saved the world, skynet is taken down!\n*\t Terminator!!!\t*\n"



```