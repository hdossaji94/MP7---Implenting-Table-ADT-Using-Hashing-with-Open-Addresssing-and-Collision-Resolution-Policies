# The makefile for MP7.
# Type:
#   make         -- to build program lab6
#
#
# Format for each entry
#    target : dependency list of targets or files
#    <tab> command 1
#    <tab> command 2
#    ...
#    <tab> last command
#    <blank line>   -- the list of commands must end with a blank line

lab7 : table.o lab7.o lab7.c table.c table.h
	gcc -Wall -g table.o lab7.o -o lab7 -lm

table.o : table.c table.c
	gcc -Wall -g -c table.c

lab7.o : lab7.c table.h 
	gcc -Wall -g -c lab7.c
