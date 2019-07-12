## Bash Scripting

Scripting is a real power tool for sys admins.

Take care of tasks with scripts. Tasks could be:
* Checking the status of user account
* Running or testing daily backup
* System health checks
* Audit reports like:
  * Number of servers in your state
  * What versions of Code Systems are running
  * Which developer account have logged onto production machines in last month

### Elements of a script
* Shebang: Shebang (Hashbang) is path to the interpreter like: `#!/bin/bash`
  * Note that `/bin` has a symbolic link to `/usr/bin`
    <pre>
    [vagrant@web ~]$ ls -l /bin
    lrwxrwxrwx. 1 root root 7 Oct  6  2015 <b>/bin -> usr/bin</b></pre>
  * If we don't add it, it will run in the shell that the system is running as.
* Comments `#`:
  * Author
  * Date
  * Contact details
  * Script usage
  * Change history (who, when. why...)
* Main code
* Exit code
  * Retune Values:
    * `$?`: to see the success of failure of a program
      * `0`: Success
      * Not `0`: Failure

<pre>
[vagrant@web ~]$ <b>cat hello.sh</b>
#!/bin/bash
echo "Hello $USER"
[vagrant@web ~]$ <b>hello.sh</b>
Hello vagrant
</pre>



### read
Example:
<pre>
[vagrant@web ~]$ <b>cat hello.sh</b>
#!/bin/bash
read -p "Enter your name: " name
echo "Hello $name"
exit 0
[vagrant@web ~]$ <b>hello.sh</b>
Enter your name: <b>Foo</b>
Hello Foo
</pre>

Example: Look at `read -s -n` option
<pre>
[vagrant@web ~]$ <b>cat multiply.sh</b>
#!/bin/bash

read -p "Enter the first number: " a
read -p "Enter the second number: " b
read -s -n1 -p "Press any key when you are ready to see the answer..."
let answer=$a*$b
echo
echo $answer
[vagrant@web ~]$ <b>multiply.sh</b>
Enter the first number: 45
Enter the second number: 5457
Press any key when you are ready to see the answer...<b>d</b>
245565
</pre>

<pre>
[vagrant@web ~]$ <b>cat login.sh</b>
read -p "Username: " username
read <b>-s</b> -p "Password: "
echo -e "\nUsername is: $username"
echo "Password is: <b>$REPLY</b>"
[vagrant@web ~]$ <b>login.sh</b>
Username: <b>vagrant</b>
Password:
Username is: vagrant
Password is: <b>123</b></pre>

### Positional Parameters
* $0 is the script name
* $1, ... $9, ${10}, ${11}, ...
* $# Total number of arguments we passed to the script
  * `if (( $# < 1 ))`: Condition test for less than 1 parameter
* $@ All arguments as a list
* $* All arguments as a single value

Example:
<pre>
[vagrant@web ~]$ <b>cat hello.sh</b>
#!/bin/bash
echo "Foo $1"
exit 0
[vagrant@web ~]$ <b>hello.sh Foo</b>
Hello Foo
</pre>

<pre>
[vagrant@web ~]$ <b>cat arguments.sh</b>
#!/bin/bash
echo -e "\n$# arguments were passwed to the script"
echo "The arguments are $@"
[vagrant@web ~]$ <b>arguments.sh Foo is not Bar</b>
4 arguments were passwed to the script
The arguments are Foo is not Bar
</pre>

