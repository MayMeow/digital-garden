# Update user GUID to match existing user in Office 365

Let's see what to do when azure ADSync synchronize you on Premise AD user to azure to `@domain.onmicrosoft.com`. In most cases it was done when i have already existing user `username@mydomain.tld`. Fortunately this is fixable problem. So what to do?

First of all you will need to find ObjectGUID of users which you want update. This can be retrieved form you local AD. But not that one you can find in MMC if you check users atribute. This format is not accepted with Azure AD, so there is another way to obtain it. We use tool called  **LDIFDE**.

This tool can export/inport data from/to Active Directory. You can find more about it here https://support.microsoft.com/en-us/help/555636.

So let's export users data to txt file by following command

## Getting ObectGuid

```powershell
ldifde -d “DistinguishedName of the user” -f “c:\temp\exporteduser.txt”
```

```powershell
# output
uSNChanged: 129668
name: Emma Wolf
objectGUID:: WhJMX8r25UigMnvHO/u3Ew==
userAccountControl: 512
badPwdCount: 0
```

So and this is right format which we will need to set up to account in Azure AD in case of sinchronization. But first you will need remove existing account. 

## Updating our Azure AD user

You will need to connect to office 365 via Powershell. So if you dont have MSOnline extension install it

```powershell
Install-Module MSOnline
# load module
Import-Module MSOnline
```

> Not working with Powershell v7 (one you can download from github)
{.is-info}


Connect to Your Office 365

```powershell
Connect-MsolService 
```

Update Guid to your existing user

```powershell
set-msoluser -userprincipalname emma@yourdomain.com -ImmutableID xxx
```

If there exists user which already using this id you will getting error on output so at first you will need remove existing user with this id.

Stop synchronization betweehn AD and Azure AD (by opening azure AD configuration)

To get existing user use:

```powershell
 Get-MsolUser -All | Where-Object {$_.ImmutableID -eq "WhJMX8r25UigMnvHO/u3Ew=="}
```

If user was deleted is perhaps in recycle Bin and is not returned in output so use following command to show it

```powershell
Get-MsolUser -All -ReturnDeletedUsers | Where-Object {$_.ImmutableID -eq “WhJMX8r25UigMnvHO/u3Ew==”}
```

This command give you same result but for deleted users.

OK. Now remove this user

```powershell
Remove-MsolUser -UserPrincipalName "emma@domain.onmicrosoft.com" -Force
```

and remove him from recycle bin

```powershell
Remove-MsolUser -UserPrincipalName "emma@domain.onmicrosoft.com" -RemoveFromRecycleBin
```

Thats all. Now close ADSync config screen and wait until synchronization si conplete or in powershell run ` Start-ADSyncSyncCycle'.

## Close Session

```powershell
[Microsoft.Online.Administration.Automation.ConnectMsolService]::ClearUserSessionState()
```