# Shell Scripts

> A comprehensive guide to Shell Scripting — from basics to real-world automation.  
> Based on the [ProgrammingKnowledge Shell Scripting Tutorial for Beginners](https://www.youtube.com/playlist?list=PLS1QulWo1RIaAsfcLW-Jk-Cx3JGRP8tjh) and extended with best practices, advanced patterns, and production-ready examples.

---

## Table of Contents

1. [What is a Shell?](#1-what-is-a-shell)
2. [What is a Shell Script?](#2-what-is-a-shell-script)
3. [Setting Up Your Environment](#3-setting-up-your-environment)
4. [Your First Script](#4-your-first-script)
5. [Variables](#5-variables)
6. [Reading User Input](#6-reading-user-input)
7. [Command Substitution](#7-command-substitution)
8. [Passing Arguments to Scripts](#8-passing-arguments-to-scripts)
9. [Operators](#9-operators)
10. [Conditional Statements](#10-conditional-statements)
11. [Case Statements](#11-case-statements)
12. [Arrays](#12-arrays)
13. [Loops](#13-loops)
14. [Functions](#14-functions)
15. [Break and Continue](#15-break-and-continue)
16. [The `test` Command](#16-the-test-command)
17. [String Manipulation](#17-string-manipulation)
18. [File Operations](#18-file-operations)
19. [Process Management](#19-process-management)
20. [Error Handling & Debugging](#20-error-handling--debugging)
21. [Real-World Examples](#21-real-world-examples)
22. [Best Practices](#22-best-practices)
23. [Resources](#23-resources)

---

## 1. What is a Shell?

A **shell** is a command-line interpreter that takes commands from the keyboard and translates them into instructions for the computer. It sits between you and the operating system kernel.

### Common Shells

| Shell | Description | Path |
|-------|-------------|------|
| **Bash** | Bourne Again SHell — the default on most Linux distros | `/bin/bash` |
| **Sh** | Bourne Shell — the original Unix shell | `/bin/sh` |
| **Zsh** | Z Shell — powerful with plugins (macOS default since Catalina) | `/bin/zsh` |
| **Fish** | Friendly Interactive SHell — user-friendly | `/usr/bin/fish` |
| **Ksh** | Korn Shell — backward-compatible with sh | `/bin/ksh` |

### Checking Your Current Shell

```bash
echo $SHELL          # Print your login shell
echo $0              # Print the current shell
bash --version       # Check Bash version
```

---

## 2. What is a Shell Script?

A **shell script** is a text file containing multiple shell commands that can be executed together. Instead of typing commands one by one, you write them in a file and run it.

### Why Use Shell Scripts?

- **Automation** — Schedule repetitive tasks (backups, deployments, monitoring)
- **Portability** — Run the same script across different Unix-like systems
- **Efficiency** — Combine multiple commands into a single execution
- **System Administration** — Manage users, services, files, and processes
- **DevOps** — CI/CD pipelines, infrastructure provisioning, container orchestration

---

## 3. Setting Up Your Environment

### Text Editors

You can write shell scripts in any text editor. Popular choices:

- **Visual Studio Code** — With the *Bash IDE* extension
- **Vim / Neovim** — Built into every Unix system
- **Nano** — Beginner-friendly terminal editor
- **Sublime Text / Atom** — GUI editors with syntax highlighting

### Installing VS Code on Linux

```bash
# Ubuntu / Debian
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
rm -f packages.microsoft.gpg
sudo apt update
sudo apt install code

# Fedora / RHEL
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
sudo dnf check-update
sudo dnf install code
```

---

## 4. Your First Script

### Script Structure

Every shell script follows this structure:

```bash
#!/bin/bash
# This is a comment

echo "Hello, World!"
```

### The Shebang (`#!`)

The **shebang** (`#!`) on the very first line tells the system which interpreter to use:

```bash
#!/bin/bash      # Use Bash
#!/bin/sh        # Use POSIX sh
#!/usr/bin/env bash  # Portable way to find bash
#!/usr/bin/python3   # Can even run Python scripts
```

> **Best Practice:** Use `#!/usr/bin/env bash` for portability across systems where bash may be in different locations.

### Creating and Running a Script

```bash
# 1. Create the file
touch myscript.sh

# 2. Open in your editor
nano myscript.sh

# 3. Add content
#!/bin/bash
echo "Hello, World!"

# 4. Save and exit (Ctrl+O, Enter, Ctrl+X in nano)

# 5. Make it executable
chmod +x myscript.sh

# 6. Run it
./myscript.sh
```

### Running Scripts Without Execute Permission

```bash
bash myscript.sh      # Pass to bash interpreter
sh myscript.sh        # Pass to sh interpreter
source myscript.sh    # Run in current shell (affects environment)
. myscript.sh         # Same as source
```

> **Note:** `source` and `.` run the script in the current shell, so variables and changes persist after execution.

---

## 5. Variables

### Defining Variables

```bash
#!/bin/bash

# Variable assignment (NO spaces around =)
name="John"
age=25
pi=3.14159

# Accessing variables (use $ prefix)
echo "Name: $name"
echo "Age: $age"

# Alternative syntax with braces (recommended)
echo "Hello, ${name}!"
```

### Variable Naming Rules

- Must start with a letter or underscore (`_`)
- Can contain letters, numbers, and underscores
- Case-sensitive (`Name` ≠ `name`)
- Cannot contain spaces or special characters
- By convention: `UPPERCASE` for constants, `lowercase` for variables

### Variable Types

```bash
#!/bin/bash

# String (default)
str="Hello World"

# Integer
num=42

# Float (stored as string — use bc for math)
float="3.14"

# Empty variable
empty=""

# Unset a variable
unset str

# Check if variable exists
if [ -z "$empty" ]; then
    echo "Variable is empty or unset"
fi
```

### Environment Variables

```bash
#!/bin/bash

# View all environment variables
env
printenv

# Common environment variables
echo "Home: $HOME"
echo "User: $USER"
echo "Shell: $SHELL"
echo "Path: $PATH"
echo "Current Directory: $PWD"
echo "Previous Directory: $OLDPWD"
echo "Process ID: $$"
echo "Random Number: $RANDOM"
echo "Exit Status of Last Command: $?"

# Create an environment variable
export MY_VAR="Hello"
# Now it's available to child processes
```

### Read-Only Variables (Constants)

```bash
#!/bin/bash

readonly PI=3.14159
readonly MAX_RETRIES=5

# This will fail:
# PI=3.14  # Error: PI: is read only
```

### Local vs Global Variables

```bash
#!/bin/bash

global_var="I am global"

my_function() {
    local local_var="I am local"
    echo "$local_var"      # Works
    echo "$global_var"     # Works
}

my_function
echo "$global_var"         # Works
echo "$local_var"          # Empty — local_var doesn't exist here
```

---

## 6. Reading User Input

### Basic Input with `read`

```bash
#!/bin/bash

# Simple prompt
echo "Enter your name:"
read name
echo "Hello, $name!"

# Inline prompt
read -p "Enter your age: " age
echo "You are $age years old."
```

### Reading Multiple Values

```bash
#!/bin/bash

read -p "Enter first and last name: " first last
echo "First: $first"
echo "Last: $last"
```

### Reading with Timeout

```bash
#!/bin/bash

# Wait 5 seconds for input
read -t 5 -p "Enter something (5s timeout): " input

if [ -z "$input" ]; then
    echo -e "
Timeout! No input received."
else
    echo "You entered: $input"
fi
```

### Reading Password (Hidden Input)

```bash
#!/bin/bash

read -sp "Enter password: " password
echo -e "
Password received (length: ${#password})"
```

### Reading into an Array

```bash
#!/bin/bash

echo "Enter multiple values separated by space:"
read -a array
echo "First: ${array[0]}"
echo "Second: ${array[1]}"
echo "All: ${array[@]}"
```

### Reading a Line from a File

```bash
#!/bin/bash

while read line; do
    echo "Line: $line"
done < file.txt
```

---

## 7. Command Substitution

Command substitution allows you to capture the output of a command and store it in a variable.

### Syntax

```bash
#!/bin/bash

# Modern syntax (recommended)
current_date=$(date)
echo "Today is: $current_date"

# Legacy backtick syntax (still works but less preferred)
current_dir=`pwd`
echo "Current directory: $current_dir"

# Command substitution in strings
echo "Today is $(date +%Y-%m-%d)"

# Nested command substitution
echo "Files in home: $(ls $(echo $HOME) | wc -l)"
```

### Practical Examples

```bash
#!/bin/bash

# Get system info
hostname=$(hostname)
kernel=$(uname -r)
uptime_info=$(uptime -p)

echo "Host: $hostname"
echo "Kernel: $kernel"
echo "Uptime: $uptime_info"

# Count files
file_count=$(ls -1 | wc -l)
echo "Files in current directory: $file_count"

# Get IP address
ip_address=$(ip addr show eth0 | grep "inet " | awk '{print $2}' | cut -d/ -f1)
echo "IP Address: $ip_address"
```

---

## 8. Passing Arguments to Scripts

### Positional Parameters

```bash
#!/bin/bash

# $0 = script name
# $1 = first argument
# $2 = second argument
# $3 = third argument
# ...
# $9 = ninth argument
# ${10} = tenth argument (must use braces)
# $# = total number of arguments
# $@ = all arguments as separate strings
# $* = all arguments as a single string
# $$ = process ID of the script

# script.sh arg1 arg2 arg3

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"
echo "Total arguments: $#"
echo "All arguments (\$@): $@"
echo "All arguments (\$*): $*"
```

### Checking Arguments

```bash
#!/bin/bash

# Check if arguments were provided
if [ $# -eq 0 ]; then
    echo "Usage: $0 <arg1> [arg2]"
    exit 1
fi

# Check minimum arguments
if [ $# -lt 2 ]; then
    echo "Error: At least 2 arguments required"
    exit 1
fi

# Access all arguments in a loop
for arg in "$@"; do
    echo "Argument: $arg"
done
```

### Shift Command

```bash
#!/bin/bash

# shift moves all positional parameters left by N positions
# $2 becomes $1, $3 becomes $2, etc.

while [ $# -gt 0 ]; do
    echo "Processing: $1"
    shift
done
```

### Getopts (Parsing Options)

```bash
#!/bin/bash

# Parse command-line options
while getopts ":a:b:h" opt; do
    case $opt in
        a)
            echo "Option -a with value: $OPTARG"
            ;;
        b)
            echo "Option -b with value: $OPTARG"
            ;;
        h)
            echo "Usage: $0 [-a value] [-b value] [-h]"
            exit 0
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
```

---

## 9. Operators

### Arithmetic Operators

```bash
#!/bin/bash

a=10
b=3

# Arithmetic expansion $(( ))
echo "Addition: $((a + b))"        # 13
echo "Subtraction: $((a - b))"     # 7
echo "Multiplication: $((a * b))"  # 30
echo "Division: $((a / b))"        # 3
echo "Modulo: $((a % b))"          # 1
echo "Exponentiation: $((a ** 2))" # 100

# Increment / Decrement
echo "Pre-increment: $((++a))"     # 11
echo "Post-increment: $((a++))"    # 11 (then a becomes 12)

# Using expr (legacy)
result=$(expr $a + $b)

# Using let
let "c = a + b"
let "a += 5"    # Compound assignment
```

### Relational Operators

```bash
#!/bin/bash

a=10
b=20

# Integer comparison
[ $a -eq $b ]   # Equal
[ $a -ne $b ]   # Not equal
[ $a -gt $b ]   # Greater than
[ $a -lt $b ]   # Less than
[ $a -ge $b ]   # Greater than or equal
[ $a -le $b ]   # Less than or equal

# Modern arithmetic comparison (bash 4+)
(( a == b ))
(( a != b ))
(( a > b ))
(( a < b ))
(( a >= b ))
(( a <= b ))
```

### String Operators

```bash
#!/bin/bash

str1="hello"
str2="world"
str3=""

# String comparison
[ "$str1" = "$str2" ]    # Equal
[ "$str1" != "$str2" ]   # Not equal
[ -z "$str3" ]           # Length is zero
[ -n "$str1" ]           # Length is non-zero
[ "$str1" ]              # Non-empty (same as -n)

# Lexicographic comparison
[[ "$str1" < "$str2" ]]   # Alphabetical order
[[ "$str1" > "$str2" ]]
```

### File Test Operators

```bash
#!/bin/bash

file="/etc/passwd"
dir="/tmp"

[ -e "$file" ]    # File exists
[ -f "$file" ]    # Is a regular file
[ -d "$dir" ]     # Is a directory
[ -r "$file" ]    # Is readable
[ -w "$file" ]    # Is writable
[ -x "$file" ]    # Is executable
[ -s "$file" ]    # Size > 0 (not empty)
[ -L "$file" ]    # Is a symbolic link
[ -b "$file" ]    # Is a block device
[ -c "$file" ]    # Is a character device
[ -p "$file" ]    # Is a named pipe
[ -S "$file" ]    # Is a socket
[ -u "$file" ]    # SUID bit set
[ -g "$file" ]    # SGID bit set
[ -k "$file" ]    # Sticky bit set

# File comparison
[ file1 -nt file2 ]  # file1 is newer than file2
[ file1 -ot file2 ]  # file1 is older than file2
[ file1 -ef file2 ]  # Same file (hard link)
```

### Logical Operators

```bash
#!/bin/bash

a=10
b=20

# AND
[ $a -lt 15 ] && [ $b -gt 15 ]
[[ $a -lt 15 && $b -gt 15 ]]

# OR
[ $a -lt 5 ] || [ $b -gt 15 ]
[[ $a -lt 5 || $b -gt 15 ]]

# NOT
[ ! -f "/nonexistent" ]

# Combined
if [ -f "file.txt" ] && [ -r "file.txt" ]; then
    echo "File exists and is readable"
fi
```

### Bitwise Operators

```bash
#!/bin/bash

a=5    # 101 in binary
b=3    # 011 in binary

echo "AND: $((a & b))"      # 1 (001)
echo "OR: $((a | b))"       # 7 (111)
echo "XOR: $((a ^ b))"      # 6 (110)
echo "NOT: $((~a))"         # -6 (two's complement)
echo "Left Shift: $((a << 1))"  # 10 (1010)
echo "Right Shift: $((a >> 1))" # 2 (10)
```

---

## 10. Conditional Statements

### If-Else

```bash
#!/bin/bash

number=10

if [ $number -gt 0 ]; then
    echo "Positive number"
elif [ $number -lt 0 ]; then
    echo "Negative number"
else
    echo "Zero"
fi
```

### If with String Comparison

```bash
#!/bin/bash

read -p "Enter username: " username

if [ "$username" = "admin" ]; then
    echo "Welcome, administrator!"
elif [ "$username" = "guest" ]; then
    echo "Welcome, guest!"
else
    echo "Unknown user"
fi
```

### If with File Tests

```bash
#!/bin/bash

filename="data.txt"

if [ -f "$filename" ]; then
    if [ -r "$filename" ] && [ -w "$filename" ]; then
        echo "File exists and is readable/writable"
    else
        echo "File exists but has limited permissions"
    fi
else
    echo "File does not exist"
fi
```

### One-Line If (Ternary Style)

```bash
#!/bin/bash

age=20
[ $age -ge 18 ] && echo "Adult" || echo "Minor"

# Or using parameter expansion
status=$([ $age -ge 18 ] && echo "Adult" || echo "Minor")
```

### Nested If

```bash
#!/bin/bash

if [ -d "/var/log" ]; then
    if [ -r "/var/log/syslog" ]; then
        echo "Can read syslog"
        if [ -s "/var/log/syslog" ]; then
            echo "Syslog has content"
        fi
    fi
fi
```

---

## 11. Case Statements

The `case` statement is a cleaner alternative to multiple `elif` conditions.

### Basic Syntax

```bash
#!/bin/bash

read -p "Enter a fruit (apple/banana/orange): " fruit

case $fruit in
    apple)
        echo "You selected Apple. Color: Red/Green"
        ;;
    banana)
        echo "You selected Banana. Color: Yellow"
        ;;
    orange)
        echo "You selected Orange. Color: Orange"
        ;;
    *)
        echo "Unknown fruit"
        ;;
esac
```

### Multiple Patterns

```bash
#!/bin/bash

read -p "Enter a day: " day

case $day in
    [Mm]onday|[Tt]uesday|[Ww]ednesday|[Tt]hursday|[Ff]riday)
        echo "Weekday"
        ;;
    [Ss]aturday|[Ss]unday)
        echo "Weekend"
        ;;
    *)
        echo "Invalid day"
        ;;
esac
```

### Pattern Matching with Globs

```bash
#!/bin/bash

filename="document.pdf"

case $filename in
    *.txt)
        echo "Text file"
        ;;
    *.pdf)
        echo "PDF document"
        ;;
    *.jpg|*.png|*.gif)
        echo "Image file"
        ;;
    *)
        echo "Unknown file type"
        ;;
esac
```

### Using Case for Menu Systems

```bash
#!/bin/bash

while true; do
    echo "1) Show date"
    echo "2) Show users"
    echo "3) Show uptime"
    echo "4) Exit"
    read -p "Choose an option: " choice

    case $choice in
        1) date ;;
        2) who ;;
        3) uptime ;;
        4) echo "Goodbye!"; break ;;
        *) echo "Invalid option" ;;
    esac
done
```

---

## 12. Arrays

### Declaring Arrays

```bash
#!/bin/bash

# Indexed arrays
fruits=("apple" "banana" "cherry" "date")
numbers=(1 2 3 4 5)

# Mixed types (all stored as strings)
mixed=(1 "hello" 3.14 "world")

# Declare explicitly
declare -a names=("Alice" "Bob" "Charlie")

# Sparse array
sparse=([0]="first" [5]="fifth" [10]="tenth")
```

### Accessing Array Elements

```bash
#!/bin/bash

fruits=("apple" "banana" "cherry")

# Single element
echo "First: ${fruits[0]}"
echo "Second: ${fruits[1]}"

# All elements
echo "All: ${fruits[@]}"
echo "All: ${fruits[*]}"

# Number of elements
echo "Count: ${#fruits[@]}"

# Length of specific element
echo "Length of first: ${#fruits[0]}"

# All indices
echo "Indices: ${!fruits[@]}"

# Last element
echo "Last: ${fruits[-1]}"
```

### Array Operations

```bash
#!/bin/bash

fruits=("apple" "banana")

# Append
fruits+=("cherry" "date")

# Insert at index
fruits[0]="apricot"

# Remove element
unset fruits[1]

# Remove entire array
unset fruits

# Slice (elements 1 to 3)
slice=("${fruits[@]:1:3}")

# Iterate
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Iterate with index
for i in "${!fruits[@]}"; do
    echo "Index $i: ${fruits[$i]}"
done
```

### Associative Arrays (Bash 4.0+)

```bash
#!/bin/bash

# Must declare first
declare -A user

user[name]="John"
user[age]=30
user[city]="New York"

# Access
echo "Name: ${user[name]}"

# All keys
echo "Keys: ${!user[@]}"

# All values
echo "Values: ${user[@]}"

# Iterate
for key in "${!user[@]}"; do
    echo "$key: ${user[$key]}"
done
```

---

## 13. Loops

### For Loop (List)

```bash
#!/bin/bash

# Iterate over a list
for item in apple banana cherry; do
    echo "Fruit: $item"
done

# Iterate over array
fruits=("apple" "banana" "cherry")
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Iterate over files
for file in *.txt; do
    echo "Processing: $file"
done

# Iterate over command output
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "User: $user"
done

# C-style for loop
for ((i=0; i<5; i++)); do
    echo "Index: $i"
done

# Range with brace expansion
for i in {1..10}; do
    echo "Number: $i"
done

# Range with step
for i in {0..100..10}; do
    echo "$i"
done
```

### While Loop

```bash
#!/bin/bash

# Basic while loop
counter=0
while [ $counter -lt 5 ]; do
    echo "Counter: $counter"
    ((counter++))
done

# Read file line by line
while read line; do
    echo "Line: $line"
done < file.txt

# Infinite loop with condition
while true; do
    read -p "Enter 'quit' to exit: " input
    if [ "$input" = "quit" ]; then
        break
    fi
    echo "You entered: $input"
done

# While with command
while ping -c 1 google.com &> /dev/null; do
    echo "Google is reachable"
    sleep 5
done
```

### Until Loop

```bash
#!/bin/bash

# Runs until condition becomes true
counter=0
until [ $counter -ge 5 ]; do
    echo "Counter: $counter"
    ((counter++))
done

# Wait for a file to exist
until [ -f "/tmp/ready.flag" ]; do
    echo "Waiting for flag file..."
    sleep 1
done
echo "Flag file found!"
```

### Select Loop (Menu)

```bash
#!/bin/bash

PS3="Choose your favorite color: "
select color in Red Green Blue Yellow Quit; do
    case $color in
        Red) echo "You chose Red (passion)" ;;
        Green) echo "You chose Green (nature)" ;;
        Blue) echo "You chose Blue (calm)" ;;
        Yellow) echo "You chose Yellow (energy)" ;;
        Quit) echo "Goodbye!"; break ;;
        *) echo "Invalid option" ;;
    esac
done
```

### Nested Loops

```bash
#!/bin/bash

# Multiplication table
for i in {1..5}; do
    for j in {1..5}; do
        printf "%4d" $((i * j))
    done
    echo
done
```

---

## 14. Functions

### Defining Functions

```bash
#!/bin/bash

# Syntax 1
function greet() {
    echo "Hello, $1!"
}

# Syntax 2 (no 'function' keyword)
farewell() {
    echo "Goodbye, $1!"
}

# Call functions
greet "Alice"
farewell "Bob"
```

### Function with Return Value

```bash
#!/bin/bash

# Functions can only return integers (0-255)
# For strings, use echo and command substitution

add() {
    local a=$1
    local b=$2
    echo $((a + b))
}

result=$(add 5 3)
echo "5 + 3 = $result"

# Return exit status
is_even() {
    local num=$1
    if ((num % 2 == 0)); then
        return 0  # Success (true)
    else
        return 1  # Failure (false)
    fi
}

if is_even 4; then
    echo "4 is even"
else
    echo "4 is odd"
fi
```

### Function with Local Variables

```bash
#!/bin/bash

global_var="I am global"

my_function() {
    local local_var="I am local"
    echo "Inside function:"
    echo "  local_var = $local_var"
    echo "  global_var = $global_var"
}

my_function

echo "Outside function:"
echo "  local_var = ${local_var:-'(empty)' }"
echo "  global_var = $global_var"
```

### Nested Functions

```bash
#!/bin/bash

outer() {
    echo "Outer function"

    inner() {
        echo "Inner function"
    }

    inner  # Call inner from outer
}

outer
# inner  # This would fail — inner is not visible here
```

### Function with Default Parameters

```bash
#!/bin/bash

greet() {
    local name=${1:-"World"}
    local greeting=${2:-"Hello"}
    echo "$greeting, $name!"
}

greet                    # Hello, World!
greet "Alice"            # Hello, Alice!
greet "Alice" "Hi"       # Hi, Alice!
```

### Recursive Functions

```bash
#!/bin/bash

factorial() {
    local n=$1
    if [ $n -le 1 ]; then
        echo 1
    else
        local prev=$(factorial $((n - 1)))
        echo $((n * prev))
    fi
}

echo "5! = $(factorial 5)"
```

---

## 15. Break and Continue

### Break

```bash
#!/bin/bash

# Exit the loop entirely
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        echo "Breaking at $i"
        break
    fi
    echo "Number: $i"
done

# Break from nested loops
for i in {1..3}; do
    for j in {1..3}; do
        if [ $j -eq 2 ]; then
            break 2  # Break out of both loops
        fi
        echo "i=$i, j=$j"
    done
done
```

### Continue

```bash
#!/bin/bash

# Skip to next iteration
for i in {1..10}; do
    if [ $((i % 2)) -eq 0 ]; then
        continue  # Skip even numbers
    fi
    echo "Odd number: $i"
done

# Continue in while loop
i=0
while [ $i -lt 10 ]; do
    ((i++))
    if [ $((i % 2)) -eq 0 ]; then
        continue
    fi
    echo "Odd: $i"
done
```

---

## 16. The `test` Command

The `test` command evaluates expressions and returns exit status 0 (true) or 1 (false).

### Syntax

```bash
# These are equivalent:
test -f file.txt
[ -f file.txt ]
[[ -f file.txt ]]   # Extended test (bash specific, more features)
```

### File Tests

```bash
#!/bin/bash

file="/etc/passwd"

if test -f "$file"; then
    echo "$file is a regular file"
fi

if [ -r "$file" ]; then
    echo "$file is readable"
fi

if [[ -s "$file" && -r "$file" ]]; then
    echo "$file exists, is non-empty, and readable"
fi
```

### String Tests

```bash
#!/bin/bash

str1="hello"
str2="world"

if test "$str1" != "$str2"; then
    echo "Strings are different"
fi

if [[ "$str1" == h* ]]; then
    echo "str1 starts with 'h'"
fi

if [[ "$str1" =~ ^h.*o$ ]]; then
    echo "str1 matches regex ^h.*o$"
fi
```

### Integer Tests

```bash
#!/bin/bash

a=10
b=20

if test "$a" -lt "$b"; then
    echo "$a is less than $b"
fi

if [[ $a -lt $b ]]; then
    echo "$a < $b"
fi

# Arithmetic evaluation (no $ needed)
if (( a < b )); then
    echo "$a < $b"
fi
```

### Differences Between `[ ]` and `[[ ]]`

| Feature | `[ ]` | `[[ ]]` |
|---------|-------|---------|
| Word splitting | Yes | No |
| Glob expansion | Yes | No |
| Regex matching | No | Yes (`=~`) |
| Pattern matching | No | Yes (`==` with globs) |
| Logical operators | `-a`, `-o` | `&&`, `\|\|` |
| String comparison | `=`, `!=` | `==`, `!=`, `<`, `>` |

---

## 17. String Manipulation

### Length and Substrings

```bash
#!/bin/bash

str="Hello, World!"

# Length
echo "Length: ${#str}"

# Substring (start at index 7, length 5)
echo "Substring: ${str:7:5}"    # World

# Substring from index
echo "From index 7: ${str:7}"   # World!

# Substring from end
echo "Last 6: ${str: -6}"       # orld!

# Remove shortest match from start
echo "${str#Hello, }"           # World!

# Remove longest match from start
echo "${str##*, }"              # World!

# Remove shortest match from end
echo "${str%, World!}"          # Hello

# Remove longest match from end
echo "${str%%Hello*}"           # (empty)
```

### Search and Replace

```bash
#!/bin/bash

str="foo bar foo baz foo"

# Replace first occurrence
echo "${str/foo/FOO}"           # FOO bar foo baz foo

# Replace all occurrences
echo "${str//foo/FOO}"          # FOO bar FOO baz FOO

# Replace at start only
echo "${str/#foo/FOO}"          # FOO bar foo baz foo

# Replace at end only
echo "${str/%foo/FOO}"          # foo bar foo baz FOO

# Delete all occurrences
echo "${str//foo/}"             # bar baz
```

### Case Conversion

```bash
#!/bin/bash

str="Hello World"

# Uppercase
echo "${str^^}"                 # HELLO WORLD

# Lowercase
echo "${str,,}"                 # hello world

# Capitalize first character
echo "${str^}"                  # Hello World

# Toggle case
echo "${str~~}"                 # hELLO wORLD

# Specific character conversion
echo "${str^^[eo]}"             # HEllO WOrld
```

### Default Values

```bash
#!/bin/bash

# Use default if unset or empty
echo "${unset_var:-default}"

# Set default if unset or empty
echo "${unset_var:=default}"

# Error if unset or empty
echo "${required_var:?Variable is required}"

# Use alternative if set
echo "${set_var:+alternative}"
```

---

## 18. File Operations

### Reading Files

```bash
#!/bin/bash

# Read entire file
content=$(cat file.txt)

# Read line by line
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Read with line numbers
line_num=0
while IFS= read -r line; do
    ((line_num++))
    echo "$line_num: $line"
done < file.txt

# Read specific line (line 5)
sed -n '5p' file.txt

# Read first N lines
head -n 5 file.txt

# Read last N lines
tail -n 5 file.txt
```

### Writing Files

```bash
#!/bin/bash

# Overwrite
echo "Hello" > file.txt

# Append
echo "World" >> file.txt

# Here document
cat > config.txt << EOF
name=John
age=30
city=New York
EOF

# Here string
cat <<< "This is a here string"

# Redirect stderr
command 2> error.log

# Redirect stdout and stderr
command > output.log 2>&1
command &> output.log

# Discard output
command > /dev/null 2>&1
```

### File Manipulation

```bash
#!/bin/bash

# Create empty file
touch newfile.txt

# Create directory
mkdir -p /path/to/nested/dir

# Copy
cp source.txt dest.txt
cp -r source_dir/ dest_dir/

# Move / Rename
mv old.txt new.txt

# Remove
rm file.txt
rm -r directory/
rm -rf directory/    # Force remove (DANGEROUS!)

# Find files
find /path -name "*.txt"
find /path -type f -size +1M
find /path -type d -name "backup*"

# Check file integrity
md5sum file.txt
sha256sum file.txt
```

---

## 19. Process Management

### Running Processes

```bash
#!/bin/bash

# Run in background
sleep 10 &

# Get process ID
pid=$!
echo "Started process with PID: $pid"

# Wait for process to finish
wait $pid
echo "Process completed"

# Run with nohup (survives logout)
nohup long_running_command &

# Disown process
disown
```

### Process Information

```bash
#!/bin/bash

# Current process ID
echo "My PID: $$"

# Parent process ID
echo "Parent PID: $PPID"

# List processes
ps aux
ps -ef

# Find process by name
pgrep firefox
pidof bash

# Kill process
kill PID
kill -9 PID        # Force kill
killall process_name
pkill process_name
```

### Signals

```bash
#!/bin/bash

# Trap signals
cleanup() {
    echo "Cleaning up before exit..."
    rm -f /tmp/temp_file_$$
    exit 0
}

trap cleanup EXIT

# Trap specific signals
trap 'echo "Interrupted!"; exit 1' INT

# Ignore signal
trap '' TERM

# Reset to default
trap - INT
```

### Common Signals

| Signal | Name | Description |
|--------|------|-------------|
| 1 | SIGHUP | Hang up (terminal closed) |
| 2 | SIGINT | Interrupt (Ctrl+C) |
| 9 | SIGKILL | Force kill (cannot be trapped) |
| 15 | SIGTERM | Termination (default for kill) |
| 18 | SIGCONT | Continue (resume) |
| 19 | SIGSTOP | Stop (suspend, cannot be trapped) |
| 20 | SIGTSTP | Stop (Ctrl+Z) |

---

## 20. Error Handling & Debugging

### Exit Status

```bash
#!/bin/bash

# Every command returns an exit status
# 0 = success, 1-255 = error

ls /nonexistent
echo "Exit status: $?"    # 2

# Set exit on error
set -e

# Or in shebang
#!/bin/bash -e
```

### Error Handling Patterns

```bash
#!/bin/bash

# Check command success
if ! mkdir -p /path/to/dir; then
    echo "Failed to create directory" >&2
    exit 1
fi

# OR with exit on failure
mkdir -p /path/to/dir || { echo "Failed"; exit 1; }

# AND chain (continue only if successful)
command1 && command2 && command3

# Pipe fail (catch errors in pipelines)
set -o pipefail

# Trace mode (print each command)
set -x
# ... commands ...
set +x
```

### Debugging Techniques

```bash
#!/bin/bash

# Enable debug mode
set -x          # Print commands as they execute
set -v          # Print input lines as they are read
set -e          # Exit on error
set -u          # Exit on undefined variable
set -o pipefail # Catch pipeline errors

# Or combine in shebang
#!/bin/bash -euxo pipefail

# Debug specific section
set -x
# Debug this section
set +x

# Print variable values
echo "DEBUG: variable=$variable" >&2

# Function for debug logging
debug() {
    if [ "${DEBUG:-0}" -eq 1 ]; then
        echo "[DEBUG] $*" >&2
    fi
}

DEBUG=1 debug "This will print"
```

### Logging

```bash
#!/bin/bash

# Log file
LOGFILE="/var/log/myscript.log"

log() {
    local level=$1
    shift
    local message="$*"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $message" | tee -a "$LOGFILE"
}

log INFO "Starting script"
log WARN "This is a warning"
log ERROR "Something went wrong"
```

---

## 21. Real-World Examples

### Example 1: System Information Script

```bash
#!/bin/bash

# system_info.sh — Display comprehensive system information

print_header() {
    echo "========================================"
    echo "  $1"
    echo "========================================"
}

print_header "SYSTEM INFORMATION"
echo "Hostname: $(hostname)"
echo "OS: $(uname -o)"
echo "Kernel: $(uname -r)"
echo "Architecture: $(uname -m)"

print_header "CPU INFORMATION"
nproc
cat /proc/cpuinfo | grep "model name" | head -1

print_header "MEMORY INFORMATION"
free -h

print_header "DISK USAGE"
df -h | grep -E '(Filesystem|/dev/sd|/dev/nvme)'

print_header "NETWORK INTERFACES"
ip -br addr show

print_header "UPTIME"
uptime -p
```

### Example 2: Backup Script

```bash
#!/bin/bash

# backup.sh — Create timestamped backups

set -euo pipefail

SOURCE_DIR="${1:-$HOME/Documents}"
BACKUP_DIR="${2:-$HOME/Backups}"
RETENTION_DAYS=30

# Validate inputs
if [ ! -d "$SOURCE_DIR" ]; then
    echo "Error: Source directory does not exist: $SOURCE_DIR" >&2
    exit 1
fi

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Generate timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="backup_$(basename "$SOURCE_DIR")_${TIMESTAMP}.tar.gz"
BACKUP_PATH="$BACKUP_DIR/$BACKUP_NAME"

echo "Creating backup: $BACKUP_PATH"

# Create backup
tar -czf "$BACKUP_PATH" -C "$(dirname "$SOURCE_DIR")" "$(basename "$SOURCE_DIR")"

# Verify backup
if [ -f "$BACKUP_PATH" ]; then
    echo "Backup created successfully"
    echo "Size: $(du -h "$BACKUP_PATH" | cut -f1)"
else
    echo "Backup failed!" >&2
    exit 1
fi

# Clean old backups
echo "Cleaning backups older than $RETENTION_DAYS days..."
find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup complete!"
```

### Example 3: User Management Script

```bash
#!/bin/bash

# user_manager.sh — Manage system users

set -euo pipefail

usage() {
    echo "Usage: $0 {create|delete|list|lock|unlock} [username]"
    exit 1
}

create_user() {
    local username=$1
    if id "$username" &>/dev/null; then
        echo "User $username already exists"
        return 1
    fi

    read -sp "Enter password: " password
    echo

    useradd -m -s /bin/bash "$username"
    echo "$username:$password" | chpasswd
    echo "User $username created successfully"
}

delete_user() {
    local username=$1
    if ! id "$username" &>/dev/null; then
        echo "User $username does not exist"
        return 1
    fi

    read -p "Are you sure you want to delete $username? [y/N]: " confirm
    if [[ "$confirm" =~ ^[Yy]$ ]]; then
        userdel -r "$username"
        echo "User $username deleted"
    else
        echo "Cancelled"
    fi
}

list_users() {
    echo "System users:"
    awk -F: '$3 >= 1000 && $3 != 65534 {print $1}' /etc/passwd
}

lock_user() {
    local username=$1
    usermod -L "$username"
    echo "User $username locked"
}

unlock_user() {
    local username=$1
    usermod -U "$username"
    echo "User $username unlocked"
}

# Main
case "${1:-}" in
    create)
        [ -z "${2:-}" ] && usage
        create_user "$2"
        ;;
    delete)
        [ -z "${2:-}" ] && usage
        delete_user "$2"
        ;;
    list)
        list_users
        ;;
    lock)
        [ -z "${2:-}" ] && usage
        lock_user "$2"
        ;;
    unlock)
        [ -z "${2:-}" ] && usage
        unlock_user "$2"
        ;;
    *)
        usage
        ;;
esac
```

### Example 4: Git Repository Cloner

```bash
#!/bin/bash

# clone_repos.sh — Clone multiple GitHub repositories

REPOS=(
    "https://github.com/user/project1"
    "https://github.com/user/project2"
    "https://github.com/user/project3"
)

BASE_DIR="${1:-$HOME/Projects}"
mkdir -p "$BASE_DIR"

for repo in "${REPOS[@]}"; do
    repo_name=$(basename "$repo" .git)
    target_dir="$BASE_DIR/$repo_name"

    if [ -d "$target_dir/.git" ]; then
        echo "Updating $repo_name..."
        git -C "$target_dir" pull
    else
        echo "Cloning $repo_name..."
        git clone "$repo" "$target_dir"
    fi
done

echo "All repositories processed!"
```

### Example 5: File Integrity Checker

```bash
#!/bin/bash

# integrity_check.sh — Check file integrity using checksums

set -euo pipefail

DIRECTORY="${1:-.}"
CHECKSUM_FILE="checksums.sha256"

generate_checksums() {
    echo "Generating checksums for $DIRECTORY..."
    find "$DIRECTORY" -type f ! -name "$CHECKSUM_FILE" -exec sha256sum {} + > "$CHECKSUM_FILE"
    echo "Checksums saved to $CHECKSUM_FILE"
}

verify_checksums() {
    if [ ! -f "$CHECKSUM_FILE" ]; then
        echo "No checksum file found. Run with 'generate' first."
        exit 1
    fi

    echo "Verifying checksums..."
    if sha256sum -c "$CHECKSUM_FILE" --quiet; then
        echo "All files verified successfully!"
    else
        echo "WARNING: Some files have been modified or corrupted!" >&2
        exit 1
    fi
}

case "${2:-verify}" in
    generate)
        generate_checksums
        ;;
    verify)
        verify_checksums
        ;;
    *)
        echo "Usage: $0 [directory] {generate|verify}"
        exit 1
        ;;
esac
```

### Example 6: Nmap Automation

```bash
#!/bin/bash

# network_scan.sh — Automated network scanning with Nmap

set -euo pipefail

TARGET="${1:-192.168.1.0/24}"
OUTPUT_DIR="nmap_results"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mkdir -p "$OUTPUT_DIR"

echo "Scanning target: $TARGET"

# Quick scan
echo "[1/3] Running quick scan..."
nmap -sn "$TARGET" -oN "$OUTPUT_DIR/quick_${TIMESTAMP}.txt"

# Port scan
echo "[2/3] Running port scan..."
nmap -sS -p- --top-ports 1000 "$TARGET" -oN "$OUTPUT_DIR/ports_${TIMESTAMP}.txt"

# Vulnerability scan (requires NSE scripts)
echo "[3/3] Running vulnerability scan..."
nmap -sV --script vuln "$TARGET" -oN "$OUTPUT_DIR/vuln_${TIMESTAMP}.txt"

echo "Scan complete! Results saved to $OUTPUT_DIR/"
```

---

## 22. Best Practices

### Script Structure

```bash
#!/usr/bin/env bash

#===============================================================================
# Script Name: example.sh
# Description: Brief description of what this script does
# Author: Your Name
# Date: 2024-01-01
# Version: 1.0.0
# Usage: ./example.sh [options] <argument>
# Dependencies: list, of, required, commands
#===============================================================================

set -euo pipefail
IFS=$'\n\t'

# Configuration
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly CONFIG_FILE="${SCRIPT_DIR}/config.conf"

# Functions
main() {
    # Main execution logic
    parse_arguments "$@"
    validate_environment
    execute_task
    cleanup
}

parse_arguments() {
    # Parse command line arguments
    while [[ $# -gt 0 ]]; do
        case $1 in
            -h|--help)
                show_help
                exit 0
                ;;
            -v|--verbose)
                VERBOSE=1
                shift
                ;;
            *)
                echo "Unknown option: $1" >&2
                exit 1
                ;;
        esac
    done
}

# ... more functions ...

# Execute main if not sourced
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

### Safety Guidelines

1. **Always quote variables** — `"$var"` instead of `$var`
2. **Use `set -euo pipefail`** — Exit on errors, undefined variables, and pipeline failures
3. **Use `#!/usr/bin/env bash`** — Portable shebang
4. **Validate all inputs** — Check arguments, file existence, and permissions
5. **Use `local` in functions** — Prevent variable leakage
6. **Use `readonly` for constants** — Prevent accidental modification
7. **Log to stderr** — `echo "error" >&2` for error messages
8. **Clean up on exit** — Use `trap` for temporary files and resources
9. **Avoid `rm -rf`** — Especially with variables
10. **Check exit codes** — Always verify command success

### Style Guide

```bash
# Indentation: 4 spaces (no tabs)
# Variable names: lowercase with underscores
# Constants: UPPERCASE with underscores
# Functions: lowercase with underscores

# Good
count=0
MAX_RETRIES=5

my_function() {
    local name=$1
    echo "Hello, $name"
}

# Bad
count = 0      # Spaces around =
maxretries=5   # Hard to read
MyFunction() { # PascalCase
```

### Portability Tips

```bash
#!/bin/bash

# Use POSIX commands when possible
# Avoid bash-specific features if targeting sh

# Portable: works in sh and bash
echo "Hello"

# Bash-specific: avoid in sh scripts
# [[ ]] instead of [ ]
# $(( )) arithmetic
# Arrays
# [[ string =~ regex ]]

# Check bash version
if ((BASH_VERSINFO[0] < 4)); then
    echo "Bash 4+ required"
    exit 1
fi

# Use command -v instead of which
if command -v git &> /dev/null; then
    echo "Git is installed"
fi
```

---

## 23. Resources

### Official Documentation

- [GNU Bash Manual](https://www.gnu.org/software/bash/manual/)
- [Bash Reference Manual](https://www.gnu.org/software/bash/manual/html_node/)
- [POSIX Shell Specification](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)

### Online Resources

- [ShellCheck](https://www.shellcheck.net/) — Static analysis tool for shell scripts
- [Explain Shell](https://explainshell.com/) — Write down a command-line to see the help text
- [Bash Hackers Wiki](https://wiki.bash-hackers.org/)
- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/)

### Books

- *The Linux Command Line* by William Shotts
- *Classic Shell Scripting* by Arnold Robbins & Nelson Beebe
- *Learning the bash Shell* by Cameron Newham

### Video Tutorial

- [Shell Scripting Tutorial for Beginners by ProgrammingKnowledge](https://www.youtube.com/playlist?list=PLS1QulWo1RIaAsfcLW-Jk-Cx3JGRP8tjh)

---

## License

This guide is provided as-is for educational purposes. Feel free to use, modify, and distribute.

---

> **Contributing:** Found an error or want to add a topic? Open an issue or pull request!

> **Note:** This guide covers Bash 4.0+. Some features may not work on older systems or POSIX sh.
