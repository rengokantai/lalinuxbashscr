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