### if
Example:
<pre>
[vagrant@web ~]$ <b>cat hello.sh</b>
#!/bin/bash
if (( $# < 1 ))
  then

    echo "Usage: $0 <name>"
    exit 1
fi
echo "Hello $1"
exit 0
[vagrant@web ~]$ <b>hello.sh</b>
Usage: /home/vagrant/hello.sh <name>
[vagrant@web ~]$ <b>echo $?</b>
1
[vagrant@web ~]$ <b>hello.sh Foo</b>
Hello Foo
[vagrant@web ~]$ <b>echo $?</b>
0
</pre>

* `;` means a new line if we want to write some of our code in 1 line. See example below:

<pre>
[vagrant@web ~]$ <b>cat hello.sh</b>
#!/bin/bash
if (( $# < 1 )); then
    echo "Usage: $(basename $0) <name>"
    exit 1
  elif (( $# > 1 )); then
    echo "Too many arguments!"
    exit 2
  else
    echo "Hello $1"
fi
exit 0
[vagrant@web ~]$ <b>hello.sh Foo Bar</b>
Too many arguments!
[vagrant@web ~]$ <b>echo $?</b>
2
</pre>

#### Primary expressions
* `-e FILE` File exists
* `-b FILE` Block device files
* `-c FILE` Char device files
* `-f FILE` Regular files
* `-d FILE` Directories
* `-r FILE` Read access
* `-w FILE` Write access
* `-x FILE` or `-s FILE` Execute permission
* `STR1 == STR2` True if the strings are equal
* `STR1 != STR2` True if the strings are not equal
* `STR1 < STR2` True if "STR" sorts before "STRI" lexicographically in the current locale
* `ARG1 OP ARG2`  "ARG1" and "ARG2" are integers. "OP" is one of -eq, -ne, -lt, -le, -gt or -ge. These arithmetic binary operators return true if "ARG1" is equal to, not equal to, less than, less than or equal to, greater than, or greater than or equal to "ARG2", respectively.


**Lab**
1. To check if the file exists or not
<pre>
[vagrant@web ~]$ <b>cat filetype.sh</b>
#!/bin/bash
if (( $# == 0 )); then
  echo -e "\n Error! No filename specified!"
  echo -e "\nUsage: $(basename $0) <filename>"
  exit 1
fi
filename="$1"
if [ -e "$filename" ];then #`-e` checks the existance a file
  echo -e "\n$filename exists"
else
  echo -e "\n$filename does not exist in the search PATH.\n"
fi
[vagrant@web ~]$ <b>filetype.sh /etc/fstab</b>

/etc/fstab exists
[vagrant@web ~]$ <b>filetype.sh hosts</b>

hosts does not exist in the search PATH.
</pre>
2. If exists, what type it is?
<pre>
[vagrant@web ~]$ <b>cat filetype.sh</b>
#!/bin/bash
if (( $# == 0 )); then
  echo -e "\n Error! No filename specified!"
  echo -e "\nUsage: $(basename $0) <filename>"
  exit 1
fi
filename="$1"
if [ -e "$filename" ];then #`-e` checks the existance a file
  echo -e "\n$filename exists"
  if [ -f "$filename" ]; then
      echo -e "\nFile $filename is a regular file"
  elif [ -b "$filename" ]; then
      echo -e "\nFile $filename is a block file"
  elif [ -c "$filename" ]; then
      echo -e "\nFile $filename is a character device file"
  elif [ -d "$filename" ]; then
      echo -e "\nFile $filename is a directory"
  else
      echo -e "\nSorry! I don't know what type of file $filename is."
  fi
else
  echo -e "\n$filename does not exist in the search PATH.\n"
fi
[vagrant@web ~]$ <b>filetype.sh /dev/pts/0</b>

/dev/pts/0 exists

File /dev/pts/0 is a character device file
[vagrant@web ~]$ <b>filetype.sh /dev/</b>

/dev/ exists

File /dev/ is a directory
[vagrant@web ~]$ <b>filetype.sh /dev/sda1</b>

/dev/sda1 exists

File /dev/sda1 is a block file
[vagrant@web ~]$ <b>filetype.sh $(which ping)</b>

/usr/bin/ping exists

File /usr/bin/ping is a regular file
</pre>

### case

<pre>
case $var in
  "value")
    statement
    statement
    ..
    ;;
  "value")
    statement
    statement
    ..
    ;;
  *)
    statement
    statement
    ..
    ;;
esac
</pre>

Example:
<pre>
[vagrant@web ~]$ <b>cat show.sh</b>
#!/bin/bash
case $1 in
  "dir")
    find /etc/ -maxdepth 1 -type d
    ;;
  "link")
    find /etc/ -maxdepth 1 -type l
    ;;
  *)
    echo "Usage: $0 dir | link"
    ;;
esac
[vagrant@web ~]$ <b>show.sh</b>
Usage: /home/vagrant/show.sh dir | link
[vagrant@web ~]$ <b>show.sh link</b>
/etc/mtab
/etc/redhat-release
/etc/system-release
/etc/rc.local
/etc/init.d
/etc/rc0.d
/etc/rc1.d
/etc/rc2.d
/etc/rc3.d
/etc/rc4.d
/etc/rc5.d
/etc/rc6.d
/etc/favicon.png
/etc/grub2.cfg
</pre>

**Lab**
<pre>
[vagrant@web ~]$ <b>cat case.sh</b>
#!/bin/bash
if (( $# == 0 )); then
  echo -e "\n Error! No filename specified!"
  echo -e "\nUsage: $(basename $0) <filename>"
  exit 1
fi

check=$(file $1 | cut -d " " -f2)

case $check in
  "ASCII")
    echo -e "\nFile $1 is a plain text file"
  ;;
  "Bourne-Again")
    echo -e "\nFile $1 is a script file"
  ;;
  "ELF")
    echo -e "\nFile $1 is an executable"
  ;;
  "block")
    echo -e "\nFile $1 is a Block Device"
  ;;
  *)
    echo -e "\nGuess I am having a bad day, I don't know $1 file type!"
  ;;
esac
[vagrant@web ~]$ <b>case.sh case.sh</b>

File case.sh is a script file
[vagrant@web ~]$ <b>case.sh /dev/sda</b>

File /dev/sda is a Block Device
[vagrant@web ~]$ <b>case.sh $(which ifconfig)</b>

File /usr/sbin/ifconfig is an executable
[vagrant@web ~]$ <b>case.sh /etc/fstab</b>

File /etc/fstab is a plain text file
</pre>

### for

<pre>
for var in list
do
  statement
  statement
done
</pre>

<pre>
[vagrant@web ~]$ <b>cat for.sh</b>
#!/bin/bash
for i in $*; do
  echo "Hello $i"
done
[vagrant@web ~]$ <b>chmod +x !$</b>
chmod +x for.sh
[vagrant@web ~]$ <b>for.sh Foo Bar</b>
Hello Foo
Hello Bar
</pre>
1. try to change the `for` into `for i in $(ls); do` and see the result.
2. Try the following for loop
    <pre>
    #!/bin/bash
    for i in $(seq 60 70); do
      echo "Hello $i"
    done
    </pre>

**Lab**
<pre>
[vagrant@web ~]$ <b>cat for.sh</b>
#!/bin/bash
if (( $# == 0 )); then
  echo -e "\n Error! No filename specified!"
  echo -e "\nUsage: $(basename $0) <filename>"
  exit 1
fi

for i in $@; do #This line is the same as `for i; do`
  check=$(file $i | cut -d " " -f2)

  case $check in
    "ASCII")
      echo -e "\nFile $i is a plain text file"
    ;;
    "Bourne-Again")
      echo -e "\nFile $i is a script file"
    ;;
    "ELF")
      echo -e "\nFile $i is an executable"
    ;;
    "block")
      echo -e "\nFile $i is a Block Device"
    ;;
    *)
      echo -e "\nGuess I am having a bad day, I don't know $1 file type!"
    ;;
  esac
