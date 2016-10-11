# lalinuxbashscr
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
#! /bin/bash
c="ls -s|sort -n"
$c      #error, cannot process pipe
eval $c
```
