Linux 101 Command Line Cheat Sheet
========

Abstract
---------
Fundamental Linux/Unix commands for the Linux/Unix command line learner. If you are experienced with Linux/Unix: you have probably mastered these commands. If not: you are in the right place.

These commands are designed for use in the Security511 Linux VM.

Where to Acquire
---------
These tools are installed natively in most Unix/Linux distributions, as well as OS X.

Examples/Use Case
---------
* [bash basics](#bash-basics)
* [cat](#cat)
* [cd](#cd)
* [echo](#echo)
* [ls](#ls)
* [network commands](#network)
* [passwd](#passwd)
* [ping](#ping)
* [pwd](#pwd)
* [sudo](#sudo)

---------
### bash basics
#### Tab-completion:
Folks who are new to the Unix/Linux command line often attempt to type everything by hand. This may work well if you type quickly and accurately. Most of us are **much** better off using tab completion.

Note that Windows PowerShell also supports tab completion, but it handles ambiguity differently. See the PowerShell cheat sheet for more information.

Type the following, and then press the `<TAB>` key:
```bash
$ cat /etc/pas
```
Then press `<TAB>`.

Note that it autocompletes to `/etc/passwd`.

Now try tabbing with ambiguity:
```bash
$ cd ~/Do
```
Then press `<TAB><TAB>`.

Note that it offers two choices: `Documents/ Downloads/`.

Now add a "w" and press `<TAB>`:
```bash
$ cd ~/Dow
```
Press `<TAB>`. It autocompletes to `~/Downloads/`.

### cat
Display a file:
```bash
$ cat example.txt
```
Concatenate (cat) FileA.txt and FileB.txt, create FileC.txt:
```bash
$ cat FileA.txt FileB.txt > FileC.txt
```
---------
### cd
Change Directory (cd) to the /tmp directory:
```bash
$ cd /tmp
```
Change to the home directory. The following commands are equivalent for the Security511 Linux VM "student" user: "~" means home directory (for example: /home/student):
```bash
$ cd
$ cd ~
$ cd /home/student
```
Change to the parent directory. For example: if you are in /tmp/subdirectory/, this will change your working directory to /tmp/:
```bash
$ cd ..
```
---------
### echo
Print (echo) the string "Cylon":
```bash
$ echo Cylon
```
Create or overwrite the file example.txt, containing the string "Cylon":
```bash
$ echo Cylon > example.txt
```
Append the string "Cylon" to the file example.txt:
```bash
$ echo Cylon >> example.txt
```

---------
### ls
List the files in the current directory (equivalent to the cmd.exe "dir" command):
```bash
$ ls
```
List the files in the current directory, long output (-l), all files including "hidden" files that begin with a "." (-a):
```bash
$ ls -la
```
List the files in the current directory, long output (-l), all files (-a), sort by time (-t):
```bash
$ ls -lat
```
List the files in the current directory, long output (-l), all files (-a), reverse (-r) sort by time (-t):
```bash
$ ls -lart
```
---------
### network commands
Show network interface configuration:
```bash
$ ifconfig
```
Show network interface configuration using "ip":
```bash
$ ip a
```
Restart networking:
```bash
$ sudo /etc/init.d/networking restart
```
---------
### passwd
Change your password:
```bash
$ passwd
```
---------
### ping
ping a host forever (until CTRL-C is pressed), see if it is up (and unfiltered):
```bash
$ ping 10.5.11.25
```
ping a host 3 times, see if it is up (and unfiltered):
```bash
$ ping -c3 10.5.11.25
```
---------
### pwd
Print Working Directory (pwd), show the current directory:
```bash
$ pwd
```
---------
### sudo
Run a command as root:
```bash
$ sudo command
```
Open a root bash shell:
```bash
$ sudo bash
```
Additional Info
--------------
A printable PDF version of this cheatsheet is available here:
[LinuxCLI101](pdfs/LinuxCLI101.pdf)

Cheat Sheet Version
--------------
#### **`Version 1.0`**
