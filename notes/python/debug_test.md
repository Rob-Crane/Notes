# `pdb`
Can be run from interactive mode with:
        import pdb
        import mymodule
        pdb.run(‘mymodule.test()’)
Alternatively, can be run from command line
        python -m pdb myscript.py
Atypical usage is to insert:
        import pdb; pdb.set_trace()    # useful to hard-code a breakpoint
    at the location to break into debugger.
pdb.run() executes statement passed as arg and can take alternate global and local variables to specify the environment should execute.  pdb.runeval() is the same except will return value of expression.  pdb.runcall() takes a function and its arguments as args (and returns value)
Debugger Commands
Most can be abbreviated to one or two letters
blank lines repeat last commands
commands the debugger doesn’t recognize are assumed to be python statements and executed in the current context
python commands can also be prefixed with ‘!’
‘;;’ separates multiple commands
a .pdbrc in ~/ or ./ is read in and executed as if it had been typed on debugger prompt.  Can be useful for aliases (aliases for commands, can take args)

h(elp) [command]


w(here)
get stack trace
d(own) / u(p) [count]
move up or down the stack trace to older/newer frames
b(reak)
b(reak) [line]
b(reak) [filename:line]
b(reak) [function]
b(reak) … [condition]
print all breakpoints
break at line num in current file

break at first executable statement with that name
must eval to true for breakpoint to be hobored
tbreak ...
like break but removed after first honoring
cl(ear) [filename:lineno]
cl(ear) [bpnumber bpnum…]
clear all breakpoints at line
clear space seperated list of breakpoints
disable [bpnumber bpnum…]
enable [bpnumber bpnum…]
disable/enable but don’t delete those breakpoints (they’ll still be visible with break)
ignore bpnumber [count]
set ignore count for breakpoint, breakpoint honored when count reaches 0
condition bpnumber [condition]


condition bpnumber
set condition that must be true for breakpoint to be honored
delete conditions associated with breakpoint, make it unconditional
commands [bpnumber]




commands
enters a mode and lets you type pdb commands that will be executed when breakpoint is reached.  Finish with ‘end’.  ‘end’ as first command clears existing commands on breakpoint

use last breakpoint set

*any command that would resume execution like continue, next, etc. will terminate the command list (because could encounter another breakpoint creating ambiguities).  Subsequent commands ignored
s(tep)

n(ext)
execute current line and stop at first possible occasion (possibly inside function)
execute current line and only stop on next line of current function
* note that “next” doesn’t necessarily mean numerically subsequent number (could go to beginning of loop block)
until

until [linenum]
execute until numerically subsequent line number reached
execute to that line num
r(eturn)
continue until current function returns
c(ont(inue))
continue until next breakpoint
j(ump) lineno
set the next line that will be executed (only available at bottom of stack).  This lets you skip code or jump back to code already executed.  Some jumps disallowed (like into for loop or out of finally clause.
l(ist)
l(ist) linenum
l(ist) [first,last]
lll
print 11 lines of code at current position
print 11 lines of code at that location
print range
print all code for current function or frame



-> marks current line in current frame
>> marks where an exception was raised
a(rgs)
print argument list of current function
p expression
pp expression
evaluate expression in current context, print value
pretty print value of expression
whatis expression
print type of expression
source expression
try to get source code for given object and display
display [expression]

display
display value of expression if it changes each time execution stops in current frame
list all display expressions in current frame
undisplay
undisplay [expression]
clear all display expressions in current frame
do not display the expression
interact
start interactive interpreter in current environment
alias name command




alias name

alias




unalias
create an alias called name that executes command
can create replaceable parameters with %1,%2, … %* is replaced by all the parameters

show current alias for name

list all aliases

*aliases can be nested and can contain anything legal as pdb command

delete specified alias
run/restart [args]
restart current program with args, preserves current debugger state
q(uit)
exit program


