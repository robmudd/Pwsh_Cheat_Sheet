# <p style="text-align: center;"> Powershell Cheat Sheet



# <p style="text-align: center;">Command Research


### Get-Help \<command\>
    Display help file for the specified command
    Use help to paginate the information
    -Full: display all info
    -Parameter *: Show all parameters for the specified command
    -Example: give examples of usage

### Get-Member (gm)
    Displays all details about the provided object type
    Good way to see all properties of an object type (as well as methods)
    Piped from other commands:
```powershell
    Get-ChildItem . | gm        # returns all properties of gci
```
### Get-Command \<pattern\>
    Find all commands fitting a certain pattern
```powershell
    Get-Command move-*       
    Get-Command *hash*              # finds Get-FileHash
```


### Get-Alias \<pattern\>
    Gets the aliases for the current session.
```powershell
    Get-Alias ls                                # get the cmdlet for the ls alias
    Get-Alias -Definition Get-ChildItem         # the aliases for Get-ChildItem
    New-Alias -Name "List" Get-ChildItem        # create new alias for Get-ChildItem
    Set-Alias -Name list -Value Get-ChildItem   # change alias
```
### about_ Files
    High-level Help Information about broad topics
```powershell
    Get-Help about_Aliases                      # access about_ file for Alias information
    Get-Help about_*                            # see all possible about_ topics
```
# <p style="text-align: center;"> Refining Results

### Select-Object (select)
    Select what properties of the object you want to display (columns)
    Select how many results to display

```powershell
    Select-Object Name, Id, CPU     # show only the Name, ID, and CPU columns
    Select-Object -First 10         # similar to head command
```
### Where-Object (where)
    Filter objects out of results

```powershell
    Get-Process | Where-Object -Property Name -EQ -Value "WUDFHOST"         # spelled out
    Get-Process | Where-Object  Name -eq WUDFHOST                           # more informally
```


### Sort-Object (sort)
    Sorts object by property value.  
    Defaults to ascending sort (use -Descending to reverse order)
    Can sort on multiple properties (separate with commas, priority first)
```powershell
    Get-Process | Sort -Property CPU -Descending            # sort processes by CPU usage
    Get-Process | Sort -Property Name, CPU -Descending      # sort processes by name, if name matches sort by CPU
```

# <p style="text-align: center;"> Conditionals

## Basic Conditional Syntax

```powershell
    $condition = $true
    if ( $condition ) {                         # check condition
        Write-Output "The condition was true"   # do something
    }                 
```

## Basic Comparison Operators

    Comparison Operators:
        -eq         equal
        -ne         not equal
        -gt/lt      greater than/less than
        -ge/le      greater or equal/less or equal
        
    Case sensitive string comparisons:
        -ceq, -cne, -cgt, -clt, -cge, -cle

## Wildcard matching
    
    -Like       allows to use wildcards:
                    ? matches any single character
                    * matches any number of characters 
    
```powershell
$value = 'S-ATX-SQL01'
if ($value -like 'S-*-SQL??') {
    Write-Host "Match found"
}
```

    Variations:
        -ilike          case insensitive
        -clike          case sensitive
        -notlike        not matched
        -inotlike       case insensitive not matched
        -cnotlike       case sensitive not matched

## REGEX matching

    -Match      allows use of REGEX for matching

```powershell
$value = 'S-ATX-SQL01'
if ( $value -match 'S-\w\w\w-SQL\d\d') {
    Write-Host "match found"
}
```


# <p style="text-align: center;"> File System
### Get-ChildItem (gci, ls, dir)
    Gets the items and child items in one or more specified locations.
    Similar to ls in bash
```powershell
    Get-ChildItem                                       # returns contents of current directory
    Get-ChildItem C:\Users\rob\Desktop\file.txt         # returns entry for file.txt
    Get-ChildItem *.txt -Recurse                        # all txt files in current directory and all child directories         
```

### Get-PSDrive
    Get drives in the current session
