notes on Bash Scripting that I took during Reindert-Jan Ekker's *Shell Scripting With Bash* course

## Making a new script
* first line of every script: `#!/bin/bash` will tell the shell to use the bash interpreter
* make executable for user: `chmod u+x <filename>`
* make executable for all: `chmod a+x <filename>`
* if you don't have access to make it executable, just use `bash filename` to run the script

## Debugging
* set the `-x` switch at the top of the file (next to `#!/bin/bash`)
* use `set -x` and `set +x` to turn debugging on and off inside a script
* run your script by passing `-x` as an option: `bash -x myscript`

## General Bash Stuff
* use the `type` command to see if a script name is taken. i.e. `type ls`.  Can also be used in expression testing
* use `$?` to capture the return value (often the exit code) of the last command
* `$#` captures the number of arguments to a shell script
* white spaces are special in BASH. to assign variables, do not use white space. i.e. `var=1`
* always use `echo` to see the value of a variable. otherwise you will simply execute the value of the variable as a command
i.e. `echo $variable` prints variable value, `$variable` executes the variable as a command
* Always a good idea to use double quotes around variable names.
* Single quotes will escape every character (completely literal strings) while double quotes allow variable insertion with `$`
* `echo -e` allows the use of escape chars
* `printf` is similar to Java for printing formatted strings.  e.g. `printf "\nyou said %s \n" $variable`
* **Brace expansion**: `touch note{1..9}.md`
* **Command Substitution**: `$(date)` is an expression which is replaced by some value (in this case, `date`) and then executed as a command


## Variables
* variables are strings by default but can be set with special attributes: `declare -i variableName` means `$variableName` will only hold integers.  any other attempted assignment will result in a 0 being assigned
* unset with a + :  `declare +i variableName`
* `let n=100/2` and `((n=100 / 2))` are examples of arithmetic expressions
* `-r` is a read only variable
* `export variable='value'` || `declare -x variable='value'`
* exports defined in a `bashrc` or other config file will be available to all subprocesses in that interactive session
* get length of someVariable: `echo ${#someVariable}`
* bash variables are globally visible

## Special variables
* `$#` is the number of script arguments
* `$?` is the exit code of last command
* `${#variable}` gets the length

## Conditionals
* `[[ $string ]]` checks to see if the string exists
* `[[ $string = "testValue" ]]` checks to see if the values are equal - MUST HAVE SPACES
* `[[ $string="testValue" ]]` is an assignment and will always return true
* `[[ -e $filename ]]` checks to see if the file exists
* `[[ -d $directory ]]` checks to see if the directory exists
* `==` and '!=' actually are doing pattern matching.  use quotes to do exact string matching
```
pattern matching: [[ hello == h*o ]]  TRUE
string matching: [[ hello == "h*o" ]] FALSE
```
* `=~` is POSIX RegEx matching
* `BASH_REMATCH` is how to access regular expression pattern match groupings.  [read more here](http://wiki.bash-hackers.org/syntax/pe?s[]=rematch)

### more conditionals
* single bracket `[` is the classical `test` command
* `[[` is the bash extension, special syntax. easier to use (for example, no quotes needed around variables. preferred way to go.
* when comparing numbers, use `-eq` `-ne` `-lt` `-gt`.  DO NOT USE traditional comparison operators

## Command Groups: an alternative to Conditionals
* alternative to `if` statements:
```
[[ $# -ne 2 ]] && { echo 'minimum of 2 arguments required' >&2; exit 1; }
```
NOTE that spaces around curly braces are required and semi-colons (or new lines) after each command are required

## Job control
* use `&` at end of line to run a job in the background
* redirect output if it will mess up your screen, i.e. `./bg-job.sh > /dev/null &`
* `top` to view sorted list of processes
* `ps` is process status
* `jobs` to inspect, `kill %1` to end first job
* `pkill` kills by name, `xkill` kills by click

## Basic for loop over files
```
for f in *.md; do
  echo $(date) > $f
done
```

## the C style for loop is also available
```
for (( <INIT>; <TEST>; <UPDATE> )); do
  # werk
done    
```

## Running scripts
* use the `source` command to read a file and import the code into the current shell process.  for example, can be used similar to import statements in other languages
* `nohup` disconnects the script from the terminal session
* `exec >logfile` at the top of a script will redirect all output of your script to a logfile


## Output streams
* `2>&1` puts the standard error stream into the standard output stream
* `> /dev/null 2>&1` puts the standard output stream to dev null and then the error output stream to standard output

## Script Parameters
* `$1 $2` etc hold all the parameters in order (positional parameters)
* `$0` holds the name of script as it was called.  can be used for error reporting for good user experience
* `$@` holds all the arguments.  best use is `"$@"` which captures multi-word string arguments as single groupings
* `$#` holds number of arguments passed in
* `getopts` is how option flags like `-a option` are passed to a script.  use a case/esac statement to parse them

## End Of Options
* use `--` to tell bash that any parameters following are not options.  
* example usage: remove a file called `-l.txt` which bash will think is an option passed to `rm`
* good idea to use `end of options` switch to provide safety against user input or file system data that starts with a dash

## Functions
* use `declare` or `local` inside a function to block scope a variable
* use `return` to exit a function
* same positional parameters as with a script
* without a return statement, function returns status of last statement executed
* to return any other value besides status, send the data to output i.e. `echo`
* `export -f functionName` is how to export to subprocesses
* redirection can be done immediately after a function declaration: `function() {} >&2`

## Pipes
* commands executed in a pipeline are run in a sub-shell which can cause a problem with local variable sharing.  one solution is to use a tempfile

## String Manipulation with Parameter expansion
* **Parameter expansion** is the technique.  One example: `${#someVariable}` provides the length of the variable
* `${variableName#pattern}` removes the shortest match from beginning of string
* `${variableName##pattern}` removes the longest match from the beginning
* `${variableName%pattern}` removes the shortest match from the end
* `${variableName%%pattern}` removes the longest match from the beginning
```
x='iliketoast/it/tastes/so/good.txt'
echo ${x#*/} #=> it/tastes/so/good.txt
echo ${x##*/} #=> good.txt
```
* **search and replace**: `${var/pattern/replacementString}` replaces the first match
* **search and replace**: `${var//pattern/replacementString}` replaces globally
* `#` matches at beginning of string
* `%` matches at end of string
* `[]` matches a pattern
* `${myVar//[aeiou]/.}` will replace every lowercase vowel with a `.`
* Default Values!  `${variableName:-value}` will evaluate to the value if the variable is empty not set
* `${var-value}` will only work if the var is unset (variable will evaluate to empty if the variable is empty)
* `${var:=value}` will actually set the value to the variable, not just evaluate as the value. `${var=value}` follows same rule as above

## nifty program to create a new bash script, set permissions and more
```
#!/bin/bash

# Was there a filename supplied?
if [[ ! $1 ]]; then
  echo "Missing argument"
  exit 1
fi

filename="$1"

if [[ -e $filename ]]; then
  echo "Script $filename already exists"
  exit 1
fi

if type $filename; then
  echo "There is already a command with that name, $filename"
  exit 1
fi

touch $filename
echo "#!/bin/bash" > $filename
chmod a+x $filename
vi $filename

exit 0
```
