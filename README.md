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


