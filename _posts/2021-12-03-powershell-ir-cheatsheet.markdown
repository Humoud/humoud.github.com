---
layout: post
title:  "PowerShell: IR Cheatsheet"
date:   2021-12-03 00:00:00 +0300
categories: dfir
---



# Sessions
```powershell
# Enter credentials
$cred = Get-Credential
# Establish Session using explicit credentials
$machine1 = New-PSSession -ComputerName machine1 -Credential $cred
# Establish Session using current user's credentials
$machine1 = New-PSSession -ComputerName machine1
# Enter Interactive Session
Enter-PSSession -Session $machine1
# Remove Session
Remove-PSSession -Session $machine1
```

# Acquiring Files From Remote Machine
```powershell
# copy file
Copy-Item -FromSession $machine1 C:\Windows\Temp\interesting.dll -Destination D:\cases\1\artifacts\
# copy directory
# issue: does not copy hidden files
Copy-Item  -FromSession $machine1 -Path "C:\Windows\Temp" -Destination "D:\cases\1\artifacts\windows\temp\" -Recurse
```

# Copy Tools to Remote Machine
```powershell
# Create folder
Invoke-Command -Session $machine1 -ScriptBlock {mkdir C:\tools}
# Copy tools
Copy-Item .\rule.yar -Destination "C:\tools\" -ToSession $machine1
Copy-Item .\yara64.exe -Destination "C:\tools\" -ToSession $machine1
Copy-Item .\autorunsc64.exe -Destination "C:\tools\" -ToSession $machine1
```

# Execute Commands on Multiple Machines
```powershell
Invoke-Command -Session $machine1,$machine2 -ScriptBlock {whoami}
```

# Operation on Files and Folders
## Create Archive
```powershell
# fails if it tries to archive files being used
# maybe push and use 7zip?
Compress-Archive -Path 'C:\Windows\Temp' -DestinationPath "D:\cases\1\artifacts\windows_temp.zip"
```

## Modify File Attributes
```powershell
# remove hidden
(get-item "file.exe" -force).Attributes -= 'Hidden'
# remove readonly
(get-item "file.exe" -force).Attributes -= 'ReadOnly'
```

## Get Directory Size
```powershell
Get-ChildItem C:\folder\ | Measure-Object -Property Length -sum
gci C:\folder | Measure-Object -Property Length -sum
```

## Grep File Contents
```powershell
Get-ChildItem -Path 'DIRECTORY' -Recurse | Select-String "STRING"
gci -Path 'DIRECTORY' -Recurse | Select-String "STRING"
```

## Grep File and Folder Names
```powershell
# Show all files with names ending with .exe
gci -Path 'C:\Temp' -Filter *.exe
# Show me all files and folders with name containing "sep" in the current directory
gci -Filter *sep*
# ls works as well
ls -Filter *sep*
# Recursively
ls -Filter *sep* -Recurse
```

# Useful Commands\Misc
```powershell
# check cpu utilization
(Get-CimInstance Win32_Processor).LoadPercentage
# Check Windows OS
(Get-WMIObject win32_operatingsystem).name
# Show logged in users
query user /server:$SERVER
```