done
[vagrant@web ~]$ <b>for.sh /etc/fstab $(which route) /dev/pts/0 for.sh /dev/sda</b>

File /etc/fstab is a plain text file

File /usr/sbin/route is an executable

Guess I am having a bad day, I don't know /etc/fstab file type!

File for.sh is a script file

File /dev/sda is a Block Device
</pre>
Note that we could use `$1` instead of passing `$i` to the loop body,then we had to use `shift` before the `done` in our for loop.

### while

This is another kind of loop but loops while a condition is TRUE. Here is the syntax:
<pre>
while [ condition ]
do
    do something
    do another thing
done
</pre>

<pre>
VAR=52

while [ $VAR -gt 42 ]
do
    echo VAR is $VAR and it is still greater than 42
    let VAR=VAR-2
    sleep "0.2"
done

echo "VAR is now $VAR and it is not greater than 42"
</pre>

### until
The until loop executes the code while a condition is not true (until it is true)

<pre>
VAR=52

<b>until</b> [ $VAR <b>-le</b> 42 ]
do
    echo VAR is $VAR and it is still greater than 42
    let VAR=VAR-2
    sleep "0.2"
done

echo "VAR is now $VAR and it is not greater than 42"
</pre>

### Creating menus with select statement
<pre>
[vagrant@web ~]$ <b>cat select.sh</b>
#!/bin/bash
options="Sunderland Newcastle"
select choice in $options ; do
  echo "REPLY variable is $REPLY"
  echo "choice variable is $choice"
done
[vagrant@web ~]$ <b>select.sh</b>
1) Sunderland
2) Newcastle
#? <b>1</b>
REPLY variable is 1
choice variable is Sunderland
#? <b>2</b>
REPLY variable is 2
choice variable is Newcastle
#? <b>^C</b>
</pre>

**Lab**
<pre>
#!/bin/bash
options="Sunderland Newcastle"
PS3="Enter your choice: "
echo -e "\nChoose which of the following English football teams you prefer:\n"
select choice in $options ; do
  echo "REPLY variable is $REPLY"
  echo "choice variable is $choice"
  case $choice in
    "Sunderland")
      echo -e "\nNice one! You've obviously got good judgement skills.\n"
      break
    ;;
    "Newcastle")
      echo -e "\nOh dear! deleting your account... "
      break
    ;;
    *)
      echo -e "\nPlease choose a number from the list above."
    ;;
  esac
