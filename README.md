# lalinuxbashscr

##2. Variables, Control
The typeset and declare commands for variables
```
typeset -i x 
# x must be an integer
```

```
declare -l    #uppercase values in the vairable are converted to lowercase
declare -u    #lowercase values in the vairable are converted to uppercase
declare -l    #readonly
declare -a    #array
declare -A   #asso array
```

Ex
```
#!/bin/bash
function f1{
  typeset x   #private x
  x=7 
  y=8
}
x=1
y=2
echo $x  #1
echo $y  #2
f1
echo $x  #1
echo $y  #8
```

Ex
```
#!/bin/bash
declare -l lstring="Ab"
declare -u ustring="Cd"
decalre -r ro="k"
declare -a arr1
declare -A arr2

echo $lstring
echo $ustring
arr1[1]="test"
echo ${arr1[1]}
arr2["ke"]="ke"
echo ${arr2["ke"]}
```

###2 Looping with for/while sequences and reading input
02:55 While pipe
```
ls -l |while
read a b c
do
echo $c
done
```
06:34 - seqs
```
`seq 1 4`
{A..Z}
{1..5}
```

for loop in a file
```\
for d in $(<filename)
```
generate a file list
```
for f in $(find . -name *.c)
```


###3 Defining functions and using return and exit
export function name
```
export -f funcname
```
###4 Using file descriptors, file redirection, pipes, and here documents
command > stdout 2> stderr < stdin  

03:03
```
cmd1 2>&1|cmd2
```
same as
```
cmd1 |& cmd2
```

here documents
```
sort<<END
...
END
```
Ex:
```
find /etc -name a >txt   #files
find /etc -name a 2>txt   #error
find /etc -name a &>txt   #both
```


file descriptor ex
```
#!/bin/bash
exec 19<data_file
lsof -p $$
cat 0<&19
exec 7>&1   #save stdout in 7
exec 1>&-  # close stdout
lsof -p $$
cat
exec 1>&7  #copy 7 to stdout
cat
```
((tbc)


###5 Control-flow case statements and if-then-else with the test command
case is not regex, just file globing
```
case $ans in
 yes|YES|y.x    #just y.x, not yax,ybx..
```

```
test -d x  #test is directory
test -f x  #test is regular file
test -s x  #success if x exists and not empty
```


string compare and numeric cmppare
 #string
```
[ $x==$y ]  #string
```
numeric
```
[ $x -eq $y ]
(($x==$x))
```

###6 Using arithmetic operators


###8 Solutions: Using local variables in functions, loops, and arithmetic
strings command, print all printable strings from file/output
```
strings a.txt
```


##3. Using Filters and Parameter
##2 Using sed and AWK for more powerful scripting
```
sed -f sedscript -n inputfile
sed '1,5p'
```

```
sed '/important/!s/print/throw'
```

Ex:sedfunc
```
sed -n "$1,$2p"
```
then
```
bash sedfunc 2 5 <inputfile
```


###3 Positional parameters and operators with braces
04:30
string operations  
${var#pre} remove matching prefix  
${var#post} remove matching sufffix

06:15 update value
```
#!/bin/bash
x=abc
abc="new"
echo $x
echo $abc
echo ${!x}  #will print new
```


unset ex
```
#!/bin/bash
unset x
a=${x:-Dog}
echo a is $a
echo x is $x

a=${x:=Dog}
echo a is $a
echo x is $x

unset x
${x:?}
```
result:
```
a is Dog
x is
a is Dog
x is Dog
error...
```
#s, offset, example
```
#!/bin/bash
a="a word"
sub=${a:2}  #word
sub=${a:2:2} #wo
length=${#a} #6
```

suffix and prefix
```
#!/bin/bash
p="/usr/bin/ke.sh"
echo ${p#/*bin/}   #ke.sh
echo ${p%.sh}    # /usr/bin/ke
```






##4. Advanced Bash
###1 Using the coproc command
A coprocess is a backgrond process where your shell gets file descriptors for the process's stdin and stdout  
Ex: we need a filter
```
#!/bin/bash
declare -l line
while
  read line
do
  echo $line | tr "abc" "ABC"
done
```
execute
```
coproc ./ke.sh
````
then
```
echo abc >& "${COPROC[1]}"
cat <& "${COPROC[0]}"
```
(0=input 1=output).  


We can also give a proc name
```
coproc my { ./ke.sh; }
```
then
```
echo abc >& "${my[1]}"
cat <& "${my[0]}"
```
show jobs
```
jobs
```



###2 Debugging scripts with -x and -u options

reports usage of unused variable
```
set -u
```
tracing output
```
bash -x ke.sh
```
or in program, set
```
set -x  #echo output
set +x #cancel echo output.
```
check syntax errors
```
bash -n ke.sh
```


###3 Signals and traps
Bash has a command, trap, that you use for catching signals and handling signals.  
A handy thing to use the trap for is to gracefully die when the user does Control + C, instead of leaving behind some temporary files, for example, you can clean those up, and then terminate.   
ctrl c = INT signal  
ctrl \  (backslash)= quit signal  

Ex
```
#! /bin/bash
trap 'int;exit' INT
trap 'echo cannot quit' QUIT
while
true
do
  sleep 5
done
```
diff between int and quit: quit only stop current script, for example, stop sleep. but cannot terminate shell


###4 Using the eval and getopt commands
```
function usage{
  echo Options are -r -h -b --branch --version --help
  exit 1
}

function handleopts{
  OPTS =`getopt -o r:hb:: -l branch:: -l help -l version -- "$@"`
  if [ $? !=0 ]
  then
    echo ERROR parsing arguments >$2
    exit 1
  fi
  eval set == "$OPTS"
  while true; do
    case '$1" in
      -r) rightway=$2
        shift 2;;
      --version|-v) echo"v1.2";
        exit 0;;
      -h|--help) usage;
        shift;;
     -b|--branch)
      case "$2" in
      "") branchname="default"; shift 2;;
      *) branchname="$2"; shift 2;;
    esac;;
  --) shift; break;;
esac
done
if [ "$#" -ne 0 ]
then
  echi Error arg "$@"
  usage
fi
}
wightway=no
haldleopts $@
```

###6 Solutions: Debugging scripts using trap, eval, getopt, and coproc
a.sh
```
#!/bin/bash
c="ls -s|sort -n"
$c      #error, cannot process pipe
eval $c
```
b.sh
```
#! /bin/bash
echo options are $@
opts="a b \$1 \$2 "
set -- "$opts"
#eval set -- $opts
echo options are: $@
```


d.sh proc test
```
while
read line
do
  echo $line |sed 's/flag/banner/g'
done
```

call
```
coproc mysed { ./d.sh; }
echo ${mysed[1]}  //58
echo ${mysed[0]}  //62
cat txt >&58
cat <&62
```