```powershell
    Get-PSDrive                     # see all available PSDrives
    cd hkcu:                        # change to HKEY_CURRENT_USER PSDrive
    cd env:; ls                     # change to environment PSDrive, list all items
```


## Export-Import Files

### Get-Content \<filepath\>
    get the contents of a text file

### Export-CSV \<filepath\>
    Export objects as csv strings and saves the strings to a file
```powershell
Get-Process | Export-CSV processes.csv
```

### Import-CSV \<filepath\>
> Creates table-like custom objects from the items in a csv file

### Export-Clixml \<filepath\>
> Creates an XML-based representation of an object(s) and stores it in a file. Clixml is best way to retain all properties of an object through export and import.

### Import-Clixml \<filepath\>
> Imports a Clixml file and creates corresponding objects in Powershell.

### ConvertTo-CSV
### ConvertFrom-CSV
### ConvertTo-Clixml
### ConvertFrom-Clixml

### Out-File \<filepath\>
- Writes output of cmdlet to specified file  
- Parameters  
    - \-Append: appends the content to a file  
    - \-NoClobber: don't overwrite an existing file  

```powershell
    Get-ChildItem | Out-File "./DirContents.txt"                # save directory contents to file
    Get-Process | Out-File "processes.txt" -Append              # append the current output to a file
```

# <p style="text-align: center;"> Iteration

## Foreach 

> - *Syntax: foreach \$\<item\> in $\<collection\> { code here referencing $item }*
>
> - This is a **Statement**, not a cmdlet. 
>
> - Iterate over the values of a collection.
>
> - Iterate over every value in collection, with temp variable $item as the current value

```powershell
    $array = '1', 'two', '3'
    foreach ($value in $array) {Write-Host $value}      # loops three times, printing 1, two, and 3  

    foreach ($file in Get-ChildItem) {                  # iterate over all gci results, only display certain results
        if ($file.Length -gt 100KB) {
            Write-Host $file
            Write-Host $file.Length
            Write-Host $file.LastAccessTime
        }
    }
```

## ForEach-Object
> - This is a *Cmdlet* -- it receives input from the pipeline
>
> - Take action on each value using the $_ operator   
>
    Parameters:  
        -Process \{ code block \}- What action to take for each object (positional argument)
        -Begin \{ code block \}- What action to take before processing any objects
        -End \{code block \}- What action to take after processing all objects

```powershell

    35, 12, 22 | ForEach-Object -Process { $_ / 2 }    # outputs 17.5, 6, 11

    33, 12, 10 | ForEach-Object -Begin {echo "hi"} -End {"bye"} -Process {echo "hello"}
                                    # Outputs hi, hello, hello, hello, bye


    # Getting formatted file hashes for all items in present working directory
    Get-FileHash * -Algorithm MD5 | ForEach-Object 
        
        -begin {
        "Evidence Hashes:"
        } 
        
        -process {
        $hash=$_.Hash; 
        $hash = $hash -replace '(....(?!$))', '$1-';
        $path = $_.Path; "$hash `t`t$path"
        }
```

# <p style="text-align: center;"> Variables

## Get-Variable
    Get all variables in the current session (not necessarily env variables, though)

```powershell
    $$                # previous command
    $?                # result of previous command
    $env:Path         # path
    $PSVersionTable
    $profile          # script that runs automatically when powershell opens
```

## Variables
   ### Storing / Recall 
    Prefix is $
   
```powershell
    $var = 'foobar'
    $var                                # foobar
    $var = 'one','two','three'
    $var                                # one \n two \n three
    $var[1]                             # two
    $var.count                          # 3
    $var.length                         # 3
```
### Storing Object(s) in Variables
    Can store an entire object(s)
```powershell
    $processes = Get-Process                                                        
    $biggest_process = Get-Process | Sort-Object -Property CPU | Select -First 1
    
    $processes                              # returns multiple process object
    $processes                              # returns a single process object
```

