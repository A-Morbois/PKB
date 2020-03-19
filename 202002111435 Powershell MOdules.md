# 202002111435 Powershell Modules
tags = #Powershell


## Get to know the module you can import

`Get-Module -ListAvailable`

## Get to know where to store your modules locally

`$env:PSModulePath -Split ';'`

## Import remote module
`$AdminServer = New-PSSession -ComputerName $AdminServerName -Credential (Get-Credential)`
` Import-Module -Name ActiveDirectory -PSSession $AdminServer -Prefix 'Rmt'`

* prefix option will prefix all function from the imported module

* psd1 file =  Metadata of your module (automate its creation with New-ModuleManifest)

## Find the module you need 

`Find-Module -name <>*`