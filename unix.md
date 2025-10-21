# Guide to UNIX Shell Programming

A terminal is a program that accepts commands to the operating system. 

The shell runs the commands.

Bash and Zsh are programming languages used to interact with the shell.

## Basic Commands
Use the `=` operator to set variables

Use the `echo` command to print text

Prepend variable names with `$` to access them

Use the `history` command to see a list of previously executed commands

The `clear` command clears the terminal screen but does not delete shell history

## Filesystems

Directories are containers that hold files or other directories.

Files are a dump a raw binary data.

Absolute paths start at the root of the filesystem. Root is denoted by `/`.

### Filesystem Commands

Use `pwd` to see the filepath of the current working directory.

Use `cat filename` to print the contents of a file.

Use `head -n 10 filename` to print the first 10 lines of a file.

Use `tail -n 10 filename` to print the last 10 lines of a file.

Use `less filename` to scroll though a file's contents in interactive mode. Add the `-N` flag to include line numbers.

Use  `touch filename` to create a new file or update the modification timestamp of an existing file.

Use `mkdir directoryname` to create a new directory.

Use `mv filename newfilename` to rename or move a file.

Use `rm filename` to delete a file or `rm -r directoryname` to delete a directory.

Use `cp sourcefile destinationfile` to copy a file. Use `cp -R sourcedir destinationdir` to copy a directory.

Use `grep searchstring file` to search for occurrences of searchstring in file. Add the `-r` flag to search in a directory.

## Permissions

`sudo` allows the user to run commands as the superuser. 