done
</pre>

### Functions
* Useful for large scripts

<pre>
[vagrant@web ~]$ cat function.sh
#!/bin/bash

educate_user()
{
# trap interrupts; we don't want user be able to do Ctrl+C.
# SIGHUP	1	Hang up detected on controlling terminal or death of controlling process
# SIGINT	2	Issued if the user sends an interrupt signal (Ctrl + C)
# SIGQUIT	3	Issued if the user sends a quit signal (Ctrl + D)
  trap '' 1 2 3
  while [ "$answer" != "Sunderland" ] ; do
    read -p "Enter the name of the world's greatest football team: " answer
    if [ "$answer" = "Sunderland" ] ; then
      tput clear
      echo "Correct! And make sure you don't forget!"
    else
      echo "Nope. Hint... Sunderland."
    fi
  done
}

options="Sunderland Newcastle Everton Liverpool Spurs"
PS3="Enter your choice: "
echo -e "\nChoose which of the following English football teams you prefer:\n"
select choice in $options ; do
  echo "REPLY variable is $REPLY"
  echo "choice variable is $choice"
  case $choice in
    "Sunderland")
      echo -e "\nNice one! You've obviously got good judgement skills.\n"
      break
    ;;
    "Newcastle")
      echo -e "\nOh dear! deleting your account... "
      break
    ;;
    "Everton")
      educate_user
    ;;
    "Liverpool")
      educate_user
    ;;
    "Spurs")
      educate_user
    ;;
    *)
      echo -e "\nPlease choose a number from the list above."
    ;;
  esac
done
</pre>

### Schedule our scripts

Cron is a Linux service.
We use `cron`, `anacron`, `at`, and `batch` for scheduling taks
* Recurring taks:
  * `cron`: down to the minute level
    * Minutes, Hour, Day of Month, Month, Day of week
    * misses jobs if the system is down!
    * system crons are in `/etc/crontab`
      * We also have extension directories for:
        * Cron files: `/etc/cron.d`, `/etc/cron.hourly`,
        * Scripts: `/etc/cron.daily`, `/etc/cron.weekly`, `/etc/cron.monthly`
      * Format is like: `Minute Hour DayOfMonth Month DayOfWeek user command`
    * User crons are like system crons, but we exclude the `user` from the format
      * Format is like: `Minute Hour DayOfMonth Month DayOfWeek command`
      * To see your crons you can use `crontab -l` (list)
      * To edit it you can use `crontab -e` (edit) which will open the cron files with a special editor and will load your inserted crons (if they are correct).
      * The files will be saved at `/var/spool/cron/tabs/` or `/var/spool/crontabs`
      * You should never edit this file; but use `crontab -e` instead
  * `anacron`
    * Can run jobs once a day
    * Doesn't miss jobs if the system is down
    * Designed to run jobs after a certain amount of time after system startup.
* One-off type tasks
  * `at`: runs at specified time and date
  * `batch` job is going to run when for example the load average dips below 0.8

**Lab: system crons**

<pre>
[uwe@localhost ~]$ <b>sudo vi /etc/cron.d/list</b>
*/5 * * * 1-5 root ls /etc > /tmp/lsList # */5 means every 5 minutes
[uwe@localhost ~]$ <b>watch ls /tmp</b>
</pre>

**Lab: user crons**

Note that we don't edit this file itself:
<pre>
[uwe@localhost ~]$ <b>cat /etc/crontab</b>
# We can reas these environment settings in when we run our jobs.
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  *  command to be executed
</pre>

### Using log with your scripts
<pre>
logger -p local5.info \
    "Script started"
logger -p local5.err \
    "Script error"
</pre>

Entry in `/etc/rsyslog.conf`
  * `local5.* /var/log/local5.log`
  
<pre>
*/5 * * * * root  /home/hosseinoliabak/pingGCPWest.sh | logger -n 172.30.51.11 -P 514
</pre>

### Troubleshooting scripts
* Using `echo` is simple and powerful:
  * Returning our variable values for example `echo $choice`
  * To see where we are in the code
* Using `exit` statuses. Exit codes except `0` are excepted as errors.
use `$?` to the exit code returned to the bash
* `xtrace` to turn on: `bash -x ./scriptname`
