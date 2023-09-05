# Restic installation

Restic is a modern backup program that can back up your files:

```bash
wget https://github.com/restic/restic/releases/download/v0.12.1/restic_0.12.1_linux_amd64.bz2
bzip2 -d restic_0.12.1_linux_amd64.bz2
chmod +x restic_0.12.1_linux_amd64
mv restic_0.12.1_linux_amd64 /usr/local/bin/restic
```

try if restic is available

```bash
restic version

# restic 0.12.1 compiled with go1.16.6 on linux/amd64
```

## Configuration

Here is example configuration for Backblaze B2

```text
export B2_ACCOUNT_ID="{account_id}"
export B2_ACCOUNT_KEY="{account_key}"
export RESTIC_PASSWORD="{restic_password}"
export RESTIC_REPOSITORY="b2:{bucket-name}"
```
 
 If you want to use more backup locations you cane remove `RESTIC_REPOSITORY` with `-r` argument.
 
 `RESTIC_PASSWORD`: is password to decrypt your backups, so keep it safe.
 
 ## More infromations
 
- https://restic.readthedocs.io/en/stable/030_preparing_a_new_repo.html
- https://restic.readthedocs.io/en/latest/045_working_with_repos.html