### Quotes
    Single quotes stores everything as string literal (ignores special characters)
    Double quotes translates special characters for you

```powershell
    $var = 'foobar'
    Write-Host "My variable is $var"            # My variable is foobar
    Write-Host 'My variable is $var'            # My variable is $var
```

### Declaring Variable type
    Can cast variables using brackets

```powershell
    [int] $number = '100'
    $number | gm                # shows this to bee Int32 despite originating from String
```

### Remove-Variable
    Delete a variable




# <p style="text-align: center;"> Input - Output

### Read-Host

    Get user input
    Syntax: $var = read-host ["User Prompt"]

### Write-Host

    Write to terminal

        It writes directly to the terminal and does not pass objects along the pipeline

    Parameters:
        -ForegroundColor
        -BackgroundColor
            Use with $PSStyle.reset to ensure formatting doesn't bleed over

```powershell
Write-Host "Hello There" -ForegroundColor magenta -BackgroundColor yellow $PSStyle.reset
```

### Write-Output

    Writes an object into the pipeline (unlike Write-Host which writes it directly to the screen)
```powershell
    Write-Output "Hello"    # similar to write-host "Hello", this displays hello
                # writes the string Hello into the pipeline
                # we are at end of pipeline, so it is written to Out-Default
                # Out-Default writes to Out-Host and it is written to Screen
```
    Here the pipeline has the ability to act on the data before pushing to screen

```powershell
    Write-Output "hello" | foreach {$_.toUpper()}   # Outputs HELLO
    Write-Host "hello" | foreach {$_.toUpper()}     # Outputs hello
```

### Other write modes:
    These will write if their configuration variable is set to Continue.  
    If set to silently continue they will display no output
    
    Write-Warning       $WarningPreference
    Write-Verbose       $VerbosePreference
    Write-Debug         $DebugPreference
    Write-Error         $ErrorActionPreference
    Write-Information   $InformationPreference

    $ErrorActionPreference can also have values including Ignore, Inquire, Stop, Break

# <p style="text-align: center;"> Regex and Strings
### Regex Quick Guide
    \d          digit [0-9]
    \D          any non-digit [^0-9]
    \w          alpha numeric [a-zA-Z0-9_]
    \W          any non word character [^a-zA-Z0-9]
    \s          whitespace character
    \S          any non-whitespace character
    .           any character
    ()          sub-expression
    \           escape the next character
    {5}         Exactly 5 of the preceeding character types
                    ie '\d{3}' matches '987'
    {5,}        5 or more of the previous character types
    {5,6}       Between 5 and 6 of the previous character types (inclusive)
    *           Zero or more of the previous character
                    '\d*-' matches '-'  and '1234-'
    ?           Zero or one of the previous character
                    'ba?d' matches 'bad' and 'bd'
    +           One of more of the previous character
                    '\d+abc' matches '1abc' and '12abc' but not 'abc'
    [^]         Not
                    '[^b,d]ad' matches 'fad' but not 'bad' or 'dad'
    ^           Start of String
    $           End of String

    (?=)        Positive Lookahead (advanced)
                \d(?=\w)    
                            matches a decimal followed by a word char, but the 
                                word char is NOT part of the match, therefore...
                \d(?=\w)\D
                            matches a decimal followed by a word char, then checks to
                                ensure that the next char is not a decimal, so 
                                '1a' matches
                                '12' does not match

### Select-String with Regex

    Search files to find matching text patterns in them
    Use -Pattern parameter to provide a REGEX pattern for matching
    Use -SimpleMatch to ignore REGEX


```powershell
    Get-ChildItem $path -Recurse | Select-String -Pattern '\d{3}-\d{2}-\d{4}'
            # Get all files with an SSN within the $path directory and subdirectories
```

### Comparisons

    -match          compare using REGEX

    -like           compare using simplified wildcard syntax (only * and ? as wildcards)

