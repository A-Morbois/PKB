# 201912161110 Powershell CheatSheet
tags = #Powershell, #Documentation, #SofwareDevelopment, #IT


## Get to know your object
`Get-Service | Get-Member`


## Dig within object 
`$(Get-Service |Group-Object Status -AsHashTable -AsString).running `
OR 
`Get-Service |Group-Object Status | Where {$_.name -eq "running"} |Select-Object -ExpandProperty Group`


## Filter 
`$(Get-Service |Group-Object Status -AsHashTable -AsString).running  | Where-Object {$_.name -match "wlan"}`

## Filter more
`Get-Service |group {$_.status -eq "running" -AND $_.canstop}`


## better groupping :
`Get-Service |Group-Object serviceType,  Status`


-----
## Find your histoy :
`(Get-PSReadlineOption).HistorySavePath`

## Find something in your historic with a specific match
`C:\Users\morboa-ext\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt |where {$_ -match "PSCode"}`
 
 
-------
## Adding or removing time / date
`$TimeToAdd = New-TimeSpan -Days 3`
`set-date  -Adjust $TimeToAdd`
// OU
`$(get-date).AddDays(3)`


----------------

## Foreach-object ( % )
`foreach-object {Run at the start}{Run for each object in the loop}{Run at the end}`

**List the function of your module** 
`$(Get-Module <ModuleName>).ExportedCommands.Keys | % {$_}`


## Manifest of a module

file.psd1


## Count the number of line 
`$b = 0; Get-ChildItem .\ -Recurse -File -filter "*.ps1" |Select-Object -Property Directory, Name | % { $b += $(get-content  "$($_.Directory)\$($_.Name)").count } ; write-host $b`

## Get Your script location
`$Currentlocation = $(get-item  -path "$($MyInvocation.MyCommand.Source)").DirectoryName`

## split line of your sting
`$line = $info.split([Environment]::NewLine)`