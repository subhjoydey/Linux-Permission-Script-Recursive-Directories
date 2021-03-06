# Linux Permission Recreated on Recursive Directories

## Overview

This project recreates the Ubuntu Permission Algorithm using a Bash Shell Script. It accepts three arguments or accepts user inputs of UserID/Username, GroupID/Groupname and Complete path to search recreate the Linux Permissions on the current and all the recursive diretories.

## Pre-Requisites

### Taking Arguments

#### CLI Arguments

You can pass the Command Line Arguments wihle running the bash script from the command line as follows:-
* ./PermissionRecursive argument1 argument2 argument3

You can then call these arguments by $1 $2 and $3 from the bash scripts. These are all stored as array elements and can be called all together as $@. This allows you to check for the number of arguments passed while calling the bash script and we use this to accept incomplete arguments from CLI as user inputs instead.

#### User Input

You can also take user input from the Command Line as:-
* read -p 'Username/UID: ' arguments[0]

This will print Username/UID: on the CLI and ask for an input which will be stored in arguments array.

### Array Uses

Some of the main array knowledge in bash scripts required in this script is how to print the entire array at once and to print the number of array elements.

* ${#arguments[@]} - This prints the number of elements in the array. This allows you to check for array lengths.
* printf '%s\n' "${my_array[@]}" - This prints the whole aray in one line.

### Understanding File Columns in /etc

#### /etc/passwd 

* (Column 1) Username: It is used when user logs in. It should be between 1 and 32 characters in length.
* (Column 2) Password: An x character indicates that encrypted password is stored in /etc/shadow file. Please note that you                          need to use the passwd command to computes the hash of a password typed at the CLI or to store/update                        the hash of the password in /etc/shadow file.
* (Column 3) User ID (UID): Each user must be assigned a user ID (UID). UID 0 (zero) is reserved for root and UIDs 1-99 are                               reserved for other predefined accounts. Further UID 100-999 are reserved by system for                                       administrative and system accounts/groups.
* (Column 4) Group ID (GID): The primary group ID (stored in /etc/group file)
* (Column 5) User ID Info: The comment field. It allow you to add extra information about the users such as user’s full name,                            phone number etc. This field use by finger command.
* (Column 6) Home directory: The absolute path to the directory the user will be in when they log in. If this directory does                              not exists then users directory becomes /
* (Column 7) Command/shell: The absolute path of a command or shell (/bin/bash). Typically, this is a shell. Please note that                             it does not have to be a shell.

#### /etc/group

* (Column 1) group_name: It is the name of group. If you run ls -l command, you will see this name printed in the group                                field.
* (Column 2) Password: Generally password is not used, hence it is empty/blank. It can store encrypted password. This is                            useful to implement privileged groups.
* (Column 3) Group ID (GID): Each user must be assigned a group ID. You can see this number in your /etc/passwd file.
* (Column 4) Group List: It is a list of user names of users who are members of the group. The user names, must be separated by commas.

### Trap Operations

Trap operations mean that whenever it calls a cleanup process is started while trapping the error messages.

function CLEANUP {
	rm credentials.txt
	rm group
	rm password
	rm executable_files1.txt
	rm Documents.txt
	rm result.txt
}
trap CLEANUP exit 1

Thus whenever we call exit 1 the cleanup process is started.

## Scripts

### * Script ./checker.bash

#### Overview

This  shell script accepts username/uid, groupname/gid and an absolute path as argument and then checks for the validity of the arguments. First we combine /etc/group and /etc/passwd into a single file sorted individually as "password" "group" into "credentials.txt". This file was then used to search for the user and group. For success of user,group we print appropriate string in file "result.txt".

#### User/Group/Path Check

* UserID/Username - We check the Username/UserID in the files /etc/passwd and /etc/group to verify if it is valid.
* GroupIF/Groupname - We check the Groupname/GroupID in the files /etc/passwd and /etc/group to verify if it is valid. This                         is done in two steps. (1) we check if the groupname/groupID exists (2) we check if the username is a                         part of the group. 
* Absolute Path - We check if the path exists. This is done by [test -d "$path"] . Where $path is a variable which holds the                   absolute path that is passed via arguments.

We then create a file result.txt which contains check status of username and userID as SUCCESS11 and SUCCESS12 respectively. If it is SUCCESS12 which means it is userID we then store the username beside the message as well.

We repeat the same for groupame/GroupID as SUCCESS21 and SUCCESS22 and print groupname beside SUCCESS22 if groupID.

### * Script ./directoryFinder.bash

#### Overview

This  shell script recursively searches for all files and directories and their sub-directories and then prints the ls -al with the complete path of the directories and files. Then all this is stored in "Documents.txt".


### * Script ./PermissionDescriptor.bash

#### Overview

This  shell script accepts username/uid and groupname/gid and we provide appropriate permission search with preference order of user->group->other with specific execute permissions. The file is then stored as "executable_files.txt". 

#### Assinging Permissions

This script then recognises the user executable permission first then it moves on to group and further into others:- 

* It finds a match as User Permission then it assigns as U which is followed by Y/N which indicates if there is execute         permission or not. 

* This script then recognises the group executable permission if the user is a part of that group then it assigns G which is   followed by Y/N that indicates if there is exeute permission or not.

* It finds a match as Others Permission then it assigns as O which is followed by Y/N which indicates if there is execute       permission or not. 

### * Script ./PermissionRecursive.bash

#### Overview

This  shell script accepts username/uid, groupname/gid and an absolute path as variable or as an interactive tool and uses these credentails and path verify executepermissions as user, group or other. This script uses several sub-scripts. At first  "checker.bash" which creates temporary file of "credentials.txt" and "result.txt". Then it calls sub-script directoryFinder.bash which creates temporary file "Documents.txt". Finally it cal ls sub-script PermissionDescriptor.bash which creates temporary file "executable_files.txt" contaiting the final file with permissions along with subdirectories. 

#### Main File 

This is the main file which calls the other shell scripts and then decides the final output in a particular order of "ls -al". This also provides error messages for particular errors and performs trap operations. Whicle exiting removes temporary files which were created during the running of the script.
