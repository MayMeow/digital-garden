# User hacks

> This working only on local users
{.is-warning}

## Hack utilman.exe

In case you forget you admin password and lock you from you computer. Boot your computer from Windows Instalation CD or USB and press `Shift` + `F10`. And use following commands

```bash
move c:\windows\system32\utilman.exe c:\windows\system32\utilman.exe.bak
copy x:\windows\system32\cmd.exe c:\windows\system32\utilman.exe
```

Now reboot you coputer.

## Reseting existing User password

At the Login Screen click the EASE OF ACCESS icon (beside the Power icon in the bottom right corner of the screen). Because of step reset-windows-10-password-create-admin4, this will launch a CMD window

Use following command to reset password

```bash
net user <change-to-your-user> <change-to-desired-password>
```

> If you dont remember username you can use `net user` to show all users on computer. (Username is first column)
{.is-info}

Now restart Windows 10 and login with the user account with the new password!

> This created a local administrator named `TEST`. If you are on a domain use the username `.\test` and no password.
{.is-info}

> Do not forget to revert back utilman exe by booting With install disk as you did at start.
{.is-danger}

## Creating new user

You can also add new user this way. 

```bash
net user <change-to-user-name> /add
net localgroup administrators <already-created-user> /add
```

and Reboot your computer.
