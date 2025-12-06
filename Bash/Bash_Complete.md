# Bash Scripting Complete Guide

## Table of Contents
- [Introduction](#introduction)
- [Getting Started](#getting-started)
- [Variables](#variables)
- [Input and Output](#input-and-output)
- [Operators](#operators)
- [Conditional Statements](#conditional-statements)
- [Loops](#loops)
- [Functions](#functions)
- [Arrays](#arrays)
- [String Manipulation](#string-manipulation)
- [Command Line Arguments](#command-line-arguments)
- [File Operations](#file-operations)
- [Process Management](#process-management)
- [Regular Expressions](#regular-expressions)
- [Error Handling](#error-handling)
- [Debugging](#debugging)
- [Best Practices](#best-practices)
- [Advanced Topics](#advanced-topics)
- [Practical Examples](#practical-examples)

---

## Introduction

Bash (Bourne Again Shell) is a command processor that typically runs in a text window where users type commands that cause actions. Bash can also read and execute commands from a file, called a shell script.

### Why Bash Scripting?

- **Automation**: Automate repetitive tasks
- **System Administration**: Manage servers and systems
- **DevOps**: CI/CD pipelines, deployment scripts
- **Task Scheduling**: Cron jobs and scheduled tasks
- **File Management**: Batch file operations
- **Text Processing**: Parse and manipulate data

---

## Getting Started

### Shebang

```bash
#!/bin/bash
# This is the shebang - tells system to use bash to execute

#!/bin/sh
# Use sh (more portable but fewer features)

#!/usr/bin/env bash
# More portable - finds bash in PATH
```

### Creating and Running Scripts

```bash
# Create script file
nano script.sh

# Make executable
chmod +x script.sh

# Run script
./script.sh

# Run without execute permission
bash script.sh
sh script.sh

# Run with debugging
bash -x script.sh
```

### First Script

```bash
#!/bin/bash

# Hello World
echo "Hello, World!"

# Multi-line script
echo "Today is $(date)"
echo "Current user: $USER"
echo "Home directory: $HOME"
```

### Comments

```bash
# Single line comment

: '
Multi-line comment
This is a comment block
Everything here is ignored
'

# Inline comment
echo "Hello" # This prints Hello
```

---

## Variables

### Variable Declaration

```bash
#!/bin/bash

# Simple variable
NAME="John"
AGE=30

# No spaces around =
# WRONG: NAME = "John"
# CORRECT: NAME="John"

# Using variables
echo $NAME
echo ${NAME}  # Recommended - clearer

# Command substitution
TODAY=$(date)
CURRENT_DIR=`pwd`  # Old style - avoid

# Assign command output
FILES=$(ls -l)
USER_COUNT=$(who | wc -l)
```

### Variable Types

```bash
# String
NAME="John Doe"
MESSAGE='Hello World'

# Integer
AGE=30
COUNT=100

# Array
COLORS=("red" "green" "blue")

# Readonly variable
readonly PI=3.14159
# PI=3.14  # This will cause error

# Environment variable
export DATABASE_URL="postgres://localhost/mydb"

# Local variable (in functions)
function myFunc() {
    local LOCAL_VAR="only in function"
}
```

### Special Variables

```bash
#!/bin/bash

# Script name
echo "Script name: $0"

# Arguments
echo "First argument: $1"
echo "Second argument: $2"

# All arguments
echo "All arguments: $@"
echo "All arguments as string: $*"

# Number of arguments
echo "Number of arguments: $#"

# Last command exit status
ls /tmp
echo "Exit status: $?"

# Current process ID
echo "PID: $$"

# Last background process ID
sleep 10 &
echo "Background PID: $!"

# Current shell options
echo "Shell options: $-"
```

### Variable Expansion

```bash
# Basic expansion
NAME="John"
echo "Hello, $NAME"
echo "Hello, ${NAME}"

# Default values
echo ${VAR:-"default"}        # Use default if VAR is unset
echo ${VAR:="default"}        # Assign default if VAR is unset
echo ${VAR:?"Variable not set"}  # Error if VAR is unset
echo ${VAR:+"alternate"}      # Use alternate if VAR is set

# String length
NAME="John Doe"
echo ${#NAME}  # Output: 8

# Substring
TEXT="Hello World"
echo ${TEXT:0:5}   # Hello
echo ${TEXT:6}     # World

# String replacement
TEXT="Hello World"
echo ${TEXT/World/Universe}    # Hello Universe
echo ${TEXT//o/0}              # Hell0 W0rld (replace all)

# Remove pattern
FILE="document.txt"
echo ${FILE%.txt}              # document (remove shortest match)
echo ${FILE%.*}                # document
echo ${FILE##*/}               # document.txt (remove longest match)

# Case conversion (Bash 4+)
TEXT="Hello World"
echo ${TEXT,,}                 # hello world (lowercase)
echo ${TEXT^^}                 # HELLO WORLD (uppercase)
echo ${TEXT^}                  # Hello world (capitalize first)
```

---

## Input and Output

### Echo and Printf

```bash
#!/bin/bash

# Echo
echo "Simple text"
echo -n "No newline"
echo -e "Line 1\nLine 2"  # Enable escape sequences
echo -e "Tab\tseparated"

# Printf (more control)
printf "Hello, %s\n" "World"
printf "Number: %d\n" 42
printf "Float: %.2f\n" 3.14159
printf "%-10s %5d\n" "Name" 123

# Formatted output
printf "%-20s %-10s %-10s\n" "Name" "Age" "City"
printf "%-20s %-10d %-10s\n" "John" 30 "NYC"
printf "%-20s %-10d %-10s\n" "Jane" 25 "LA"
```

### Reading Input

```bash
#!/bin/bash

# Simple read
echo "Enter your name:"
read NAME
echo "Hello, $NAME"

# Read with prompt
read -p "Enter your age: " AGE
echo "You are $AGE years old"

# Read password (hidden input)
read -sp "Enter password: " PASSWORD
echo
echo "Password saved"

# Read multiple values
read -p "Enter first and last name: " FIRST LAST
echo "First: $FIRST, Last: $LAST"

# Read with timeout
read -t 5 -p "Enter input (5 sec timeout): " INPUT
if [ $? -ne 0 ]; then
    echo "Timeout!"
fi

# Read single character
read -n 1 -p "Press any key to continue..."
echo

# Read from file
while read line; do
    echo "Line: $line"
done < file.txt

# Read CSV
while IFS=, read -r name age city; do
    echo "Name: $name, Age: $age, City: $city"
done < data.csv

# Here document
cat << EOF
This is a here document
Multiple lines
Can contain variables: $USER
EOF

# Here string
cat <<< "This is a here string"

# Read into array
readarray -t LINES < file.txt
# Or
mapfile -t LINES < file.txt
```

### Output Redirection

```bash
# Redirect stdout to file (overwrite)
echo "Hello" > output.txt

# Redirect stdout to file (append)
echo "World" >> output.txt

# Redirect stderr
ls /nonexistent 2> error.log

# Redirect both stdout and stderr
command > output.txt 2>&1
command &> output.txt  # Shorthand

# Redirect to /dev/null (discard)
command > /dev/null 2>&1

# Redirect stdin
command < input.txt

# Here document to command
cat > file.txt << EOF
Line 1
Line 2
EOF

# Tee (output to both file and stdout)
echo "Test" | tee output.txt
echo "Test" | tee -a output.txt  # Append

# Multiple redirections
{
    echo "Line 1"
    echo "Line 2"
} > output.txt
```

---

## Operators

### Arithmetic Operators

```bash
#!/bin/bash

# Let command
let "a = 5 + 3"
let "b = a * 2"
let "c = b / 4"
let "d = b % 3"
echo "a=$a, b=$b, c=$c, d=$d"

# Double parentheses (preferred)
((a = 5 + 3))
((b = a * 2))
((c = b / 4))
((d = b % 3))
echo "a=$a, b=$b, c=$c, d=$d"

# Arithmetic expansion
result=$((5 + 3))
echo $result

# Increment/Decrement
((count++))
((count--))
((count += 5))
((count -= 3))
((count *= 2))
((count /= 2))

# Operations
a=10
b=3
echo "a + b = $((a + b))"
echo "a - b = $((a - b))"
echo "a * b = $((a * b))"
echo "a / b = $((a / b))"
echo "a % b = $((a % b))"
echo "a ** b = $((a ** b))"  # Exponentiation

# BC for floating point
result=$(echo "scale=2; 10 / 3" | bc)
echo $result  # 3.33
```

### Comparison Operators

```bash
#!/bin/bash

# Numeric comparison
if [ 5 -eq 5 ]; then echo "Equal"; fi
if [ 5 -ne 3 ]; then echo "Not equal"; fi
if [ 5 -gt 3 ]; then echo "Greater than"; fi
if [ 5 -ge 5 ]; then echo "Greater or equal"; fi
if [ 3 -lt 5 ]; then echo "Less than"; fi
if [ 5 -le 5 ]; then echo "Less or equal"; fi

# Modern syntax (preferred)
if (( 5 == 5 )); then echo "Equal"; fi
if (( 5 != 3 )); then echo "Not equal"; fi
if (( 5 > 3 )); then echo "Greater than"; fi
if (( 5 >= 5 )); then echo "Greater or equal"; fi
if (( 3 < 5 )); then echo "Less than"; fi
if (( 5 <= 5 )); then echo "Less or equal"; fi

# String comparison
if [ "abc" = "abc" ]; then echo "Equal"; fi
if [ "abc" != "def" ]; then echo "Not equal"; fi
if [ "abc" \< "def" ]; then echo "Less than"; fi
if [ "def" \> "abc" ]; then echo "Greater than"; fi
if [ -z "" ]; then echo "String is empty"; fi
if [ -n "text" ]; then echo "String is not empty"; fi

# Modern string comparison
if [[ "abc" == "abc" ]]; then echo "Equal"; fi
if [[ "abc" != "def" ]]; then echo "Not equal"; fi
```

### Logical Operators

```bash
#!/bin/bash

# AND
if [ 5 -gt 3 ] && [ 10 -gt 5 ]; then
    echo "Both true"
fi

if [ 5 -gt 3 -a 10 -gt 5 ]; then
    echo "Both true"
fi

if [[ 5 -gt 3 && 10 -gt 5 ]]; then
    echo "Both true"
fi

# OR
if [ 5 -gt 3 ] || [ 2 -gt 5 ]; then
    echo "At least one true"
fi

if [ 5 -gt 3 -o 2 -gt 5 ]; then
    echo "At least one true"
fi

if [[ 5 -gt 3 || 2 -gt 5 ]]; then
    echo "At least one true"
fi

# NOT
if [ ! 2 -gt 5 ]; then
    echo "Not true"
fi

if ! [ 2 -gt 5 ]; then
    echo "Not true"
fi
```

### File Test Operators

```bash
#!/bin/bash

FILE="/etc/passwd"

# File existence
if [ -e "$FILE" ]; then echo "File exists"; fi
if [ ! -e "$FILE" ]; then echo "File does not exist"; fi

# File type
if [ -f "$FILE" ]; then echo "Regular file"; fi
if [ -d "$FILE" ]; then echo "Directory"; fi
if [ -L "$FILE" ]; then echo "Symbolic link"; fi
if [ -b "$FILE" ]; then echo "Block device"; fi
if [ -c "$FILE" ]; then echo "Character device"; fi
if [ -p "$FILE" ]; then echo "Named pipe"; fi
if [ -S "$FILE" ]; then echo "Socket"; fi

# Permissions
if [ -r "$FILE" ]; then echo "Readable"; fi
if [ -w "$FILE" ]; then echo "Writable"; fi
if [ -x "$FILE" ]; then echo "Executable"; fi

# File properties
if [ -s "$FILE" ]; then echo "File size > 0"; fi
if [ -O "$FILE" ]; then echo "Owned by user"; fi
if [ -G "$FILE" ]; then echo "Owned by group"; fi

# Special permissions
if [ -u "$FILE" ]; then echo "SUID set"; fi
if [ -g "$FILE" ]; then echo "SGID set"; fi
if [ -k "$FILE" ]; then echo "Sticky bit set"; fi

# File comparison
if [ file1.txt -nt file2.txt ]; then echo "file1 newer than file2"; fi
if [ file1.txt -ot file2.txt ]; then echo "file1 older than file2"; fi
if [ file1.txt -ef file2.txt ]; then echo "Same file"; fi
```

---

## Conditional Statements

### If-Else

```bash
#!/bin/bash

# Simple if
if [ condition ]; then
    echo "Condition is true"
fi

# If-else
if [ $1 -gt 10 ]; then
    echo "Greater than 10"
else
    echo "Less than or equal to 10"
fi

# If-elif-else
if [ $1 -gt 10 ]; then
    echo "Greater than 10"
elif [ $1 -eq 10 ]; then
    echo "Equal to 10"
else
    echo "Less than 10"
fi

# Multiple conditions
if [ $1 -gt 5 ] && [ $1 -lt 15 ]; then
    echo "Between 5 and 15"
fi

# Using [[ ]] (recommended - more features)
if [[ $1 -gt 5 && $1 -lt 15 ]]; then
    echo "Between 5 and 15"
fi

# Pattern matching with [[ ]]
if [[ "$1" == *.txt ]]; then
    echo "Text file"
fi

# Regex matching
if [[ "$EMAIL" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi

# One-line if
[ -f file.txt ] && echo "File exists"
[ ! -f file.txt ] && echo "File does not exist"

# Ternary-like operation
result=$( [ $a -gt $b ] && echo "$a" || echo "$b" )
```

### Case Statements

```bash
#!/bin/bash

# Basic case
case "$1" in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    restart)
        echo "Restarting..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac

# Multiple patterns
case "$1" in
    start|START)
        echo "Starting..."
        ;;
    stop|STOP)
        echo "Stopping..."
        ;;
    *)
        echo "Unknown command"
        ;;
esac

# Pattern matching
case "$filename" in
    *.txt)
        echo "Text file"
        ;;
    *.jpg|*.png|*.gif)
        echo "Image file"
        ;;
    *.sh)
        echo "Shell script"
        ;;
    *)
        echo "Unknown file type"
        ;;
esac

# Ranges
case "$grade" in
    [Aa])
        echo "Excellent"
        ;;
    [Bb])
        echo "Good"
        ;;
    [Cc])
        echo "Average"
        ;;
    [Dd])
        echo "Poor"
        ;;
    [Ff])
        echo "Fail"
        ;;
    *)
        echo "Invalid grade"
        ;;
esac

# Fall through (no break)
case "$1" in
    verbose)
        VERBOSE=true
        ;&  # Fall through
    debug)
        DEBUG=true
        ;;
esac
```

---

## Loops

### For Loop

```bash
#!/bin/bash

# Basic for loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# Range
for i in {1..10}; do
    echo "Number: $i"
done

# Range with step
for i in {0..20..2}; do
    echo "Even number: $i"
done

# C-style for loop
for ((i=1; i<=10; i++)); do
    echo "Count: $i"
done

# Loop over files
for file in *.txt; do
    echo "Processing: $file"
done

# Loop over command output
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "User: $user"
done

# Loop over array
FRUITS=("Apple" "Banana" "Orange")
for fruit in "${FRUITS[@]}"; do
    echo "Fruit: $fruit"
done

# Loop over directories
for dir in */; do
    echo "Directory: $dir"
done

# Loop over arguments
for arg in "$@"; do
    echo "Argument: $arg"
done

# Loop with index
for i in "${!FRUITS[@]}"; do
    echo "Index $i: ${FRUITS[$i]}"
done
```

### While Loop

```bash
#!/bin/bash

# Basic while
counter=1
while [ $counter -le 5 ]; do
    echo "Counter: $counter"
    ((counter++))
done

# While with condition
while true; do
    read -p "Enter 'quit' to exit: " input
    if [ "$input" == "quit" ]; then
        break
    fi
done

# Read file line by line
while read line; do
    echo "Line: $line"
done < file.txt

# While with IFS (field separator)
while IFS=: read -r user pass uid gid info home shell; do
    echo "User: $user, Shell: $shell"
done < /etc/passwd

# Infinite loop
while true; do
    echo "Press Ctrl+C to stop"
    sleep 1
done

# While with multiple conditions
counter=0
while [ $counter -lt 10 ] && [ $counter -ne 5 ]; do
    echo "Counter: $counter"
    ((counter++))
done
```

### Until Loop

```bash
#!/bin/bash

# Basic until (runs until condition is true)
counter=1
until [ $counter -gt 5 ]; do
    echo "Counter: $counter"
    ((counter++))
done

# Until with file check
until [ -f /tmp/done.txt ]; do
    echo "Waiting for file..."
    sleep 1
done

# Until service is running
until systemctl is-active --quiet nginx; do
    echo "Waiting for nginx to start..."
    sleep 2
done
```

### Loop Control

```bash
#!/bin/bash

# Break - exit loop
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break
    fi
    echo "Number: $i"
done

# Continue - skip to next iteration
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue
    fi
    echo "Number: $i"
done

# Break multiple loops
for i in {1..3}; do
    for j in {1..3}; do
        if [ $i -eq 2 ] && [ $j -eq 2 ]; then
            break 2  # Break outer loop
        fi
        echo "i=$i, j=$j"
    done
done

# Continue with label (Bash 4+)
for i in {1..5}; do
    for j in {1..5}; do
        [[ $j -eq 3 ]] && continue 2
        echo "$i-$j"
    done
done
```

---

## Functions

### Function Declaration

```bash
#!/bin/bash

# Method 1
function greet {
    echo "Hello, World!"
}

# Method 2 (preferred)
greet() {
    echo "Hello, World!"
}

# Call function
greet
```

### Function Parameters

```bash
#!/bin/bash

# Function with parameters
greet() {
    echo "Hello, $1 $2"
}

greet "John" "Doe"

# Access all parameters
print_all() {
    echo "All parameters: $@"
    echo "Number of parameters: $#"
}

print_all one two three

# Default parameter
greet_with_default() {
    local name=${1:-"Guest"}
    echo "Hello, $name"
}

greet_with_default
greet_with_default "John"
```

### Return Values

```bash
#!/bin/bash

# Return numeric status (0-255)
is_even() {
    if (( $1 % 2 == 0 )); then
        return 0  # Success (true)
    else
        return 1  # Failure (false)
    fi
}

if is_even 4; then
    echo "4 is even"
fi

# Return string via echo
get_user() {
    echo "john_doe"
}

user=$(get_user)
echo "User: $user"

# Return multiple values
get_info() {
    echo "John Doe 30 Engineer"
}

read name surname age profession <<< $(get_info)
echo "Name: $name $surname"
echo "Age: $age"
echo "Profession: $profession"
```

### Local Variables

```bash
#!/bin/bash

# Global variable
GLOBAL_VAR="I am global"

my_function() {
    local LOCAL_VAR="I am local"
    GLOBAL_VAR="Modified global"

    echo "Inside function:"
    echo "  Local: $LOCAL_VAR"
    echo "  Global: $GLOBAL_VAR"
}

my_function

echo "Outside function:"
echo "  Local: $LOCAL_VAR"      # Empty
echo "  Global: $GLOBAL_VAR"    # Modified
```

### Recursive Functions

```bash
#!/bin/bash

# Factorial
factorial() {
    if [ $1 -le 1 ]; then
        echo 1
    else
        local prev=$(factorial $(($1 - 1)))
        echo $(($1 * prev))
    fi
}

result=$(factorial 5)
echo "5! = $result"

# Fibonacci
fibonacci() {
    if [ $1 -le 1 ]; then
        echo $1
    else
        local a=$(fibonacci $(($1 - 1)))
        local b=$(fibonacci $(($1 - 2)))
        echo $((a + b))
    fi
}

echo "Fibonacci(10) = $(fibonacci 10)"
```

---

## Arrays

### Indexed Arrays

```bash
#!/bin/bash

# Declare array
FRUITS=("Apple" "Banana" "Orange")

# Alternative declaration
declare -a FRUITS
FRUITS[0]="Apple"
FRUITS[1]="Banana"
FRUITS[2]="Orange"

# Access elements
echo ${FRUITS[0]}        # Apple
echo ${FRUITS[1]}        # Banana

# All elements
echo ${FRUITS[@]}        # All elements
echo ${FRUITS[*]}        # All elements (different in quotes)

# Number of elements
echo ${#FRUITS[@]}

# Indices
echo ${!FRUITS[@]}

# Add element
FRUITS+=("Grape")
FRUITS[${#FRUITS[@]}]="Mango"

# Loop over array
for fruit in "${FRUITS[@]}"; do
    echo "Fruit: $fruit"
done

# Loop with index
for i in "${!FRUITS[@]}"; do
    echo "Index $i: ${FRUITS[$i]}"
done

# Slice array
echo ${FRUITS[@]:1:2}    # Elements from index 1, length 2

# Replace in array
FRUITS=("${FRUITS[@]/Apple/Pear}")

# Remove element
unset FRUITS[1]

# Remove all elements
unset FRUITS

# Array from string
IFS=',' read -ra ARRAY <<< "one,two,three"

# Array from command
FILES=($(ls))
```

### Associative Arrays

```bash
#!/bin/bash

# Declare associative array (Bash 4+)
declare -A PERSON

# Assign values
PERSON[name]="John"
PERSON[age]=30
PERSON[city]="New York"

# Access values
echo ${PERSON[name]}
echo ${PERSON[age]}

# All keys
echo ${!PERSON[@]}

# All values
echo ${PERSON[@]}

# Check if key exists
if [ ${PERSON[name]+_} ]; then
    echo "Key 'name' exists"
fi

# Loop over associative array
for key in "${!PERSON[@]}"; do
    echo "$key: ${PERSON[$key]}"
done

# Declare and initialize
declare -A CONFIG=(
    [host]="localhost"
    [port]="8080"
    [user]="admin"
)

# Remove key
unset PERSON[age]
```

---

## String Manipulation

### String Operations

```bash
#!/bin/bash

# Length
STR="Hello World"
echo ${#STR}             # 11

# Substring
echo ${STR:0:5}          # Hello
echo ${STR:6}            # World
echo ${STR: -5}          # World (space before - is important)

# Concatenation
FIRST="Hello"
SECOND="World"
FULL="$FIRST $SECOND"
echo $FULL

# Uppercase/Lowercase (Bash 4+)
echo ${STR,,}            # hello world
echo ${STR^^}            # HELLO WORLD
echo ${STR^}             # Hello world (first letter)
echo ${STR^^[hw]}        # HELLO WORLD (specific chars)

# Replace
echo ${STR/World/Universe}       # Hello Universe (first)
echo ${STR//o/0}                 # Hell0 W0rld (all)
echo ${STR/#Hello/Hi}            # Hi World (prefix)
echo ${STR/%World/Universe}      # Hello Universe (suffix)

# Remove pattern
FILE="document.tar.gz"
echo ${FILE%.gz}                 # document.tar (remove shortest suffix)
echo ${FILE%%.*}                 # document (remove longest suffix)
echo ${FILE#*.}                  # tar.gz (remove shortest prefix)
echo ${FILE##*.}                 # gz (remove longest prefix)

# Default values
echo ${VAR:-"default"}           # Use default if unset
echo ${VAR:="default"}           # Assign and use default
echo ${VAR:?"Error message"}     # Error if unset
echo ${VAR:+"alternative"}       # Use alternative if set

# Check if substring exists
if [[ "$STR" == *"World"* ]]; then
    echo "Contains 'World'"
fi

# Trim whitespace
STR="  hello world  "
STR="${STR#"${STR%%[![:space:]]*}"}"   # Remove leading
STR="${STR%"${STR##*[![:space:]]}"}"   # Remove trailing
```

### String Comparison

```bash
#!/bin/bash

STR1="hello"
STR2="world"

# Equality
if [ "$STR1" = "$STR2" ]; then
    echo "Equal"
fi

if [ "$STR1" != "$STR2" ]; then
    echo "Not equal"
fi

# Using [[ ]] (preferred)
if [[ "$STR1" == "$STR2" ]]; then
    echo "Equal"
fi

# Pattern matching
if [[ "$STR1" == h* ]]; then
    echo "Starts with 'h'"
fi

# Regex
if [[ "$STR1" =~ ^h[a-z]+$ ]]; then
    echo "Matches regex"
fi

# Empty/not empty
if [ -z "$STR1" ]; then
    echo "String is empty"
fi

if [ -n "$STR1" ]; then
    echo "String is not empty"
fi

# Lexicographic comparison
if [[ "$STR1" < "$STR2" ]]; then
    echo "$STR1 comes before $STR2"
fi
```

---

## Command Line Arguments

### Accessing Arguments

```bash
#!/bin/bash

# Script name
echo "Script: $0"

# Arguments
echo "First argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"

# All arguments
echo "All arguments: $@"
echo "All arguments (string): $*"

# Number of arguments
echo "Number of arguments: $#"

# Shift arguments
shift
echo "After shift, first argument: $1"

# Check if argument provided
if [ -z "$1" ]; then
    echo "No argument provided"
    exit 1
fi
```

### Getopts

```bash
#!/bin/bash

# Parse options
while getopts "a:b:c" opt; do
    case $opt in
        a)
            echo "Option a: $OPTARG"
            ;;
        b)
            echo "Option b: $OPTARG"
            ;;
        c)
            echo "Option c"
            ;;
        \?)
            echo "Invalid option: -$OPTARG"
            exit 1
            ;;
        :)
            echo "Option -$OPTARG requires an argument"
            exit 1
            ;;
    esac
done

# Shift processed options
shift $((OPTIND-1))

# Remaining arguments
echo "Remaining arguments: $@"

# Usage: ./script.sh -a value1 -b value2 -c arg1 arg2
```

### Long Options

```bash
#!/bin/bash

# Parse long options
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            echo "Usage: $0 [options]"
            exit 0
            ;;
        -v|--verbose)
            VERBOSE=true
            shift
            ;;
        -f|--file)
            FILE="$2"
            shift 2
            ;;
        -o|--output)
            OUTPUT="$2"
            shift 2
            ;;
        *)
            echo "Unknown option: $1"
            exit 1
            ;;
    esac
done

echo "Verbose: $VERBOSE"
echo "File: $FILE"
echo "Output: $OUTPUT"
```

---

## File Operations

### Reading Files

```bash
#!/bin/bash

# Read entire file
content=$(cat file.txt)

# Read line by line
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt

# Read with line numbers
while IFS= read -r line; do
    echo "$LINENO: $line"
done < file.txt

# Read into array
mapfile -t lines < file.txt
# or
readarray -t lines < file.txt

# Read specific lines
sed -n '5,10p' file.txt

# Read last N lines
tail -n 10 file.txt

# Read first N lines
head -n 10 file.txt
```

### Writing Files

```bash
#!/bin/bash

# Write to file (overwrite)
echo "Hello World" > file.txt

# Append to file
echo "New line" >> file.txt

# Write multiple lines
cat > file.txt << EOF
Line 1
Line 2
Line 3
EOF

# Write with variables
cat > config.txt << EOF
Host: $HOSTNAME
User: $USER
Date: $(date)
EOF

# Append multiple lines
cat >> file.txt << EOF
Additional line 1
Additional line 2
EOF
```

### File Operations

```bash
#!/bin/bash

# Check if file exists
if [ -f file.txt ]; then
    echo "File exists"
fi

# Create file
touch newfile.txt

# Copy file
cp source.txt destination.txt
cp -r sourcedir/ destdir/

# Move/rename file
mv oldname.txt newname.txt
mv file.txt /path/to/destination/

# Delete file
rm file.txt
rm -f file.txt           # Force
rm -rf directory/        # Recursive

# Create directory
mkdir newdir
mkdir -p path/to/newdir  # Create parents

# File info
stat file.txt
ls -l file.txt

# File size
size=$(stat -f%z file.txt)  # macOS
size=$(stat -c%s file.txt)  # Linux

# Modification time
mtime=$(stat -f%m file.txt)  # macOS
mtime=$(stat -c%Y file.txt)  # Linux

# Find files
find /path -name "*.txt"
find /path -type f -mtime -7
find /path -size +100M
```

---

## Process Management

### Running Processes

```bash
#!/bin/bash

# Run command
ls -l

# Run in background
sleep 60 &

# Get background PID
BGPID=$!
echo "Background PID: $BGPID"

# Wait for background process
wait $BGPID

# Run multiple in background
command1 &
command2 &
command3 &
wait  # Wait for all

# Disown process (continue after shell exit)
sleep 60 &
disown

# Job control
jobs                # List jobs
fg %1               # Bring job to foreground
bg %1               # Resume job in background
kill %1             # Kill job
```

### Exit Codes

```bash
#!/bin/bash

# Check exit code
ls /tmp
if [ $? -eq 0 ]; then
    echo "Command succeeded"
fi

# Set exit code
exit 0   # Success
exit 1   # General error
exit 2   # Misuse of shell command
exit 126 # Command cannot execute
exit 127 # Command not found
exit 130 # Script terminated by Ctrl+C

# Use exit code
command
if [ $? -ne 0 ]; then
    echo "Command failed"
    exit 1
fi

# Logical operators with exit codes
command1 && command2  # Run command2 if command1 succeeds
command1 || command2  # Run command2 if command1 fails
```

### Signal Handling

```bash
#!/bin/bash

# Trap signals
trap "echo 'Caught SIGINT'; exit" INT
trap "echo 'Caught SIGTERM'; exit" TERM

# Cleanup on exit
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
}
trap cleanup EXIT

# Ignore signal
trap '' INT  # Ignore Ctrl+C

# Reset to default
trap - INT

# Multiple signals
trap "echo 'Signal caught'" INT TERM HUP

# List traps
trap -p

# Common signals:
# INT  (2)  - Ctrl+C
# TERM (15) - Termination
# HUP  (1)  - Hangup
# QUIT (3)  - Quit
# KILL (9)  - Force kill (cannot be trapped)
```

---

## Regular Expressions

### Pattern Matching

```bash
#!/bin/bash

# Basic regex with =~
if [[ "hello123" =~ [0-9]+ ]]; then
    echo "Contains numbers"
fi

# Capture groups
if [[ "abc123def" =~ ([a-z]+)([0-9]+)([a-z]+) ]]; then
    echo "Full match: ${BASH_REMATCH[0]}"
    echo "Group 1: ${BASH_REMATCH[1]}"
    echo "Group 2: ${BASH_REMATCH[2]}"
    echo "Group 3: ${BASH_REMATCH[3]}"
fi

# Email validation
EMAIL="user@example.com"
if [[ "$EMAIL" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi

# IP address validation
IP="192.168.1.1"
if [[ "$IP" =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
    echo "Valid IP format"
fi

# Phone number
PHONE="123-456-7890"
if [[ "$PHONE" =~ ^[0-9]{3}-[0-9]{3}-[0-9]{4}$ ]]; then
    echo "Valid phone"
fi
```

### Grep with Regex

```bash
#!/bin/bash

# Basic grep
grep "pattern" file.txt

# Extended regex
grep -E "pattern1|pattern2" file.txt
egrep "pattern1|pattern2" file.txt

# Case insensitive
grep -i "pattern" file.txt

# Show line numbers
grep -n "pattern" file.txt

# Invert match
grep -v "pattern" file.txt

# Count matches
grep -c "pattern" file.txt

# Show only matching part
grep -o "pattern" file.txt

# Recursive
grep -r "pattern" /path/

# Context
grep -A 3 "pattern" file.txt  # 3 lines after
grep -B 3 "pattern" file.txt  # 3 lines before
grep -C 3 "pattern" file.txt  # 3 lines before and after
```

---

## Error Handling

### Error Checking

```bash
#!/bin/bash

# Exit on error
set -e

# Exit on undefined variable
set -u

# Exit on pipe failure
set -o pipefail

# Combine all
set -euo pipefail

# Check command success
if ! command; then
    echo "Command failed"
    exit 1
fi

# Or operator
command || { echo "Failed"; exit 1; }

# And operator
command && echo "Success"

# Check exit code
command
if [ $? -ne 0 ]; then
    echo "Command failed with code $?"
    exit 1
fi
```

### Error Messages

```bash
#!/bin/bash

# Write to stderr
echo "Error message" >&2

# Function for errors
error() {
    echo "ERROR: $1" >&2
    exit 1
}

error "Something went wrong"

# Warning function
warn() {
    echo "WARNING: $1" >&2
}

warn "This is a warning"

# Info function
info() {
    echo "INFO: $1"
}

info "This is information"
```

### Try-Catch Pattern

```bash
#!/bin/bash

# Try-catch pattern
{
    command1 &&
    command2 &&
    command3
} || {
    echo "One of the commands failed"
    exit 1
}

# With specific error handling
if ! command1; then
    echo "command1 failed"
    exit 1
fi

if ! command2; then
    echo "command2 failed"
    exit 2
fi
```

---

## Debugging

### Debug Modes

```bash
#!/bin/bash

# Print commands before execution
set -x

# Turn off debug
set +x

# Debug specific section
set -x
command1
command2
set +x

# Verbose mode
set -v

# Run script in debug mode
bash -x script.sh

# Conditional debugging
DEBUG=true
if [ "$DEBUG" = true ]; then
    set -x
fi
```

### Debug Functions

```bash
#!/bin/bash

# Debug function
debug() {
    if [ "$DEBUG" = true ]; then
        echo "DEBUG: $@" >&2
    fi
}

DEBUG=true
debug "This is a debug message"

# Stack trace
print_stack() {
    local i=0
    local FRAMES=${#BASH_SOURCE[@]}

    echo "Stack trace:" >&2
    for ((i=1; i<FRAMES; i++)); do
        echo "  $i: ${BASH_SOURCE[$i]}:${BASH_LINENO[$((i-1))]} ${FUNCNAME[$i]}" >&2
    done
}

# Assertion
assert() {
    if ! "$@"; then
        echo "Assertion failed: $@" >&2
        print_stack
        exit 1
    fi
}

assert [ -f /etc/passwd ]
```

---

## Best Practices

### Script Template

```bash
#!/bin/bash

#############################################
# Script: script_name.sh
# Description: What this script does
# Author: Your Name
# Date: 2024-01-01
#############################################

# Strict mode
set -euo pipefail

# Global variables
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"

# Functions
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS]

Description of what the script does.

OPTIONS:
    -h, --help      Show this help message
    -v, --verbose   Enable verbose mode
    -f, --file      Input file

EXAMPLES:
    $SCRIPT_NAME -f input.txt
    $SCRIPT_NAME --verbose

EOF
}

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error() {
    echo "ERROR: $*" >&2
    exit 1
}

cleanup() {
    # Cleanup code here
    log "Cleaning up..."
}

main() {
    log "Script started"

    # Main script logic here

    log "Script completed"
}

# Trap errors
trap cleanup EXIT
trap 'error "Script interrupted"' INT TERM

# Parse arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            usage
            exit 0
            ;;
        -v|--verbose)
            set -x
            shift
            ;;
        -f|--file)
            FILE="$2"
            shift 2
            ;;
        *)
            error "Unknown option: $1"
            ;;
    esac
done

# Run main function
main "$@"
```

### Best Practices Checklist

```bash
#!/bin/bash

# 1. Always use shebang
#!/bin/bash

# 2. Use strict mode
set -euo pipefail

# 3. Quote variables
echo "$VAR"
echo "${VAR}"

# 4. Use [[ ]] instead of [ ]
if [[ "$VAR" == "value" ]]; then

# 5. Use functions
my_function() {
    local var="local value"
}

# 6. Check if file exists before operations
if [[ -f "$FILE" ]]; then
    cat "$FILE"
fi

# 7. Use meaningful variable names
USER_COUNT=$(who | wc -l)  # Good
c=$(who | wc -l)           # Bad

# 8. Use readonly for constants
readonly MAX_RETRIES=3

# 9. Validate input
if [[ -z "$1" ]]; then
    echo "Error: No argument provided"
    exit 1
fi

# 10. Use arrays for lists
FILES=("file1.txt" "file2.txt" "file3.txt")

# 11. Handle errors
command || { echo "Command failed"; exit 1; }

# 12. Use subshells for grouping
(cd /tmp && ls)  # Directory change doesn't affect parent

# 13. Use process substitution
while read line; do
    echo "$line"
done < <(command)

# 14. Proper exit codes
exit 0  # Success
exit 1  # Error

# 15. Document your code
# This function does something important
my_function() {
    # Implementation
}
```

---

## Advanced Topics

### Process Substitution

```bash
#!/bin/bash

# Compare output of two commands
diff <(ls dir1) <(ls dir2)

# Feed command output as file
while read line; do
    echo "$line"
done < <(command)

# Multiple process substitutions
paste <(cut -d: -f1 /etc/passwd) <(cut -d: -f7 /etc/passwd)
```

### Command Substitution

```bash
#!/bin/bash

# Modern syntax (preferred)
current_date=$(date)
files=$(ls -l)

# Old syntax (avoid)
current_date=`date`

# Nested substitution
result=$(echo $(date))
```

### Parallel Execution

```bash
#!/bin/bash

# Run commands in parallel
command1 &
command2 &
command3 &
wait

# Parallel processing with xargs
cat files.txt | xargs -P 4 -I {} process_file {}

# GNU parallel (install first)
parallel command ::: item1 item2 item3
find . -name "*.txt" | parallel process_file

# Background jobs with control
for i in {1..10}; do
    (
        # Process in subshell
        process_item $i
    ) &

    # Limit concurrent jobs
    if (( $(jobs -r | wc -l) >= 4 )); then
        wait -n  # Wait for any job to finish
    fi
done
wait  # Wait for all remaining jobs
```

### Advanced Parameter Expansion

```bash
#!/bin/bash

# Indirect reference
var="name"
name="John"
echo ${!var}  # John

# Array manipulation
arr=(a b c d e)
echo ${arr[@]:1:3}        # b c d (slice)
echo ${arr[@]/c/C}        # a b C d e (replace)
echo ${#arr[@]}           # 5 (length)

# String manipulation
path="/home/user/document.txt"
echo ${path##*/}          # document.txt (basename)
echo ${path%/*}           # /home/user (dirname)
echo ${path%.txt}         # /home/user/document
echo ${path##*.}          # txt (extension)

# Case conversion (Bash 4+)
echo ${path^^}            # Uppercase
echo ${path,,}            # Lowercase
echo ${path^}             # Capitalize first
```

### Here Documents and Strings

```bash
#!/bin/bash

# Here document
cat << EOF
Line 1
Line 2
Variable: $VAR
EOF

# Here document without variable expansion
cat << 'EOF'
Line 1
Line 2
Variable: $VAR (not expanded)
EOF

# Here document with indentation
cat <<- EOF
	This is indented
	Tabs are removed
EOF

# Here string
cat <<< "Single line"
grep "pattern" <<< "$variable"

# Multi-line variable
read -r -d '' VAR << EOF
Line 1
Line 2
Line 3
EOF
```

### Co-processes

```bash
#!/bin/bash

# Start co-process
coproc MYCOPROC {
    while read line; do
        echo "Received: $line"
    done
}

# Write to co-process
echo "Hello" >&${MYCOPROC[1]}

# Read from co-process
read -u ${MYCOPROC[0]} response

# Close co-process
exec {MYCOPROC[1]}>&-
wait $MYCOPROC_PID
```

---

## Practical Examples

### Backup Script

```bash
#!/bin/bash

# Backup script
set -euo pipefail

readonly BACKUP_SOURCE="/var/www"
readonly BACKUP_DEST="/backup"
readonly DATE=$(date +%Y%m%d_%H%M%S)
readonly BACKUP_FILE="backup_${DATE}.tar.gz"
readonly LOG_FILE="/var/log/backup.log"
readonly RETENTION_DAYS=7

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error() {
    log "ERROR: $*"
    exit 1
}

# Create backup directory
mkdir -p "$BACKUP_DEST" || error "Cannot create backup directory"

# Create backup
log "Starting backup of $BACKUP_SOURCE"
tar -czf "$BACKUP_DEST/$BACKUP_FILE" "$BACKUP_SOURCE" 2>&1 | tee -a "$LOG_FILE" || error "Backup failed"

# Verify backup
if [[ -f "$BACKUP_DEST/$BACKUP_FILE" ]]; then
    size=$(du -h "$BACKUP_DEST/$BACKUP_FILE" | cut -f1)
    log "Backup created successfully: $BACKUP_FILE ($size)"
else
    error "Backup file not found"
fi

# Delete old backups
log "Removing backups older than $RETENTION_DAYS days"
find "$BACKUP_DEST" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete

log "Backup completed successfully"
```

### System Monitor

```bash
#!/bin/bash

# System monitoring script
set -euo pipefail

readonly ALERT_EMAIL="admin@example.com"
readonly CPU_THRESHOLD=80
readonly MEM_THRESHOLD=80
readonly DISK_THRESHOLD=90

check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    if (( $(echo "$cpu_usage > $CPU_THRESHOLD" | bc -l) )); then
        echo "ALERT: CPU usage is ${cpu_usage}%"
        return 1
    fi
    return 0
}

check_memory() {
    local mem_usage=$(free | grep Mem | awk '{printf("%.0f", $3/$2 * 100)}')
    if (( mem_usage > MEM_THRESHOLD )); then
        echo "ALERT: Memory usage is ${mem_usage}%"
        return 1
    fi
    return 0
}

check_disk() {
    while read line; do
        local usage=$(echo "$line" | awk '{print $5}' | sed 's/%//')
        local mount=$(echo "$line" | awk '{print $6}')
        if (( usage > DISK_THRESHOLD )); then
            echo "ALERT: Disk usage on $mount is ${usage}%"
            return 1
        fi
    done < <(df -h | grep '^/dev/')
    return 0
}

main() {
    local alerts=()

    check_cpu || alerts+=("CPU")
    check_memory || alerts+=("Memory")
    check_disk || alerts+=("Disk")

    if [ ${#alerts[@]} -gt 0 ]; then
        echo "Alerts: ${alerts[*]}" | mail -s "System Alert" "$ALERT_EMAIL"
    fi
}

main
```

### Deployment Script

```bash
#!/bin/bash

# Deployment script
set -euo pipefail

readonly APP_NAME="myapp"
readonly DEPLOY_DIR="/var/www/$APP_NAME"
readonly BACKUP_DIR="/backup/$APP_NAME"
readonly GIT_REPO="https://github.com/user/repo.git"
readonly BRANCH="main"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

error() {
    log "ERROR: $*"
    exit 1
}

backup() {
    log "Creating backup..."
    local backup_name="backup_$(date +%Y%m%d_%H%M%S)"
    mkdir -p "$BACKUP_DIR"
    cp -r "$DEPLOY_DIR" "$BACKUP_DIR/$backup_name" || error "Backup failed"
    log "Backup created: $backup_name"
}

deploy() {
    log "Starting deployment..."

    # Pull latest code
    cd "$DEPLOY_DIR" || error "Cannot access deploy directory"
    git fetch origin || error "Git fetch failed"
    git checkout "$BRANCH" || error "Git checkout failed"
    git pull origin "$BRANCH" || error "Git pull failed"

    # Install dependencies
    if [[ -f "package.json" ]]; then
        npm install || error "NPM install failed"
    fi

    # Build application
    if [[ -f "package.json" ]]; then
        npm run build || error "Build failed"
    fi

    # Restart application
    sudo systemctl restart "$APP_NAME" || error "Service restart failed"

    log "Deployment completed successfully"
}

rollback() {
    log "Rolling back to previous version..."
    local latest_backup=$(ls -t "$BACKUP_DIR" | head -1)
    if [[ -z "$latest_backup" ]]; then
        error "No backup found for rollback"
    fi

    rm -rf "$DEPLOY_DIR"
    cp -r "$BACKUP_DIR/$latest_backup" "$DEPLOY_DIR"
    sudo systemctl restart "$APP_NAME"
    log "Rollback completed"
}

main() {
    case "${1:-deploy}" in
        deploy)
            backup
            deploy || {
                log "Deployment failed, rolling back..."
                rollback
                exit 1
            }
            ;;
        rollback)
            rollback
            ;;
        *)
            echo "Usage: $0 {deploy|rollback}"
            exit 1
            ;;
    esac
}

main "$@"
```

### Log Analyzer

```bash
#!/bin/bash

# Log analyzer
set -euo pipefail

readonly LOG_FILE="${1:-/var/log/syslog}"

# Count log levels
count_levels() {
    echo "=== Log Level Summary ==="
    grep -oE "(ERROR|WARNING|INFO)" "$LOG_FILE" | sort | uniq -c | sort -nr
}

# Top IPs
top_ips() {
    echo "=== Top 10 IP Addresses ==="
    grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" "$LOG_FILE" | sort | uniq -c | sort -nr | head -10
}

# Errors in last hour
recent_errors() {
    echo "=== Errors in Last Hour ==="
    local cutoff=$(date -d '1 hour ago' '+%Y-%m-%d %H:%M:%S')
    awk -v cutoff="$cutoff" '$0 > cutoff' "$LOG_FILE" | grep -i error
}

# Generate report
generate_report() {
    {
        echo "Log Analysis Report"
        echo "Generated: $(date)"
        echo "Log File: $LOG_FILE"
        echo ""
        count_levels
        echo ""
        top_ips
        echo ""
        recent_errors
    } > log_report.txt

    echo "Report generated: log_report.txt"
}

generate_report
```

---

## Conclusion

This comprehensive Bash scripting guide covers everything from basics to advanced topics. Key takeaways:

1. **Variables**: Understanding variable types and expansion
2. **Control Flow**: if/else, case, loops
3. **Functions**: Reusable code blocks with parameters and return values
4. **Arrays**: Both indexed and associative arrays
5. **String Manipulation**: Powerful built-in string operations
6. **Error Handling**: Proper error checking and debugging
7. **Best Practices**: Writing clean, maintainable scripts

### Additional Resources

- [Bash Manual](https://www.gnu.org/software/bash/manual/)
- [Advanced Bash Scripting Guide](https://tldp.org/LDP/abs/html/)
- [ShellCheck](https://www.shellcheck.net/) - Script analysis tool
- [Bash FAQ](http://mywiki.wooledge.org/BashFAQ)
- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)

### Tools for Bash Development

```bash
# ShellCheck - syntax and best practice checker
shellcheck script.sh

# Bashate - style checker
bashate script.sh

# Bash debugging
bash -x script.sh
bash -n script.sh  # Syntax check only
```

---

**Remember**: Always test scripts thoroughly, use version control, and follow security best practices when handling sensitive data!
