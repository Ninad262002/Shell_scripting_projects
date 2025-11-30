# The problem statement

You are presented with a problem.

Your company has hired many new developers, and you need to automate the creation of user accounts and passwords for each of them.
As a SysOps engineer, write a Bash script that reads a text file containing the employees’ usernames and group names, where each line is formatted as username;groups.
The text file can also specify multiple groups for the user, formatted as username; group1, group2.
In addition to the multiple groups specified by the text file, each user must have a personal group named after their username.
The script should create users and groups as specified, set up home directories with appropriate permissions and ownership, and generate random user passwords.
Additionally, store the generated passwords securely in /var/secure/user_passwords.csv, and log all actions to /var/log/user_management.log.



# Setting up the project
Before diving head first into creating the script itself, let's define what it needs to automate. The script must:

Read User and Group Information: The script will rely on a text file containing user and group information
Create User and Group: For each user specified in the file, the script will create a user account and a personal group
Assign Additional Groups: If additional groups are listed for a user (e.g., user;group1,group2), the script will also assign the user to those groups
Create Home Directories: Each user will have a dedicated home directory created.
Generate Random Passwords: Secure random passwords will be generated for each user and stored in /var/secure/user_passwords.csv
Log Actions: The script will log all its activities to /var/log/user_management.log


1. To check weather user is root or not
   if [ $EUID -ne 0 ];then
     echo"Please run as root"
     exit 1
   fi

hear $EUID is effective user , which is used to check weather user is root or not , 0 is assign to root.
exit 1 - Exit with status 1, indicating an error
exit 0 - Exit with status 0 , indicating success

2. To check weather you have pass the file or not
   if [ -z "$1" ]; then
      echo"Please pass the file"
      exit 1
   fi

   -z This is a test operator that checks if the length of the string "$1" is zero.

3. Define the file paths for the logfile, and the password file
INPUT_FILE="$1"
LOG_FILE="/var/log/user_management.log"
PASSWORD_FILE="/var/secure/user_passwords.csv"

This are the location where we will store out user password and logs 

4. Generate logfiles and password files and grant the user the permissions to edit the password file
touch $LOG_FILE
mkdir -p /var/secure
chmod 700 /var/secure
touch $PASSWORD_FILE
chmod 600 $PASSWORD_FILE

5. Generate logs and passwords 
log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}

generate_password() {
    openssl rand -base64 12
}

6. main logic
   while IFS=";" read -r username groups || [ -n "$username" ]; do
   
   IFS is a internal field seperator , it seperates line , and same them in two variable "username" and "groups"
   [-n $username] it checks weather their is any character present in $username
   Or operator ( || )
     If read fails, check if username is NOT empty.
     If read works → loop continues normally
     If read fails → OR part is checked
     If $username has data → loop still runs
     If $username is empty → loop stops

    username=$(echo "$username" | xargs)
    groups=$(echo "$groups" | xargs)
   it removes extra space from beginning and end

7. Check if the personal group exists, create one if it doesn't
if ! getent group "$username" &>/dev/null; then
    echo "Group $username does not exist, adding it now"
    groupadd "$username"
    log_message "Created personal group $username"

fi

:- getent - Get entry from system databases.
  It checks if a user, group, host, etc. exists in the system database.
:- her &> /dev/null to hide all output
:- ! means NOT
:- which means if group not exit then run below commmands
:- if group exist then skip
:- log_message save message in log file

8.  Check if the user exists
    if id -u "$username" &>/dev/null; then
        echo "User $username exists"
        log_message "User $username already exists"
    else
        # Create a new user with the created group if the user does not exist
        useradd -m -g $username -s /bin/bash "$username"
        log_message "Created a new user $username"
    fi

9. Check if the groups were specified
    if [ -n "$groups" ]; then
        # Read through the groups saved in the groups variable created earlier and split each group by ','
        IFS=',' read -r -a group_array <<< "$groups"
   
   It splits the string $groups using comma ( , )
   And stores each piece into an array called group_array
   - a  Put the split values into an array
   - <<< It feeds the value of $groups into read
10.  Check if the groups were specified
    if [ -n "$groups" ]; then
        # Read through the groups saved in the groups variable created earlier and split each group by ','
        IFS=',' read -r -a group_array <<< "$groups"

        # Loop through the groups 
        for group in "${group_array[@]}"; do
            group=$(echo "$group" | xargs)
            if ! getent group "$group" &>/dev/null; then
                 If the group does not exist, create a new group
                groupadd "$group"
                log_message "Created group $group."
            fi
            Add the user to each group
            usermod -aG "$group" "$username"
            log_message "Added user $username to group $group."
        done
    fi
11. Create and set a user password
    password=$(generate_password)
    echo "$username:$password" | chpasswd
    # Save user and password to a file
    echo "$username,$password" >> $PASSWORD_FILE
   

   
   
   

