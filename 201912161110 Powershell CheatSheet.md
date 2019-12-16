# 201912161110 Powershell CheatSheet
tags = #Powershell, #Documentation


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
