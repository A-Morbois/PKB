# 201912161113 Powershell Credential
tags = #Powershell , #Documentation, #Credential, #SofwareDevelopment, #IT

Credential 

$username = "antoine.morbois@acme.com"
$password = "Abcd1234"
$secureStringPwd = $password | ConvertTo-SecureString -AsPlainText -Force 
$secureStringText = $secureStringPwd | ConvertFrom-SecureString
Set-Content "C:\temp\ExportedPassword.txt" $secureStringText


$AESKeyFilePath = "C:\Users\morboa-ext\Documents\WindowsPowerShell\"
$AESKey = New-Object Byte[] 32 [Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($AESKey)

Set-Content $AESKeyFilePath $AESKey 

$password = $passwordSecureString | ConvertFrom-SecureString -Key $AESKey
Add-Content $credentialFilePath $password*


-#####
$username = "reasonable.admin@acme.com.au"
$AESKey = Get-Content $AESKeyFilePath
$pwdTxt = Get-Content $SecurePwdFilePath
$securePwd = $pwdTxt | ConvertTo-SecureString -Key $AESKey
$credObject = New-Object System.Management.Automation.PSCredential -ArgumentList $username, $securePwd