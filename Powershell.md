# Powershell Notes

# Command Research

### Get-Help \<command\>
    Display help file for the specified command
    Use help to paginate the information
    -Full: display all info
    -Parameter: Show all parameters for the specified command
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
    Get-Alias -Definition Get-ChildItem         # the aliases for the Get-ChildItem cmdlet
    New-Alias -Name "List" Get-ChildItem        # create new alias for Get-ChildItem
    Set-Alias -Name list -Value Get-ChildItem   # change alias
```
### about_ Files
    High-level Help Information about broad topics
```powershell
    Get-Help about_Aliases                      # access about_ file for Alias information
    Get-Help about_*                            # see all possible about_ topics
```
# Refining Results

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



# File System
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
    Creates table-like custom objects from the items in a csv file

### Export-Clixml \<filepath\>
    Creates an XML-based representation of an object(s) and stores it in a file. Clixml is best way to retain all properties of an object through export and import.

### Import-Clixml \<filepath\>
    Imports a Clixml file and creates corresponding objects in Powershell.

### ConvertTo-CSV
### ConvertFrom-CSV
### ConvertTo-Clixml
### ConvertFrom-Clixml

### Out-File \<filepath\>
    Writes output of cmdlet to specified file
        -Append appends the content to a file
        -NoClobber don't overwrite an existing file

```powershell
    Get-ChildItem | Out-File "./DirContents.txt"                # save directory contents to file
    Get-Process | Out-File "processes.txt" -Append              # append the current output to a file
```

# Iteration
## For-Each
    ***TODO***


# Variables

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
Adding to the file.