```powershell
    if ('123abc' -match '\d{3}\w{3}' ) {            # returns true
        Write-Host "Condition True"
    }    

    if ('123abc' -like '*3a*' ) {         # returns true...would not work for -match
        Write-Host "Condition True"
    }
```

    match and like can be used to search array values for matches

```powershell
    $data = 'abc', 'def', 'abcd', 'defg'
    $data -match '\wbc'                     # returns 'abc' and 'abcd'
    $data -like '*d*'                       # returns 'def','abcd', and 'defg'
```

    variations: imatch, cmatch, notmatch, ilike, clike, notlike

### Replacement (modifying strings)

    -Replace operator can perform string modification using regex pattern matching

    Syntax: <string> -Replace <pattern_to_find>, <replacement_pattern>

    Variants: -creplace, -ireplace

```powershell

    $str = '123Hello'
    $str -replace '\d{3}', ''       # outputs Hello but does not modify $str

    $str = $str - replace '\d(?=\D)(\w)', 'xx'      # now $str = '12xxello'

    $hash = (Get-FileHash file.exe).hash
    $hash = $hash -replace '(....)', '$1-'   # add a dash to every 4 chars of a hash
    $hash = $hash -replace '(.$)', ''       # remove trailing dash

```

### String.Contains()

    Checks to see if a string contains a specified substring

    Does NOT allow regex.  Simple matching only.

    Faster than other forms of searching (but, again, no REGEX)

```powershell
    $str = '123abc'
    $str.Contains('3ab')            # returns true
```

### Split

    Splits a string into substrings given a delimiter character/pattern

    Syntax: <string> -split <pattern>

    Variants: isplit, csplit 

```powershell
    $str = 'Word1,wOrd2,woRd3'
    $str -split ','             # outputs word1 word2 and word3 separately (as array)

    ($str -split '.').count     # outputs 18 ... char count including new line

    $str -csplit [A-Z]          # outputs ord1,w rd2,wo and d3

```

### Switch

    Switch Command is regex capable with -Regex parameter

    Syntax: switch -regex ($variable_to_check)

```powershell
    switch -regex ($string) {

        '\d{3}-\d{2}-\d{4}' {
            Write-Host 'Contains Possible SSN'
        } 'd{4}-d{4}-d{4}-d{4}' {
            Write-Host 'Contains Possible Credit Card'
        } 'd{3}-d{3}-d{4}' {
            Write-Host 'Conatins possible Phone Number'
        }
    
    }
```

### $Matches variable and Named Matches (Advanced)
    
    Use of the -match operator creates a $matches variable containing all identified matches
        This is relevant when we have sub-expressions in our regex (using parenthesis)

```powershell
    $str = 'hello from me'
    $str -match '(^hel).*(me$)'
    $matches                        # outputs 'hello from me', 'hel', and 'me
    $matches[0]                     # outputs 'hello from me'     
```

    We can name our matches with ?<Match_Name> ...substitute desired name for Match_Name

```powershell
    $str = 'hello from me'
    $str -match '($<Beginning>^hel).*(?<End>me$)'
    $matches.beginning              # outputs 'hel'
    $matches.ednd                   # outputs 'me'
    $matches[0]                     # outputs 'hello from me'
    $matches[1]                     # no output     
```

# <p style="text-align: center;"> Jobs

### Start-Job

    Begin Running a job in the background (in a new process)

    Syntax: Start-Job -scriptblock { <command name> }

    Parameters:
        -ScriptBlock: specify the script/command to run
        -FilePath:      specify a script file to run in lieu of a scriptblock

### Start-ThreadJob

    Begin running a job in a new thread of the existing process (fast than start-job)

    Syntax: Start-ThreadJob -scriptblock { <command name> }

### Get-Job

    Show all jobs running

    Will show you if job finished running and if there is output waiting to give you

### Receive-Job

    Get the pending results of any completed job

    -Keep parameter: get the output of the job but keep it available for future reference

### Stop-Job
    Terminate Job but keep output available to user

### Remove-Job
    Terminate job and remove any cached output
