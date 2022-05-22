title: A Simple Introduction to Make
date: 2016-04-10 23:11:55
tags:
- Make
- Linux
---
GNU Make is a tool which controls the generation of executables and other non-source files of a program from the program's source files.  
Make gets its knowledge of how to build your program from a file called the makefile, which lists each of the non-source files and how to compute it from other files. When you write a program, you should write a makefile for it, so that it is possible to use Make to build and install the program.  
I will introduce some basic skills about using make.  
<!--more-->

## Format of make 
The format of make rule is:
```bash
target: prerequisite
	command
```
`target` is the output or middle objects. `prerequisite` is the requiring files for target. When `prerequisite` files have update, then when you execute `make` command, the utility will generate target. the `command` indicates how to generate `target`. `command` can be any shell commands. But generally, `commmand` contains the compiling commands. A example of make command:
```bash
file: file.c file.h
	gcc -o file file.c file.h
```
There can be many targets in make file, but the first target will be executed when type `make`.
##Define variables
We can define variables and use it in Makefile. for example:
```bash
OBJ = file.o
file: $(OBJ)
	gcc -o file $(OBJ)
$(OBJ): file.c file.h
	gcc -c file.c file.h
```
In this example, we define `OBJ` as `file.o` and use it later to replace `file.o`.  
It can be quite useful if there are many objects files in target or prerequisite.  
Sometimes we can move object files or head files to other directories, at this time, we can define variables to reduce our typing. For example, you have `*.h` file in `lib` directory in current path, you can write like this:
```bash
LIB = lib
file: file.c $(LIB)/file.h
	gcc -o file file.c $(LIB)/file.h
```

## Phony target 
Phony target is a kind of label in make. It's similar to target, but it has no prerequisite for most time, and we can append it to `make` command to execute command defined in it. For example:
```bash
.PHONY: clean
clean: 
	rm *.o file
```
When we type `make clean` in command line, `rm *.o file` will be executed.  
NOTE: in order to avoid phony target has the same name with file in directory, we add `.PHONY clean` to make sure that clean command must be executed.  
Sometimes phony target can have prerequisite, and place it as the first target, then this phony target will be execute. This is very helpful when you want generate several executable files and you just want type a `make`. For example:
```bash
all: prog1 prog2 prog3  
.PHONY: all  
  
prog1: prog1.o utils.o  
	cc -o prog1 prog1.o utils.o  
  
prog2: prog2.o  
	cc -o prog2 prog2.o  
  
prog3: prog3.o sort.o utils.o  
	cc -o prog3 prog3.o sort.o utils.o 
```

## Automatic variables
There are some default variables in each make rule. We can use it to simplify our work. There are some useful automatic variables:
```bash
$@: The file name of the target of the rule
$%: The target member name, when the target is an archive member
$<: The name of the first prerequisite
$?: The names of all the prerequisites that are newer than the target, with spaces between them
$^: The names of all the prerequisites, with spaces between them

```
For example, if we have a makefile like this:
```bash
CC=gcc
CFLAG=I.
DEPS=hellomake.h

%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAG)
```
Where `$@` indicates the `.o` file and `@<` indicates the corresponding `.c` file.

## Other skills
1.	comments begin with `#`, just like shell
2.	the comment begin with `@` will not be display, so we can echo like this:
```bash
@echo 'Compiling begin...'
```
3.	We can choose make file using `-f` options: `make -f myMakefile` will choose `myMakefile` as rule file.
4.	Adding `-n` in make will not do make really, just test if all things are okay.
5.	We can use `include` to add other makefiles into here, for example: `include Makefile1 Makefile2`.
6.	Add `-` in front of a command will ignore the errors occurring when execute it. 

## Reference:
1.	<https://www.gnu.org/software/make/manual/html_node/>
2.	<http://blog.csdn.net/haoel/article/details/2886>
3.	<http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/>
