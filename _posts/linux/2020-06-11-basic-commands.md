---
author: hung
title: Basics of Unix / Linux Commands
date: 2020-06-11 22:43 +0200
categories: [Linux, Basics]
tags: [macos, linux]
---

*   List what inside directory  
    `$ ls  
    $ ls -a  
    $ ls -l`
*   Make new directory  
    `$ mkdir`
*   Change to directory  
    `$ cd [directory]`  
    Current directory (.)  
    `$ cd .`  
    Parent directory (..)  
    `$ cd ..`
*   Print working directory  
    `$ pwd`
*   Copy file (makes a copy of file1 in the current working directory and calls it file2)  
    `$ cp _file1_ _file2_  
    // copy file to current directory:  
    $ cp /home/hungpham/something.txt .  
    // copy folder:  
    $ cp -a /source/. /dest/  
    $ cp -R ~/Desktop/MyFolder /Documents`
*   Move file or folder  
    `$ mv _file1_ _file2_`
*   Remove files:  
    `$ rm [file]` Remove folder:  
    `$ rm -r [folder]`
*   Clear terminal screen:  
    `$ clear`
*   Display the contents of a file on the screen (concatenate):  
    `$ cat [file.txt]`
*   Write the contents of a file onto the screen a page at a time:  
    `$ less [file]`  
    Press the [**space-bar**] if you want to see another page, and type [**q**] if you want to quit reading. As you can see, less is used in preference to cat for long files.

*   Write the first ten lines of a file to the screen:  
    `$ head [file]  
    $ head -10 [file]`
*   Write the last ten lines of a file to the screen:  
    `$ tail [file]`
*   Search through text with `less` command:  
    `$ less [file]`  
    (still in less) type [/] followed by the word to search:  
    `/[word]`  
    Type [n] to see next occurrence.

*   grep: searches files for specified words or patterns  
    `$ grep [word] [file]  
    $ grep -i [word] [file]`
    *   -v display those lines that do NOT match
    *   -n precede each matching line with the line number
    *   -c print only the total count of matched lines`grep -ivc [word] [file]`
*   Word count:  
    `$ wc -w [file]`  
    Line count:  
    `$ wc -l [file]`
*   The `>` symbol: redirect the output of a command  
    `$ cat > [file]`  
    Join file:  
    `$ cat [file1] [file2] > [bigfile]`
*   The form >> appends standard output to a file  
    `$ cat >> file`
*   `<`: redirect the input of a command
*   Sort alphabetically or numerically sorts a list  
    `$ sort  
    $ sort < [bigfile]  
    $ sort < [bigfile] > [sortedfile]`
*   See who is on the system  
    `$ who  
    $ who > names.txt  
    `
*   Pipes: `|`  
    `$ who | sort  
    $ who | wc -l`  
    Example: display all lines of [list1] and [list2] containing the letter 'p', and sort the result  
    `$ cat list1 list2 | grep p | sort`
*   `*`: wildcard, which match against none or more character(s) in a file (or directory) name  
    Example: list all files in the current directory starting with list....  
    `$ ls list*  
    $ ls *.java`  

    The character `?` will match exactly one character. So ?ouse will match files like house and mouse, but not grouse.  
    `$ ls ?list`
*   On-line manuals  
    `$ man wc  
    $ whatis wc`
*   When not sure of the exact name of a command  
    `$ apropos [keyword]  
    $ apropos move`
*   Change permission of file or directory:  
    `$ chmod a+rw file`  
    See [_<u>How to change access rights of files/directories</u>_](/unix-linux/access-rights)
*   Sleep for 5 seconds:

    `$ sleep 5`
*   See process (running and done):

    `$ ps`
*   Continue suspending process in background:

    `$ bg`
*   List all processes:

    `jobs`
*   Foreground a process

    `$ fg [jobnumber]`
*   Kill job

    `$ kill [jobnumber]`

    Force to kill

    `$ kill -9 [jobnumber]`