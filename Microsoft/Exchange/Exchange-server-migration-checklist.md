# Exchange server migration checklist

**Work in Progress**

## Before you start

> :exclamation: Active Directory Forest Functional Level (FFL): Must be **Windows Server 2008 R2 or newer** (FFL 2008 for Exchange 2013).
{.is-warning}


* [ ] Memory: For Mailbox role, 8GB memory minimum; **Recommend 16GB or higher**.
* [ ] Domain Controllers (DC): Must be **Windows Server 2008 R2 or newer**.
* [ ] Do **not** disable (IPV6).
* [ ] Existing Exchange server must be **Exchange 2010 SP3 or later**, (FFL 2007 for Exchange 2013).

## Compatibility table

| Exchange | Windows Server 2016 | Windows Server 2012 (R2) | Reasons |
| ------ | ------ | ------ | ----- |
| 2016 | :heavy_check_mark: | :heavy_check_mark: | |
| 2013 | :heavy_multiplication_x: | :heavy_check_mark: | Missing Server-Gui-Shell,Server-Gui-Mgmt-Infra features |

## Add server to domain

* [ ] Rename server to your desired name and restart
* [ ] Add Server to Domain
* [ ] Set Static IP Address or add Reservation to DHCP server

# 1. Configure disks

| Disk| MountPoint | File System |
| ------ | ------ | ------ |
| Page File | `D:\` | NTFS 64KB Cluster |
| Database | `C:\ExchangeDatabases\DB01` | ReFS |
| Database Log | `C:\ExchangeDatabases\DB01_Log` | ReFS |

## 1.1. PageFile disk

* [ ]  Create page file disk On disk D - format disk as NTFS with 64KB cluster size

Open `Win+R` then write `diskmgmt.msc`. Initialize and format Page File disk like in table above.

Page file sizes for each Exchange Server must be configured correctly. Each server should have the page file configured to be the amount of RAM, plus 10MB, up to a maximum of 32GB + 10MB.

### 1.1.1. Page file

* [ ] To configure the Page file size, right click on the Start Menu and choose System
* [ ] The system information window should open within the control panel. Choose Advanced system settings
* [ ]  Next, the System Properties window will appear with the Advanced tab selected. Within Performance, choose Settings

We will then adjust the Virtual Memory settings (In Advanced tab) and perform the following actions

* [ ] Unselect Automatically manage paging file size for all drives
* [ ] Set a page file size to match the current virtual machine RAM, plus 10MB, for example
* [ ] Restart server to apply memory changes

### 1.1.2. Example page file disk sizes

| RAM Size | Page file size |
| ----- | ----- |
| 8GB | (8 * 1024) + 10 = **8192 MB** |
| 16GB | (16 * 1024) + 10 = **16394 MB** |
| 32GB | (32 * 1024) + 10 = **32778 MB** |

## 1.2. DB and DB_Log disks

* [ ] Initialize new disks before start as **GPT** which is recommended for Exchange and supports disk sizes over 2TB, should they be required

> **The process to create the ReFS volume with the correct settings requires PowerShell.** Copy following script to `.ps1` file.
{.is-warning}


* [ ] **Check your disks number before you run this scripts.**

```powershell
function Format-ExchangeDisk
{
  param($Disk, $Label, $BaseDirectory=”C:\ExchangeDatabases”)
  New-Item -ItemType   Directory -Path   “$($BaseDirectory)\$($Label)”
  $Partition = Get-Disk -Number $Disk | New-Partition   -UseMaximumSize
  if ($Partition)
  {
    $Partition | Format-Volume -FileSystem ReFS -NewFileSystemLabel $Label -SetIntegrityStreams:$False
    $Partition | Add-PartitionAccessPath -AccessPath “$($BaseDirectory)\$($Label)”
  }
}
```

To format disk run `Format-ExchangeDisk -Disk <your-disk-number> -Label <desired-disk-label>`

* [ ] Create DB01 disk
* [ ] Create DB01_log disk

You can add more disks if you want or need them.

Examples based on table above

* to format disk 2 run `Format-ExchangeDisk -Disk 2 -Label DB01`
* to format disk 3 run `Format-ExchangeDisk -Disk 3 -Label DB01_Log`

# 2. Exchange prerequisites

* [ ] Install prerequisites

```powershell
Install-WindowsFeature NET-Framework-45-Features, RPC-over-HTTP-proxy, RSAT-Clustering, RSAT-Clustering-CmdInterface, RSAT-Clustering-Mgmt, RSAT-Clustering-PowerShell, Web-Mgmt-Console, WAS-Process-Model, Web-Asp-Net45, Web-Basic-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dir-Browsing, Web-Dyn-Compression, Web-Http-Errors, Web-Http-Logging, Web-Http-Redirect, Web-Http-Tracing, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Lgcy-Mgmt-Console, Web-Metabase, Web-Mgmt-Console, Web-Mgmt-Service, Web-Net-Ext45, Web-Request-Monitor, Web-Server, Web-Stat-Compression, Web-Static-Content, Web-Windows-Auth, Web-WMI, Windows-Identity-Foundation
```

* On a Windows Server 2012 R2 or Windows Server 2012 computer, run the following command.

```powershell
Install-WindowsFeature RSAT-ADDS
```
* Exception - follow this link [Windows Media Audio Voice Codec Component Not Installed](https://docs.microsoft.com/en-us/previous-versions/office/exchange-server-analyzer/bb871633(v=exchg.80)?redirectedfrom=MSDN)

```powershell
Install-WindowsFeature Desktop-Experience
```

* [ ] Reboot server after instalation Desktop experience feature

* On a Windows Server 2008 R2 SP1 computer, run the following command.

```powershell
Add-WindowsFeature RSAT-ADDS
```

* Reboot server
* Download and install the [.Net Framework 4.5.2](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net452-web-installer) or [.Net Framework 4.7](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net47-web-installer) for CU23

Offline installer is [Here](https://support.microsoft.com/en-us/topic/microsoft-net-framework-4-7-2-offline-installer-for-windows-05a72734-2127-a15d-50cf-daf56d5faec2)

For Server Exchange Server 2016 or 2013 CU23

* Download and install the [Microsoft Unified Communications Managed API Core Runtime, version 4.0](https://www.microsoft.com/en-us/download/details.aspx?id=34992)

* Download the [Visual C++ Redistributable Package for Visual Studio 2013](https://www.microsoft.com/download/details.aspx?id=40784)

# 3. Prepare Active directory

* This step is irreversible; therefore, it is essential that a full backup of Active Directory is performed before we perform this step.
While logged on as a domain user that's a member of the Enterprise Admins and Schema Admins, launch an elevated command prompt

```powershell
setup.exe /PrepareSchema /IAcceptExchangeServerLicenseTerms
```

Expect the schema update to take between 5 and 15 minutes to execute.

* [ ] Next prepare Active Directory. This will prepare the Configuration Container of our Active Directory forest, upgrading the AD objects that support the Exchange Organization. We'll perform this preparation using the following command.

```powershell
setup.exe /PrepareAD /IAcceptExchangeServerLicenseTerms
```

* [ ] Final step to prepare Active Direcotry is to run the Domain Preparation. Smaller organizations has only one domain so we can run the following command:

```powershell
setup.exe /PrepareDomain /IAcceptExchangeServerLicenseTerms
```

If you have more than one domain within the same Active Directory forest with mail-enabled users, then you will need to prepare each domain. The easiest way to prepare multiple domains is to replace the `/PrepareDomain` switch with `/PrepareAllDomains`.

# 4. Exchange 2016 Setup

* [ ] To install Exchange via setup.exe we will use the /Mode swithc to specify that we will be preforming install. In addition to the `/Mode` switch we need to specify role that we will install.

```powershell
setup.exe /Mode:Install /Roles:Mailbox,ClientAccess /IAcceptExchangeServerLicenseTerms
```

* [ ] After successfull installation restart server.

# 5. Postinstallation setup (IN PROGRESS)

* [ ] Configure Outlook Anywhere 

```powershell
Get-ExchangeServer -Identity "EX2K13" | Enable-OutlookAnywhere -DefaultAuthenticationMethod NTLM -ExternalHostname "mail.smmi.sk" -SSLOffloading $false
```

* [ ] Internet recieve connector

```powershell
Get-ExchangeServer "EX2K13" | New-ReceiveConnector -Usage Internet -Bindings 192.168.1.26:25 -Name "External recieve Connector"
```

# Trouble Shooting

## Server Recovery

Write down old serveer name and IP configuration

* First we need to reset the computer account from AD
- Deploy new operating system with windows updates
- Assign same ip address
- Join to domain with the same name as before
- Assign same drive letters on the server [1. Configure disks](#h-1-configure-disks)
- Install pre-requisites which is required [2. Exchange prerequisites](#h-2-exchange-prerequisites)
- We will use Setup `/m:RecoverServer` mode installation

## Database repair

> Run with Powershell with elevated rights from `C:\Program Files\Microsoft\Exchange Server\V15\Mailbox\` folder
{.is-warning}

> Path above is standard installation path. Change it as you need. Version __V15__ is version for Exchange server 2013
{.is-info}


To check database status run following command

```bash
Eseutil.exe /mh C:\path\to\database\database_name.edb
```

Check the state and if you see something like `dirty shytdown`, the database si incosinstent or corrupt and requires recovery.

### Soft recovery

To preform soft recovery use following command

```bash
eseutil /r <log file index> /l "<path to log files>" /d "<path to database>"
```

- `<log file index>` is name of log file something like `E00`
- `<path to log files>` Full path to folder where log files are stored
- `<path to database>` full path to folder where databases are stored

### Hard recovery

> Backup your database !!!
{.is-danger}


```bash
eseutil /p C:\path\to\database\database_name.edb
```

### Database Defragmentation

```bash
eseutil /D C:\path\to\database\database_name.edb
```

## Qeueues & Messages

List all queues

```bash
Get-Queue -Server <ServerName>
```

List first 100 messages in Queue, to list all remove `-ResultSize`

```bash
Get-Message -Server <ServerName> -ResultSize 100 -SortOrder +FromAddress
```

Show last error on mesages in queue

```bash
Get-Message -Server <ServerName> -ResultSize 100 -SortOrder +FromAddress | select FromAddress,lastError
```

# Maintenance

## Certificate management

> Certificate must be PFX file which contains public and private key
{.is-warning}

> For more information check https://docs.microsoft.com/en-us/powershell/module/exchange/import-exchangecertificate?view=exchange-ps
{.is-info}


To import certificate trough powershell use command as follows:

```bash
>Import-ExchangeCertificate -Server EXCHANGE-SERVER -FileName C:\\path\to\your\certifikate.pfx -Password (ConvertTo-SecureString -String "pa$$word" -AsPlainText -Force)
```

Right after import you will see certificate thumbrpint. Write it down, you will need it later. If you didn't do that you can list all certificates with `Get-ExchangeCertificate`.

## Replace od certificate with new one

Now when you have imported certificate set it as default for required services.

```bash
Get-ExchangeCertificate -Thumbprint Thumbprint | Enable-ExchangeCertificate -Server EXCHANGE -Services SMTP,IMAP,IIS
```

## Adding new mailboxes

To create new Mailbox use

```bash
New-Mailbox -Name <Name> -UserPrincipalName <UPN> -Password (ConvertTo-SecureString -String '<Password>' -AsPlainText -Force) [-Alias <Alias>] [-FirstName <FirstName>] [-LastName <LastName>] [-DisplayName <DisplayName>] -[OrganizationalUnit <OU>]
```

Or when you have already existing user and want to create new mailbox for him, you can do this as follows

```bash
Enable-Mailbox -Identity <identity> -Database <database>
```