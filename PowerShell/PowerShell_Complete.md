# PowerShell Complete Guide

## Table of Contents
- [Introduction](#introduction)
- [Getting Started](#getting-started)
- [Variables and Data Types](#variables-and-data-types)
- [Operators](#operators)
- [Conditional Statements](#conditional-statements)
- [Loops](#loops)
- [Functions](#functions)
- [Arrays and Collections](#arrays-and-collections)
- [Objects and Classes](#objects-and-classes)
- [Pipeline and Filtering](#pipeline-and-filtering)
- [File and Directory Operations](#file-and-directory-operations)
- [Text Processing](#text-processing)
- [Working with JSON and XML](#working-with-json-and-xml)
- [Remote Management](#remote-management)
- [Active Directory](#active-directory)
- [Registry Operations](#registry-operations)
- [Event Logs](#event-logs)
- [Scheduled Tasks](#scheduled-tasks)
- [Error Handling](#error-handling)
- [Modules and Scripts](#modules-and-scripts)
- [Security and Execution Policy](#security-and-execution-policy)
- [Performance and Optimization](#performance-and-optimization)
- [Best Practices](#best-practices)
- [Practical Examples](#practical-examples)

---

## Introduction

PowerShell is a cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework. It's built on .NET and provides powerful tools for system administration and automation.

### PowerShell vs Command Prompt

| Feature | PowerShell | Command Prompt |
|---------|-----------|----------------|
| Object-oriented | Yes | No (text-based) |
| Cmdlets | Yes | No |
| Scripting | Advanced | Basic |
| Pipeline | Objects | Text |
| Remote Management | Built-in | Limited |
| Cross-platform | Yes (Core) | Windows only |

### PowerShell Versions

- **PowerShell 5.1**: Windows PowerShell (pre-installed on Windows 10/11)
- **PowerShell 7+**: PowerShell Core (cross-platform, modern)

---

## Getting Started

### Running PowerShell

```powershell
# Windows PowerShell (5.1)
# Start > Windows PowerShell

# PowerShell 7+
# pwsh.exe

# Run as Administrator
# Right-click > Run as Administrator

# Check version
$PSVersionTable
$PSVersionTable.PSVersion

# Get PowerShell edition
$PSVersionTable.PSEdition  # Desktop or Core
```

### Basic Commands (Cmdlets)

```powershell
# Cmdlet naming convention: Verb-Noun

# Get help
Get-Help
Get-Help Get-Process
Get-Help Get-Process -Examples
Get-Help Get-Process -Full
Get-Help Get-Process -Online

# Update help
Update-Help

# Get commands
Get-Command
Get-Command -Verb Get
Get-Command -Noun Service
Get-Command *Process*

# Get aliases
Get-Alias
Get-Alias -Definition Get-ChildItem

# Create alias
New-Alias -Name list -Value Get-ChildItem
Set-Alias -Name list -Value Get-ChildItem

# Clear screen
Clear-Host
cls  # Alias

# Get location
Get-Location
pwd  # Alias

# Change location
Set-Location C:\Users
cd C:\Users  # Alias

# List items
Get-ChildItem
ls  # Alias
dir  # Alias

# Get member (properties and methods)
Get-Process | Get-Member
"Hello" | Get-Member
```

### Creating Your First Script

```powershell
# Create script file: script.ps1
# Note: .ps1 extension

# Simple script
Write-Host "Hello, World!"

# Save and run
.\script.ps1

# Run with full path
C:\Scripts\script.ps1

# Dot source (run in current scope)
. .\script.ps1
```

### Comments

```powershell
# Single-line comment

<#
Multi-line comment
This is a comment block
Everything here is ignored
#>

# Inline comment
Write-Host "Hello" # This prints Hello

<#
.SYNOPSIS
    Script description
.DESCRIPTION
    Detailed description
.PARAMETER Name
    Parameter description
.EXAMPLE
    Example usage
#>
```

---

## Variables and Data Types

### Variable Declaration

```powershell
# Variable (loosely typed)
$name = "John"
$age = 30
$isActive = $true

# Strongly typed
[string]$name = "John"
[int]$age = 30
[bool]$isActive = $true

# Multiple assignment
$a, $b, $c = 1, 2, 3

# Check variable type
$name.GetType()

# List all variables
Get-Variable

# Remove variable
Remove-Variable -Name name
```

### Data Types

```powershell
# String
$text = "Hello, World!"
$text = 'Single quotes'

# Integer
$number = 42
[int]$count = 100

# Long
[long]$bigNumber = 9223372036854775807

# Float/Double
[float]$price = 19.99
[double]$pi = 3.14159

# Decimal
[decimal]$money = 1234.56

# Boolean
$isTrue = $true
$isFalse = $false

# DateTime
$now = Get-Date
$date = [DateTime]"2024-01-01"

# Array
$array = @(1, 2, 3, 4, 5)
$array = 1, 2, 3, 4, 5

# ArrayList
$list = [System.Collections.ArrayList]@()
$list.Add("Item1")

# Hashtable
$hash = @{
    Name = "John"
    Age = 30
    City = "NYC"
}

# Ordered hashtable
$ordered = [ordered]@{
    First = 1
    Second = 2
    Third = 3
}

# PSCustomObject
$person = [PSCustomObject]@{
    Name = "John"
    Age = 30
    City = "NYC"
}

# Null
$null

# Type casting
[string]$num = 42
[int]$str = "42"
```

### Automatic Variables

```powershell
# Common automatic variables
$PSVersionTable      # PowerShell version
$HOME                # User home directory
$PWD                 # Current directory
$PROFILE             # Profile script path
$_                   # Current pipeline object
$?                   # Last command success status
$LASTEXITCODE        # Last exit code
$Error               # Array of error objects
$Host                # Host information
$PID                 # Process ID
$PSScriptRoot        # Script directory
$PSCommandPath       # Full script path
$Args                # Script arguments
$true, $false        # Boolean values
$null                # Null value
```

### String Operations

```powershell
# Concatenation
$firstName = "John"
$lastName = "Doe"
$fullName = $firstName + " " + $lastName
$fullName = "$firstName $lastName"

# String interpolation
$greeting = "Hello, $name!"
$path = "C:\Users\$env:USERNAME\Documents"

# Escape character
$text = "Line 1`nLine 2"  # Newline
$tab = "Col1`tCol2"       # Tab

# Here-string
$text = @"
Line 1
Line 2
Variable: $name
"@

# Here-string (literal)
$text = @'
Line 1
Line 2
No variable expansion: $name
'@

# String methods
$text = "Hello, World!"
$text.Length
$text.ToUpper()
$text.ToLower()
$text.Replace("World", "PowerShell")
$text.Contains("World")
$text.StartsWith("Hello")
$text.EndsWith("!")
$text.Substring(0, 5)
$text.Split(",")
$text.Trim()

# Format string
$formatted = "{0} is {1} years old" -f "John", 30
$formatted = [string]::Format("{0} is {1} years old", "John", 30)

# Join strings
$joined = "apple", "banana", "orange" -join ", "
```

---

## Operators

### Arithmetic Operators

```powershell
# Basic operations
$a = 10
$b = 3

$sum = $a + $b        # 13
$diff = $a - $b       # 7
$prod = $a * $b       # 30
$quot = $a / $b       # 3.33...
$mod = $a % $b        # 1

# Increment/Decrement
$a++
$a--
$a += 5
$a -= 3
$a *= 2
$a /= 2
$a %= 3

# Power
[Math]::Pow(2, 3)     # 8

# Math operations
[Math]::Abs(-5)       # 5
[Math]::Ceiling(3.2)  # 4
[Math]::Floor(3.8)    # 3
[Math]::Round(3.5)    # 4
[Math]::Sqrt(16)      # 4
[Math]::Max(5, 10)    # 10
[Math]::Min(5, 10)    # 5
```

### Comparison Operators

```powershell
# Equality
5 -eq 5               # True (equal)
5 -ne 3               # True (not equal)
5 -gt 3               # True (greater than)
5 -ge 5               # True (greater or equal)
3 -lt 5               # True (less than)
5 -le 5               # True (less or equal)

# Case-sensitive comparison
"hello" -ceq "HELLO"  # False
"hello" -ieq "HELLO"  # True (case-insensitive)

# String comparison
"abc" -eq "abc"       # True
"abc" -ne "def"       # True
"abc" -gt "aaa"       # True (lexicographic)

# Pattern matching
"Hello" -like "H*"    # True
"Hello" -notlike "W*" # True
"Hello" -match "^H"   # True (regex)
"Hello" -notmatch "^W" # True

# Contains
@(1,2,3) -contains 2  # True
@(1,2,3) -notcontains 5 # True
2 -in @(1,2,3)        # True
5 -notin @(1,2,3)     # True

# Type comparison
$value -is [string]
$value -isnot [int]
```

### Logical Operators

```powershell
# AND
($a -gt 5) -and ($b -lt 10)

# OR
($a -eq 5) -or ($b -eq 3)

# NOT
-not ($a -eq 5)
!($a -eq 5)

# XOR
($a -eq 5) -xor ($b -eq 3)

# Combining
($a -gt 5) -and (($b -lt 10) -or ($c -eq 0))
```

### Bitwise Operators

```powershell
# Bitwise AND
5 -band 3             # 1

# Bitwise OR
5 -bor 3              # 7

# Bitwise XOR
5 -bxor 3             # 6

# Bitwise NOT
-bnot 5               # -6

# Shift left
5 -shl 2              # 20

# Shift right
20 -shr 2             # 5
```

### Assignment Operators

```powershell
$a = 10               # Assignment
$a += 5               # Add and assign
$a -= 3               # Subtract and assign
$a *= 2               # Multiply and assign
$a /= 2               # Divide and assign
$a %= 3               # Modulo and assign
```

---

## Conditional Statements

### If-Else

```powershell
# Simple if
if ($age -gt 18) {
    Write-Host "Adult"
}

# If-else
if ($age -gt 18) {
    Write-Host "Adult"
} else {
    Write-Host "Minor"
}

# If-elseif-else
if ($score -ge 90) {
    Write-Host "A"
} elseif ($score -ge 80) {
    Write-Host "B"
} elseif ($score -ge 70) {
    Write-Host "C"
} else {
    Write-Host "F"
}

# Multiple conditions
if (($age -gt 18) -and ($hasLicense -eq $true)) {
    Write-Host "Can drive"
}

# One-line if
if ($value -gt 10) { Write-Host "Greater than 10" }

# Ternary operator (PowerShell 7+)
$result = $age -gt 18 ? "Adult" : "Minor"

# Null coalescing (PowerShell 7+)
$value = $null
$result = $value ?? "Default"
$result = $value ?? $fallback ?? "Default"
```

### Switch Statement

```powershell
# Basic switch
switch ($day) {
    1 { Write-Host "Monday" }
    2 { Write-Host "Tuesday" }
    3 { Write-Host "Wednesday" }
    default { Write-Host "Other day" }
}

# Multiple values
switch ($color) {
    "Red" { Write-Host "Color is red" }
    "Blue" { Write-Host "Color is blue" }
    { $_ -in "Green", "Yellow" } { Write-Host "Color is $color" }
    default { Write-Host "Unknown color" }
}

# Wildcard matching
switch -Wildcard ($filename) {
    "*.txt" { Write-Host "Text file" }
    "*.ps1" { Write-Host "PowerShell script" }
    "*.exe" { Write-Host "Executable" }
}

# Regex matching
switch -Regex ($text) {
    "^Error" { Write-Host "Error message" }
    "^Warning" { Write-Host "Warning message" }
    "\d+" { Write-Host "Contains numbers" }
}

# Case sensitive
switch -CaseSensitive ($text) {
    "Hello" { Write-Host "Exact match" }
}

# Switch on array
switch (@(1, 2, 3, 4)) {
    1 { Write-Host "One" }
    2 { Write-Host "Two" }
    3 { Write-Host "Three" }
    default { Write-Host "Other: $_" }
}

# Continue and break
switch ($value) {
    1 {
        Write-Host "One"
        break  # Exit switch
    }
    2 {
        Write-Host "Two"
        continue  # Continue to next case
    }
}
```

---

## Loops

### For Loop

```powershell
# Basic for loop
for ($i = 0; $i -lt 10; $i++) {
    Write-Host "Count: $i"
}

# Nested for loop
for ($i = 1; $i -le 3; $i++) {
    for ($j = 1; $j -le 3; $j++) {
        Write-Host "$i x $j = $($i * $j)"
    }
}

# Descending
for ($i = 10; $i -ge 0; $i--) {
    Write-Host $i
}

# Step by 2
for ($i = 0; $i -lt 20; $i += 2) {
    Write-Host "Even: $i"
}
```

### ForEach Loop

```powershell
# ForEach loop
$numbers = 1, 2, 3, 4, 5
foreach ($num in $numbers) {
    Write-Host "Number: $num"
}

# ForEach with files
$files = Get-ChildItem -Path C:\Temp
foreach ($file in $files) {
    Write-Host $file.Name
}

# ForEach-Object (pipeline)
1..10 | ForEach-Object {
    Write-Host "Number: $_"
}

# Short form
1..10 | % { Write-Host $_ }

# With Begin, Process, End
1..5 | ForEach-Object -Begin {
    Write-Host "Starting..."
} -Process {
    Write-Host "Processing: $_"
} -End {
    Write-Host "Finished!"
}

# Parallel processing (PowerShell 7+)
1..10 | ForEach-Object -Parallel {
    Start-Sleep -Seconds 1
    "Processing $_"
} -ThrottleLimit 3
```

### While Loop

```powershell
# While loop
$count = 0
while ($count -lt 5) {
    Write-Host "Count: $count"
    $count++
}

# While with user input
$continue = "y"
while ($continue -eq "y") {
    $continue = Read-Host "Continue? (y/n)"
}

# Infinite loop
while ($true) {
    Write-Host "Press Ctrl+C to stop"
    Start-Sleep -Seconds 1
}
```

### Do-While and Do-Until

```powershell
# Do-While (executes at least once)
$count = 0
do {
    Write-Host "Count: $count"
    $count++
} while ($count -lt 5)

# Do-Until (executes until condition is true)
$count = 0
do {
    Write-Host "Count: $count"
    $count++
} until ($count -ge 5)
```

### Loop Control

```powershell
# Break - exit loop
for ($i = 0; $i -lt 10; $i++) {
    if ($i -eq 5) {
        break
    }
    Write-Host $i
}

# Continue - skip to next iteration
for ($i = 0; $i -lt 10; $i++) {
    if ($i -eq 5) {
        continue
    }
    Write-Host $i
}

# Break with label
:outerLoop for ($i = 1; $i -le 3; $i++) {
    for ($j = 1; $j -le 3; $j++) {
        if ($i -eq 2 -and $j -eq 2) {
            break outerLoop
        }
        Write-Host "$i-$j"
    }
}
```

---

## Functions

### Function Declaration

```powershell
# Basic function
function Say-Hello {
    Write-Host "Hello, World!"
}

# Call function
Say-Hello

# Function with parameters
function Say-Hello {
    param(
        [string]$Name
    )
    Write-Host "Hello, $Name!"
}

Say-Hello -Name "John"

# Multiple parameters
function Add-Numbers {
    param(
        [int]$Number1,
        [int]$Number2
    )
    return $Number1 + $Number2
}

$result = Add-Numbers -Number1 5 -Number2 3
```

### Advanced Parameters

```powershell
# Advanced function with full parameter support
function Get-UserInfo {
    [CmdletBinding()]
    param(
        # Mandatory parameter
        [Parameter(Mandatory=$true)]
        [string]$UserName,

        # Optional with default
        [Parameter(Mandatory=$false)]
        [string]$Domain = "CONTOSO",

        # Parameter from pipeline
        [Parameter(ValueFromPipeline=$true)]
        [string]$ComputerName,

        # Parameter validation
        [ValidateRange(1, 100)]
        [int]$Age,

        # ValidateSet
        [ValidateSet("Admin", "User", "Guest")]
        [string]$Role,

        # ValidatePattern
        [ValidatePattern("^\d{3}-\d{3}-\d{4}$")]
        [string]$Phone,

        # ValidateScript
        [ValidateScript({Test-Path $_})]
        [string]$Path,

        # Switch parameter
        [switch]$Verbose
    )

    process {
        Write-Host "User: $Domain\$UserName"
        if ($Verbose) {
            Write-Host "Verbose output enabled"
        }
    }
}

# Call with parameters
Get-UserInfo -UserName "john" -Age 30 -Role "Admin"
```

### Return Values

```powershell
# Return value
function Get-Sum {
    param([int]$a, [int]$b)
    return $a + $b
}

$result = Get-Sum -a 5 -b 3

# Return object
function Get-PersonObject {
    param([string]$Name, [int]$Age)

    return [PSCustomObject]@{
        Name = $Name
        Age = $Age
    }
}

$person = Get-PersonObject -Name "John" -Age 30

# Return multiple values
function Get-MinMax {
    param([int[]]$Numbers)

    $min = ($Numbers | Measure-Object -Minimum).Minimum
    $max = ($Numbers | Measure-Object -Maximum).Maximum

    return @{
        Min = $min
        Max = $max
    }
}

$result = Get-MinMax -Numbers @(1, 5, 3, 9, 2)
Write-Host "Min: $($result.Min), Max: $($result.Max)"

# Early return
function Test-Number {
    param([int]$Number)

    if ($Number -lt 0) {
        return "Negative"
    }
    if ($Number -eq 0) {
        return "Zero"
    }
    return "Positive"
}
```

### Parameter Sets

```powershell
# Function with parameter sets
function Get-Data {
    [CmdletBinding(DefaultParameterSetName='ByName')]
    param(
        [Parameter(ParameterSetName='ByName', Mandatory=$true)]
        [string]$Name,

        [Parameter(ParameterSetName='ById', Mandatory=$true)]
        [int]$Id,

        [Parameter(ParameterSetName='All')]
        [switch]$All
    )

    switch ($PSCmdlet.ParameterSetName) {
        'ByName' { Write-Host "Getting by name: $Name" }
        'ById' { Write-Host "Getting by ID: $Id" }
        'All' { Write-Host "Getting all" }
    }
}

# Call with different parameter sets
Get-Data -Name "John"
Get-Data -Id 123
Get-Data -All
```

### Pipeline Functions

```powershell
# Function that accepts pipeline input
function Convert-ToUpper {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline=$true)]
        [string]$InputString
    )

    process {
        $InputString.ToUpper()
    }
}

# Usage
"hello", "world" | Convert-ToUpper

# Begin, Process, End blocks
function Process-Items {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline=$true)]
        [object]$InputObject
    )

    begin {
        Write-Host "Starting processing..."
        $count = 0
    }

    process {
        $count++
        Write-Host "Processing item $count : $InputObject"
    }

    end {
        Write-Host "Processed $count items total"
    }
}

# Usage
1..5 | Process-Items
```

---

## Arrays and Collections

### Arrays

```powershell
# Create array
$array = @(1, 2, 3, 4, 5)
$array = 1, 2, 3, 4, 5
$array = 1..10

# Empty array
$empty = @()

# Single element array
$single = @(1)
$single = ,1

# Mixed types
$mixed = @(1, "two", 3.0, $true)

# Access elements
$array[0]           # First element
$array[-1]          # Last element
$array[1..3]        # Range
$array[0,2,4]       # Specific indices

# Array properties
$array.Length
$array.Count

# Add elements
$array += 6
$array = $array + 7

# Remove element (create new array)
$array = $array | Where-Object { $_ -ne 3 }

# Check if contains
$array -contains 5
5 -in $array

# Array methods
$array.GetType()
$array.Clear()
$array.Clone()

# Multi-dimensional array
$matrix = @(
    @(1, 2, 3),
    @(4, 5, 6),
    @(7, 8, 9)
)
$matrix[0][1]       # Access element

# Sort array
$sorted = $array | Sort-Object
$sorted = $array | Sort-Object -Descending

# Reverse array
$reversed = $array | Sort-Object -Descending
[Array]::Reverse($array)

# Filter array
$filtered = $array | Where-Object { $_ -gt 3 }

# Transform array
$doubled = $array | ForEach-Object { $_ * 2 }

# Join array
$joined = $array -join ", "

# Split string to array
$array = "a,b,c" -split ","
```

### ArrayList

```powershell
# Create ArrayList
$list = [System.Collections.ArrayList]@()
$list = New-Object System.Collections.ArrayList

# Add items
$list.Add("Item1") | Out-Null
$list.AddRange(@("Item2", "Item3"))

# Access items
$list[0]

# Remove items
$list.Remove("Item1")
$list.RemoveAt(0)
$list.Clear()

# Properties
$list.Count
$list.Capacity

# Methods
$list.Contains("Item1")
$list.IndexOf("Item2")
$list.Insert(1, "NewItem")
$list.Sort()
$list.Reverse()
```

### Hashtables

```powershell
# Create hashtable
$hash = @{
    Name = "John"
    Age = 30
    City = "NYC"
}

# Ordered hashtable
$ordered = [ordered]@{
    First = 1
    Second = 2
    Third = 3
}

# Access values
$hash["Name"]
$hash.Name

# Add/Update
$hash["Email"] = "john@example.com"
$hash.Phone = "123-456-7890"

# Remove
$hash.Remove("City")

# Check if key exists
$hash.ContainsKey("Name")
$hash.ContainsValue("John")

# Get keys and values
$hash.Keys
$hash.Values

# Loop through hashtable
foreach ($key in $hash.Keys) {
    Write-Host "$key : $($hash[$key])"
}

# Clone hashtable
$clone = $hash.Clone()

# Clear hashtable
$hash.Clear()

# Count
$hash.Count
```

### Generic Collections

```powershell
# Generic List
$list = [System.Collections.Generic.List[string]]::new()
$list.Add("Item1")
$list.Add("Item2")

# Generic Dictionary
$dict = [System.Collections.Generic.Dictionary[string,int]]::new()
$dict.Add("One", 1)
$dict.Add("Two", 2)

# Queue (FIFO)
$queue = [System.Collections.Queue]::new()
$queue.Enqueue("First")
$queue.Enqueue("Second")
$item = $queue.Dequeue()

# Stack (LIFO)
$stack = [System.Collections.Stack]::new()
$stack.Push("First")
$stack.Push("Second")
$item = $stack.Pop()
```

---

## Objects and Classes

### Working with Objects

```powershell
# Create custom object
$person = [PSCustomObject]@{
    Name = "John Doe"
    Age = 30
    City = "NYC"
}

# Access properties
$person.Name
$person.Age

# Add property
$person | Add-Member -MemberType NoteProperty -Name Email -Value "john@example.com"

# Add method
$person | Add-Member -MemberType ScriptMethod -Name GetInfo -Value {
    return "$($this.Name) is $($this.Age) years old"
}

# Call method
$person.GetInfo()

# Select properties
$person | Select-Object Name, Age

# Get object members
$person | Get-Member

# Convert to JSON
$person | ConvertTo-Json

# Convert from JSON
$json = '{"Name":"Jane","Age":25}'
$object = $json | ConvertFrom-Json
```

### PowerShell Classes (PowerShell 5+)

```powershell
# Define class
class Person {
    # Properties
    [string]$Name
    [int]$Age
    [string]$Email

    # Constructor
    Person([string]$name, [int]$age) {
        $this.Name = $name
        $this.Age = $age
    }

    # Method
    [string]GetInfo() {
        return "$($this.Name) is $($this.Age) years old"
    }

    # Static method
    static [Person]Create([string]$name, [int]$age) {
        return [Person]::new($name, $age)
    }
}

# Create instance
$person = [Person]::new("John", 30)
$person = New-Object Person("John", 30)

# Access properties
$person.Name
$person.Age = 31

# Call method
$person.GetInfo()

# Static method
$person = [Person]::Create("Jane", 25)
```

### Inheritance

```powershell
# Base class
class Animal {
    [string]$Name

    Animal([string]$name) {
        $this.Name = $name
    }

    [void]MakeSound() {
        Write-Host "Some sound"
    }
}

# Derived class
class Dog : Animal {
    [string]$Breed

    Dog([string]$name, [string]$breed) : base($name) {
        $this.Breed = $breed
    }

    # Override method
    [void]MakeSound() {
        Write-Host "Woof! I'm $($this.Name)"
    }
}

# Usage
$dog = [Dog]::new("Rex", "Labrador")
$dog.MakeSound()
```

---

## Pipeline and Filtering

### Pipeline Basics

```powershell
# Basic pipeline
Get-Process | Where-Object { $_.CPU -gt 10 }

# Multiple pipeline stages
Get-Process |
    Where-Object { $_.CPU -gt 10 } |
    Sort-Object CPU -Descending |
    Select-Object -First 10

# $_ represents current object
Get-ChildItem | ForEach-Object { $_.Name }
```

### Where-Object (Filtering)

```powershell
# Filter with scriptblock
Get-Service | Where-Object { $_.Status -eq "Running" }

# Simplified syntax
Get-Service | Where-Object Status -eq "Running"

# Multiple conditions
Get-Process | Where-Object {
    ($_.CPU -gt 10) -and ($_.WorkingSet -gt 100MB)
}

# Filter operators
Get-ChildItem | Where-Object Name -like "*.txt"
Get-ChildItem | Where-Object Name -match "^\d+"
Get-Process | Where-Object CPU -gt 10

# Common filters
Get-ChildItem | Where-Object PSIsContainer  # Directories only
Get-ChildItem | Where-Object {-not $_.PSIsContainer}  # Files only
```

### Select-Object

```powershell
# Select properties
Get-Process | Select-Object Name, CPU, Id

# Select first/last
Get-Process | Select-Object -First 10
Get-Process | Select-Object -Last 5

# Select unique
Get-Process | Select-Object -Unique ProcessName

# Calculated properties
Get-Process | Select-Object Name,
    @{Name="CPUUsage"; Expression={$_.CPU}},
    @{Name="MemoryMB"; Expression={$_.WorkingSet / 1MB}}

# Exclude properties
Get-Process | Select-Object * -ExcludeProperty Handles
```

### Sort-Object

```powershell
# Sort ascending
Get-Process | Sort-Object CPU

# Sort descending
Get-Process | Sort-Object CPU -Descending

# Multiple sort
Get-Process | Sort-Object CPU, WorkingSet

# Sort with property
Get-ChildItem | Sort-Object LastWriteTime -Descending

# Unique sort
Get-Process | Sort-Object ProcessName -Unique
```

### Group-Object

```powershell
# Group by property
Get-Process | Group-Object ProcessName

# Group with count
Get-Service | Group-Object Status

# Access groups
$groups = Get-Process | Group-Object ProcessName
$groups[0].Name
$groups[0].Count
$groups[0].Group

# Hashtable output
Get-Process | Group-Object ProcessName -AsHashTable
```

### Measure-Object

```powershell
# Count
Get-Process | Measure-Object

# Statistics
Get-Process | Measure-Object CPU -Average -Maximum -Minimum -Sum

# Count specific property
Get-Process | Measure-Object -Property CPU

# String statistics
Get-Content file.txt | Measure-Object -Line -Word -Character
```

---

## File and Directory Operations

### Get-ChildItem (List Files)

```powershell
# List current directory
Get-ChildItem
ls  # Alias
dir # Alias

# List specific path
Get-ChildItem -Path C:\Users

# Recursive
Get-ChildItem -Recurse

# Filter by name
Get-ChildItem -Filter "*.txt"
Get-ChildItem -Include "*.txt", "*.log" -Recurse
Get-ChildItem -Exclude "*.tmp"

# Directories only
Get-ChildItem -Directory

# Files only
Get-ChildItem -File

# Hidden files
Get-ChildItem -Hidden
Get-ChildItem -Force  # Include hidden

# By attributes
Get-ChildItem -Attributes ReadOnly
Get-ChildItem -Attributes Directory, !Hidden

# Depth limit
Get-ChildItem -Recurse -Depth 2
```

### File Operations

```powershell
# Create file
New-Item -Path "file.txt" -ItemType File
New-Item -Path "file.txt" -ItemType File -Force  # Overwrite

# Create directory
New-Item -Path "NewFolder" -ItemType Directory
mkdir NewFolder  # Alias

# Copy
Copy-Item -Path "source.txt" -Destination "dest.txt"
Copy-Item -Path "Folder" -Destination "NewFolder" -Recurse

# Move/Rename
Move-Item -Path "old.txt" -Destination "new.txt"
Rename-Item -Path "old.txt" -NewName "new.txt"

# Delete
Remove-Item -Path "file.txt"
Remove-Item -Path "Folder" -Recurse
Remove-Item -Path "*.tmp" -Force

# Test if exists
Test-Path -Path "file.txt"
Test-Path -Path "C:\Folder" -PathType Container

# Get item
Get-Item -Path "file.txt"

# Get file info
$file = Get-Item "file.txt"
$file.Name
$file.FullName
$file.Length
$file.LastWriteTime
$file.CreationTime
$file.Extension
$file.Directory
```

### File Content

```powershell
# Read file
Get-Content -Path "file.txt"
cat file.txt  # Alias

# Read as single string
Get-Content -Path "file.txt" -Raw

# Read first/last lines
Get-Content -Path "file.txt" -TotalCount 10
Get-Content -Path "file.txt" -Tail 10

# Read with encoding
Get-Content -Path "file.txt" -Encoding UTF8

# Write file (overwrite)
Set-Content -Path "file.txt" -Value "Hello, World!"
"Hello, World!" | Set-Content -Path "file.txt"

# Append to file
Add-Content -Path "file.txt" -Value "New line"
"New line" | Add-Content -Path "file.txt"

# Write array
$lines = "Line 1", "Line 2", "Line 3"
$lines | Set-Content -Path "file.txt"

# Out to file
Get-Process | Out-File -FilePath "processes.txt"
Get-Process | Out-File "processes.txt" -Append
Get-Process | Out-File "processes.txt" -Encoding UTF8

# Clear content
Clear-Content -Path "file.txt"
```

### Path Operations

```powershell
# Join paths
Join-Path -Path "C:\Users" -ChildPath "Documents"

# Split path
Split-Path -Path "C:\Users\Documents\file.txt" -Parent
Split-Path -Path "C:\Users\Documents\file.txt" -Leaf

# Get absolute path
Resolve-Path -Path ".\file.txt"

# Test path type
Test-Path -Path "C:\Windows" -PathType Container
Test-Path -Path "file.txt" -PathType Leaf

# Get parent directory
(Get-Item "C:\Users\Documents\file.txt").Directory

# Convert path
Convert-Path -Path "~\Documents"  # Expand ~

# Get temp path
$env:TEMP
[System.IO.Path]::GetTempPath()
```

---

## Text Processing

### String Search and Replace

```powershell
# Select-String (like grep)
Get-Content file.txt | Select-String "pattern"
Select-String -Path "*.txt" -Pattern "error"

# Case-insensitive
Select-String -Path "file.txt" -Pattern "error" -CaseSensitive

# Show context
Select-String -Path "file.txt" -Pattern "error" -Context 2,2

# Multiple files
Select-String -Path "*.log" -Pattern "error"

# Regex
Select-String -Path "file.txt" -Pattern "\d{3}-\d{3}-\d{4}"

# Replace in string
$text = "Hello, World!"
$text -replace "World", "PowerShell"

# Replace in file
(Get-Content file.txt) -replace "old", "new" | Set-Content file.txt

# Regex replace
$text -replace "\d+", "NUMBER"
```

### String Splitting and Joining

```powershell
# Split string
$text = "apple,banana,orange"
$array = $text -split ","

# Split with regex
$text = "apple banana  orange"
$array = $text -split "\s+"

# Join array
$array = "apple", "banana", "orange"
$text = $array -join ", "

# Join with newline
$text = $array -join "`n"
```

### Text Formatting

```powershell
# Format operator
$name = "John"
$age = 30
$text = "Name: {0}, Age: {1}" -f $name, $age

# String formatting
$text = [string]::Format("Name: {0}, Age: {1}", $name, $age)

# Format table
Get-Process | Format-Table Name, CPU, WorkingSet
Get-Process | ft Name, CPU  # Alias

# Format list
Get-Process | Format-List *
Get-Process | fl *  # Alias

# Format wide
Get-Process | Format-Wide Name -Column 3

# Custom format
Get-Process | Format-Table @{
    Label="Process Name"
    Expression={$_.Name}
}, @{
    Label="Memory (MB)"
    Expression={$_.WorkingSet / 1MB}
    FormatString="N2"
}
```

---

## Working with JSON and XML

### JSON

```powershell
# Convert to JSON
$data = @{
    Name = "John"
    Age = 30
    Hobbies = @("Reading", "Coding")
}
$json = $data | ConvertTo-Json

# Pretty print
$json = $data | ConvertTo-Json -Depth 10

# Convert from JSON
$json = '{"Name":"John","Age":30}'
$object = $json | ConvertFrom-Json

# Read JSON file
$data = Get-Content "data.json" | ConvertFrom-Json

# Write JSON file
$data | ConvertTo-Json | Set-Content "data.json"

# Access properties
$data.Name
$data.Hobbies[0]
```

### XML

```powershell
# Create XML
[xml]$xml = @"
<root>
    <person>
        <name>John</name>
        <age>30</age>
    </person>
</root>
"@

# Access elements
$xml.root.person.name
$xml.root.person.age

# Load XML file
[xml]$xml = Get-Content "data.xml"

# Save XML file
$xml.Save("C:\path\to\file.xml")

# Select nodes
$xml.SelectNodes("//person")
$xml.SelectSingleNode("//person[@id='1']")

# Create XML programmatically
$xml = [xml]"<root></root>"
$person = $xml.CreateElement("person")
$name = $xml.CreateElement("name")
$name.InnerText = "John"
$person.AppendChild($name)
$xml.root.AppendChild($person)

# Convert to XML
$data | ConvertTo-Xml
```

### CSV

```powershell
# Import CSV
$data = Import-Csv "data.csv"

# Import with custom delimiter
$data = Import-Csv "data.csv" -Delimiter ";"

# Import without header
$data = Import-Csv "data.csv" -Header "Name","Age","City"

# Export to CSV
$data | Export-Csv "output.csv" -NoTypeInformation

# Append to CSV
$data | Export-Csv "output.csv" -Append -NoTypeInformation

# Convert to CSV
$data | ConvertTo-Csv

# Convert from CSV
$csv | ConvertFrom-Csv
```

---

## Remote Management

### PowerShell Remoting

```powershell
# Enable remoting (run as admin)
Enable-PSRemoting -Force

# Check remoting status
Test-WSMan

# Connect to remote computer
Enter-PSSession -ComputerName Server01
Enter-PSSession -ComputerName Server01 -Credential (Get-Credential)

# Exit remote session
Exit-PSSession

# Run command on remote computer
Invoke-Command -ComputerName Server01 -ScriptBlock {
    Get-Service
}

# Run on multiple computers
Invoke-Command -ComputerName Server01, Server02 -ScriptBlock {
    Get-Process
}

# Run with credentials
$cred = Get-Credential
Invoke-Command -ComputerName Server01 -Credential $cred -ScriptBlock {
    Get-Service
}

# Run script file
Invoke-Command -ComputerName Server01 -FilePath "C:\Scripts\script.ps1"

# Persistent session
$session = New-PSSession -ComputerName Server01
Invoke-Command -Session $session -ScriptBlock { Get-Service }
Remove-PSSession -Session $session

# Copy files to remote
Copy-Item -Path "local.txt" -Destination "C:\remote\" -ToSession $session

# Copy from remote
Copy-Item -Path "C:\remote\file.txt" -Destination "C:\local\" -FromSession $session

# Configure trusted hosts (for workgroup)
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "Server01" -Force
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force
```

### SSH Remoting (PowerShell 7+)

```powershell
# Connect via SSH
Enter-PSSession -HostName user@server.com -SSHTransport

# Run command via SSH
Invoke-Command -HostName user@server.com -ScriptBlock { Get-Process }

# With key authentication
Invoke-Command -HostName user@server.com -KeyFilePath "C:\keys\id_rsa" -ScriptBlock { ... }
```

---

## Active Directory

### AD Module

```powershell
# Import module
Import-Module ActiveDirectory

# Get AD user
Get-ADUser -Identity john.doe
Get-ADUser -Filter "Name -like '*John*'"
Get-ADUser -Filter * -Properties *

# Create user
New-ADUser -Name "John Doe" `
    -GivenName "John" `
    -Surname "Doe" `
    -SamAccountName "john.doe" `
    -UserPrincipalName "john.doe@contoso.com" `
    -Path "OU=Users,DC=contoso,DC=com" `
    -AccountPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) `
    -Enabled $true

# Modify user
Set-ADUser -Identity john.doe -Department "IT"
Set-ADUser -Identity john.doe -Add @{telephoneNumber="555-1234"}

# Remove user
Remove-ADUser -Identity john.doe

# Get AD group
Get-ADGroup -Identity "Domain Admins"
Get-ADGroup -Filter "Name -like '*Admin*'"

# Create group
New-ADGroup -Name "IT Team" -GroupScope Global -GroupCategory Security

# Add user to group
Add-ADGroupMember -Identity "IT Team" -Members john.doe

# Remove user from group
Remove-ADGroupMember -Identity "IT Team" -Members john.doe

# Get group members
Get-ADGroupMember -Identity "Domain Admins"

# Get user groups
Get-ADPrincipalGroupMembership -Identity john.doe

# Get AD computer
Get-ADComputer -Identity PC01
Get-ADComputer -Filter * -Properties *

# Search AD
Get-ADObject -Filter "Name -like '*Server*'"

# Get OU
Get-ADOrganizationalUnit -Filter *
```

---

## Registry Operations

### Registry Access

```powershell
# Registry provider
Get-PSDrive -PSProvider Registry

# Navigate registry
Set-Location HKLM:\
Set-Location HKCU:\

# List keys
Get-ChildItem -Path HKLM:\SOFTWARE

# Get value
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion"

# Get specific value
Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\..." -Name "PropertyName"

# Create key
New-Item -Path "HKLM:\SOFTWARE\MyApp"

# Create value
New-ItemProperty -Path "HKLM:\SOFTWARE\MyApp" `
    -Name "Version" `
    -Value "1.0" `
    -PropertyType String

# Set value
Set-ItemProperty -Path "HKLM:\SOFTWARE\MyApp" `
    -Name "Version" `
    -Value "2.0"

# Remove value
Remove-ItemProperty -Path "HKLM:\SOFTWARE\MyApp" -Name "Version"

# Remove key
Remove-Item -Path "HKLM:\SOFTWARE\MyApp"

# Test if exists
Test-Path "HKLM:\SOFTWARE\MyApp"
```

---

## Event Logs

### Working with Event Logs

```powershell
# Get event logs
Get-EventLog -List

# Get events
Get-EventLog -LogName Application -Newest 10
Get-EventLog -LogName System -EntryType Error

# Filter events
Get-EventLog -LogName System -After (Get-Date).AddDays(-1)
Get-EventLog -LogName Application -Source "MyApp"

# Get Windows Events (newer)
Get-WinEvent -LogName Application -MaxEvents 10

# Filter with hashtable
Get-WinEvent -FilterHashtable @{
    LogName = 'System'
    Level = 2  # Error
    StartTime = (Get-Date).AddDays(-1)
}

# Filter with XPath
Get-WinEvent -LogName Application -FilterXPath "*[System[EventID=1000]]"

# Create event log
New-EventLog -LogName "MyApp" -Source "MyAppSource"

# Write event
Write-EventLog -LogName "MyApp" `
    -Source "MyAppSource" `
    -EventId 1001 `
    -EntryType Information `
    -Message "Application started"

# Remove event log
Remove-EventLog -LogName "MyApp"

# Clear event log
Clear-EventLog -LogName Application
```

---

## Scheduled Tasks

### Task Scheduler

```powershell
# Get scheduled tasks
Get-ScheduledTask
Get-ScheduledTask -TaskName "MyTask"

# Create scheduled task
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-File C:\Scripts\script.ps1"

$trigger = New-ScheduledTaskTrigger -Daily -At 9am

Register-ScheduledTask -TaskName "MyTask" `
    -Action $action `
    -Trigger $trigger `
    -Description "My scheduled task"

# Create task with options
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" `
    -LogonType ServiceAccount -RunLevel Highest

$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries

Register-ScheduledTask -TaskName "MyTask" `
    -Action $action `
    -Trigger $trigger `
    -Principal $principal `
    -Settings $settings

# Run task
Start-ScheduledTask -TaskName "MyTask"

# Disable task
Disable-ScheduledTask -TaskName "MyTask"

# Enable task
Enable-ScheduledTask -TaskName "MyTask"

# Remove task
Unregister-ScheduledTask -TaskName "MyTask" -Confirm:$false

# Get task info
$task = Get-ScheduledTask -TaskName "MyTask"
$task.State
$task.LastRunTime
$task.NextRunTime
```

---

## Error Handling

### Try-Catch-Finally

```powershell
# Basic try-catch
try {
    Get-Content "nonexistent.txt"
} catch {
    Write-Host "Error: $_"
}

# Specific exception
try {
    $result = 1 / 0
} catch [System.DivideByZeroException] {
    Write-Host "Cannot divide by zero"
} catch {
    Write-Host "General error: $_"
}

# Finally block
try {
    $file = [System.IO.File]::Open("file.txt", "Open")
    # Do something
} catch {
    Write-Host "Error: $_"
} finally {
    if ($file) {
        $file.Close()
    }
}

# Access error details
try {
    Get-Content "nonexistent.txt"
} catch {
    Write-Host "Error message: $($_.Exception.Message)"
    Write-Host "Error type: $($_.Exception.GetType().FullName)"
    Write-Host "Stack trace: $($_.ScriptStackTrace)"
}
```

### Error Handling Preferences

```powershell
# Error action preference
$ErrorActionPreference = "Stop"        # Terminate on error
$ErrorActionPreference = "Continue"    # Continue (default)
$ErrorActionPreference = "SilentlyContinue"  # Suppress errors
$ErrorActionPreference = "Inquire"     # Prompt user

# Per-command error action
Get-Content "file.txt" -ErrorAction Stop
Get-Content "file.txt" -ErrorAction SilentlyContinue

# Warning preference
$WarningPreference = "Continue"
$WarningPreference = "SilentlyContinue"

# Verbose preference
$VerbosePreference = "Continue"
```

### Error Variable

```powershell
# Automatic error variable
$Error  # Array of errors

# Last error
$Error[0]

# Clear errors
$Error.Clear()

# Error variable per command
Get-Content "file.txt" -ErrorVariable myError
$myError
```

### Throw and Write-Error

```powershell
# Throw terminating error
throw "This is a critical error"

# Throw with exception
throw [System.ArgumentException]"Invalid argument"

# Write non-terminating error
Write-Error "This is an error"
Write-Error "Error" -ErrorAction Stop  # Make it terminating

# Write warning
Write-Warning "This is a warning"

# Write verbose
Write-Verbose "Verbose message" -Verbose
```

---

## Modules and Scripts

### Script Files

```powershell
# Script parameters
param(
    [Parameter(Mandatory=$true)]
    [string]$Name,

    [int]$Age = 0,

    [switch]$Verbose
)

Write-Host "Name: $Name"
Write-Host "Age: $Age"

# Run script
.\script.ps1 -Name "John" -Age 30

# Dot source script (run in current scope)
. .\script.ps1 -Name "John"
```

### Modules

```powershell
# List modules
Get-Module -ListAvailable
Get-Module  # Loaded modules

# Import module
Import-Module ModuleName

# Import from path
Import-Module "C:\Path\To\Module.psm1"

# Remove module
Remove-Module ModuleName

# Get module commands
Get-Command -Module ModuleName

# Module locations
$env:PSModulePath -split ";"

# Create module
# File: MyModule.psm1

function Get-Something {
    Write-Host "Getting something..."
}

function Set-Something {
    param([string]$Value)
    Write-Host "Setting to $Value"
}

Export-ModuleMember -Function Get-Something, Set-Something

# Module manifest
# Create manifest
New-ModuleManifest -Path MyModule.psd1 `
    -Author "Your Name" `
    -Description "My module" `
    -ModuleVersion "1.0"
```

### PowerShell Gallery

```powershell
# Find module
Find-Module -Name ModuleName

# Install module
Install-Module -Name ModuleName
Install-Module -Name ModuleName -Scope CurrentUser

# Update module
Update-Module -Name ModuleName

# Uninstall module
Uninstall-Module -Name ModuleName

# Publish module (requires API key)
Publish-Module -Name ModuleName -NuGetApiKey "your-api-key"

# Install script
Install-Script -Name ScriptName

# Save module without installing
Save-Module -Name ModuleName -Path C:\Modules
```

---

## Security and Execution Policy

### Execution Policy

```powershell
# Get execution policy
Get-ExecutionPolicy
Get-ExecutionPolicy -List

# Set execution policy
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
Set-ExecutionPolicy -ExecutionPolicy Unrestricted
Set-ExecutionPolicy -ExecutionPolicy Restricted

# Scopes
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
Set-ExecutionPolicy -Scope LocalMachine -ExecutionPolicy RemoteSigned

# Bypass execution policy
PowerShell.exe -ExecutionPolicy Bypass -File script.ps1

# Execution policies:
# Restricted - No scripts allowed
# AllSigned - Only signed scripts
# RemoteSigned - Remote scripts must be signed
# Unrestricted - All scripts allowed
# Bypass - Nothing is blocked
```

### Script Signing

```powershell
# Get certificate
$cert = Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert

# Sign script
Set-AuthenticodeSignature -FilePath script.ps1 -Certificate $cert

# Verify signature
Get-AuthenticodeSignature -FilePath script.ps1
```

### Credentials

```powershell
# Get credential
$cred = Get-Credential

# Use credential
Get-WmiObject -Class Win32_Service -ComputerName Server01 -Credential $cred

# Create credential
$password = ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("username", $password)

# Store credential (encrypted)
$cred | Export-Clixml -Path cred.xml

# Import credential
$cred = Import-Clixml -Path cred.xml

# Secure string
$secure = Read-Host "Enter password" -AsSecureString
$secure = ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force

# Convert secure string to plain text (use carefully!)
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secure)
$plain = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
```

---

## Performance and Optimization

### Performance Tips

```powershell
# Use .NET classes when faster
# Instead of Get-ChildItem for simple tasks
[System.IO.Directory]::GetFiles("C:\Temp")

# Use pipeline efficiently
# Bad (processes all, then selects)
Get-Process | Select-Object -First 10

# Good (stops after 10)
Get-Process | Select-Object -First 10 -Wait

# Avoid ForEach-Object for small datasets
# Use foreach loop instead
$items = 1..100
foreach ($item in $items) {
    # Faster for small datasets
}

# Use ArrayList for adding items
$list = [System.Collections.ArrayList]@()
1..1000 | ForEach-Object { $list.Add($_) | Out-Null }

# Use StringBuilder for string concatenation
$sb = [System.Text.StringBuilder]::new()
1..1000 | ForEach-Object { $sb.Append("Item $_`n") | Out-Null }
$result = $sb.ToString()

# Filter left, format right
Get-Process | Where-Object CPU -gt 10 | Format-Table  # Good
Get-Process | Format-Table | Where-Object CPU -gt 10  # Bad
```

### Measure Performance

```powershell
# Measure-Command
Measure-Command {
    Get-Process
}

# Compare methods
$time1 = Measure-Command {
    # Method 1
}

$time2 = Measure-Command {
    # Method 2
}

Write-Host "Method 1: $($time1.TotalMilliseconds)ms"
Write-Host "Method 2: $($time2.TotalMilliseconds)ms"

# Start-Transcript (log session)
Start-Transcript -Path "session.log"
# Commands...
Stop-Transcript
```

---

## Best Practices

### Script Template

```powershell
<#
.SYNOPSIS
    Brief description of script

.DESCRIPTION
    Detailed description of what the script does

.PARAMETER Name
    Description of the Name parameter

.PARAMETER Path
    Description of the Path parameter

.EXAMPLE
    .\Script.ps1 -Name "John" -Path "C:\Data"
    Description of this example

.NOTES
    Author: Your Name
    Date: 2024-01-01
    Version: 1.0
#>

[CmdletBinding()]
param(
    [Parameter(Mandatory=$true, HelpMessage="Enter a name")]
    [ValidateNotNullOrEmpty()]
    [string]$Name,

    [Parameter(Mandatory=$false)]
    [ValidateScript({Test-Path $_})]
    [string]$Path = "C:\Default",

    [switch]$Force
)

begin {
    Write-Verbose "Starting script execution"
    $ErrorActionPreference = "Stop"
}

process {
    try {
        # Main logic here
        Write-Verbose "Processing $Name"

    } catch {
        Write-Error "An error occurred: $_"
        throw
    }
}

end {
    Write-Verbose "Script execution completed"
}
```

### Best Practices Checklist

```powershell
# 1. Use approved verbs
Get-Verb

# 2. Use cmdlet binding
[CmdletBinding()]

# 3. Use parameter validation
[ValidateNotNullOrEmpty()]
[ValidateRange(1, 100)]
[ValidateSet("Option1", "Option2")]

# 4. Write verbose output
Write-Verbose "Processing item"

# 5. Handle errors properly
try { } catch { }

# 6. Use comment-based help
<# .SYNOPSIS #>

# 7. Use meaningful names
$userAccount (Good)
$x (Bad)

# 8. Use proper formatting
# One statement per line
# Consistent indentation

# 9. Test scripts
Pester tests

# 10. Use modules for reusability

# 11. Avoid hardcoded values
# Use parameters or config files

# 12. Use pipeline when appropriate
Get-Service | Where-Object Status -eq "Running"

# 13. Close resources in finally blocks
try { } finally { $resource.Close() }

# 14. Use proper data types
[int]$age
[string]$name

# 15. Document your code
# Comments, help, examples
```

---

## Practical Examples

### System Information Script

```powershell
<#
.SYNOPSIS
    Get comprehensive system information
#>

function Get-SystemInfo {
    [CmdletBinding()]
    param(
        [string]$ComputerName = $env:COMPUTERNAME
    )

    $info = [PSCustomObject]@{
        ComputerName = $ComputerName
        OS = (Get-WmiObject Win32_OperatingSystem).Caption
        OSVersion = [System.Environment]::OSVersion.Version
        Architecture = (Get-WmiObject Win32_OperatingSystem).OSArchitecture
        Manufacturer = (Get-WmiObject Win32_ComputerSystem).Manufacturer
        Model = (Get-WmiObject Win32_ComputerSystem).Model
        Processor = (Get-WmiObject Win32_Processor).Name
        TotalRAM = "{0:N2} GB" -f ((Get-WmiObject Win32_ComputerSystem).TotalPhysicalMemory / 1GB)
        FreeRAM = "{0:N2} GB" -f ((Get-WmiObject Win32_OperatingSystem).FreePhysicalMemory / 1MB / 1024)
        Disks = Get-WmiObject Win32_LogicalDisk -Filter "DriveType=3" | Select-Object DeviceID,
            @{Name="Size(GB)";Expression={"{0:N2}" -f ($_.Size/1GB)}},
            @{Name="Free(GB)";Expression={"{0:N2}" -f ($_.FreeSpace/1GB)}}
        LastBoot = (Get-WmiObject Win32_OperatingSystem).ConvertToDateTime(
            (Get-WmiObject Win32_OperatingSystem).LastBootUpTime)
        Uptime = (Get-Date) - (Get-WmiObject Win32_OperatingSystem).ConvertToDateTime(
            (Get-WmiObject Win32_OperatingSystem).LastBootUpTime)
    }

    return $info
}

# Usage
Get-SystemInfo | Format-List
```

### Backup Script

```powershell
<#
.SYNOPSIS
    Backup files to specified location
#>

function Start-Backup {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$true)]
        [string]$Source,

        [Parameter(Mandatory=$true)]
        [string]$Destination,

        [int]$RetentionDays = 7
    )

    begin {
        $timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
        $backupName = "Backup_$timestamp"
        $backupPath = Join-Path $Destination $backupName
        $logFile = Join-Path $Destination "backup_$timestamp.log"

        Start-Transcript -Path $logFile
    }

    process {
        try {
            Write-Verbose "Starting backup from $Source to $backupPath"

            # Create backup directory
            New-Item -Path $backupPath -ItemType Directory -Force | Out-Null

            # Copy files
            Copy-Item -Path "$Source\*" -Destination $backupPath -Recurse -Force

            # Create archive
            $zipFile = "$backupPath.zip"
            Compress-Archive -Path $backupPath -DestinationPath $zipFile

            # Remove uncompressed backup
            Remove-Item -Path $backupPath -Recurse -Force

            Write-Host "Backup completed: $zipFile" -ForegroundColor Green

            # Clean old backups
            Get-ChildItem -Path $Destination -Filter "Backup_*.zip" |
                Where-Object { $_.CreationTime -lt (Get-Date).AddDays(-$RetentionDays) } |
                Remove-Item -Force

            Write-Host "Cleaned backups older than $RetentionDays days" -ForegroundColor Yellow

        } catch {
            Write-Error "Backup failed: $_"
            throw
        }
    }

    end {
        Stop-Transcript
    }
}

# Usage
Start-Backup -Source "C:\Data" -Destination "D:\Backups" -RetentionDays 7
```

### Service Monitor

```powershell
<#
.SYNOPSIS
    Monitor services and restart if stopped
#>

function Watch-Service {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$true)]
        [string[]]$ServiceName,

        [int]$IntervalSeconds = 60,

        [string]$LogPath = "C:\Logs\ServiceMonitor.log"
    )

    while ($true) {
        foreach ($service in $ServiceName) {
            $svc = Get-Service -Name $service -ErrorAction SilentlyContinue

            if ($svc) {
                if ($svc.Status -ne 'Running') {
                    $message = "$(Get-Date) - $service is $($svc.Status). Attempting restart..."
                    Write-Host $message -ForegroundColor Yellow
                    $message | Add-Content -Path $LogPath

                    try {
                        Start-Service -Name $service
                        $message = "$(Get-Date) - $service restarted successfully"
                        Write-Host $message -ForegroundColor Green
                    } catch {
                        $message = "$(Get-Date) - Failed to restart $service : $_"
                        Write-Host $message -ForegroundColor Red
                    }

                    $message | Add-Content -Path $LogPath
                } else {
                    Write-Host "$(Get-Date) - $service is running" -ForegroundColor Green
                }
            } else {
                Write-Warning "Service $service not found"
            }
        }

        Start-Sleep -Seconds $IntervalSeconds
    }
}

# Usage
Watch-Service -ServiceName "W32Time", "Spooler" -IntervalSeconds 30
```

### User Management Script

```powershell
<#
.SYNOPSIS
    Bulk user creation from CSV
#>

function New-BulkUsers {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$true)]
        [string]$CsvPath,

        [string]$DefaultPassword = "P@ssw0rd123!"
    )

    try {
        $users = Import-Csv -Path $CsvPath
        $securePassword = ConvertTo-SecureString $DefaultPassword -AsPlainText -Force

        foreach ($user in $users) {
            try {
                # Check if user exists
                if (Get-LocalUser -Name $user.Username -ErrorAction SilentlyContinue) {
                    Write-Warning "User $($user.Username) already exists"
                    continue
                }

                # Create user
                New-LocalUser -Name $user.Username `
                    -Password $securePassword `
                    -FullName $user.FullName `
                    -Description $user.Description

                Write-Host "Created user: $($user.Username)" -ForegroundColor Green

                # Add to group if specified
                if ($user.Group) {
                    Add-LocalGroupMember -Group $user.Group -Member $user.Username
                    Write-Host "  Added to group: $($user.Group)" -ForegroundColor Cyan
                }

            } catch {
                Write-Error "Failed to create user $($user.Username): $_"
            }
        }
    } catch {
        Write-Error "Failed to process CSV: $_"
    }
}

# CSV Format:
# Username,FullName,Description,Group
# jdoe,John Doe,IT Department,Administrators
# jsmith,Jane Smith,HR Department,Users

# Usage
New-BulkUsers -CsvPath "users.csv"
```

---

## Conclusion

This comprehensive PowerShell guide covers everything from basics to advanced topics. Key takeaways:

1. **Object-Oriented**: PowerShell works with objects, not just text
2. **Pipeline**: Powerful data processing with |
3. **Cmdlets**: Consistent Verb-Noun naming
4. **Remoting**: Built-in remote management capabilities
5. **Modules**: Extensible with modules and galleries
6. **Error Handling**: try-catch-finally for robust scripts
7. **Best Practices**: Comment-based help, parameter validation

### Additional Resources

- [Official PowerShell Documentation](https://docs.microsoft.com/powershell/)
- [PowerShell Gallery](https://www.powershellgallery.com/)
- [PowerShell GitHub](https://github.com/PowerShell/PowerShell)
- [Microsoft Learn - PowerShell](https://learn.microsoft.com/training/browse/?terms=PowerShell)
- [PSScriptAnalyzer](https://github.com/PowerShell/PSScriptAnalyzer) - Script analysis tool

### PowerShell Tools

```powershell
# Install PSScriptAnalyzer
Install-Module -Name PSScriptAnalyzer

# Analyze script
Invoke-ScriptAnalyzer -Path script.ps1

# Install Pester (testing framework)
Install-Module -Name Pester

# Run tests
Invoke-Pester
```

---

**Remember**: Always test scripts in a non-production environment, follow the principle of least privilege, and keep your PowerShell version updated!
