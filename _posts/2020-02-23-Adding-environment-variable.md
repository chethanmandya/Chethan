---
title: Adding environment variableEnvironment variables In Unix, Linux
tags: Linux, Unix
article_header:
  type: cover
  image:
---



### @Adding environment variables In Unix, Linux Environment
In Linux Or Unix environment , All your environment variables are stored in the .bash_profile file which is stored in the root directory. Every time you want to add new environment variable, you will make changes to this file. 

Say, you want to add an environment variable for username, you can do it with following commands. First edit the .bash_profile in the root directory

vi ~/.bash_profileand 

add the following line to this file 

export USERNAME="Chethan"

Where USERNAME is the key and Chethan is the value of this environment variable. 

Now bash usually requires you to restart the terminal to reflect the changes in bash_profile file. However, this can be eliminated by just typing following command. This command will update the system with new environment variable.

~/.bash_profile 

this is done, 

you can view the list of all environment variables in the system by typing **printenv** on the terminal.

Accessing environment variablesOnce you have this variable, you can access them in any shell script with following syntax, ENV["USERNAME"]

This will produce the value of Chethan in the script.

### @Removing environment variable 

Once it is set they remain under system unless unset explicitly. As you will see if you remove the variables from .bash_profile, refresh the system and then run **printenv**, it will still show the removed environment variable. The solution is to remove them from bash_profile and unset them from command line.

First step is simple. 

Go to bash_profile file remove the line which sets the new variable and run . ~/.bash_profile on the command line.

Once this is done, run the following command in the terminal,

unset USERNAME

This will get completely rid of eliminated variable. Now, if you type printenv in the terminal you will see that variable is no longer in the list.

<!--more-->

