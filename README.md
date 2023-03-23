# Service Operations Engineer
![Screenshot_56](https://user-images.githubusercontent.com/106797604/227261643-6dbb8c3b-fa16-499c-a03f-60fe19d2a767.png)

# Task1 Використовуючи Bash Shell напишіть скрипт, який повинен виконувати наступні дії:
 ### 1.1 витягти усіх користувачів з файлу /etc/passwd з id користувача більшим ніж 1000
![Screenshot_61](https://user-images.githubusercontent.com/106797604/227277748-6f1e9e04-9d41-4654-a48c-d9cd8590ab7d.png)
![Screenshot_62](https://user-images.githubusercontent.com/106797604/227277956-5f9cde52-0510-49b3-aa98-4ce05b743c76.png)

### 1.2 порівнює сьогоднішню дату та дату їх останнього входу до системи


Сценарій greps для рядка «Failed passw», щоб отримати деталі невдалої спроби входу, і розміщує ці рядки в /tmp/failed.$$.log
Сценарій greps для рядка «Accepted password|Accepted publickey|keyboard-interactive» для отримання успішних спроб входу та розміщує ці рядки в /tmp/success.$$.log
$$ у назві тимчасового файлу буде автоматично замінено на PID сценарію
Наступним кроком буде виділення користувачів, які успішно ввійшли та які не ввійшли.
Команда awk витягує п’яте поле з кінця (ім’я користувача) і передає його для сортування та uniq для створення списку користувачів.
Далі унікальні IP-адреси витягуються за допомогою регулярного виразу та команди egrep.
Вкладені цикли for перебирають IP-адресу та користувачів, вилучаючи рядки з кожною комбінацією IP-адреси та користувача. Якщо кількість спроб для цієї комбінації IP/користувач > 0, час першого входження витягується за допомогою grep, head і cut. Якщо кількість спроб > 1, останній час витягується за допомогою хвоста замість голови.
Про цю спробу входу повідомляється за допомогою форматованої команди printf.
Нарешті, тимчасовий файл видаляється.

![Screenshot_63](https://user-images.githubusercontent.com/106797604/227281668-0b7d9ed3-0806-4e4e-b4e2-1018288e0e42.png)
![Screenshot_64](https://user-images.githubusercontent.com/106797604/227281662-2c652c94-7a5f-457f-b3a3-cc29ca90d638.png)

![Screenshot_65](https://user-images.githubusercontent.com/106797604/227282376-b43dc4ef-c2cc-4ea9-adff-052a273e3487.png)




 ### 1.3 якщо користувач не заходив у систему понад 30 днів, то видалити їх облікові записи

 
![Screenshot_67](https://user-images.githubusercontent.com/106797604/227290272-5dfbe971-1447-41cf-9a4a-d98329ce4b37.png)

![Screenshot_68](https://user-images.githubusercontent.com/106797604/227291587-b1edf15b-f58e-4f21-813e-3d490697276b.png)




# BUT IF YOU WANT DELETE USER EARLIER THAN 30 DAYS

![Screenshot_69](https://user-images.githubusercontent.com/106797604/227303149-b13fd63e-358b-4da8-8b1b-6d94e5728fa4.png)
![Screenshot_70](https://user-images.githubusercontent.com/106797604/227303143-3c1b58a9-5695-4cb7-8873-ee976a8ebbf6.png)

![Screenshot_71](https://user-images.githubusercontent.com/106797604/227304146-d803dbca-1ceb-4d0b-ac32-ce7d01e26783.png)
![Screenshot_72](https://user-images.githubusercontent.com/106797604/227304169-c3ddec61-bb6c-459c-80f5-06f13ccf2a45.png)
![Screenshot_73](https://user-images.githubusercontent.com/106797604/227304203-4d94c230-c84b-4e04-b5ab-455855469c18.png)
![Screenshot_74](https://user-images.githubusercontent.com/106797604/227304248-3cf3c5d1-484a-4a3d-a6dc-1b6812a4c0d0.png)
![Screenshot_75](https://user-images.githubusercontent.com/106797604/227304284-109672b5-dd44-4725-87c0-43b40137c4f9.png)



# 1.1
```


#!/bin/bash
echo "###################################################################################################"
echo "================================in files `hostname`_userlogins.txt================================="
echo "###################################################################################################"
(
cat /etc/passwd | awk -F: '$3>=1000{print $1,$4,$5}' | sort | \
while read username gid gecos
do
        printf "%-10s|%-10s|%-10s|%-10s|%-15s|%-15s|%s\n" "Status" "User" "UserID" "Last login" "First login" "Difference" "Time range"
        last_login=`last -1 ${username}|grep \^${username}|awk '{print $4,$5}'`
# Get line from /etc/passwd
        passwd=$(getent passwd "$gid")
# Get data from passwd
        nameuser=$(echo "$passwd" | cut -d: -f1)
        realname=$(echo "$passwd" | cut -d: -f5)
# Get info from last, strip last 2 lines since they're not useful for us. Use
# ISO format so that date can parse them
        lastlog=$(last --time-format iso "$nameuser" | head -n-2)
# Get first & last line; we only need the date
        login_last=$(echo "$lastlog" | head -n1 | tr -s ' ' | cut -d ' ' -f 4)
        first_login=$(echo "$lastlog" | tail -n1 | tr -s ' ' | cut -d ' ' -f 4)
# Parse dates with date, output time in seconds since 1-1-1970 ('epoch')
#        diff=$(( $(date --date "$login_last" +%s) - $(date --date "$first_login" +%s) ))
         diff=$(( $(date  --date="today" +"%s")-$(date --date "$login_last" +%s) ))
# Format the date
        diff_fmt=$(date --date @$diff +'%d days %H hours %M minutes %S seconds')
        printf "%-10s|%-10s|%-10s|%-10s|%-15s|%-15s|%-s\n" "Success" "$username" "$gid" "$login_last" "$first_login" "$diff_fmt" "$diff";
done
) > `hostname`_userlogins.txt


```


# 1.2

```
AUTHLOG=/var/log/auth.log

if [[ -n $1 ]];
then
  AUTHLOG=$1
  echo Using Log file : $AUTHLOG
fi

# Collect the failed login attempts
FAILED_LOG=/tmp/failed.$$.log
egrep "Failed pass" $AUTHLOG > $FAILED_LOG

# Collect the successful login attempts
SUCCESS_LOG=/tmp/success.$$.log
egrep "Accepted password|Accepted publickey|keyboard-interactive" $AUTHLOG > $SUCCESS_LOG

# extract the users who failed
failed_users=$(cat $FAILED_LOG | awk '{ print $(NF-5) }' | sort | uniq)

# extract the users who successfully logged in
success_users=$(cat $SUCCESS_LOG | awk '{ print $(NF-5) }' | sort | uniq)
# extract the IP Addresses of successful and failed login attempts
failed_ip_list="$(egrep -o "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" $FAILED_LOG | sort | uniq)"
success_ip_list="$(egrep -o "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" $SUCCESS_LOG | sort | uniq)"

# Print the heading
printf "%-10s|%-10s|%-10s|%-15s|%-15s|%s\n" "Status" "User" "Attempts" "IP address" "Host" "Time range"

# Loop through IPs and Users who failed.

for ip in $failed_ip_list;
do
  for user in $failed_users;
    do
    # Count failed login attempts by this user from this IP
    attempts=`grep $ip $FAILED_LOG | grep " $user " | wc -l`

    if [ $attempts -ne 0 ]
    then
      first_time=`grep $ip $FAILED_LOG | grep " $user " | head -1 | cut -c-16`
      time="$first_time"
      if [ $attempts -gt 1 ]
      then
        last_time=`grep $ip $FAILED_LOG | grep " $user " | tail -1 | cut -c-16`
        time="$first_time -> $last_time"
      fi
      HOST=$(host $ip 8.8.8.8 | tail -1 | awk '{ print $NF }' )
      printf "%-10s|%-10s|%-10s|%-15s|%-15s|%-s\n" "Failed" "$user" "$attempts" "$ip"  "$HOST" "$time";
    fi
  done
done

for ip in $success_ip_list;
do
  for user in $success_users;
    do
    # Count successful login attempts by this user from this IP
    attempts=`grep $ip $SUCCESS_LOG | grep " $user " | wc -l`

    if [ $attempts -ne 0 ]
    then
      first_time=`grep $ip $SUCCESS_LOG | grep " $user " | head -1 | cut -c-16`
      time="$first_time"
      if [ $attempts -gt 1 ]
      then
        last_time=`grep $ip $SUCCESS_LOG | grep " $user " | tail -1 | cut -c-16`
        time="$first_time -> $last_time"
      fi
      HOST=$(host $ip 8.8.8.8 | tail -1 | awk '{ print $NF }' )
      printf "%-10s|%-10s|%-10s|%-15s|%-15s|%-s\n" "Success" "$user" "$attempts" "$ip"  "$HOST" "$time";
    fi
  done
done

rm -f $FAILED_LOG
rm -f $SUCCESS_LOG
```


# 1.3


 ```
echo "###################################################################################################"
echo "==================Deletes Users Haven't Been Logged In  more than 30==============================="
echo "======================in files `hostname`_delete_users.txt=========================================="
echo "###################################################################################################"

(
# This script takes everyone with id>1000 from /etc/passwd and removes every user account in case if it hasn't been used for the last 30 days.

# Make sure that script is being executed with root priviligies.

if [[ "${UID}" -ne 0 ]]
then
echo "You should run this script as a root!"
exit 1
fi

# First of all we need to know id limit (min & max)

USER_MIN=$(grep "^UID_MIN" /etc/login.defs)

USER_MAX=$(grep "^UID_MAX" /etc/login.defs)

# Print all users accounts with id>=1000 and <=6000 (default).

awk -F':' -v "min=${USER_MIN##UID_MIN}" -v "max=${USER_MAX##UID_MAX}" ' { if ( $3 >= min && $3 <= max ) print $0}' /etc/passwd

# This function deletes users which hasn't log in in the last 30 days

# Make a color output message

for accounts in ` lastlog -b 30 | sed "1d" | awk ' { print $1 } '`

do

userdel $accounts 2>/dev/null

done

echo -e "\e[36mYou have successfully deleted all user's account which nobody logged in in the past 30 days.\e[0,"

exit 0
) > `hostname`_delete_users.txt

 
 ```
 
 
# 1.4 BUT IF YOU WANT DELETE USER EARLIER THAN 30 DAYS 

```
echo "======================BUT IF YOU WANT DELETE USER EARLIER THAN 30 DAYS============================="
echo "###################################################################################################"
echo "================================Define Functions get_answer========================================"
echo "###################################################################################################"
function get_answer {
unset ANSWER
ASK_COUNT=0
while [ -z "$ANSWER" ] #While no answer is given, keep asking.
do
 ASK_COUNT=$[ $ASK_COUNT + 1 ]
 case $ASK_COUNT in #If user gives no answer in time allotted
 2)
 echo
 echo "Please answer the question."
 echo
 ;;
 3)
 echo
 echo "One last try...please answer the question."
 echo
 ;;
 4)
 echo
 echo "Since you refuse to answer the question..."
 echo "exiting program."
 echo
 #
 exit
 ;;
 esac


 echo
 if [ -n "$LINE2" ]
 then #Print 2 lines
 echo $LINE1
 echo -e $LINE2" \c"
 else #Print 1 line
 echo -e $LINE1" \c"
 fi
#Allow 60 seconds to answer before time-out
 read -t 60 ANSWER
done
# Do a little variable clean-up
unset LINE1
unset LINE2

} 

echo "###################################################################################################"
echo "================================Define Functions process_answer===================================="
echo "###################################################################################################"

function process_answer {
case $ANSWER in
y|Y|YES|yes|Yes|yEs|yeS|YEs|yES )
# If user answers "yes", do nothing.
;;
*)
# If user answers anything but "yes", exit script
 echo
 echo $EXIT_LINE1
 echo $EXIT_LINE2
 echo
 exit
;;
esac

# Do a little variable clean-up

unset EXIT_LINE1
unset EXIT_LINE2

}

echo "###################################################################################################"
echo "=====================                Step #1              ========================================="
echo "###################################################################################################"
echo "=====================  Determine User Account name to Delete    ==================================="
echo "Step #1 - Determine User Account name to Delete "
LINE1="Please enter the username of the user "
LINE2="account you wish to delete from system:"
get_answer
USER_ACCOUNT=$ANSWER
# Double check with script user that this is the correct User Account
LINE1="Is $USER_ACCOUNT the user account "
LINE2="you wish to delete from the system? [y/n]"
get_answer

# Call process_answer funtion:
# if user answers anything but "yes", exit script
EXIT_LINE1="Because the account, $USER_ACCOUNT, is not "
EXIT_LINE2="the one you wish to delete, we are leaving the script..."
process_answer

# Check that USER_ACCOUNT is really an account on the system
USER_ACCOUNT_RECORD=$(cat /etc/passwd | grep -w $USER_ACCOUNT)
if [ $? -eq 1 ] # If the account is not found, exit script
then
 echo
 echo "Account, $USER_ACCOUNT, not found. "
 echo "Leaving the script..."
 echo
 exit
fi
#
echo
echo "I found this record:"
echo $USER_ACCOUNT_RECORD

LINE1="Is this the correct User Account? [y/n]"
get_answer
# Call process_answer function:
# if user answers anything but "yes", exit script
EXIT_LINE1="Because the account, $USER_ACCOUNT, is not "
EXIT_LINE2="the one you wish to delete, we are leaving the script..."
process_answer



echo "###################################################################################################"
echo "=======================             Step #2                ========================================"
echo "###################################################################################################"
echo "=============Search for any running processes that belong to the User Account======================"
echo "Step #2 - Find process on system belonging to user account"
ps -u $USER_ACCOUNT >/dev/null #Are user processes running?
case $? in
1) # No processes running for this User Account
 echo "There are no processes for this account currently running."
 echo
;;
0) # Processes running for this User Account.
 # Ask Script User if wants us to kill the processes.
 #
 echo "$USER_ACCOUNT has the following processes running: "
 echo
 ps -u $USER_ACCOUNT
 LINE1="Would you like me to kill the process(es)? [y/n]"
 get_answer
 case $ANSWER in
 y|Y|YES|yes|Yes|yEs|yeS|YEs|yES ) # If user answers "yes",
 # kill User Account processes.
 echo
 echo "Killing off process(es)..."
 # List user processes running code in variable, COMMAND_1
 COMMAND_1="ps -u $USER_ACCOUNT --no-heading"
 # Create command to kill proccess in variable, COMMAND_3
 COMMAND_3="xargs -d \\n /usr/bin/sudo /bin/kill -9"
 # Kill processes via piping commands together
 $COMMAND_1 | gawk '{print $1}' | $COMMAND_3
 echo "Process(es) killed."
 ;;
 *) # If user answers anything but "yes", do not kill.
 echo "Will not kill the process(es)"
 ;;
 esac
;;
esac

echo "###################################################################################################"
echo "============================             Step #3         =========================================="
echo "###################################################################################################"
echo "=====================Create a report of all files owned by User Account============================"

echo "Step #3 - Find files on system belonging to user account"
echo "Creating a report of all files owned by $USER_ACCOUNT."
echo "It is recommended that you backup/archive these files,"
echo "and then do one of two things:"
echo " 1) Delete the files"
echo " 2) Change the files' ownership to a current user account."
echo "Please wait. This may take a while..."
REPORT_DATE=$(date +%y%m%d)
REPORT_FILE=$USER_ACCOUNT"_Files_"$REPORT_DATE".rpt"
find / -user $USER_ACCOUNT > $REPORT_FILE 2>/dev/null
echo "Report is complete."
echo "Name of report: $REPORT_FILE"
echo "Location of report: $(pwd)"


echo "###################################################################################################"
echo "================================          Step #4    =============================================="
echo "###################################################################################################"
echo "====================================Remove User Account============================================"
echo "Step #4 - Remove user account"
LINE1="Remove $USER_ACCOUNT's account from system? [y/n]"
get_answer
# Call process_answer function:
# if user answers anything but "yes", exit script
EXIT_LINE1="Since you do not wish to remove the user account,"
EXIT_LINE2="$USER_ACCOUNT at this time, exiting the script..."
process_answer
userdel $USER_ACCOUNT #delete user account
echo "User account, $USER_ACCOUNT, has been removed"


echo "###################################################################################################"
echo "================================       FINISHED      =============================================="
echo "###################################################################################################"
echo "============================Name of report: $REPORT_FILE==========================================="

exit



```