---
layout: post
title: Access Rights of Files/Directories
date: 2019-06-11 22:43 +0200
categories: [Linux, Basics]
tags: [linux]
---

![](/assets/images/downloadList.png)

**From the left side:**

The first character shows if it is a file or directory.

File: `-`  
Directory: `d`

Next 9 characters are the access rights (or permissions), and are taken as three groups of 3.

*   The left group of 3 gives the file permissions for the user that owns the file (or directory).
*   The middle group gives the permissions for the group of people to whom the file (or directory) belongs.
*   The rightmost group gives the permissions for all others.

**<u>Access rights on files:</u>**

*   `r (or -)` indicates read permission (or otherwise), that is, the presence or absence of permission to read and copy the file
*   `w (or -)` indicates write permission (or otherwise), that is, the permission (or otherwise) to change a file
*   `x (or -)` indicates execution permission (or otherwise), that is, the permission to execute a file, where appropriate

**<u>Access rights on directories:</u>**

*   `r` allows users to list files in the directory
*   `w` means that users may delete files from the directory or move files into it
*   `x` means the right to access files in the directory. This implies that you may read files in the directory provided you have read permission on the individual files.

Examples: `-rwxrwxrwx`: a file that everyone can read, write and execute (and delete)

On MacOS the symbol `@` indicates that the file has [_<u>extended attributes</u>_](https://en.wikipedia.org/wiki/Extended_file_attributes#OS_X).

The number after shows the number of files inside a directory. The `.` and `..` are also counted.  
If it is a file, the number is `1`.

Next is the owner of the file or directory (in this case: _hungpham_).

The word _staff_ is the group, this can be either _staff_ or _sysadm_ group.

Next is the size of file or directory.

Finally, the day of creation is shown.

### How to change access rights

Command:

`$ chmod [option] [whom][+,-,=][permission] [filename]`  

<table style="width: 60%; border: 1px solid lightgray; text-align: center">

<tbody>

<tr style="margin-left: 0px">

<th>Symbol</th>

<th>Meaning</th>

</tr>

<tr>

<td>u</td>

<td>user</td>

</tr>

<tr>

<td>g</td>

<td>group</td>

</tr>

<tr>

<td>o</td>

<td>other</td>

</tr>

<tr>

<td>a</td>

<td>all</td>

</tr>

<tr>

<td>r</td>

<td>read permission</td>

</tr>

<tr>

<td>w</td>

<td>write (delete) permission</td>

</tr>

<tr>

<td>x</td>

<td>execution (or access directory) permission</td>

</tr>

<tr>

<td>+</td>

<td>add permission</td>

</tr>

<tr>

<td>-</td>

<td>remove permission</td>

</tr>

<tr>

<td>=</td>

<td>set exactly equal</td>

</tr>

</tbody>

</table>

Example: give read and write permissions on the file to all

`$ chmod a+rw file`  

[Reference!](http://www.ee.surrey.ac.uk/Teaching/Unix/index.html)